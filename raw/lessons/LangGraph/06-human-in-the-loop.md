---
type: lesson
tags:
  - LangGraph
  - interrupt
  - Human-in-the-loop
  - Python
  - Agent
created: 2026-04-28
updated: 2026-04-28
topic: 人在回路——让研究助手在关键步骤停下来等你审核
difficulty: advanced
prerequisites:
  - "[[raw/lessons/LangGraph/05-checkpointer-memory|第 5 章：记忆与检查点——让研究助手跨轮记住上下文]]"
sources:
  - https://docs.langchain.com/oss/python/langgraph/interrupts
  - https://docs.langchain.com/oss/python/langgraph/graph-api
status: completed
series:
  name: LangGraph 系统学习
  part: 6
---

# 人在回路——让研究助手在关键步骤停下来等你审核

> 一句话摘要：有些 agent 任务不是“自己跑完就好”，而是在关键节点必须暂停让人确认。本章会让 Research Copilot 升级为支持人工审核的 v4：你将掌握 `interrupt()`、`Command(resume=...)`、带分支的恢复思维，并理解为什么 human-in-the-loop 是 LangGraph 最有现实价值的能力之一。

## 学习目标

完成本课后，你应该能够：
- [ ] 理解为什么研究助手、审批流、高风险操作都需要 human-in-the-loop
- [ ] 会在节点中使用 `interrupt()` 暂停执行
- [ ] 会用 `Command(resume=...)` 恢复图执行
- [ ] 理解“暂停前暴露什么信息给人看”是设计的一部分
- [ ] 能把 Research Copilot 升级为支持审核的 v4
- [ ] 会调试“暂停了但无法恢复”这类常见问题

## 前置知识

| 知识点 | 要求 | Wiki 参考 |
|--------|------|-----------|
| checkpointer | 知道为什么线程状态需要保存 | [[raw/lessons/LangGraph/05-checkpointer-memory|第 5 章：记忆与检查点——让研究助手跨轮记住上下文]] |
| `thread_id` | 知道会话线程如何区分 | [[raw/lessons/LangGraph/05-checkpointer-memory|第 5 章：记忆与检查点——让研究助手跨轮记住上下文]] |
| Graph API | 能读懂基础节点和边 | [[raw/lessons/LangGraph/02-stategraph-basics|第 2 章：StateGraph、节点与状态——让研究助手先跑起来]] |
| Python 条件分支 | 知道 `if/else` 语义 | — |

---

## 直观理解

### 类比一：Research Copilot 像“会先给导师看草稿的助理”

并不是所有任务都适合 agent 自己一口气跑完。

例如下面这些情况：

- 准备发送邮件
- 输出正式研究结论
- 删除数据
- 修改重要配置
- 给出高风险建议

这时你真正想要的不是“更自动”，而是：

> **先自动生成草稿，再停下来让我确认。**

这就是 human-in-the-loop 的现实价值。

### 类比二：`interrupt()` 像“举手等老师点头”

你可以把 `interrupt()` 理解成：

- 图执行到这里先别继续
- 把当前关键信息展示给人
- 等待外部给出意见
- 再带着那份意见恢复执行

它不是报错，不是失败，而是一种**设计出来的暂停**。

---

## 一、为什么 human-in-the-loop 不是“可选花活”？

很多新手刚接触 agent 时，会默认把目标想成：

> “最好自动从头跑到尾。”

但真实系统里，更常见的高质量目标其实是：

- 自动处理低风险部分
- 在关键点停下来请求确认
- 让人负责最终责任节点

这尤其适合：

- 研究助手
- 审批流
- 内容发布
- 外部系统操作
- 高成本工具调用

所以 human-in-the-loop 不是“自动化不够彻底”，而是：

> **对现实责任边界的尊重。**

---

## 二、Python 补课：联合类型、分支返回值与“恢复数据”的思维

> [!info] Python 补课：为什么这章需要分支返回思维？
> 因为暂停后恢复，外部传回来的内容可能是：
> - 一个布尔决策
> - 一段修改意见
> - 一个结构化字典
>
> 这意味着你的节点逻辑要开始具备“根据恢复结果分支”的能力。

### 2.1 最简单的恢复值：`True / False`

例如：

- `True` = 批准
- `False` = 拒绝

这适合最小审批场景。

### 2.2 更真实的恢复值：结构化字典

例如：

```python
{
    "action": "approve",
    "edited_summary": "..."
}
```

这会让审核不是“只能点头或摇头”，而是还能修改内容。

### 2.3 这章的 Python 重点不是语法花样，而是恢复协议

你真正要掌握的是：

> **当图停下来后，人会传回什么？节点又该怎样基于这个结果继续走？**

---

## 三、最小可运行示例：审批节点 + `interrupt()`

> [!example]- 最小审批流（点击展开）
>
> ```python
> from typing import Literal, Optional
> from typing_extensions import TypedDict
> from langgraph.checkpoint.memory import MemorySaver
> from langgraph.graph import StateGraph, START, END
> from langgraph.types import Command, interrupt
>
>
> class ReviewState(TypedDict):
>     draft: str
>     status: Optional[Literal["pending", "approved", "rejected"]]
>
>
> def review_node(state: ReviewState) -> Command[Literal["approve", "reject"]]:
>     decision = interrupt(
>         {
>             "question": "是否批准这段研究结论？",
>             "draft": state["draft"],
>         }
>     )
>
>     return Command(goto="approve" if decision else "reject")
>
>
> def approve_node(state: ReviewState):
>     return {"status": "approved"}
>
>
> def reject_node(state: ReviewState):
>     return {"status": "rejected"}
>
>
> builder = StateGraph(ReviewState)
> builder.add_node("review", review_node)
> builder.add_node("approve", approve_node)
> builder.add_node("reject", reject_node)
> builder.add_edge(START, "review")
> builder.add_edge("approve", END)
> builder.add_edge("reject", END)
>
> graph = builder.compile(checkpointer=MemorySaver())
>
> config = {"configurable": {"thread_id": "review-1"}}
>
> first = graph.invoke(
>     {"draft": "初版研究结论：LangGraph 的记忆应从 checkpoint 视角理解。", "status": "pending"},
>     config=config,
> )
>
> print(first["__interrupt__"])
>
> second = graph.invoke(Command(resume=True), config=config)
> print(second["status"])
> ```

### 3.1 这段代码最值得盯住哪两行？

第一行：

```python
decision = interrupt({...})
```

它让图在这里暂停，并把审核所需信息暴露给外部。

第二行：

```python
graph.invoke(Command(resume=True), config=config)
```

它表示：

- 回到同一条线程
- 继续这次暂停的执行
- 把 `True` 作为 `interrupt()` 的返回值交回节点逻辑

### 3.2 为什么 `interrupt()` 返回的不是立即结果？

因为它不是普通函数调用，而是一个“暂停点”。

第一次执行到这里时，图会停住。

等你后续用 `Command(resume=...)` 恢复时，那个 `resume` 里的值才会变成 `interrupt()` 的返回值。

这是本章最关键的运行时思维转换。

---

## 四、Research Copilot v4：让研究结论先过你这一关

### 4.1 本章项目升级目标

Research Copilot v4 要获得的核心能力是：

- 先自动整理研究结论草稿
- 在真正输出前暂停
- 让你决定：批准、拒绝，或修改后批准

### 4.2 为什么这对研究助手尤其重要？

因为研究助手不是聊天玩具，它很可能会输出：

- 阅读建议
- 方向判断
- 文献优先级
- 综述摘要
- 研究问题陈述

这些内容往往更适合：

> 先生成草稿，再由人确认最终责任。

---

## 五、Research Copilot v4：更贴近真实场景的示例

> [!example]- 支持人工编辑的审核节点（点击展开）
>
> ```python
> from typing_extensions import TypedDict
> from langgraph.checkpoint.memory import MemorySaver
> from langgraph.graph import StateGraph, START, END
> from langgraph.types import interrupt, Command
>
>
> class ResearchState(TypedDict):
>     question: str
>     generated_text: str
>
>
> def draft_summary(state: ResearchState):
>     return {
>         "generated_text": (
>             f"关于“{state['question']}”，建议先梳理概念定义，再阅读代表论文，最后分析应用场景。"
>         )
>     }
>
>
> def human_review(state: ResearchState):
>     updated = interrupt(
>         {
>             "instruction": "请审核并可直接修改这段研究建议",
>             "content": state["generated_text"],
>         }
>     )
>     return {"generated_text": updated}
>
>
> builder = StateGraph(ResearchState)
> builder.add_node("draft_summary", draft_summary)
> builder.add_node("human_review", human_review)
> builder.add_edge(START, "draft_summary")
> builder.add_edge("draft_summary", "human_review")
> builder.add_edge("human_review", END)
>
> graph = builder.compile(checkpointer=MemorySaver())
>
> config = {"configurable": {"thread_id": "research-review-1"}}
> first = graph.invoke({"question": "LangGraph 中的 interrupt 有什么用？", "generated_text": ""}, config=config)
>
> second = graph.invoke(
>     Command(resume="关于 interrupt，建议从暂停-恢复机制、审批流和线程恢复三方面理解。"),
>     config=config,
> )
> ```

### 5.1 这个版本比布尔批准更接近真实系统

因为它不再只是：

- 批准 / 拒绝

而是：

- 允许人直接修改内容再继续

这在研究助手、草稿审核、内容发布里特别实用。

---

## 六、`Command` 不只会 `resume`，它其实是“控制图”的统一原语

### 6.1 本章你最需要先掌握的是 `resume`

本章先聚焦：

```python
Command(resume=...)
```

它的意思是：

- 把某个值作为暂停点的恢复输入
- 从当前线程的中断位置继续执行

### 6.2 但你也要建立一个更大的心智模型

`Command` 在 LangGraph 中不只是恢复工具，它还是一种统一控制原语。

例如别的场景里它还能用于：

- 控制下一步 `goto`
- 同时更新状态与控制流

本章先不要把范围扩太大，但你要知道：

> `resume` 只是 `Command` 能力的一部分。

---

## 七、状态 / 流程 walkthrough：暂停与恢复到底是怎么发生的？

### 7.1 第一次调用

```text
START -> draft_summary -> human_review -> interrupt() -> 暂停
```

图不会继续到 `END`，而是返回一个包含 `__interrupt__` 的结果。

### 7.2 暂停时系统暴露了什么？

比如：

```python
{
    "instruction": "请审核并可直接修改这段研究建议",
    "content": "关于某研究主题的草稿..."
}
```

这就是你交给“人”的审核材料。

### 7.3 第二次恢复调用

```python
graph.invoke(Command(resume="修改后的内容"), config=config)
```

这时图会：

- 回到同一条线程
- 找到刚才的暂停点
- 把 `resume` 的值喂回 `interrupt()`
- 然后继续往下走到 `END`

### 7.4 用 ASCII 图看暂停-恢复过程

```text
第一次调用：
START -> 生成草稿 -> human_review -> interrupt() -> [暂停]

第二次调用：
Command(resume=审核结果) -> 回到暂停点 -> 继续执行 -> END
```

> [!important] 本章最重要的运行时直觉
> `interrupt()` 不是报错，也不是 return。
>
> 它更像是：
>
> **“把图挂起，等外部世界给出下一步输入后再继续。”**

---

## 八、为什么这章和第 5 章强绑定？

如果没有第 5 章的 checkpointer 和 `thread_id`，你几乎无法优雅地理解这章。

因为暂停以后最重要的问题正是：

- 这次暂停属于哪条线程？
- 稍后恢复时从哪里继续？

所以你会发现：

- 第 5 章解决“状态持续性”
- 第 6 章利用这种持续性做“人工审核恢复”

这也是为什么本系列不建议跳学。

---

## 九、常见报错与调试

### 9.1 没有 checkpointer 就谈恢复

如果你试图做 `interrupt()` 恢复，却没有为图配置合适的 checkpointer，体验通常会非常混乱。

因为暂停恢复天然依赖状态被保存。

### 9.2 恢复时 `thread_id` 不一致

第一次暂停时：

```python
thread_id = "review-1"
```

恢复时如果换成了：

```python
thread_id = "review-2"
```

那系统就找不到同一条暂停线程。

### 9.3 恢复值结构与节点预期不一致

如果你的节点预期是：

```python
{"action": "approve", "edited_summary": "..."}
```

你却只传了：

```python
True
```

逻辑就会不匹配。

所以要尽早固定“恢复协议”。

### 9.4 把 `interrupt()` 当成普通输入函数

错误理解：

- “它会像 `input()` 一样当场拿到结果”

更准确的理解：

- 它会让图暂停
- 结果来自**未来的恢复调用**

### 9.5 调试建议：先看 `__interrupt__`

第一次暂停后，不要急着猜系统要什么。

先直接看：

```python
result["__interrupt__"]
```

你会看到当前暂停点实际暴露给外部的信息。

---

## 十、动手练习

### 练习 1：实现“批准 / 拒绝”双分支

要求：

- `interrupt()` 返回布尔值
- `True` 走批准节点
- `False` 走拒绝节点
- 在最终状态里记录 `status`

### 练习 2：实现“可编辑审核”

要求：

- 暂停时把草稿给人看
- 恢复时允许传回修改后的完整文本
- 最终输出使用修改后的版本

### 练习 3：把审核节点接回 Research Copilot 主线

要求：

- 先生成 Research Copilot 的研究建议草稿
- 在输出前插一个 `human_review` 节点
- 让它变成真正的 Research Copilot v4

---

## 常见误区

| 误区 | 正确理解 |
|------|----------|
| “human-in-the-loop 说明 agent 不够强。” | 不对。很多高质量系统恰恰要求在关键节点保留人工责任边界。 |
| “`interrupt()` 就像普通 `input()`。” | 不是。它会暂停图执行，等待未来的 `Command(resume=...)` 恢复。 |
| “暂停后只要再 invoke 一次就行，不必管线程。” | 错。恢复必须回到同一条线程，否则无法定位暂停状态。 |
| “审核只能做批准/拒绝。” | 不够。更真实的系统往往支持人工修改内容后再继续。 |
| “本章只是个高级附加功能。” | 不是。它是 LangGraph 在审批流、内容流、研究流中最具现实价值的能力之一。 |

---

## 思考题

### 基础理解

**Q1. 为什么很多真实 agent 系统必须支持 human-in-the-loop？**

> [!hint]- 💡 提示
> 想想高风险输出、正式发布、外部操作这些场景。

> [!success]- ✅ 参考答案
> 因为很多真实任务涉及责任边界和风险控制。系统可以自动完成大部分准备工作，但在关键节点仍然需要人确认，例如审核研究结论、确认邮件发送、批准外部操作。human-in-the-loop 不是自动化失败，而是现实系统中常见且必要的设计。

**Q2. `interrupt()` 和 `Command(resume=...)` 的配合关系是什么？**

> [!hint]- 💡 提示
> 谁负责暂停，谁负责把值送回来？

> [!success]- ✅ 参考答案
> `interrupt()` 负责在节点内部创建一个暂停点，并把审核所需信息暴露给外部。之后，外部再通过 `Command(resume=...)` 把恢复值送回同一条线程，这个值会成为 `interrupt()` 的返回值，图再继续执行。

### 深入思考

**Q3. 为什么说“暂停前要暴露什么给人看”本身就是系统设计的一部分？**

> [!hint]- 💡 提示
> 如果给人的信息不完整，他能做出好决策吗？

> [!success]- ✅ 参考答案
> 因为人是否能高质量审核，取决于系统在暂停时提供的信息是否充分、清晰、可修改。若只给出模糊提示，人就无法判断该批准什么；若给出过多无关细节，又会增加审核成本。所以暂停点暴露的数据结构，实际上就是 human-in-the-loop 的用户界面契约。

**Q4. 为什么本章与第 5 章的 checkpointer 强绑定？**

> [!hint]- 💡 提示
> 暂停后如果没有保存状态，恢复时靠什么回去？

> [!success]- ✅ 参考答案
> 因为 `interrupt()` 的恢复依赖状态持续性。图暂停后，系统必须知道这次暂停属于哪条线程、停在什么位置、状态长什么样。checkpointer 正是负责保存这些信息的关键机制。所以没有第 5 章的线程化持久化心智模型，第 6 章很难真正讲透。

---

## 关键要点回顾

- ✅ human-in-the-loop 是很多真实 agent 系统的核心能力，不是附属功能
- ✅ `interrupt()` 用于设计性暂停，`Command(resume=...)` 用于恢复执行
- ✅ 恢复值不是普通同步输入，而是未来恢复调用注入回来的值
- ✅ 审核协议设计很重要：你要先想清楚人会看到什么、返回什么
- ✅ Research Copilot v4 已经具备“生成草稿 -> 等你审核 -> 再继续”的现实工作流雏形

---

## 扩展阅读

- 📄 [Interrupts](https://docs.langchain.com/oss/python/langgraph/interrupts) — 本章最核心的官方参考
- 📄 [Graph API](https://docs.langchain.com/oss/python/langgraph/graph-api) — 查 `Command` 与控制流相关能力

## 相关页面

- [[raw/lessons/LangGraph/05-checkpointer-memory|第 5 章：记忆与检查点——让研究助手跨轮记住上下文]]
- [[raw/lessons/LangGraph/07-streaming|第 7 章：流式执行——实时看见研究助手正在如何思考与推进]]
- [[wiki/concepts/langgraph-api-map|LangGraph API 学习地图]]

## 下一步学习

下一章我们会解决一个体验层的重要问题：

- 为什么 `invoke()` 虽然能得到最终结果，但看不到中间过程
- `stream()` 和 `stream_mode` 能让你实时看到什么
- Research Copilot 如何把执行过程流式展示出来

继续阅读：[[raw/lessons/LangGraph/07-streaming|第 7 章：流式执行——实时看见研究助手正在如何思考与推进]]
