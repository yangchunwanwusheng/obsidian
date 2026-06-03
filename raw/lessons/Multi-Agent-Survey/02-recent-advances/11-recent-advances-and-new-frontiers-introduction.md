---
type: lesson
tags: [多智能体, multi-agent, survey, 论文翻译, 学习笔记]
created: 2026-05-07
updated: 2026-05-07
topic: LLM 多智能体综述中文精读（第 2 篇-1）摘要、导论与应用框架
difficulty: intermediate
prerequisites:
  - [[raw/paper/multi-agent/00-reading-manual]]
  - [[raw/lessons/Multi-Agent-Survey/02-recent-advances/10-recent-advances-and-new-frontiers-overview]]
sources:
  - [[raw/paper/multi-agent/02-a-survey-on-llm-based-multi-agent-system-recent-advances-and-new-frontiers-in-application-2024.pdf]]
  - https://arxiv.org/abs/2412.17481
status: completed
series:
  name: "Multi-Agent Survey｜Recent Advances and New Frontiers in Application（2024）"
  part: 11
paper_meta:
  paper_id: "multi-agent-survey-02"
  title: "A Survey on LLM-based Multi-Agent System: Recent Advances and New Frontiers in Application"
  authors: "Shuaihang Chen, Yuanxing Liu, Wei Han, Weinan Zhang, Ting Liu"
  year: "2024"
  link: "https://arxiv.org/abs/2412.17481"
  section: "Abstract + Introduction"
---

# LLM 多智能体综述中文精读（第 2 篇-1）：摘要、导论与应用框架

> 本篇覆盖原论文的 **Abstract + Section 1 Introduction**，重点不是重新定义 LLM 多智能体，而是说明：在已有综述之后，为什么还需要一篇更强调**应用新前沿**的新版全景图。

## 本章导读

> [!note] 译者注（非原文）
> 这一篇最值得抓住的，是作者重新组织领域的方式。
>
> 在第 1 篇里，你已经看到了“系统怎么拆”；而这篇一上来就在问：
> - 这些系统到底被拿去做什么；
> - 为什么旧综述已经不够覆盖新工作；
> - 如果按“用途”而不是按“技术部件”来整理，领域地图会变成什么样。
>
> 所以这篇导论是一次“地图重绘”。

## 正文翻译

> [!warning] 易混点（非原文）
> 从这里开始进入**原文内容的中文翻译**。所有额外解释都会用单独 callout 标为非原文。

### 原文标题：A Survey on LLM-based Multi-Agent System: Recent Advances and New Frontiers in Application

**基于 LLM 的多智能体系统综述：最新进展与应用新前沿**

作者：Shuaihang Chen，Yuanxing Liu，Wei Han，Weinan Zhang，Ting Liu

### 原文标题：Abstract

基于大型语言模型的多智能体系统（LLM-based Multi-Agent Systems, LLM-MAS）自大型语言模型（LLMs）兴起以来，已经成为研究热点。然而，随着新的相关工作不断涌入，现有综述越来越难以对这些研究进行全面覆盖。本文因此给出一篇针对该方向的综合性综述。

作者首先讨论 LLM-MAS 的定义，并提出一个能够覆盖大量既有工作的框架。随后，作者从三类应用角度概览 LLM-MAS：

1. **解决复杂任务（solving complex tasks）**
2. **模拟特定场景（simulating specific scenarios）**
3. **评估生成式智能体（evaluating generative agents）**

在已有研究基础上，作者还进一步指出这一领域面临的若干挑战，并提出未来研究方向。

> [!tip] 理解提示（非原文）
> 这篇摘要最重要的信息，就是它把应用直接拆成三大类。后面整篇论文的主体，也基本是围绕这三个用途展开。

### 原文标题：1 Introduction

多智能体系统（Multi-Agent Systems, MAS）由于其适应性，以及处理复杂、分布式挑战的能力，近年来得到了显著发展。与单智能体设定相比，MAS 更能准确表示现实世界，因为许多真实世界应用天然就包含多个同时互动的决策者。

然而，受限于传统强化学习（RL）智能体的参数规模，以及其缺乏通用知识与通用能力，传统智能体很难处理复杂决策任务，例如在开发任务中与其他智能体协同合作。

近年来，大型语言模型（LLMs），如 Llama 3 和 GPT-4，在大规模网络语料训练基础上取得了显著成功。与强化学习智能体相比，以 LLM 为核心控制器的**生成式智能体（generative agents）**，即使不经过额外训练，也往往在推理、长轨迹决策等方面表现更好。此外，生成式智能体能够通过自然语言接口与人类交互，这使得这种交互更加灵活，也更容易解释。

正是在这些优势基础上，**基于 LLM 的多智能体系统（LLM-MAS）** 出现了。研究者已经对这些新兴工作进行过综述，并提出过一般性框架。然而，随着相关研究数量持续增长，一些新的工作已经超出了原有框架的覆盖范围。因此，本文在前人综述基础上，试图从更新的视角重新审视 LLM-MAS，重点关注最新进展，并讨论未来可能的重要研究方向。

作者收集了 **125 篇** 相关论文，这些论文主要来自 2023 和 2024 年的顶级人工智能会议，例如：
- *ACL
- NeurIPS
- AAAI
- ICLR

同时也纳入了一些虽然尚未正式发表、但很有价值的 arXiv 论文。作者还说明，这批论文列表被维护在一个公开 GitHub 仓库中：
<https://github.com/bianhua-12/Multi-generative_Agent_System_survey>

基于 LLM-MAS 的**用途（purpose）**，作者将其应用总结为三类：

1. **任务求解（task-solving）**
2. **面向特定问题的模拟（simulation for specific problems）**
3. **对生成式智能体的评估（evaluation of generative agents）**

图 1 展示了作者为 LLM-MAS 应用提出的整体框架：

- **(i) 解决复杂任务**：多个智能体会自然地把任务拆成若干子任务，因此有望提升任务性能。
- **(ii) 模拟特定场景**：研究者把 LLM-MAS 当作某个特定领域问题的沙盒（sandbox），用于模拟该领域中的主体互动。
- **(iii) 评估生成式智能体**：与传统任务评估相比，LLM-MAS 具有动态评估能力，因此更灵活，也更不容易受到数据泄漏影响。

对于这三类应用中的每一类，作者都会分别讨论：
- 代表性 LLM-MAS
- 相关资源（resources）
- 它们的评估方式（evaluation）

相较于此前的综述工作，本文声称有三点鲜明贡献：

1. **聚焦应用的分类体系（taxonomy）**：作者基于 LLM-MAS 的应用目的，提出了一个更新的分类体系。
2. **更多资源整理**：作者分析了开源框架，以及带有 benchmark 或 dataset 的研究工作，以帮助研究社区更快进入该方向。
3. **挑战与未来**：作者系统讨论了 LLM-MAS 当前面临的挑战，并尝试照亮未来研究方向。

### 图 1：应用框架总览（原图语义保留）

**图 1 原图题中文翻译：**
LLM-MAS 的应用框架，以及 LLM-MAS、生成式智能体与 LLM 之间关系的概览。右角虚线矩形表示与已有综述对齐的内容，圆角矩形表示本文新引入的原创贡献。

> [!note] 译者注（非原文）
> 图 1 的关键意义，不是把框画出来，而是把研究领域按“用途”重排：
>
> - 不是先问 agent 有哪些部件；
> - 而是先问这些系统被拿去完成什么类型的工作。
>
> 这会自然地把后续讨论推进到“资源、benchmark、系统级评测”层面，而不只停留在组件分析。

## 本章小结

> [!note] 译者注（非原文）
> 这一章最重要的五个判断：
> 1. 作者认为旧综述已经不足以覆盖 LLM-MAS 近一轮快速增长的新工作。
> 2. 本文最重要的重组动作，是把 LLM-MAS 按用途分成三大类：复杂任务求解、特定场景模拟、生成式智能体评估。
> 3. 这篇论文比第 1 篇更强调**应用分类、资源整理与评估问题**。
> 4. 作者明确收集了 125 篇 2023-2024 的代表工作，说明这篇确实试图当一张“更新版地图”。
> 5. 从一开始，作者就把 benchmark、dataset 和 challenges 视作核心组成，而不是附属材料。

## 术语对照

| 英文术语 | 中文译法 | 本章语境说明 |
|---|---|---|
| LLM-MAS | 基于 LLM 的多智能体系统 | 本文讨论对象 |
| generative agents | 生成式智能体 | 以 LLM 为核心控制器的 agent |
| task-solving | 任务求解 / 复杂任务求解 | 以完成任务为目标 |
| simulation for specific problems | 面向特定问题的模拟 | 在特定领域里构造可交互模拟 |
| evaluation of generative agents | 评估生成式智能体 | 把 LLM-MAS 当成评测环境 |
| taxonomy | 分类体系 / 分类学 | 作者重新组织领域的方式 |
| resources | 资源 | 包括框架、代码、数据集、benchmark |

## 思考题

### 题目 1：为什么这篇论文不满足于复用旧综述的框架，而要重新按“用途”组织领域？

> [!tip] 理解提示（非原文）
> 想一想：当论文数量快速增长时，只按系统组件来讲，会不会越来越难看出“它们到底被用来干什么”。

> [!note] 译者注（非原文）
> 因为在应用爆发阶段，研究者更关心的是“这些系统正在进入哪些真实任务与模拟场景”，而不是只关心部件有哪些。按用途分类，更有利于形成研究地图，也更方便讨论 benchmark、资源与评测差异。

### 题目 2：为什么“评估生成式智能体”会被作者单独当成一大类应用？

> [!warning] 易混点（非原文）
> 不要把它理解成普通 benchmark 打分。这里的重点是：LLM-MAS 本身可以成为一种动态评估环境。

> [!note] 译者注（非原文）
> 因为一旦把多个 agent 放进可交互系统里，评估就不再只是“回答对不对”，而可以变成“在互动、博弈、协作和长期行为中表现如何”。这类动态评估比静态题库更灵活，也更接近真实使用场景。

## 相关页面

- [[raw/lessons/Multi-Agent-Survey/02-recent-advances/10-recent-advances-and-new-frontiers-overview]]
- [[raw/lessons/Multi-Agent-Survey/02-recent-advances/12-recent-advances-and-new-frontiers-core-components]]
- [[raw/paper/multi-agent/02-a-survey-on-llm-based-multi-agent-system-recent-advances-and-new-frontiers-in-application-2024.pdf]]

## 下一步学习

- 上一篇：[[raw/lessons/Multi-Agent-Survey/02-recent-advances/10-recent-advances-and-new-frontiers-overview|10-overview]]
- 下一篇：[[raw/lessons/Multi-Agent-Survey/02-recent-advances/12-recent-advances-and-new-frontiers-core-components|12-core-components]]
