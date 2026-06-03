---
type: lesson
tags: [LangGraph, Agent架构, 中间件模式, Checkpointer, BaseContext]
created: 2026-05-25
updated: 2026-05-25
difficulty: advanced
prerequisites: [LangChain基础, LangGraph概念, Python dataclass, 异步编程]
topic: 开源项目深度拆解
status: completed
series: { name: "ai-interview-伯乐深度拆解", part: 2 }
---

# LangGraph Agent 系统深度拆解

> 这是整个项目的核心。理解了这个系统，就理解了伯乐的 Agent 如何被创建、配置、执行和管理。

---

## 一、整体设计 — 为什么不是手写 Agent 循环？

### 传统 Agent 循环 vs LangGraph Agent

**传统方式**（手写 while 循环）：
```python
while not done:
    response = llm.invoke(messages)
    if response.tool_calls:
        results = execute_tools(response.tool_calls)
        messages.extend(results)
    else:
        done = True
```

**LangGraph 方式**：
```python
graph = create_agent(model=model, middleware=[...], checkpointer=...)
async for msg, meta in graph.astream(messages, context=context):
    yield msg, meta
```

| 维度 | 手写循环 | LangGraph |
|------|---------|-----------|
| 工具调用处理 | 手动管理 | 自动循环 |
| 状态持久化 | 自己实现 | Checkpointer 一行配置 |
| 中断恢复 | 非常复杂 | 原生支持 |
| 能力组合 | if/else 堆叠 | 中间件栈 |
| 可观测性 | 手动日志 | LangSmith 集成 |

---

## 二、BaseAgent — Agent 的生命周期管理器

### 2.1 源码拆解

`BaseAgent` 是所有 Agent 的基类（`src/agents/common/base.py`，约 300 行）。核心职责：

```
BaseAgent
├── 图管理       → get_graph() / reload_graph()
├── 消息流       → stream_messages() / stream_values() / invoke_messages()
├── Checkpointer → _get_checkpointer() / _create_postgres_checkpointer()
├── 元数据       → load_metadata() / get_info()
├── 历史查询     → get_history()
└── 上下文       → context_schema / module_name
```

### 2.2 核心方法：stream_messages()

这是 Agent 对外暴露的**主入口**：

```python
async def stream_messages(self, messages, input_context=None, **kwargs):
    graph = await self.get_graph()           # ① 获取编译好的图
    context = self.context_schema()           # ② 创建上下文实例
    context.update(input_context or {})       # ③ 合并运行时配置

    input_config = {
        "configurable": {
            "thread_id": context.thread_id,   # ④ 关键：thread_id 决定会话隔离
            "user_id": context.user_id
        },
        "recursion_limit": 300,               # ⑤ 防止无限循环
    }

    # ⑥ 设置请求级上下文（供中间件异步访问）
    token = set_agent_request_context(...)
    try:
        async for msg, metadata in graph.astream(
            {"messages": messages},
            stream_mode="messages",           # ⑦ 流模式：逐个消息产出
            context=context,
            config=input_config,
        ):
            yield msg, metadata               # ⑧ 逐条产出，上层转 SSE
    finally:
        reset_agent_request_context(token)    # ⑨ 清理请求上下文
```

**关键设计点**：

1. **thread_id 的作用**：LangGraph 通过 `thread_id` 隔离不同会话的对话历史。同一个 `thread_id` 的后续调用会自动从 Checkpointer 恢复之前的消息状态。

2. **recursion_limit: 300**：LangGraph 的递归限制。Agent 在工具调用循环中每轮都会增加递归计数，超过限制则强制终止。300 对于面试场景足够（通常 10-20 轮工具调用）。

3. **request_context 的 ContextVar 机制**：使用 Python 的 `contextvars` 实现请求级上下文传递，允许中间件在异步调用链的任何位置访问当前请求的 `thread_id` 和 `user_id`，而不需要通过参数层层传递。

### 2.3 Checkpointer — 会话持久化的秘密

```python
async def _get_checkpointer(self):
    backend = os.getenv("LANGGRAPH_CHECKPOINTER_BACKEND", "sqlite")

    if backend == "postgres":
        checkpointer = await self._create_postgres_checkpointer()

    if checkpointer is None:
        checkpointer = AsyncSqliteSaver(await self.get_async_conn())

    return checkpointer
```

**双后端设计**：

| 场景 | 后端 | 特点 |
|------|------|------|
| 本地开发 | SQLite | 零配置，文件存储，单用户够用 |
| 生产环境 | PostgreSQL | 高并发、持久化、可备份 |

**SQLite 的兼容性补丁**：
```python
# 关键补丁：LangGraph 需要 is_alive() 方法，但 aiosqlite 不提供
if not hasattr(conn, "is_alive"):
    conn.is_alive = lambda: True
```

**PostgreSQL Checkpointer 的初始化**：
```python
async def _create_postgres_checkpointer(self):
    conn_str = postgres_url.replace("+asyncpg", "")  # asyncpg→psycopg

    # 方式1：工厂方法（新版 LangGraph）
    saver = AsyncPostgresSaver.from_conn_string(conn_str)

    # 方式2：直接构造（旧版兼容）
    saver = AsyncPostgresSaver(conn_str)

    # 必须调用 setup() 创建 checkpointer 所需的表
    await saver.setup()

    return saver
```

> **Checkpointer 做了什么**：每次 Agent 图执行完一步（无论是 LLM 调用还是工具执行），Checkpointer 都会自动将当前的 State（包含所有消息）序列化存储。下次相同的 `thread_id` 进入时，自动恢复状态。

---

## 三、BaseContext — 可配置上下文体系

### 3.1 设计精要

```python
@dataclass(kw_only=True)
class BaseContext:
    thread_id: str        # 会话ID（不可配置）
    user_id: str          # 用户ID（不可配置）
    system_prompt: str    # 系统提示词（可配置）
    model: str            # LLM模型（可配置，支持 Annotated 元数据标记）
    tools: list[dict]     # 工具列表（可配置）
    knowledges: list[str] # 知识库列表（可配置）
    mcps: list[str]       # MCP服务器列表（可配置）
    skills: list[str]     # 技能列表（可配置）
```

**配置优先级**（从高到低）：
```
API 传入的参数  >  config.yaml 文件  >  dataclass 默认值
    运行时              持久化              类定义
```

### 3.2 Annotated 元数据 — 前后端类型契约

```python
# 普通字段：前端只能看到类型和默认值
model: str = field(default="default_model")

# Annotated 字段：前端可以获取额外元数据用于渲染
system_prompt: Annotated[str, {"__template_metadata__": {"kind": "prompt"}}] = field(...)
model: Annotated[str, {"__template_metadata__": {"kind": "llm"}}] = field(...)
tools: Annotated[list[dict], {"__template_metadata__": {"kind": "tools"}}] = field(...)
```

`get_configurable_items()` 方法会提取这些元数据，前端根据 `kind` 决定渲染方式：
- `"prompt"` → 多行文本编辑器
- `"llm"` → 模型选择下拉框
- `"tools"` → 工具多选列表
- `"knowledges"` → 知识库多选

### 3.3 配置持久化

```python
# 保存：只序列化可配置字段
BaseContext.save_to_file(config_dict, module_name)
# → saves/agents/{module_name}/config.yaml

# 加载：三层优先级合并
context = BaseContext.from_file(module_name, input_context)
# → 类默认 → config.yaml → 运行时覆盖
```

> **为什么不用数据库存配置？** YAML 文件更适合开发阶段。Agent 配置文件可以直接用编辑器打开修改，也方便版本控制。对于频繁修改的场景，后续可以迁移到数据库 + 缓存。

---

## 四、中间件体系 — 可插拔的能力组合

### 4.1 为什么需要中间件？

LangChain 的 `create_agent` 内置了 Agent 循环，但**不内置具体的能力**（如知识库检索、对话摘要、文件访问）。中间件提供了在 Agent 循环中注入能力的方式。

### 4.2 中间件的工作原理

LangChain 的中间件实现了 `AgentMiddleware` 接口：

```python
class AgentMiddleware:
    # 在 Agent 循环开始前
    async def abefore_agent(self, state, runtime):
        # 修改 state（注入提示词/上下文/工具）
        return {"messages": state["messages"] + [context_message]}

    # 在每次 LLM 调用前
    async def abefore_model(self, state, runtime):
        # 修改传给 LLM 的消息（添加/移除/修改）
        return {"messages": modified_messages}

    # 在 LLM 调用后
    async def aafter_model(self, state, runtime):
        # 后处理（如摘要触发判断）
        return state
```

**执行的实际上是一个洋葱模型**：

```
请求进入
  → middleware1.abefore_agent
    → middleware2.abefore_model
      → middleware3.abefore_model
        → [LLM 调用]
      → middleware3.aafter_model
    → middleware2.aafter_model
  → middleware1.aafter_model
→ 输出
```

### 4.3 面试 Agent 的中间件栈（逐层分析）

```python
middleware=[
    # 第1层：附件保存
    save_attachments_to_fs,
    # → 拦截用户上传的文件，保存到 MinIO，返回可读路径

    # 第2层：文件系统访问（只读）
    _create_interview_filesystem_middleware(),
    # → 提供 read_file 工具，但限制只能读已上传的附件
    # → 通过自定义 tool description 引导 Agent 行为

    # 第3层：面试知识库工具注入
    InterviewKnowledgeBaseMiddleware(),
    # → 注入 query_kb（RAG检索）、pick_random_technical_question、
    #   start_code_assessment 三个面试专用工具

    # 第4层：运行时配置覆盖
    RuntimeConfigMiddleware(),
    # → 允许运行时修改 model、system_prompt 等配置
    # → 不重启服务即可切换模型

    # 第5层：RAG 上下文注入
    OpenVikingContextMiddleware(agent_id=self.id),
    # → 从 OpenViking 向量库中检索相关文档片段
    # → 注入到 system prompt 作为上下文

    # 第6层：视频分析上下文
    VideoContextMiddleware(),
    # → 注入实时视频分析结果（情绪、注意力等）
    # → 仅在视频面试模式下生效

    # 第7层：任务清单追踪
    TodoListMiddleware(system_prompt=INTERVIEW_TODO_PROMPT),
    # → 注入 write_todos 工具
    # → Agent 必须在每轮更新 6 个固定任务的完成状态

    # 第8层：工具调用修补
    PatchToolCallsMiddleware(),
    # → 来自 deepagents 库
    # → 修复某些 LLM 的工具调用格式问题（如参数名错误）

    # 第9层：长对话摘要
    OpenVikingSummaryMiddleware(
        model=model,
        trigger=("tokens", 30000),        # 超过30000 token 触发
        trim_tokens_to_summarize=2000,     # 保留最近2000 token
        max_retention_ratio=0.5,           # 最多保留50%的原始消息
    ),
    # → 当对话超过阈值，自动将早期消息摘要化
    # → 解决 LLM 上下文窗口限制

    # 第10层：工具调用限频
    ToolCallLimitMiddleware(
        tool_name="query_kb",
        run_limit=1,                       # query_kb 最多调用1次
        exit_behavior="continue",          # 超过限制后不报错，继续
    ),
    # → 防止 Agent 反复检索知识库消耗 token

    # 第11层：模型重试
    ModelRetryMiddleware(),
    # → LLM 调用失败时自动重试
    # → 处理 API 限流、网络波动等瞬时错误
]
```

> **中间件顺序真的很重要！** 例如，SummaryMiddleware 必须放在最后（靠近底层），这样它看到的才是经过所有中间件处理后的完整消息。如果在它后面还有中间件修改消息，摘要可能会不准确。

### 4.4 FilesystemMiddleware 的安全设计

```python
INTERVIEW_FILESYSTEM_PROMPT = """
你只能使用 read_file 工具读取当前会话中已经提供的附件内容。
- 只允许读取系统已明确给出的附件 file_path。
- 不要为了找简历而遍历目录、搜索其他文件或读取无关路径。
"""

# 只保留 read_file 工具，移除 write_file、edit_file 等
middleware.tools = [
    tool for tool in middleware.tools
    if getattr(tool, "name", "") == "read_file"
]
```

> **安全考虑**：Agent 不应该有写文件的能力（防止误操作），只能读已上传的附件。这是通过 "prompt 约束 + 工具白名单" 双重保证的。

---

## 五、请求上下文传递 — ContextVar 机制

```python
# src/agents/common/runtime_request_context.py

_agent_request_ctx: ContextVar[dict] = ContextVar("agent_request_ctx", default={})

def set_agent_request_context(thread_id, user_id, target_position=""):
    ctx = {"thread_id": thread_id, "user_id": user_id, ...}
    token = _agent_request_ctx.set(ctx)
    return token

def get_current_thread_id():
    return _agent_request_ctx.get().get("thread_id", "")

def get_current_user_id():
    return _agent_request_ctx.get().get("user_id", "")
```

**为什么用 ContextVar 而不是传参？**

1. 中间件的接口是固定的（LangChain 定义），无法额外传参
2. 但中间件内部需要知道当前是哪个用户/会话
3. ContextVar 在异步调用链中自动传播，各中间件 `get_current_xxx()` 即可获取

---

## 六、从零手搓一个类似 Agent 系统的关键步骤

### 需要实现的核心组件：

1. **BaseAgent 基类**
   - `get_graph()` — 编译 LangGraph 图
   - `stream_messages()` — SSE 流式输出
   - `_get_checkpointer()` — 持久化配置

2. **BaseContext 数据类**
   - `from_file()` / `save_to_file()` — YAML 配置读写
   - `get_configurable_items()` — 前端配置界面元数据
   - `update()` — 运行时参数覆盖

3. **中间件体系**
   - 至少需要：知识库注入、对话摘要、工具限频
   - 实现 `AgentMiddleware` 接口

4. **请求上下文**
   - ContextVar 实现
   - `set` / `reset` 配对调用

---

## 下一步学习

阅读 [[03-interview-agent-deep-dive|面试 Agent 深度剖析]]，理解 6 阶段面试流程、TodoList 驱动机制和 System Prompt 工程。
