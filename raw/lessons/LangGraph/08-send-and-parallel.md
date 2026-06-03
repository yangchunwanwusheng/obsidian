---
type: lesson
tags:
  - LangGraph
  - Send
  - Parallel
  - Python
  - Agent
created: 2026-04-28
updated: 2026-04-28
topic: 并行与分发——让研究助手同时处理多个检索任务
difficulty: advanced
prerequisites:
  - "[[raw/lessons/LangGraph/07-streaming|第 7 章：流式执行——实时看见研究助手正在如何思考与推进]]"
sources:
  - https://docs.langchain.com/oss/python/langgraph/use-graph-api
  - https://docs.langchain.com/oss/python/langgraph/streaming
status: completed
series:
  name: LangGraph 系统学习
  part: 8
---

# 并行与分发——让研究助手同时处理多个检索任务

> 一句话摘要：前面几章的图大多还是“一步接一步”的串行思维，本章会让 Research Copilot 升级为支持并行检索的 v6。你将掌握 `Send`、动态扇出、结果聚合，以及为什么多来源研究任务天然适合 map-reduce 风格。

## 学习目标

完成本课后，你应该能够：
- [ ] 理解为什么研究任务经常适合拆成多个并行子任务
- [ ] 会用 `Send` 动态分发多个任务
- [ ] 能看懂 map-reduce 风格在 LangGraph 里的基本形态
- [ ] 理解为什么状态聚合需要提前设计字段规则
- [ ] 能把 Research Copilot 升级为并行检索多个来源的 v6
- [ ] 知道并行图常见的调试重点在哪里

## 前置知识

| 知识点 | 要求 | Wiki 参考 |
|--------|------|-----------|
| 状态与图结构 | 能读懂节点、边、状态 | [[raw/lessons/LangGraph/02-stategraph-basics|第 2 章：StateGraph、节点与状态——让研究助手先跑起来]] |
| 流式观察 | 知道如何看 `updates` | [[raw/lessons/LangGraph/07-streaming|第 7 章：流式执行——实时看见研究助手正在如何思考与推进]] |
| 工具调用 | 理解检索工具接入图 | [[raw/lessons/LangGraph/04-tools-and-react|第 4 章：工具与 ReAct——让研究助手学会调用检索工具]] |
| Python 列表 | 知道列表可存多个任务 | — |

---

## 直观理解

### 类比一：研究助手不该一个人顺序查完所有资料

假设你要研究一个主题：

> “LangGraph 中 memory、interrupt、streaming 的联系是什么？”

如果你让一个助理顺序做：

1. 先查 memory
2. 再查 interrupt
3. 再查 streaming
4. 最后汇总

当然能做，但会显得很慢。

更合理的思路往往是：

- 一个人查 memory
- 一个人查 interrupt
- 一个人查 streaming
- 最后统一汇总

这就是并行分工的直觉。

### 类比二：`Send` 像“派发任务卡”

`Send` 可以理解成：

- 不只是“走向某个固定下一节点”
- 而是“给这个节点派出多份任务卡，每份卡片带自己的输入”

也就是说，它解决的不是简单路由，而是：

> **动态扇出多个任务实例。**

---

## 一、为什么并行能力对 Research Copilot 很自然？

研究任务天然有一个特点：

- 一个主题常常可以拆成多个子问题
- 多个来源常常可以独立检索
- 最后再做汇总比从头串行到底更合理

例如一个研究助手常见的工作步骤就可能是：

- 查定义
- 查代表论文
- 查应用场景
- 查相关概念对比

这些很多都适合并行进行。

所以本章的升级不是硬塞的“高级技巧”，而是对研究任务自然结构的映射。

---

## 二、Python 补课：列表推导式与“任务拆分结果是个列表”

> [!info] Python 补课：为什么这章列表思维特别重要？
> 因为并行分发前，你通常先会得到一个：
> - 主题列表
> - 子任务列表
> - 来源列表
>
> 然后再把这个列表映射成多份 `Send(...)` 任务。

### 2.1 最常见的拆分模式

```python
subjects = ["memory", "interrupt", "streaming"]
```

接着你会写：

```python
[Send("retrieve_topic", {"subject": s}) for s in subjects]
```

这其实就是把“一个总任务”映射成“多个子任务实例”。

### 2.2 这章真正要学的是“任务列表思维”

也就是说：

- 不再只想着下一步去一个节点
- 而是开始想着：
  - 我能不能把当前问题拆成多个并行输入？
  - 每个输入的最小状态应该是什么？

---

## 三、最小可运行示例：`Send` 的 map-reduce 风格

> [!example]- 最小并行分发示例（点击展开）
>
> ```python
> import operator
> from typing_extensions import TypedDict, Annotated
> from langgraph.graph import StateGraph, START, END
> from langgraph.types import Send
>
>
> class OverallState(TypedDict):
>     topic: str
>     subtopics: list[str]
>     notes: Annotated[list[str], operator.add]
>     summary: str
>
>
> def split_topic(state: OverallState):
>     return {"subtopics": ["核心定义", "代表论文", "应用场景"]}
>
>
> def fan_out(state: OverallState):
>     return [Send("retrieve_subtopic", {"subtopic": s}) for s in state["subtopics"]]
>
>
> def retrieve_subtopic(state: dict):
>     return {"notes": [f"关于 {state['subtopic']} 的初步检索结果"]}
>
>
> def summarize(state: OverallState):
>     return {"summary": "\n".join(state["notes"])}
>
>
> builder = StateGraph(OverallState)
> builder.add_node("split_topic", split_topic)
> builder.add_node("retrieve_subtopic", retrieve_subtopic)
> builder.add_node("summarize", summarize)
> builder.add_edge(START, "split_topic")
> builder.add_conditional_edges("split_topic", fan_out, ["retrieve_subtopic"])
> builder.add_edge("retrieve_subtopic", "summarize")
> builder.add_edge("summarize", END)
>
> graph = builder.compile()
> ```

### 3.1 这段代码里最新颖的地方在哪里？

不是节点本身，而是这一句：

```python
return [Send("retrieve_subtopic", {"subtopic": s}) for s in state["subtopics"]]
```

它表示：

- 对 `subtopics` 中的每个元素
- 都派发一个去 `retrieve_subtopic` 的任务
- 每个任务带自己的局部输入

这就是扇出。

### 3.2 为什么 `notes` 要用 `Annotated[..., operator.add]`？

因为多个并行子任务都会产出：

```python
{"notes": [某一条笔记]}
```

如果你不提前定义好“怎么聚合这些结果”，就会出问题。

`operator.add` 的语义很直白：

- 列表 + 列表 = 追加拼接

这使得多份子任务结果可以汇总到同一个 `notes` 通道里。

---

## 四、Research Copilot v6：把一个研究问题拆成多个并行检索来源

### 4.1 本章项目升级目标

Research Copilot v6 的核心升级是：

- 不再只会顺序检索
- 而是能把研究问题拆成多个并行来源或维度
- 最后再把多路结果聚合成更完整的研究起点

### 4.2 更贴近研究助手的拆分方式

对于一个主题，你可以并行拆成：

- 概念解释
- 代表论文
- 应用场景
- 相关概念对比

也可以按来源拆成：

- 论文线
- 博客线
- 现有知识库线

本章先不追求最复杂，而是先建立：

> **一个大研究问题可以被拆成多个可独立推进的小任务。**

---

## 五、Research Copilot v6：更贴近课程主线的示例

> [!example]- 按研究维度并行拆分（点击展开）
>
> ```python
> import operator
> from typing_extensions import TypedDict, Annotated
> from langgraph.graph import StateGraph, START, END
> from langgraph.types import Send
>
>
> class ResearchState(TypedDict):
>     topic: str
>     subtopics: list[str]
>     notes: Annotated[list[str], operator.add]
>     summary: str
>
>
> def plan_research(state: ResearchState):
>     return {
>         "subtopics": [
>             f"{state['topic']} 的核心定义",
>             f"{state['topic']} 的代表论文",
>             f"{state['topic']} 的应用场景",
>         ]
>     }
>
>
> def dispatch_subtasks(state: ResearchState):
>     return [Send("retrieve_one", {"subtopic": s}) for s in state["subtopics"]]
>
>
> def retrieve_one(state: dict):
>     return {"notes": [f"检索结果：{state['subtopic']}"]}
>
>
> def synthesize(state: ResearchState):
>     return {
>         "summary": "\n".join(
>             [
>                 "Research Copilot 并行检索汇总：",
>                 *state["notes"],
>             ]
>         )
>     }
>
>
> builder = StateGraph(ResearchState)
> builder.add_node("plan_research", plan_research)
> builder.add_node("retrieve_one", retrieve_one)
> builder.add_node("synthesize", synthesize)
> builder.add_edge(START, "plan_research")
> builder.add_conditional_edges("plan_research", dispatch_subtasks, ["retrieve_one"])
> builder.add_edge("retrieve_one", "synthesize")
> builder.add_edge("synthesize", END)
> graph = builder.compile()
> ```

### 5.1 这段图最值得吸收的不是语法，而是架构套路

它背后的套路其实非常通用：

1. 先规划要拆成哪些子任务
2. 再动态扇出多个任务
3. 每个任务独立产出局部结果
4. 最后统一聚合汇总

这就是最小 map-reduce 风格。

---

## 六、为什么并行图最容易翻车的地方不是 `Send` 本身，而是“状态聚合设计”？

很多人第一次学并行时，会把注意力都放在：

- `Send` 怎么写
- 条件边怎么接

但真正更容易出问题的是：

> 多个并行任务写回同一份状态时，结果怎么合并？

### 6.1 一个典型问题

如果三个并行任务都返回：

```python
{"notes": ["...某条结果..."]}
```

你得先定义清楚：

- 是替换？
- 是拼接？
- 是去重后合并？

### 6.2 这就是为什么本章强调聚合规则

并行图往往不是卡在“会不会扇出”，而是卡在：

- 多路结果如何回收
- 回收后顺序是否重要
- 哪个字段适合做聚合通道

---

## 七、状态 / 流程 walkthrough：并行图到底怎么跑？

### 7.1 第一步：规划子任务

`plan_research` 生成：

```python
subtopics = [
    "LangGraph memory 的核心定义",
    "LangGraph memory 的代表论文",
    "LangGraph memory 的应用场景",
]
```

### 7.2 第二步：扇出多个 `Send`

`dispatch_subtasks` 返回多个：

```python
Send("retrieve_one", {"subtopic": ...})
```

你可以把它理解成：

- 给同一个节点派发多张任务卡
- 每张卡只带自己要检索的子主题

### 7.3 第三步：并行子任务回写局部结果

每个 `retrieve_one` 产出一条：

```python
{"notes": ["某条检索结果"]}
```

### 7.4 第四步：统一汇总

`synthesize` 最后读取聚合后的 `notes` 列表，生成总总结。

### 7.5 用 ASCII 图看扇出

```text
START
  │
  ▼
plan_research
  │
  ▼
Send -> retrieve_one(定义)
  ├──► retrieve_one(论文)
  └──► retrieve_one(应用)
          │
          ▼
      synthesize
          │
          ▼
         END
```

---

## 八、并行与 streaming 为什么是天然搭配？

你在上一章学过 `stream()` 之后，会发现并行图特别适合配合流式观察。

原因很简单：

- 并行图更复杂
- 多路子任务同时推进
- 如果只看最终结果，很难知道中间发生了什么

所以本章一个很重要的实践建议是：

> **并行图几乎总是值得配合 `stream_mode="updates"` 观察。**

这样你就能看到：

- 哪路子任务先返回
- 哪个节点回写了什么
- 聚合结果如何逐步长出来

---

## 九、常见报错与调试

### 9.1 状态聚合规则没定义好

如果多个子任务都写同一个字段，而你没有为这个字段定义合适的聚合规则，就很容易：

- 被后写的覆盖前写的
- 或结果结构不符合预期

### 9.2 子任务输入状态过重

并行任务最好只拿自己需要的最小输入。

如果你给每个 `Send` 都塞一大坨无关状态，会让图更难理解，也更难调试。

### 9.3 误以为 `Send` 只是普通条件跳转

不是。

普通条件跳转更像：

- 选择去 A 还是 B

而 `Send` 更像：

- 动态派出多份去同一节点或多个任务实例

### 9.4 汇总节点过早依赖未准备好的字段

在设计汇总节点前，要先确认并行子任务确实会稳定地产出你需要的字段。

### 9.5 调试建议：先把并行任务数量控制在 2-3 个

本章最适合的学习方式不是一上来就十几路并行，而是：

- 先 2-3 路
- 先看 `updates`
- 先确认聚合字段正确

这样心智负担最小。

---

## 十、动手练习

### 练习 1：把 3 路并行改成“按来源并行”

要求：

- 改成并行拆成：
  - 论文来源
  - 博客来源
  - 本地知识库来源
- 每个来源返回一条不同风格的结果文本

### 练习 2：增加一个 `best_summary` 汇总字段

要求：

- 在 `synthesize` 中不要只拼接结果
- 再额外生成一条“最值得先看的研究入口”总结

### 练习 3：配合 `stream_mode="updates"` 观察扇出过程

要求：

- 运行并行图
- 记录你看到的每次状态更新
- 总结：并行图和线性图在可观测性上最大的不同是什么

---

## 常见误区

| 误区 | 正确理解 |
|------|----------|
| “并行一定比串行好。” | 不一定。只有任务彼此足够独立时，并行才真正有价值。 |
| “学会 `Send` 就等于掌握并行图。” | 不够。真正难点通常在状态聚合与结果汇总设计。 |
| “并行图就是更花哨的条件路由。” | 不是。条件路由是在分支选择，`Send` 更偏动态扇出多个任务实例。 |
| “每个并行子任务都该拿完整大状态。” | 通常不必。最小必要输入更易读、更稳。 |
| “并行图只看最终 summary 就够了。” | 不够。并行图尤其适合配合 `stream()` 观察中间状态演化。 |

---

## 思考题

### 基础理解

**Q1. 为什么研究任务天然适合并行分发？**

> [!hint]- 💡 提示
> 一个主题能不能拆成多个相对独立的研究维度？

> [!success]- ✅ 参考答案
> 因为研究问题往往可以拆成多个彼此相对独立的子任务，例如概念定义、代表论文、应用场景、相关概念对比等。这些部分很多可以同时推进，最后再统一汇总，因此天然适合并行分发。

**Q2. `Send` 和普通条件分支的核心区别是什么？**

> [!hint]- 💡 提示
> 一个是选路，一个是派多张任务卡。

> [!success]- ✅ 参考答案
> 普通条件分支更像是在几个候选路径中选择一条，而 `Send` 更像是基于当前状态动态创建多份任务实例，把它们派发到目标节点执行。所以 `Send` 解决的是扇出与多任务分发，而不只是简单路由选择。

### 深入思考

**Q3. 为什么说并行图真正难的地方往往不是 `Send`，而是聚合设计？**

> [!hint]- 💡 提示
> 多个子任务都会回写同一状态时，会发生什么？

> [!success]- ✅ 参考答案
> 因为扇出本身只是把任务拆开，真正棘手的是这些子任务结果如何安全、清晰地合并回主状态。你必须提前定义字段如何聚合、是否保序、是否去重、汇总节点如何读取这些结果。如果聚合规则没设计好，并行图即使能跑，也很容易产生混乱结果。

**Q4. 为什么并行图几乎总是值得配合 `stream()` 观察？**

> [!hint]- 💡 提示
> 多路子任务同时推进时，只看最终结果够不够？

> [!success]- ✅ 参考答案
> 因为并行图的中间过程本身比线性图更复杂：多个子任务可能交错返回、不同字段逐步聚合、汇总节点依赖的中间数据也更多。如果只看最终结果，很难判断哪一步出了问题。配合 `stream_mode="updates"`，你可以清楚地看到多路任务如何逐步把主状态拼起来。

---

## 关键要点回顾

- ✅ `Send` 让图从“选一条路走”升级为“动态派出多份任务”
- ✅ 并行检索非常适合 Research Copilot 这种多来源、多维度研究任务
- ✅ 并行图的关键难点通常在状态聚合，而不是扇出语法本身
- ✅ `Annotated[..., operator.add]` 这类聚合规则在并行场景特别重要
- ✅ Research Copilot v6 已经具备最小 map-reduce 风格的并行研究能力

---

## 扩展阅读

- 📄 [Graph API / Send](https://docs.langchain.com/oss/python/langgraph/use-graph-api) — 查看官方 map-reduce 风格示例
- 📄 [Streaming](https://docs.langchain.com/oss/python/langgraph/streaming) — 回查如何观察并行状态更新

## 相关页面

- [[raw/lessons/LangGraph/07-streaming|第 7 章：流式执行——实时看见研究助手正在如何思考与推进]]
- [[raw/lessons/LangGraph/09-subgraphs-and-composition|第 9 章：子图与组合——把研究助手拆成可复用模块]]
- [[wiki/concepts/langgraph-api-map|LangGraph API 学习地图]]

## 下一步学习

下一章我们会进一步升级代码组织方式：

- 为什么图越来越大后，不能继续所有节点都堆在一个文件里
- 子图如何作为可复用模块接入父图
- Research Copilot 如何拆成搜索、整理、汇总等可组合子系统

继续阅读：[[raw/lessons/LangGraph/09-subgraphs-and-composition|第 9 章：子图与组合——把研究助手拆成可复用模块]]
