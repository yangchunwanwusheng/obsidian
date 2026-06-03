---
type: lesson
tags: [多智能体, multi-agent, survey, 论文翻译, 学习笔记]
created: 2026-05-07
updated: 2026-05-07
topic: LLM 多智能体综述中文精读（第 2 篇-3）复杂任务求解：推理框架、通信优化、资源与评测
difficulty: intermediate
prerequisites:
  - [[raw/lessons/Multi-Agent-Survey/02-recent-advances/12-recent-advances-and-new-frontiers-core-components]]
sources:
  - [[raw/paper/multi-agent/02-a-survey-on-llm-based-multi-agent-system-recent-advances-and-new-frontiers-in-application-2024.pdf]]
  - https://arxiv.org/abs/2412.17481
status: completed
series:
  name: "Multi-Agent Survey｜Recent Advances and New Frontiers in Application（2024）"
  part: 13
paper_meta:
  paper_id: "multi-agent-survey-02"
  title: "A Survey on LLM-based Multi-Agent System: Recent Advances and New Frontiers in Application"
  authors: "Shuaihang Chen, Yuanxing Liu, Wei Han, Weinan Zhang, Ting Liu"
  year: "2024"
  link: "https://arxiv.org/abs/2412.17481"
  section: "Section 3 LLM-MAS for Solving Complex Tasks"
---

# LLM 多智能体综述中文精读（第 2 篇-3）：复杂任务求解

> 本篇覆盖原论文 **Section 3 LLM-MAS for Solving Complex Tasks**。这是整篇论文三大应用分类中的第一类：用多智能体协作来完成任务。

## 本章导读

> [!note] 译者注（非原文）
> 上一节讲了"系统底座长什么样"，这一节开始讲"这些底座最先被拿去做什么"。
>
> 作者关心的不是抽象定义，而是一个实用问题：当任务太长、太难、太需要分工时，多个 LLM agent 到底能不能像一个团队那样工作？如果能，又应该如何被评价？

## 正文翻译

> [!warning] 易混点（非原文）
> 从这里开始是原文第 3 节的中文翻译；所有额外解释都单独标注为非原文。

### 原文标题：3 LLM-MAS for Solving Complex Tasks

完成一个复杂任务通常需要多个角色、多个步骤等。这对单个智能体来说很困难，但多个智能体协同工作则很适合这类任务（Islam et al., 2024）。此外，这些智能体中的每一个都可以被独立训练（Shen et al., 2024; Yu et al., 2024）。与单个智能体相比，LLM-MAS 可以取得更好的结果。也就是说，多智能体协作将提升整体性能（Du et al., 2023）。

> [!tip] 理解提示（非原文）
> 这一节的核心不是"多 agent 一定更强"，而是：**当任务结构本身具有可分工性时，多 agent 更有机会把复杂度拆开处理。**

### 原文标题：3.1 Representative LLM-MAS for Solving Complex Tasks

这一领域目前是一个热门研究方向。最近，研究者主要关注**多智能体推理框架**和**多智能体通信优化**，下面分别讨论。

#### LLM-MAS 推理框架（LLM-MAS reasoning framework）

作者按照推理的流水线总结了三个方面：

**(i) 多阶段框架（multi-stage framework）**

多阶段框架指的是一种流水线，其中智能体在不同阶段充当串行的问题解决者（Qian et al., 2024b）。

> [!note] 译者注（非原文）
> 简单来说，就是把任务按阶段拆开，每个阶段的 agent 完成一部分后传给下一个阶段的 agent。就像工厂流水线：设计 → 编码 → 测试。

**(ii) 集体决策框架（collective decision-making）**

集体决策指的是不同的智能体针对一个目标进行投票或辩论（Zhao et al., 2024c）。

> [!note] 译者注（非原文）
> 这和多阶段流水线不同——集体决策是"多个 agent 同时面对同一个问题"，通过讨论或投票来汇聚不同观点。

**(iii) 自我修正框架（self-refine）**

自我修正指的是 LLM-MAS 中的自我反思机制。

> [!note] 译者注（非原文）
> Self-refine 可以是单 agent 也可以是多 agent。关键特征是：agent 对自己的输出进行反思和迭代修正，而不是一次就定结论。

作者还提到：研究者提出了将多智能体应用于自然科学的框架（Chen et al., 2024a），以增强数据分析、模型模拟和决策过程（Yin et al., 2024）。Zhang et al. (2023a) 提出了一个实现自适应和适应性合作的框架。智能体合作中的 scaling law 也被探索（Qian et al., 2024c），但没有发现显著的规律。

> [!tip] 理解提示（非原文）
> 三种推理框架的区分很重要：
> - 多阶段 = 流水线分工（按阶段拆任务）
> - 集体决策 = 多人讨论（对一个目标汇聚观点）
> - 自我修正 = 迭代优化（对已有结果反思改进）
>
> 大多数实际系统都是三者的混合。

#### LLM-MAS 通信优化（LLM-MAS communication optimization）

LLM-MAS 中的全连接通信可能导致组合爆炸和隐私泄露等问题。基于此，作者在通信优化中总结了两个方面：

**(i) 速度优化（speed optimization）**

速度优化指的是研究者试图加速智能体之间的通信，例如使用非语言通信（Liu et al., 2024b）或更短的生成（Chen et al., 2024g）。

**(ii) 分布式讨论（distributed discussion）**

分布式讨论指的是智能体试图在没有足够信息的情况下解决任务（Liu et al., 2024a）。智能体需要彼此通信以实现目标（Zhang et al., 2023a），即使某个智能体没有完整信息（Liu et al., 2024a）。

> [!warning] 易混点（非原文）
> 不要把"分布式讨论"简单理解为"聊天"。它的核心挑战是：**每个 agent 只有部分信息，必须通过通信来补全全局视图。**
>
> 这和集体决策的区别在于：集体决策强调"多个视角汇聚"，分布式讨论强调"信息不完整条件下的协作"。

### 原文标题：3.2 Resources of LLM-MAS for Solving Complex Tasks

作者在表 1 中总结了常见的和开源的 LLM-MAS 任务求解资源，包括代码、数据集和基准测试。

#### 数据集（Data set）

传统 NLP 任务的所有数据集都可用于评估。此外，遵循 ECL（Qian et al., 2024a），Qian et al. (2024b) 在 SRDD 数据集上评估生成软件的质量，并系统评估智能体在软件开发领域的能力。

#### 开源社区（Open source community）

开源和工业界社区也为 LLM-MAS 的发展做出了重大贡献：

- **MetaGPT**（Hong et al., 2023）：为生成式智能体分配不同角色，形成复杂任务的协作实体。
- **AgentScope**（Gao et al., 2024）：以消息交换为核心通信机制，开发了支持本地和分布式部署之间无缝切换以及自动并行优化的分布式框架。
- **Swarm**（OpenAI, 2024）：一个实验性的多智能体编排框架，注重人体工程学和轻量级。与之前发布的 Assistants API 不同，Swarm 让开发者对上下文、步骤和工具调用拥有细粒度控制。

> [!note] 译者注（非原文）
> 这一段的核心信息是：复杂任务求解已经从"单个系统展示能力"走向"围绕框架、benchmark 和数据集形成研究生态"。

### 原文标题：3.3 Evaluation of LLM-MAS for solving complex task

#### 在特定任务上的表现（Performance on specific tasks）

如表 1 所示，LLM-MAS 的性能可以通过特定任务来评估，这种方式直观且方便。例如：

- 在 APP 系统（Zhang et al., 2023b）中，智能体完成特定任务所需的平均步骤数和工具使用数作为指标；
- 在 BOLAA（Liu et al., 2023c）中，智能体检索的召回率和 QA 准确率作为评估指标；
- 在狼人杀游戏（Xu et al., 2023c）中，虚拟玩家的胜率自然也是评估指标；
- 在招聘会系统（Li et al., 2023）中，招聘方正确招募目标求职者的比例作为评估指标；
- 在拍卖系统（Chen et al., 2024c）中，商品预测价格与实际价格之间的 Spearman 相关系数，以及竞标者的技能也通过 TrueSkill 分数（Graepel et al., 2007）来衡量；
- 在 Stanford Town（Park et al., 2023）中，虚拟智能体和人类智能体生成的行为质量通过人工排序并使用 TrueSkill 评估；
- 在城市模拟系统（Xu et al., 2023a）中，完成导航等特定任务的成功率也是评估指标。

#### 通信成本分析（Communication cost analysis）

首要关注的是系统的运行成本。鉴于当代系统中有相当一部分将 LLM 作为核心模块，系统运行期间产生的额外开销已成为一个关键研究领域。例如，Mou et al. (2024) 使用系统的实际运行时间作为关键指标，强调管理这种运行开销的重要性。

> [!warning] 易混点（非原文）
> 这里最容易犯的错是：看到多 agent 在某个 demo 里做成了一次任务，就以为系统能力已经被证明。
>
> 但从评估视角看，真正的问题是：它是否稳定？是否普遍优于单 agent？增益是不是值得额外成本？

### 关于表 1 的重构

原文 Table 1 整理了复杂任务求解方向的代码与基准测试资源，按推理框架和通信优化两大子方向组织：

**推理框架（Reasoning Framework）**

| 子方向 | 论文 | 数据集 / Benchmark |
|---|---|---|
| 多阶段（Multi-stage） | (Qian et al., 2024b), (Du et al., 2024) | SRDD |
| 多阶段（Multi-stage） | (Yue et al., 2024) | SMART（自建） |
| 多阶段（Multi-stage） | (Liu et al., 2023c) | WebShop |
| 多阶段（Multi-stage） | (Lin et al., 2024) | FG-C, CG-O |
| 多阶段（Multi-stage） | (Islam et al., 2024) | HumanEval, EvalPlus, MBPP, APPS, xCodeEval, CodeContest |
| 多阶段（Multi-stage） | (Shen et al., 2024) | ToolBench, ToolAlpaca |
| 集体决策（Collective Decision-Making） | (Zhao et al., 2024c) | MCQA |
| 集体决策 | (Cheng et al., 2024) | ESConv, P4G |
| 集体决策 | (Liang et al., 2024) | MT-Bench |
| 集体决策 | (Lei et al., 2024) | MATH |
| 集体决策 | (Zhang et al., 2024a) | MMLU, MATH, Chess Move Validity |
| 集体决策 | (Wang et al., 2024d) | TriviaQA |
| 自我修正（Self-Refine） | (Wang et al., 2024c) | FOLIO-wiki |
| 自我修正 | (Chen et al., 2024e) | StrategyQA, CSQA, GSM8K, AQuA, MATH, Date Understanding, ANLI |
| 自我修正 | (Chen et al., 2024a) | TriviaQA |
| 自我修正 | (Tang et al., 2024) | Trans-Review, AutoTransform, T5-Review |
| 自我修正 | (Zhang et al., 2023a) | Overcooked-AI |

**通信优化（Communication Optimization）**

| 子方向 | 论文 | 数据集 / Benchmark |
|---|---|---|
| 速度优化（Speed Optimization） | (Liu et al., 2024b) | HotpotQA, NarrativeQA, MultifieldQA |
| 分布式讨论（Distributed） | (Chen et al., 2024f) | TriviaQA, Natural Questions, HotpotQA, 2WikiMultiHopQA |
| 分布式讨论 | (Liu et al., 2024a) | InformativeBench |

> [!note] 译者注（非原文）
> 表 1 的真正价值是：它把"复杂任务求解"从若干分散案例，整理成了一类可横向比较的研究对象。
>
> 注意：所有论文均有代码链接，这在一定程度上说明复杂任务求解方向的工程成熟度较高。

## 本章小结

> [!note] 译者注（非原文）
> 这一节最值得记住的六点：
> 1. 复杂任务求解是 LLM-MAS 最自然、最核心的一类应用场景。
> 2. 推理框架分三类：多阶段流水线、集体决策（投票/辩论）、自我修正（反思迭代）。
> 3. 通信优化关注两个核心问题：加速通信、在信息不完整条件下协作。
> 4. 作者明显强调资源生态：框架（MetaGPT、AgentScope、Swarm）、benchmark、数据集都很重要。
> 5. 复杂任务的评估不能只看最终答案，还要看通信成本和系统开销。
> 6. 从这里开始，你已经能看到这篇综述的野心：把 LLM-MAS 从"案例展示"推进到"系统研究对象"。

## 术语对照

| 英文术语 | 中文译法 | 本章语境说明 |
|---|---|---|
| solving complex tasks | 复杂任务求解 | 通过多 agent 协作完成复杂目标 |
| reasoning framework | 推理框架 | agent 如何组织和执行推理过程 |
| multi-stage framework | 多阶段框架 | 按流水线阶段分配不同 agent |
| collective decision-making | 集体决策 | 多 agent 通过投票或辩论形成判断 |
| self-refine | 自我修正 / 自反思修正 | 对已有输出进行迭代反思改进 |
| communication optimization | 通信优化 | 减少 agent 间通信的开销 |
| speed optimization | 速度优化 | 加速 agent 间的通信效率 |
| distributed discussion | 分布式讨论 | 在信息不完整条件下通过通信协作 |
| communication cost analysis | 通信成本分析 | 评估多 agent 系统的运行开销 |
| open source community | 开源社区 | 提供框架和工具的研究与工业群体 |

## 思考题

### 题目 1：三种推理框架（多阶段、集体决策、自我修正）分别适合什么类型的任务？

> [!hint]- 思考提示
> 想一想每种框架的核心特征：串行分工 vs 多视角汇聚 vs 迭代优化。

> [!success]- 参考答案
> - **多阶段框架**适合任务本身可以按阶段拆分的场景（如软件开发：设计→编码→测试）。
> - **集体决策**适合需要多视角判断的场景（如多选题、复杂推理、评估打分）。
> - **自我修正**适合输出可以被迭代改进的场景（如代码审查、文本修改、数学推导）。
>
> 实际系统往往是混合的。

### 题目 2：为什么评估复杂任务求解类 LLM-MAS 时不能只看最终答案？

> [!hint]- 思考提示
> 想想通信成本和系统稳定性。

> [!success]- 参考答案
> 因为多智能体系统的价值不仅体现在最终结果，还体现在过程是否合理、通信成本是否可接受、表现是否稳定。只看最终答案，会掩盖冗余通信、偶然成功和高成本换来的提升等问题。

### 题目 3：全连接通信为什么会导致"组合爆炸"？

> [!hint]- 思考提示
> N 个 agent 如果两两之间都要通信，通信数量是多少？

> [!success]- 参考答案
> N 个 agent 全连接需要 $O(N^2)$ 条通信通道。当 N 增大时，通信轮数和 token 消耗呈平方增长，这就是"组合爆炸"。因此需要速度优化和结构化通信来缓解。

## 相关页面

- [[raw/lessons/Multi-Agent-Survey/02-recent-advances/12-recent-advances-and-new-frontiers-core-components]]
- [[raw/lessons/Multi-Agent-Survey/02-recent-advances/14-recent-advances-and-new-frontiers-simulating-specific-scenarios]]
- [[raw/paper/multi-agent/02-a-survey-on-llm-based-multi-agent-system-recent-advances-and-new-frontiers-in-application-2024.pdf]]

## 下一步学习

- 上一篇：[[raw/lessons/Multi-Agent-Survey/02-recent-advances/12-recent-advances-and-new-frontiers-core-components|12-core-components]]
- 下一篇：[[raw/lessons/Multi-Agent-Survey/02-recent-advances/14-recent-advances-and-new-frontiers-simulating-specific-scenarios|14-simulating-specific-scenarios]]
