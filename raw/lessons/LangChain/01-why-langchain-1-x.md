---
type: lesson
tags:
  - LangChain
  - LangChain 1.x
  - Agent
  - Python
  - Structured Output
  - Tool Calling
  - Messages
created: 2026-04-29
updated: 2026-04-29
topic: 为什么 LangChain 1.x 是下一步
difficulty: intermediate
prerequisites:
  - "[[raw/lessons/LangGraph-to-Harness/01-from-graph-to-runtime|第 1 章：从 Graph 到 Runtime——为什么下一步是 LangChain 1.x 与 Harness]]"
sources:
  - https://docs.langchain.com/oss/python/langchain/overview
  - https://docs.langchain.com/oss/python/langchain/agents
  - https://docs.langchain.com/oss/python/langchain/messages
  - https://docs.langchain.com/oss/python/langchain/models
  - https://docs.langchain.com/oss/python/langchain/structured-output
  - https://docs.langchain.com/oss/python/langgraph/overview
status: completed
series:
  name: LangChain 1.x 系统学习
  part: 1
---

# 为什么 LangChain 1.x 是下一步

> 一句话摘要：学完 LangGraph 后，下一步不是继续堆更多图能力，而是补上现代 agent engineering 的“标准组件层”。LangChain 1.x 的价值，不在于替你思考，而在于让你用统一抽象去组织模型、消息、工具、结构化输出和常见 agent loop。

## 学习目标

完成本课后，你应该能够：
- [ ] 理解为什么后续课程必须坚持 **LangChain 1.x** 主线
- [ ] 说清 LangChain 1.x 与 LangGraph 的分工关系
- [ ] 理解 LangChain 1.x 对模型、消息、工具、结构化输出做了哪些统一抽象
- [ ] 知道什么情况下应该直接用 LangChain 1.x，什么情况下应该下沉到 LangGraph 或更底层实现
- [ ] 建立“标准组件层”思维，而不是把 LangChain 当成单纯的 API 集合
- [ ] 看懂后续为什么它会成为 Claude Code-like harness 的重要零件库

## 前置知识

| 知识点 | 要求 | Wiki 参考 |
|--------|------|-----------|
| LangGraph runtime 直觉 | 知道状态、路由、工具回路在做什么 | [[raw/lessons/LangGraph-to-Harness/01-from-graph-to-runtime|第 1 章：从 Graph 到 Runtime——为什么下一步是 LangChain 1.x 与 Harness]] |
| Tool Calling | 知道模型可以决定是否调用工具 | [[raw/lessons/LangGraph/04-tools-and-react|第 4 章：工具与 ReAct——让研究助手学会调用检索工具]] |
| Python 基础 | 能读函数、列表、字典、类型标注 | — |
| LLM 基础印象 | 知道 model / prompt / response 的基本关系 | — |

---

## 直观理解

### 类比一：LangChain 1.x 像“agent 工程的标准接口层”

如果你完全不用框架，后面很快会碰到一堆重复问题：

- 不同模型供应商的接口不一样怎么办？
- 消息到底是字符串、对象还是字典？
- 工具定义如何暴露给模型？
- 结构化输出怎么保证格式稳定？
- 同一个任务是直接问答还是 agent loop？

LangChain 1.x 的价值，就是把这些高频问题统一成比较稳定的抽象。

它不是替你做全部决策，而是先把“零件接口”标准化。

### 类比二：LangGraph 更像“流程控制板”，LangChain 1.x 更像“零件箱”

- LangGraph 关注：流程怎么走、状态怎么传、什么时候中断、怎么恢复、怎么并行
- LangChain 1.x 关注：模型怎么接、消息怎么表示、工具怎么描述、结果怎么结构化、常见 agent loop 怎么快速搭

所以最好的理解方式不是二选一，而是：

> **LangChain 1.x 管通用零件，LangGraph 管复杂编排。**

---

## 一、为什么必须明确强调“1.x”而不是泛泛说 LangChain？

### 1.1 因为历史内容太多，版本混学会严重干扰理解

LangChain 有很长一段发展历史，所以互联网上很容易搜到：

- 老教程
- 老概念命名
- 老 API 写法
- 老式链条拼装思路

如果不明确限定版本，很容易出现一种学习幻觉：

> “我好像看了很多 LangChain 内容，但总感觉体系是碎的。”

这不是你的问题，而是版本上下文混杂导致的。

所以后续我们会坚持：

- 以 **LangChain 1.x 文档体系** 为主线
- 以现代 agent engineering 视角来组织学习
- 不把旧时代零散教程当成主教材

### 1.2 为什么 1.x 特别适合你现在这个阶段？

因为你已经不是完全初学者了。

你已经学过 LangGraph，所以你现在最需要的是：

- 一个上层视角，帮助你把零件认识完整
- 一个现代接口层，帮助你理解常见 agent 能力如何被标准化
- 一个和后续 harness 工程能自然衔接的知识结构

而 LangChain 1.x 恰好适合这个位置。

---

## 二、LangChain 1.x 到底在系统里解决什么问题？

### 2.1 它不主要解决“模型更聪明”，而是解决“工程接口更统一”

这点很重要。

LangChain 1.x 的核心价值不是：
- 自动让模型回答更神奇

而是：
- 帮你把常见 agent 组件抽象清楚
- 降低系统拼装时的接口摩擦
- 让你更专注于系统设计，而不是每次都从底层协议重新拼

### 2.2 它统一的几类关键东西

#### A. 模型抽象
你可以用统一方式接不同 provider 的 chat model。

这对后面做 harness 很重要，因为你不希望整个系统被某一个模型供应商锁死。

#### B. 消息抽象
不是简单的字符串，而是更明确的消息结构。

这让你更容易处理：
- `SystemMessage`
- `HumanMessage`
- `AIMessage`
- `ToolMessage`

#### C. 工具抽象
LangChain 1.x 让“工具是什么、工具参数是什么、模型怎么知道它存在”这件事更标准化。

#### D. 结构化输出
这是后面做 coding agent 非常关键的部分。

因为很多时候你不想只要一段自然语言，而想要：
- JSON 风格结果
- schema 约束的输出
- 明确字段的任务对象

#### E. 常见 agent 架构
它提供高层 agent 入口，帮助你快速搭常见模式，而不用每次从零开始组织 loop。

---

## 三、LangChain 1.x 和 LangGraph 到底是什么关系？

### 3.1 最容易记住的版本

你可以先把两者关系记成：

- **LangChain 1.x**：标准组件层
- **LangGraph**：流程编排层

### 3.2 为什么很多人会混淆？

因为两者都和 agent 有关，而且经常一起出现。

但它们关注的问题并不完全一样：

| 维度 | LangChain 1.x | LangGraph |
|------|---------------|-----------|
| 关注点 | 通用组件与高层 agent 抽象 | 状态图、路由、运行控制 |
| 更像 | 标准零件与常见装配件 | 工作流引擎 |
| 强项 | model / tools / messages / structured output / prebuilt agent | checkpoint / interrupt / conditional routing / parallel / subgraph |
| 适合场景 | 标准化 agent 入口、快速上手常见模式 | 复杂多步流程、长任务、恢复、组合 |

### 3.3 为什么这两者对你都重要？

因为你最终不是要写单个 demo，而是要做 Claude Code-like harness。

那种系统既需要：
- LangChain 1.x 带来的统一零件层

也需要：
- LangGraph 带来的可控 runtime 层

所以后续最健康的学习姿势是：

> **先用 LangChain 1.x 认清常用零件，再用 LangGraph 控制复杂运行流。**

---

## 四、整体流程 / 架构图

### 4.1 LangChain 1.x 的核心组件地图

```text
用户任务
   │
   ▼
Messages ─────► Chat Model ─────► AIMessage
   │                │                 │
   │                ├──── tool_calls ─┘
   │                │
   │                └──── structured output
   │
   ▼
Tools / Schemas ──► Tool execution ──► ToolMessage ──► 再喂回模型
```

这张图想表达的是：

- **messages** 负责上下文组织
- **chat model** 负责推理与决定是否调用工具
- **tools** 负责真实动作
- **ToolMessage** 负责把动作结果回写给模型
- **structured output** 负责把结果变成稳定对象，而不是模糊文本

### 4.2 一张“它是什么 / 它不是什么 / 它落到哪一层”的对照表

| 组件 | 它是什么 | 它不是什么 | 在 Harness 中落到哪一层 |
|------|----------|------------|--------------------------|
| Chat model | 统一的模型调用入口 | 不是完整系统 | model adapter / provider layer |
| Messages | 标准消息对象体系 | 不是纯字符串数组 | conversation state / prompt assembly |
| Tools | 给模型可调用的动作能力 | 不是随便暴露任意函数 | tool registry / executor |
| Structured output | 用 schema 约束输出结果 | 不是只是“让 JSON 更好看” | plan objects / review objects / task summaries |
| Agent abstraction | 常见 agent loop 的高层入口 | 不是替代底层理解 | planner-executor skeleton |

---

## 五、Python 补课：这一章里为什么 `type hints` 和 schema 思维突然重要？

### 5.1 `type hints` 是什么？

先看一个最小例子：

```python
def search_docs(query: str) -> str:
    return f"results for {query}"
```

这里的 `query: str` 和 `-> str` 就叫类型标注（type hints）。

它们的意思分别是：
- `query: str`：这个参数应该是字符串
- `-> str`：这个函数预计返回字符串

### 5.2 为什么在 agent 工程里它比普通脚本更重要？

因为你后面会频繁处理：
- 工具参数
- 结构化输出字段
- schema 约束
- 任务对象
- 状态对象

这些如果没有清晰类型边界，很快就会混乱。

### 5.3 什么叫 schema 思维？

所谓 schema，可以先简单理解成：

> **提前规定一个数据结构应该长什么样。**

例如：
- 一个任务对象必须有 `title`、`steps`、`risk_level`
- 一个 structured output 必须有 `summary`、`files`、`next_action`

在 Claude Code-like harness 里，这种思维非常关键，因为 coding agent 不能永远只吐一段模糊自然语言。

### 5.4 一个更具体的 Python 例子：把“自由文本计划”变成“可验证计划对象”

```python
from dataclasses import dataclass
from enum import Enum


class RiskLevel(str, Enum):
    LOW = "low"
    MEDIUM = "medium"
    HIGH = "high"


@dataclass
class PlanItem:
    title: str
    reason: str
    risk: RiskLevel
```

这段代码想表达什么？

- `PlanItem` 不是随便一段文字，而是**固定字段的数据结构**
- `risk` 不是任意文本，而是有限合法值
- 后面如果你要把计划展示给用户、写进日志、交给 reviewer，它就更稳定

换句话说：

> schema 思维本质上是在问：这个结果以后还要不要被系统继续使用？如果要，就别只用模糊自然语言。

---

## 六、LangChain 1.x 最值得你学的，不是“所有功能”，而是这些关键能力

### 6.1 Messages
后面你会系统学习：
- 为什么 messages 不只是字符串列表
- 为什么 system / user / assistant / tool 消息要区别对待
- 为什么这会直接影响 runtime 可解释性

### 6.2 Tools
这部分和你在 LangGraph 第 4 章的知识可以自然接上。

但现在你会更进一步理解：
- 工具是怎样被描述给模型的
- 为什么工具接口设计会影响调用成功率
- 工具 schema 为什么重要

### 6.3 Structured Output
这是你后续做 harness 时必须认真掌握的一块。

因为很多 agent 中间状态和最终结果，都更适合结构化表达，而不是自由文本。

例如：
- 计划结果
- 风险分析
- 变更摘要
- 代码审查意见
- 任务分类结果

### 6.4 Agent abstraction
LangChain 1.x 的一个重要作用，就是帮你更快构建常见 agent loop。

但请记住：

> 高层 abstraction 的价值，是帮助你看清常见模式，不是替代你理解底层系统结构。

### 6.5 Observability 周边思维
虽然 tracing / callbacks / eval 这些内容后面会单讲，但你现在要先知道：

真正的 agent 工程，不只关心“结果是什么”，还关心：
- 过程是什么
- 为什么失败
- 是否稳定
- 能否回归测试

---

## 七、一个最小工具回路示例：为什么 ToolMessage 很关键？

官方文档里的核心思想其实很朴素：

1. 模型先读用户消息
2. 如果判断需要工具，就返回一个带 `tool_calls` 的 `AIMessage`
3. 你执行对应工具
4. 再把工具结果封装成 `ToolMessage` 喂回模型
5. 模型基于工具结果继续回答

下面是精简版示意：

```python
from langchain.messages import HumanMessage, AIMessage, ToolMessage


messages = [HumanMessage(content="帮我查询 README 里有没有安装步骤")]

ai_message = AIMessage(
    content=[],
    tool_calls=[{
        "name": "read_file",
        "args": {"path": "README.md"},
        "id": "call_1",
    }],
)

messages.append(ai_message)

messages.append(
    ToolMessage(
        content="README.md 中包含安装步骤：先 uv sync，再 uv run app.py",
        tool_call_id="call_1",
    )
)
```

### 7.1 这段代码在说什么？

- `HumanMessage` 表示用户意图
- `AIMessage` 表示模型的判断结果：它决定要调工具
- `tool_calls` 是模型给出的“动作建议”
- `ToolMessage` 是工具执行结果回写

### 7.2 为什么这很重要？

因为它让你第一次真正看见：

> 模型不是直接“知道答案”，而是通过消息协议和工具协议参与任务完成。

### 7.3 它在 Harness 中落到哪里？

- LangChain 里，它是标准消息与工具调用协议
- Harness 里，它会落成：
  - tool registry
  - executor
  - tool result log
  - step trace

---

## 八、Structured Output：provider-native 与 tool strategy 有什么区别？

官方文档把结构化输出大致分成两条路：

### 8.1 Provider-native structured output
如果某些模型供应商原生支持结构化输出，那么可以直接走 provider-native 路线。

它的优点通常是：
- 输出更稳定
- 约束更强
- 结果更接近“真正 schema 对象”

### 8.2 Tool calling strategy
如果模型没有原生结构化输出，或者原生能力不稳定，LangChain 可以用 **tool calling strategy** 来实现结构化结果。

本质上是：
- 把“输出某个结构对象”伪装成一次工具调用
- 让模型按工具参数 schema 来填写结果

### 8.3 为什么这点对你特别重要？

因为你做 harness 时会不断遇到这样的问题：

- 我是应该相信模型自由文本？
- 还是要求它产出可验证字段？
- 如果底层 provider 不支持原生 schema，我该怎么办？

LangChain 1.x 的价值就在这里：

> 它不仅告诉你“结构化输出很重要”，还给你提供跨 provider 的落地策略。

---

## 九、什么情况下用 LangChain 1.x，什么情况下继续下沉？

### 9.1 更适合先用 LangChain 1.x 的场景

当你需要：
- 快速接一个 chat model
- 标准化 messages
- 暴露工具给模型
- 做结构化输出
- 起一个常规 agent 原型

这时 LangChain 1.x 很合适。

### 9.2 更适合下沉到 LangGraph 的场景

当你需要：
- 多阶段条件路由
- checkpoint / resume
- human-in-the-loop
- 多分支并行
- 子图组合
- 明确状态机式运行控制

这时 LangGraph 更合适。

### 9.3 更适合继续下沉到你自己的 Harness 的场景

当你开始关心：
- CLI 与交互面
- session 持久化
- 权限系统
- 风险动作审批
- 任务历史
- 回放与审查
- 多 agent 分工
- 工程级验证

这时你就进入 harness 设计层了。

---

## 十、这一章对你的长期目标有什么意义？

你的目标不是“会写几个 LangChain 例子”，而是：

- 学会现代 agent engineering 的上层抽象
- 把这些抽象转成 Claude Code-like harness 的工程零件
- 后面逐步做出一个可以运行、可以扩展、可以验证的系统

所以本章最大的意义是：

> **把 LangChain 1.x 从“一个热门库”重新定义成“你后续系统设计中的零件标准层”。**

---

## 常见误区

| 误区 | 正确理解 |
|------|----------|
| “LangChain 就是帮我少写几行代码的库。” | 不够准确。它更重要的价值是统一模型、消息、工具、结构化输出和常见 agent 抽象。 |
| “既然学了 LangGraph，就不需要 LangChain。” | 不对。LangGraph 偏 runtime 控制，LangChain 1.x 偏标准组件层，两者是互补关系。 |
| “LangChain 的版本无所谓，能跑就行。” | 对系统学习来说不行。必须坚持 1.x 主线，避免历史包袱干扰。 |
| “高层 agent abstraction 会让我不必理解底层。” | 恰恰相反。越想做可靠系统，越要知道高层 abstraction 省掉了什么、遮住了什么。 |
| “Structured output 只是可有可无的小优化。” | 对 coding agent 来说很关键。很多计划、任务、审查结果都更适合结构化表达。 |

---

## 思考题

### 基础理解

**Q1. 为什么后续课程必须明确坚持 LangChain 1.x，而不能随便混用旧教程？**

> [!hint]- 💡 提示
> 想想历史 API 和现代文档的差异会给学习结构带来什么问题。

> [!success]- ✅ 参考答案
> 因为 LangChain 历史内容很多，旧教程和旧 API 很容易把学习者带进过时抽象，导致体系碎裂。坚持 1.x 主线，能保证你始终站在现代 agent engineering 视角下学习模型、消息、工具和结构化输出。

**Q2. LangChain 1.x 和 LangGraph 最简洁的分工理解是什么？**

> [!hint]- 💡 提示
> 一个偏零件层，一个偏流程层。

> [!success]- ✅ 参考答案
> 最简洁的理解是：LangChain 1.x 偏标准组件层，负责模型、消息、工具、结构化输出和高层 agent 抽象；LangGraph 偏流程编排层，负责状态、路由、中断、恢复、并行和组合。

### 深入思考

**Q3. 为什么说 schema 思维对 Claude Code-like harness 很重要？**

> [!hint]- 💡 提示
> 一个 coding agent 的中间结果和最终结果，真的总适合自由文本吗？

> [!success]- ✅ 参考答案
> 不适合总用自由文本。Claude Code-like harness 需要处理计划、任务、风险分析、变更摘要、审查意见等很多中间对象。schema 思维可以让这些结果更稳定、更可验证，也更方便后续自动处理和回放。

**Q4. 为什么学习 LangChain 1.x 不应该停留在“会跑 demo”？**

> [!hint]- 💡 提示
> 想想你的最终目标是什么。

> [!success]- ✅ 参考答案
> 因为你的目标不是单次演示，而是构建一个长期演进的 Claude Code-like harness。只有把 LangChain 1.x 看成标准组件层，理解它统一了哪些接口、适合哪些场景、什么时候需要下沉，才能真正服务后续系统设计。

---

## 关键要点回顾

- ✅ 后续必须明确坚持 **LangChain 1.x** 主线，避免旧版本历史包袱
- ✅ LangChain 1.x 的核心价值不是“更聪明”，而是“更统一”
- ✅ 它最值得你学的是 model、messages、tools、structured output、agent abstraction
- ✅ LangChain 1.x 与 LangGraph 的关系是互补而不是替代
- ✅ 对 Claude Code-like harness 来说，schema 与结构化输出是核心工程能力之一
- ✅ 学 LangChain 1.x 的目标不是背 API，而是建立标准组件层视角

---

## 扩展阅读

- 📄 [LangChain Overview](https://docs.langchain.com/oss/python/langchain/overview) — 官方总览，理解 LangChain 1.x 的现代定位
- 📄 [LangChain Agents](https://docs.langchain.com/oss/python/langchain/agents) — 看高层 agent 如何组织模型、工具与结构化输出
- 📄 [LangChain Messages](https://docs.langchain.com/oss/python/langchain/messages) — 理解 `HumanMessage` / `AIMessage` / `ToolMessage` 的语义差异
- 📄 [LangChain Models](https://docs.langchain.com/oss/python/langchain/models) — 观察模型绑定工具与调用回路的标准形式
- 📄 [LangChain Structured Output](https://docs.langchain.com/oss/python/langchain/structured-output) — 对比 provider-native 与 tool strategy
- 📄 [LangGraph Overview](https://docs.langchain.com/oss/python/langgraph/overview) — 对照理解 LangGraph 与 LangChain 的边界

## 相关页面

- [[raw/lessons/LangGraph-to-Harness/01-from-graph-to-runtime|第 1 章：从 Graph 到 Runtime——为什么下一步是 LangChain 1.x 与 Harness]]
- [[raw/lessons/LangChain/02-langchain-1-x-core-abstractions|第 2 章：LangChain 1.x 的分层结构与核心抽象]]
- [[raw/lessons/LangChain/03-models-messages-and-provider-boundaries|第 3 章：Models、Messages 与 Provider Boundaries]]
- [[raw/lessons/LangChain/04-tools-and-structured-output|第 4 章：Tools 与 Structured Output——让结果可调用、可验证、可组合]]
- [[raw/lessons/Agent-Engineering/01-claude-code-like-systems-overview|第 1 章：Claude Code-like 系统总览]]
- [[raw/lessons/Agent-Engineering/02-harness-mvp-architecture|第 2 章：Harness MVP 架构——一个最小 coding agent 系统怎么拆]]
- [[raw/lessons/LangGraph/04-tools-and-react|第 4 章：工具与 ReAct——让研究助手学会调用检索工具]]

## 下一步学习

下一篇开始把视角从“框架定位”转向“组件地图”：

- LangChain 1.x 到底分哪些层？
- model / messages / tools / structured output 怎么彼此配合？
- 哪些抽象应该复用，哪些边界需要你自己掌握？

继续阅读：[[raw/lessons/LangChain/02-langchain-1-x-core-abstractions|第 2 章：LangChain 1.x 的分层结构与核心抽象]]
