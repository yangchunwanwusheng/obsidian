---
type: lesson
tags: [多智能体, multi-agent, survey, 协作机制, 论文翻译, 学习笔记]
created: 2026-05-08
updated: 2026-05-08
topic: LLM 多智能体综述中文精读（第 3 篇-02）协作系统定义与形式化
difficulty: intermediate
prerequisites:
  - [[raw/lessons/Multi-Agent-Survey/03-collaboration-mechanisms/01-collaboration-mechanisms-introduction-background]]
sources:
  - [[raw/paper/multi-agent/03-multi-agent-collaboration-mechanisms-a-survey-of-llms-2025.pdf]]
  - https://arxiv.org/abs/2501.06322
status: completed
series:
  name: "Multi-Agent Survey｜Collaboration Mechanisms（2025）"
  part: 2
paper_meta:
  paper_id: "multi-agent-survey-03"
  title: "Multi-Agent Collaboration Mechanisms: A Survey of LLMs"
  authors: "Khanh-Tung Tran, Dung Dao, Minh-Duong Nguyen, Quoc-Viet Pham, Barry O’Sullivan, Hoang D. Nguyen"
  year: "2025"
  link: "https://arxiv.org/abs/2501.06322"
  section: "Section 3"
---

# LLM 多智能体综述中文精读（第 3 篇-02）：协作系统定义与形式化

> 本篇覆盖 **Section 3 Multi-Agent Collaboration Concept**。这一节的目标，是把“多智能体协作”从直觉描述，提升到可分析的系统定义与数学框架。

## 本章导读

> [!note] 译者注（非原文）
> 这节很重要，因为它回答了一个常被忽略的问题：
> **我们到底在形式化什么？**
>
> 作者不是简单说“很多 agent 一起做任务”，而是把 agent、自身目标、环境、协作通道、系统输入输出都定义出来。这样后面的 cooperation / competition / topology / orchestration 才有统一语言可说。

## 正文翻译

> [!warning] 易混点（非原文）
> 这一节的公式主要是“定义型公式”，不是为了推导数值结果，而是为了规定系统里有哪些对象、它们如何关联。

### 原文标题：3.1 Agent and Collaborative System Definition

作者先把一个 agent 形式化表示为：

`a = {m, o, e, x, y}`

其中：
- `m`：模型（model）
- `o`：目标（objective）
- `e`：环境（environment）
- `x`：输入感知（input perception）
- `y`：输出 / 动作（output / action）

作者对各部分进一步解释如下：

1. **模型 `m = {arch, mem, adp}`**
   - `arch`：模型架构；
   - `mem`：agent 自身记忆；
   - `adp`：可选适配器或额外模块。

   在 LLM agent 的场景中，`arch` 通常就是语言模型，而 `mem` 往往对应 agent 的 system prompt 或任务记忆。

2. **目标 `o`**
   表示 agent 的目标函数或行动目标，用来指导它在系统中采取什么行动。

3. **环境 `e`**
   指 agent 所处的上下文与外部条件。在 LLM 场景里，一个很直接的限制就是 context window 的 token 上限。

4. **输入 `x`**
   表示 agent 感知到的输入，如文本或传感器数据。对于 LLM，通常被编码成 token 序列。

5. **输出 `y`**
   由如下函数给出：

   `y = m(o, e, x)`

   也就是说，agent 用自己的模型、目标和环境上下文，对输入作出响应。

作者强调：LLM-based agents 往往先通过大规模预训练获得通用知识，再在协作环境中发挥作用。每个 agent 还可能配备不同外部工具，例如计算器、Python 解释器等。

随后，作者把单个 agent 推广为一个多智能体协作系统 `S`。系统包含：

- `A = {a_i}`：多个 LLM-based agents；
- `O_collab`：系统整体的协作目标；
- `E`：共享环境；
- `C = {c_j}`：一组协作通道 / collaboration channels；
- `x_collab`：系统输入；
- `y_collab`：系统输出。

作者给出系统输出形式：

`y_collab = S(O_collab, E, x_collab | A, C)`

也就是说，系统输出不仅取决于输入、目标和环境，也取决于：
- 有哪些 agent；
- 这些 agent 是通过哪些协作通道互相作用的。

作者尤其强调 `C` —— collaboration channels —— 是整套框架的核心。每条协作通道都可以由以下机制来区分：
- actors
- types
- structures
- strategies

如果两个通道在这些方面不同，就应被视为不同通道。

作者还假设 agent 之间存在某种**共同基础（common ground）**，例如都使用英语、都围绕同一主题交流，否则协作会因接口不兼容而失败。

图 2 展示了作者提出的统一框架：每个 agent 都有模型、目标、环境、输入和输出，而系统的中心关注点是多个协作通道如何连接这些 agent，并促成协调与编排。

作者进一步指出，协作可能发生在不同层次：
- **后期协作**：例如集成多个输出做投票 / 汇总；
- **中期协作**：例如共享模型参数、联邦式聚合；
- **早期协作**：例如共享数据、上下文、环境信息。

> [!note] 译者注（非原文）
> 这个分层很有价值。它告诉你：multi-agent collaboration 不一定只发生在“生成完答案之后大家投票”，也可能在输入、训练、参数、执行中途就发生。

### 原文标题：3.2 Problem Definition

作者接着指出：在 LLM-powered MAS 中，agent 之间协作是必要的，它们共享一个共同目标或一组目标。每次协作至少包含三件事：

1. 委派若干 agent，根据它们的专长与资源分工；
2. 定义它们将如何协作；
3. 在 agent 间形成决策并走向最终目标。

作者将系统再次形式化为：多个 channel 共同作用，驱动协作输出形成。每条 channel 都代表一种允许 agent 一起工作的机制。

这里作者特别强调：
**working together 不只是 communication（信息交换），而是更深层的 coordination and management。**

也就是说：
- 单纯传消息还不够；
- 还要考虑谁来接、何时接、如何分工、如何合并、如何冲突消解。

作者进一步举例说明：
- 在 peer-to-peer 结构里，某些 agent 可以竞争，另一些则合作；
- 一旦 type / structure / strategy 不同，它们就构成不同的 collaboration channel。

这也是为什么后面要把 type、strategy、structure、orchestration 分开讨论：它们本质上是构造 channel 的不同维度。

## 本章小结

> [!note] 译者注（非原文）
> 这一章最关键的四点：
> 1. 单个 agent 被形式化为 `model + objective + environment + input + output`。
> 2. 多智能体系统真正新增的核心对象，不只是 `多个 agent`，而是 `collaboration channels`。
> 3. 作者把协作看成一种系统级机制，而不只是“互相发消息”。
> 4. 只要 actors / types / structures / strategies 不同，就可以视为不同协作通道。

## 术语对照

| 英文术语 | 中文译法 | 本章语境说明 |
|---|---|---|
| collaboration channel | 协作通道 | agent 之间协作发生的机制化通路 |
| common ground | 共同基础 | agent 之间能理解彼此接口与语境 |
| collective objective | 集体目标 / 协作目标 | 系统级整体目标 |
| input perception | 输入感知 | agent 读到的输入信息 |
| coordination | 协调 | 让 agent 行动彼此兼容 |
| orchestration | 编排 / 调度 | 安排执行顺序与依赖关系 |

## 思考题

### 题目 1：为什么作者要把 collaboration channels 设为系统的核心对象？

> [!tip] 理解提示（非原文）
> 想象两个系统都用了 4 个 agent，但一个是中心化指挥，另一个是自由辩论。它们真的能算“同一种系统”吗？

> [!note] 译者注（非原文）
> 不能。因为真正决定系统行为的，不只是 agent 数量，而是 agent 如何通过不同 channel 发生协作。channel 才是多智能体系统的组织学核心。

### 题目 2：为什么作者强调“working together 不只是 communication”？

> [!warning] 易混点（非原文）
> 很多人会把 multi-agent 简化为“agent A 把话发给 agent B”。这只是最低层的表面现象。

> [!note] 译者注（非原文）
> 因为协作还包含任务划分、顺序安排、角色选择、冲突处理、结果聚合等管理层问题。只有消息交换而没有协调与编排，通常无法形成稳定有效的集体智能。

## 相关页面

- [[raw/lessons/Multi-Agent-Survey/03-collaboration-mechanisms/01-collaboration-mechanisms-introduction-background]]
- [[raw/lessons/Multi-Agent-Survey/03-collaboration-mechanisms/03-collaboration-mechanisms-types]]
- [[raw/paper/multi-agent/03-multi-agent-collaboration-mechanisms-a-survey-of-llms-2025.pdf]]

## 下一步学习

- 上一篇：[[raw/lessons/Multi-Agent-Survey/03-collaboration-mechanisms/01-collaboration-mechanisms-introduction-background|01-introduction-background]]
- 下一篇：[[raw/lessons/Multi-Agent-Survey/03-collaboration-mechanisms/03-collaboration-mechanisms-types|03-types]]
