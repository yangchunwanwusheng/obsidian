---
type: lesson
tags: [生产部署, Docker Compose, 性能优化, 面试场景, 踩坑总结]
created: 2026-05-25
updated: 2026-05-25
difficulty: advanced
prerequisites: [Docker Compose, LangGraph部署, FastAPI生产化]
topic: 开源项目深度拆解
status: completed
series: { name: "ai-interview-伯乐深度拆解", part: 9 }
---

# 部署与生产落地实战 — 踩坑指南与解决方案

> 从开发环境到生产环境的鸿沟是巨大的。本节聚焦伯乐类 AI 面试系统在生产部署中会遇到的实际问题，每一条都来自真实工程经验或社区共识。

---

## 一、LangGraph Checkpointer 的生产陷阱

### 坑1：InMemorySaver 导致服务重启后状态丢失

**问题**：开发时使用 `InMemorySaver`，一切正常。部署到生产后，每次容器重启，所有对话状态丢失。

**根因**：`InMemorySaver` 将状态存在内存中，进程退出即消失。

**解决方案**：
```python
# ❌ 开发环境可以，生产不行
from langgraph.checkpoint.memory import InMemorySaver
checkpointer = InMemorySaver()

# ✅ 生产环境：PostgreSQL 持久化
from langgraph.checkpoint.postgres.aio import AsyncPostgresSaver

async def get_checkpointer():
    saver = AsyncPostgresSaver.from_conn_string(
        "postgresql://user:pass@host:5432/db"
    )
    await saver.setup()  # 必须调用！创建 checkpoints 表
    return saver
```

**伯乐的做法**：通过 `LANGGRAPH_CHECKPOINTER_BACKEND` 环境变量切换，开发用 SQLite，生产用 PostgreSQL。

### 坑2：thread_id 冲突导致会话数据串扰

**问题**：最初用 `user_id` 做 `thread_id`，结果同一用户同时开多个面试时，状态互相覆盖。

**解决方案**：
```python
# ❌ 危险：同一用户的多个会话会冲突
thread_id = str(user_id)

# ✅ 安全：复合 ID 策略
import uuid
thread_id = f"interview_{user_id}_{uuid.uuid4().hex[:8]}"
```

**伯乐的做法**：`BaseContext.thread_id` 默认用 `uuid.uuid4()`，确保每次新会话生成唯一 ID。

### 坑3：Checkpointer 连接池耗尽

**问题**：高并发时，每个 Agent 实例都创建新的数据库连接，连接数爆炸。

**解决方案**：
```python
# ❌ 每次都创建新连接
saver = AsyncPostgresSaver(f"postgresql://...")

# ✅ 使用连接池
from psycopg_pool import AsyncConnectionPool

pool = AsyncConnectionPool(
    "postgresql://user:pass@host/db",
    min_size=5,
    max_size=20,
    timeout=30,
)

saver = AsyncPostgresSaver(pool)
```

**伯乐的改进空间**：当前版本每次 `get_graph()` 时创建新的 checkpointer 实例。高并发场景下建议改为单例 + 连接池。

---

## 二、Docker Compose 生产部署陷阱

### 坑4：开发配置泄露到生产

**问题**：`docker-compose.yml` 中 `--reload` 热重载参数、源码挂载等开发配置直接用于生产。

**解决方案**：
```yaml
# 开发环境 (docker-compose.yml)
api:
  command: uvicorn server.main:app --reload  # 热重载
  volumes:
    - ./src:/app/src  # 源码挂载

# 生产环境 (docker-compose.prod.yml)
api:
  command: uvicorn server.main:app --workers 4  # 多 worker
  # 不挂载源码！镜像已在构建时打包好
```

**伯乐的做法**：`docker-compose.yml`（开发）和 `docker-compose.prod.yml`（生产）完全分离，这是正确做法。

### 坑5：PostgreSQL 健康检查不够严格

**问题**：`pg_isready` 成功不一定代表数据库完全可用（可能在 recovery 模式）。

**改进方案**：
```yaml
healthcheck:
  test: ["CMD-SHELL", 
    "pg_isready -U postgres -d ai_interview && 
     psql -U postgres -d ai_interview -c 'SELECT 1' > /dev/null 2>&1"]
  interval: 5s
  timeout: 5s
  retries: 30
```

### 坑6：没有配置资源限制

**问题**：容器可能无限占用内存/CPU，影响宿主机其他服务。

**改进方案**：
```yaml
api:
  deploy:
    resources:
      limits:
        cpus: '2'
        memory: 4G
      reservations:
        cpus: '1'
        memory: 2G
```

### 坑7：日志无限增长

**问题**：Docker 容器日志默认不限制大小，长期运行后可能占满磁盘。

**改进方案**：
```yaml
# docker-compose.yml 或 /etc/docker/daemon.json
services:
  api:
    logging:
      driver: "json-file"
      options:
        max-size: "100m"
        max-file: "5"
```

---

## 三、SSE 流式对话的生产问题

### 坑8：Nginx 缓冲导致流式响应失效

**问题**：生产环境用 Nginx 反向代理时，SSE 流式输出被缓冲，用户看到的是延迟后的批量输出。

**解决方案**：
```nginx
location /api/chat/ {
    proxy_pass http://api:5050;
    
    # 关键配置
    proxy_buffering off;           # 关闭代理缓冲
    proxy_cache off;               # 关闭缓存
    proxy_set_header X-Accel-Buffering no;  # 禁用 Nginx 加速缓冲
    
    # 长连接超时
    proxy_read_timeout 300s;       # 面试可能持续很久
    proxy_send_timeout 300s;
}
```

### 坑9：SSE 连接泄漏

**问题**：用户中途关闭浏览器，后端 SSE 连接未释放，资源泄漏。

**解决方案**：
```python
async def stream_chat(request: Request):
    async def event_generator():
        try:
            async for msg, meta in agent.stream_messages(...):
                if await request.is_disconnected():  # 客户端断开检测
                    break
                yield f"data: {json.dumps(msg)}\n\n"
        except asyncio.CancelledError:
            pass
        finally:
            # 确保释放资源
            await cleanup_session(thread_id)

    return StreamingResponse(event_generator(), media_type="text/event-stream")
```

---

## 四、LLM API 调用的生产问题

### 坑10：模型 API 限流导致面试中断

**问题**：SiliconFlow 等免费 API 有严格的 RPM（每分钟请求数）限制。高并发时触发 429 错误，面试中断。

**解决方案**：
```python
# 1. 使用 tenacity 实现指数退避重试
from tenacity import retry, stop_after_attempt, wait_exponential

@retry(
    stop=stop_after_attempt(3),
    wait=wait_exponential(multiplier=2, min=4, max=30),
    retry=retry_if_exception_type(RateLimitError),
)
async def call_llm_with_retry(messages):
    return await llm.ainvoke(messages)

# 2. 实现请求队列
# 使用 Redis + 令牌桶算法控制请求速率

# 3. 多 Provider 负载均衡
# 配置多个 API provider，主 Provider 限流时自动切换
```

**伯乐的做法**：`ModelRetryMiddleware` 提供了基础重试，但建议增强为多 Provider 切换。

### 坑11：长对话超出上下文窗口

**问题**：面试可能持续 30+ 轮对话，LLM 上下文窗口不够用。

**解决方案**：
```python
# 1. 滑动窗口 — 保留最近 N 条消息
# 2. 摘要中间件 — 早期消息压缩为摘要
# 伯乐的做法（推荐）：
OpenVikingSummaryMiddleware(
    model=model,
    trigger=("tokens", 30000),        # 超过30000 token 触发
    trim_tokens_to_summarize=2000,     # 保留最近2000 token
    max_retention_ratio=0.5,           # 最多保留50%
)
```

### 坑12：模型响应超时

**问题**：DeepSeek 等大模型在高峰时段响应可能超过 60 秒。

**解决方案**：
```python
# 1. API 层设置合理超时
from openai import AsyncOpenAI
client = AsyncOpenAI(
    base_url="...",
    timeout=120.0,  # 2分钟超时
    max_retries=2,
)

# 2. 前端显示"思考中"状态
# 使用心跳机制：每5秒给前端发一个空事件，防止浏览器超时

# 3. 降级策略
# 超时 → 自动切换到更小的模型
```

---

## 五、WebSocket 语音面试的生产问题

### 坑13：双向流式 WebSocket 的时序问题

**问题**：语音面试涉及三个并发流：麦克风输入（ASR）、LLM 推理（Agent）、TTS 输出（音频播放）。三者时序混乱会导致体验极差。

**解决方案**：
```python
# 三阶段流水线
ASR 转录 → [VAD断句] → LLM思考 → TTS合成 → 播放
   ↑                                              │
   └──────────── 用户听到回复后继续说话 ──────────────┘

# 关键点：
# 1. VAD（Voice Activity Detection）在用户说话时不处理 Agent 回复
# 2. 使用 asyncio.Queue 作为缓冲区，避免竞态条件
# 3. WebSocket 心跳保活（每30秒 ping/pong）
```

### 坑14：浏览器音频上下文自动播放限制

**问题**：Chrome/Safari 等浏览器要求用户交互后才能播放音频，WebSocket 收到音频数据但无法播放。

**解决方案**：
```javascript
// 前端：点击"开启语音面试"按钮时恢复 AudioContext
const audioContext = new AudioContext();

async function startVoiceInterview() {
    // 必须先由用户点击触发
    await audioContext.resume();
    // 然后才建立 WebSocket 连接
    connectWebSocket();
}
```

**伯乐的做法**：已在文档中明确说明"需要在页面中点击'开启语音面试'后才会开始播放"。

### 坑15：VAD 断句阈值调优

**问题**：默认 3 秒无声才触发断句。太短→频繁打断，太长→用户等太久。

**调优建议**：
```
面试场景推荐值：
- 技术回答（长句子）：2-3秒
- 行为面试（短句子）：1.5-2秒
- 非母语者：3-4秒（留更多思考时间）

伯乐的默认 3 秒适合大多数中文面试场景
```

---

## 六、知识库/RAG 的生产问题

### 坑16：大量文件导入导致 API 超时

**问题**：一次性导入知识库（如 JavaGuide 完整仓库），文件解析+Embedding 可能耗时数十分钟，API 网关超时。

**解决方案**：
```python
# 1. 异步分批处理
async def import_knowledge_files(files, batch_size=10):
    for batch in chunked(files, batch_size):
        tasks = [parse_and_index(f) for f in batch]
        await asyncio.gather(*tasks)

# 2. 状态机追踪
# UPLOADED → PARSING → PARSED → INDEXING → INDEXED
# 前端轮询进度

# 3. 使用 ARQ 后台任务
# 导入操作放入 Redis 队列，后台 Worker 处理
```

**伯乐的做法**：`kb-import` 容器作为一次性任务，在启动时完成导入，不走 API 超时限制。

### 坑17：Embedding 维度不匹配

**问题**：BGE-M3 默认输出 1024 维，但某些数据库/索引预设了其他维度。

**解决方案**：
```python
# 统一在配置中指定维度
OPENVIKING_EMBEDDING_DIMENSION=1024

# 重建索引时检查维度
if existing_dimension != expected_dimension:
    logger.warning("维度不匹配，需要重建索引")
    await rebuild_index(kb_id)
```

### 坑18：文档解析质量不稳定

**问题**：同样一份 PDF，MinerU 和 PaddleX 的解析结果差异很大。

**解决方案**：
```python
# 1. 多解析器容错
async def parse_with_fallback(file_path):
    for parser in [mineru_parser, paddlex_parser, docling_parser]:
        try:
            result = await parser.parse(file_path)
            if quality_check(result):  # 质量检查
                return result
        except Exception:
            continue
    raise ParseError("所有解析器均失败")

# 2. 解析结果质量评分
# 检查：文本长度、表格保留率、图片 OCR 正确率
```

---

## 七、数据库与性能问题

### 坑19：PostgreSQL 慢查询

**问题**：对话历史表（messages）随时间增长，查询越来越慢。

**优化**：
```sql
-- 1. 索引优化
CREATE INDEX idx_messages_thread_id_created 
ON messages(thread_id, created_at);

CREATE INDEX idx_messages_conversation_id 
ON messages(conversation_id);

-- 2. 分区表（按月分区）
CREATE TABLE messages (
    id BIGSERIAL,
    thread_id VARCHAR(255),
    created_at TIMESTAMPTZ DEFAULT NOW(),
    ...
) PARTITION BY RANGE (created_at);

-- 3. 定期归档
-- 超过 90 天的对话归档到冷存储
```

### 坑20：Redis 内存压力

**问题**：Redis Stream 存储 Run 事件，大量长时间运行的 Agent 导致内存增长。

**优化**：
```python
# 1. TTL 机制（伯乐已实现）
RUN_EVENTS_STREAM_TTL_SECONDS=7200   # 2小时自动过期
RUN_CANCEL_KEY_TTL_SECONDS=1800      # 取消信号30分钟过期

# 2. Stream 长度限制
MAX_STREAM_LENGTH = 10000  # 每个 stream 最多保留10000条

# 3. 定期清理过期 key
# Redis 的 maxmemory-policy 配置为 allkeys-lru
```

---

## 八、安全与权限

### 坑21：超级管理员密码泄露

**问题**：`.env` 文件中的 `AI_INTERVIEW_SUPER_ADMIN_PASSWORD` 可能被提交到 Git。

**防御**：
```bash
# 1. .gitignore 中已包含 .env
# 2. 提供 .env.template 不含密钥
# 3. 使用 pre-commit hook 扫描敏感信息
# 伯乐已有 ruff BLE 规则（待启用）和 security-guard hook
```

### 坑22：Agent 可以通过文件系统访问敏感路径

**风险**：Agent 的 FilesystemMiddleware 如果配置不当，可能读取宿主机的敏感文件。

**缓解**：
```python
# 1. Docker 容器限制文件系统
# 只挂载必要的目录

# 2. 中间件白名单（伯乐的做法）
# FilesystemMiddleware 自定义 tool_description
# "只允许读取系统已明确给出的附件 file_path"

# 3. 容器运行用户
# Dockerfile 中 USER nonroot
```

---

## 九、前端生产问题

### 坑23：CORS 配置过宽

**问题**：开发时 `allow_origins=["*"]` 可能被带到生产。

**伯乐的做法**：
```python
# 从环境变量读取 CORS 白名单
AI_INTERVIEW_CORS_ORIGINS=http://localhost:5173,https://bole.example.com
```

### 坑24：静态资源缓存策略

**问题**：前端更新后，用户浏览器缓存旧版本 JS/CSS。

**解决方案**：
```nginx
# Vite 构建的文件带 hash，可以长期缓存
location /assets/ {
    expires 1y;
    add_header Cache-Control "public, immutable";
}

# index.html 不缓存
location / {
    expires -1;
    add_header Cache-Control "no-cache";
}
```

---

## 十、监控与告警建议

### 必备监控指标

| 指标 | 工具 | 告警阈值 |
|------|------|---------|
| API 响应时间 P95 | Prometheus + Grafana | > 30s |
| LLM 调用失败率 | LangSmith / 自定义日志 | > 5% |
| Checkpointer 写入延迟 | PostgreSQL slow query log | > 100ms |
| WebSocket 断连率 | 自定义指标 | > 10% |
| Redis 内存使用率 | Redis INFO | > 80% |
| 容器资源使用 | Docker stats / cAdvisor | CPU > 80%, Mem > 90% |

### 日志策略

```python
# 结构化日志（伯乐使用 colorlog）
logger.info("agent_run_started", extra={
    "thread_id": thread_id,
    "user_id": user_id,
    "agent_name": "InterviewAgent",
    "timestamp": datetime.now().isoformat(),
})

# 生产环境：
# - 日志级别: WARNING+
# - 输出到 stdout（Docker 收集）
# - 使用 ELK/Loki 聚合
```

---

## 总结：生产就绪检查清单

- [ ] Checkpointer 使用 PostgreSQL（非 InMemorySaver）
- [ ] thread_id 使用 UUID + 前缀策略
- [ ] docker-compose.yml 和 docker-compose.prod.yml 分离
- [ ] Nginx 关闭 SSE 缓冲（proxy_buffering off）
- [ ] LLM 调用配置重试和降级策略
- [ ] 长对话配置 SummaryMiddleware
- [ ] WebSocket 配置心跳和重连
- [ ] 容器配置资源限制和日志轮转
- [ ] CORS 配置生产白名单
- [ ] 启用监控和告警
- [ ] .env 不提交到 Git
- [ ] PostgreSQL 配置慢查询日志
- [ ] Redis 配置内存限制和淘汰策略
