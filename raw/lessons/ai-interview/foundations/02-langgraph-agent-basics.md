---
type: lesson
tags: [LangGraph, LangChain, Agent, 基础知识, ReAct]
created: 2026-05-25
updated: 2026-05-25
difficulty: beginner
prerequisites: [Python基础]
topic: 开源项目深度拆解
status: completed
series: { name: "ai-interview-伯乐深度拆解-基础篇", part: 2 }
---

# LangGraph 与 Agent 基础概念

> LangChain 是什么？LangGraph 又是什么？Agent 到底怎么工作的？本文从零讲清 Agent 框架的核心概念。

---

## 一、先有 LangChain，再有 LangGraph

### LangChain 是什么？

**LangChain 是一个 LLM 应用开发框架**。它提供了一套标准化的组件：

```
LangChain 核心组件：

┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐
│  Model   │  │ Messages │  │  Tools   │  │  Prompt  │  │  Chains  │
│ 模型抽象  │  │ 消息对象  │  │ 工具定义  │  │ 提示模板  │  │  调用链   │
└──────────┘  └──────────┘  └──────────┘  └──────────┘  └──────────┘
```

**用 LangChain 调用 LLM 只需要 3 行**：
```python
from langchain_openai import ChatOpenAI

llm = ChatOpenAI(model="deepseek-chat", base_url="https://api.deepseek.com")
response = llm.invoke("你好，请用一句话介绍自己")
```

**但 LangChain 有自己的局限**：它的 `Chain` 是线性的（A→B→C），不适合需要循环和条件分支的 Agent 场景。

### LangGraph 是什么？

**LangGraph 是 LangChain 团队推出的状态图框架**，专门解决 Agent 的循环调用问题。

核心思想：**把 Agent 的行为建模为有向图**。

```
一个最简单的 Agent 图：

         ┌─────────┐
         │  START  │
         └────┬────┘
              │
              ▼
         ┌─────────┐     需要调用工具     ┌─────────┐
         │   LLM   │ ──────────────────→ │  Tools  │
         │  节点   │                      │  节点   │
         └────┬────┘                      └────┬────┘
              │                                │
              │ 不需要调用工具                   │
              ▼                                │
         ┌─────────┐                           │
         │   END   │  ←────────────────────────┘
         └─────────┘   工具结果返回 LLM，继续循环
```

> **一句话区分**：LangChain 给你零件（模型、消息、工具），LangGraph 给你编排能力（循环、分支、状态管理）。

---

## 二、Agent 到底是怎么工作的？

### Agent = LLM + 工具 + 循环

```
传统程序：                         Agent：
                                  ┌──────────────────────────┐
输入 → 处理 → 输出                 │  while not 任务完成:       │
                                  │    思考: "我现在该干什么？" │
  (单次执行，线性)                  │    决策: "需要调用哪个工具？" │
                                  │    行动: 调用工具获取结果   │
                                  │    观察: 分析工具返回结果   │
                                  │    反思: "我的判断对吗？"   │
                                  └──────────────────────────┘
                                        (循环执行，自适应)
```

### 经典模式：ReAct（Reasoning + Acting）

这是目前最主流的 Agent 模式，伯乐的面试 Agent 也是基于此。

```
用户："帮我查一下 HashMap 的底层原理"

第 1 轮：
  思考: "用户想了解 HashMap，我需要搜索知识库"
  行动: 调用 search_knowledge("HashMap 底层原理")
  观察: 返回了 3 段 HashMap 相关文档

第 2 轮：
  思考: "我有了 HashMap 的资料，可以回答了"
  行动: 直接回复用户（不需要再调工具）
  输出: "HashMap 底层是数组+链表+红黑树..."
```

### 代码视角：Agent 循环的骨架

```python
# 一个最简 Agent 循环（就是 LangGraph 内部做的事）

async def agent_loop(user_message, tools, llm):
    messages = [HumanMessage(content=user_message)]
    
    while True:
        # 1. LLM 思考
        response = await llm.ainvoke(messages)
        
        # 2. 检查是否有工具调用
        if not response.tool_calls:
            # 没有工具调用 → LLM 觉得可以回复了
            return response.content
        
        # 3. 执行工具调用
        for tool_call in response.tool_calls:
            tool = tools[tool_call["name"]]
            result = await tool.ainvoke(tool_call["args"])
            
            # 4. 把工具结果加入对话
            messages.append(response)           # LLM 的思考
            messages.append(result)             # 工具的执行结果
            # 然后继续循环 → LLM 看到结果后再决定下一步
```

---

## 三、LangGraph 的核心概念

### 3.1 State（状态）

State 是图执行过程中共享的数据容器。所有节点都能读写它。

```python
from typing import Annotated, Sequence
from langgraph.graph import add_messages
from langchain_core.messages import AnyMessage

class AgentState(TypedDict):
    messages: Annotated[Sequence[AnyMessage], add_messages]
    # add_messages: 新消息不是覆盖旧消息，而是追加
```

**伯乐的做法**：`BaseState` 只包含 `messages` 一个字段，极简设计。

### 3.2 Node（节点）

节点是图中的处理单元。每个节点做一件事。

```python
# 节点就是一个普通函数
def chatbot(state: AgentState):
    """调用 LLM，返回新的消息"""
    response = llm.invoke(state["messages"])
    return {"messages": [response]}

def tools(state: AgentState):
    """执行工具调用"""
    last_message = state["messages"][-1]
    results = execute_tools(last_message.tool_calls)
    return {"messages": results}
```

### 3.3 Edge（边）

边决定节点之间如何连接。

```python
# 普通边：无条件跳转
graph.add_edge("chatbot", "tools")    # chatbot → tools

# 条件边：根据状态决定跳转
def should_continue(state: AgentState) -> str:
    last = state["messages"][-1]
    if last.tool_calls:
        return "tools"     # 有工具调用 → 跳 tools 节点
    return "__end__"       # 没有 → 结束

graph.add_conditional_edges("chatbot", should_continue)
```

### 3.4 Checkpointer（检查点/持久化）

```python
from langgraph.checkpoint.memory import MemorySaver

# 每一轮对话后自动保存状态
graph = builder.compile(checkpointer=MemorySaver())

# 下次同一个 thread_id 进来，自动恢复历史
config = {"configurable": {"thread_id": "user_123_chat"}}
response = graph.invoke({"messages": [new_message]}, config)
# ← graph 会自动加载之前的所有消息！
```

**这是 LangGraph 最强大的功能之一**：你不需要手动管理对话历史，Checkpointer 全自动处理。

### 3.5 完整的最小 Agent 图

```python
from langgraph.graph import StateGraph, START, END

# 1. 建图
builder = StateGraph(AgentState)

# 2. 加节点
builder.add_node("chatbot", chatbot)
builder.add_node("tools", tools)

# 3. 加边
builder.add_edge(START, "chatbot")
builder.add_conditional_edges("chatbot", should_continue)
builder.add_edge("tools", "chatbot")  # 工具执行完，回到 LLM 继续思考

# 4. 编译（此时才真正可用）
graph = builder.compile(checkpointer=checkpointer)
```

---

## 四、langchain.agents.create_agent 是什么？

你可能会困惑：伯乐代码里用的是 `create_agent()`，上面讲的却是手动建图。它们的关系是：

```python
# 手动建图（底层 API）
from langgraph.graph import StateGraph
graph = StateGraph(AgentState)
graph.add_node(...)
# ... 需要自己写所有节点

# create_agent（高层 API）—— 伯乐用的方式
from langchain.agents import create_agent
graph = create_agent(
    model=llm,
    middleware=[...],        # 中间件 = 插件化的能力注入
    checkpointer=checkpointer,
)
# create_agent 内部已经帮你建好了 ReAct 循环图
# 你只需要配置 model、middleware、checkpointer
```

`create_agent` 是 LangChain 1.x 新增的高级 API，它封装了 ReAct 循环的建图细节。

---

## 五、中间件（Middleware）—— Agent 的插件系统

### 什么是中间件？

中间件是在 Agent 循环中注入额外行为的组件。类似于 Web 框架的中间件。

```python
# 一个最简单的中间件
class KnowledgeBaseMiddleware(AgentMiddleware):
    def __init__(self):
        # 中间件可以注入工具
        self.tools = [query_knowledge_base]

# 使用
create_agent(
    model=llm,
    middleware=[KnowledgeBaseMiddleware()],  # 这个 Agent 就有了知识库检索能力
)
```

### 中间件在 Agent 循环中的位置

```
请求进入
  ↓
middleware[0].before_agent()     ← 可以注入 system prompt
  ↓
middleware[1].before_model()     ← 可以注入检索上下文
  ↓
[ LLM 调用 ]
  ↓
middleware[1].after_model()      ← 可以触发摘要
  ↓
middleware[0].after_agent()      ← 可以保存日志
  ↓
输出
```

### 伯乐的中间件栈（11 层）

伯乐面试 Agent 使用了 11 个中间件，每个负责一项能力：

```
save_attachments_to_fs        → 保存用户上传的简历文件
FilesystemMiddleware          → 提供 read_file 工具（只读）
InterviewKnowledgeBaseMiddleware → 注入面试专用工具（query_kb 等）
RuntimeConfigMiddleware       → 允许运行时切换模型
OpenVikingContextMiddleware   → 注入 RAG 检索到的知识
VideoContextMiddleware        → 注入视频分析结果（表情/情绪）
TodoListMiddleware            → 强制维护 6 步面试任务清单
PatchToolCallsMiddleware      → 修复 LLM 工具调用格式错误
OpenVikingSummaryMiddleware   → 长对话自动摘要（防超上下文窗口）
ToolCallLimitMiddleware       → 限制工具调用次数（防滥用）
ModelRetryMiddleware          → LLM 调用失败自动重试
```

**这就是"可插拔"的精髓**：要给 Agent 加新能力？加一个中间件。要去掉某个能力？移除对应中间件。

---

## 六、常见误区

| 误解                        | 真相                             |
| ------------------------- | ------------------------------ |
| "LangGraph 必须用 LangChain" | LangGraph 可以独立使用，不依赖 LangChain |
| "create_agent 是唯一方式"      | 也可以手动建图。create_agent 是高层封装     |
| "中间件越多越好"                 | 顺序很重要，太多也会增加复杂度和延迟             |
| "Agent = ChatBot"         | Agent 多了工具使用和自主决策能力            |

---

## 下一步学习

阅读 [[03-embedding-and-rerank|Embedding 与 Rerank 基础概念]]，理解文字如何变成向量、向量如何比较相似度。
