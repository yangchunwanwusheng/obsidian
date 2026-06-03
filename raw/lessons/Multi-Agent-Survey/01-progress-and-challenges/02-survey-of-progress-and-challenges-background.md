---
type: lesson
tags: [多智能体, multi-agent, survey, 论文翻译, 学习笔记]
created: 2026-05-07
updated: 2026-05-07
topic: LLM 多智能体综述中文精读（二）背景：单智能体能力与单体/多体差异
difficulty: intermediate
prerequisites:
  - [[raw/lessons/Multi-Agent-Survey/01-progress-and-challenges/01-survey-of-progress-and-challenges-introduction]]
sources:
  - [[raw/paper/multi-agent/01-large-language-model-based-multi-agents-survey-of-progress-and-challenges-2024.pdf]]
  - https://arxiv.org/abs/2402.01680
status: completed
series:
  name: "Multi-Agent Survey｜Progress and Challenges（2024）"
  part: 2
paper_meta:
  paper_id: "multi-agent-survey-01"
  title: "Large Language Model based Multi-Agents: A Survey of Progress and Challenges"
  authors: "Taicheng Guo, Xiuying Chen, Yaqi Wang, Ruidi Chang, Shichao Pei, Nitesh V. Chawla, Olaf Wiest, Xiangliang Zhang"
  year: "2024"
  link: "https://arxiv.org/abs/2402.01680"
  section: "Section 2 Background"
---

# LLM 多智能体综述中文精读（二）：背景

> 本篇覆盖原论文 **Section 2 Background**，用于搭建进入多智能体之前的最小前置：单智能体 LLM agent 已经具备什么能力？它与多智能体系统到底差在哪里？

## 本章导读

> [!note] 译者注（非原文）
> 这一节虽然短，但非常重要。因为作者并不是凭空开始讲多智能体，而是先默认你理解：
> - 单个 LLM agent 已经能做哪些事；
> - 多智能体不是“单体更大更强”的延伸，而是**问题结构改变**后的系统升级。
>
> 如果这一层没分清，后面看“画像设定、通信结构、能力获取”时就很容易把多智能体误解成“多轮 prompt engineering”。

## 正文翻译

> [!warning] 易混点（非原文）
> 从这里开始是原文第 2 节内容的中文翻译；中间若有额外解释，会单独标明为非原文。

### 原文标题：2 Background

### 原文标题：2.1 Single-Agent Systems Powered LLMs

作者首先介绍由 LLM 驱动的单智能体系统（single-agent systems powered by LLMs）的能力背景，这部分主要沿用了已有讨论。

#### 决策式思考（Decision-making Thought）

这个术语指的是：在提示（prompts）的引导下，基于 LLM 的智能体能够把复杂任务拆解成更小的子目标，针对每一部分进行系统化思考（有时还会探索多条可能路径），并从过去的经验中学习，以便在复杂任务中做出更好的决策。

这种能力提升了单个基于 LLM 的智能体的自主性（autonomy），也增强了其解决问题的有效性。

#### 工具使用（Tool-use）

基于 LLM 的智能体具备工具使用能力，意味着它们可以调用外部工具与资源来完成任务。这种能力增强了它们的功能性，使其能够在多样且动态变化的环境中更有效地行动。

#### 记忆（Memory）

这里的“记忆能力”指的是：基于 LLM 的智能体可以使用**上下文中的学习（in-context learning）** 作为短期记忆，也可以使用**外部向量数据库（external vector database）** 作为长期记忆，从而在较长时间范围内保存和检索信息。

这种能力使得单个基于 LLM 的智能体能够维持上下文一致性（contextual coherence），并增强其从交互中学习的能力。

> [!tip] 理解提示（非原文）
> 作者这里列出的三项背景能力，其实对应了今天很多 agent 系统里的三块经典能力：
> - **思考/规划**
> - **调用工具/执行动作**
> - **记忆/检索历史**
>
> 你也可以把它们理解为：单智能体已经可以“想、做、记”。多智能体研究则是在问：**当不止一个 agent 同时存在时，系统会新增哪些结构性问题？**

### 原文标题：2.2 Single-Agent VS. Multi-Agent Systems

由 LLM 赋能的单智能体系统已经展示出鼓舞人心的认知能力。此类系统的构建重点，通常放在其**内部机制**的设计，以及它与**外部环境**之间的交互方式上。

与之相对，LLM 多智能体系统（LLM-MA systems）更强调：

- 多样化的智能体画像（diverse agent profiles）
- 智能体之间的交互（inter-agent interactions）
- 集体决策过程（collective decision-making processes）

从这个角度看，更动态、更复杂的任务，可以通过多个自治智能体的协作来处理。每个智能体都被赋予独特的策略与行为方式，并与其他智能体进行通信。

> [!note] 译者注（非原文）
> 这段话虽然看起来平淡，但它实际上给出了一个非常关键的分界：
>
> **单智能体的核心问题**往往是：
> - 我这个 agent 自己怎样更会想？
> - 怎样更会用工具？
> - 怎样更会记忆？
>
> **多智能体的核心问题**则会变成：
> - 该有哪些不同角色？
> - 角色之间如何通信？
> - 决策如何协调？
> - 多个局部能力怎样变成整体协作能力？
>
> 所以，多智能体不只是“在单智能体外面再套一层壳”，而是把研究焦点从**单个认知体**扩展到**协作系统本身**。

## 本章小结

> [!note] 译者注（非原文）
> 这一节可以压缩成两个结论：
> 1. 单智能体 LLM agent 的典型基础能力包括：决策式思考、工具使用、记忆。
> 2. 多智能体系统真正新增的不是“更强的单体能力”，而是“多角色 + 多交互 + 集体决策”的系统复杂性。
>
> 换句话说，作者是在为后面第 3 节铺路：既然多智能体比单智能体多出了这些结构性问题，那就必须专门分析**接口、画像、通信、能力获取**四个维度。

## 术语对照

| 英文术语 | 中文译法 | 本章语境说明 |
|---|---|---|
| Decision-making Thought | 决策式思考 | 指任务拆解、逐步推理、经验学习的能力 |
| Tool-use | 工具使用 | 指调用外部 API、解释器、资源等 |
| Memory | 记忆 | 包括短期上下文记忆和长期外部记忆 |
| in-context learning | 上下文学习 | 作为短期记忆的一种表现形式 |
| vector database | 向量数据库 | 常用于长期记忆和检索增强 |
| inter-agent interactions | 智能体之间的交互 | 多智能体系统的核心新增维度 |
| collective decision-making | 集体决策 | 多个 agent 协同形成最终判断 |

## 思考题

### 题目 1：如果一个单智能体已经能规划、用工具、带记忆，为什么还需要多智能体？

> [!tip] 理解提示（非原文）
> 不要只从“能力更强”角度想，也从“任务结构更复杂”角度想。

> [!note] 译者注（非原文）
> 因为很多问题不是“一个人更聪明一点”就能解决，而是天然需要**角色分工、信息交换、相互校正与协作决策**。例如软件开发、多人博弈、社会模拟、政策推演等，这些任务的结构本身就更接近“团队问题”而不是“个人问题”。

### 题目 2：单智能体与多智能体的研究焦点有什么本质不同？

> [!warning] 易混点（非原文）
> 不要把差异只理解成“参数量更多”或“上下文更长”。

> [!note] 译者注（非原文）
> 单智能体更关注一个 agent 的内部能力设计；多智能体更关注多个 agent 之间的角色组织、通信方式和协作机制。前者核心是“一个主体如何更聪明”，后者核心是“多个主体如何一起形成有效系统”。

## 相关页面

- [[raw/lessons/Multi-Agent-Survey/01-progress-and-challenges/01-survey-of-progress-and-challenges-introduction]]
- [[raw/lessons/Multi-Agent-Survey/01-progress-and-challenges/03-survey-of-progress-and-challenges-dissecting-llm-ma]]
- [[raw/paper/multi-agent/01-large-language-model-based-multi-agents-survey-of-progress-and-challenges-2024.pdf]]

## 下一步学习

- 上一篇：[[raw/lessons/Multi-Agent-Survey/01-progress-and-challenges/01-survey-of-progress-and-challenges-introduction|01-introduction]]
- 下一篇：[[raw/lessons/Multi-Agent-Survey/01-progress-and-challenges/03-survey-of-progress-and-challenges-dissecting-llm-ma|03-dissecting-llm-ma]]
