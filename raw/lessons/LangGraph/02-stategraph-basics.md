---
type: lesson
tags:
  - LangGraph
  - StateGraph
  - Python
  - Agent
  - 工作流
created: 2026-04-28
updated: 2026-04-28
topic: StateGraph、节点与状态——让研究助手先跑起来
difficulty: beginner
prerequisites:
  - "[[raw/lessons/LangGraph/01-langgraph-overview|第 1 章：LangGraph 学习地图与研究助手项目总览]]"
sources:
  - https://docs.langchain.com/oss/python/langgraph/overview
  - https://docs.langchain.com/oss/python/langgraph/quickstart
  - https://docs.langchain.com/oss/python/langgraph/graph-api
  - https://docs.langchain.com/oss/python/langgraph/streaming
status: completed
series:
  name: LangGraph 系统学习
  part: 2
---

# StateGraph、节点与状态——让研究助手先跑起来

> 一句话摘要：本章会把 LangGraph 的最小骨架真正跑起来。你将写出一个最小可运行的 Research Copilot v0，理解 `StateGraph`、node、edge、`START`、`END`、局部状态更新，并建立整个系列最重要的底层心智模型。

## 学习目标

完成本课后，你应该能够：
- [ ] 能解释为什么 LangGraph 一上来先定义 state，而不是先堆 prompt
- [ ] 理解 `StateGraph`、node、edge、`START`、`END` 的职责分工
- [ ] 会用 `TypedDict` 定义一个最小工作流状态
- [ ] 能写出并运行一个最小 3 节点 Research Copilot v0
- [ ] 能从“状态变化”的角度阅读一张 LangGraph 图
- [ ] 遇到常见报错时，知道先检查 state 字段、节点返回值和边连接关系

## 前置知识

| 知识点             | 要求                | Wiki 参考                                       |                                 |
| --------------- | ----------------- | --------------------------------------------- | ------------------------------- |
| 系列总览            | 读过第 1 章即可         | [[raw/lessons/LangGraph/01-langgraph-overview | 第 1 章：LangGraph 学习地图与研究助手项目总览]] |
| Python 函数       | 知道函数有参数和返回值       | —                                             |                                 |
| `dict` / `list` | 知道字典存字段、列表存多个元素   | —                                             |                                 |
| Agent 基本印象      | 知道 agent 可能需要多步执行 | [[research/01-topics/agent-memory             | Agent Memory 研究方向]]             |

---

## 直观理解

### 类比一：StateGraph 像“实验记录本 + 流程图”

想象你在做一个小研究任务：

> “给我一个关于 Agent Memory 的快速研究起点。”

如果没有结构，你很可能会：

- 看一眼问题就开始写
- 中途忘记前面整理过什么
- 修改输出时把原始问题也弄乱
- 后面想加检索或审核时，完全找不到插入点

LangGraph 的思路是先把这件事拆开：

1. 先准备共享记录本 → **state**
2. 再画出处理流程 → **graph**
3. 每一步只做自己负责的一小段 → **node**

> [!tip] 关键直觉
> LangGraph 的核心不是“写一个什么都做的超级函数”，而是：
>
> **把任务拆成若干小步骤，让每一步只负责给 state 增加一块新信息。**

### 类比二：节点像“接力赛中的单一职责选手”

在接力赛里，最糟糕的不是选手不够强，而是每个人都在抢着做别人的活。

LangGraph 通过图结构把边界说清楚：

- **node**：我只负责这一小步
- **edge**：我做完以后交给谁
- **state**：接力棒上到底携带了什么信息

这让系统特别适合做“逐步扩展”的 agent 项目：先做最小图，再加消息、工具、记忆、审批、并行。

---

## 一、先建立最重要的心智模型：LangGraph = State + Nodes + Edges

### 1.1 什么是 state？

在 LangGraph 里，**state 是整个流程共享的数据工作台**。

它回答的是：

- 当前用户问了什么？
- 我已经整理出了哪些中间结果？
- 目前流程进行到哪里？
- 下一步还可以基于什么继续工作？

对初学者来说，可以先把 state 理解成一个**结构化字典**。

我们这章的最小 Research Copilot v0，只需要 3 个字段：

- `question`：原始问题
- `notes`：中间研究笔记
- `answer`：最终输出

### 1.2 什么是 node？

node 就是一个处理步骤。

典型行为是：

1. 读取当前 state
2. 做一小步处理
3. 返回局部更新

例如：

- `understand_question`：先把问题转成更明确的研究任务
- `draft_notes`：根据任务草拟研究笔记
- `write_answer`：把前两步结果组织成初版回答

### 1.3 什么是 edge？

edge 就是“做完这一步以后去哪里”。

最简单的线性流就是：

```text
START → understand_question → draft_notes → write_answer → END
```

本章先不讲复杂条件路由，先把最基础的直线流真正学稳。

### 1.4 `START` / `END` 为什么重要？

很多新手第一次看图，会觉得：

> “我自己知道哪里开始、哪里结束，不就行了吗？”

但显式写出 `START` 和 `END` 的价值非常大：

- 图的入口明确
- 图的出口明确
- 调试时边界更清楚
- 后面加分支时不会迷路

> [!tip] 地铁图类比
> 没有 `START` / `END` 的图，就像一张没标首末站的地铁图。你也许勉强能看懂，但线路一复杂，马上会混乱。

---

## 二、Python 补课：为什么这里先讲 `TypedDict`

> [!info] Python 补课：`TypedDict` 是什么？
> `TypedDict` 用来描述“这个字典应该有哪些字段、每个字段是什么类型”。
>
> ```python
> from typing_extensions import TypedDict
>
> class ResearchState(TypedDict):
>     question: str
>     notes: str
>     answer: str
> ```
>
> 它不是在创建一个普通 class 实例，而是在给字典写一张“字段说明书”。

### 2.1 为什么不用裸 `dict` 开始？

因为一旦状态字段变多，如果不先说清结构，就很容易出现这些问题：

- 这个键到底叫 `note` 还是 `notes`？
- 这里装的是字符串、列表，还是消息对象？
- 后面节点到底依赖了哪些字段？
- 哪个节点负责更新哪个字段？

所以本系列从一开始就养成习惯：

> **先定义状态结构，再写节点。**

### 2.2 类型注解在这里的实际价值

你不需要把类型系统学得很重，但在 LangGraph 里，类型注解至少有 3 个实用价值：

1. 让自己不容易写错字段
2. 让编辑器更容易提示你 state 结构
3. 让后面章节升级到 `MessagesState`、`Command`、`Send` 时更自然

### 2.3 “局部状态更新”思维是什么？

在普通 Python 编程里，你可能很习惯写这种风格：

```python
def process(data):
    data["x"] = ...
    data["y"] = ...
    data["z"] = ...
    return data
```

但在 LangGraph 节点设计里，更推荐这种思路：

```python
def process(state):
    return {"x": ...}
```

也就是说：

- 我只提交自己负责更新的字段
- 我不偷偷覆盖别的字段
- 我让这个节点的职责一眼可见

> [!info] 这其实是一种工程习惯
> 你可以把它理解成 Git 提交：
> - 好的节点只提交自己这一小步改动
> - 不好的节点像“顺手改了一堆地方但没告诉你”

### 2.4 字段命名约定为什么要尽早固定

本章虽然只有 3 个字段，但你现在就应该意识到：

- `question` 是输入原题
- `notes` 是中间材料
- `answer` 是最终交付物

一旦这些名字定下来，后面章节的升级版就要尽量沿着它们演进，而不是每章随意重命名。这样项目主线才连续。

---

## 三、Research Copilot v0：本章真正要交付的最小项目

### 3.1 本章目标不要贪大

这一章的目标不是做完整研究助手，而是做一个**最小可运行版本**：

输入：
- 一个研究问题

流程：
1. 理解问题
2. 整理研究笔记
3. 写出初版回答

输出：
- 一个结构化、像“研究起点”的简短回答

### 3.2 本章代码基线：后面章节都会从这里长出来

本章不是临时 demo，而是整个系列的项目基线：

- 第 3 章会把这套状态升级成消息态
- 第 4 章会在这个流程上接工具
- 第 5 章会给它加记忆
- 第 6 章会给它加人工审核

所以你现在写的不是“玩具代码”，而是 **Research Copilot v0 的底盘**。

### 3.3 先画流程，再写代码

```text
用户问题
   │
   ▼
understand_question
   │
   ▼
draft_notes
   │
   ▼
write_answer
   │
   ▼
最终回答
```

---

## 四、最小可运行示例：Research Copilot v0

> [!example]- 最小可运行版本（点击展开）
>
> ```python
> from typing_extensions import TypedDict
> from langgraph.graph import StateGraph, START, END
>
>
> class ResearchState(TypedDict):
>     question: str
>     notes: str
>     answer: str
>
>
> def understand_question(state: ResearchState):
>     question = state["question"]
>     return {
>         "notes": f"研究任务：先澄清这个问题的核心范围——{question}"
>     }
>
>
> def draft_notes(state: ResearchState):
>     notes = state["notes"]
>     return {
>         "notes": notes + "\n- 先找定义\n- 再找代表性论文\n- 最后整理应用场景"
>     }
>
>
> def write_answer(state: ResearchState):
>     return {
>         "answer": (
>             f"你的研究问题是：{state['question']}\n\n"
>             f"我建议的起步路径是：\n{state['notes']}\n\n"
>             "先从概念定义、代表论文和应用场景三部分入手。"
>         )
>     }
>
>
> builder = StateGraph(ResearchState)
> builder.add_node("understand_question", understand_question)
> builder.add_node("draft_notes", draft_notes)
> builder.add_node("write_answer", write_answer)
>
> builder.add_edge(START, "understand_question")
> builder.add_edge("understand_question", "draft_notes")
> builder.add_edge("draft_notes", "write_answer")
> builder.add_edge("write_answer", END)
>
> graph = builder.compile()
>
> result = graph.invoke(
>     {
>         "question": "LangGraph 中的 agent memory 应该怎么设计？",
>         "notes": "",
>         "answer": "",
>     }
> )
>
> print(result["answer"])
> ```

### 4.1 先别急着背代码，先看整体结构

这段代码只做了 7 件事：

1. 定义状态 schema
2. 写 3 个节点函数
3. 创建 `StateGraph`
4. 注册节点
5. 添加边
6. `compile()`
7. `invoke()` 执行

如果你能说清这 7 步，这章就已经成功了一大半。

### 4.2 `StateGraph(ResearchState)` 在做什么？

```python
builder = StateGraph(ResearchState)
```

这一步是在告诉 LangGraph：

- 整个图围绕 `ResearchState` 运转
- 后续节点读取和更新的都是这个状态结构
- 图的组织方式以这个 schema 为中心

### 4.3 `add_node()` 在做什么？

```python
builder.add_node("understand_question", understand_question)
```

你可以把它理解成注册一个步骤：

- 这个步骤的名字叫 `understand_question`
- 它实际运行的 Python 函数也叫 `understand_question`

### 4.4 `add_edge()` 在做什么？

```python
builder.add_edge(START, "understand_question")
```

这表示图的入口先进入 `understand_question`。

后面这几条边共同定义了线性流：

```python
builder.add_edge("understand_question", "draft_notes")
builder.add_edge("draft_notes", "write_answer")
builder.add_edge("write_answer", END)
```

### 4.5 `compile()` 为什么重要？

```python
graph = builder.compile()
```

你可以把 `compile()` 理解成：

- 检查图结构
- 生成可执行图对象
- 让后续 `invoke()` 或 `stream()` 成为可能

第 2 章先只用 `invoke()`，第 7 章再专门讲 `stream()`。

### 4.6 `invoke()` 为什么是本章最值得跑通的一步

```python
result = graph.invoke({...})
```

这一步代表：

- 你传入初始状态
- LangGraph 按图结构执行节点
- 返回最终状态

如果你连这一步都没真的跑通，只停留在“看懂代码”，那你还没有真正进入 LangGraph。

---

## 五、状态流转 walkthrough：这张图到底让 state 发生了什么？

很多初学者第一次读图代码，只盯着“顺序”。

这还不够。

更重要的是问：

> **每个节点执行之后，state 比刚才多了什么？**

### 5.1 初始状态

```python
{
    "question": "LangGraph 中的 agent memory 应该怎么设计？",
    "notes": "",
    "answer": ""
}
```

### 5.2 经过 `understand_question`

```python
{
    "question": "LangGraph 中的 agent memory 应该怎么设计？",
    "notes": "研究任务：先澄清这个问题的核心范围——LangGraph 中的 agent memory 应该怎么设计？",
    "answer": ""
}
```

这里真正发生的变化是：

- 原始问题仍然保留
- `notes` 从空字符串变成了研究任务描述
- `answer` 还没生成

### 5.3 经过 `draft_notes`

```python
{
    "question": "LangGraph 中的 agent memory 应该怎么设计？",
    "notes": "研究任务：先澄清这个问题的核心范围——LangGraph 中的 agent memory 应该怎么设计？\n- 先找定义\n- 再找代表性论文\n- 最后整理应用场景",
    "answer": ""
}
```

这里的关键不是“又执行了一步”，而是：

- `notes` 被进一步扩展
- state 中已经累积出中间成果

### 5.4 经过 `write_answer`

```python
{
    "question": "LangGraph 中的 agent memory 应该怎么设计？",
    "notes": "...",
    "answer": "你的研究问题是：..."
}
```

到这里，图才真正完成了一个闭环：

- 输入问题
- 形成中间笔记
- 产出交付物

### 5.5 用 ASCII 图把状态演化看得更清楚

```text
初始 state
(question, notes='', answer='')
        │
        ▼
understand_question
新增 / 更新: notes
        │
        ▼
draft_notes
再次更新: notes
        │
        ▼
write_answer
新增 / 更新: answer
        │
        ▼
最终 state
(question, notes='已有内容', answer='最终输出')
```

> [!important] 本章最重要的直觉
> 写 LangGraph 时，最有效的问题不是“下一行该写什么”，而是：
>
> **这个节点执行之后，state 应该新增或更新什么信息？**

---

## 六、把例子升级一点：更像真实研究助手的版本

前面的版本已经能跑，但还比较朴素。我们可以稍微加一点结构，让它更像真正的 Research Copilot。

> [!example]- 结构更清晰的 v0+（点击展开）
>
> ```python
> from typing_extensions import TypedDict
> from langgraph.graph import StateGraph, START, END
>
>
> class ResearchState(TypedDict):
>     question: str
>     clarified_task: str
>     notes: str
>     answer: str
>
>
> def understand_question(state: ResearchState):
>     question = state["question"]
>     return {
>         "clarified_task": f"请围绕“{question}”给出一个面向初学者的研究入门路径。"
>     }
>
>
> def draft_notes(state: ResearchState):
>     task = state["clarified_task"]
>     return {
>         "notes": (
>             f"任务说明：{task}\n"
>             "1. 先解释核心概念\n"
>             "2. 再列关键问题\n"
>             "3. 再给代表性阅读方向"
>         )
>     }
>
>
> def write_answer(state: ResearchState):
>     return {
>         "answer": (
>             f"研究主题：{state['question']}\n\n"
>             f"澄清后的任务：{state['clarified_task']}\n\n"
>             f"建议起步：\n{state['notes']}\n\n"
>             "你可以先做概念梳理，再进入论文阅读。"
>         )
>     }
>
>
> builder = StateGraph(ResearchState)
> builder.add_node("understand_question", understand_question)
> builder.add_node("draft_notes", draft_notes)
> builder.add_node("write_answer", write_answer)
>
> builder.add_edge(START, "understand_question")
> builder.add_edge("understand_question", "draft_notes")
> builder.add_edge("draft_notes", "write_answer")
> builder.add_edge("write_answer", END)
>
> graph = builder.compile()
> ```

### 6.1 这个升级版为什么更好？

因为它开始显式拆出一个中间字段：

- `clarified_task`

这比把一切都塞进 `notes` 更清楚，原因是：

- “问题澄清”本身是一个独立成果
- 后面章节如果接检索工具，会更容易基于 `clarified_task` 发出检索请求
- state 设计越清楚，图就越容易继续长大

> [!tip] 好图往往来自好状态设计
> LangGraph 的很多“高级感”，其实并不来自复杂 API，而来自你是否把状态拆得清楚。

---

## 七、为什么这一章故意先不讲 `MessagesState`

你可能会问：

> “官方不是也经常直接用 `MessagesState` 吗？为什么这里还先自己写 `TypedDict`？”

答案是：

**因为这章要先把最底层的 state 思维讲透。**

如果一开始就直接进入消息对象、消息追加、工具调用，新手很容易把注意力都放在：

- 消息怎么存？
- 消息对象长什么样？
- 为什么有 reducer？

反而忽略了更基础的问题：

- state 到底是什么？
- node 到底更新什么？
- 图到底怎么走？

所以本章先用最朴素的 `TypedDict` state。下一章再升级到：

- `MessagesState`
- `add_messages`
- reducer 思维

这是一条更稳的学习路线：

> **先学抽象骨架，再学常见特化形态。**

---

## 八、常见报错与调试：新手第一次跑图时最容易卡在哪里？

### 8.1 报错一：导入失败

常见现象：

```python
from langgraph.graph import StateGraph, START, END
```

运行时报 `ModuleNotFoundError`。

常见原因：

- 还没安装 `langgraph`
- 没用 `uv run`，而是跑到了错误环境
- 虚拟环境不是当前项目的

排查顺序：

1. 重新确认你在项目目录里
2. 用 `uv add langgraph` 确认依赖已安装
3. 用 `uv run python -c "from langgraph.graph import StateGraph"` 再测一次

### 8.2 报错二：state 缺字段

例如你传入：

```python
result = graph.invoke({
    "question": "..."
})
```

但节点里又访问了：

```python
state["notes"]
```

这就很容易出问题。

解决思路：

- 初始化时把本章约定的字段都给齐
- 或者先检查哪些字段是“后续节点一定会读取”的

### 8.3 报错三：字段名拼错

最常见的低级错误包括：

- `note` vs `notes`
- `answers` vs `answer`
- `clarify_task` vs `clarified_task`

这类问题最适合的预防方式就是：

- 先固定 `TypedDict`
- 节点中围绕同一组字段名写代码

### 8.4 报错四：节点返回值不是字典

错误写法：

```python
def draft_notes(state: ResearchState):
    return state["notes"] + "..."
```

更合理的写法是：

```python
def draft_notes(state: ResearchState):
    return {"notes": state["notes"] + "..."}
```

因为节点应该返回**状态更新字典**，而不是随便返回一个字符串。

### 8.5 报错五：边连错导致流程不完整

例如：

- 少连了一条边
- 写错节点名
- 没接到 `END`

排查思路：

- 先用纸笔写出 `START -> A -> B -> END`
- 再逐条对照 `add_edge()`
- 把“图结构”和“节点函数”分开检查

> [!tip] 调试优先级建议
> 新手第一次跑图时，先按下面顺序排查最有效：
> 1. 导入和环境对不对
> 2. 初始 state 字段全不全
> 3. 节点是否返回字典
> 4. 字段名是否一致
> 5. 边是否连完整

---

## 九、动手练习：这章必须真正写的 3 个练习

### 练习 1：给 state 增加 `topic_scope`

目标：

把原本只用 `notes` 表达任务的方式升级一下，新增一个字段：

```python
topic_scope: str
```

要求：

- 在 `understand_question` 里先填写 `topic_scope`
- 在 `draft_notes` 里读取它
- 最终回答里把它展示出来

你会练到：

- 如何修改 `TypedDict`
- 如何让节点之间通过新增字段协作

### 练习 2：把 3 节点改成 4 节点

新增一个节点：

```text
collect_keywords
```

职责：

- 从问题里提炼 2-3 个“建议检索关键词”
- 先写成简单字符串即可

你会练到：

- 图结构不是固定的
- 节点职责可以逐步细化
- 后面第 4 章接工具时，这类拆分非常重要

### 练习 3：故意制造一个字段拼写错误，再自己修复

例如故意把：

```python
"notes"
```

改成：

```python
"note"
```

然后观察：

- 哪个节点最先出问题
- 你是怎么定位到错误字段名的
- 修复以后图是否恢复正常

你会练到：

- state 驱动调试
- 为什么字段命名约定这么重要

---

## 十、如果你今天只想真正记住一件事

> [!important] LangGraph 的第一性原理
> 先不要把 LangGraph 想成“一个 agent 库”。
>
> 更准确的第一印象应该是：
>
> **一个围绕共享状态运转的流程图执行系统。**
>
> 当你真正接受这个视角后：
> - 为什么要先定义 state
> - 为什么 node 只做一小步
> - 为什么要有 `START` / `END`
> - 为什么后面会自然长出消息、工具、记忆、中断、流式、并行
>
> 这些都会变得顺理成章。

---

## 常见误区

| 误区 | 正确理解 |
|------|----------|
| “LangGraph 的重点是把很多 prompt 串起来。” | 不够准确。真正的重点是**状态驱动的流程组织**。prompt 只是节点里可能发生的一种行为。 |
| “node 应该尽量什么都做，免得图太碎。” | 错。node 更适合单一职责，负责对 state 做一小步清晰更新。 |
| “state 就是普通字典，随便加字段就行。” | 技术上能跑，但工程上很容易混乱。最好先定义清楚结构，比如用 `TypedDict`。 |
| “`START` / `END` 只是形式主义。” | 不是。它们让流程边界显式可见，后续扩展、调试、讲解都会更轻松。 |
| “只有接入 LLM 才算真正的 LangGraph。” | 不对。先把图和状态建清楚，后续再接模型、工具、记忆，才是更稳的学习路径。 |

---

## 思考题

> [!note] 如何使用
> 先自己回答，再看提示和答案。你会更容易发现自己卡在“图结构理解”还是“Python 语法”。

### 基础理解

**Q1. 为什么这一章先定义 `ResearchState`，而不是先写模型调用代码？**

> [!hint]- 💡 提示
> 想一想：如果共享状态里有什么字段都没定义清楚，后面每个节点会发生什么？

> [!success]- ✅ 参考答案
> 因为 LangGraph 的核心是围绕共享状态组织流程。先定义 `ResearchState`，等于先定义“整个流程的工作台上有什么”。这样后面每个节点才能明确：我读取哪些字段、更新哪些字段、依赖哪些已有结果。如果先写模型调用，很容易变成一段黑盒脚本，而不是结构清晰的图工作流。

**Q2. node 为什么通常只返回部分更新，而不是完整 state？**

> [!hint]- 💡 提示
> 想想单一职责和调试可读性。

> [!success]- ✅ 参考答案
> 因为 node 最好只提交自己负责的改动。这样职责更清楚，也更容易调试：你能一眼看出这个节点到底在改什么。如果每个节点都返回完整 state，很容易出现偷偷覆盖字段、职责混乱、后续难以排查的问题。

### 深入思考

**Q3. 如果把 `draft_notes` 和 `write_answer` 强行合并成一个“大节点”，会损失什么？**

> [!hint]- 💡 提示
> 想想：中间结果还能不能单独查看？后面还能不能只替换其中一步？

> [!success]- ✅ 参考答案
> 会损失流程的可解释性和可扩展性。拆成两个节点时，你可以单独看到“研究笔记草稿”这个中间状态，也可以以后只替换 `write_answer` 的逻辑而保留 `draft_notes`。如果合并成一个大节点，中间成果会消失，后续扩展成工具调用、人工审核或并行分工时也会更困难。

**Q4. 为什么说“好图往往来自好状态设计”？**

> [!hint]- 💡 提示
> node 和 edge 是不是都围绕 state 在组织？

> [!success]- ✅ 参考答案
> 因为 node 的职责、图的拆分方式、后续能否并行或暂停，最终都取决于 state 中有哪些字段、这些字段如何被逐步补全。state 设计清楚，节点自然知道自己该负责哪部分；state 设计混乱，节点就容易职责重叠、边也会变得难讲清楚。LangGraph 的很多设计质量，根本上都来自 state schema 的质量。

---

## 关键要点回顾

- ✅ `StateGraph` 是 LangGraph 的核心骨架：围绕 state、node 和 edge 来组织流程
- ✅ state 可以先理解成“共享工作台”或“实验记录本”
- ✅ node 做的是“小步更新 state”，而不是包揽一切
- ✅ `START` / `END` 让图的边界显式化，后续扩展更稳
- ✅ `TypedDict` 是这一阶段最值得掌握的 Python 补课点，它让状态结构更清晰
- ✅ 本章的 Research Copilot v0 是整个系列的项目基线，不是一次性玩具 demo

---

## 扩展阅读

- 📄 [LangGraph Overview](https://docs.langchain.com/oss/python/langgraph/overview) — 官方总览，适合对照本章的最小例子
- 📄 [Quickstart](https://docs.langchain.com/oss/python/langgraph/quickstart) — 可对照图 API 最小 agent 结构，理解 `StateGraph` 的经典构建方式
- 📄 [Graph API](https://docs.langchain.com/oss/python/langgraph/graph-api) — 后续会频繁查阅的底层参考
- 📄 [Streaming](https://docs.langchain.com/oss/python/langgraph/streaming) — 提前预习后面如何观察图执行过程
- 📝 [[research/01-topics/agent-memory|Agent Memory 研究方向]] — 带着真实研究场景理解为什么后面需要记忆、检查点与长任务执行

## 相关页面

- [[raw/lessons/LangGraph/01-langgraph-overview|第 1 章：LangGraph 学习地图与研究助手项目总览]]
- [[raw/lessons/LangGraph/03-state-and-messages|第 3 章：State 与 Messages——让研究助手真正进入对话态]]
- [[wiki/concepts/state-graph|StateGraph]]
- [[research/01-topics/agent-memory|Agent Memory 研究方向总览]]
- [[wiki/concepts/series-navigation-consistency|系列课程导航一致性检查]]

## 下一步学习

下一章我们会从“普通状态字典”进一步升级到对话式 agent 更常见的状态形态：

- 什么是 `MessagesState`
- 为什么消息更新不能只靠普通列表随手拼接
- `add_messages` 在解决什么问题
- Research Copilot 如何从单次流程变成消息驱动的对话助手

继续阅读：[[raw/lessons/LangGraph/03-state-and-messages|第 3 章：State 与 Messages——让研究助手真正进入对话态]]
