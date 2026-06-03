---
type: lesson
tags:
  - LangChain
  - LangChain 1.x
  - Agent
  - Python
  - Abstractions
  - System Design
created: 2026-04-29
updated: 2026-04-29
topic: LangChain 1.x 的分层结构与核心抽象
difficulty: intermediate
prerequisites:
  - "[[raw/lessons/LangChain/01-why-langchain-1-x|第 1 章：为什么 LangChain 1.x 是下一步]]"
  - "[[raw/lessons/LangGraph-to-Harness/02-from-primitives-to-system-modules|第 2 章：从原语到系统模块——LangGraph 能力如何落到真实 Agent 架构]]"
sources:
  - https://docs.langchain.com/oss/python/langchain/overview
  - https://docs.langchain.com/oss/python/langchain/models
  - https://docs.langchain.com/oss/python/langchain/messages
  - https://docs.langchain.com/oss/python/langchain/tools
  - https://docs.langchain.com/oss/python/langchain/agents
  - https://docs.langchain.com/oss/python/langchain/structured-output
status: completed
series:
  name: LangChain 1.x 系统学习
  part: 2
---

# LangChain 1.x 的分层结构与核心抽象

> 一句话摘要：如果第 1 章回答的是“为什么现在该学 LangChain 1.x”，那么这一章回答的就是“它内部到底由哪些核心抽象组成”。你真正要掌握的，不是零散 API，而是一张稳定的组件地图：模型、消息、工具、结构化输出、agent abstraction 各自负责什么，又分别在 Claude Code-like harness 中落到哪里。

## 学习目标

完成本课后，你应该能够：
- [ ] 画出 LangChain 1.x 的最小组件地图
- [ ] 说清 chat model、messages、tools、structured output、agent abstraction 的职责边界
- [ ] 理解什么叫“分层结构”，为什么它比背 API 更重要
- [ ] 区分 LangChain 1.x 负责什么、LangGraph 负责什么、Harness 负责什么
- [ ] 看懂为什么同一个任务会同时经过 message 层、model 层、tool 层和 schema 层
- [ ] 在后续学习中用统一术语描述 agent 组件，而不是临时发明说法

## 前置知识

| 知识点 | 要求 | Wiki 参考 |
|--------|------|-----------|
| LangChain 1.x 定位 | 知道它是标准组件层 | [[raw/lessons/LangChain/01-why-langchain-1-x|第 1 章：为什么 LangChain 1.x 是下一步]] |
| 桥接映射 | 知道 graph 原语会落到真实系统模块 | [[raw/lessons/LangGraph-to-Harness/02-from-primitives-to-system-modules|第 2 章：从原语到系统模块——LangGraph 能力如何落到真实 Agent 架构]] |
| Python 基础 | 能读函数、类、列表、字典、类型标注 | — |
| LLM 基础 | 知道 prompt、response、tool calling 的基本印象 | — |

---

## 直观理解

### 类比一：LangChain 1.x 像“标准化零件仓库”

如果你自己从零拼一个 agent，很快会遇到很多重复问题：

- 模型接法各家不一样怎么办？
- 用户输入到底是字符串，还是消息对象？
- 工具怎样描述给模型？
- 输出怎么变成稳定结构，而不是随意文本？
- 多步任务怎样快速起一个常见 agent 入口？

LangChain 1.x 的核心价值，不是替你发明业务逻辑，而是把这些高频零件做成比较统一的接口。

所以你现在不该把它理解成：

> “一个会不断长新 API 的大库。”

而更应该理解成：

> **一个帮助你标准化 agent 组件的接口层。**

### 类比二：它像一个多层建筑，而不是一个扁平工具箱

很多初学者会把 LangChain 当成“同一层上摆着一堆函数”。

这会导致两个问题：
- 看什么都像孤立功能
- 一换场景就不知道该从哪一层下手

更好的看法是：

```text
用户任务
  │
  ▼
Messages / Context organization
  │
  ▼
Chat Model interface
  │
  ├──► Tool calling
  ├──► Structured output
  └──► Agent abstraction
```

也就是说，LangChain 1.x 不是平铺的“词条集合”，而是一套有层次的组件组织方式。

---

## 一、为什么“分层结构”是你真正该学会的东西？

### 1.1 因为工程系统不是按 API 名称组织的

真实系统不会说：
- 这里放 `invoke()`
- 那里放 `bind_tools()`
- 那里再放几个 message 类

真实系统是按职责组织的：
- 谁负责上下文组织？
- 谁负责模型调用？
- 谁负责工具协议？
- 谁负责结果结构化？
- 谁负责更高层的 agent 行为入口？

如果你只按 API 学，很容易出现这种状态：

> “我知道这些函数存在，但我不知道它们在系统里为什么要一起出现。”

### 1.2 分层思维能直接服务后续 Harness

后面做 Claude Code-like harness 时，你会不断需要回答：

- 这个逻辑属于 provider adapter，还是属于 executor？
- 这是 message assembly 问题，还是 prompt policy 问题？
- 这是 tool schema 的问题，还是 permission layer 的问题？
- 这一步该返回自由文本，还是结构化对象？

这些判断都依赖你现在建立的“分层地图”。

---

## 二、LangChain 1.x 的最小组件地图

下面这张图可以先当作本章的总图。

```text
用户任务
   │
   ▼
Messages / Prompt Context
   │
   ▼
Chat Model Interface
   │
   ├──► Tool Calling
   │        │
   │        ▼
   │     Tool Execution Result
   │
   ├──► Structured Output
   │        │
   │        ▼
   │     Parsed / Validated Object
   │
   └──► Agent Abstraction
            │
            ▼
      Common Agent Loop Entry
```

### 2.1 用一句话记住每层

| 组件 | 一句话作用 |
|------|------------|
| Messages | 把上下文组织成模型能理解、系统也能追踪的消息对象 |
| Chat Model | 提供统一的模型调用入口，隔离 provider 差异 |
| Tools | 把“可执行动作”标准化暴露给模型 |
| Structured Output | 把结果从自由文本收束成可验证对象 |
| Agent Abstraction | 把常见多步行为组织成更高层入口 |

### 2.2 为什么这里没有把 LangGraph 画进去？

因为这一章在讲 LangChain 1.x 自己的组件层。

但你要记住：
- LangChain 1.x 更偏“零件与接口”
- LangGraph 更偏“运行控制与编排”

后面如果你的任务需要：
- checkpoint
- interrupt
- conditional routing
- fan-out / parallel
- subgraph composition

那你就会继续下沉到 LangGraph。

---

## 三、Chat Model：为什么模型接口要被单独看成一层？

### 3.1 因为“模型能力”和“系统能力”不是一回事

在很多新手视角里，所有东西都像是“模型在做”。

但更准确地说：
- 模型负责推理、生成、选择是否调工具
- 系统负责组织消息、提供工具、校验输出、记录状态

如果你不把 model interface 单独看成一层，就容易把系统问题误判成模型问题。

例如：
- 工具调用失败，不一定是模型笨，也可能是工具描述差
- 输出解析失败，不一定是模型差，也可能是 schema 设计差
- 多轮上下文乱掉，不一定是 provider 问题，也可能是 message 组织有问题

### 3.2 这一层在 LangChain 1.x 里主要提供什么？

核心是：
- 统一调用 chat model
- 统一喂入消息对象
- 统一接收 `AIMessage` 结果
- 允许模型绑定工具、结构化输出策略等高层能力

### 3.3 它在 Harness 中落到哪里？

在 Claude Code-like harness 里，这一层通常会落到：
- provider adapter
- model configuration
- inference policy
- token / timeout / retry 配置

换句话说，LangChain 里的“chat model 层”，在 Harness 里会变成“模型接入层”。

---

## 四、Messages：为什么消息对象比字符串重要得多？

### 4.1 因为 agent 的上下文不是“一整段拼好的 prompt”那么简单

你当然可以把所有上下文拼成一个大字符串，但这样很快会遇到问题：
- 用户说了什么？
- 系统规则是什么？
- 上一次 assistant 回答是什么？
- 哪一段是工具执行结果？
- 哪个工具调用对应哪个返回值？

如果全部混成字符串，系统的可解释性和可维护性会迅速下降。

### 4.2 LangChain 1.x 的消息对象给了你什么？

你会经常看到这些对象：
- `SystemMessage`
- `HumanMessage`
- `AIMessage`
- `ToolMessage`

它们的价值不是“面向对象写法更优雅”，而是：

> **把上下文不同角色明确区分开来。**

### 4.3 一个最小消息示例

```python
from langchain.chat_models import init_chat_model
from langchain.messages import SystemMessage, HumanMessage

model = init_chat_model("gpt-5-nano")

messages = [
    SystemMessage("你是一个解释 LangChain 抽象的助教。"),
    HumanMessage("请用通俗语言解释 structured output。"),
]

response = model.invoke(messages)
```

### 4.4 这段 Python 在说什么？

先看整体结构：
- `init_chat_model(...)` 初始化一个 chat model 入口
- `messages` 是一个消息对象列表
- `model.invoke(messages)` 把这些消息交给模型

再拆关键语法：

#### `from ... import ...`
这是 Python 的导入语句。
它的意思是：
- 从 `langchain.chat_models` 导入 `init_chat_model`
- 从 `langchain.messages` 导入消息类

#### 列表 `[]`
`messages = [...]` 表示把多条消息按顺序装进一个列表。
在 agent 系统里，列表顺序很重要，因为它决定上下文时间顺序。

#### 函数调用 `model.invoke(messages)`
`invoke` 可以先理解成：“把一组输入交给模型执行一次调用”。

### 4.5 它在 Harness 中落到哪里？

- 对话历史组织
- prompt assembly
- tool result 回写
- 可观测执行记录

也就是说，messages 在 LangChain 中是标准上下文对象；在 Harness 中，它会成为 conversation state 的核心组成部分。

---

## 五、Tools：为什么它不是“顺便加个函数”那么简单？

### 5.1 工具的本质，是把系统动作暴露给模型

只要你让模型调工具，本质上就在做一件大事：

> 允许语言模型不只“说”，还可以“请求系统做事”。

这会立刻带来工程问题：
- 工具怎么描述？
- 参数怎么约束？
- 执行失败怎么办？
- 结果如何回写？
- 哪些工具允许直接调用，哪些要过权限门？

### 5.2 所以工具层其实是协议层

在 LangChain 1.x 里，tools 不是一个“小插件功能”，而是一个协议：
- 给模型看见工具名和说明
- 告诉模型参数 schema
- 模型返回 `tool_calls`
- 系统执行工具
- 再把结果通过 `ToolMessage` 喂回模型

### 5.3 这件事在 Harness 中怎么落地？

在 Claude Code-like harness 里，tool 层通常会进一步拆成：
- tool registry
- execution adapter
- permission policy
- error handling
- logging / tracing

所以你以后不要只问：

> “这个工具能不能被调起来？”

还要问：

> “这个工具在系统里是不是一个安全、可观察、可恢复的动作单元？”

---

## 六、Structured Output：为什么它是“工程抽象”，不是“格式小技巧”？

### 6.1 自由文本很强，但不总够用

自然语言输出的优点是灵活。
但在 agent 工程里，很多结果并不是“给人看一眼就结束”，而是还要被系统继续消费。

例如：
- 计划对象
- 风险分析对象
- 任务分类结果
- 审查意见
- 变更摘要
- 下一步动作建议

如果这些都只是模糊的一段文本，那么后续：
- 很难自动校验
- 很难稳定复用
- 很难做 trace / replay
- 很难做 reviewer 二次处理

### 6.2 Structured Output 真正解决什么？

它要解决的是：

> **如何让模型输出变成可解析、可验证、可组合的数据对象。**

### 6.3 在 LangChain 1.x 里它通常和哪些东西关联？

- schema
- Pydantic / dataclass / TypedDict / JSON Schema
- provider-native structured output
- tool strategy

这些术语你暂时不必一次记全，但要先记住一个大方向：

> 模型输出不一定只是自然语言，它也可以是带结构约束的对象。

### 6.4 它在 Harness 中落到哪里？

- planner 的 plan object
- reviewer 的 review result
- permission checker 的 decision object
- summary generator 的 structured report

这也是为什么后面做 coding agent 时，structured output 会越来越重要。

---

## 七、Agent Abstraction：为什么它是“高层入口”，不是“万能黑箱”？

### 7.1 高层 agent 抽象的意义

LangChain 1.x 不只是给你基础零件，也提供更高层的 agent 入口。

它的价值通常在于：
- 帮你快速起常见 agent loop
- 把 model / tools / structured output 先装配成可用骨架
- 让你先站在更高层看到典型模式

### 7.2 但它不等于“系统已经设计好了”

高层 agent abstraction 不能替代你思考：
- 权限边界
- checkpoint / resume
- 日志与回放
- 风险动作审批
- 多模块职责划分

所以正确心态是：
- 把它当作常见模式入口
- 不把它当作真实系统设计的终点

### 7.3 在 Harness 中它通常会长成什么？

它会成为：
- planner-executor skeleton
- task loop seed
- higher-level orchestration starting point

也就是说，高层 abstraction 常常是原型起点，而不是成熟系统的全部。

---

## 八、一张“它解决什么 / 不解决什么 / 落到哪里”的总对照表

| 抽象 | 它解决什么 | 它不解决什么 | 在 Harness 中落到哪 |
|------|------------|--------------|----------------------|
| Chat Model | 统一模型调用接口 | 不负责整个任务系统 | provider / model adapter |
| Messages | 标准化上下文表示 | 不直接决定流程控制 | conversation state / prompt assembly |
| Tools | 标准化模型可调用动作 | 不自动解决安全与权限 | tool registry / executor |
| Structured Output | 让结果可验证 | 不自动替你设计好 schema | plan/review/summary objects |
| Agent Abstraction | 快速起常见 agent 入口 | 不替代完整工程架构 | execution skeleton / prototype layer |

这张表非常重要，因为它能防止你对 LangChain 1.x 产生两种相反误解：
- 要么把它神化成“什么都能包办”
- 要么把它轻视成“只是语法糖”

更准确的理解是：

> **它负责统一高频组件与常见入口，但系统边界、运行控制和产品工程仍然需要你自己设计。**

---

## 九、Python 补课：为什么这一章开始会频繁出现 dataclass、TypedDict、Pydantic、schema 思维？

### 9.1 因为你开始从“文本编程”进入“对象编程”

到了这一阶段，你经常不再只是要一段句子，而是要一类对象：
- 一个计划对象
- 一个工具参数对象
- 一个任务分类对象
- 一个审查结果对象

这时“结构稳定的数据表示”就很关键。

### 9.2 一个最小例子

```python
from dataclasses import dataclass
from enum import Enum


class OutputMode(str, Enum):
    FREE_TEXT = "free_text"
    STRUCTURED = "structured"


@dataclass
class TaskSummary:
    title: str
    risk: str
    output_mode: OutputMode
```

### 9.3 这段代码想表达什么？

- `TaskSummary` 是一个有固定字段的数据对象
- `output_mode` 不是任意字符串，而是有限合法值
- 系统看到这个对象后，更容易决定下一步流程

### 9.4 为什么这和 LangChain 抽象直接相关？

因为后面：
- tools 依赖参数 schema
- structured output 依赖结果 schema
- agent loop 依赖可传递对象

所以你现在看到的这些 Python 结构，不是旁枝末节，而是在为后续系统工程铺路。

---

## 十、LangChain 1.x、LangGraph、Harness 的边界再收一次

这是本章必须反复强调的重点。

### 10.1 LangChain 1.x 更偏什么？

更偏：
- 标准组件层
- 模型接口
- 消息对象
- 工具协议
- 结构化输出
- 常见 agent 高层入口

### 10.2 LangGraph 更偏什么？

更偏：
- runtime control
- stateful execution
- checkpoint / resume
- interrupt
- conditional routing
- parallel / subgraph

### 10.3 Harness 更偏什么？

更偏：
- session management
- planner / executor / reviewer 模块
- permission gate
- logging / replay / verification
- CLI / UX / product workflow

### 10.4 一句话总收束

> **LangChain 1.x 给你标准零件，LangGraph 给你运行控制，Harness 负责把它们装配成真实系统。**

---

## 常见误区

| 误区 | 正确理解 |
|------|----------|
| “LangChain 1.x 就是一堆 API 名称。” | 更准确地说，它是一组稳定的组件抽象与高层入口。 |
| “只要会 `invoke()` 和 `bind_tools()` 就算学会了。” | 还不够。你需要知道这些能力分别处在哪一层，以及它们如何进入系统。 |
| “Messages 只是把字符串包装一下。” | 不对。消息对象是在给上下文分角色、分责任、分时间顺序。 |
| “Structured output 只是输出 JSON 更好看。” | 错。它的核心价值是让结果可验证、可复用、可组合。 |
| “LangChain agent abstraction 可以替代系统设计。” | 不行。它可以帮你快速起步，但权限、恢复、验证、审计等仍属于系统工程。 |

---

## 思考题

### 基础理解

**Q1. 为什么说 LangChain 1.x 更应该被理解成“组件地图”，而不是“API 词典”？**

> [!hint]- 💡 提示
> 想想系统设计时最重要的是函数名，还是职责边界。

> [!success]- ✅ 参考答案
> 因为真实系统是按职责而不是按函数名组织的。LangChain 1.x 的价值在于把模型、消息、工具、结构化输出和 agent abstraction 统一成稳定抽象，帮助你理解它们各自负责什么、彼此如何配合，以及后续怎样进入 Harness，而不是只让你记住零散 API。

**Q2. Messages 相比“把所有内容拼成一个大字符串”好在哪里？**

> [!hint]- 💡 提示
> 想想系统消息、用户消息、工具结果是不是同一种东西。

> [!success]- ✅ 参考答案
> Messages 的优势在于把上下文按角色和时间顺序明确分开，系统规则、用户请求、模型输出、工具结果都能被区分和追踪。这使得上下文更可解释，也更适合多步 agent 系统继续处理。

### 深入思考

**Q3. 为什么 structured output 对 Claude Code-like harness 特别重要？**

> [!hint]- 💡 提示
> 一个计划、审查意见或风险判断，是否只适合用自然语言模糊描述？

> [!success]- ✅ 参考答案
> 不适合。Claude Code-like harness 中很多中间结果和最终结果都需要被系统继续消费，例如计划对象、风险对象、代码审查对象、任务总结对象。structured output 让这些结果更稳定、更可验证，也更便于后续自动处理、记录和回放。

**Q4. 为什么高层 agent abstraction 不能替代你理解底层边界？**

> [!hint]- 💡 提示
> 高层入口帮你更快开始，但它会不会自动解决权限、恢复和验证问题？

> [!success]- ✅ 参考答案
> 不能。高层 abstraction 主要是帮你快速起一个常见 agent 骨架，但真实系统还需要你自己设计权限边界、checkpoint / resume、日志、验证、风险审批和模块分工。如果不理解底层边界，就很容易把高层入口误当成完整系统。

---

## 关键要点回顾

- ✅ LangChain 1.x 的核心价值是提供稳定的组件抽象，而不是单纯堆 API
- ✅ 最关键的五类抽象是：chat model、messages、tools、structured output、agent abstraction
- ✅ 分层思维比背函数名更重要，因为真实系统是按职责组织的
- ✅ Messages 让上下文更可解释，tools 让动作可协议化，structured output 让结果可验证
- ✅ 高层 agent abstraction 是原型入口，不是完整工程系统
- ✅ 在长期目标里，LangChain 1.x 是标准零件层，而不是最终整机

---

## 扩展阅读

- 📄 [LangChain Overview](https://docs.langchain.com/oss/python/langchain/overview) — 总览 LangChain 1.x 在现代 agent engineering 中的位置
- 📄 [LangChain Models](https://docs.langchain.com/oss/python/langchain/models) — 理解 chat model 接口和结构化输出入口
- 📄 [LangChain Messages](https://docs.langchain.com/oss/python/langchain/messages) — 理解消息对象体系和 tool message 协议
- 📄 [LangChain Tools](https://docs.langchain.com/oss/python/langchain/tools) — 理解工具定义与模型可调用动作
- 📄 [LangChain Agents](https://docs.langchain.com/oss/python/langchain/agents) — 观察高层 agent abstraction 的定位
- 📄 [LangChain Structured Output](https://docs.langchain.com/oss/python/langchain/structured-output) — 看 schema、provider strategy 与 tool strategy 的正式说明

## 相关页面

- [[raw/lessons/LangChain/01-why-langchain-1-x|第 1 章：为什么 LangChain 1.x 是下一步]]
- [[raw/lessons/LangChain/03-models-messages-and-provider-boundaries|第 3 章：Models、Messages 与 Provider Boundaries]]
- [[raw/lessons/LangChain/04-tools-and-structured-output|第 4 章：Tools 与 Structured Output——让结果可调用、可验证、可组合]]
- [[raw/lessons/LangGraph-to-Harness/02-from-primitives-to-system-modules|第 2 章：从原语到系统模块——LangGraph 能力如何落到真实 Agent 架构]]
- [[raw/lessons/Agent-Engineering/02-harness-mvp-architecture|第 2 章：Harness MVP 架构——一个最小 coding agent 系统怎么拆]]

## 下一步学习

接下来最自然的推进有两步，而且最好连着看：

1. 先去 [[raw/lessons/Agent-Engineering/02-harness-mvp-architecture|第 2 章：Harness MVP 架构——一个最小 coding agent 系统怎么拆]]，把这些抽象真正放进系统模块图里
2. 再去 [[raw/lessons/LangChain/03-models-messages-and-provider-boundaries|第 3 章：Models、Messages 与 Provider Boundaries]]，把模型接口与消息协议拆得更细

这样你就不会只停留在“知道有这些组件”，而会继续进入“这些组件在系统里如何各司其职”。
