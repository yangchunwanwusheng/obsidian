---
type: lesson
tags: [多智能体, multi-agent, survey, 通信机制, 论文翻译, 学习笔记]
created: 2026-05-15
updated: 2026-05-15
topic: LLM 多智能体综述中文精读（第 4 篇-01）摘要、导论与背景
difficulty: intermediate
prerequisites:
  - [[raw/lessons/Multi-Agent-Survey/04-beyond-self-talk/00-beyond-self-talk-essence]]
sources:
  - https://arxiv.org/abs/2502.14321
status: completed
series:
  name: "Multi-Agent Survey｜Beyond Self-Talk: Communication-Centric LLM-MAS（2025）"
  part: 1
paper_meta:
  paper_id: "multi-agent-survey-04"
  title: "Beyond Self-Talk: A Communication-Centric Survey of LLM-Based Multi-Agent Systems"
  authors: "Bingyu Yan, Zhibo Zhou, Litian Zhang, Lian Zhang, Ziyi Zhou, Dezhuang Miao, Zhoujun Li, Chaozhuo Li, Xiaoming Zhang"
  year: "2025"
  link: "https://arxiv.org/abs/2502.14321"
  section: "Abstract + Section 1 + Section 2"
---

# LLM 多智能体综述中文精读（第 4 篇-01）：摘要、导论与背景

> 本篇覆盖 **Abstract、Section 1 Introduction、Section 2 Background**。核心任务是把论文的问题意识读清楚：作者为什么认为"通信"应该是 LLM-MAS 研究的中心轴线，而不是一个附属话题。

## 本章导读

> [!note] 译者注（非原文）
> 第 4 篇与第 3 篇形成接力关系。第 3 篇从"协作"切入，建立了五维框架；第 4 篇从"通信"切入，提出了两级分析框架。两篇合在一起，基本覆盖了 LLM-MAS 系统设计的核心知识体系。
>
> 本篇的 Section 1 和 Section 2 有一个重要的修辞策略：作者先承认现有综述的贡献，再精准指出它们共同的盲区——"把通信视为自然而然的事情，而没有把通信本身当作一个需要系统研究的设计空间"。这是这篇论文立论的关键。

## 正文翻译

> [!warning] 易混点（非原文）
> 从这里开始进入原文翻译。所有补充解释都会使用 Obsidian callout 明确标注为"非原文"。

### 原文标题：Abstract

随着基于大语言模型的多智能体系统（LLM-MAS）的快速发展，研究者们已经探索了各种应用场景，从协作问题求解到社会模拟。然而，现有综述通常将通信视为多智能体系统的一个附属方面，而非其核心组织原则。

本文提出了一种**以通信为中心的框架（communication-centric framework）**来理解和分析 LLM-MAS。作者认为，通信不仅是 agent 之间交换信息的手段，更是塑造系统架构、决定协作效率、影响智能涌现行为的根本机制。

具体而言，作者将 LLM-MAS 的通信机制分为两个分析层级：
- **系统间通信（System-Level Communication）**：关注整个多智能体系统的通信架构设计，包括架构类型（Architecture）、通信目标（Communication Goals）和通信协议（Communication Protocols）；
- **系统内通信（System Internal Communication）**：关注单次通信行为中的微观机制，包括通信策略（Strategies）、通信范式（Paradigms）、通信对象（Objects）和通信内容（Content）。

通过这一框架，作者系统回顾了现有 LLM-MAS 中的通信设计选择，识别了关键挑战，并指出了面向更高效、更安全、更可扩展的 agent 通信的未来研究方向。

> [!tip] 理解提示（非原文）
> 摘要里最关键的词是 "communication-centric"。作者不是简单地在 multi-agent 话题里多加一个"通信"子话题，而是把整个分析框架重新以通信为轴心来组织。这决定了全文的叙事结构。

### 原文标题：1 Introduction

#### 1.1 从"自言自语"到"对话"

近年来，基于大语言模型的智能体（LLM-based agents）取得了显著进展。单个 LLM agent 已经能够执行复杂的推理、规划和工具使用任务。然而，这些 agent 从根本上仍然是"自言自语的"（self-talking）——它们在自己的推理循环中运作，与外界的交互仅限于工具调用和环境反馈。

作者指出，真正的突破来自于让多个 LLM agent 彼此通信。当一个 agent 不再只是与自己对话，而是与其他 agent 进行有意义的交互时，系统就能展现出远超个体能力之和的集体智能。

但问题在于：**当前的研究重心严重偏向 agent 的个体能力（推理、规划、记忆、工具使用），而 agent 之间的通信机制却被相对忽视了**。大多数 LLM-MAS 系统采用简单、临时的通信设计（例如固定的轮询顺序或简单的消息转发），缺乏对通信本身的系统性思考。

#### 1.2 现有综述的盲区

作者回顾了近年来的多篇 LLM-MAS 综述，指出它们虽然各有贡献，但普遍存在一个共同的盲区：

1. **以 agent 为中心，而非以通信为中心**：大多数综述的结构是按照 agent 的组件（profile、memory、planning、action）来组织的，通信只是"action"下面的一个子话题；
2. **通信被视为实现细节**：现有的综述倾向于把通信当作"agent 之间自然而然发生的事情"，而很少深入分析通信架构、策略、范式等设计选择如何影响系统性能；
3. **缺少统一的通信分析框架**：不同综述使用不同的术语和分类方式来描述通信，导致该领域的知识积累碎片化。

作者因此提出：**需要一篇以通信为第一性原理的综述，将通信从"附属话题"提升为"中心分析维度"**。

#### 1.3 本文的核心贡献

基于以上分析，作者给出了本文的四个主要贡献：

1. **提出以通信为中心的两级分析框架**：系统间通信（System-Level）关注整体架构设计，系统内通信（System Internal）关注微观交互机制。这一框架为理解和比较不同 LLM-MAS 的通信设计提供了统一的语言；

2. **系统回顾现有 LLM-MAS 中的通信设计选择**：涵盖五种架构类型（Flat / Hierarchical / Team / Society / Hybrid）、三种通信目标（Cooperation / Competition / Mixed）、三种新兴通信协议（MCP / A2A / ANP），以及系统内通信的四种策略、三种范式、四类对象和两种内容类型；

3. **识别关键挑战与开放问题**：包括通信协议标准化、多模态通信、通信安全与隐私、通信效率优化、大规模 agent 通信的可扩展性等；

4. **为未来研究提供方向性指引**：从系统设计、通信标准化、安全性和基准评测等角度，给出了具体的研究建议。

#### 1.4 论文结构

论文的后续结构如下：
- **Section 2**：背景知识——LLM-based agents、传统多智能体系统、LLM-MAS
- **Section 3**：系统间通信——架构、通信目标、通信协议
- **Section 4**：系统内通信——策略、范式、对象、内容
- **Section 5**：挑战与未来方向
- **Section 6**：结论

### 原文标题：2 Background

#### 2.1 LLM-Based Agents

作者首先回顾了 LLM-based agent 的基本构成。一个典型的 LLM agent 包含三个核心组件：

- **Brain（大脑）**：由 LLM 提供核心推理和决策能力。LLM 不仅负责语言理解和生成，还承担任务分解、计划制定和决策选择的角色；
- **Perception（感知）**：agent 通过多种模态的输入（文本、图像、代码、结构化数据）感知外部环境。感知模块将原始输入转化为 LLM 可以处理的表示形式；
- **Action（行动）**：agent 通过工具调用（API 调用、代码执行、数据库查询）、环境交互和通信来执行行动。行动模块是 agent 影响外部世界的手段。

作者特别强调，在这三个组件中，**通信（communication）跨越了 Perception 和 Action 两个组件**：agent 既需要通过感知来接收其他 agent 的消息，也需要通过行动来发送自己的消息。这意味着通信天然地处于 agent 架构的核心位置——它不是某个组件的附属功能，而是连接 agent 内部与外部世界的桥梁。

此外，作者还回顾了 agent 的关键能力：
- **推理能力（Reasoning）**：包括逻辑推理、数学推理、常识推理等；
- **规划能力（Planning）**：将复杂任务分解为可执行的子任务序列；
- **记忆能力（Memory）**：包括短期记忆（当前会话上下文）和长期记忆（外部知识库、向量数据库）；
- **工具使用能力（Tool Use）**：调用外部 API、执行代码、查询数据库等。

> [!tip] 理解提示（非原文）
> 作者在这里埋了一个很重要的论点：通信不是 agent 的"第五种能力"，而是横跨感知和行动的基础设施。这个观点为后面把通信作为中心分析维度铺平了道路。

#### 2.2 传统多智能体系统（Traditional MAS）

在 LLM 出现之前，多智能体系统（MAS）已经是人工智能的一个成熟研究领域。传统 MAS 关注的核心问题包括：

- **Agent 架构**：反应式 agent（直接对环境刺激做出反应）vs. 慎思式 agent（基于内部模型进行推理和规划）vs. 混合式 agent；
- **通信语言**：如 KQML（Knowledge Query and Manipulation Language）和 FIPA-ACL（Foundation for Intelligent Physical Agents - Agent Communication Language），定义了 agent 之间交换消息的标准格式和语义；
- **协调机制**：包括合同网协议（Contract Net Protocol）、拍卖机制、投票机制等，用于在多个 agent 之间分配任务和解决冲突；
- **组织模型**：如层级组织、市场模型、社会模型等，定义了 agent 之间的长期结构关系。

作者指出，传统 MAS 的通信研究有两个重要遗产：
1. **通信行为理论（Speech Act Theory）**：来自语言哲学（Austin, Searle），将通信视为一种行动——"说话就是做事"。这一理论深刻影响了传统 MAS 的通信语言设计；
2. **黑板系统（Blackboard Systems）**：一种共享工作空间模型，多个专家 agent 在一个公共的黑板上读写信息，通过协作完成复杂问题求解。

然而，传统 MAS 也面临着根本性局限：agent 的通信能力受限于预定义的通信语言和协议，缺乏 LLM 所具备的自然语言理解与生成的灵活性。

#### 2.3 LLM-Based Multi-Agent Systems（LLM-MAS）

LLM 的出现为 MAS 注入了全新的可能性。与传统 MAS 相比，LLM-MAS 展现出以下关键差异：

**（1）自然语言通信取代形式化通信语言**

传统 MAS 中的 agent 使用 KQML、FIPA-ACL 等形式化语言通信，消息的语义受限于预定义的本体和通信原语。LLM-MAS 则使 agent 能够使用自然语言进行自由、灵活的通信——agent 可以用自然语言表达意图、协商、辩论、解释。

这一转变带来了两方面的影响：
- **优势**：通信的灵活性和表达力大幅提升，agent 能够处理更复杂、更微妙的交互场景；
- **挑战**：自然语言的模糊性和不确定性也引入了新的问题，如误解、歧义、信息过载。

> [!warning] 易混点（非原文）
> 这里要区分两个概念：传统 MAS 也用"通信"这个词，但它指的是 KQML 等形式化消息；LLM-MAS 的通信指的是自然语言对话。两者虽然都叫"通信"，但机制和挑战完全不同。

**（2）从预定义角色到涌现角色**

在传统 MAS 中，agent 的角色和能力通常是预先编程的。LLM-MAS 则可以通过 prompt 动态定义 agent 的角色、知识和行为倾向。这意味着：

- agent 的角色可以在运行时调整；
- agent 可以扮演更复杂的、更像人类的角色（如"批判性评审者"、"创造性头脑风暴者"）；
- 多个 agent 之间可能出现涌现性的角色分化。

**（3）通信作为一种核心能力而非辅助功能**

在传统 MAS 中，通信通常被视为协调的辅助手段——agent 先有能力，然后通过通信来协调这些能力。在 LLM-MAS 中，**通信本身就是能力的来源**：

- agent 的推理可以通过与另一 agent 的辩论来增强；
- agent 的知识可以通过从其他 agent 获取信息来扩展；
- agent 的错误可以通过 peer review 来纠正。

作者总结道：在 LLM-MAS 中，通信不再是协调的附属品，而是智能涌现的核心机制。

**（4）从封闭世界到开放世界**

传统 MAS 通常在封闭、受控的环境中运行，通信的内容和格式都是预定义的。LLM-MAS 则运行在开放世界中，agent 可以：

- 与未知的其他 agent 通信；
- 处理从未见过的通信内容；
- 适应动态变化的通信环境和需求。

## 本章小结

> [!note] 译者注（非原文）
> 这一章建立了理解全文所需的四个核心判断：
>
> 1. **"Beyond Self-Talk" 是全文的核心主张**：LLM-MAS 不能只关注单个 agent 的自我推理，必须把 agent 之间的通信作为第一性原理来研究。
>
> 2. **现有综述的共同盲区**：通信被当作附属话题而非中心分析维度，缺少统一的通信分析框架。
>
> 3. **LLM-MAS 的通信与传统 MAS 的通信有本质区别**：自然语言取代了形式化通信语言，通信从辅助功能变成了智能涌现的核心机制。
>
> 4. **通信跨越 agent 的感知和行动两个组件**：它不是"第五种能力"，而是连接 agent 内部与外部的基础设施。

## 术语对照

| 英文术语 | 中文译法 | 本章语境说明 |
|----------|---------|-------------|
| communication-centric | 以通信为中心的 | 论文的核心方法论立场 |
| self-talk | 自言自语 | 指单 agent 的内部推理循环 |
| system-level communication | 系统间通信 | 整体架构层面的通信设计 |
| system internal communication | 系统内通信 | 单次通信行为中的微观机制 |
| Speech Act Theory | 言语行为理论 | 来自语言哲学，认为说话本身就是行动 |
| KQML / FIPA-ACL | 知识查询与操作语言 / FIPA agent 通信语言 | 传统 MAS 的形式化通信语言标准 |
| Blackboard System | 黑板系统 | 共享工作空间模型 |
| emerging roles | 涌现角色 | 非预定义的、通过交互自然形成的角色分化 |

## 思考题

### 题目 1：为什么作者说"自然语言通信"对 LLM-MAS 既是优势也是挑战？

> [!tip] 理解提示（非原文）
> 想一想：自然语言的灵活性和表达力，在什么场景下反而会成为问题？

> [!note] 译者注（非原文）
> 优势在于：agent 可以表达更复杂、更微妙的意图，而不受预定义通信原语的限制。挑战在于：自然语言本质上是模糊的——同一句话可以被不同 agent 做出不同解读，这可能导致误解、信息偏差在 agent 之间传播放大、以及通信效率下降（因为自然语言消息通常比结构化消息更长）。

### 题目 2：传统 MAS 的"通信"和 LLM-MAS 的"通信"根本差别在哪里？

> [!warning] 易混点（非原文）
> 不要因为两个领域都用"通信"这个词，就认为它们是同一回事。

> [!note] 译者注（非原文）
> 三个根本差别：（1）形式化 vs. 自然语言——传统 MAS 用 KQML/FIPA-ACL 等预定义语言，LLM-MAS 用自由的自然语言对话；（2）辅助功能 vs. 核心能力——传统 MAS 中通信是协调手段，LLM-MAS 中通信本身就是知识和推理的来源；（3）封闭世界 vs. 开放世界——传统 MAS 在受控环境中运行，LLM-MAS 在开放、动态的环境中运行。

## 相关页面

- [[raw/lessons/Multi-Agent-Survey/04-beyond-self-talk/00-beyond-self-talk-essence]]
- [[raw/lessons/Multi-Agent-Survey/03-collaboration-mechanisms/00-collaboration-mechanisms-essence]]

## 下一步学习

- 上一篇：[[raw/lessons/Multi-Agent-Survey/04-beyond-self-talk/00-beyond-self-talk-essence|00-essence]]
- 下一篇：[[raw/lessons/Multi-Agent-Survey/04-beyond-self-talk/02-beyond-self-talk-system-level|02-system-level]]
