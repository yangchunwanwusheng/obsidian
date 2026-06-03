---
type: lesson
tags:
  - LangGraph
  - MemorySaver
  - SqliteSaver
  - Python
  - Agent
created: 2026-04-28
updated: 2026-04-28
topic: 记忆与检查点——让研究助手跨轮记住上下文
difficulty: intermediate
prerequisites:
  - "[[raw/lessons/LangGraph/04-tools-and-react|第 4 章：工具与 ReAct——让研究助手学会调用检索工具]]"
sources:
  - https://docs.langchain.com/oss/python/langgraph/add-memory
  - https://docs.langchain.com/oss/python/langgraph/thinking-in-langgraph
  - https://docs.langchain.com/oss/python/langgraph/interrupts
status: completed
series:
  name: LangGraph 系统学习
  part: 5
---

# 记忆与检查点——让研究助手跨轮记住上下文

> 一句话摘要：前面几章的 Research Copilot 已经会对话、会调工具，但它还没有真正稳定的“跨轮记忆”。本章会帮你理解 checkpointer 的核心地位，掌握 `MemorySaver`、`SqliteSaver`、`thread_id`，并把 Research Copilot 升级为真正能接着上次继续工作的 v3。

## 学习目标

完成本课后，你应该能够：
- [ ] 理解为什么“有 messages”不等于“有跨轮记忆”
- [ ] 知道 checkpointer 在 LangGraph 里的角色
- [ ] 会使用 `MemorySaver` 和 `SqliteSaver` 编译图
- [ ] 理解 `thread_id` 为什么是会话持续性的关键
- [ ] 能把 Research Copilot 升级为可跨轮延续的 v3
- [ ] 知道记忆、消息历史、检查点三者之间的关系与区别

## 前置知识

| 知识点 | 要求 | Wiki 参考 |
|--------|------|-----------|
| 工具回路 | 读懂最小 ReAct 风格图 | [[raw/lessons/LangGraph/04-tools-and-react|第 4 章：工具与 ReAct——让研究助手学会调用检索工具]] |
| `MessagesState` | 知道消息列表如何累积 | [[raw/lessons/LangGraph/03-state-and-messages|第 3 章：State 与 Messages——让研究助手真正进入对话态]] |
| Python `with` | 看过文件或数据库上下文管理语法最好 | — |
| SQLite 基本直觉 | 知道它是轻量本地数据库即可 | — |

---

## 直观理解

### 类比一：messages 像聊天记录，checkpointer 像“会话存档系统”

前一章我们已经有 `messages` 了，所以很多人会自然产生一个误解：

> “既然消息都在列表里，那不就已经有记忆了吗？”

不完全对。

`messages` 只是**当前运行中的上下文内容**。

而 checkpointer 的作用更像是：

- 把这次会话状态真正存起来
- 下次还能接着这条会话继续
- 系统中断后还能恢复
- 人工审批暂停后还能继续往下走

所以更准确地说：

- `messages` = 聊天内容
- checkpointer = 会话持久化机制

### 类比二：`thread_id` 像“会话编号”

如果你有两个不同研究任务：

- 线程 A：研究 LangGraph memory
- 线程 B：研究 RAG 评估

系统必须知道：

- 哪些状态属于 A
- 哪些状态属于 B

`thread_id` 就像一张会话编号卡片，告诉 LangGraph：

> “这次调用属于哪条对话线程。”

---

## 一、为什么“消息历史”还不等于真正的记忆？

### 1.1 当前进程里的上下文，不等于可恢复状态

如果你只是在一次 `invoke()` 调用里维护 `messages`，那么它确实能让图在**这次执行中**看到历史。

但如果你希望：

- 下次调用还能接着上次对话走
- 人工审批时中断，稍后再恢复
- 程序重启后尽量保留工作状态

你就不能只靠“内存里正好还有一个 Python 变量”。

### 1.2 LangGraph 的现代“记忆”主线其实是 checkpoint / persistence

这也是为什么本系列不会把旧 memory 组件当主角。

在现代 LangGraph 语境里，更重要的思路是：

- 图状态如何被 checkpoint
- 会话如何通过 `thread_id` 区分
- 如何在后续调用中恢复同一条线程

所以本章更准确的标题其实应该理解为：

> **记忆 = 持续化状态 + 可恢复执行**

---

## 二、Python 补课：`with` 和 SQLite 的最小使用感

> [!info] Python 补课：为什么这章会出现 `with`？
> 因为很多持久化资源都要遵循“打开 -> 使用 -> 关闭”的模式。
>
> `with` 能帮你把这件事写得更安全。

### 2.1 `with` 的直觉理解

```python
with something as x:
    ...
```

你可以把它理解成：

- 进入上下文时申请资源
- 离开时自动收尾

对 SQLite 连接来说，这种风格尤其自然。

### 2.2 SQLite 在这章不需要学数据库设计

你这章不需要掌握 SQL，也不需要理解复杂建模。

你只需要建立一个直觉：

- `SqliteSaver` 会把 checkpoint 信息保存在本地 SQLite 存储里
- 它比纯内存版更接近“可持久化”的真实使用场景

---

## 三、`MemorySaver`：最小的检查点体验

### 3.1 它是什么？

`MemorySaver` 是一个非常适合教学和本地试验的 checkpointer。

它的特点：

- 使用简单
- 不需要数据库文件
- 适合理解“图如何保存和恢复状态”

### 3.2 但它也不是终点

它更像“课堂实验版”：

- 方便
- 快速
- 适合先理解机制

但如果你想更接近真实项目，就需要进一步看到 `SqliteSaver`。

---

## 四、最小可运行示例：给图接上 `MemorySaver`

> [!example]- 使用 `MemorySaver` 的最小示例（点击展开）
>
> ```python
> from langgraph.checkpoint.memory import MemorySaver
> from langgraph.graph import StateGraph, START, END, MessagesState
> from langchain.messages import HumanMessage, AIMessage
>
>
> class ResearchState(MessagesState):
>     notes: str
>
>
> def draft_notes(state: ResearchState):
>     latest = state["messages"][-1]
>     return {
>         "notes": f"围绕“{latest.content}”优先整理定义、代表论文和应用场景。"
>     }
>
>
> def write_reply(state: ResearchState):
>     return {
>         "messages": [
>             AIMessage(content=f"已记录你的研究问题。建议起步路径：\n{state['notes']}")
>         ]
>     }
>
>
> builder = StateGraph(ResearchState)
> builder.add_node("draft_notes", draft_notes)
> builder.add_node("write_reply", write_reply)
> builder.add_edge(START, "draft_notes")
> builder.add_edge("draft_notes", "write_reply")
> builder.add_edge("write_reply", END)
>
> memory = MemorySaver()
> graph = builder.compile(checkpointer=memory)
>
> config = {"configurable": {"thread_id": "research-thread-1"}}
>
> result = graph.invoke(
>     {
>         "messages": [HumanMessage(content="LangGraph checkpoint 和 memory 的关系是什么？")],
>         "notes": "",
>     },
>     config=config,
> )
>
> print(result["messages"][-1].content)
> ```

### 4.1 这段代码相较前一章真正新增了什么？

只有两个关键变化：

1. `compile(checkpointer=memory)`
2. `invoke(..., config={"configurable": {"thread_id": ...}})`

但就是这两步，把图从“当前运行里有上下文”升级成了“具备线程级持续性能力”。

### 4.2 `thread_id` 为什么必须认真看待？

如果你没有 `thread_id`，系统就难以稳定地区分不同会话。

而有了：

```python
config = {"configurable": {"thread_id": "research-thread-1"}}
```

LangGraph 就知道：

- 当前调用属于哪条线程
- 后续恢复或继续时该去找哪条状态链

---

## 五、Research Copilot v3：真正的跨轮延续感来自哪里？

### 5.1 本章项目升级目标

Research Copilot v3 要获得的核心能力不是“回答更长”，而是：

- 同一线程的多轮调用能延续上下文
- 为后面第 6 章 `interrupt()` 留好恢复基础
- 为第 7 章 `stream()`、第 9 章子图调试提供更稳的状态基础

### 5.2 从产品直觉上看，这意味着什么？

用户第一次问：

- “我想研究 LangGraph memory。”

第二次追问：

- “那它和 checkpoint 的关系是什么？”

如果系统没有线程持续性，它会像第一次见到你一样。

如果有了线程化 checkpoint，它就更像：

> “我记得我们刚才在讨论同一条研究线。”

---

## 六、`SqliteSaver`：从课堂版走向更真实的本地持久化

### 6.1 为什么还要学 `SqliteSaver`？

`MemorySaver` 很适合教学，但它毕竟更偏内存级体验。

`SqliteSaver` 更接近：

- 本地开发项目
- 可落盘保存
- 需要跨进程 / 更稳的本地持久化体验

### 6.2 最小示例

> [!example]- 使用 `SqliteSaver` 的最小示例（点击展开）
>
> ```python
> import sqlite3
> from langgraph.checkpoint.sqlite import SqliteSaver
> from langgraph.graph import StateGraph, START, END, MessagesState
> from langchain.messages import HumanMessage, AIMessage
>
>
> def draft_notes(state: MessagesState):
>     latest = state["messages"][-1]
>     return {
>         "messages": [AIMessage(content=f"我先为问题“{latest.content}”建立研究备忘。")]
>     }
>
>
> builder = StateGraph(MessagesState)
> builder.add_node("draft_notes", draft_notes)
> builder.add_edge(START, "draft_notes")
> builder.add_edge("draft_notes", END)
>
> conn = sqlite3.connect("research-copilot.db")
> checkpointer = SqliteSaver(conn)
> graph = builder.compile(checkpointer=checkpointer)
>
> config = {"configurable": {"thread_id": "thread-langgraph-memory"}}
>
> result = graph.invoke(
>     {
>         "messages": [HumanMessage(content="给我一个 LangGraph memory 的研究起点")]
>     },
>     config=config,
> )
>
> print(result["messages"][-1].content)
> ```

### 6.3 它的学习重点不在数据库，而在“持久化心智模型”

本章不是在教你 SQLite 开发，而是在教你：

- checkpoint 可以不只存在于内存
- 状态可以更稳定地保存到本地存储
- 这对审批流、中断恢复、本地开发都非常重要

---

## 七、状态 / 流程 walkthrough：checkpointer 之后，图发生了什么变化？

### 7.1 图结构可能没怎么变

你会发现很重要的一件事：

- 节点还是那些节点
- 边还是那些边
- messages 还是那条 messages

变化主要发生在“图的运行时能力”上。

### 7.2 之前：只有这次运行能看到上下文

```text
invoke -> 执行 -> 结果返回 -> 结束
```

### 7.3 现在：运行时状态有了线程级归属

```text
thread_id = research-thread-1
       │
       ▼
本次图执行状态被 checkpoint
       │
       ▼
下一次同 thread_id 调用时可以延续
```

### 7.4 这对后面章节的意义

- 第 6 章：`interrupt()` 暂停后要恢复，离不开 checkpointer
- 第 7 章：流式输出时更适合观察带 checkpoint 的真实执行
- 第 9 章：子图组合时，调试状态会更有意义

---

## 八、消息历史、记忆、检查点三者到底怎么区分？

这是本章最容易混淆的地方。

### 8.1 一张对比表

| 概念 | 它是什么 | 主要解决什么问题 |
|------|---------|-----------------|
| `messages` | 当前状态中的消息通道 | 让对话上下文能在图内流动 |
| 记忆（本课程语境） | 可延续的会话状态能力 | 让系统不是每次都从零开始 |
| checkpointer | 保存 / 恢复状态的机制 | 让线程状态可持续、可恢复 |

### 8.2 更准确的课程表述

所以本系列在现代语境下，更建议你这样理解：

> 对话上下文由 `messages` 承载，
> 会话持续性由 checkpointer 实现，
> “记忆能力”是二者在真实系统中的综合结果。

---

## 九、常见报错与调试

### 9.1 忘了传 `thread_id`

如果你已经接了 checkpointer，却没有传：

```python
config = {"configurable": {"thread_id": "..."}}
```

那你通常就得不到稳定的线程级延续体验。

### 9.2 把“同一线程”和“不同任务”混在一个 `thread_id`

如果你把完全不同的研究任务都塞进同一个 `thread_id`，系统状态就会混杂。

所以要养成习惯：

- 同一会话线索 → 同一 `thread_id`
- 不同会话目标 → 不同 `thread_id`

### 9.3 误以为 `MemorySaver` 就是永久存储

它更适合理解机制和本地实验。

如果你需要更接近“可落盘”的体验，应进一步看 `SqliteSaver`。

### 9.4 混淆“messages 变长”和“跨轮持久化”

消息变长只能说明：

- 在当前状态里，上下文在累积

但不自动说明：

- 这份状态已经稳稳保存在可恢复系统里

### 9.5 调试建议：先打印 thread_id 和最后一条消息

每次调试跨轮问题，优先确认：

- 当前使用的 `thread_id` 是什么
- 最后一条消息是谁发的
- 是否是你期望的那条线程

---

## 十、动手练习

### 练习 1：同一个图，跑两个不同 `thread_id`

要求：

- 线程 A 研究 LangGraph memory
- 线程 B 研究 LangGraph streaming
- 分别调用图
- 比较它们的上下文是否互相独立

### 练习 2：把 `MemorySaver` 版本改成 `SqliteSaver` 版本

要求：

- 保持节点逻辑不变
- 只替换 checkpointer
- 自己总结：图结构有没有变？变化主要发生在哪里？

### 练习 3：模拟“继续同一条研究线”

要求：

- 第一次调用：问“LangGraph memory 是什么？”
- 第二次调用：同一个 `thread_id`，继续问“它和 checkpoint 有什么关系？”
- 观察系统如何延续同一条线索

---

## 常见误区

| 误区 | 正确理解 |
|------|----------|
| “只要有 `messages` 就已经有长期记忆。” | 不够。`messages` 只是状态内容，真正的持续性要靠 checkpointer。 |
| “本章讲的 memory 就是旧式 memory 组件。” | 不是。本章主线是现代 LangGraph 的 checkpoint / persistence 思维。 |
| “`thread_id` 只是一个随便写写的参数。” | 错。它是线程级状态归属的关键标识。 |
| “接了 checkpointer 后图结构会完全变样。” | 不一定。很多时候节点和边几乎不变，变化主要在运行时能力层。 |
| “`SqliteSaver` 这章是在学数据库。” | 不对。本章重点仍然是状态持久化心智模型，而不是 SQL 技巧。 |

---

## 思考题

### 基础理解

**Q1. 为什么“消息历史”还不等于真正的跨轮记忆？**

> [!hint]- 💡 提示
> 想想程序结束后、暂停后、下一次调用时会发生什么。

> [!success]- ✅ 参考答案
> 因为消息历史只是当前状态里的上下文内容，并不自动保证这些状态会在线程级别被保存、恢复或延续。真正的跨轮记忆需要有 checkpoint / persistence 机制，让系统知道这条会话属于谁、该从哪里继续。

**Q2. `thread_id` 在本章扮演什么角色？**

> [!hint]- 💡 提示
> 它是不是像会话编号？

> [!success]- ✅ 参考答案
> `thread_id` 就像一张会话编号卡，用来区分不同对话线程。LangGraph 会利用它判断当前执行属于哪条状态链，从而实现同一线程的继续执行和不同线程的隔离。

### 深入思考

**Q3. 为什么说本章的核心升级主要发生在“运行时能力”而不是“图结构”上？**

> [!hint]- 💡 提示
> 节点和边是不是可能几乎不变？

> [!success]- ✅ 参考答案
> 因为接入 checkpointer 后，很多时候节点函数和边连接都可以保持不变。变化主要在于：图的状态现在可以被保存、恢复、按线程区分、支持暂停后继续执行。所以这是运行时层面的能力跃迁，而不是图形表面结构的大重写。

**Q4. 为什么本系列把“记忆”主线讲成 checkpoint / persistence，而不是旧 memory 组件？**

> [!hint]- 💡 提示
> 想想现代 LangGraph 最强调什么。

> [!success]- ✅ 参考答案
> 因为在现代 LangGraph 中，更重要的是图状态如何被持续保存、恢复和关联到线程，而不是把 memory 理解成一个额外外挂对象。checkpoint / persistence 更贴近 LangGraph 真正的运行机制，也更适合后续的人在回路、中断恢复和本地开发场景。

---

## 关键要点回顾

- ✅ `messages` 负责承载对话上下文，但不自动等于跨轮持久化
- ✅ checkpointer 是现代 LangGraph“记忆能力”的关键机制之一
- ✅ `MemorySaver` 适合教学与快速体验，`SqliteSaver` 更接近真实本地持久化
- ✅ `thread_id` 是线程级状态归属的核心标识
- ✅ Research Copilot v3 的升级重点不是“更会回答”，而是“更会延续同一条工作线”

---

## 扩展阅读

- 📄 [Add Memory](https://docs.langchain.com/oss/python/langgraph/add-memory) — 进一步看消息状态与记忆如何结合
- 📄 [Thinking in LangGraph](https://docs.langchain.com/oss/python/langgraph/thinking-in-langgraph) — 理解为什么持久化是图式系统的重要部分
- 📄 [Interrupts](https://docs.langchain.com/oss/python/langgraph/interrupts) — 提前预习下一章中断恢复为什么离不开 checkpointer

## 相关页面

- [[raw/lessons/LangGraph/04-tools-and-react|第 4 章：工具与 ReAct——让研究助手学会调用检索工具]]
- [[raw/lessons/LangGraph/06-human-in-the-loop|第 6 章：人在回路——让研究助手在关键步骤停下来等你审核]]
- [[wiki/concepts/langgraph-api-map|LangGraph API 学习地图]]

## 下一步学习

下一章我们会真正让“人”进入系统：

- 在关键步骤调用 `interrupt()` 暂停
- 用 `Command(resume=...)` 恢复执行
- 让 Research Copilot 在高风险输出前先等你审核

继续阅读：[[raw/lessons/LangGraph/06-human-in-the-loop|第 6 章：人在回路——让研究助手在关键步骤停下来等你审核]]
