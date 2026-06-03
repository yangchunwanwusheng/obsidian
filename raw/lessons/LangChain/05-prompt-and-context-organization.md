---
type: lesson
tags:
  - LangChain
  - LangChain 1.x
  - Prompt
  - Context Engineering
  - Messages
  - Python
  - Agent
created: 2026-04-29
updated: 2026-04-29
topic: Prompt 与上下文组织——什么应放在哪一层
difficulty: intermediate
prerequisites:
  - "[[raw/lessons/LangChain/03-models-messages-and-provider-boundaries|第 3 章：Models、Messages 与 Provider Boundaries]]"
  - "[[raw/lessons/LangChain/04-tools-and-structured-output|第 4 章：Tools 与 Structured Output——让结果可调用、可验证、可组合]]"
  - "[[raw/lessons/Agent-Engineering/03-planner-executor-reviewer-loop|第 3 章：Planner / Executor / Reviewer——受控执行回路是怎样形成的]]"
sources:
  - https://docs.langchain.com/oss/python/langchain/context-engineering
  - https://docs.langchain.com/oss/python/langchain/runtime
  - https://docs.langchain.com/oss/python/langchain/short-term-memory
  - https://docs.langchain.com/oss/python/langchain/messages
  - https://docs.langchain.com/oss/python/langchain/overview
status: completed
series:
  name: LangChain 1.x 系统学习
  part: 5
---

# Prompt 与上下文组织——什么应放在哪一层

> 一句话摘要：在 agent 工程里，prompt 不是“写一句更聪明的话”这么简单，而是决定模型在某一时刻到底看见哪些信息、这些信息按什么层次排列、哪些该保留、哪些该压缩。上下文组织得好，模型才能更稳定地选工具、填参数、交结构化结果；组织得差，再强的模型也会像在杂乱工位上盲做任务。

## 学习目标

完成本课后，你应该能够：
- [ ] 理解为什么 prompt 与上下文组织本质上是信息分层设计，而不只是文案技巧
- [ ] 说清 system、human、history、tool results 分别适合放什么内容
- [ ] 理解为什么 runtime context 不会自动进入模型可见上下文
- [ ] 看懂 `ChatPromptTemplate` 与 `MessagesPlaceholder` 在 LangChain 1.x 中扮演的角色
- [ ] 识别截断、摘要、选择性注入三种常见上下文控制策略
- [ ] 把 prompt / context 组织映射到 Claude Code-like harness 的 planner、executor、reviewer 三个落点

## 前置知识

| 知识点 | 要求 | Wiki 参考 |
|--------|------|-----------|
| Messages 边界 | 已知道 `SystemMessage`、`HumanMessage`、`AIMessage`、`ToolMessage` 的角色分工 | [[raw/lessons/LangChain/03-models-messages-and-provider-boundaries|第 3 章：Models、Messages 与 Provider Boundaries]] |
| Tools 与 Structured Output | 已知道工具调用协议和结构化输出对象的系统意义 | [[raw/lessons/LangChain/04-tools-and-structured-output|第 4 章：Tools 与 Structured Output——让结果可调用、可验证、可组合]] |
| Planner / Executor / Reviewer | 已知道一个 coding agent 会在规划、执行、复查三层之间循环 | [[raw/lessons/Agent-Engineering/03-planner-executor-reviewer-loop|第 3 章：Planner / Executor / Reviewer——受控执行回路是怎样形成的]] |
| Python 基础 | 能读 `import`、函数调用、列表、f-string | — |

---

## 直观理解

### 类比一：上下文像“给专家的工作简报”

假设你要请一位很强的工程顾问帮你排查线上 bug。

如果你只给他一句话：
- “帮我修一下登录问题。”

他当然也许能说几句建议，但很难稳定地下判断。

如果你给他的是一份整理好的工作简报：
- 系统规则：不能直接改生产数据库
- 当前任务：登录接口偶发 500
- 已知线索：错误出现在 `auth/login.py`
- 最近调查结果：日志里出现 `print(error)`
- 限制条件：先读文件，再给修改方案

那他的判断质量通常会高很多。

agent 里的 prompt 与 context 组织，就是在做这件事：

> **让模型不是在一团信息噪声里猜，而是在结构化工作简报上做判断。**

### 类比二：分层组织像“把不同材料装进不同信封”

你给一个执行人员派任务时，通常不会把所有东西乱塞成一张长纸条。

更常见的是分层：
- 封面规则：这次任务的总原则
- 任务单：这次具体要做什么
- 历史往来：之前沟通过什么
- 附件材料：日志、文件片段、工具结果

如果全塞成一层，会出 3 类问题：
1. 重点淹没在噪声里
2. 模型分不清“长期规则”和“临时现象”
3. 后续很难截断、摘要和重组

所以你可以先把本章压缩成一句话：

> **prompt engineering 在 agent 里更接近信息架构，而不是写作修辞。**

---

## 一、为什么这一章必须接在 Tools 与 Structured Output 后面？

第 4 章已经建立了两条重要能力链：

- tools：模型如何请求系统行动
- structured output：系统如何要求模型交付稳定对象

但这两条链条要稳定运转，前提是模型在做决定时看到了**合适的信息**。

### 1.1 tools 能不能用对，先看上下文有没有把任务讲清楚

例如一个 coding agent 面前有四个工具：
- read_file
- search_code
- edit_file
- run_command

如果上下文里只有一句：
- “帮我修复登录逻辑。”

模型很容易：
- 不知道应该先 `search_code` 还是直接 `edit_file`
- 不知道应该在哪个目录搜
- 不知道这次任务是先给方案还是直接动手

也就是说，**工具选择质量不仅取决于工具定义，还取决于上下文有没有把现场交代清楚。**

### 1.2 structured output 能不能填稳，也要看上下文有没有给足原料

例如你要求 reviewer 输出这样一个对象：

```python
{
  "status": "pass" | "needs_changes",
  "issues": [...],
  "changed_files": [...],
  "next_action": "..."
}
```

如果上下文里根本没有：
- 被改了哪些文件
- 原始任务要求是什么
- reviewer 应该按什么标准检查

那 structured output 再漂亮，也只是“结构化地胡说”。

所以第 4 章讲的是：
- 系统如何定义动作协议和结果协议

本章讲的是：
- **这些协议生效时，信息该怎么喂给模型**

---

## 二、模型真正会看到什么？先把“可见上下文”和“运行时上下文”分开

这是一个特别容易混淆的点。

### 2.1 模型真正看到的，首先是 messages

在 LangChain 1.x 语境里，模型最常直接看到的是一组 messages，例如：
- `SystemMessage`
- `HumanMessage`
- `AIMessage`
- `ToolMessage`

它们组成了一条有顺序、有角色的上下文时间线。

从模型视角看，这条时间线才是“眼前材料”。

### 2.2 但系统手里还可能有另一层信息：runtime context

LangChain 文档里有一个很重要的现实：

> runtime context 是每次运行时附带给 agent / harness 的配置或元数据，**不会自动进入模型 prompt**。

这意味着像下面这些东西，可能存在于系统里：
- 当前用户 ID
- feature flag
- 权限模式
- 数据库连接
- 工作目录
- 当前会话配置

但如果你没有显式把它们转成 message 或动态 system prompt，模型其实看不见。

### 2.3 为什么这件事非常重要？

因为很多人会产生一种错觉：
- “我明明把信息传进 agent 了，模型为什么没用上？”

答案常常是：
- 这份信息只存在于 harness/runtime 层
- 但并没有真正注入到模型可见的 messages 中

所以这里要先建立一个边界：

| 层 | 作用 | 模型是否自动可见 |
|----|------|------------------|
| messages | 模型直接阅读的上下文 | ✅ 是 |
| runtime context | 系统运行时携带的额外数据 | ❌ 否，除非显式注入 |
| tool results | 工具执行后回写的信息 | ✅ 如果被写成 `ToolMessage` |
| store / memory | 系统保存的长期信息 | ❌ 否，除非被取出并注入 |

这会直接影响你后面做 context engineering 的方式。

---

## 三、在 LangChain 1.x 里，prompt 组织不是纯字符串，而是消息模板

### 3.1 为什么现代 agent 更适合用消息模板，而不是大字符串？

因为 agent 的上下文天然就是多层的：
- 长期规则
- 当前任务
- 历史对话
- 工具执行结果

如果你把它们硬拼成一个超长字符串，短期当然也能跑，但很快会遇到问题：
- 不知道哪段是系统规则
- 不知道哪段是历史
- 不好插入新工具结果
- 不好截断历史
- 不好在不同阶段重组不同上下文

所以 LangChain 1.x 提供的思路是：

> **先用 message-level 的模板表达结构，再把变量填进去。**

### 3.2 一个最小例子

```python
from langchain_core.prompts import ChatPromptTemplate, MessagesPlaceholder
from langchain.messages import HumanMessage, ToolMessage

prompt = ChatPromptTemplate.from_messages(
    [
        (
            "system",
            "你是一个 coding agent。先读再改，高风险动作必须确认。"
        ),
        MessagesPlaceholder(variable_name="chat_history"),
        ("human", "{task}"),
        MessagesPlaceholder(variable_name="tool_results"),
    ]
)

formatted_prompt = prompt.invoke(
    {
        "task": "请定位 login 函数中的错误处理逻辑，并给出修改方案。",
        "chat_history": [
            HumanMessage(content="这个仓库里登录逻辑大概在哪个目录？"),
        ],
        "tool_results": [
            ToolMessage(
                content="src/auth/login.py: line 48 uses print(error)",
                tool_call_id="call_123",
            )
        ],
    }
)
```

### 3.3 这段代码在说什么？

先看整体结构：
- `ChatPromptTemplate.from_messages(...)`：定义一个“按消息分层”的 prompt 模板
- `MessagesPlaceholder(...)`：留一个插槽，后面动态塞入一组消息
- `prompt.invoke(...)`：把变量填进去，得到一份已经排好顺序的 prompt 输入

再拆关键 Python 语法：

#### `from ... import ...`
这是 Python 的导入语句，表示：
- 从 `langchain_core.prompts` 里导入 `ChatPromptTemplate` 和 `MessagesPlaceholder`
- 从 `langchain.messages` 里导入消息对象

#### `("human", "{task}")`
这里是一个二元组，你可以先把它理解成：
- 第一项说明消息角色
- 第二项说明消息模板文本

`{task}` 是一个变量占位符，后面会在 `prompt.invoke(...)` 时被替换。

#### `MessagesPlaceholder(variable_name="chat_history")`
这个对象的作用不是产生一条消息，而是：

> **在这个位置插入一组已经准备好的消息。**

它特别适合放：
- 历史对话
- agent scratchpad
- 工具执行结果

### 3.4 为什么这比大字符串更适合 agent？

因为你现在可以很自然地回答：
- system 规则放哪里
- 当前任务放哪里
- 历史消息插在哪里
- 工具结果回到哪里

这正是 agent 需要的“可重组上下文结构”。

---

## 四、什么信息该放在哪一层？

这是本章最核心的一张判断表。

| 信息类型 | 更适合放哪层 | 为什么 | 常见错误 |
|----------|-------------|--------|----------|
| 角色与全局原则 | `SystemMessage` | 这是长期约束，不应和临时任务混在一起 | 塞进 user 输入，导致规则被任务噪声淹没 |
| 当前任务描述 | `HumanMessage` | 这是本轮要解决的具体问题 | 和系统规则混成一大段 |
| 历史对话 | `MessagesPlaceholder(chat_history)` | 它是可截断、可摘要、可替换的一组消息 | 每次都整段硬拼进最新 user message |
| 工具执行结果 | `ToolMessage` 或单独 placeholder | 它属于执行反馈，不属于用户意图 | 把文件内容伪装成用户话语 |
| 运行时配置 | runtime context + 必要时注入 system prompt | 这类信息不一定每次都该暴露给模型 | 以为“传给 agent 就等于传给模型” |

### 4.1 system message 最适合装“长期不轻易变化的规则”

例如：
- 你是一个 coding agent
- 先读再改
- 高风险动作前必须确认
- 回复风格尽量简洁
- 优先给出结构化总结

这些不是本轮任务的临时说明，而是当前会话或当前系统模式下的**总原则**。

### 4.2 human message 最适合装“这次到底要你做什么”

例如：
- 把 login 函数的错误处理从 `print` 改成 logging
- 先不要动手修改，只给方案
- 我只想看和 `auth/` 目录相关的内容

它是本轮任务意图，不该和系统长期规则混为一谈。

### 4.3 history 最好当成“可管理资源”，不要当成神圣原文仓库

很多初学者会本能觉得：
- 历史越完整越好

但实际工程里，history 是最容易膨胀失控的部分。

它更应该被当成：
- 可以截断
- 可以摘要
- 可以按相关性选择性带入

的资源，而不是每次都必须全量原样塞进去。

### 4.4 tool results 应该像附件，而不是伪装成用户补充说明

如果工具读出了一个大文件片段，这份内容更适合：
- 作为 `ToolMessage`
- 或者作为一个明确的工具结果槽位插入

而不应该伪装成：
- “用户又说了一大段文件内容”

因为这样会破坏语义边界。

---

## 五、上下文窗口为什么是系统设计问题，而不是模型参数细节？

### 5.1 因为上下文不是免费无限的

在 coding agent 场景里，真正最容易撑爆上下文的不是闲聊，而是：
- 文件内容
- 搜索结果
- 编译输出
- 错误日志
- 大段历史消息

所以你迟早要面对一个现实：

> **不是所有信息都能永远同时放在模型眼前。**

### 5.2 一个直观的 token 预算表

假设你使用一个上下文窗口较大的模型，做一个中等复杂度的代码任务。你可能会分配成这样：

| 部分 | 典型内容 | 粗略预算 |
|------|----------|----------|
| system rules | 行为原则、权限规则、输出格式要求 | 2k |
| current task | 当前任务描述、目标文件范围、限制条件 | 3k |
| history summary | 之前几轮的摘要，不是全文 | 6k |
| tool results | 最近搜索结果、错误日志、文件片段 | 20k |
| focused file content | 当前最相关的文件正文 | 30k |
| output reserve | 给模型留出的回答空间 | 余量 |

这里真正昂贵的往往不是 prompt 文案，而是：
- 工具产出的原始材料
- 文件片段
- 错误日志

所以 context engineering 在 coding agent 中，本质上很像做资源调度。

### 5.3 为什么这不是“后端小优化”，而是系统骨架问题？

因为它会反过来影响：
- planner 一次能看见多少任务背景
- executor 一次能带进多少文件内容
- reviewer 一次能检查多少变更

换句话说，**窗口限制会直接塑造系统的工作粒度。**

---

## 六、当信息太多时，通常怎么组织？三种常见策略

### 6.1 策略一：截断（trimming）

最简单的办法是：
- 只保留最近 N 轮消息

一个极简 Python 例子：

```python
def keep_recent_messages(messages: list, keep_last: int = 6) -> list:
    return messages[-keep_last:]
```

这段代码的意思很直接：
- `messages[-keep_last:]` 是 Python 切片
- 表示从列表尾部取最近的若干项

它的优点是简单、快。
缺点是也很明显：
- 你可能把早期但关键的信息裁掉了

所以它更适合：
- 对话较短
- 历史价值递减明显
- 当前问题主要依赖最近上下文

### 6.2 策略二：摘要（summarization）

第二种方法不是全删，而是：
- 把早期历史压缩成摘要
- 再把摘要带入后续上下文

例如：
- 原始历史 30 轮，太长
- 用模型先生成一段“到目前为止做了什么、有哪些结论”的摘要
- 后续只保留最近几轮原文 + 早期摘要

它适合：
- 多步长任务
- 任务状态需要延续
- 但不需要保留所有细枝末节的场景

### 6.3 策略三：选择性注入（selective retrieval）

第三种方法更像工程系统：
- 不是看时间近不近
- 而是看当前任务相关不相关

例如当前要 review `auth/login.py`，那你真正要带进模型的，也许是：
- 登录相关的 system 规则
- 当前任务目标
- 最近一次关于 login 的工具结果
- `auth/login.py` 的局部文件内容

而不需要把：
- 上一轮讨论部署脚本的内容
- 与登录无关的搜索结果
- 很早之前的无关对话

一股脑全塞进去。

### 6.4 在 Harness 里这三种策略分别落到哪？

| 策略 | 在系统里常落到哪里 |
|------|------------------|
| 截断 | session / history manager |
| 摘要 | summarizer / memory compaction step |
| 选择性注入 | planner / retriever / repo exploration layer |

这也说明，本章虽然讲的是 LangChain 组件层，但它天然会指向后面的 Harness 设计。

---

## 七、为什么 prompt 组织会直接影响工具调用质量？

回忆第 4 章的 tool calling 生命周期：

```text
用户请求
  -> Messages / Model
  -> AIMessage(tool_calls)
  -> 系统执行工具
  -> ToolMessage
  -> 模型继续推理
```

真正决定 `AIMessage(tool_calls)` 质量的，不只是工具 docstring，还包括模型在当时看到了什么。

### 7.1 一个差的上下文组织是什么样？

例如把所有信息混成一条 user message：
- 你是一个 coding agent
- 先读再改
- 这个仓库很大
- 现在帮我修登录问题
- 上次你读过 `auth/`
- 工具 read_file 返回了这些内容
- 错误日志如下

这样的问题是：
- 模型分不清哪些是规则
- 哪些是任务
- 哪些是执行结果
- 哪些只是历史补充

最终它更容易：
- 直接调用错工具
- 忽略限制条件
- 在参数里漏掉关键目标

### 7.2 一个更好的组织是什么样？

- `SystemMessage`：先读再改、高风险需确认
- `HumanMessage`：本轮任务是什么
- `ToolMessage`：刚刚读出的文件片段
- `chat_history`：上一轮只保留和当前任务相关的对话

这样模型更容易做出这样的判断：
- 先 `search_code`
- 找到候选文件后 `read_file`
- 暂不直接 `edit_file`

也就是说，**上下文组织本身就在塑造工具选择路径。**

---

## 八、为什么上下文组织也会影响 structured output 质量？

structured output 经常会给人一种错觉：
- 只要 schema 写好了，输出自然会稳定

其实不完全是这样。

### 8.1 schema 只能约束“长什么样”，不能凭空创造“填什么”

例如你要求 planner 产出：
- task_summary
- risk_level
- target_files
- next_step

如果上下文里没有：
- 当前任务边界
- 用户到底想先调研还是先修改
- 仓库里哪些文件可能相关

那模型即便按 schema 输出，也可能：
- `target_files` 乱猜
- `risk_level` 判断失真
- `next_step` 过于空泛

### 8.2 所以 structured output 的稳定性依赖两件事

1. schema 设计得清楚
2. 上下文提供了足够原料

第 4 章更偏第 1 点。
本章更偏第 2 点。

它们合起来，才构成真正可用的结构化输出链。

---

## 九、这章知识在 Harness 中最终落到哪里？

### 9.1 落点一：planner 的任务输入组织

planner 不只是“想一想”，它需要稳定输入，例如：
- 当前任务描述
- 约束规则
- 可用工具清单
- 相关历史摘要
- 当前上下文窗口里最关键的项目信息

如果 planner 的输入组织混乱，它的计划质量通常会一起下降。

### 9.2 落点二：executor 的动作上下文选择

executor 每次行动时，通常不该把全世界都带进去，而应该只带：
- 当前步骤要做什么
- 当前步骤相关文件
- 最近工具结果
- 必要约束

这其实就是后面多步执行系统里的“局部上下文打包”。

### 9.3 落点三：reviewer 的检查范围定义

reviewer 不是简单“再问一遍模型”。

它需要看到：
- 原始任务要求
- 实际变更内容
- 验证结果
- 审查标准

如果 reviewer 上下文给不全，它不是“审查能力差”，而是压根没有足够材料可审。

### 9.4 一张收束表

| Harness 模块 | 最依赖哪类上下文组织 |
|--------------|----------------------|
| planner | 目标、约束、工具清单、历史摘要 |
| executor | 当前步骤、相关文件、工具结果 |
| reviewer | 任务要求、变更内容、验证结果、检查标准 |

所以本章的真正意义是：

> **把“模型眼前到底应该摆什么材料”这件事，从模糊经验提升成系统设计问题。**

---

## 常见误区

| 误区 | 正确理解 |
|------|----------|
| “prompt engineering 就是把 system message 写得更花。” | 更准确地说，它是在设计模型每一轮看见的信息结构。 |
| “只要信息足够多，模型就更稳。” | 错。信息过多会带来噪声、窗口挤压和重点淹没。 |
| “我把数据传进 agent 了，模型就一定看见了。” | 不一定。runtime context 默认不会自动进入模型 prompt。 |
| “history 越完整越好。” | 真实系统里 history 必须被截断、摘要或按相关性筛选。 |
| “structured output 稳不稳定只和 schema 有关。” | 它同样依赖上下文是否提供了足够、清晰的原料。 |

---

## 思考题

### 基础理解

**Q1. 为什么说在 agent 系统里，prompt 更像信息分层设计，而不是单纯文案技巧？**

> [!hint]- 💡 提示
> 想一想：模型到底是在读“漂亮文字”，还是在读一组被组织好的工作材料？

> [!success]- ✅ 参考答案
> 因为 agent 里的 prompt 不只是为了让一句话更好听，而是在决定模型此刻能看到哪些信息、这些信息按什么顺序和角色排列。system 规则、用户任务、历史对话、工具结果如果混在一起，会直接降低模型判断质量；分层清楚则能提升工具选择、参数填写和结构化输出稳定性。

**Q2. runtime context 为什么不能直接等同于模型看到的上下文？**

> [!hint]- 💡 提示
> LangChain 文档里强调了 runtime context 默认会不会自动进入 prompt？

> [!success]- ✅ 参考答案
> 因为 runtime context 是系统运行时附带的元数据或配置，它默认不会自动进入模型可见的 messages。只有当 harness 或 middleware 显式把它注入 system prompt、messages 或工具逻辑时，模型才真正能利用这部分信息。

### 深入思考

**Q3. 一个 coding agent 在 review 结果经常漏项，除了模型能力问题，还可能是哪类上下文组织问题？**

> [!hint]- 💡 提示
> reviewer 有没有同时看到原始要求、变更内容和验证结果？

> [!success]- ✅ 参考答案
> 很可能是 reviewer 的输入材料不完整。例如只给了最终回答，没有给原始任务要求；或者给了变更摘要，却没有给真实文件 diff；又或者没有把测试 / lint 结果带进去。这样 reviewer 即便输出了结构化结论，也可能只是基于不完整材料做判断。

**Q4. 为什么说历史消息不应该被当成“永远原样保留的神圣档案”？**

> [!hint]- 💡 提示
> 想想 token 预算、任务相关性和不同阶段的信息密度。

> [!success]- ✅ 参考答案
> 因为历史消息会不断膨胀，而上下文窗口有限。真实系统必须根据当前任务需要，对历史做截断、摘要或选择性注入。否则关键信息会被噪声淹没，工具结果和文件内容也会把窗口挤爆。history 更适合作为可管理资源，而不是每轮都必须全量重放的档案库。

---

## 关键要点回顾

- ✅ prompt 与上下文组织在 agent 里本质上是信息分层设计
- ✅ models 直接看到的主要是 messages，而 runtime context 默认不会自动进入 prompt
- ✅ `ChatPromptTemplate` 与 `MessagesPlaceholder` 让 system、task、history、tool results 能以结构化方式组合
- ✅ history 不是越长越好，而是要做截断、摘要和选择性注入
- ✅ tool calling 质量不只取决于工具 schema，也取决于模型当时看见了什么
- ✅ structured output 的稳定性既依赖 schema，也依赖上下文是否给足原料
- ✅ 这章知识最终会落到 harness 的 planner、executor、reviewer 三层输入组织上

---

## 扩展阅读

- 📄 [LangChain Context Engineering](https://docs.langchain.com/oss/python/langchain/context-engineering) — 理解 LangChain 如何讨论上下文组织、运行时信息与模型可见信息之间的边界
- 📄 [LangChain Runtime](https://docs.langchain.com/oss/python/langchain/runtime) — 理解 runtime context 如何被 middleware 读取与注入，而不是默认自动暴露给模型
- 📄 [LangChain Short-term Memory](https://docs.langchain.com/oss/python/langchain/short-term-memory) — 观察动态 prompt 与短期上下文管理的关系
- 📄 [LangChain Messages](https://docs.langchain.com/oss/python/langchain/messages) — 回看消息对象的正式语义边界
- 📄 [LangChain Overview](https://docs.langchain.com/oss/python/langchain/overview) — 从整体框架视角复习 prompt / messages / tools 在组件图中的位置

## 相关页面

- [[raw/lessons/LangChain/03-models-messages-and-provider-boundaries|第 3 章：Models、Messages 与 Provider Boundaries]]
- [[raw/lessons/LangChain/04-tools-and-structured-output|第 4 章：Tools 与 Structured Output——让结果可调用、可验证、可组合]]
- [[raw/lessons/LangChain/06-memory-state-history-boundaries|第 6 章：Memory / State / History——哪些属于组件层，哪些属于系统层]]
- [[raw/lessons/Agent-Engineering/03-planner-executor-reviewer-loop|第 3 章：Planner / Executor / Reviewer——受控执行回路是怎样形成的]]
- [[raw/lessons/full-plan|LangGraph → LangChain 1.x → Claude Code-like Harness 学习与写作总规划]]

## 下一步学习

现在你已经把“动作协议”和“结果协议”进一步推进到了“信息该怎样组织”这一层，下一步更自然的问题已经从“怎样摆材料”进入到“这些材料到底属于哪种状态层”：
- 哪些内容只是模型眼前的 messages history？
- 哪些内容属于当前 conversation 的短期 state？
- 哪些信息值得跨会话进入 long-term store？

建议优先继续阅读：
- [[raw/lessons/LangChain/06-memory-state-history-boundaries|第 6 章：Memory / State / History——哪些属于组件层，哪些属于系统层]]

如果你想回看整条路线，再决定下一步，也可以返回总路线图：[[raw/lessons/full-plan|LangGraph → LangChain 1.x → Claude Code-like Harness 学习与写作总规划]]
