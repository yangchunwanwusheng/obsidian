---
type: lesson
tags: [AI面试, 系统架构, FastAPI, Docker, 微服务]
created: 2026-05-25
updated: 2026-05-25
difficulty: advanced
prerequisites: [FastAPI基础, Docker Compose, 微服务概念]
topic: 开源项目深度拆解
status: completed
series: { name: "ai-interview-伯乐深度拆解", part: 1 }
---

# 整体架构深度剖析

> 从顶层视角拆解伯乐的架构设计哲学、服务拓扑、分层策略与关键设计决策。

---

## 一、架构全景图

```
                          ┌──────────────────────────┐
                          │     Nginx (生产模式)       │
                          │     端口 80 → 前端静态     │
                          │     /api/* → 后端代理      │
                          └──────────┬───────────────┘
                                     │
              ┌──────────────────────┼──────────────────────┐
              ▼                      ▼                      ▼
    ┌─────────────────┐   ┌─────────────────┐   ┌─────────────────┐
    │   web-dev:5173  │   │   api-dev:5050  │   │ worker-dev      │
    │   Vue 3 + Vite  │   │   FastAPI       │   │ ARQ 后台任务     │
    │   (热重载)       │   │   (热重载)       │   │                 │
    └─────────────────┘   └────────┬────────┘   └────────┬────────┘
                                   │                      │
                    ┌──────────────┼──────────────────────┤
                    │              │                      │
                    ▼              ▼                      ▼
          ┌─────────────┐ ┌─────────────┐ ┌─────────────────────┐
          │ PostgreSQL  │ │   Redis 7   │ │       MinIO         │
          │    16       │ │ 缓存/队列    │ │    对象存储           │
          │   :5432     │ │             │ │    :9000/:9001       │
          └─────────────┘ └─────────────┘ └─────────────────────┘
```

---

## 二、核心服务详解

### 2.1 API 服务 (FastAPI on Uvicorn)

**入口文件**：`server/main.py`

**应用组装流程**：
```python
# 1. 创建 FastAPI 应用
app = FastAPI(lifespan=lifespan)

# 2. 注册路由（所有路由统一在 /api 前缀下）
app.include_router(router, prefix="/api")

# 3. 中间件注册顺序（重要！后注册的先执行）
#    a. CORS 中间件（最底层，允许跨域）
#    b. AccessLogMiddleware（访问日志）
#    c. LoginRateLimitMiddleware（登录限流）
#    d. AuthMiddleware（鉴权，当前注释掉了实际逻辑）
```

**中间件执行顺序**（洋葱模型）：
```
请求 → LoginRateLimit → Auth → AccessLog → 路由处理 → AccessLog → Auth → LoginRateLimit → 响应
```

**异常处理体系**（精心设计的三层结构）：
```python
# 第一层：HTTPException → {"detail": str, "code": "http_xxx"}
# 第二层：知识库异常 → 404/400/500 分类处理
#    KBNotFoundError   → 404
#    KBOperationError  → 400
#    KnowledgeBaseException → 500
# 第三层：兜底 Exception → 500 + 日志（不泄露 traceback）
```

> **设计分析**：统一的 `{"detail": str, "code": str}` 响应形状是所有 API 的契约。前端只需解析这两个字段即可处理所有错误情况，极大降低了前端的错误处理复杂度。

### 2.2 Worker 服务 (ARQ 任务队列)

**入口文件**：`server/worker_main.py`

```python
# WorkerSettings 定义 ARQ worker 的配置
# 工作模式：
#   1. 接收来自 API 服务的 Redis 队列任务
#   2. 执行 Agent Run（长时间运行的 LLM 调用）
#   3. 通过 Redis Stream 向前端推送执行事件
```

**为什么需要独立的 Worker？**

| 问题 | 解决方案 |
|------|---------|
| Agent Run 可能执行数分钟 | 不能阻塞 API 的事件循环 |
| 大量并发 Run 请求 | Worker 池化处理，Redis 队列缓冲 |
| 前端需要实时进度 | Redis Stream 推送事件，前端轮询 |
| 需要取消正在运行的 Run | Redis 键作为取消信号 |

**数据流**：
```
用户点击"运行Agent" 
  → API 接收请求，创建 Run 记录 (status=queued)
  → 将 Run 任务推入 Redis 队列
  → Worker 取出任务，执行 Agent
  → Worker 通过 Redis Stream 推送事件
  → 前端轮询 /api/runs/{id}/events 获取进度
```

### 2.3 前端服务 (Vue 3 + Vite)

**开发模式** (`docker-compose.yml`):
```yaml
web:
  target: development      # 多阶段构建的目标
  command: pnpm install && pnpm run server  # Vite dev server
  volumes:                 # 挂载源码，支持热重载
    - ./web/src:/app/src
    - ./web/vite.config.js:/app/vite.config.js
```

**生产模式** (`docker-compose.prod.yml`):
```yaml
web:
  target: production       # 多阶段构建的目标
  command: nginx -g "daemon off;"  # Nginx 提供静态文件
  ports:
    - "80:80"              # 直接暴露 80 端口
```

> **设计分析**：多阶段 Docker 构建是前端容器化的最佳实践。开发阶段用 Vite dev server（热重载），生产阶段用 Nginx（高效静态文件服务 + API 代理）。同一份 Dockerfile，不同 target，保证了环境一致性。

---

## 三、后端分层架构详解

```
server/                        ← 应用层（薄层）
  ├── main.py                  ← 应用入口
  ├── routers/                 ← API 路由（请求验证、参数解析）
  └── utils/                   ← 工具函数（中间件、认证）

src/                           ← 核心业务逻辑（厚层）
  ├── agents/                  ← LangGraph Agent 层
  │   ├── common/              ← 基础设施（BaseAgent, BaseContext, 中间件）
  │   ├── interview_agent/     ← 面试 Agent
  │   ├── chatbot/             ← 通用聊天 Agent
  │   ├── reporter/            ← 报表 Agent
  │   ├── deep_agent/          ← 深度分析 Agent
  │   └── skills/              ← 技能定义（SKILLS.md）
  ├── knowledge/               ← 知识库系统
  │   ├── base.py              ← 抽象基类（~1200行，核心）
  │   ├── implementations/     ← 具体实现（OpenViking）
  │   ├── chunking/            ← 分块策略
  │   ├── indexing.py          ← 索引管理
  │   └── manager.py           ← 知识库生命周期管理
  ├── models/                  ← 模型封装
  │   ├── chat.py              ← LLM 聊天模型
  │   ├── embed.py             ← Embedding 模型
  │   └── rerank.py            ← Rerank 模型
  ├── plugins/                 ← 文档解析插件
  ├── repositories/            ← 数据访问层（SQLAlchemy）
  ├── services/                ← 业务服务层
  └── storage/                 ← 存储抽象层
      ├── minio/               ← 对象存储
      └── postgres/            ← 数据库连接管理
```

### 3.1 为什么 server/ 要薄、src/ 要厚？

这是一个经典的**关注点分离**决策：

| 层 | 职责 | 不做什么 |
|----|------|---------|
| `server/` | HTTP 请求/响应处理、参数验证、状态码 | 不包含业务逻辑 |
| `src/` | 纯业务逻辑，与 HTTP 无关 | 不导入 FastAPI/Request |

好处：
- `src/` 的代码可以在非 HTTP 环境（如 Worker）中直接复用
- 单元测试不需要启动 HTTP 服务器
- 更换 Web 框架时只需改 `server/` 层

### 3.2 Router 命名规范

```
server/routers/
├── auth_router.py       ←  /api/auth/*
├── chat_router.py       ←  /api/chat/*
├── knowledge_router.py  ←  /api/knowledge/*
├── resume_router.py     ←  /api/resume/*
├── job_router.py        ←  /api/job/*
├── evaluation_router.py ←  /api/evaluation/*
├── mcp_router.py        ←  /api/mcp/*
├── skill_router.py      ←  /api/skill/*
├── tool_router.py       ←  /api/tool/*
├── system_router.py     ←  /api/system/*
├── department_router.py ←  /api/department/*
├── task_router.py       ←  /api/task/*
├── video_router.py      ←  /api/video/*
├── dashboard_router.py  ←  /api/dashboard/*
└── mindmap_router.py    ←  /api/mindmap/*
```

---

## 四、数据存储架构

### 4.1 PostgreSQL 16 — 关系型数据主存储

存储内容：
- 用户/部门/角色（认证与权限）
- Agent 配置与元数据
- 对话记录与消息历史
- 知识库元数据与文件状态
- 面试记录与评分卡
- 简历结构化数据

### 4.2 Redis 7 — 缓存与消息队列

存储/用途：
- **ARQ 任务队列**：Agent Run 的异步执行
- **Redis Stream**：Run 事件流（前端轮询）
- **限流计数器**：登录频率限制
- **Cancel 信号键**：Run 取消机制
- ARQ 内置的 Job 结果存储

### 4.3 MinIO — 对象存储

存储内容：
- 用户上传的原始文件（简历 PDF/Word/图片等）
- 解析后的文档（Markdown 中间产物）
- 向量索引文件
- 其他二进制大对象

> **为什么用 MinIO 而不是本地文件系统？** MinIO 兼容 S3 API，可以无缝迁移到云上的 S3/OSS。同时它提供了 Web 控制台（端口 9001），方便开发调试时查看存储内容。

---

## 五、Docker 服务拓扑

```
┌─────────────────────────────────────────────────────────────┐
│                    app-network (bridge)                     │
│                                                             │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌────────────┐ │
│  │   api    │  │  worker  │  │   web    │  │ kb-import  │ │
│  │  :5050   │  │          │  │  :5173   │  │ (一次性)    │ │
│  └────┬─────┘  └────┬─────┘  └──────────┘  └─────┬──────┘ │
│       │             │                             │        │
│       ├─────────────┼─────────────────────────────┘        │
│       │             │                                      │
│  ┌────▼─────┐  ┌────▼─────┐  ┌──────────┐                │
│  │ postgres │  │  redis   │  │  minio   │                │
│  │  :5432   │  │          │  │ :9000/01 │                │
│  └──────────┘  └──────────┘  └──────────┘                │
│                                                             │
│  ─────────────── profile: all ───────────────              │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐                │
│  │ mineru-  │  │ mineru-  │  │ paddlex  │                │
│  │ vllm     │  │ api      │  │  :8080   │                │
│  │ :30000   │  │ :30001   │  │  (GPU)   │                │
│  │ (GPU)    │  │          │  │          │                │
│  └──────────┘  └──────────┘  └──────────┘                │
│                                                             │
│  ──────────── OJ 子系统 ────────────────                   │
│  ┌──────────┐  ┌──────────┐  ┌──────────────────────────┐ │
│  │oj-backend│  │oj-judge  │  │oj-postgres / oj-redis    │ │
│  │ :8000    │  │ :8080    │  │(独立实例)                  │ │
│  └──────────┘  └──────────┘  └──────────────────────────┘ │
└─────────────────────────────────────────────────────────────┘
```

### 服务依赖与启动顺序

```
postgres (healthy) ──┐
redis (healthy)    ──┼──→ api (healthy) ──→ web / kb-import
minio (healthy)    ──┘          │
                                └──→ worker
```

启动顺序由 `depends_on` + `condition: service_healthy` 保证：

```yaml
api:
  depends_on:
    postgres:
      condition: service_healthy  # 必须等 PG 准备好
    redis:
      condition: service_healthy  # 必须等 Redis 准备好
    minio:
      condition: service_healthy  # 必须等 MinIO 准备好
```

---

## 六、核心设计哲学

### 6.1 拒绝过度设计

从 CLAUDE.md 中可以看到这条原则贯穿始终：
> "Avoid over-engineering. Only make changes that are directly requested or clearly necessary."

具体体现：
- 不使用微服务拆分，所有业务逻辑在一个容器内
- 中间件体系只在需要时添加，不预置"可能用到"的中间件
- 不使用 ORM 的复杂关系映射，Repository 层保持简单

### 6.2 可配置优于硬编码

配置的三层体系：
```
环境变量 (.env) → 运行时配置 (config.yaml) → 类默认值
```

所有关键参数（模型选择、分块策略、Embedding 维度、超时时间）都通过配置注入，确保同一份代码在不同环境下行为可调。

### 6.3 热重载开发体验

```yaml
# 后端热重载
command: uvicorn server.main:app --host 0.0.0.0 --port 5050 --reload
volumes:
  - ./server:/app/server   # 挂载源码
  - ./src:/app/src         # 挂载核心逻辑

# 前端热重载
command: pnpm run server   # Vite dev server
volumes:
  - ./web/src:/app/src     # 挂载前端源码
```

开发体验：修改代码 → 保存 → 自动重载 → 刷新浏览器，无需手动重启容器。

---

## 下一步学习

阅读 [[02-langgraph-agent-system|LangGraph Agent 系统深度拆解]]，深入理解 BaseAgent、BaseContext、中间件栈和 Checkpointer 机制。
