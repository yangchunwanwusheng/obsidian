---
type: lesson
tags: [多智能体, multi-agent, survey, 论文翻译, 学习笔记]
created: 2026-05-07
updated: 2026-05-07
topic: LLM 多智能体综述中文精读系列总览（第 1 篇：Progress and Challenges）
difficulty: intermediate
prerequisites:
  - [[raw/paper/multi-agent/00-reading-manual]]
sources:
  - [[raw/paper/multi-agent/00-reading-manual]]
  - [[raw/paper/multi-agent/01-large-language-model-based-multi-agents-survey-of-progress-and-challenges-2024.pdf]]
  - https://arxiv.org/abs/2402.01680
status: in-progress
series:
  name: "Multi-Agent Survey｜Progress and Challenges（2024）"
  part: 0
paper_meta:
  paper_id: "multi-agent-survey-01"
  title: "Large Language Model based Multi-Agents: A Survey of Progress and Challenges"
  authors: "Taicheng Guo, Xiuying Chen, Yaqi Wang, Ruidi Chang, Shichao Pei, Nitesh V. Chawla, Olaf Wiest, Xiangliang Zhang"
  year: "2024"
  link: "https://arxiv.org/abs/2402.01680"
  section: "overview"
---

# Multi-Agent Survey 中文精读系列总览（第 1 篇）

> 这是一套面向学习者的“论文友好版全文中文翻译笔记”。当前先完成推荐阅读顺序中的第 1 篇：**Progress and Challenges（2024）**，并把整套写法固化成后续可复用流程。

## 本系列目标

1. 把 [[raw/paper/multi-agent/00-reading-manual|多智能体综述论文阅读手册]] 中推荐顺序的论文，逐步整理成**可连续阅读的中文学习笔记**。
2. 对每篇论文坚持两个原则：
   - **全文保留**：不擅自删减原文结构与信息。
   - **理解友好**：允许补充解释，但必须明确标注为**非原文**。
3. 让后续第 2-6 篇能够沿用同一模板、同一术语口径、同一导航规范继续推进。

## 这套笔记与阅读手册的关系

- [[raw/paper/multi-agent/00-reading-manual]] 负责回答：**先读哪篇、为什么这么排、每篇应该抓什么问题。**
- 当前这套“中文精读系列”负责回答：**这篇论文逐段到底在说什么，我应该怎样把它真正读懂。**

你可以把二者理解为：
- 阅读手册 = 路线图
- 中文精读系列 = 逐章展开的课程化正文

## 当前覆盖范围

当前已完成的是推荐顺序中的**第 1 篇**：

- 原文：[[raw/paper/multi-agent/01-large-language-model-based-multi-agents-survey-of-progress-and-challenges-2024.pdf]]
- arXiv：<https://arxiv.org/abs/2402.01680>
- 角色定位：奠基综述 / 主线第一篇 / 先搭骨架

## 非原文标注规则

为了避免“翻译”和“讲解”混在一起，所有额外补充都必须明确挂标签：

> [!note] 译者注（非原文）
> 用于补充背景、解释上下文、说明作者真正想表达什么。

> [!tip] 理解提示（非原文）
> 用于帮助抓重点、建立阅读顺序、降低理解门槛。

> [!warning] 易混点（非原文）
> 用于指出容易误解的概念边界、术语差别或阅读陷阱。

**硬规则**：
- 不在普通正文段落里偷偷夹带解释。
- 原文翻译与额外讲解必须视觉分层。
- 图、表、公式无法直接原样抽取时，也必须先保留原文语义，再单独添加解释。

## 统一术语约定（当前版）

| 英文术语 | 当前统一译法 | 备注 |
|---|---|---|
| Large Language Model (LLM) | 大型语言模型 | 首次出现保留英文缩写 |
| LLM-based agent | 基于 LLM 的智能体 | 必要时简称“LLM 智能体” |
| Multi-Agent System (MAS) | 多智能体系统 | 若特指 LLM 场景，则写 LLM 多智能体系统 |
| LLM-based Multi-Agent / LLM-MA | 基于 LLM 的多智能体 / LLM 多智能体 | 本系列统一优先写“LLM 多智能体” |
| environment | 环境 | 具体语境下可写“任务环境/操作环境” |
| profiling | 画像设定 / 角色设定 | 本文多处强调 agent profile，故保留“画像设定”色彩 |
| communication | 通信 | 若强调语言交互，也可写“交流/消息交换” |
| cooperative | 协作式 | 与 debate / competitive 区分 |
| debate | 辩论式 | 用于多代理争辩求解 |
| competitive | 竞争式 | 用于目标冲突场景 |
| capability acquisition | 能力获取 | 指智能体如何通过反馈、记忆、自演化提升能力 |
| benchmark | 基准 / 基准测试 | 视句子语义二选一 |
| world simulation | 世界模拟 | 也可理解为“社会/环境模拟”总类 |
| orchestration | 编排 / 协同编排 | 当前先统一写“编排” |

> [!note] 译者注（非原文）
> 后续第 2-6 篇如果出现更合适的固定译法，会优先保持术语一致，而不是每篇自由翻译。

## 本篇论文的分篇导航

| part | 文件 | 覆盖内容 |
|---|---|---|
| 0 | [[raw/lessons/Multi-Agent-Survey/01-progress-and-challenges/00-overview|00-overview]] | 系列目标、术语口径、导航规则、当前论文入口 |
| 1 | [[raw/lessons/Multi-Agent-Survey/01-progress-and-challenges/01-survey-of-progress-and-challenges-introduction|01-introduction]] | 摘要、导论、研究动机、图 1 |
| 2 | [[raw/lessons/Multi-Agent-Survey/01-progress-and-challenges/02-survey-of-progress-and-challenges-background|02-background]] | 单智能体背景、单体 vs 多体 |
| 3 | [[raw/lessons/Multi-Agent-Survey/01-progress-and-challenges/03-survey-of-progress-and-challenges-dissecting-llm-ma|03-dissecting-llm-ma]] | 环境接口、画像设定、通信、能力获取、图 2-3、表 1 |
| 4 | [[raw/lessons/Multi-Agent-Survey/01-progress-and-challenges/04-survey-of-progress-and-challenges-applications|04-applications]] | 问题求解与世界模拟两大应用主线 |
| 5 | [[raw/lessons/Multi-Agent-Survey/01-progress-and-challenges/05-survey-of-progress-and-challenges-supporting-tools-and-challenges|05-supporting-tools-and-challenges]] | 开源框架、数据集基准、挑战与机会、结论、表 2 |
| 6 | [[raw/lessons/Multi-Agent-Survey/01-progress-and-challenges/06-survey-of-progress-and-challenges-references|06-references]] | 参考文献全文保留 |

## 后续 5 篇预留位

> [!tip] 理解提示（非原文）
> 下面只预留路线，不提前创建空文件，避免产生断链。

1. 第 2 篇：*A Survey on LLM-based Multi-Agent System: Recent Advances and New Frontiers in Application (待后续创建)*
2. 第 3 篇：*Multi-Agent Collaboration Mechanisms: A Survey of LLMs (待后续创建)*
3. 第 4 篇：*Beyond Self-Talk: A Communication-Centric Survey of LLM-Based Multi-Agent Systems (待后续创建)*
4. 第 5 篇：*LLMs Working in Harmony: A Survey on the Technological Aspects of Building Effective LLM-Based Multi-Agent Systems (待后续创建)*
5. 第 6 篇：*Creativity in LLM-based Multi-Agent Systems: A Survey (待后续创建)*

## 推荐阅读顺序（针对当前第 1 篇）

1. 先读 [[raw/lessons/Multi-Agent-Survey/01-progress-and-challenges/01-survey-of-progress-and-challenges-introduction]]：建立“为什么需要这篇综述”的大图景。
2. 再读 [[raw/lessons/Multi-Agent-Survey/01-progress-and-challenges/02-survey-of-progress-and-challenges-background]]：分清单智能体与多智能体的基本边界。
3. 重点读 [[raw/lessons/Multi-Agent-Survey/01-progress-and-challenges/03-survey-of-progress-and-challenges-dissecting-llm-ma]]：这部分是整篇最重要的框架骨架。
4. 然后读 [[raw/lessons/Multi-Agent-Survey/01-progress-and-challenges/04-survey-of-progress-and-challenges-applications]]：把骨架映射到真实应用。
5. 再读 [[raw/lessons/Multi-Agent-Survey/01-progress-and-challenges/05-survey-of-progress-and-challenges-supporting-tools-and-challenges]]：看工具、基准和未来挑战。
6. 最后保留 [[raw/lessons/Multi-Agent-Survey/01-progress-and-challenges/06-survey-of-progress-and-challenges-references]] 作为检索索引。

## 使用方法

- 如果你是第一次接触 LLM 多智能体：按 01 → 05 顺序读。
- 如果你是为了找研究问题：重点读 03、04、05。
- 如果你是为了后面继续翻译其他综述：重点观察本系列的标题层级、非原文标注规则、术语表累积方式。

## 本系列当前复用模板

- [[_templates/paper-translation-lesson|paper-translation-lesson 模板]]

这个模板已经把以下规范固化：
- 全文翻译优先
- 原文与补充解释严格分层
- 图/表/公式必须保留语义
- 每篇都有术语对照、学习小结、思考题、上一篇/下一篇导航

## 相关页面

- [[raw/paper/multi-agent/00-reading-manual]]
- [[wiki/index]]
- [[wiki/log]]

## 下一步学习

- 开始正文：[[raw/lessons/Multi-Agent-Survey/01-progress-and-challenges/01-survey-of-progress-and-challenges-introduction|01-introduction]]
- 若想先看整体定位，再回到原始推荐顺序：[[raw/paper/multi-agent/00-reading-manual]]
