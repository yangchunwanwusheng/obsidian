---
type: lesson
tags: [多智能体, multi-agent, survey, 架构, CMD, Chain-of-Agents, Agent-Forest, MoA, 论文翻译, 学习笔记]
created: 2026-05-15
updated: 2026-05-15
topic: LLM 多智能体综述中文精读（第 5 篇-01）导论与架构
difficulty: intermediate
prerequisites:
  - [[raw/lessons/Multi-Agent-Survey/05-llms-working-in-harmony/00-llms-harmony-essence]]
sources:
  - https://arxiv.org/abs/2504.01963
status: completed
series:
  name: "Multi-Agent Survey｜LLMs Working in Harmony（2025）"
  part: 1
paper_meta:
  paper_id: "multi-agent-survey-05"
  title: "LLMs Working in Harmony: A Survey on the Technological Aspects of Building Effective LLM-Based Multi Agent Systems"
  authors: "RM Aratchige, Dr. WMKS Ilmini"
  year: "2025"
  link: "https://arxiv.org/abs/2504.01963"
  section: "Section I + Section II-A"
---

# LLM 多智能体综述中文精读（第 5 篇-01）：导论与架构

> 本篇覆盖 **Section I Introduction** 和 **Section II-A Architecture**。核心任务是理解作者的研究动机——为什么要把技术拆成四根支柱，以及当前多智能体架构设计的四种主要范式的优劣。

## 本章导读

> [!note] 译者注（非原文）
> 第 5 篇的气质和前几篇很不一样。它不是从"多智能体有什么用"切入，而是直接用工程化的视角提问：**如果你想搭一个真正的 LLM-MAS，在架构、规划、记忆、工程框架这四个维度上，分别有什么可选的方案？各自优劣是什么？**
>
> 本章先从 Introduction 建立问题意识，然后深入四种架构方案的逐一拆解。

## 正文翻译

> [!warning] 易混点（非原文）
> 从这里开始进入原文翻译。所有补充解释都会单独标成非原文。

### 原文标题：I Introduction

LLM 的出现彻底改变了人工智能。Transformer 架构的提出（"Attention is All You Need"）用注意力机制替代了传统序列模型，大幅提升了机器翻译性能并缩短了训练时间。随后 GPT 等模型展现出前所未有的自然语言处理能力，能处理从文本生成到摘要等多种任务。

尽管 LLM 取得了巨大成功，其应用仍存在局限。幻觉问题依然严重——模型可能在没有外部验证的情况下生成不准确的信息，在需要精确性的场景中限制了可靠性。LLM 在处理复杂或抽象概念时也面临困难——即使 GPT-4 这样的先进模型在模仿人类推理方面也常常表现不佳。

多智能体系统可以缓解这些问题：让不同的 agent 在复杂任务上协作，改善决策质量，尤其对需要深度推理或专业技能的任异效果显著。虽然已有关于 LLM 多智能体系统的研究，但在**识别构建这些系统的最佳技术方法**方面仍存在空白。

本综述旨在通过回答两个研究问题来填补这一空白：

1. **当前有哪些最先进的技术和方法可用于 LLM 多智能体系统？**
2. **其中哪些在实践中最为有效？**

> [!tip] 理解提示（非原文）
> 这篇论文的定位非常明确：不是再写一篇"多智能体全景地图"，而是做"技术选型指南"。两个研究问题直指工程实践。

### 原文标题：II Literature Review

LLM 在多智能体系统中的文献研究仍处于早期阶段，存在显著的研究空白，特别是在理解和改进多智能体范式方面。本综述批判性地审视了 LLM 多智能体系统的当前研究，聚焦于四个技术维度：**Architecture（架构）、Planning（规划）、Memory（记忆）和 Technologies/Frameworks（技术/框架）**。这些方面塑造了 LLM 多智能体系统的潜力和局限性。

### 原文标题：II-A Architecture

多智能体架构在 LLM 领域的研究相对有限。在开发能有效编排多个 agent 协作解决复杂推理任务的框架方面，仍存在重大空白。虽然部分研究已开始应对这些挑战，但仍需要更全面的解决方案。

> [!note] 译者注（非原文）
> 以下四种架构代表了当前 LLM-MAS 架构设计的四个主要方向：讨论式（CMD）、链式（CoA）、投票式（Agent Forest）、分层式（MoA）。

#### II-A.1 Conquer-and-Merge Discussion (CMD)

Wang 等人提出的 CMD 框架利用多个 LLM 驱动的 agent 进行开放式讨论来解决推理任务。其灵感来自 Minsky 的"心智社会"（Society of Mind, 1988），通过模拟人类辩论，每个 agent 贡献不同视角以提升整体推理能力。

CMD 的结构是让一组 agent 对一个问题进行讨论，每个 agent 在几轮交互中生成观点和解释。讨论由共享的历史响应引导，agent 之间可以基于彼此的输入进行构建。

**优势：**
- 性能优于单 agent 方法（如 Chain-of-Thought）
- agent 可以彼此构建和完善推理

**局限：**
- 将 LLM 会话简化为 agent，错失了整合 Tree-of-Thought 等更复杂推理技术的机会
- 仅在推理任务上测试过，是否适用于战略规划或实时决策等更广泛领域尚未验证
- 仅测试了 Bard、Gemini Pro、ChatGPT-3.5 等少数模型，泛化性需进一步评估

> [!tip] 理解提示（非原文）
> CMD 的核心思路是"讨论出真知"。它的强项是让不同 agent 互相纠错、互相补充，但弱项也很明显——讨论机制太简单，没有利用更高级的推理搜索策略。

#### II-A.2 Chain-of-Agents (CoA)

Zhang 等人提出的 Chain-of-Agents (CoA) 框架专门处理超出单个 LLM token 限制的长上下文任务。它由**worker agent** 和 **manager agent** 组成：worker agent 按顺序处理输入的不同片段，将结果传递给 manager agent 进行最终聚合。

CoA 的关键创新是**交错读取-处理方法（interleaved read-process method）**：agent 在接收完整上下文之前就开始处理输入块，从而降低长上下文任务的处理复杂度，并通过分工提高可解释性。

**优势：**
- 有效降低长上下文任务的处理复杂度
- 通过分工提高可解释性

**局限：**
- agent 之间的通信可以进一步优化（通过上下文学习或微调 LLM 来改善交互质量）
- 需要进一步优化计算成本和延迟，特别是在需要多轮 agent 通信的任务中

> [!tip] 理解提示（非原文）
> CoA 解决了一个非常实际的问题：很多真实场景的上下文长度超过了单个 LLM 的 token 限制。与其先用长上下文模型汇总再推理，不如让多个 agent 分段处理、汇总协作。但也正因如此，通信开销和延迟就成了新的瓶颈。

#### II-A.3 Agent Forest

Li 等人提出的 Agent Forest 方法聚焦于通过**简单增加 agent 数量**来扩展 LLM 性能。其核心理念是"More Agents is All You Need"——采用采样-投票方法：多个 agent 生成回答，最终答案通过多数投票决定。

实验表明，agent 数量增加确实能提升性能，尤其是对复杂任务。但这种收益取决于任务固有的难度，对过于复杂或过于简单的任务，收益递减。

**优势：**
- 证明了 agent 数量确实能带来性能增益
- 对复杂任务特别有效

**局限：**
- 性能提升依赖任务难度，对极难或过简单的任务收益递减
- 需要大量 LLM 查询，显著增加计算成本，采样阶段需要优化以提高成本效率

> [!tip] 理解提示（非原文）
> Agent Forest 的思路很直觉化——"人多力量大"。但它揭示了一个重要的缩放规律：agent 数量增加带来的收益是边际递减的。这也意味着，单纯堆 agent 并不能无限提升系统能力。

#### II-A.4 Mixture-of-Agents (MoA)

Wang 等人提出的 Mixture-of-Agents (MoA) 架构采用分层设计：agent 分为 **proposer（提议者）** 和 **aggregator（聚合者）** 两种角色。Proposer 生成多样化的响应，Aggregator 将这些响应合成为高质量的输出。

MoA 在多个基准测试（包括 AlpacaEval 2.0 和 MT-Bench）上表现出色，其核心优势在于**利用不同 LLM 的优势**，让不同 agent 承担专业化角色。

**优势：**
- 充分发挥不同 LLM 的特长
- 在多个基准测试上表现令人印象深刻

**局限：**
- 并非所有 LLM 在两种角色上表现同样出色——例如 WizardLM 作为 proposer 表现出众，但在 aggregator 角色上表现困难
- 增加 agent 和 aggregator 的数量会增加协作管理的复杂度，未来可能需要更复杂的编排技术

> [!tip] 理解提示（非原文）
> MoA 是目前作者最推荐的架构方案。它与 Agent Forest 的关键区别在于：不是简单地让更多 agent 投票，而是通过 proposer-aggregator 的分工，让"产生多样性"和"整合质量"成为两个独立优化的阶段。这本质上是一种"分而治之 + 集成学习"的思路。

## 架构对比总览（Table I 翻译）

| 架构 | 关键特征 | 优势 | 局限 |
|------|---------|------|------|
| CMD | 多 LLM agent 开放式讨论；模拟人类辩论提升推理 | 优于单 agent 方法；agent 可彼此构建 | 缺乏高级推理技术整合；测试领域有限；模型泛化性待验证 |
| CoA | worker agent 顺序处理输入片段；manager agent 汇总结果；交错读取-处理 | 降低长上下文任务复杂度；提高可解释性 | 需改进 agent 间通信；需降低计算成本和延迟 |
| Agent Forest | 多 agent 采样-投票方法；多数投票决定最终答案 | 性能随 agent 数量提升；对复杂任务有效 | 收益依赖任务难度；多 LLM 查询增加计算成本 |
| MoA | proposer + aggregator 分层设计；协作生成并合成高质量输出 | 利用不同 LLM 优势；基准测试表现亮眼 | 角色效果因模型而异；多 agent 增加管理复杂度 |

## 本章小结

> [!note] 译者注（非原文）
> 本章建立了四个关键判断：
> 1. 这篇论文的四柱框架（Architecture / Planning / Memory / Frameworks）直接对应工程实践需求。
> 2. 四种架构范式各有所长：CMD 适合辩论式推理，CoA 适合长文本处理，Agent Forest 适合通过规模提升鲁棒性，MoA 适合追求整体输出质量。
> 3. MoA 的分层设计思路（proposer + aggregator）是最有推广价值的架构范式。
> 4. 所有架构都面临相同的规模化挑战：更多 agent 带来更多性能，但也引入更复杂的管理开销。

## 术语对照

| 英文术语 | 中文译法 | 本章语境说明 |
|----------|---------|-------------|
| proposer | 提议者 | MoA 架构中负责生成多样化响应的 agent |
| aggregator | 聚合者 | MoA 架构中负责整合响应的 agent |
| interleaved read-process | 交错读取-处理 | CoA 架构中的核心处理机制 |
| majority voting | 多数投票 | Agent Forest 中决定最终答案的机制 |
| Society of Mind | 心智社会 | Minsky 提出的理论，多 agent 系统的理论源头 |

## 思考题

### 题目 1：为什么 MoA 的分层设计比 Agent Forest 的纯投票更高效？

> [!tip] 理解提示（非原文）
> 想一想投票和"分工协作"的区别：投票只是让更多人表达意见后取多数，而分工是让不同人做不同的、专业化的事。

> [!note] 译者注（非原文）
> Agent Forest 的投票方法本质上是"重复采样取平均"，它只能减少方差但不能引入新信息。MoA 的分层设计则是"信息提炼"：proposer 负责扩大搜索空间、产生多样性，aggregator 负责压缩提炼、提升质量。这类似于 ensemble learning 中 stacking 优于 simple voting 的道理——前者利用了一个元学习器来学习如何组合基础模型的输出。

### 题目 2：为什么在 CMD 的讨论架构中，仅使用 LLM 的基础对话能力是不够的？

> [!tip] 理解提示（非原文）
> 想一想：如果两个 agent 只是在对话，它们能自动进行深度推理搜索吗？还是容易陷入重复和偏航？

> [!note] 译者注（非原文）
> 因为 LLM 的原始对话能力倾向于"生成下一个可能的 token"，而非"探索推理空间"。在没有 Tree-of-Thought 等搜索机制的情况下，多 agent 的讨论容易陷入局部最优——agent 可能反复重复类似观点，而不是系统地探索逻辑分支、回溯和剪枝。这也是为什么 CMD 需要整合更复杂的推理技术来真正超越单 agent 方法。

## 相关页面

- [[raw/lessons/Multi-Agent-Survey/05-llms-working-in-harmony/00-llms-harmony-essence]]
- [[raw/lessons/Multi-Agent-Survey/05-llms-working-in-harmony/02-llms-harmony-planning]]
- [[raw/lessons/Multi-Agent-Survey/03-collaboration-mechanisms/05-collaboration-mechanisms-structures]]

## 下一步学习

- 上一篇：[[raw/lessons/Multi-Agent-Survey/05-llms-working-in-harmony/00-llms-harmony-essence\|00-essence]]
- 下一篇：[[raw/lessons/Multi-Agent-Survey/05-llms-working-in-harmony/02-llms-harmony-planning\|02-planning]]
