---
type: lesson
tags: [AI面试, LangGraph, RAG, FastAPI, Vue3, 开源拆解]
created: 2026-05-25
updated: 2026-05-25
difficulty: advanced
prerequisites: [Python, FastAPI, LangChain基础, Vue3基础, Docker基础]
topic: 开源项目深度拆解
status: in-progress
series: { name: "ai-interview-伯乐深度拆解", part: 0 }
---

# 伯乐 Bole — AI 模拟面试与智能知识库平台 深度拆解（精华篇）

> **一句话总结**：伯乐是一个基于 LangGraph v1 + FastAPI + Vue3 的全栈 AI 面试与知识库平台，核心创新在于将 LangGraph 的中间件体系应用于真实面试场景，实现了从简历解析→知识库 RAG→结构化面试→代码考核→多维度评分的完整闭环。

---

## 项目定位与核心价值

### 这个项目解决什么问题？

传统的 AI 面试工具（如各家招聘平台的 AI 初筛）往往是**单一轮次的固定题库问答**，缺乏：
- 针对候选人**真实简历**的深度追问
- 多轮次、多阶段的**结构化面试流程**
- 结合岗位 JD 的**智能匹配度评估**
- 真实的**代码考核与在线判题**能力

伯乐将这三个需求整合到一个统一的 **LangGraph Agent 编排框架**中，通过中间件体系实现了可插拔的能力组合。

### 核心价值（为什么值得深入拆解）

| 维度 | 价值 |
|------|------|
| **Agent 架构** | 展示了 LangGraph v1 + 中间件模式在生产环境的最佳实践 |
| **RAG 落地** | 完整的多格式文档解析→分块→Embedding→Rerank→检索链路 |
| **前端工程** | Vue3 + Pinia + Ant Design Vue 的全栈协作模式 |
| **DevOps** | Docker Compose 全容器化管理，开发/生产环境分离 |
| **面试场景** | 6 阶段任务清单驱动的面试流程，含代码考核与视频分析 |
| **MCP 集成** | 内置 MCP 客户端，支持动态挂载外部工具服务器 |

---

## 技术栈全景图

```
┌─────────────────────────────────────────────────────────────────┐
│                      前端层 (Vue 3 + Vite)                       │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌────────────────────┐ │
│  │ Vue Router│ │  Pinia   │ │Ant Design│ │  Less + CSS变量    │ │
│  │  路由守卫  │ │ 状态管理  │ │  Vue 4.x │ │  暗色模式支持      │ │
│  └──────────┘ └──────────┘ └──────────┘ └────────────────────┘ │
├─────────────────────────────────────────────────────────────────┤
│                     API 网关层 (FastAPI)                         │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌────────────────────┐ │
│  │CORS中间件 │ │限流中间件  │ │鉴权中间件 │ │ 异常处理(统一形状)  │ │
│  └──────────┘ └──────────┘ └──────────┘ └────────────────────┘ │
├─────────────────────────────────────────────────────────────────┤
│                     Agent 编排层 (LangGraph v1)                  │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │                   BaseAgent                              │   │
│  │  ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌──────────────┐  │   │
│  │  │Checkpointer│ │Context │ │Middleware│ │ Toolkits     │  │   │
│  │  │PG/SQLite │ │ 运行时配置│ │  中间件栈 │ │ 工具集       │  │   │
│  │  └─────────┘ └─────────┘ └─────────┘ └──────────────┘  │   │
│  └──────────────────────────────────────────────────────────┘   │
│  ┌──────────────┐ ┌──────────────┐ ┌────────────────────────┐   │
│  │InterviewAgent│ │  Chatbot     │ │   Reporter / DeepAgent │   │
│  │6阶段面试流程 │ │  通用对话     │ │   报表 / 深度分析      │   │
│  └──────────────┘ └──────────────┘ └────────────────────────┘   │
├─────────────────────────────────────────────────────────────────┤
│                     RAG 知识库层 (OpenViking)                    │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌────────────────────┐ │
│  │文档解析   │ │分块策略   │ │Embedding │ │  Rerank + 检索     │ │
│  │MinerU/   │ │General/  │ │BGE-M3    │ │  混合检索策略      │ │
│  │PaddleX/  │ │QA/Books/ │ │1024维    │ │                    │ │
│  │DeepSeek  │ │Laws      │ │          │ │                    │ │
│  └──────────┘ └──────────┘ └──────────┘ └────────────────────┘ │
├─────────────────────────────────────────────────────────────────┤
│                     基础设施层                                   │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌────────────────────┐ │
│  │PostgreSQL│ │ Redis 7  │ │ MinIO    │ │  ARQ 任务队列       │ │
│  │ 16       │ │          │ │对象存储   │ │  后台 Worker       │ │
│  └──────────┘ └──────────┘ └──────────┘ └────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

---

## 核心架构决策与设计亮点

### 1. LangGraph v1 的 create_agent 中间件体系

这是整个项目最核心的设计决策。不同于传统的"自己写 while 循环调 LLM"，伯乐使用了 LangChain 提供的 `create_agent` + 中间件栈模式：

```python
# 核心模式
create_agent(
    model=model,
    system_prompt=context.system_prompt,
    middleware=[
        save_attachments_to_fs,          # 1. 附件保存
        FilesystemMiddleware(...),        # 2. 文件系统访问
        InterviewKnowledgeBaseMiddleware(), # 3. 知识库工具注入
        RuntimeConfigMiddleware(),        # 4. 运行时配置覆盖
        OpenVikingContextMiddleware(),    # 5. RAG上下文注入
        VideoContextMiddleware(),         # 6. 视频分析上下文
        TodoListMiddleware(...),          # 7. 任务清单追踪
        PatchToolCallsMiddleware(),       # 8. 工具调用修补
        OpenVikingSummaryMiddleware(),    # 9. 长对话摘要
        ToolCallLimitMiddleware(),        # 10. 工具调用限频
        ModelRetryMiddleware(),           # 11. 模型重试
    ],
    checkpointer=await self._get_checkpointer(),
)
```

**设计优势**：
- 每个中间件职责单一，可独立测试和替换
- 按列表顺序执行，顺序影响行为（如 Summary 在最后，保证截断前数据完整）
- 新增能力只需添加/移除中间件，不需要修改核心逻辑

### 2. Checkpointer 的双后端策略

```python
# 开发环境: SQLite (零配置)
checkpointer = AsyncSqliteSaver(await self.get_async_conn())

# 生产环境: PostgreSQL (高并发、持久化)
checkpointer = AsyncPostgresSaver.from_conn_string(conn_str)
```

设计要点：
- 通过环境变量 `LANGGRAPH_CHECKPOINTER_BACKEND` 切换
- SQLite 需要 patch `is_alive()` 方法兼容 LangGraph 的接口检测
- PostgreSQL 模式下自动调用 `setup()` 创建必要的表

### 3. 配置优先级体系

```
运行时配置 (API 传入)  >  文件配置 (config.yaml)  >  类默认值
        最高                      中等                  最低
```

实现方式：`BaseContext.update()` 方法逐层覆盖，前端可通过 API 动态修改 Agent 配置。

### 4. 面试流程的 6 阶段 TodoList 驱动

面试 Agent 通过 `TodoListMiddleware` 注入固定的 6 个 todo 任务，LangGraph 的 Agent 在每轮对话中自主管理任务状态：

```
1. 发起开场并请候选人自我介绍  → pending → in_progress → completed
2. 追问项目经历与技术细节      → pending → in_progress → completed
3. 相关技术知识提问            → pending → in_progress → completed
4. 代码考核                   → pending → in_progress → completed
5. 评估岗位匹配度与风险点       → pending → in_progress → completed
6. 输出总结与评分卡            → pending → in_progress → completed
```

这种设计使得 Agent 行为高度**可控且可预测**，系统提示词 + TodoList 双重约束保证了面试流程的一致性。

---

## 数据流全景

```
用户上传简历 PDF
    │
    ▼
文档解析（MinerU / PaddleX / DeepSeek OCR）
    │
    ▼
分块（General / QA / Books / Laws 策略）
    │
    ▼
Embedding（BGE-M3, 1024维）
    │
    ▼
存入向量索引（OpenViking）
    │
    ▼
用户发起面试 → InterviewAgent 读取简历 → 注入 System Prompt
    │
    ▼
SSE 流式对话 ←→ LangGraph Agent 循环
    │                    │
    │    ┌───────────────┼───────────────┐
    │    ▼               ▼               ▼
    │ query_kb()   pick_question()  start_code()
    │ (RAG检索)     (随机抽题)      (在线判题)
    │    │               │               │
    │    ▼               ▼               ▼
    │ OpenViking    面试题库        OJ判题系统
    │    │
    ▼    ▼
最终输出：评分卡 + 面试总结 + 改进建议
```

---

## 关键文件索引

| 文件 | 作用 | 行数 |
|------|------|------|
| `src/agents/common/base.py` | Agent 基类（图管理、checkpointer、消息流） | ~300 |
| `src/agents/common/context.py` | 可配置上下文（模型、工具、知识库、技能） | ~200 |
| `src/agents/interview_agent/graph.py` | 面试 Agent 图定义（中间件组合） | ~140 |
| `src/agents/interview_agent/context.py` | 面试上下文（含 6 阶段 system prompt） | ~180 |
| `src/knowledge/base.py` | 知识库抽象基类（文件生命周期管理） | ~1200 |
| `src/services/chat_stream_service.py` | 核心流式对话（SSE、消息保存） | ~900 |
| `src/services/interview_coding_service.py` | 代码考核（在线判题集成） | ~2500 |
| `src/services/interview_result_service.py` | 面试结果（评分卡、报告） | ~3500 |
| `src/services/voice_interview_service.py` | 语音面试（WebSocket + TTS + ASR） | ~1500 |
| `src/plugins/mineru_official_parser.py` | MinerU 文档解析 | ~400 |
| `src/models/embed.py` | Embedding 模型封装 | ~300 |
| `src/models/rerank.py` | Rerank 模型封装 | ~180 |
| `web/src/apis/` | 前端 API 层 | ~15 文件 |

---

## 学习路线图

### 基础概念补课（先读这 6 篇，零基础友好）🆕

1. **[[foundations/01-what-is-rag|RAG 是什么]]** — 检索增强生成的完整原理，从零到一手搓最小 RAG
2. **[[foundations/02-langgraph-agent-basics|LangGraph 与 Agent 基础]]** — LangChain/LangGraph 区别、Agent 循环、ReAct、中间件
3. **[[foundations/03-embedding-and-rerank|Embedding 与 Rerank 基础]]** — 文字如何变向量、相似度计算、BGE-M3 选型
4. **[[foundations/04-sse-websocket-basics|SSE 与 WebSocket 协议]]** — 实时通信两种方式、伯乐的选型逻辑
5. **[[foundations/05-docker-compose-basics|Docker Compose 基础]]** — 容器化部署概念、compose 文件解析
6. **[[foundations/06-fastapi-vue-basics|FastAPI 与 Vue3 基础]]** — 前后端框架核心概念

### 代码深度拆解（在理解基础概念后阅读）

按以下顺序阅读本系列笔记，可从零到一掌握该项目的全部技术栈：

1. **[[01-architecture-overview|整体架构深度剖析]]** — 理解分层设计、数据流和服务拓扑
2. **[[02-langgraph-agent-system|LangGraph Agent 系统深度拆解]]** — BaseAgent/BaseContext/中间件/Checkpointer
3. **[[03-interview-agent-deep-dive|面试 Agent 深度剖析]]** — 6 阶段流程、TodoList 驱动、System Prompt 工程
4. **[[04-rag-knowledge-system|RAG 知识库系统]]** — OpenViking、分块策略、Embedding/Rerank
5. **[[05-document-parsing-pipeline|文档解析管线]]** — MinerU/PaddleX/DeepSeek OCR/RapidOCR
6. **[[06-streaming-chat-and-agent-run|流式对话与 Agent Run 系统]]** — SSE 流式、ARQ 任务队列
7. **[[07-voice-interview-system|语音面试系统]]** — WebSocket、豆包 TTS、Fun-ASR
8. **[[08-frontend-architecture|前端架构深度剖析]]** — Vue3/Pinia/Ant Design/API 层
9. **[[09-deployment-and-production|部署与生产落地实战]]** — Docker Compose、性能优化、坑点总结
