---
type: lesson
tags: [SSE, WebSocket, 实时通信, 协议基础, 基础知识]
created: 2026-05-25
updated: 2026-05-25
difficulty: beginner
prerequisites: [HTTP协议基础]
topic: 开源项目深度拆解
status: completed
series: { name: "ai-interview-伯乐深度拆解-基础篇", part: 4 }
---

# SSE 与 WebSocket 协议基础

> 伯乐使用 SSE 做对话流式传输，WebSocket 做语音实时通信。这两种技术有什么区别？各自适合什么场景？本文从零讲清。

---

## 一、HTTP 的"请求-响应"模式之痛

传统 HTTP 的工作模式：

```
客户端                    服务器
  │                         │
  │ ──── 请求 ────────────→ │
  │                         │ (处理中...)
  │ ←──── 响应 ──────────── │
  │                         │
  │ ──── 请求 ────────────→ │
  │                         │ (处理中...)
  │ ←──── 响应 ──────────── │
  │                         │
```

**问题**：服务器不能主动推送数据给客户端！

对于 AI 对话场景，这意味着：
- LLM 生成完整个回答后才能一次性返回 → 用户等很久
- 服务器发现了什么需要通知客户端 → 做不到

---

## 二、SSE（Server-Sent Events）— 服务端推送

### 2.1 SSE 是什么？

**SSE 是一种基于 HTTP 的"服务器主动推送"技术。**

```
客户端                    服务器
  │                         │
  │ ──── 建立 SSE 连接 ───→ │
  │                         │
  │ ←── data: token_1 ──── │ LLM 生成第1个token
  │ ←── data: token_2 ──── │ LLM 生成第2个token
  │ ←── data: token_3 ──── │ LLM 生成第3个token
  │ ←── data: [DONE] ───── │ 生成完毕
  │                         │
```

**关键特征**：
- 连接建立后，服务器可以**持续推送**数据
- 客户端**只能接收**，不能通过同一条连接发送
- 基于 HTTP，所有 HTTP 基础设施（代理、负载均衡）都能用

### 2.2 SSE 的数据格式

```
HTTP/1.1 200 OK
Content-Type: text/event-stream
Cache-Control: no-cache
Connection: keep-alive

data: {"type": "chunk", "content": "HashMap"}

data: {"type": "chunk", "content": " 使用"}

data: {"type": "chunk", "content": " 链表法"}

data: {"type": "done"}

```

每条消息以 `data:` 开头，以两个换行 `\n\n` 结束。

### 2.3 伯乐如何使用 SSE？

```python
# 后端：FastAPI StreamingResponse
from fastapi.responses import StreamingResponse

async def stream_chat(agent, messages):
    async def event_generator():
        async for msg, metadata in agent.stream_messages(messages):
            chunk = msg.content
            yield f"data: {json.dumps({'type': 'chunk', 'content': chunk})}\n\n"
        yield f"data: {json.dumps({'type': 'done'})}\n\n"
    
    return StreamingResponse(
        event_generator(),
        media_type="text/event-stream",
    )
```

```javascript
// 前端：EventSource 或 fetch + ReadableStream
const response = await fetch('/api/chat/stream', {
    method: 'POST',
    body: JSON.stringify({ messages, thread_id }),
});

const reader = response.body.getReader();
while (true) {
    const { done, value } = await reader.read();
    if (done) break;
    // 解析 SSE 数据
    const text = new TextDecoder().decode(value);
    // 追加到聊天界面
    appendToChat(text);
}
```

### 2.4 SSE 的优缺点

| 优点 | 缺点 |
|------|------|
| 基于 HTTP，防火墙友好 | 只能服务器→客户端（单向） |
| 浏览器原生支持自动重连 | 二进制数据需要 base64 编码 |
| Nginx 代理简单 | 连接数限制（HTTP/1.1 每个域名 6 个） |
| 实现简单 | |

---

## 三、WebSocket — 全双工实时通信

### 3.1 WebSocket 是什么？

**WebSocket 是独立的双向通信协议。**

```
客户端                    服务器
  │                         │
  │ ──── HTTP 升级请求 ───→ │  (握手阶段用的是 HTTP)
  │ ←── 101 Switching ──── │  (协议升级为 WebSocket)
  │                         │
  │ ════ 双向通信通道 ═══════ │
  │ ←── 音频数据 ────────── │  (服务器推送 TTS 音频)
  │ ──→ 音频数据 ────────── │  (客户端发送麦克风输入)
  │ ←── 转写文本 ────────── │
  │ ──→ 心跳 ping ────────→ │
  │ ←── 心跳 pong ──────── │
  │                         │
```

### 3.2 WebSocket 的握手过程

```
客户端请求（看起来像 HTTP，其实是升级请求）：
GET /voice/interview HTTP/1.1
Host: api:5050
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Key: dGhlIHNhbXBsZSBub25jZQ==

服务器响应（同意升级）：
HTTP/1.1 101 Switching Protocols
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Accept: s3pPLMBiTxaQ9kYGzzhZRbK+xOo=

# 此时协议已从 HTTP 切换为 WebSocket
# 之后的所有通信都是 WebSocket 帧格式，不再是 HTTP
```

### 3.3 伯乐如何使用 WebSocket？

语音面试的三个并发数据流全部走同一条 WebSocket：

```
WebSocket 连接 (ws://api:5050/voice/interview)

客户端 → 服务器：
  {"type": "audio", "data": <PCM音频数据>}     ← 麦克风输入
  {"type": "ping"}                              ← 心跳

服务器 → 客户端：
  {"type": "transcript", "text": "我觉得...", "is_final": false}  ← 实时转写
  {"type": "transcript", "text": "我觉得微服务...", "is_final": true} ← 最终转写
  {"type": "audio", "data": <TTS音频数据>}      ← Agent 回复的语音
  {"type": "pong"}                              ← 心跳响应
  {"type": "agent_thinking"}                    ← Agent 正在思考
```

### 3.4 伯乐为什么语音用 WebSocket 而不是 SSE？

| 需求 | SSE 能做到吗？ | WebSocket |
|------|:---:|:---:|
| 服务器推送 TTS 音频 | ✅ | ✅ |
| 客户端发送麦克风音频 | ❌ **做不到** | ✅ |
| 低延迟双向通信 | ❌ | ✅ |
| 同一连接传输多种消息 | ✅ | ✅ |

**结论**：语音面试需要客户端持续发送音频，SSE 不支持客户端→服务器，必须用 WebSocket。

---

## 四、SSE vs WebSocket — 选型决策树

```
你需要实时通信吗？
  │
  ├── 不需要 → 普通 HTTP 请求即可
  │
  └── 需要
       │
       ├── 只需要服务器→客户端推送？
       │     └── 是 → **用 SSE**
       │            (AI 对话流式输出、消息通知、日志推送)
       │
       └── 需要双向通信？
             └── 是 → **用 WebSocket**
                    (语音通话、在线协作、实时游戏)
```

**伯乐的选择**：
- 文字对话 → SSE（服务器推 AI 回复，客户端只需发一次问题）
- 语音面试 → WebSocket（双向音频流）

---

## 五、常见踩坑

| 坑 | 解决方案 |
|----|---------|
| Nginx 缓冲 SSE 输出 | `proxy_buffering off;` |
| 浏览器限制同域名连接数 | HTTP/2 或域名分片 |
| WebSocket 断线没有自动重连 | 前端实现指数退避重连 |
| 移动网络 WebSocket 频繁断开 | 30 秒心跳 + 断线重连 |
| SSE 连接不释放 | 检测 `request.is_disconnected()` |

---

## 下一步学习

阅读 [[05-docker-compose-basics|Docker Compose 基础概念]]，理解容器化部署。
