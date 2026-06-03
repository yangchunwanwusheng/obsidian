---
type: lesson
tags: [FastAPI, Vue3, 前后端, 基础知识]
created: 2026-05-25
updated: 2026-05-25
difficulty: beginner
prerequisites: [Python基础, JavaScript基础]
topic: 开源项目深度拆解
status: completed
series: { name: "ai-interview-伯乐深度拆解-基础篇", part: 6 }
---

# FastAPI 与 Vue3 基础概念

> 伯乐的前后端技术栈：FastAPI（Python 后端）和 Vue3（JavaScript 前端）。本文从零讲清它们为什么被选中，以及核心概念。

---

## 一、FastAPI — Python 的现代 Web 框架

### 1.1 为什么选 FastAPI？

| 特性 | FastAPI | Flask | Django |
|------|:---:|:---:|:---:|
| 异步支持 | ✅ 原生 | ❌ 需扩展 | ⚠️ 3.1+ |
| 自动生成 API 文档 | ✅ Swagger | ❌ 需插件 | ⚠️ 需配置 |
| 数据验证 | ✅ Pydantic | ❌ 手动 | ✅ DRF |
| 性能 | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐ |
| 学习曲线 | 低 | 低 | 高 |

伯乐需要异步处理大量的 LLM 调用（HTTP 请求到 OpenAI/SiliconFlow API），FastAPI 的异步特性是刚需。

### 1.2 一个最小的 FastAPI 应用

```python
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()

# 请求模型（自动验证输入）
class ChatRequest(BaseModel):
    message: str
    thread_id: str

# API 路由
@app.post("/api/chat")
async def chat(request: ChatRequest):
    # 自动：如果 message 为空或 thread_id 缺失，返回 422 错误
    return {"reply": f"你好，你说的是：{request.message}"}

# 启动：uvicorn main:app --reload
```

### 1.3 FastAPI 的核心概念

**路由（Router）** — 把相关 API 组织在一起：
```python
# server/routers/chat_router.py
from fastapi import APIRouter

router = APIRouter(prefix="/chat", tags=["chat"])

@router.post("/stream")
async def stream_chat(request: ChatRequest):
    ...

# server/main.py 中注册
app.include_router(router, prefix="/api")
```

**依赖注入（Depends）** — 自动注入认证信息：
```python
from fastapi import Depends

async def get_current_user(token: str = Header(...)):
    return verify_token(token)

@app.get("/api/me")
async def me(user = Depends(get_current_user)):
    return user  # user 自动注入，不需要手动解析 token
```

**中间件（Middleware）** — 在请求前后执行：
```python
@app.middleware("http")
async def add_process_time_header(request, call_next):
    # 请求前...
    start = time.time()
    response = await call_next(request)
    # 请求后...
    response.headers["X-Process-Time"] = str(time.time() - start)
    return response
```

**流式响应（StreamingResponse）** — 逐块返回数据：
```python
from fastapi.responses import StreamingResponse

@app.post("/api/chat/stream")
async def stream():
    async def generate():
        for i in range(10):
            yield f"data: chunk_{i}\n\n"  # 逐条推送
    return StreamingResponse(generate(), media_type="text/event-stream")
```

### 1.4 Pydantic — 数据验证引擎

```python
from pydantic import BaseModel, Field

class CreateAgentRequest(BaseModel):
    name: str = Field(..., min_length=1, max_length=50)
    model: str = "deepseek-chat"
    temperature: float = Field(default=0.7, ge=0, le=2.0)
    tools: list[str] = []

# 自动验证：
# - name 不能为空
# - temperature 必须在 0-2 之间
# - tools 默认为空列表

# 如果传入 {"name": "", "temperature": 3.0}
# → 自动返回 422 错误，包含详细验证信息
```

---

## 二、Vue 3 — 前端框架

### 2.1 为什么选 Vue？

| 特性 | Vue 3 | React | Angular |
|------|:---:|:---:|:---:|
| 学习曲线 | ⭐⭐ 平缓 | ⭐⭐⭐ 需要 JSX | ⭐⭐⭐⭐⭐ 陡峭 |
| 响应式 | ✅ 自动追踪 | ⚠️ 需手动优化 | ✅ Zone.js |
| 官方状态管理 | Pinia（好用） | 无（Redux复杂） | RxJS（难） |
| 官方路由 | Vue Router | React Router | 内置 |
| 国内生态 | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐ |

伯乐选择 Vue 的理由：开发效率高，Pinia 简洁，Ant Design Vue 成熟。

### 2.2 Vue 3 核心概念

**单文件组件（SFC）**：
```vue
<template>
  <!-- HTML：页面结构 -->
  <div class="chat-container">
    <div v-for="msg in messages" :key="msg.id">
      {{ msg.content }}
    </div>
  </div>
</template>

<script setup>
// JavaScript：组件逻辑
import { ref, onMounted } from 'vue'

const messages = ref([])  // 响应式数据，变化时自动更新页面

onMounted(async () => {
  const data = await fetch('/api/messages')
  messages.value = await data.json()
})
</script>

<style scoped>
/* CSS：组件样式（scoped 表示只影响本组件） */
.chat-container { padding: 16px; }
</style>
```

**响应式系统** — Vue 的核心魔法：
```javascript
import { ref, computed, watch } from 'vue'

const count = ref(0)

// 自动追踪依赖，count 变 → double 自动变
const double = computed(() => count.value * 2)

// 自动追踪依赖，count 变 → 自动执行
watch(count, (newVal, oldVal) => {
  console.log(`count 从 ${oldVal} 变为 ${newVal}`)
})

// 改变 count，double 自动更新，watch 自动触发
count.value++  // double 自动变为 2，控制台自动输出
```

### 2.3 Pinia — 状态管理

```javascript
// web/src/stores/agent.js
import { defineStore } from 'pinia'

export const useAgentStore = defineStore('agent', () => {
    const currentAgent = ref(null)
    const threadId = ref(uuid())

    // 切换 Agent 时自动重置 threadId
    watch(currentAgent, () => {
        threadId.value = uuid()
    })

    return { currentAgent, threadId }
})

// 任何组件中都能访问同一个 store 实例
// 组件 A 修改了 currentAgent
// 组件 B 自动看到最新值
```

### 2.4 Vue Router — 路由与权限

```javascript
const routes = [
    {
        path: '/',
        component: MainLayout,
        meta: { requiresAuth: true },  // 需要登录
        children: [
            { path: 'agents', component: AgentList },
            { path: 'interview/:id', component: InterviewSession },
        ]
    },
    { path: '/login', component: Login },
]

// 路由守卫：未登录自动跳转
router.beforeEach((to, from, next) => {
    if (to.meta.requiresAuth && !getToken()) {
        next('/login')
    } else {
        next()
    }
})
```

### 2.5 Vite — 构建工具

```javascript
// vite.config.js
export default {
    server: {
        port: 5173,
        proxy: {
            '/api': 'http://api:5050'  // 开发时代理到后端
        }
    }
}
```

Vite 的特点：
- 开发时**毫秒级热重载**（改了代码，浏览器瞬间看到效果）
- 构建时**秒级打包**（比 Webpack 快 10-100 倍）
- 原生支持 TypeScript、CSS 预处理器

---

## 三、前后端如何协作？

```
┌─────────────────────┐
│   浏览器 (Vue 3)     │
│                     │
│  用户点击"发送消息"   │
│       │             │
│       ▼             │
│  fetAPI 请求 ─────────→  FastAPI 后端
│  POST /api/chat      │       │
│       │             │       ▼
│       │             │  InterviewAgent
│       │             │       │
│       │             │       ▼
│       │             │  SSE 流式返回
│       │  ← ─ ─ ─ ─ ─ ─ ─ ┘
│       ▼             │
│  逐字显示在聊天框    │
│                     │
└─────────────────────┘
```

---

## 下一步

回到 [[raw/lessons/ai-interview/00-essence|伯乐深度拆解总览]]，在理解了所有基础概念后，重新阅读代码拆解笔记，会有完全不同的理解深度。
