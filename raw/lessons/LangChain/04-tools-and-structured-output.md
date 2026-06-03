---
type: lesson
tags:
  - LangChain
  - LangChain 1.x
  - Tools
  - Structured Output
  - Python
  - Agent
  - Schema
created: 2026-04-29
updated: 2026-04-29
topic: Tools 与 Structured Output——让结果可调用、可验证、可组合
difficulty: intermediate
prerequisites:
  - "[[raw/lessons/LangChain/02-langchain-1-x-core-abstractions|第 2 章：LangChain 1.x 的分层结构与核心抽象]]"
  - "[[raw/lessons/LangChain/03-models-messages-and-provider-boundaries|第 3 章：Models、Messages 与 Provider Boundaries]]"
  - "[[raw/lessons/Agent-Engineering/03-planner-executor-reviewer-loop|第 3 章：Planner / Executor / Reviewer——受控执行回路是怎样形成的]]"
sources:
  - https://docs.langchain.com/oss/python/langchain/tools
  - https://docs.langchain.com/oss/python/langchain/messages
  - https://docs.langchain.com/oss/python/langchain/structured-output
  - https://docs.langchain.com/oss/python/langchain/agents
  - https://docs.langchain.com/oss/python/langchain/context-engineering
status: completed
series:
  name: LangChain 1.x 系统学习
  part: 4
---

# Tools 与 Structured Output——让结果可调用、可验证、可组合

> 一句话摘要：现代 agent 工程最关键的两条能力链，一条是“模型如何请求系统行动”，另一条是“系统如何要求模型产出稳定对象”。前者是 tools，后者是 structured output。它们共同决定一个 agent 是只会说话，还是既能做事、又能把结果稳定交给系统继续处理。

## 学习目标

完成本课后，你应该能够：
- [ ] 说清 tools 在 LangChain 1.x 中到底扮演什么角色
- [ ] 理解为什么工具调用本质上是模型与系统之间的动作协议
- [ ] 理解 structured output 的核心价值是“可验证对象”，而不只是“好看的 JSON”
- [ ] 区分 provider-native structured output 与 tool strategy 的基本思路
- [ ] 看懂 tools 与 structured output 如何一起支撑 planner / executor / reviewer 回路
- [ ] 把这一章知识映射到 Claude Code-like harness 的 tool registry、executor、plan/review object 等模块

## 前置知识

| 知识点 | 要求 | Wiki 参考 |
|--------|------|-----------|
| LangChain 组件层 | 已理解 model / messages / tools / structured output 的大图 | [[raw/lessons/LangChain/02-langchain-1-x-core-abstractions|第 2 章：LangChain 1.x 的分层结构与核心抽象]] |
| 模型与消息边界 | 已知道 `AIMessage` / `ToolMessage` 和 provider boundary | [[raw/lessons/LangChain/03-models-messages-and-provider-boundaries|第 3 章：Models、Messages 与 Provider Boundaries]] |
| 执行回路 | 已知道 planner / executor / reviewer 的系统位置 | [[raw/lessons/Agent-Engineering/03-planner-executor-reviewer-loop|第 3 章：Planner / Executor / Reviewer——受控执行回路是怎样形成的]] |
| Python 基础 | 能读函数、装饰器、dataclass、Pydantic 风格 schema | — |

---

## 直观理解

### 类比一：Tools 像“给模型发的工作证”

如果模型只能输出自然语言，它最多只是一个会解释问题的人。

一旦你给它 tools，它就多了一种能力：
- 它可以请求系统帮它做动作
- 例如查文件、搜代码、访问数据库、发请求、运行命令

所以 tools 的本质不是“外挂函数”，而是：

> **让模型从只会说，变成会请求系统行动。**

### 类比二：Structured Output 像“要求它交表格，而不是只交感想”

如果你让一个实习生汇报任务进展，你当然可以接受一段自然语言。

但很多时候你真正想要的是结构明确的表格：
- 任务标题
- 风险等级
- 涉及文件
- 下一步动作

这就是 structured output 的思路：
- 不只要“说点什么”
- 而要“按规定结构交一个对象”

所以你可以先把它记成：

> **tools 解决“怎么做事”，structured output 解决“怎么稳定交结果”。**

---

## 一、为什么这两章通常应该连着学？

### 1.1 因为它们共同构成 agent 的输入输出协议

如果没有 tools，模型很难真正接触外部世界。
如果没有 structured output，模型做完后又很难把结果稳定交给系统继续处理。

所以很多 agent 场景里，这两者会形成一个很自然的组合：
- 先通过 tools 做外部动作
- 再通过 structured output 返回稳定对象

### 1.2 在 Harness 中它们分别落到哪？

- tools -> executor / tool registry / permission layer
- structured output -> planner object / reviewer object / summary object / decision object

也就是说，它们刚好对应“行动”和“交付”两条主线。

---

## 二、Tools 到底是什么？

### 2.1 tools 不是“随便把函数暴露给模型”

这是一个非常危险但常见的误解。

在 LangChain 1.x 里，tool 不是单纯的 Python 函数，而是一个带描述、带参数语义、可被模型识别和调用的动作接口。

它要回答的问题包括：
- 这个工具叫什么？
- 它应该什么时候被用？
- 参数长什么样？
- 返回结果是什么形态？

### 2.2 所以 tool 的本质是什么？

它本质上是一份“动作合同”：
- 系统承诺：如果模型按这个格式请求，我就执行这个动作
- 模型承诺：如果要用这个动作，就按这个 schema 来填写参数

这就是为什么我们说：

> **tool calling 是协议，而不是技巧。**

---

## 三、一个最小 Tool 示例：为什么 docstring 和参数语义很关键？

### 3.1 最小例子

```python
from langchain.tools import tool


@tool(parse_docstring=True)
def search_orders(user_id: str, status: str, limit: int = 10) -> str:
    """Search for user orders by status.

    Use this when the user asks about order history or wants to check
    order status. Always filter by the provided status.

    Args:
        user_id: Unique identifier for the user
        status: Order status: 'pending', 'shipped', or 'delivered'
        limit: Maximum number of results to return
    """
    return f"searching orders for {user_id}"
```

### 3.2 这段代码在说什么？

先看整体：
- `@tool(...)` 把普通函数声明成可供模型使用的工具
- 函数名、参数、返回值、docstring 一起构成工具描述

再拆关键语法：

#### `@tool`
这是 Python 装饰器。你可以先把它理解成：

> “在不改函数主体的前提下，给这个函数加上一层工具语义。”

#### `parse_docstring=True`
表示系统会尝试从 docstring 中提取更明确的工具说明和参数说明。

#### 参数类型标注
- `user_id: str`
- `status: str`
- `limit: int = 10`

这些都在帮助系统和模型理解工具参数应该长什么样。

### 3.3 为什么这很关键？

因为模型不是读你脑子，它只能读你暴露给它的工具定义。

工具描述写得越模糊，模型越容易：
- 选错工具
- 填错参数
- 在不该调用的时候调用

这说明 tool design 本身就是 agent 工程的一部分。

---

## 四、Tool Calling 的最小生命周期长什么样？

下面这条链非常重要。

```text
用户请求
  │
  ▼
Messages -> Model
  │
  ▼
AIMessage(tool_calls)
  │
  ▼
系统执行对应工具
  │
  ▼
ToolMessage(工具结果)
  │
  ▼
模型继续推理 / 生成最终回答
```

### 4.1 一个更贴近 LangChain 的示意代码

```python
from langchain.chat_models import init_chat_model

model = init_chat_model("gpt-5-nano")


def get_weather(location: str) -> str:
    """Get the weather at a location."""
    return "Sunny, 72°F"


model_with_tools = model.bind_tools([get_weather])
response = model_with_tools.invoke("What's the weather in Paris?")

for tool_call in response.tool_calls:
    print(tool_call["name"])
    print(tool_call["args"])
```

### 4.2 这段代码在说什么？

- `bind_tools(...)`：把工具能力绑定到模型可见范围
- `response.tool_calls`：模型返回它想调用的工具
- 系统随后再去真正执行对应工具

也就是说，模型不是直接执行 Python 函数，而是先提出一份工具调用请求。

这再一次证明：

> **tool calling 不是模型自己在跑代码，而是模型通过协议向系统申请动作。**

---

## 五、为什么 ToolMessage 很关键？

### 5.1 因为工具结果必须重新进入消息时间线

如果工具只是被执行，但结果没有被正式回写，模型下一步就无法稳定基于该结果继续推理。

所以你需要 `ToolMessage` 这类对象，把工具输出变成正式上下文的一部分。

### 5.2 一个最小示例

```python
from langchain.messages import HumanMessage, AIMessage, ToolMessage

messages = [HumanMessage(content="帮我查询 README 里有没有安装步骤")]

ai_message = AIMessage(
    content=[],
    tool_calls=[{
        "name": "read_file",
        "args": {"path": "README.md"},
        "id": "call_1"
    }]
)

tool_message = ToolMessage(
    content="README.md 中包含安装步骤：先 uv sync，再 uv run app.py",
    tool_call_id="call_1"
)
```

### 5.3 系统层意义是什么？

- 工具请求变成 `AIMessage`
- 工具结果变成 `ToolMessage`
- 后续推理继续基于消息协议进行

所以工具系统和消息系统并不是两条独立线，而是高度耦合的。

---

## 六、Structured Output 到底解决什么问题？

### 6.1 它不只是“让输出像 JSON”

很多人第一次接触 structured output，会误以为它只是：
- 格式更整齐
- 更像 API 返回

但更深层的价值其实是：

> **让模型输出变成可以被系统继续依赖的对象。**

### 6.2 哪些场景最需要它？

例如：
- planner 产出计划
- reviewer 产出检查结论
- executor 产出变更摘要
- triage 系统产出分类结果
- permission gate 产出风险判断

这些结果如果只是自由文本，很难稳定继续流转。

### 6.3 所以 structured output 真正在问什么？

它在问：
- 这个输出以后还要不要被程序处理？
- 要不要校验字段？
- 要不要保证合法值？
- 要不要被后续模块消费？

如果答案是“要”，那 structured output 就通常很重要。

---

## 七、Schema 思维：为什么 dataclass / Pydantic / TypedDict 都会高频出现？

### 7.1 因为你要提前规定“对象应该长什么样”

所谓 schema，可以先理解成：

> **一份结构约束说明。**

例如一个 review 结果对象，你可能希望它至少有：
- `status`
- `summary`
- `missing_items`
- `next_action`

如果没有 schema，模型就可能：
- 少字段
- 字段名漂移
- 值域不稳定
- 输出格式难解析

### 7.2 一个最小例子

```python
from pydantic import BaseModel, Field
from typing import Literal


class ProductReview(BaseModel):
    rating: int | None = Field(description="The rating of the product", ge=1, le=5)
    sentiment: Literal["positive", "negative"] = Field(description="The sentiment")
    key_points: list[str] = Field(description="Key points")
```

### 7.3 这段代码在说什么？

- `BaseModel`：定义一个结构化对象类型
- `Field(...)`：为字段补充说明和约束
- `Literal[...]`：限制字段只能取几个合法值之一

这类 schema 非常适合 agent 系统，因为它能让结果更稳定。

---

## 八、provider-native structured output 与 tool strategy 有什么区别？

### 8.1 provider-native structured output

有些模型供应商原生支持结构化输出。

这时 LangChain 可以更直接地要求模型按 schema 产出结构结果。

它通常意味着：
- 约束更强
- 返回更像真正的对象
- 稳定性往往更好

### 8.2 tool strategy

如果 provider 没有稳定的原生结构化输出能力，LangChain 也可以通过 tool strategy 来实现类似目标。

本质思路是：
- 把“输出某个结构对象”包装成一种工具调用
- 让模型按工具参数 schema 来填写结果

这很巧妙，因为它复用了工具协议。

### 8.3 为什么这个区别重要？

因为你在工程中经常会遇到：
- 某些 provider 支持原生 schema
- 某些 provider 更适合走 tool calling
- 某些场景你更重视稳定性而不是灵活文本

所以你需要知道：

> structured output 不是只有一种落地方式。

---

## 九、一个最小 structured output 示例

### 9.1 Dataclass 风格示意

```python
from dataclasses import dataclass
from typing import Literal
from langchain.agents import create_agent
from langchain.agents.structured_output import ToolStrategy


@dataclass
class ProductReview:
    rating: int | None
    sentiment: Literal["positive", "negative"]
    key_points: list[str]


agent = create_agent(
    model="gpt-5.4",
    tools=[],
    response_format=ToolStrategy(ProductReview)
)
```

### 9.2 这段代码在说什么？

- `ProductReview` 定义了你希望得到的结构
- `ToolStrategy(ProductReview)` 表示让 agent 以工具策略产出这个结构
- 结果不再只是随意文本，而更接近一个正式对象

### 9.3 它对 Harness 的价值是什么？

你完全可以把 `ProductReview` 换成：
- `PlanObject`
- `ReviewResult`
- `PermissionDecision`
- `TaskSummary`

这样 structured output 就直接服务于系统主回路，而不是只服务于 demo。

---

## 十、为什么 tools 与 structured output 会天然支撑 planner / executor / reviewer？

### 10.1 对 executor 的支撑：tools

executor 需要做事，所以它最依赖：
- tool registry
- tool schema
- tool calling
- tool result message

### 10.2 对 planner / reviewer 的支撑：structured output

planner 和 reviewer 虽然也会用模型，但它们更经常需要的是：
- 稳定计划对象
- 稳定审查对象
- 稳定风险判断对象

也就是说：
- tools 更偏行动能力
- structured output 更偏判断结果的对象化

### 10.3 三角色合起来看

| 角色 | 最依赖的 LangChain 能力 |
|------|--------------------------|
| Planner | messages + model + structured output |
| Executor | messages + model + tools |
| Reviewer | messages + model + structured output |

这张表很重要，因为它把 LangChain 组件层和 Harness 主回路正式接起来了。

---

## 十一、Python 补课：为什么这章的关键词是“协议”和“对象”，而不是“函数调用技巧”？

### 11.1 因为你已经不只是在写脚本，而是在定义系统契约

当你定义一个 tool 时，你在定义：
- 系统允许模型请求什么动作
- 参数必须长什么样
- 结果会怎样返回

当你定义 structured output 时，你在定义：
- 模型交付的对象应该长什么样
- 哪些字段必须存在
- 哪些值必须合法

这两者都更像“契约设计”，而不只是语法技巧。

### 11.2 一个更贴近 Harness 的最小对象例子

```python
from dataclasses import dataclass
from typing import Literal


@dataclass
class PermissionDecision:
    action: str
    decision: Literal["allow", "ask_user", "deny"]
    reason: str
```

### 11.3 这段代码的意义是什么？

它在说明：
- 权限判断结果适合是一个结构化对象
- 上层系统可以稳定读取 `decision`
- UI 层可以稳定展示 `reason`
- logging 层可以稳定记录该对象

这正是 structured output 在 Harness 中的典型落点。

---

## 十二、这一章对你后续路线的意义

学完这一章后，你应该把两个重要直觉建立起来：

### 12.1 工具不是“让模型更酷”，而是让系统动作可协议化

只有动作被标准化为 tool interface，后面你才更容易做：
- 权限门
- 统一日志
- 失败恢复
- 回放

### 12.2 structured output 不是“格式优化”，而是让结果真正进入系统

只有结果被稳定对象化，后面你才更容易做：
- planner 产出计划对象
- reviewer 产出审查对象
- session 记录阶段结果
- UI 展示结构化信息

所以这章其实在回答一个非常根本的问题：

> 一个 agent 怎样既能行动，又能把行动结果稳定纳入系统流程？

---

## 常见误区

| 误区 | 正确理解 |
|------|----------|
| “tool 就是把函数给模型用一下。” | 更准确地说，tool 是模型与系统之间的动作协议。 |
| “工具描述随便写也行，反正模型很聪明。” | 不对。工具说明、参数语义和返回形式会直接影响调用质量。 |
| “structured output 只是让 JSON 更工整。” | 错。它的核心价值是让结果变成系统可验证、可继续消费的对象。 |
| “provider-native 和 tool strategy 没什么区别。” | 它们的实现路径和能力边界不同，实际工程里要根据 provider 支持情况选择。 |
| “只有 executor 才和 LangChain tools 有关。” | planner 和 reviewer 虽然更常依赖 structured output，但整个回路都建立在统一消息 / 模型 / 协议之上。 |

---

## 思考题

### 基础理解

**Q1. 为什么说 tool calling 的本质是“动作协议”，而不是“模型自己执行代码”？**

> [!hint]- 💡 提示
> 模型返回的是工具请求，还是直接运行了你的 Python 函数？

> [!success]- ✅ 参考答案
> 因为模型本身不会直接执行你的 Python 代码，它返回的是一份工具调用请求，说明想调用哪个工具、参数是什么。真正的执行由系统完成，执行结果再作为 ToolMessage 回写给模型，所以本质上是模型和系统之间的一套动作协议。

**Q2. structured output 为什么比自由文本更适合计划、审查、权限判断这类场景？**

> [!hint]- 💡 提示
> 这些结果后面还要不要继续被系统消费？

> [!success]- ✅ 参考答案
> 因为计划、审查和权限判断往往不是给人看一眼就结束，而是要被系统继续处理、记录、展示或回放。structured output 能保证字段存在、值域稳定、格式可解析，因此更适合这类系统对象场景。

### 深入思考

**Q3. 为什么说 tools 更偏行动能力，而 structured output 更偏交付能力？**

> [!hint]- 💡 提示
> 一个负责“让系统去做”，一个负责“让结果能继续流”。

> [!success]- ✅ 参考答案
> tools 的核心价值是把外部动作暴露给模型，例如读文件、查信息、运行命令，因此更偏执行与行动；structured output 的核心价值是把模型结果收束成稳定对象，使计划、审查、总结等结果能够继续被系统消费，因此更偏结果交付与对象化。

**Q4. 为什么 tool design 本身也属于 agent engineering，而不是随手写几个函数就够了？**

> [!hint]- 💡 提示
> 模型能否正确选择工具、填对参数、理解什么时候该调用，依赖什么？

> [!success]- ✅ 参考答案
> 因为模型只能基于你提供的工具定义来理解工具。工具名、docstring、参数说明、返回形式、错误行为都会直接影响调用质量。设计不好的工具会让模型选错、填错、乱调，所以 tool design 本身就是 agent 工程的一部分。

---

## 关键要点回顾

- ✅ tools 让模型获得“请求系统行动”的能力，本质上是动作协议
- ✅ ToolMessage 让工具结果正式回到消息时间线中
- ✅ structured output 的核心价值是把结果收束成系统可验证对象
- ✅ provider-native structured output 与 tool strategy 是两种不同的落地路径
- ✅ tools 更偏执行能力，structured output 更偏结果交付能力
- ✅ 它们共同支撑了 planner / executor / reviewer 这条主回路

---

## 扩展阅读

- 📄 [LangChain Tools](https://docs.langchain.com/oss/python/langchain/tools) — 理解工具定义、绑定与使用方式
- 📄 [LangChain Messages](https://docs.langchain.com/oss/python/langchain/messages) — 理解 ToolMessage 和 tool_calls 的消息协议
- 📄 [LangChain Structured Output](https://docs.langchain.com/oss/python/langchain/structured-output) — 理解 schema、ToolStrategy 与 ProviderStrategy
- 📄 [LangChain Agents](https://docs.langchain.com/oss/python/langchain/agents) — 观察工具、模型、response_format 如何在高层 agent 中结合
- 📄 [LangChain Context Engineering](https://docs.langchain.com/oss/python/langchain/context-engineering) — 观察 docstring 和上下文设计如何影响工具使用效果

## 相关页面

- [[raw/lessons/LangChain/03-models-messages-and-provider-boundaries|第 3 章：Models、Messages 与 Provider Boundaries]]
- [[raw/lessons/Agent-Engineering/03-planner-executor-reviewer-loop|第 3 章：Planner / Executor / Reviewer——受控执行回路是怎样形成的]]
- [[raw/lessons/Agent-Engineering/02-harness-mvp-architecture|第 2 章：Harness MVP 架构——一个最小 coding agent 系统怎么拆]]
- [[raw/lessons/LangGraph/04-tools-and-react|第 4 章：工具与 ReAct——让研究助手学会调用检索工具]]

## 下一步学习

到这里，LangChain 这一小段主线已经把最关键的组件讲出来了：
- 分层结构
- model / message / provider boundary
- tools / structured output

接下来最自然的下一步有两种：

- **继续深挖组件层**：进入 prompt 与上下文组织，理解模型到底在看什么、什么信息该放在哪一层
- **回到 Harness 主线**：继续把工具层、权限层、session / checkpoint / resume、verification 层逐步装配成完整系统

如果你想继续顺着 LangChain 1.x 主线往下读，先看：
- [[raw/lessons/LangChain/05-prompt-and-context-organization|第 5 章：Prompt 与上下文组织——什么应放在哪一层]]

如果你想立刻回到系统装配主线，继续看：
- [[raw/lessons/Agent-Engineering/04-tool-layer-read-search-edit-run|第 4 章：工具层——读文件、搜代码、编辑文件、运行命令为什么构成最小 action surface]]
