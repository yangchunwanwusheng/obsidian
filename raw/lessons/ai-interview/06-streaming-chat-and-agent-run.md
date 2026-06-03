---
type: lesson
tags: [SSE, 流式对话, ARQ, Agent Run, 异步任务]
created: 2026-05-25
updated: 2026-05-25
difficulty: advanced
prerequisites: [FastAPI异步编程, Redis, SSE协议]
topic: 开源项目深度拆解
status: completed
series: { name: "ai-interview-伯乐深度拆解", part: 6 }
---

# 流式对话与 Agent Run 系统

> 拆解伯乐的两个核心通信机制：实时对话的 SSE 流式传输，和长时间 Agent 任务的 ARQ 异步执行系统。

---

## 一、SSE 流式对话服务

### 1.1 为什么用 SSE 而不是 WebSocket？

| 特性 | SSE | WebSocket | 伯乐的选择 |
|------|-----|-----------|-----------|
| 方向 | 单向（服务端→客户端） | 双向 | **SSE**（对话场景服务端推送为主） |
| 协议 | HTTP（兼容性好） | 独立协议（需要升级握手） | **SSE**（Nginx 代理更简单） |
| 断线重连 | 浏览器原生支持 | 需手动实现 | **SSE**（内置重连） |
| 适用场景 | 实时推送（股票、日志） | 实时双向（聊天、游戏） | 对话用 SSE，语音用 WebSocket |

### 1.2 核心代码拆解

`src/services/chat_stream_service.py`（约 900 行）是流式对话的核心：

```python
async def stream_chat(
    agent: BaseAgent,
    messages: list,
    conversation_id: str,
    thread_id: str,
    user_id: str,
    context: BaseContext,
) -> AsyncGenerator[str, None]:
    """
    流式对话的核心函数
    
    返回：SSE 格式的事件流
    """
    
    # ① 保存用户消息到数据库
    await save_user_message(conversation_id, messages[-1], user_id)
    
    # ② 创建消息缓冲区（收集完整回复用于保存）
    full_response = []
    
    # ③ 流式遍历 Agent 输出
    async for msg, metadata in agent.stream_messages(
        messages=messages,
        input_context=context,
    ):
        # ④ 区分消息类型
        if isinstance(msg, AIMessageChunk):
            # LLM 的增量文本 → 立即推送给前端
            chunk = msg.content
            full_response.append(chunk)
            yield f"data: {json.dumps({'type': 'chunk', 'content': chunk})}\n\n"
            
        elif isinstance(msg, ToolMessage):
            # 工具调用结果 → 推送给前端（展示检索过程）
            yield f"data: {json.dumps({'type': 'tool_result', 'content': msg.content})}\n\n"
    
    # ⑤ 流结束后保存完整回复
    await save_assistant_message(conversation_id, "".join(full_response))
    
    # ⑥ 发送结束事件
    yield f"data: {json.dumps({'type': 'done'})}\n\n"
```

### 1.3 中断检测

```python
# 检测客户端是否断开连接
if await request.is_disconnected():
    logger.info(f"客户端断开，停止流式输出 thread={thread_id}")
    break
```

> **为什么需要中断检测？** 用户可能切换页面或关闭浏览器。如果不检测断开，Agent 会继续推理，浪费 LLM API 调用次数和服务器资源。

### 1.4 消息保存策略

```
对话开始
  │
  ├── 用户发送消息 → 立即保存到数据库（状态: sent）
  │
  ├── Agent 开始推理 → 流式输出 chunk
  │     ├── chunk_1 → 前端展示（不保存）
  │     ├── chunk_2 → 前端展示（不保存）
  │     └── chunk_n → 前端展示（不保存）
  │
  ├── 流结束 → 合并所有 chunk → 保存完整回复（状态: completed）
  │
  └── 如果中断 → 保存已生成部分（状态: interrupted）
```

**为什么不全量实时保存？** 每 N 个 chunk 写一次数据库最平衡，减少数据库压力同时保证数据不丢失。

---

## 二、Agent Run 系统

### 2.1 为什么需要 Agent Run？

简单对话用 SSE 就够了。但有些任务（如报表生成、批量知识库检索、深度分析）可能需要执行数分钟，SSE 的长连接不适合：

| 场景 | SSE 流式 | ARQ + 轮询 |
|------|---------|-----------|
| 实时对话（< 30s） | ✅ 最佳 | ❌ 轮询延迟高 |
| 长任务（> 2min） | ❌ 连接可能超时 | ✅ 后台执行 |
| 批量任务 | ❌ 阻塞对话 | ✅ 队列缓冲 |
| 需要取消 | ❌ 只能断连 | ✅ 取消信号 |

### 2.2 系统架构

```
┌─────────┐     create_run()     ┌─────────┐     enqueue     ┌─────────┐
│  Client  │ ──────────────────→ │   API   │ ──────────────→ │  Redis  │
│         │                      │ (FastAPI)│                │ (Queue) │
└─────────┘                      └─────────┘                └────┬────┘
     │                                │                          │
     │  poll /api/runs/{id}/events   │                     ┌────▼────┐
     │ ← ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─│                     │  Worker  │
     │         (轮询事件)             │                     │  (ARQ)   │
     │                                │                     └────┬────┘
     │                                │                          │
     │                                │              push events │
     │                                │  ┌──────────────────────┘
     │                                ▼  ▼
     │                          ┌─────────┐
     │                          │  Redis  │
     │                          │ (Stream)│ ← 事件流
     │                          └─────────┘
```

### 2.3 核心组件

**ARQ Worker 配置**：
```python
# server/worker_main.py
from arq.connections import RedisSettings

class WorkerSettings:
    functions = [process_agent_run]  # 注册任务处理函数
    redis_settings = RedisSettings(host="redis", port=6379)
    max_jobs = 10                    # 最大并发任务数
    job_timeout = 600                # 单个任务最长10分钟
```

**Run 生命周期**：
```
create_run()
  │
  ▼
queued ──→ Worker 取出任务
              │
              ▼
          running ──→ Agent 执行中
              │         │
              │         ├── 推送进度事件 → Redis Stream
              │         │
              │         ▼
              │      completed / failed
              │
              ├── 用户取消 → cancel 信号写入 Redis
              │         │
              │         ▼
              │      cancelled
              │
              └── Worker 超时 → failed (timeout)
```

**取消机制**：
```python
# Worker 在执行 Agent 前检查取消信号
async def process_agent_run(run_id, ...):
    cancel_key = f"run:cancel:{run_id}"
    
    # 每个步骤后检查
    async for step in agent.execute():
        if await redis.exists(cancel_key):
            await redis.delete(cancel_key)
            return {"status": "cancelled"}
        # 继续执行
```

---

## 三、Run 事件流与前端轮询

### 3.1 Redis Stream 事件格式

```json
{
  "run_id": "abc123",
  "event_type": "progress",  // progress | log | tool_call | complete | error
  "timestamp": "2026-05-25T10:30:00Z",
  "data": {
    "step": 3,
    "total_steps": 10,
    "message": "正在检索知识库...",
    "tool_name": "query_kb",
    "tool_result": "..."
  }
}
```

### 3.2 前端轮询逻辑

```javascript
// 前端轮询（不是真轮询，而是长轮询 + 指数退避）
async function pollRunEvents(runId, lastEventId) {
    const response = await fetch(`/api/runs/${runId}/events?after=${lastEventId}`);
    const events = await response.json();
    
    for (const event of events) {
        handleEvent(event);
        lastEventId = event.id;
    }
    
    if (runStatus === 'running' || runStatus === 'queued') {
        // 继续轮询，间隔随运行时间增长
        const delay = Math.min(1000 * Math.pow(1.2, stepCount), 10000);
        setTimeout(() => pollRunEvents(runId, lastEventId), delay);
    }
}
```

> **为什么不是 WebSocket？** 对于这种"偶尔更新"的场景，轮询比 WebSocket 更简单可靠。WebSocket 需要维护常连接，断线还需重连，而轮询天然容错。

---

## 四、从零实现类似系统的关键步骤

### 最小可行的流式对话系统：
1. FastAPI `StreamingResponse` + `text/event-stream`
2. Agent 的 `astream()` 逐条产出消息
3. 前端 `EventSource` 消费 SSE 事件
4. 中断检测 `request.is_disconnected()`

### 最小可行的 Agent Run 系统：
1. Redis 作为任务队列（`LPUSH` / `BRPOP`）
2. 独立 Worker 进程消费任务
3. Redis Stream 传递事件
4. 前端轮询 API 获取进度
5. Redis 键作为取消信号

---

## 下一步学习

阅读 [[07-voice-interview-system|语音面试系统]]，理解 WebSocket 双向通信、TTS/ASR 集成和实时音频处理。
