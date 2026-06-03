---
type: lesson
tags: [Vue3, Pinia, AntDesign, Vite, 前端架构, 状态管理, 路由守卫]
created: 2026-05-25
updated: 2026-05-25
difficulty: intermediate
prerequisites: [Vue3, Pinia, Ant Design Vue, Vite]
topic: 开源项目深度拆解
status: completed
series: { name: "ai-interview-伯乐深度拆解", part: 8 }
---

# 前端架构深度剖析

> 伯乐的前端是一个典型的 Vue3 企业级 SPA，使用 Pinia 状态管理、Ant Design Vue 组件库、Vite 构建工具。本节拆解其架构设计、API 层抽象、路由守卫和主题系统。

---

## 一、技术栈与架构概览

```
web/src/
├── apis/                    # API 层（统一管理所有 HTTP 请求）
│   ├── base.js             # 基础方法：apiGet, apiPost, apiAdminGet 等
│   ├── auth.js             # 认证 API
│   ├── agent.js            # 智能体 API
│   ├── chat.js             # 对话 API
│   ├── knowledge.js        # 知识库 API
│   ├── resume.js           # 简历 API
│   ├── interview.js        # 面试 API
│   └── ...
├── views/                   # 页面组件（按功能模块拆分）
│   ├── Agent/              # 智能体管理页
│   ├── InterviewSession/   # 面试会话页（核心页面）
│   ├── Resume/             # 简历管理页
│   ├── Database/           # 知识库管理页
│   ├── Dashboard/          # 仪表盘
│   ├── Extensions/         # 扩展管理（MCP/Tools/Skills）
│   └── Auth/               # 登录/注册
├── components/              # 通用组件
│   ├── AgentChat/          # Agent 对话组件（最核心）
│   ├── ToolCallingResult/  # 工具调用结果展示
│   ├── dashboard/          # 仪表盘组件
│   ├── modals/             # 模态框组件
│   └── sources/            # 知识来源展示
├── stores/                  # Pinia 状态管理
│   ├── user.js             # 用户认证状态
│   ├── agent.js            # 当前智能体状态
│   ├── chatUI.js           # 对话界面状态
│   ├── config.js           # 应用配置
│   ├── database.js         # 知识库状态
│   ├── tasker.js           # 任务管理状态
│   └── theme.js            # 主题状态（暗色模式）
├── router/                  # Vue Router
│   └── index.js            # 路由定义 + 权限守卫
└── assets/
    └── css/
        └── base.css         # 全局设计变量（颜色/阴影/滚动条）
```

---

## 二、API 层设计 — 集中式 HTTP 请求管理

### 2.1 核心抽象

```javascript
// web/src/apis/base.js

const BASE_URL = import.meta.env.VITE_API_URL || 'http://localhost:5050';

// 基础方法：自动附加 token
export async function apiGet(path, params = {}) {
    const token = useUserStore().token;
    const response = await fetch(`${BASE_URL}/api${path}`, {
        headers: {
            'Authorization': `Bearer ${token}`,
            'Content-Type': 'application/json',
        },
    });
    if (!response.ok) {
        const error = await response.json();
        throw new ApiError(error.detail, error.code, response.status);
    }
    return response.json();
}

export async function apiPost(path, body = {}) { /* 同上 */ }
export async function apiAdminGet(path, params) { /* 管理员专用 */ }
export async function apiSuperAdminGet(path, params) { /* 超级管理员专用 */ }
```

**设计优点**：
- 所有 API 调用集中在一处，方便统一修改 base URL、拦截器
- Token 自动附加，避免每个请求手动添加
- 统一错误处理（抛出 `ApiError`，组件层统一 catch）

### 2.2 按模块拆分

```javascript
// web/src/apis/agent.js
export function getAgents() {
    return apiGet('/agents');
}

export function updateAgentConfig(agentId, config) {
    return apiPost(`/agents/${agentId}/config`, config);
}

// web/src/apis/chat.js
export function streamChat(agentId, messages, threadId) {
    // SSE 流式请求，不走 apiGet（需要特殊处理）
    const token = useUserStore().token;
    return fetch(`${BASE_URL}/api/chat/stream`, {
        method: 'POST',
        headers: {
            'Authorization': `Bearer ${token}`,
            'Content-Type': 'application/json',
        },
        body: JSON.stringify({ agent_id: agentId, messages, thread_id: threadId }),
    });
}
```

---

## 三、Pinia 状态管理 — Composition API 风格

### 3.1 User Store

```javascript
// web/src/stores/user.js
export const useUserStore = defineStore('user', () => {
    // state
    const token = ref(localStorage.getItem('token') || '');
    const userInfo = ref(null);
    const isAdmin = computed(() => userInfo.value?.role === 'admin');
    
    // actions
    async function login(username, password) {
        const res = await apiPost('/auth/token', { username, password });
        token.value = res.access_token;
        localStorage.setItem('token', res.access_token);
        await fetchUserInfo();
    }
    
    function logout() {
        token.value = '';
        userInfo.value = null;
        localStorage.removeItem('token');
        router.push('/login');
    }
    
    return { token, userInfo, isAdmin, login, logout };
});
```

### 3.2 Agent Store（核心）

```javascript
// web/src/stores/agent.js
export const useAgentStore = defineStore('agent', () => {
    const currentAgent = ref(null);
    const agents = ref([]);
    const config = ref({});
    const threadId = ref(generateThreadId());
    
    // 切换 Agent 时重置 threadId
    watch(currentAgent, () => {
        threadId.value = generateThreadId();
    });
    
    return { currentAgent, agents, config, threadId };
});
```

**关键设计**：切换 Agent 时自动生成新的 `thread_id`，确保不同 Agent 的对话历史隔离。

---

## 四、路由与权限守卫

### 4.1 路由结构

```javascript
const routes = [
    {
        path: '/',
        component: AppLayout,     // 主布局（含侧边栏、顶栏）
        meta: { requiresAuth: true },
        children: [
            { path: 'agents', component: () => import('@/views/Agent/List.vue') },
            { path: 'agents/:id', component: () => import('@/views/Agent/Chat.vue') },
            { path: 'interview/:id', component: () => import('@/views/InterviewSession/Index.vue') },
            { path: 'knowledge', component: () => import('@/views/Database/List.vue') },
            { path: 'resume', component: () => import('@/views/Resume/List.vue') },
            // ...需要登录的页面
        ]
    },
    {
        path: '/',
        component: BlankLayout,   // 空白布局（无侧边栏）
        children: [
            { path: 'login', component: () => import('@/views/Auth/Login.vue') },
        ]
    }
];
```

### 4.2 权限守卫

```javascript
router.beforeEach((to, from, next) => {
    const userStore = useUserStore();
    
    // 1. 需要登录但未登录 → 跳转登录页
    if (to.meta.requiresAuth && !userStore.token) {
        return next('/login');
    }
    
    // 2. 需要管理员权限 → 检查角色
    if (to.meta.requiresAdmin && !userStore.isAdmin) {
        return next('/agents');  // 非管理员重定向到智能体页面
    }
    
    // 3. 需要超级管理员 → 检查角色
    if (to.meta.requiresSuperAdmin && !userStore.isSuperAdmin) {
        return next('/agents');
    }
    
    next();
});
```

---

## 五、设计系统 — CSS 变量主题

### 5.1 全局设计变量

```css
/* web/src/assets/css/base.css */

:root {
    /* 主色调 */
    --color-primary: #4f46e5;
    --color-primary-hover: #4338ca;
    
    /* 背景 */
    --color-bg-primary: #ffffff;
    --color-bg-secondary: #f9fafb;
    --color-bg-tertiary: #f3f4f6;
    
    /* 文字 */
    --color-text-primary: #111827;
    --color-text-secondary: #6b7280;
    
    /* 边框 */
    --color-border: #e5e7eb;
    
    /* 阴影 — 克制、不使用重阴影 */
    --shadow-sm: 0 1px 2px rgba(0, 0, 0, 0.05);
    --shadow-md: 0 4px 6px -1px rgba(0, 0, 0, 0.1);
    
    /* 滚动条 */
    --scrollbar-width: 6px;
    --scrollbar-color: #d1d5db;
}

/* 暗色模式 */
.dark {
    --color-bg-primary: #111827;
    --color-bg-secondary: #1f2937;
    --color-text-primary: #f9fafb;
    --color-text-secondary: #9ca3af;
    --color-border: #374151;
}
```

### 5.2 UI 设计原则

从 CLAUDE.md 中提取的前端设计规范：

- **图标**：优先使用 `lucide-vue-next`
- **样式**：通过 `less` 编写，引用全局 CSS 变量
- **简洁性**：禁止滥用重阴影、悬停位移、高饱和度渐变色
- **一致性**：所有颜色从 `base.css` 的变量中取值
- **暗色模式**：通过 `.dark` 类切换，所有组件自动适配

---

## 六、AgentChat 组件 — 对话核心

### 6.1 组件结构

```
AgentChat.vue
├── 消息列表
│   ├── 用户消息 (右对齐，蓝色气泡)
│   ├── Agent 消息（左对齐，灰色气泡，支持 Markdown 渲染）
│   └── 工具调用结果（可折叠卡片，按工具类型分组件）
├── 输入区
│   ├── 文本输入框
│   ├── 附件上传按钮
│   └── 发送按钮（支持 Enter 发送）
├── 面试侧边栏（面试模式下显示）
│   ├── 简历摘要
│   ├── 面试阶段指示器
│   ├── TodoList 展示
│   └── 评分卡预览
└── 代码工作台（代码考核时显示）
    ├── Monaco Editor
    └── OJ 判题结果
```

### 6.2 SSE 流式消费

```javascript
// 使用 fetch + ReadableStream 消费 SSE
async function sendMessage(text) {
    const response = await streamChat(agentId, messages, threadId);
    const reader = response.body.getReader();
    const decoder = new TextDecoder();
    
    let aiMessage = { role: 'assistant', content: '' };
    messages.value.push(aiMessage);
    
    while (true) {
        const { done, value } = await reader.read();
        if (done) break;
        
        const chunk = decoder.decode(value);
        const lines = chunk.split('\n');
        
        for (const line of lines) {
            if (line.startsWith('data: ')) {
                const data = JSON.parse(line.slice(6));
                
                if (data.type === 'chunk') {
                    // 增量追加文本
                    aiMessage.content += data.content;
                } else if (data.type === 'tool_result') {
                    // 展示工具调用结果
                    toolResults.value.push(data.content);
                } else if (data.type === 'done') {
                    // 流结束
                }
            }
        }
    }
}
```

### 6.3 ToolCallingResult — 按工具类型渲染

```vue
<template>
  <div class="tool-call-card">
    <div class="tool-header" @click="expanded = !expanded">
      <span class="tool-name">{{ toolCall.name }}</span>
      <span class="tool-status">{{ toolCall.status }}</span>
    </div>
    <div v-if="expanded" class="tool-content">
      <!-- 根据工具类型渲染不同组件 -->
      <QueryKBResult v-if="toolCall.name === 'query_kb'" :result="toolCall.result" />
      <CodeResult v-if="toolCall.name === 'start_code_assessment'" :result="toolCall.result" />
      <DefaultToolResult v-else :result="toolCall.result" />
    </div>
  </div>
</template>
```

---

## 七、从零手搓前端架构的关键步骤

### 最小的 AI 面试前端需要：

1. **Vue3 + Vite**：SPA 框架 + 构建工具
2. **API 层**：统一的 HTTP 请求管理（`apis/` 目录）
3. **SSE 消费**：`fetch + ReadableStream` 处理流式响应
4. **Pinia Store**：至少需要 User（认证）和 Agent（会话）两个 store
5. **路由守卫**：未登录跳转、权限检查
6. **CSS 变量体系**：统一颜色、间距、暗色模式
7. **AgentChat 组件**：消息列表 + 输入区 + Markdown 渲染 + 工具调用展示

---

## 下一步学习

阅读 [[09-deployment-and-production|部署与生产落地实战]]，理解 Docker Compose 部署、性能优化和常见踩坑。
