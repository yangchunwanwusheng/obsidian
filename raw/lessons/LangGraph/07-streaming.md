---
type: lesson
tags:
  - LangGraph
  - stream
  - Streaming
  - Python
  - Agent
created: 2026-04-28
updated: 2026-04-28
topic: 流式执行——实时看见研究助手正在如何思考与推进
difficulty: intermediate
prerequisites:
  - "[[raw/lessons/LangGraph/06-human-in-the-loop|第 6 章：人在回路——让研究助手在关键步骤停下来等你审核]]"
sources:
  - https://docs.langchain.com/oss/python/langgraph/streaming
  - https://docs.langchain.com/oss/python/langgraph/quickstart
status: completed
series:
  name: LangGraph 系统学习
  part: 7
---
 
# 流式执行——实时看见研究助手正在如何思考与推进

> 一句话摘要：前几章的图虽然越来越强，但你大多只能等 `invoke()` 最终返回。本章会让 Research Copilot 升级为支持流式可视化的 v5：你将掌握 `graph.stream()`、`stream_mode`、`version="v2"` 的基础心智模型，并学会把“图是怎么一步步推进的”直接展示出来。

## 学习目标

完成本课后，你应该能够：
- [ ] 理解为什么 `invoke()` 不足以支撑良好的 agent 用户体验
- [ ] 会用 `graph.stream()` 观察图执行过程
- [ ] 了解 `updates`、`values`、`messages`、`custom` 等常见 `stream_mode`
- [ ] 知道为什么官方推荐在现代用法中显式写 `version="v2"`
- [ ] 能把 Research Copilot 升级成支持流式展示过程的 v5
- [ ] 会调试“为什么没看到流输出”这类问题

## 前置知识

| 知识点 | 要求 | Wiki 参考 |
|--------|------|-----------|
| 工具回路 | 理解 `llm_call -> tool_node -> llm_call` | [[raw/lessons/LangGraph/04-tools-and-react|第 4 章：工具与 ReAct——让研究助手学会调用检索工具]] |
| checkpointer / 线程 | 知道线程状态可持续 | [[raw/lessons/LangGraph/05-checkpointer-memory|第 5 章：记忆与检查点——让研究助手跨轮记住上下文]] |
| 人工审核流 | 知道图可以暂停恢复 | [[raw/lessons/LangGraph/06-human-in-the-loop|第 6 章：人在回路——让研究助手在关键步骤停下来等你审核]] |
| Python `for` 循环 | 能读懂迭代输出 | — |

---

## 直观理解

### 类比一：`invoke()` 像“最后交卷”，`stream()` 像“实时看过程”

如果一个研究助理最后只给你一份结论，你当然能知道结果。

但你看不到：

- 他先做了什么
- 哪一步卡住了
- 有没有走工具
- 哪个节点更新了什么状态

而 `stream()` 就像把整个工作过程透明化：

- 先更新了哪部分 state
- 当前在执行哪个节点
- 模型 token 正在如何输出
- 工具结果何时返回

### 类比二：不同 `stream_mode` 像不同摄像机机位

你可以把 `stream_mode` 想成不同机位：

- `updates`：只拍“这一步新改了什么”
- `values`：每一步都给你完整状态快照
- `messages`：拍 LLM token 实时输出
- `custom`：拍你自己想暴露的业务进度信息

不是每次都要全开，关键是知道你现在想看什么。

---

## 一、为什么 `invoke()` 不够了？

到第 6 章为止，我们主要使用的是：

```python
graph.invoke(...)
```

它当然很重要，但它有一个明显限制：

> 你往往只能在结束后看到最终结果。

对于简单图，这没问题。

但当图开始拥有：

- 工具调用
- 多步消息回路
- 人工审核
- 并行分发
- 子图

你会越来越想看到：

- 系统当前在做什么
- 状态是怎么变化的
- 为什么结果是这样得出来的

所以 `stream()` 本质上解决的是：

> **agent 的可观测性与交互体验问题。**

---

## 二、Python 补课：迭代器 / 生成器直觉为什么在这章突然重要了？

> [!info] Python 补课：为什么 `stream()` 不是直接返回一个最终对象？
> 因为“流式输出”本身的语义就是：
> - 不是等全部结束再一次性给你
> - 而是边执行边不断产出一段段内容

这意味着你要开始适应：

```python
for chunk in graph.stream(...):
    ...
```

### 2.1 最低限度理解就够了

你现在不需要去学完整生成器理论。

只要记住：

- `invoke()` 更像“返回最终结果”
- `stream()` 更像“不断吐出中间片段”

### 2.2 这章真正要学的是“怎么读流”

也就是说：

- 每个 `chunk` 是什么
- 用哪个 `stream_mode` 会看到什么
- 如何按 `chunk["type"]` 解读内容

---

## 三、最小可运行示例：先看 `updates`

> [!example]- 用 `stream_mode="updates"` 观察状态更新（点击展开）
>
> ```python
> from typing_extensions import TypedDict
> from langgraph.graph import StateGraph, START, END
>
>
> class State(TypedDict):
>     topic: str
>     notes: str
>     answer: str
>
>
> def refine_topic(state: State):
>     return {"topic": state["topic"] + "（研究入门版）"}
>
>
> def draft_notes(state: State):
>     return {"notes": f"围绕 {state['topic']} 先整理定义、论文和应用场景。"}
>
>
> def write_answer(state: State):
>     return {"answer": f"关于 {state['topic']}，建议先从：{state['notes']} 开始。"}
>
>
> graph = (
>     StateGraph(State)
>     .add_node(refine_topic)
>     .add_node(draft_notes)
>     .add_node(write_answer)
>     .add_edge(START, "refine_topic")
>     .add_edge("refine_topic", "draft_notes")
>     .add_edge("draft_notes", "write_answer")
>     .add_edge("write_answer", END)
>     .compile()
> )
>
> for chunk in graph.stream(
>     {"topic": "LangGraph memory", "notes": "", "answer": ""},
>     stream_mode="updates",
>     version="v2",
> ):
>     if chunk["type"] == "updates":
>         for node_name, state_update in chunk["data"].items():
>             print(node_name, state_update)
> ```

### 3.1 为什么先从 `updates` 开始？

因为它最适合建立“状态在变什么”的直觉。

你看到的不是全量状态，而是：

- 哪个节点刚更新了什么字段

这对初学者非常友好。

---

## 四、`version="v2"` 是什么，为什么本系列显式写它？

官方文档强调，现代流式格式更推荐显式写：

```python
version="v2"
```

### 4.1 它带来的好处

在 v2 输出格式中，每个 chunk 都更统一，典型形状类似：

```python
{
    "type": "updates" | "values" | "messages" | "custom" | ...,
    "ns": (),
    "data": ...
}
```

这意味着：

- 单模式、多模式、子图流输出更统一
- 你更容易按 `chunk["type"]` 分支处理
- 教学上更稳定，不容易因为输出形态变化而混乱

### 4.2 本系列为什么要显式写出来

因为这门课的目标不是“凑巧跑通”，而是建立稳定心智模型。

既然 v2 格式更统一，那就应该从现在开始把它当成推荐写法。

---

## 五、常见 `stream_mode` 到底分别看什么？

### 5.1 `updates`

只看每一步的**状态增量**。

适合：

- 学习图执行
- 调试哪个节点改了什么
- 避免被全量状态刷屏

### 5.2 `values`

看每一步后的**完整状态快照**。

适合：

- 你想看全量 state 最终如何一步步长大
- 状态字段不多，便于全程观察

### 5.3 `messages`

看 LLM 的 token / 消息流。

适合：

- 用户界面要实时展示模型输出
- 你想知道模型何时开始真正说话

### 5.4 `custom`

看你自己在节点里定义的业务进度信息。

适合：

- “正在检索论文...”
- “正在汇总知识库...”
- “正在生成综述草稿...”

这类比纯技术状态更贴近用户体验的提示。

---

## 六、Research Copilot v5：让过程可见，而不只是结果可见

### 6.1 本章项目升级目标

Research Copilot v5 的核心升级不是多一个能力点，而是：

> 把已有能力的执行过程透明化。

也就是说：

- 用户能看到图现在推进到哪了
- 开发者能看到哪个节点刚更新了什么
- 后面做并行和子图时，可观测性不至于崩掉

### 6.2 这对真实产品体验为什么很重要？

因为用户对长任务最怕的是：

- 没反应
- 不知道卡住没
- 不知道系统在忙什么

流式过程展示能显著降低这种“黑盒焦虑”。

---

## 七、Research Copilot v5：同时观察 `updates` 和 `messages`

> [!example]- 多模式流式观察（点击展开）
>
> ```python
> for part in graph.stream(
>     inputs,
>     stream_mode=["updates", "messages"],
>     version="v2",
> ):
>     if part["type"] == "updates":
>         for node_name, state_update in part["data"].items():
>             print(f"[update] {node_name}: {state_update}")
>     elif part["type"] == "messages":
>         msg, metadata = part["data"]
>         if msg.content:
>             print(msg.content, end="", flush=True)
> ```

### 7.1 为什么多模式很有价值？

因为你经常会同时想看两种东西：

- 系统内部状态更新
- LLM 面向用户的文本输出

这会让“图如何推进”和“用户看到什么”同时可见。

---

## 八、`values` 与 `updates` 应该怎么选？

### 8.1 经验法则

- 学入门：优先 `updates`
- 看全貌：用 `values`
- 状态很大时：优先 `updates`
- 想做完整调试回放：`values` 更直观

### 8.2 一个简单对比

假设当前节点只更新：

```python
{"notes": "..."}
```

那么：

- `updates` 只告诉你 `notes` 改了什么
- `values` 会把 `topic`、`notes`、`answer` 全部状态都带上

如果状态很大，`values` 会更啰嗦；如果你想看整体快照，它又更完整。

---

## 九、`messages` 模式到底在流什么？

很多同学会误解：

> “只有我显式对模型调用 `.stream()` 才能流消息吗？”

官方文档强调的一点是：

- 即使节点里对模型用的是普通 `.invoke()`
- 在 `graph.stream(..., stream_mode="messages", version="v2")` 下，依然可以收到 LLM 相关消息流事件

这让你更容易在图级别统一观察模型输出。

---

## 十、`custom` 模式为什么对产品体验特别有价值？

如果你只给用户看底层状态字段，很多时候并不友好。

例如用户其实不关心：

- 哪个字段叫 `notes`
- 哪个节点名叫 `retrieve_docs`

用户更关心的是：

- 正在检索资料
- 正在整理证据
- 正在生成结论

`custom` 模式就是桥梁：

> 你可以把技术执行过程翻译成更可读的业务进度事件。

本系列后面如果你想做更真实的前端体验，这会非常有用。

---

## 十一、状态 / 流程 walkthrough：流式输出和普通调用到底差在哪里？

### 11.1 `invoke()` 心智模型

```text
输入 -> 整个图跑完 -> 返回最终结果
```

### 11.2 `stream()` 心智模型

```text
输入 -> 节点 1 更新 -> 节点 2 更新 -> token 输出 -> 节点 3 更新 -> ... -> 结束
```

你得到的不是“最后一块砖”，而是整个建房子的过程。

### 11.3 用 ASCII 图理解

```text
invoke:
[全部执行完成] -> 一次性得到最终 state

stream:
[步骤1更新] -> [步骤2更新] -> [LLM消息流] -> [步骤3更新] -> ...
```

---

## 十二、常见报错与调试

### 12.1 忘了在现代写法中显式指定 `version="v2"`

如果你在看官方现代示例时发现自己的输出结构和预期不一致，先检查是否显式写了：

```python
version="v2"
```

### 12.2 以为所有 `chunk` 结构都一样，不判断 `type`

多模式流式时，务必按：

```python
if chunk["type"] == ...:
```

分别处理。

不要假设所有 `data` 都同一种结构。

### 12.3 为什么没看到消息流？

可能原因包括：

- 当前图里没有 LLM 调用
- 你看的不是 `messages` 模式
- 过滤逻辑把消息跳过了

### 12.4 `values` 太吵看不清重点

这不是报错，但很常见。

如果你发现全量状态刷屏太厉害，先改回：

```python
stream_mode="updates"
```

### 12.5 调试建议：先从单模式开始

建议顺序：

1. 先只看 `updates`
2. 再加 `messages`
3. 需要产品化进度提示时再加 `custom`

这样最不容易一开始就被流事件淹没。

---

## 十三、动手练习

### 练习 1：把第 2 章最小图改成 `stream_mode="updates"`

要求：

- 不改节点逻辑
- 只把 `invoke()` 改成 `stream()`
- 观察每步更新内容

### 练习 2：给 Research Copilot 增加“业务进度文案”

要求：

- 在关键节点设计自定义进度信息
- 让输出更像：
  - 正在分析研究问题
  - 正在规划检索路径
  - 正在组织回答

### 练习 3：比较 `updates` 与 `values`

要求：

- 同一个图分别跑两种模式
- 自己总结：
  - 哪个更适合教学
  - 哪个更适合调试全量状态
  - 哪个更适合大型图

---

## 常见误区

| 误区 | 正确理解 |
|------|----------|
| “流式输出只是前端体验优化。” | 不止。它也是理解图执行、调试状态流和增强可观测性的核心工具。 |
| “`invoke()` 过时了，以后都不用。” | 不是。`invoke()` 仍然非常有用，只是在长任务和交互式场景下，`stream()` 更有优势。 |
| “`updates` 和 `values` 差不多。” | 差别很大。一个是增量更新，一个是全量快照。 |
| “只要开了 `messages` 就能看懂一切。” | 不够。`messages` 主要看模型输出，状态层推进通常更适合看 `updates` / `values`。 |
| “v2 只是可有可无的小版本参数。” | 不建议这样想。v2 输出格式更统一，教学和调试都更稳定。 |

---

## 思考题

### 基础理解

**Q1. 为什么 `invoke()` 不足以支撑复杂 agent 的良好体验？**

> [!hint]- 💡 提示
> 想想长任务期间，用户和开发者分别看不到什么。

> [!success]- ✅ 参考答案
> 因为 `invoke()` 通常只能在图完全执行结束后返回最终结果。对于包含工具调用、多步路由、审批和并行处理的复杂图，这会让用户看不到进度，也让开发者难以理解中间状态如何变化。`stream()` 则能把这些中间过程逐步暴露出来。

**Q2. `updates` 和 `values` 的核心区别是什么？**

> [!hint]- 💡 提示
> 一个是只看改动，一个是看快照。

> [!success]- ✅ 参考答案
> `updates` 只显示每一步新增或改变的状态字段，更简洁、更适合观察节点到底改了什么；`values` 则显示每一步后的完整状态快照，更适合想全面理解状态如何逐步长大时使用。

### 深入思考

**Q3. 为什么本系列推荐在现代流式示例里显式写 `version="v2"`？**

> [!hint]- 💡 提示
> 想想输出结构的一致性对教学有什么帮助。

> [!success]- ✅ 参考答案
> 因为 v2 的流输出格式更统一，不论是单模式、多模式还是子图场景，都使用一致的 `{"type", "ns", "data"}` 结构。这让教学时的心智模型更稳定，也让调试代码更容易按 `chunk["type"]` 分支处理，避免因格式差异带来额外混乱。

**Q4. 为什么说 `custom` 模式对产品体验特别重要？**

> [!hint]- 💡 提示
> 用户想看的是底层字段名，还是“正在做什么”？

> [!success]- ✅ 参考答案
> 因为用户通常不关心技术状态字段，而更关心系统当前在执行什么业务步骤，例如“正在检索资料”“正在整理证据”“正在生成结论”。`custom` 模式可以把底层图执行翻译成更自然的进度反馈，从而显著提升交互体验。

---

## 关键要点回顾

- ✅ `stream()` 解决的是复杂图的可观测性与交互体验问题
- ✅ `updates` 适合看增量状态，`values` 适合看全量状态快照
- ✅ `messages` 适合观察 LLM 输出流，`custom` 适合暴露业务进度事件
- ✅ `version="v2"` 提供更统一、稳定的流输出格式
- ✅ Research Copilot v5 的升级重点不是新增业务能力，而是让已有能力的执行过程变得可见

---

## 扩展阅读

- 📄 [Streaming](https://docs.langchain.com/oss/python/langgraph/streaming) — 本章最核心的官方参考
- 📄 [Quickstart](https://docs.langchain.com/oss/python/langgraph/quickstart) — 看函数式 API 中的 `.stream()` 最小示例

## 相关页面

- [[raw/lessons/LangGraph/06-human-in-the-loop|第 6 章：人在回路——让研究助手在关键步骤停下来等你审核]]
- [[raw/lessons/LangGraph/08-send-and-parallel|第 8 章：并行与分发——让研究助手同时处理多个检索任务]]
- [[wiki/concepts/langgraph-api-map|LangGraph API 学习地图]]

## 下一步学习

下一章我们会进一步升级执行能力：

- 为什么复杂研究任务适合拆成多个并行子任务
- `Send` 如何让图动态扇出多个任务
- Research Copilot 如何同时检索多个来源

继续阅读：[[raw/lessons/LangGraph/08-send-and-parallel|第 8 章：并行与分发——让研究助手同时处理多个检索任务]]
