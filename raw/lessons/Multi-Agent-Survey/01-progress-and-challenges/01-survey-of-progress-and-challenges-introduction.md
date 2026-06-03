---
type: lesson
tags: [多智能体, multi-agent, survey, 论文翻译, 学习笔记]
created: 2026-05-07
updated: 2026-05-07
topic: LLM 多智能体综述中文精读（一）导论：问题背景、研究动机与论文结构
difficulty: intermediate
prerequisites:
  - [[raw/paper/multi-agent/00-reading-manual]]
  - [[raw/lessons/Multi-Agent-Survey/01-progress-and-challenges/00-overview]]
sources:
  - [[raw/paper/multi-agent/01-large-language-model-based-multi-agents-survey-of-progress-and-challenges-2024.pdf]]
  - https://arxiv.org/abs/2402.01680
status: completed
series:
  name: "Multi-Agent Survey｜Progress and Challenges（2024）"
  part: 1
paper_meta:
  paper_id: "multi-agent-survey-01"
  title: "Large Language Model based Multi-Agents: A Survey of Progress and Challenges"
  authors: "Taicheng Guo, Xiuying Chen, Yaqi Wang, Ruidi Chang, Shichao Pei, Nitesh V. Chawla, Olaf Wiest, Xiangliang Zhang"
  year: "2024"
  link: "https://arxiv.org/abs/2402.01680"
  section: "Abstract + Introduction"
---

# LLM 多智能体综述中文精读（一）：导论

> 本篇覆盖原论文的 **Abstract + Section 1 Introduction**，目标是先建立这篇综述的基本问题意识：为什么 LLM 多智能体（LLM-based Multi-Agent, LLM-MA）值得单独作为一个研究方向来系统梳理。

## 本章导读

> [!note] 译者注（非原文）
> 这一篇最重要的任务，不是让你马上记住所有系统名称，而是先回答三个问题：
> 1. 为什么“单个 LLM 智能体”还不够？
> 2. 为什么研究者开始转向“多个智能体协作”？
> 3. 这篇综述准备从哪些维度来搭建整个领域的认知骨架？
>
> 如果你后面想做多智能体方向研究，这一篇就是“总开场”。

## 正文翻译

> [!warning] 易混点（非原文）
> 从这里开始进入**原文内容的中文翻译**。若中间出现解释性补充，会使用单独 callout 标注，避免与原文混淆。

### 原文标题：Large Language Model based Multi-Agents: A Survey of Progress and Challenges

**基于大型语言模型的多智能体：进展与挑战综述**

作者：Taicheng Guo，Xiuying Chen，Yaqi Wang，Ruidi Chang，Shichao Pei，Nitesh V. Chawla，Olaf Wiest，Xiangliang Zhang

### 原文标题：Abstract

大型语言模型（Large Language Models, LLMs）已经在广泛的任务上取得了显著成功。由于 LLM 在规划与推理方面展现出令人印象深刻的能力，它们已经被用作自治智能体（autonomous agents），以自动完成许多任务。最近，在“将单个 LLM 用作单一规划或决策智能体”这一发展基础之上，基于 LLM 的多智能体系统（LLM-based multi-agent systems）已经在复杂问题求解与世界模拟（world simulation）方面取得了可观进展。

为了向社区提供这一动态领域的整体概览，我们撰写了这篇综述，对基于 LLM 的多智能体系统的关键方面以及其挑战进行深入讨论。我们的目标是让读者在以下问题上获得扎实的认识：

- 基于 LLM 的多智能体会模拟哪些领域与环境？
- 这些智能体是如何被设定画像（profile）的，它们又如何彼此通信？
- 哪些机制有助于智能体能力的增长？
- 对于希望进入这一研究方向的读者，哪些常用数据集或基准（benchmarks）值得优先了解？

为了使研究者能够持续跟进最新工作，我们还维护了一个开源 GitHub 仓库，专门用于整理基于 LLM 的多智能体系统研究。

> [!tip] 理解提示（非原文）
> 摘要里其实已经把整篇论文的主线说完了：
> - **场景**：它们被放到哪些环境里；
> - **角色**：这些 agent 是怎么被设定的；
> - **交互**：它们怎么交流；
> - **成长**：它们怎么变强；
> - **评测**：研究者用什么 benchmark 在衡量它们。
>
> 这五个问题，基本就是后面 2-6 节的组织骨架。

### 原文标题：1 Introduction

大型语言模型（LLMs）最近显示出了惊人的潜力：它们在推理和规划能力上，似乎开始接近人类水平。这种能力恰好契合了人们对自治智能体的期待——也就是那种能够感知周围环境、做出决策，并据此采取行动的系统。

因此，基于 LLM 的智能体（LLM-based agent）迅速成为研究热点并得到快速发展。研究者希望利用它们来理解与生成类人指令，并在广泛语境中支持更复杂的交互与决策。已有一些及时出现的综述工作，系统总结了基于 LLM 的单智能体（single-agent）研究进展。

在单个基于 LLM 的智能体已经展示出鼓舞人心能力的基础上，**基于 LLM 的多智能体（LLM-MA）** 被进一步提出，用于利用多个智能体的**集体智能（collective intelligence）**，以及它们各自专门化的画像、技能与角色。

与仅依赖单个 LLM 驱动智能体的系统相比，多智能体系统提供了更高级的能力，原因主要有两点：

1. **将 LLM 专门化为多个不同的智能体**，使每个智能体拥有不同能力；
2. **让这些不同智能体发生互动**，从而更有效地模拟复杂的真实世界环境。

在这种设定下，多个自治智能体会协同进行规划、讨论与决策，这种过程类似于人类群体在解决问题时所表现出的合作方式。这个方向一方面利用了 LLM 强大的文本生成能力，把“文本”本身用作智能体之间的通信媒介；另一方面也利用了 LLM 覆盖广泛领域知识的优势，以及它们在特定任务上进一步专门化的潜力。

最近的研究已经表明：基于 LLM 的多智能体可以被用在多种任务之中，并取得了很有前景的结果。例如：

- 软件开发（software development）
- 多机器人系统（multi-robot systems）
- 社会模拟（society simulation）
- 政策模拟（policy simulation）
- 游戏模拟（game simulation）

由于这一领域具有跨学科特征，它吸引的研究者并不只来自传统 AI 社群，也包括社会科学、心理学和政策研究等背景的研究者。相关论文数量正在快速增长，如图 1 所示，因此，LLM 多智能体研究的影响力也在持续扩大。

尽管如此，早期相关工作大多是彼此独立开展的，缺少一篇系统性的综述，来统一总结这些工作、搭建这个方向的完整蓝图（blueprint），并系统审视未来的研究挑战。这正是本文工作的意义所在，也是作者撰写本综述的直接动机。

作者期望，这篇综述不仅能为 LLM 研究与开发本身提供价值，也能支持更广泛的跨学科研究中对 LLM 的使用。读者将获得对 LLM 多智能体系统的系统性理解，把握建立基于 LLM 的多智能体系统时涉及的基本概念，并掌握这一快速变化领域中的最新研究趋势与应用。

作者同时承认：这一方向目前仍处于早期阶段，而且还在快速演化，不断出现新的方法论和应用。为了补充论文本身、提供一个可持续更新的资源，作者维护了一个开源 GitHub 仓库。作者希望这篇综述能够激发这一方向的进一步探索与创新，并推动其在更广研究领域中的应用。

为了帮助不同背景的读者理解 LLM-MA 技术，并补足已有综述尚未回答的问题，作者将全文组织如下：

- 在第 2 节，先介绍背景知识；
- 接着回答一个核心问题：**LLM 多智能体系统是如何与“协作式任务求解环境”对齐的？**
- 为了回答这一问题，作者在第 3 节提出一个整体框架，用于定位、区分并连接 LLM-MA 系统的不同方面；
- 在第 3 节中，作者重点讨论四个维度：
  1. **智能体—环境接口（agents-environment interface）**：智能体如何与任务环境发生交互；
  2. **智能体画像设定（agent profiling）**：一个智能体如何被 LLM 刻画成以特定方式行动的角色；
  3. **智能体通信（agent communication）**：智能体如何交换消息并协同合作；
  4. **智能体能力获取（agent capability acquisition）**：智能体如何发展能力，以更有效地解决问题。
- 从另一个角度看，研究 LLM-MA 也可以从“应用”切入，因此作者在第 4 节将现有应用划分为两条主线：
  - 面向问题求解（problem-solving）的多智能体
  - 面向世界模拟（world simulation）的多智能体
- 在第 5 节，为了帮助读者找到合适工具与资源，作者介绍了用于研究 LLM-MA 的开源实现框架，以及常用的数据集与基准；
- 在第 6 节，作者基于前述总结，进一步讨论未来的研究挑战与机会；
- 最后在第 7 节给出全文结论。

### 图 1：研究增长趋势（原图语义保留）

**图 1 原图题中文翻译：**
基于 LLM 的多智能体研究领域正在快速上升。针对“问题求解”和“世界模拟”两大方向，作者将当前工作继续细分为若干类别，并按每 3 个月的时间窗口统计不同类别论文数量。图中每个叶节点上的数字表示该类别论文的数量。

> [!note] 译者注（非原文）
> 这张图的真正作用不是给你一个精确文献计数，而是传递一个研究判断：
> **这个领域不是零散出现的几个 demo，而是在迅速形成一个结构化增长的研究版图。**
>
> 换句话说，这篇综述想做的，不只是“列论文”，而是给这个正在爆炸增长的方向建立一张地图。

## 本章小结

> [!note] 译者注（非原文）
> 这一章读完，至少要抓住下面几点：
> 1. 这篇论文把 LLM 多智能体看成一个**独立且快速增长**的研究方向。
> 2. 作者认为单智能体已经证明了 LLM 的规划/推理潜力，而多智能体进一步释放了**角色分工 + 交互协作**的价值。
> 3. 本文的核心骨架不是“按应用随便罗列”，而是围绕四个系统维度展开：环境接口、画像设定、通信、能力获取。
> 4. 应用部分会被作者重新归纳为两大主线：**问题求解**与**世界模拟**。
> 5. 如果你后面要做研究，真正值得反复回看的不是例子本身，而是这篇综述给出的**分析框架**。

## 术语对照

| 英文术语 | 中文译法 | 本章语境说明 |
|---|---|---|
| Large Language Model (LLM) | 大型语言模型 | 本文一切能力讨论都以 LLM 为底座 |
| autonomous agent | 自治智能体 | 指能感知、决策、行动的主体 |
| single-agent | 单智能体 | 由一个 LLM 或一个主要 agent 主导 |
| multi-agent | 多智能体 | 多个 agent 协同参与规划、讨论、决策 |
| collective intelligence | 集体智能 | 多个 agent 协作带来的整体能力提升 |
| world simulation | 世界模拟 | 用多个 agent 模拟社会、游戏、经济等复杂系统 |
| blueprint | 蓝图 / 总体框架 | 作者认为当前领域缺失的系统总结 |

## 思考题

### 题目 1：为什么作者认为“多智能体”不是“把单智能体复制很多份”这么简单？

> [!tip] 理解提示（非原文）
> 想一想作者在导论里强调的两个增益来源：专门化角色，以及角色之间的互动。

> [!note] 译者注（非原文）
> 关键不在“数量变多”，而在“**差异化分工 + 协作机制**”开始出现。多个相同 agent 并排放着，不一定就构成真正的 multi-agent system；只有当它们承担不同能力、发生结构化交流、共同作用于环境时，才更接近作者定义下的 LLM-MA。

### 题目 2：为什么这篇论文把“问题求解”和“世界模拟”并列为两大应用主线？

> [!warning] 易混点（非原文）
> 很多人一看到多智能体，就只想到“协同干活”。但作者明确认为，另一大主线是“模拟世界中的多主体互动”。

> [!note] 译者注（非原文）
> 因为 LLM 多智能体有两种很不同的用途：
> - 一种是**把它们当成协作问题求解器**，目标是把任务做成；
> - 另一种是**把它们当成可交互的社会/行为模型**，目标是观察复杂系统怎样演化。
>
> 后者不一定追求“任务完成率”，而更关注群体行为、传播、博弈和社会动态。

## 相关页面

- [[raw/paper/multi-agent/01-large-language-model-based-multi-agents-survey-of-progress-and-challenges-2024.pdf]]
- [[raw/lessons/Multi-Agent-Survey/01-progress-and-challenges/00-overview]]
- [[raw/paper/multi-agent/00-reading-manual]]

## 下一步学习

- 上一篇：[[raw/lessons/Multi-Agent-Survey/01-progress-and-challenges/00-overview|00-overview]]
- 下一篇：[[raw/lessons/Multi-Agent-Survey/01-progress-and-challenges/02-survey-of-progress-and-challenges-background|02-background]]
