---
type: lesson
tags: [多智能体, multi-agent, survey, 论文翻译, 学习笔记]
created: 2026-05-07
updated: 2026-05-07
topic: LLM 多智能体综述中文精读（第 2 篇-6）挑战、未来方向、结论与局限
difficulty: intermediate
prerequisites:
  - [[raw/lessons/Multi-Agent-Survey/02-recent-advances/15-recent-advances-and-new-frontiers-evaluating-generative-agents]]
sources:
  - [[raw/paper/multi-agent/02-a-survey-on-llm-based-multi-agent-system-recent-advances-and-new-frontiers-in-application-2024.pdf]]
  - https://arxiv.org/abs/2412.17481
status: completed
series:
  name: "Multi-Agent Survey｜Recent Advances and New Frontiers in Application（2024）"
  part: 16
paper_meta:
  paper_id: "multi-agent-survey-02"
  title: "A Survey on LLM-based Multi-Agent System: Recent Advances and New Frontiers in Application"
  authors: "Shuaihang Chen, Yuanxing Liu, Wei Han, Weinan Zhang, Ting Liu"
  year: "2024"
  link: "https://arxiv.org/abs/2412.17481"
  section: "Section 6 Challenges and Future Directions + Section 7 Conclusion + Limitations + Acknowledgments"
---

# LLM 多智能体综述中文精读（第 2 篇-6）：挑战、未来方向与结论

> 本篇覆盖原论文 **Section 6 Challenges and Future Directions**、**Section 7 Conclusion**、**Limitations** 和 **Acknowledgments**。这是整篇论文的收尾部分：作者从智能体自身、交互过程和系统评测三个层面总结挑战，并提出未来方向。

## 本章导读

> [!note] 译者注（非原文）
> 这一部分和第 1 篇综述的"挑战"部分形成互补：
> - 第 1 篇更关注系统性挑战（架构、安全性、伦理）；
> - 第 2 篇更关注从"应用实战"中暴露出的具体问题（幻觉、效率爆炸、累积误差、评测标准缺失）。
>
> 两篇合在一起，基本覆盖了 LLM-MAS 当前已知的主要瓶颈。

## 正文翻译

> [!warning] 易混点（非原文）
> 从这里开始是原文第 6-7 节、局限性和致谢的中文翻译；所有额外解释都单独标注为非原文。

### 原文标题：6 Challenges and Future Directions

虽然之前关于 LLM-MAS 的工作已经取得了许多显著的成功，但该领域仍处于初始阶段，在发展过程中有几个重大挑战需要解决。下面，作者概述了几个关键挑战以及潜在的未来方向。

### 原文标题：6.1 Challenges posed by generative agents

生成式智能体是 LLM-MAS 的组成部分。然而，由于基础模型 LLM 的固有特征，生成式智能体存在一些不足，下面仔细讨论。

#### 挑战

**(i) 模拟中的广义对齐（Generalized alignment for simulation）**

当智能体被用于真实世界模拟时，一个完美的生成式智能体应该能够诚实描绘多样特征（Wang et al., 2024a）。然而，由于基础模型的训练方式（OpenAI et al., 2024），生成式智能体通常无法与模拟对象对齐。

> [!note] 译者注（非原文）
> "广义对齐"指的是：当 agent 被要求扮演某个角色（如一位老年人、一位政治保守派、一位低收入者）时，它是否真的能如实反映该角色应该有的行为模式？
>
> 目前的 LLM 通常倾向于"安全"和"礼貌"，这和某些真实角色的行为并不一致。

**(ii) 幻觉（Hallucination）**

生成式智能体在与其他智能体交互时有一定概率产生幻觉（Du et al., 2023）。各种增强方法可以缓解这个问题，但不能根本解决。

**(iii) 缺乏足够的長文本能力（Lack of sufficient long text capability）**

当处理复杂信息时，生成式智能体会因为缺乏长文本能力而遗忘输入信息（Zhao et al., 2024a）。

> [!tip] 理解提示（非原文）
> 三个挑战的关系：
> - 广义对齐 → agent 不能如实扮演角色
> - 幻觉 → agent 会生成不实信息
> - 长文本不足 → agent 会遗忘上下文
>
> 它们共同指向一个核心问题：**当前 LLM 作为 agent 控制器的基础能力还不够强。**

#### 未来方向

单个智能体或基础模型能力的提升一直是一个热门话题。研究者专注于增强对齐、减少幻觉和改善长文本能力。新一代 OpenAI 模型 o1 的提出为研究者提供了新思路，即使用更复杂的推理来增强模型能力。

> [!note] 译者注（非原文）
> 作者提到的 o1 模型是 2024 年 9 月发布的推理增强模型。其核心思想是：通过更长的推理链（chain-of-thought）来提升模型在复杂任务上的表现。

### 原文标题：6.2 Challenges posed by interactions

由于 LLM-MAS 的复杂性、自回归特性等特征，系统在实际应用中存在许多问题。下面列出两个主要问题：效率爆炸和累积效应。

#### 挑战

**(i) 效率爆炸（Efficiency explosion）**

由于 LLM 的自回归架构，推理速度通常较慢。然而，生成式智能体需要为每个动作多次查询 LLM，例如从记忆中提取信息、在采取行动前制定计划等。当 LLM-MAS 规模扩大时，这个问题会被放大，特别是对于具有大行动空间的生成式智能体。SoMoSiMu-Bench（Mou et al., 2024）用基于规则的智能体替代边缘生成式智能体来缓解这个问题。但对于具有复杂行动空间的生成式智能体的 LLM-MAS，这个问题仍未解决。

> [!note] 译者注（非原文）
> 效率爆炸的根源：
> - 单次 LLM 推理已经慢（自回归生成）；
> - 一个 agent 一次行动需要多次 LLM 调用（记忆检索 + 规划 + 行动生成）；
> - N 个 agent 同时运行 → 调用次数呈乘法增长。
>
> 所以规模化是多智能体系统走向实际应用的硬约束。

**(ii) 累积效应（Accumulative Effect）**

由于 LLM-MAS 的每一轮都基于上一轮的结果，而且 LLM-MAS 对后续结果有很大影响。研究者已经使用基于规则的模型进行中间纠错（Chen et al., 2024c），但仍有很大的改进空间。IOA（Chen et al., 2024f）提出了一种类互联网通信架构，使 LLM-MAS 更具可扩展性，增强了对动态任务的适应性。

> [!warning] 易混点（非原文）
> 累积效应 ≠ 简单的错误传播。
>
> 累积效应意味着：前一轮的小偏差（不一定显式表现为"错误"）会逐渐改变后续所有 agent 的行为轨迹，最终可能导致整个系统偏离预期。
>
> 这在模拟场景中尤其危险：如果虚拟社会的早期行为稍有偏差，后续所有"社会演化"都会偏离真实。

#### 未来方向

工业界和学术界一直在努力降低 LLM-MAS 的通信成本，例如基于对齐的方法 OPTIMA（Chen et al., 2024g）和工业化并行消息方法 AgentScope（Gao et al., 2024），但仍处于基础阶段，具有较大的研究空间。

### 原文标题：6.3 Challenges of Evaluating for LLM-MAS

#### 缺乏群体行为的客观指标（Lack of objective metrics for group behavior）

如第 4.3 节所示，由于多智能体环境的多样性、复杂性和不可预测性，很难从当前工作中获得足够详细、具体和直接的系统评估指标。目前，研究者主要通过比较系统与真实环境的分布来评估 LLM-MAS，这缺乏对 LLM-MAS 运行过程的细节评估。

#### 自动化评估与基准测试（Automated evaluation and benchmark）

同类不同 LLM-MAS 之间无法比较，因为缺乏针对 LLM-MAS 的基准测试。此外，还缺乏一个既可用于个体评估又可用于总体评估的通用基准框架，该框架能够评估大多数 LLM-MAS。

> [!note] 译者注（非原文）
> 评测挑战可以概括为两个问题：
> 1. 群体行为没有好的量化指标；
> 2. 不同系统之间没有统一的比较标准。

#### 未来方向

研究大规模 LLM-MAS 将成为新的研究热点，研究者将从中评估和发现新的规模效应。与此同时，通用测试基准和评估方法也将在未来研究中出现。

### 原文标题：7 Conclusion

在本综述中，作者系统总结了基于 LLM 的多智能体系统（LLM-MAS）领域的现有研究。他们从三个应用方面展示和回顾了这些研究：任务求解、模拟以及 LLM-MAS 的评估。作者提供了详细的分类法，以在现有研究之间建立联系，总结了每个方面的主要技术及其发展历史。除了回顾之前的工作，作者还提出了该领域的几个挑战，预计将指导潜在的未来方向。

### 原文标题：Limitations

作者已尽最大努力，但一些局限性可能仍然存在。由于页面限制，作者只能对每种方法进行简要总结，而没有详尽的技术细节。另一方面，作者主要从 ACL、NeurIPS、ICLR、AAAI 和 arXiv 收集研究，有可能遗漏了在其他场所发表的重要工作。

在应用方面，作者主要在表 1、表 2 和表 3 中列出了有开源代码的代表性 LLM-MAS 资源。更完整的论文可以在 https://github.com/bianhua-12/Multi-generative_Agent_System_survey 找到。

作者认识到其工作的时效性，并将紧跟研究社区内的讨论，在未来更新观点并补充被忽略的工作。

### 原文标题：Acknowledgments

本研究得到了国家重点研发计划（No. 2022YFF0902100）和黑龙江省自然科学基金（YQ2021F006）的支持。

## 本章小结

> [!note] 译者注（非原文）
> 这一节最值得记住的七点：
> 1. 挑战分为三层：智能体自身（对齐/幻觉/长文本）、交互过程（效率爆炸/累积效应）、评测（群体指标缺失/benchmark 缺失）。
> 2. 广义对齐问题是模拟场景独有的：agent 能否如实扮演目标角色？
> 3. 效率爆炸是规模化的硬约束：N 个 agent × 多次调用 × 自回归推理 = 成本爆炸。
> 4. 累积效应是长链条系统的系统性风险：小偏差逐渐放大。
> 5. 群体行为没有好的量化指标——这是模拟和评测方向共同的瓶颈。
> 6. 作者的局限性声明诚实且具体：页面限制、检索范围、时效性。
> 7. 从第一篇到第二篇，你已经完整读了两篇 LLM-MAS 综述，覆盖了从基本骨架到应用前沿的全景。

## 术语对照

| 英文术语 | 中文译法 | 本章语境说明 |
|---|---|---|
| generalized alignment | 广义对齐 / 泛化对齐 | agent 能否如实扮演目标角色 |
| hallucination | 幻觉 | LLM 生成不实信息 |
| long text capability | 长文本能力 | 处理长输入而不遗忘的能力 |
| efficiency explosion | 效率爆炸 | 多 agent 同时调用 LLM 导致成本剧增 |
| accumulative effect | 累积效应 | 前一轮偏差不断影响后一轮 |
| group behavior | 群体行为 | 多 agent 系统的整体动态 |
| objective metrics | 客观指标 | 可量化、可比较的评测标准 |
| automated evaluation | 自动化评估 | 不依赖人工的评估方法 |
| scale effect | 规模效应 | 系统规模变化时表现出的新现象 |
| autoregressive | 自回归 | 逐 token 生成文本的模型架构 |

## 思考题

### 题目 1：广义对齐和传统 RLHF 对齐有什么不同？

> [!hint]- 思考提示
> 传统对齐追求"安全有用"，广义对齐追求什么？

> [!success]- 参考答案
> 传统 RLHF 对齐追求模型输出安全、有用、符合人类偏好。广义对齐追求的是：当 agent 被要求扮演某个特定角色时，它的行为是否如实反映了该角色应有的特征（如偏见、情绪、认知局限等）。两者方向不同——前者要"去掉偏差"，后者要"保留偏差"。

### 题目 2：为什么累积效应在模拟场景中比在任务求解场景中更危险？

> [!hint]- 思考提示
> 想想模拟场景的目标函数和任务求解有什么不同。

> [!success]- 参考答案
> 任务求解场景有明确的最终目标（如代码写对、问答正确），偏差可以通过最终结果来检测。模拟场景的目标是"拟合真实"，但"真实"本身是复杂的——早期的小偏差会导致后续的群体行为完全偏离真实，而你可能很难在短时间内发现，因为模拟本身就是探索未知的过程。

### 题目 3：作者为什么把"研究大规模 LLM-MAS"标记为未来研究热点？

> [!hint]- 思考提示
> 想想当前大多数研究用了多少 agent。

> [!success]- 参考答案
> 当前大多数 LLM-MAS 研究使用的 agent 数量很少（通常 2-25 个），而真实世界的社会模拟、城市模拟等场景需要成千上万个 agent。Pan et al. (2024) 已经尝试了 $10^6$ 规模，但还非常初步。大规模化是否会带来新的涌现行为、新的效率瓶颈、新的评测需求，都是开放问题。

## 相关页面

- [[raw/lessons/Multi-Agent-Survey/02-recent-advances/15-recent-advances-and-new-frontiers-evaluating-generative-agents]]
- [[raw/lessons/Multi-Agent-Survey/02-recent-advances/17-recent-advances-and-new-frontiers-references]]
- [[raw/lessons/Multi-Agent-Survey/01-progress-and-challenges/05-survey-of-progress-and-challenges-supporting-tools-and-challenges]]
- [[raw/paper/multi-agent/02-a-survey-on-llm-based-multi-agent-system-recent-advances-and-new-frontiers-in-application-2024.pdf]]

## 下一步学习

- 上一篇：[[raw/lessons/Multi-Agent-Survey/02-recent-advances/15-recent-advances-and-new-frontiers-evaluating-generative-agents|15-evaluating-generative-agents]]
- 下一篇：[[raw/lessons/Multi-Agent-Survey/02-recent-advances/17-recent-advances-and-new-frontiers-references|17-references]]
