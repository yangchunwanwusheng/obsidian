---
type: lesson
tags: [多智能体, multi-agent, survey, 论文翻译, 学习笔记]
created: 2026-05-07
updated: 2026-05-07
topic: LLM 多智能体综述中文精读（第 2 篇-5）评估生成式智能体：评价与训练
difficulty: intermediate
prerequisites:
  - [[raw/lessons/Multi-Agent-Survey/02-recent-advances/14-recent-advances-and-new-frontiers-simulating-specific-scenarios]]
sources:
  - [[raw/paper/multi-agent/02-a-survey-on-llm-based-multi-agent-system-recent-advances-and-new-frontiers-in-application-2024.pdf]]
  - https://arxiv.org/abs/2412.17481
status: completed
series:
  name: "Multi-Agent Survey｜Recent Advances and New Frontiers in Application（2024）"
  part: 15
paper_meta:
  paper_id: "multi-agent-survey-02"
  title: "A Survey on LLM-based Multi-Agent System: Recent Advances and New Frontiers in Application"
  authors: "Shuaihang Chen, Yuanxing Liu, Wei Han, Weinan Zhang, Ting Liu"
  year: "2024"
  link: "https://arxiv.org/abs/2412.17481"
  section: "Section 5 LLM-MAS for Evaluating Generative Agents"
---

# LLM 多智能体综述中文精读（第 2 篇-5）：评估生成式智能体

> 本篇覆盖原论文 **Section 5 LLM-MAS for Evaluating Generative Agents**。这是三大应用分类中的第三类：把多智能体系统本身当成评测或训练环境来使用。

## 本章导读

> [!note] 译者注（非原文）
> 这一节是整篇论文里最有"元视角"的部分。
>
> 前两类应用——复杂任务求解和场景模拟——都是在"用系统做事情"。但这一类不同：研究者把 LLM-MAS 本身当成了一个**工具**，用来评估或训练生成式智能体。
>
> 也就是说，多智能体系统从"被研究对象"变成了"研究工具"。

## 正文翻译

> [!warning] 易混点（非原文）
> 从这里开始是原文第 5 节的中文翻译；所有额外解释都单独标注为非原文。

### 原文标题：5 LLM-MAS for Evaluating Generative Agents

随着 LLM 在社区中的盛行，如何评估 LLM 的能力是一个开放问题。现有的评估方法存在以下不足：

**(i) 评估能力受限（constrained evaluation abilities）**
**(ii) 基准测试脆弱（vulnerable benchmarks）**
**(iii) 指标不客观（unobjective metrics）**

LLM-MAS 的复杂性和多样性表明，LLM-MAS 可以用于评估 LLM。然而，如何设计具体的评估指标和评估方法仍然困扰着研究者。

同样，LLM-MAS 也可以用于训练生成式智能体。作者总结了训练的三个方面：

**(i) 监督微调（Supervised Fine-Tuning, SFT）**
**(ii) 强化学习（Reinforcement Learning, RL）**
**(iii) 合成训练数据（Synthesizing data for training）**

> [!tip] 理解提示（非原文）
> 注意这个分类：评估 + 训练。LLM-MAS 既可以作为"考官"（评估 agent 能力），也可以作为"健身房"（提供训练信号和数据）。
>
> 这是三大应用中最具"元视角"的一类。

### 原文标题：5.1 Representative LLM-MAS for Evaluating Generative Agents

LLM-MAS 可以向智能体提供奖励，这些奖励可以用于评估或训练生成式智能体，下面分别讨论。

#### 生成式智能体的评估（Evaluation of generative agents）

研究者通过将生成式智能体放入 LLM-MAS 中来研究它们。在 LLM-MAS 中，研究者可以进一步研究 LLM 在不同场景中的策略能力，例如：

- **长期策略能力**（Chen et al., 2024c）
- **领导策略**（Xu et al., 2023c）
- **竞争策略**（Zhao et al., 2024b）

在情感领域，**MuMA-ToM**（Shi et al., 2024）用于评估智能体通过视频和文本描述在真实家庭环境中理解和推理人类交互的能力。

> [!note] 译者注（非原文）
> 这些评估方式和传统 NLP benchmark 有本质不同：
> - 传统方式：给定问题 → 给出答案 → 比较正确率
> - LLM-MAS 评估：把 agent 放进一个多智能体环境 → 观察它的策略、领导力、竞争力等**动态能力**
>
> 这种方式更灵活，也更难出现数据泄露问题。

#### 生成式智能体的训练（Training on generative agents）

**监督微调（SFT）方面**：Li et al. (2024c) 增强数据以在 LLM-MAS 上对生成式智能体进行监督微调。

**多智能体强化学习（MARL）方面**：Xu et al. (2023c) 创建了生成式智能体，通过提出一个新框架来克服 LLM 的内在偏差，该框架使用多智能体强化学习来增强生成式智能体。

对于 LLM-MAS，Yue et al. (2024) 将知识密集型任务中的复杂轨迹拆分为子任务，提出了多智能体框架的共训练范式——**长短轨迹学习（Long- and Short-Trajectory Learning）**——在保持每个智能体细粒度性能的同时确保协同。

> [!note] 译者注（非原文）
> "长短轨迹学习"的思路是：长轨迹学习全局协同，短轨迹学习局部能力，两者结合来训练多智能体。

**RLHF 方面**：RLHF 因其高成本而受到批评。Liu et al. (2023a) 提出了一种基于多智能体系统的对齐方案，有效解决了与基于奖励的 RL 优化相关的不稳定性和奖励博弈问题。

**合成数据方面**：无论哪种方式，LLM-MAS 本质上被视为 RL 中的环境，通过不同方式从环境中获取奖励。

> [!warning] 易混点（非原文）
> 这里有一个关键区分：
> - **评估（Evaluation）**：观察 agent 在环境中的表现，不更新参数
> - **训练（Training）**：利用环境反馈来更新 agent 参数（SFT / RL / 合成数据）
>
> 两者都把 LLM-MAS 当环境，但目标不同。

### 原文标题：5.2 Resources of LLM-MAS for evaluations

表 3 展示了作者总结的带有代码、数据集和基准测试的工作，作为未来研究者的参考。

### 关于表 3 的重构

原文 Table 3 整理了评估方向的代码与基准测试资源：

**生成式智能体评估（Evaluation of generative agents）**

| 子方向 | 论文 | 数据集 / Benchmark |
|---|---|---|
| 策略（Strategy） | (Liu et al., 2023b) | AGENTBENCH |
| 策略 | (Bandi and Harrasse, 2024) | MT-Bench |
| 策略 | (Chan et al., 2023) | ChatEval |
| 策略 | (Chen et al., 2024d) | LLMARENA |
| 策略 | (Xu et al., 2023b) | MAgIC |
| 策略 | (Huang et al., 2024a) | MLAgentBench |
| 策略 | (Chen et al., 2024c) | AUCARENA |
| 情感（Emotion） | (Zhang et al., 2024b) | PsySafe |
| 情感 | (Shi et al., 2024) | MuMA-ToM |

**生成式智能体训练（Training on generative agents）**

| 子方向 | 论文 | 数据集 / Benchmark |
|---|---|---|
| 在 LLM-MAS 上进行 SFT | (Li et al., 2024c) | MT-Bench, AlpacaEval |
| 在 LLM-MAS 上进行 MARL | (Xu et al., 2023c) | 无 |
| 合成数据 | (Liu et al., 2023a) | HH, Moral Stories, MIC, ETHICS-Deontology, TruthfulQA |

> [!note] 译者注（非原文）
> 对比三个表：
> - 表 1（任务求解）：子方向多、benchmark 丰富
> - 表 2（模拟）：benchmark 较少
> - 表 3（评估）：子方向清晰但训练方向的资源明显不足
>
> 这也反映了研究成熟度的梯度。

## 本章小结

> [!note] 译者注（非原文）
> 这一节最值得记住的五点：
> 1. LLM-MAS 从"被研究对象"变成了"研究工具"——既可以评估也可以训练生成式智能体。
> 2. 传统评估方法有三个不足：能力受限、benchmark 脆弱、指标不客观。
> 3. LLM-MAS 评估的核心优势是：动态、灵活、更难出现数据泄露。
> 4. 训练方面涵盖三条路径：SFT、MARL、合成数据。
> 5. 训练方向的公开资源明显不足，这也是作者标记的研究空间。

## 术语对照

| 英文术语 | 中文译法 | 本章语境说明 |
|---|---|---|
| evaluating generative agents | 评估生成式智能体 | 用多 agent 环境来评测单个 agent 能力 |
| constrained evaluation abilities | 评估能力受限 | 传统评估方法无法覆盖动态交互能力 |
| vulnerable benchmarks | 脆弱的基准测试 | benchmark 容易被泄露或过拟合 |
| unobjective metrics | 不客观的指标 | 评估标准依赖主观判断 |
| Supervised Fine-Tuning (SFT) | 监督微调 | 使用标注数据对模型进行微调 |
| Reinforcement Learning (RL) | 强化学习 | 通过奖励信号优化策略 |
| synthesizing data | 合成数据 | 通过系统交互自动生成训练数据 |
| reward gaming | 奖励博弈 | 模型利用奖励函数漏洞获得高分 |
| Long- and Short-Trajectory Learning | 长短轨迹学习 | 同时学习全局协同和局部能力 |
| RLHF | 基于人类反馈的强化学习 | 利用人类偏好信号训练模型 |

## 思考题

### 题目 1：为什么说 LLM-MAS 评估比传统 NLP benchmark "更难出现数据泄露"？

> [!hint]- 思考提示
> 想想传统 benchmark 和多智能体环境的区别。

> [!success]- 参考答案
> 传统 benchmark 的测试数据是固定的，模型可能在训练时见过题目。而 LLM-MAS 评估是动态的：agent 需要在一个多交互环境中做出策略决策，场景和对手行为都是动态变化的，无法提前记忆所有可能的交互路径。

### 题目 2：评估（evaluation）和训练（training）在使用 LLM-MAS 时的本质区别是什么？

> [!hint]- 思考提示
> 关键在于"参数是否被更新"。

> [!success]- 参考答案
> 评估时，agent 在环境中被观察但不更新参数，目的是了解其当前能力。训练时，环境反馈被用来更新 agent 的参数（通过 SFT、RL 或合成数据），目的是提升其能力。两者都把 LLM-MAS 当作环境，但目标不同。

### 题目 3：为什么 RLHF 的成本很高？多智能体方案如何试图缓解这个问题？

> [!hint]- 思考提示
> RLHF 需要什么？多 agent 方案用什么来替代？

> [!success]- 参考答案
> RLHF 需要大量人工标注偏好数据来训练奖励模型，成本很高。多智能体方案（如 Liu et al., 2023a）通过让多个 agent 在系统中自动交互来生成对齐信号，减少对人工标注的依赖，从而缓解成本问题。

## 相关页面

- [[raw/lessons/Multi-Agent-Survey/02-recent-advances/14-recent-advances-and-new-frontiers-simulating-specific-scenarios]]
- [[raw/lessons/Multi-Agent-Survey/02-recent-advances/16-recent-advances-and-new-frontiers-challenges-future-and-conclusion]]
- [[raw/paper/multi-agent/02-a-survey-on-llm-based-multi-agent-system-recent-advances-and-new-frontiers-in-application-2024.pdf]]

## 下一步学习

- 上一篇：[[raw/lessons/Multi-Agent-Survey/02-recent-advances/14-recent-advances-and-new-frontiers-simulating-specific-scenarios|14-simulating-specific-scenarios]]
- 下一篇：[[raw/lessons/Multi-Agent-Survey/02-recent-advances/16-recent-advances-and-new-frontiers-challenges-future-and-conclusion|16-challenges-future-and-conclusion]]
