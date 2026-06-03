---
type: lesson
tags:
  - LangGraph
  - MessagesState
  - Python
  - Agent
  - 工作流
created: 2026-04-28
updated: 2026-04-28
topic: State 与 Messages——让研究助手真正进入对话态
difficulty: intermediate
prerequisites:
  - "[[raw/lessons/LangGraph/02-stategraph-basics|第 2 章：StateGraph、节点与状态——让研究助手先跑起来]]"
sources:
  - https://docs.langchain.com/oss/python/langgraph/overview
  - https://docs.langchain.com/oss/python/langgraph/graph-api
  - https://docs.langchain.com/oss/python/langgraph/add-memory
status: completed
series:
  name: LangGraph 系统学习
  part: 3
---

# State 与 Messages——让研究助手真正进入对话态

> 一句话摘要：第 2 章的 Research Copilot v0 只能处理一次性输入，本章会把它升级成真正的“对话态”系统。你将学会 `MessagesState`、`add_messages`、消息 reducer，以及为什么消息状态不能只靠普通列表随手拼接。

## 学习目标

完成本课后，你应该能够：
- [ ] 理解为什么对话型 agent 需要消息状态，而不是只存一个 `question`
- [ ] 会使用 `MessagesState` 或 `Annotated[..., add_messages]` 定义消息通道
- [ ] 理解 reducer 在消息追加中的作用
- [ ] 能写出一个支持多轮对话的 Research Copilot v1
- [ ] 能看懂消息列表在每个节点后是如何增长的
- [ ] 遇到消息相关报错时，知道先检查消息类型、返回值结构和 reducer 定义

## 前置知识

| 知识点 | 要求 | Wiki 参考 |
|--------|------|-----------|
| `StateGraph` | 能读懂最小线性图 | [[raw/lessons/LangGraph/02-stategraph-basics|第 2 章：StateGraph、节点与状态——让研究助手先跑起来]] |
| `TypedDict` | 知道如何定义状态字段 | [[raw/lessons/LangGraph/02-stategraph-basics|第 2 章：StateGraph、节点与状态——让研究助手先跑起来]] |
| Python `list` | 知道列表会按顺序保存多个元素 | — |
| 多轮对话直觉 | 知道聊天不是一次性输入输出 | — |

---

## 直观理解

### 类比一：`MessagesState` 像“聊天记录本”

第 2 章的最小 state 更像一张一次性任务卡：

- 用户提一个问题
- 系统做 3 步
- 输出一个答案

但真正的对话型助手不是这样。它更像一段不断增长的聊天记录：

- 用户先问第一个问题
- 系统回答
- 用户继续追问
- 系统需要记得刚才聊到哪里

这时，只存一个 `question` 字段已经不够了。

你真正需要的是一个持续累积的：

> **messages 列表**

### 类比二：reducer 像“自动追加规则”

假设你有一本聊天记录本。新的一句话来了，你希望发生什么？

- 不是把整本记录本擦掉重写
- 而是把新内容**追加**到原来的记录后面

`add_messages` 做的正是这件事：

> 当节点返回新的消息时，不替换旧消息，而是按 LangGraph 约定把消息正确追加进去。

---

## 一、为什么第 2 章的 state 到这里就不够了？

### 1.1 一次性工作流 vs 对话型工作流

第 2 章的 state 只有：

- `question`
- `notes`
- `answer`

它很适合“一次性处理一个任务”。但一进入对话场景，很快就会遇到问题：

- 用户问完第一句后又追问怎么办？
- 系统回答后，这条回答存在哪里？
- 工具调用后返回的结果如何进入上下文？
- 后面模型节点如何看到完整聊天历史？

### 1.2 为什么不能只手动维护一个 `list[str]`

你当然可以写一个普通列表，但消息状态不仅仅是“几段字符串”：

- 用户消息和 AI 消息角色不同
- 工具消息和普通回答也不同
- 后面工具调用需要更规范的消息结构
- LangGraph / LangChain 生态已经有一套消息对象约定

所以本系列从现在开始，不再只把“对话”当作字符串拼接，而是转入更标准的消息状态思维。

---

## 二、Python 补课：`Annotated` 与 reducer 思维

> [!info] Python 补课：`Annotated` 是什么？
> `Annotated[T, extra]` 的意思是：
> - 数据主类型还是 `T`
> - 但我们额外给它附加一层“解释”或“元信息”
>
> 在 LangGraph 里，这个 `extra` 经常就是 reducer。

### 2.1 自己定义 messages 字段时的标准写法

官方 Graph API 常见写法是：

```python
from typing import Annotated
from typing_extensions import TypedDict
from langchain.messages import AnyMessage
from langgraph.graph.message import add_messages

class ResearchChatState(TypedDict):
    messages: Annotated[list[AnyMessage], add_messages]
    notes: str
```

这里的重点是：

- `messages` 是一个消息列表
- `add_messages` 告诉 LangGraph：这个字段更新时用“追加消息”的规则处理

### 2.2 为什么 reducer 很重要？

如果没有 reducer，节点每次返回：

```python
{"messages": [new_message]}
```

LangGraph 很可能会把它理解成“新的 `messages` 值”，而不是“请把这条新消息加到原列表后面”。

而有了 `add_messages`，语义就清晰了：

> **这不是替换，而是消息通道上的追加更新。**

### 2.3 更简单的现成版本：`MessagesState`

如果你只需要一个标准消息通道，LangGraph 已经提供了现成状态类型：

```python
from langgraph.graph import MessagesState
```

它本质上就是一个内建的消息状态定义，已经帮你把 `messages` 通道和 reducer 准备好了。

### 2.4 扩展 `MessagesState`

你通常不会只要消息，还会想保留自己的额外字段，比如：

- `notes`
- `research_focus`
- `draft_answer`

这时可以直接继承：

```python
from langgraph.graph import MessagesState

class ResearchState(MessagesState):
    notes: str
    research_focus: str
```

> [!tip] 这一章最推荐的新手路径
> - 如果你想彻底理解 reducer：先看 `Annotated[..., add_messages]`
> - 如果你想快速进入实作：直接用 `MessagesState` 扩展

---

## 三、Research Copilot v1：从“一次性流程”升级到“对话态助手”

### 3.1 本章升级目标

Research Copilot v0 只有一次输入和一次输出。

Research Copilot v1 的目标是：

- 接收消息列表而不是单个 `question`
- 让系统回答也进入消息列表
- 保留一个可持续增长的对话上下文
- 为第 4 章工具调用打好消息地基

### 3.2 这一章的核心设计变化

从：

```text
question -> notes -> answer
```

升级到：

```text
messages -> 读取最后一条用户问题 -> 生成研究笔记 -> 追加 AI 回复
```

这意味着我们开始进入 LangGraph 更典型的 agent 状态模式。

---

## 四、最小可运行示例：带 `MessagesState` 的 Research Copilot v1

> [!example]- 最小可运行版本（点击展开）
>
> ```python
> from langgraph.graph import StateGraph, START, END, MessagesState
> from langchain.messages import HumanMessage, AIMessage
>
>
> class ResearchState(MessagesState):
>     notes: str
>
>
> def understand_latest_question(state: ResearchState):
>     latest_message = state["messages"][-1]
>     return {
>         "notes": f"围绕问题“{latest_message.content}”先整理定义、代表论文和应用场景。"
>     }
>
>
> def write_research_reply(state: ResearchState):
>     latest_message = state["messages"][-1]
>     reply = AIMessage(
>         content=(
>             f"你刚才问的是：{latest_message.content}\n\n"
>             f"我的建议起步路径：\n{state['notes']}\n\n"
>             "建议先从概念定义入手，再进入代表性论文阅读。"
>         )
>     )
>     return {"messages": [reply]}
>
>
> builder = StateGraph(ResearchState)
> builder.add_node("understand_latest_question", understand_latest_question)
> builder.add_node("write_research_reply", write_research_reply)
>
> builder.add_edge(START, "understand_latest_question")
> builder.add_edge("understand_latest_question", "write_research_reply")
> builder.add_edge("write_research_reply", END)
>
> graph = builder.compile()
>
> result = graph.invoke(
>     {
>         "messages": [HumanMessage(content="LangGraph 中的 memory 和 checkpoint 有什么区别？")],
>         "notes": "",
>     }
> )
>
> print(result["messages"][-1].content)
> ```

### 4.1 这段代码比第 2 章多了什么？

主要多了 3 个变化：

1. state 改成基于 `MessagesState`
2. 输入变成 `messages`
3. 输出不再只是 `answer` 字符串，而是把 AI 回复追加到消息列表

### 4.2 为什么 `return {"messages": [reply]}` 外面是列表？

因为消息通道是一个列表，而不是单条消息字段。

即使你本次只新增一条 AI 回复，也应该以“新增消息列表”的形式返回：

```python
{"messages": [reply]}
```

然后让 `add_messages` / `MessagesState` 的 reducer 帮你处理追加逻辑。

### 4.3 为什么读最后一条消息时写 `state["messages"][-1]`

因为这一章先采用最简单的对话场景：

- 用户刚发来一条消息
- 系统读取最新输入
- 生成回应

后面到工具调用和多轮记忆时，消息列表会更丰富，但“最新消息”仍然是常见切入点。

---

## 五、如果你想更彻底理解：自己写 `add_messages` 版本

有些同学会担心：

> “我如果只会继承 `MessagesState`，是不是还不够懂底层？”

那可以看这个更显式的版本。

> [!example]- `Annotated + add_messages` 版本（点击展开）
>
> ```python
> from typing import Annotated
> from typing_extensions import TypedDict
> from langchain.messages import AnyMessage, HumanMessage, AIMessage
> from langgraph.graph import StateGraph, START, END
> from langgraph.graph.message import add_messages
>
>
> class ResearchState(TypedDict):
>     messages: Annotated[list[AnyMessage], add_messages]
>     notes: str
>
>
> def draft_notes(state: ResearchState):
>     last_user_message = state["messages"][-1]
>     return {
>         "notes": f"请围绕“{last_user_message.content}”总结定义、代表论文和研究问题。"
>     }
>
>
> def answer(state: ResearchState):
>     return {
>         "messages": [
>             AIMessage(content=f"好的，我已经为这个问题整理了起步路径：\n{state['notes']}")
>         ]
>     }
>
>
> builder = StateGraph(ResearchState)
> builder.add_node("draft_notes", draft_notes)
> builder.add_node("answer", answer)
> builder.add_edge(START, "draft_notes")
> builder.add_edge("draft_notes", "answer")
> builder.add_edge("answer", END)
> graph = builder.compile()
> ```

### 5.1 `MessagesState` 与 `add_messages` 的关系

你可以这样记：

- `add_messages` = 消息字段的更新规则
- `MessagesState` = 内建好的常用消息状态模板

所以：

> `MessagesState` 不是另一套神秘机制，而是把消息通道这套常见写法预封装好了。

---

## 六、Research Copilot v1 的状态流转 walkthrough

### 6.1 初始状态

```python
{
    "messages": [HumanMessage(content="LangGraph 中的 memory 和 checkpoint 有什么区别？")],
    "notes": ""
}
```

### 6.2 经过 `understand_latest_question`

```python
{
    "messages": [HumanMessage(content="LangGraph 中的 memory 和 checkpoint 有什么区别？")],
    "notes": "围绕问题“LangGraph 中的 memory 和 checkpoint 有什么区别？”先整理定义、代表论文和应用场景。"
}
```

这里变化的是：

- `messages` 暂时不变
- `notes` 获得了中间分析结果

### 6.3 经过 `write_research_reply`

```python
{
    "messages": [
        HumanMessage(content="LangGraph 中的 memory 和 checkpoint 有什么区别？"),
        AIMessage(content="你刚才问的是：...")
    ],
    "notes": "围绕问题..."
}
```

这里发生了本章最关键的事情：

- 新的 AI 回复没有覆盖原来的用户消息
- 而是被追加到了消息列表尾部

### 6.4 用 ASCII 图看消息增长

```text
初始 messages:
[HumanMessage("问题 A")]
        │
        ▼
节点 1：提炼笔记
messages 不变，notes 更新
        │
        ▼
节点 2：生成回复
messages 追加 AIMessage
        │
        ▼
最终 messages:
[HumanMessage("问题 A"), AIMessage("回答 A")]
```

> [!important] 本章最重要的新直觉
> 第 2 章是“字段型 state 思维”，第 3 章开始进入“通道型 state 思维”。
>
> 也就是说：
> - 有些字段像 `notes`，是普通状态字段
> - 有些字段像 `messages`，会持续增长并通过 reducer 更新

---

## 七、多轮对话会如何延续？

本章示例虽然只跑了一轮，但它已经具备多轮对话的基础：

- 你保留了完整 `messages`
- 新消息会继续追加
- 下次再进图时，系统就能看到更长的上下文

你现在可以把它理解成：

- 第 2 章：一次性任务卡
- 第 3 章：开始有聊天记录

第 5 章加入 checkpointer 后，这个对话态才真正跨轮稳定记住。

---

## 八、常见报错与调试

### 8.1 报错一：忘了把新消息放进列表

错误写法：

```python
return {"messages": AIMessage(content="hello")}
```

更稳的写法：

```python
return {"messages": [AIMessage(content="hello")]}
```

原因：

- `messages` 通道是列表
- reducer 期待的是“新增消息序列”

### 8.2 报错二：节点把 `messages` 当成普通字符串列表

如果你后面要接 LangChain / LangGraph 的消息生态，最好尽早习惯使用消息对象，而不是自己随手拼：

- `HumanMessage(...)`
- `AIMessage(...)`

这样后面工具调用、模型绑定、消息流式输出都会更自然。

### 8.3 报错三：忘了 reducer，结果消息被覆盖

如果你用自定义 `TypedDict` 写 `messages` 字段，却没写：

```python
Annotated[list[AnyMessage], add_messages]
```

就很可能发生“更新时替换而不是追加”。

### 8.4 报错四：读取最后一条消息时假设过强

下面这种写法：

```python
latest_message = state["messages"][-1]
```

在本章最小示例里没有问题，但到后面如果最后一条是工具消息、AI 消息，就要更仔细判断角色。

这也是为什么本章只先讲最基本版本。

### 8.5 调试建议：先打印消息长度和最后一条类型

当你怀疑消息状态出了问题，先做最简单的检查：

```python
print(len(state["messages"]))
print(type(state["messages"][-1]))
```

你会立刻知道：

- 消息有没有被正确追加
- 最后一条到底是谁发的

---

## 九、动手练习

### 练习 1：为 state 增加 `research_focus`

要求：

- 继承 `MessagesState`
- 新增字段 `research_focus: str`
- 在第一个节点里根据用户问题生成一个更聚焦的研究方向描述
- 在 AI 回复里展示它

你会练到：

- 扩展内建状态
- 让普通字段与消息字段共存

### 练习 2：支持第二轮追问

要求：

- 先运行一次图，得到第一轮 AI 回复
- 再在 `messages` 末尾追加一个新的 `HumanMessage`
- 再次调用图，观察系统如何基于更长消息列表生成第二轮回复

你会练到：

- 多轮对话的真实消息增长
- 为什么消息历史是 agent 的第一层记忆

### 练习 3：自己写一个 `Annotated + add_messages` 版本

不要只用 `MessagesState`，尝试把：

- `messages`
- `notes`
- `research_focus`

都手动写进 `TypedDict`。

你会练到：

- reducer 的底层写法
- `MessagesState` 究竟帮你省了什么

---

## 常见误区

| 误区 | 正确理解 |
|------|----------|
| “有了 `MessagesState`，就不需要普通字段了。” | 错。消息字段和普通业务字段通常要一起存在，比如 `notes`、`summary`、`research_focus`。 |
| “消息就是字符串列表。” | 在现代 LangGraph / LangChain 生态里，更推荐使用标准消息对象。 |
| “返回 `{"messages": new_message}` 也差不多。” | 不稳。更标准的写法是返回消息列表，让 reducer 负责追加。 |
| “`MessagesState` 和 `add_messages` 是两套互不相关的东西。” | 不是。`MessagesState` 可以理解为内建好的消息状态模板，而 `add_messages` 是它背后的关键更新规则之一。 |
| “本章学了消息状态，就等于已经有长期记忆。” | 还不够。消息列表是对话上下文，真正跨轮稳定保存要到第 5 章的 checkpointer。 |

---

## 思考题

### 基础理解

**Q1. 为什么对话型 Research Copilot 不再只存一个 `question` 字段？**

> [!hint]- 💡 提示
> 想想用户第二轮追问时，系统需要知道什么。

> [!success]- ✅ 参考答案
> 因为对话型助手需要保留完整上下文。只存一个 `question` 只能表达当前问题，无法自然表示“用户问过什么、系统答过什么、当前是第几轮、最新回复是什么”。消息列表更适合承载这种持续增长的上下文。

**Q2. `add_messages` 在解决什么问题？**

> [!hint]- 💡 提示
> 新消息来了，是替换历史还是追加历史？

> [!success]- ✅ 参考答案
> `add_messages` 解决的是消息通道的更新语义问题。它告诉 LangGraph：当节点返回新的消息时，不是把旧消息全替换掉，而是按标准消息规则把新消息追加到已有历史中。

### 深入思考

**Q3. 为什么说第 3 章是从“字段型 state”升级到“通道型 state”？**

> [!hint]- 💡 提示
> 对比 `notes` 和 `messages` 的更新方式。

> [!success]- ✅ 参考答案
> 因为像 `notes` 这样的字段通常表示某个当前值，而 `messages` 这种字段则更像一条会持续增长的状态通道。它不是简单替换，而是持续追加，并依赖 reducer 来定义更新规则。所以本章不只是多了一个字段，而是多了一种新的状态组织方式。

**Q4. `MessagesState` 为什么值得在这一章单独学？**

> [!hint]- 💡 提示
> 想想后面第 4-7 章都会建立在什么基础上。

> [!success]- ✅ 参考答案
> 因为后面的工具调用、ReAct 循环、流式输出、多轮记忆、人工审核，几乎都离不开标准消息状态。如果没有 `MessagesState` 和消息 reducer 的心智模型，后面你会一直把 agent 代码理解成“几段字符串来回拼接”，这会严重限制你对 LangGraph 的理解深度。

---

## 关键要点回顾

- ✅ 第 2 章的最小 state 更适合一次性流程，第 3 章开始进入对话型 agent 的标准状态模式
- ✅ `MessagesState` 是内建好的消息状态模板，适合快速进入实作
- ✅ `Annotated[list[AnyMessage], add_messages]` 能帮助你理解消息 reducer 的底层机制
- ✅ 消息状态的关键不是“存个列表”，而是“定义消息如何累积”
- ✅ Research Copilot v1 已经从一次性任务卡升级成真正有聊天记录的系统

---

## 扩展阅读

- 📄 [LangGraph Overview](https://docs.langchain.com/oss/python/langgraph/overview) — 看官方最小消息态示例
- 📄 [Graph API](https://docs.langchain.com/oss/python/langgraph/graph-api) — 查 `MessagesState`、`add_messages` 的标准写法
- 📄 [Add Memory](https://docs.langchain.com/oss/python/langgraph/add-memory) — 提前看看消息状态如何与记忆一起工作

## 相关页面

- [[raw/lessons/LangGraph/02-stategraph-basics|第 2 章：StateGraph、节点与状态——让研究助手先跑起来]]
- [[raw/lessons/LangGraph/04-tools-and-react|第 4 章：工具与 ReAct——让研究助手学会调用检索工具]]
- [[wiki/concepts/state-graph|StateGraph]]
- [[wiki/concepts/langgraph-api-map|LangGraph API 学习地图]]

## 下一步学习

下一章我们会让 Research Copilot 真正“动起来”：

- 定义工具
- 用 `ToolNode` 执行工具
- 用 `tools_condition` 决定是否进入工具调用
- 再对比高层 helper `create_react_agent`

继续阅读：[[raw/lessons/LangGraph/04-tools-and-react|第 4 章：工具与 ReAct——让研究助手学会调用检索工具]]
