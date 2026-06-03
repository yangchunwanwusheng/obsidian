---
type: lesson
tags:
  - LangChain
  - LangChain 1.x
  - Memory
  - State
  - History
  - Messages
  - Store
  - Python
  - Agent
created: 2026-04-29
updated: 2026-04-29
topic: Memory / State / History——哪些属于组件层，哪些属于系统层
difficulty: intermediate
prerequisites:
  - "[[raw/lessons/LangChain/05-prompt-and-context-organization|第 5 章：Prompt 与上下文组织——什么应放在哪一层]]"
  - "[[raw/lessons/LangChain/03-models-messages-and-provider-boundaries|第 3 章：Models、Messages 与 Provider Boundaries]]"
  - "[[raw/lessons/Agent-Engineering/06-session-memory-checkpoint-resume|第 6 章：Session、Memory、Checkpoint、Resume——为什么不能把状态全塞进一个记忆盒子]]"
sources:
  - https://docs.langchain.com/oss/python/langchain/short-term-memory
  - https://docs.langchain.com/oss/python/langchain/long-term-memory
  - https://docs.langchain.com/oss/python/langchain/context-engineering
  - https://docs.langchain.com/oss/python/langchain/runtime
  - https://docs.langchain.com/oss/python/langchain/tools
status: completed
series:
  name: LangChain 1.x 系统学习
  part: 6
---

# Memory / State / History——哪些属于组件层，哪些属于系统层

> 一句话摘要：在 LangChain 1.x 里，`memory` 这个词如果不加限定，很容易把三件不同的事混成一团：模型眼前正在滚动的 **history**、当前会话里可读可改的 **state**、以及跨会话长期保留的 **store / long-term memory**。把它们分清，才能知道哪些能力 LangChain 组件层已经提供，哪些仍然要由更下层的 runtime 或更上层的 harness 自己负责。

## 学习目标

完成本课后，你应该能够：
- [ ] 说清为什么 `memory` 在 agent 工程里是一个容易误导的大词
- [ ] 区分 history、state、store / long-term memory 三者的时间尺度与责任边界
- [ ] 理解 LangChain 1.x 文档里“short-term memory”为什么更接近 state，而不是一个神秘黑盒
- [ ] 看懂 `AgentState`、`ToolRuntime`、`runtime.state`、`runtime.store` 分别在表达什么
- [ ] 判断哪些问题属于 LangChain 组件层，哪些已经进入 runtime / harness 层
- [ ] 把本章知识接到后面的仓库探索、任务拆解、verification 设计上

## 前置知识

| 知识点 | 要求 | Wiki 参考 |
|--------|------|-----------|
| Messages 边界 | 已知道 `SystemMessage`、`HumanMessage`、`AIMessage`、`ToolMessage` 的语义分工 | [[raw/lessons/LangChain/03-models-messages-and-provider-boundaries|第 3 章：Models、Messages 与 Provider Boundaries]] |
| Prompt 与上下文组织 | 已知道模型真正看到的是 messages，runtime context 默认不会自动进入 prompt | [[raw/lessons/LangChain/05-prompt-and-context-organization|第 5 章：Prompt 与上下文组织——什么应放在哪一层]] |
| Session / Checkpoint / Resume | 已知道系统层为什么要区分会话状态、恢复快照和恢复动作 | [[raw/lessons/Agent-Engineering/06-session-memory-checkpoint-resume|第 6 章：Session、Memory、Checkpoint、Resume——为什么不能把状态全塞进一个记忆盒子]] |
| Python 基础 | 能读 `class`、类型标注、字典访问、函数参数 | — |

---

## 直观理解

### 类比一：白板、会议纪要、档案柜，不是一回事

想象你在带一个项目组：

- **history** 像会议现场白板上按时间留下的对话与记录
- **state** 像当前项目工作台上可随时更新的任务状态板
- **long-term memory / store** 像档案柜里长期保存的用户偏好、团队约定、历史资料

这三者看起来都在“记东西”，但用途完全不同：
- 白板强调“刚才发生了什么”
- 状态板强调“当前系统现在知道什么、做到哪了”
- 档案柜强调“下次再来还值得继续保留什么”

如果你把它们全叫 memory，讨论很快就会乱掉。

### 类比二：聊天记录、当前棋盘、存档资料库

再换一个更接近 agent 的比喻：

- **history** 像你和棋友刚才几步怎么走的记录
- **state** 像当前棋盘局面：轮到谁、哪些子还在、当前策略标签是什么
- **store** 像长期棋谱库：这个用户偏爱什么开局、以前总结过哪些模式

其中最容易误会的是：

> **history 常常是 state 的一部分，但 history 不等于整个 state；而 state 也不等于长期 memory。**

这就是本章要拆开的核心边界。

---

## 一、为什么 `memory` 是一个危险但又绕不过去的词？

### 1.1 因为大家常用同一个词指三种不同时间尺度

在很多泛化讨论里，memory 可能同时指：
- 对话历史
- 当前任务状态
- 用户长期偏好
- 上轮工具结果
- 可恢复的中间字段
- 以后还想复用的知识

问题是，这些内容并不属于同一层。

如果你直接问：
- “这个 agent 有没有 memory？”

其实几乎还等于没问，因为它至少可能在问三件不同的事：
1. 模型会不会看见前面几轮 history？
2. 当前 conversation state 会不会在执行中持续更新？
3. 用户信息会不会被存进跨会话 store？

### 1.2 在 LangChain 1.x 里，更好的问法是什么？

更精确的问法通常是：
- 这份信息是 **messages history** 吗？
- 这份信息属于 **conversation-scoped state** 吗？
- 这份信息要不要进入 **cross-conversation store**？

一旦这样问，系统边界就会清楚很多。

### 1.3 本章先给一个最短结论

在 LangChain 1.x 语境里，你可以先用下面这个近似图记住：

| 词 | 更接近什么 | 时间尺度 |
|----|------------|----------|
| history | messages 时间线 | 当前会话内最近上下文 |
| short-term memory | state | 当前 conversation / thread |
| long-term memory | store | 跨会话 / 跨任务 |

注意：这是“更接近”，不是绝对同义词，但足够帮助你先避免大部分混淆。

---

## 二、LangChain 1.x 把上下文来源拆成了哪几层？

第 5 章已经讲过一个关键事实：

> runtime context 不会自动进入模型可见 prompt。

本章在此基础上，进一步把 LangChain 1.x 常见的数据来源拆成三层：

| 层 | 典型内容 | 生命周期 | 模型是否自动可见 |
|----|----------|----------|------------------|
| context | 用户 ID、权限模式、外部配置、API 句柄 | 本次运行 | ❌ 否，默认不直接可见 |
| state | 当前消息、工具结果、短期字段、阶段状态 | 当前 conversation / thread | ⚠️ 部分可见，取决于是否被组织进消息流 |
| store | 用户偏好、长期资料、跨会话沉淀 | 跨 conversation | ❌ 否，必须显式读取与注入 |

这里最关键的是两点：

### 2.1 state 是“短期工作内存”，不是“永久知识库”

LangChain 文档里把 state 视为 short-term memory。它更像当前会话工作台，而不是长期档案库。

### 2.2 store 才是更接近长期 memory 的东西

如果你想让信息跨会话保留，例如：
- 用户偏好中文回复
- 用户喜欢测试优先
- 某个固定仓库的长期约定

这类信息更适合进入 store，而不是一直塞在当前 state 里。

---

## 三、history 到底是什么？为什么它重要，但又不能膨胀成全部状态？

### 3.1 history 最常见的形态，是一串 messages

回忆第 3 章：在 LangChain 里，消息通常是：
- `SystemMessage`
- `HumanMessage`
- `AIMessage`
- `ToolMessage`

它们按顺序组成一条时间线。这条时间线就是最典型的 history。

history 的价值是：
- 让模型知道前面说了什么
- 让工具调用结果成为后续推理依据
- 让 reviewer / debugger 能回看过程

### 3.2 但 history 不等于整个 state

很多人一开始会把“当前状态”理解成“把历史消息全留着就行”。这不够。

因为一个 agent 的 state 里还可能有：
- 当前用户名
- 当前任务阶段
- 当前选择的文件范围
- 当前风险标签
- 当前 planner 产出的结构化字段

这些东西未必都要直接以 messages 形式出现，但它们确实属于当前 conversation 的状态。

所以你可以记住一句很重要的话：

> **history 更像 state 中最贴近模型可见时间线的那一部分。**

### 3.3 history 为什么不能无限膨胀？

因为一旦 history 无限制增长，就会出现第 5 章已经埋下的问题：
- token 预算被挤爆
- 早期无关内容淹没当前重点
- 工具结果与文件内容竞争窗口
- planner / executor / reviewer 都看不清现场

所以 history 很重要，但它必须被管理，而不是被神圣化。

---

## 四、state 到底是什么？为什么 LangChain 1.x 说 short-term memory 更像它？

### 4.1 state 是“当前 conversation 里可持续更新的一组字段”

在 LangChain 1.x 的 agent 语境里，state 不是一句玄学描述，而是一份结构化状态对象。

它通常至少包含：
- `messages`
- 若干自定义字段

比如：
- `user_name`
- `selected_repo`
- `plan_status`
- `risk_level`

### 4.2 一个最小例子

```python
from langchain.agents import create_agent, AgentState


class CustomState(AgentState):
    user_name: str


agent = create_agent(
    model="gpt-5-nano",
    tools=[],
    state_schema=CustomState,
)
```

### 4.3 这段代码在说什么？

先看 Python 结构：

- `class CustomState(AgentState):`：定义一个继承自 `AgentState` 的状态类型
- `user_name: str`：给这个状态对象增加一个字符串字段
- `state_schema=CustomState`：告诉 agent，本次运行要按这个状态结构来组织短期状态

它的系统意义是：

> **短期 memory 不是一团模糊记忆，而是一份有 schema 的 conversation state。**

### 4.4 为什么这很重要？

因为一旦 state 可结构化，你就能更稳定地做这些事：
- 工具读写当前状态
- middleware 根据状态决定注入哪些上下文
- planner / executor 共享阶段信息
- reviewer 获取更清晰的现场而不是只看自然语言

这比“把所有临时信息混在一段历史文本里”强得多。

---

## 五、tool 为什么也会影响 state？

第 4 章讲过，tool calling 不是模型直接执行代码，而是通过协议向系统申请动作。

到了本章，你还要再补一层：

> **工具不仅能返回结果给模型，还可能更新当前 state。**

### 5.1 一个带状态更新的最小例子

```python
from langchain.tools import tool, ToolRuntime
from langchain.agents import create_agent, AgentState
from langchain.messages import ToolMessage
from langgraph.types import Command


class CustomState(AgentState):
    user_name: str


@tool
def update_user_info(runtime: ToolRuntime) -> Command:
    """Look up and update user info."""
    name = "John Smith"
    return Command(
        update={
            "user_name": name,
            "messages": [
                ToolMessage(
                    "Successfully looked up user information",
                    tool_call_id=runtime.tool_call_id,
                )
            ],
        }
    )
```

### 5.2 这段代码在说什么？

先拆 Python：

- `@tool`：把普通函数声明成工具
- `runtime: ToolRuntime`：工具运行时可以拿到当前运行现场
- `Command(update={...})`：返回一个“更新状态”的指令，而不只是纯文本
- `"user_name": name`：把新字段写回 state
- `"messages": [...]`：把工具结果同时写回消息时间线

### 5.3 这里最值得你记住的不是 API，而是边界

这个例子清楚说明：
- history 可以更新，因为 `messages` 被追加了
- state 可以更新，因为 `user_name` 被写入了
- 这两者相关，但不是同一个概念

也就是说：

> **messages history 是 state 里最重要的一条时间线，但 state 还可以承载额外短期字段。**

---

## 六、long-term memory / store 又是什么？

### 6.1 store 解决的是“跨 conversation 仍要留下什么”

如果说 state 是当前会话的短期工作内存，那么 store 更像长期资料库。

它适合保存：
- 用户长期偏好
- 稳定的账号资料
- 多轮多会话复用的事实
- 已沉淀的经验，而不是一次性执行噪音

### 6.2 一个最小例子

```python
from dataclasses import dataclass
from langchain.tools import tool, ToolRuntime
from langgraph.store.memory import InMemoryStore


@dataclass
class Context:
    user_id: str


store = InMemoryStore()
store.put(("users",), "user_123", {"language": "Chinese"})


@tool
def get_user_info(runtime: ToolRuntime[Context]) -> str:
    """Look up user info."""
    user_id = runtime.context.user_id
    user_info = runtime.store.get(("users",), user_id)
    return str(user_info.value) if user_info else "Unknown user"
```

### 6.3 这段代码在说什么？

先讲 Python：

- `@dataclass`：定义一个轻量数据对象，这里用来表达运行时 context
- `store.put(...)`：往长期存储里写一条数据
- `runtime.context.user_id`：从本次运行的 context 中取用户 ID
- `runtime.store.get(...)`：从 store 中读取跨会话数据

系统层意义则是：

- `context` 提供本次运行的定位信息
- `store` 提供可跨会话复用的长期信息
- 这份长期信息只有在工具或系统显式读取时才会进入后续流程

### 6.4 为什么 store 不该被当成“超级 state”？

因为它解决的是不同问题：
- state 关注当前这条 conversation 正在发生什么
- store 关注以后别的 conversation 还值不值得继续保留

把两者混起来，会造成：
- 当前临时噪音污染长期资料
- 长期资料被误当成当前现场事实
- session 结束后不知道该清什么、不该清什么

---

## 七、把三者放在一张图里看：它们到底怎么协作？

```text
用户请求
  ↓
messages history（最近对话、ToolMessage、SystemMessage）
  ↓
agent 当前 state（messages + 自定义短期字段）
  ↓
工具 / middleware / planner / executor 读取与更新 state
  ↓
必要时再从 store 取长期资料补进来
  ↓
模型继续推理
```

这张图说明了一个很实用的层级关系：

1. **history** 常常直接以 messages 形态进入模型视野
2. **state** 是承载 history 以及其他短期字段的工作容器
3. **store** 是更远处的长期存储，按需取用，而不是默认全灌给模型

这也解释了为什么很多系统里会同时出现：
- history trimming
- state schema
- memory retrieval / store access

它们并不是重复设计，而是在不同层管理不同时间尺度的信息。

---

## 八、哪些属于 LangChain 组件层，哪些已经超出，进入系统层？

这是本章最关键的判断表。

| 问题 | 更偏 LangChain 组件层 | 更偏系统层 / Harness 层 |
|------|----------------------|-------------------------|
| 如何表示消息历史？ | `SystemMessage`、`HumanMessage`、`ToolMessage` | 如何裁剪、摘要、回放这些历史 |
| 如何表示短期状态？ | `AgentState`、`state_schema` | 哪些字段该进 state、何时清理、何时压缩 |
| 如何让工具读写状态？ | `ToolRuntime`、`Command(update=...)` | 哪些工具允许改 state、改完后怎么验证 |
| 如何保存长期信息？ | `store` 接口、`runtime.store` | 存什么命名空间、何时写入、何时检索、是否需要权限控制 |
| 如何跨中断恢复？ | 组件层只提供部分基础 | checkpoint / resume 语义、线程恢复策略通常在更下层 runtime 或更上层 harness |

### 8.1 LangChain 已经给你的，是“表达与接线能力”

LangChain 1.x 在这一章最主要给你的，不是完整产品级 memory system，而是：
- 表达 messages 的标准对象
- 表达短期 state 的 schema 方式
- 让工具访问 `state` / `context` / `store` 的运行时入口

### 8.2 但 LangChain 不会替你做所有系统决策

例如这些问题，仍然主要属于 harness：
- 什么时候把旧 history 摘要化？
- 哪些 repo 扫描结果只保留当前任务，哪些值得长期存？
- 哪些长期记忆需要用户确认后才能写入？
- 多步任务失败后，state 应该回滚到哪里？

这正好和 [[raw/lessons/Agent-Engineering/06-session-memory-checkpoint-resume|AE06]] 形成互补：
- AE06 从系统装配角度讲状态骨架
- 本章从 LangChain 组件层讲“有哪些标准表达方式可用”

---

## 九、最常见的混淆是什么？

### 9.1 混淆一：把 history 当成全部 memory

错误理解：
- “只要把聊天记录全留着，memory 就做好了。”

问题在于：
- 聊天记录不一定包含结构化短期字段
- 聊天记录更不等于长期偏好存储
- 全量保留 history 还会迅速挤爆窗口

### 9.2 混淆二：把 state 当成长期档案柜

错误理解：
- “把用户偏好、项目规则、当前任务中间值全放 state 就好。”

问题在于：
- state 是 conversation-scoped
- 大量长期信息塞进去会让短期现场变脏
- 下一次会话未必该无条件继承这些字段

### 9.3 混淆三：把 store 当成默认 prompt

错误理解：
- “既然已经存进 store，模型下次自然会知道。”

不对。

store 里的信息必须被：
- 工具显式读取
- middleware 显式注入
- 或系统显式转换成模型可见消息

否则它只是“被保存了”，不是“被模型看见了”。

### 9.4 混淆四：把组件层能力误当成完整恢复系统

LangChain / LangGraph 给了不少状态与存储原语，但这不等于：
- 你的产品已经自动拥有完备 resume 设计
- 你的 multi-step task 自动具备回滚策略
- 你的 permission gate 自动知道哪些记忆可写入长期层

这些依然要靠更高一层的系统设计来收口。

---

## 十、这章知识会怎样落到后面的 Harness 章节？

### 10.1 对项目理解 / 仓库探索的影响

当 agent 开始做 repo exploration 时，会产生大量中间材料：
- 搜索命中
- 目录结构摘要
- 候选文件列表
- 已确认的关键模块

这时你就必须知道：
- 哪些只是当前 state 的短期字段
- 哪些应该写成 history 让模型继续看
- 哪些值得沉淀进长期 store

### 10.2 对任务拆解的影响

在多步任务里，planner / executor 经常要共享：
- 当前步骤编号
- 已完成步骤
- 上一步工具结果摘要
- 当前风险级别

这些更接近 state，而不是 long-term memory。

### 10.3 对 verification 的影响

reviewer / verifier 想做稳定判断时，最依赖的是：
- 原始任务要求
- 实际改动结果
- 最近验证输出
- 必要的结构化状态字段

如果这些边界没分清，verification 会非常混乱：
- 该看的没带进去
- 不该带的旧噪音又跟进来了

所以本章并不是在讲一个“附属 memory 功能”，而是在给后面的系统模块清理责任边界。

---

## 常见误区

| 误区 | 正确理解 |
|------|----------|
| “memory 就是聊天历史。” | 聊天历史更接近 history / messages 时间线，不等于全部 memory。 |
| “short-term memory 是某个神秘黑盒。” | 在 LangChain 1.x 里，它更接近结构化的 conversation state。 |
| “state 和 history 是一回事。” | history 常常是 state 的一部分，但 state 还可以有额外短期字段。 |
| “store 里的东西模型会自动知道。” | 不会。store 必须被显式读取并注入后，模型才会利用。 |
| “LangChain 既然有 state/store，就等于完整 memory system 已经做完。” | LangChain 给的是表达和接线原语；压缩、清理、恢复、权限、策略仍要靠系统层设计。 |

---

## 思考题

### 基础理解

**Q1. 为什么说 `memory` 在 agent 工程里是一个需要拆开的词，而不是直接拿来就用的词？**

> [!hint]- 💡 提示
> 想想它到底可能指 history、state 还是 store。

> [!success]- ✅ 参考答案
> 因为 memory 在讨论中经常同时指对话历史、短期会话状态、长期跨会话存储等不同层次的信息。如果不拆开，系统设计就会把不同时间尺度、不同责任边界的内容混在一起，导致上下文膨胀、状态污染和恢复混乱。

**Q2. history、state、store 三者最简短的区别是什么？**

> [!hint]- 💡 提示
> 按“模型眼前的时间线”“当前会话工作台”“长期档案库”来想。

> [!success]- ✅ 参考答案
> history 更像消息时间线，强调前面发生了什么；state 更像当前会话的短期工作容器，既包含 messages，也可能包含额外短期字段；store 更像长期档案库，保存跨会话仍有价值的信息。

### 深入思考

**Q3. 如果一个 repo agent 每次都把全部历史搜索结果、全部文件片段和全部旧结论一起塞进 prompt，真正出问题的是 history、state 还是系统策略？**

> [!hint]- 💡 提示
> 先想“能不能塞”，再想“该不该塞”。

> [!success]- ✅ 参考答案
> 从表示能力上看，state 和 history 都可以装下这些内容；但真正出问题的是系统策略。因为是否保留、裁剪、摘要、按相关性注入，主要是 harness 层的上下文管理问题。LangChain 提供了表达方式，不会替你自动做正确的压缩和筛选策略。

**Q4. 为什么说 store 不是“默认对模型可见的长期记忆”，而是“可被检索的长期资料层”？**

> [!hint]- 💡 提示
> 想想 runtime.store 和 messages 之间是不是天然直通。

> [!success]- ✅ 参考答案
> 因为 store 中的数据默认只是被保存，并不会自动变成模型可见消息。只有当工具、middleware 或系统逻辑显式从 store 读取并把结果注入上下文后，模型才真正能利用这些长期信息。所以 store 更像检索得到的长期资料层，而不是总在模型眼前展开的记忆墙。

---

## 关键要点回顾

- ✅ `memory` 是一个需要拆开的总称，不宜直接拿来当设计答案
- ✅ 在 LangChain 1.x 里，history 更接近 messages 时间线
- ✅ short-term memory 更接近 state，而不是神秘黑盒
- ✅ long-term memory 更接近 store，服务跨会话保留
- ✅ history 常常属于 state 的一部分，但 state 不只等于 history
- ✅ LangChain 提供的是 messages、state、store 的表达与接线原语
- ✅ 是否裁剪 history、何时写入 store、如何恢复执行，仍然属于更高层的 harness 决策

---

## 扩展阅读

- 📄 [LangChain Short-term Memory](https://docs.langchain.com/oss/python/langchain/short-term-memory) — 观察 LangChain 1.x 如何把短期 memory 具体落到 state 与工具可更新字段上
- 📄 [LangChain Long-term Memory](https://docs.langchain.com/oss/python/langchain/long-term-memory) — 理解 store 为什么更像跨会话持久层
- 📄 [LangChain Context Engineering](https://docs.langchain.com/oss/python/langchain/context-engineering) — 复习 context / state / store 三类数据源的整体关系
- 📄 [LangChain Runtime](https://docs.langchain.com/oss/python/langchain/runtime) — 看 `ToolRuntime` 如何把 context 与 store 暴露给工具
- 📄 [LangChain Tools](https://docs.langchain.com/oss/python/langchain/tools) — 看 `runtime.state`、`runtime.store` 在工具中的实际入口

## 相关页面

- [[raw/lessons/LangChain/03-models-messages-and-provider-boundaries|第 3 章：Models、Messages 与 Provider Boundaries]]
- [[raw/lessons/LangChain/05-prompt-and-context-organization|第 5 章：Prompt 与上下文组织——什么应放在哪一层]]
- [[raw/lessons/Agent-Engineering/06-session-memory-checkpoint-resume|第 6 章：Session、Memory、Checkpoint、Resume——为什么不能把状态全塞进一个记忆盒子]]
- [[raw/lessons/Agent-Engineering/07-task-decomposition-and-multi-step-strategy|第 7 章：任务分解与多步执行策略——复杂任务为什么不能只靠临场乱试]]
- [[raw/lessons/full-plan|LangGraph → LangChain 1.x → Claude Code-like Harness 学习与写作总规划]]

## 下一步学习

现在你已经把“模型可见的 messages 时间线”“当前 conversation 的短期 state”“跨会话的长期 store”区分清楚了，下一步最自然的问题是：
- agent 面对一个陌生仓库时，该把哪些理解留在当前 state？
- 哪些搜索结论只属于本轮任务，哪些值得沉淀？
- 为什么仓库探索不能一上来就盲改代码？

按当前交替路线，建议优先继续：
- [[raw/lessons/Agent-Engineering/08-project-understanding-and-repo-exploration|Harness 第 8 章：项目理解与仓库探索——为什么 agent 不能一上来就盲改代码]]

如果你想先沿 LangChain 组件层继续走，也可以接着读：
- [[raw/lessons/LangChain/07-langchain-vs-langgraph-boundaries|第 7 章：LangChain 1.x 与 LangGraph 的边界——什么时候该上图，什么时候停在组件层]]
