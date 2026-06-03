---
type: lesson
tags: [多智能体, multi-agent, survey, 论文翻译, 学习笔记]
created: 2026-05-07
updated: 2026-05-07
topic: LLM 多智能体综述中文精读（第 2 篇-2）核心组件：生成式智能体与环境
difficulty: intermediate
prerequisites:
  - [[raw/lessons/Multi-Agent-Survey/02-recent-advances/11-recent-advances-and-new-frontiers-introduction]]
sources:
  - [[raw/paper/multi-agent/02-a-survey-on-llm-based-multi-agent-system-recent-advances-and-new-frontiers-in-application-2024.pdf]]
  - https://arxiv.org/abs/2412.17481
status: completed
series:
  name: "Multi-Agent Survey｜Recent Advances and New Frontiers in Application（2024）"
  part: 12
paper_meta:
  paper_id: "multi-agent-survey-02"
  title: "A Survey on LLM-based Multi-Agent System: Recent Advances and New Frontiers in Application"
  authors: "Shuaihang Chen, Yuanxing Liu, Wei Han, Weinan Zhang, Ting Liu"
  year: "2024"
  link: "https://arxiv.org/abs/2412.17481"
  section: "Section 2 Core Components of LLM-MAS"
---

# LLM 多智能体综述中文精读（第 2 篇-2）：核心组件

> 本篇覆盖原论文 **Section 2 Core Components of LLM-MAS**。作者把 LLM-MAS 的底座收拢为两个核心要素：**生成式智能体（generative agents）** 与 **环境（environment）**。

## 本章导读

> [!note] 译者注（非原文）
> 和第 1 篇用“环境接口、画像、通信、能力获取”四维拆法相比，这篇更像是在做一次压缩：
> - agent 这边，作者聚焦生成式智能体本身；
> - system 这边，作者聚焦环境如何定义交互。
>
> 这种写法更适合为后面按应用分类做铺垫。

## 正文翻译

> [!warning] 易混点（非原文）
> 从这里开始是原文第 2 节的中文翻译；所有补充说明都单独标注为非原文。

### 原文标题：2 Core Components of LLM-MAS

LLM-MAS 指的是：一个系统中包含一组生成式智能体，它们能够在共享环境设定中进行互动与协作。接下来，作者将分别讨论生成式智能体与环境。

### 原文标题：2.1 Generative Agents

生成式智能体（generative agents）是 LLM-MAS 中的组成部分。它们具有角色定义，能够感知环境、做出决策，并执行复杂动作以改变环境。

它们可以是游戏中的一个玩家，也可以是社交媒体上的一个用户；它们扮演着推动 LLM-MAS 发展、并影响系统结果的核心角色。

与传统智能体相比，生成式智能体需要能够执行更复杂的行为。例如，它们需要基于历史信息生成完整、个性化的博客文章。因此，除了以 LLM 作为核心能力底座之外，生成式智能体通常还需要具备以下特征：

#### (i) Profiling（画像设定）

画像设定用于通过自然语言描述角色，从而把其行为绑定到特定身份上；或者根据不同任务，为每个生成式智能体定制不同提示词。

#### (ii) Memory（记忆）

记忆用于存储历史轨迹，并在后续行动中检索相关记忆。它既帮助智能体执行长期行动，也缓解了 LLM 上下文窗口有限的问题。通常会包含三层记忆：

- 长期记忆（long-term memory）
- 短期记忆（short-term memory）
- 感知记忆（sensory memory）

#### (iii) Planning（规划）

规划指的是：为未来较长一段时间制定总体行为安排。

#### (iv) Action（行动）

行动负责执行生成式智能体与环境之间的交互。生成式智能体有时需要从多个候选行为中选择一个去执行，例如在狼人杀里决定投谁；有时则需要生成不受强制约束的自由行为，例如生成一段自然语言文本。

生成式智能体还可以彼此通信，以在系统中实现合作。

作者把生成式智能体的通信目的大致分为两类：

##### 1. 实现协作（collaboration）

智能体把自己获得的信息分享给其他智能体，在一定程度上把多个个体聚合成一个完整系统，从而实现超越单个智能体独立运行时的性能。

##### 2. 实现共识（consensus）

通信也可以用来让某些智能体在行为或策略上更加相似，从而更快收敛到纳什均衡（Nash equilibrium）之类的稳定状态。

作者还把通信内容大致分为两类：

##### 1. 自然语言（natural language）

自然语言形式的通信具有很高的可解释性与灵活性，但难以优化，因此更适合追求共识的场景，例如 ChatDev 或 job fair 这类系统。

##### 2. 自定义内容（custom content）

自定义内容可能是一个向量，也可能是一个系统内部离散信号，除了系统中的生成式智能体外，没有人能直接理解这种表示。它虽然可解释性差，但更容易使用策略梯度等方法进行优化，因此常用于追求协作效率的场景。

> [!tip] 理解提示（非原文）
> 这里有一个很重要的区分：
> - **自然语言通信**更强在解释与对齐；
> - **自定义通信**更强在可优化性。
>
> 这其实已经埋下了后面“效率 vs 可解释性”的核心张力。

### 原文标题：2.2 Environment

环境设定（environmental settings）包括三部分：**规则（rules）**、**工具（tools）** 与 **干预接口（intervention interfaces）**。

#### (i) Tools（工具）

工具负责把智能体的行动指令翻译成具体结果。生成式智能体向环境发送行动指令，而环境则把这些指令转化为“动作已经发生”的记录。

不同场景下的行动空间不同。例如：
- 在社交媒体场景中，行动空间包括“点赞”“评论”“关注”等；
- 在软件开发场景中，行动空间会覆盖更大的对话链与操作链，因此通常比社交网络更复杂。

#### (ii) Rules（规则）

规则定义了生成式智能体之间的通信方式，或者它们与环境之间的交互方式，因而直接规定了整个系统的行为结构。

不同场景中可能还有一些特殊规则，例如：
- 游戏规则
- 社会行为规范

通常，在一个大规模系统中，某些处于边缘位置的生成式智能体，其行动空间较小，因此更容易被规则型模型替代。

#### (iii) Intervention（干预）

干预为外部系统提供了介入接口。这种干预可以来自：
- 人类
- 监督模型
- 甚至另一个生成式智能体

干预的目的可能是：
- 主动读取系统中的信息；
- 或被动中断系统，以防止不可控行为发生。

> [!note] 译者注（非原文）
> 这一节把环境写得很“系统工程化”。环境不再只是舞台背景，而是一个真正决定：
> - agent 能做什么；
> - 多 agent 怎么说话；
> - 外部什么时候可以插手；
> - 系统如何避免失控。

## 本章小结

> [!note] 译者注（非原文）
> 这节最值得记住的六点：
> 1. 这篇论文把 LLM-MAS 压缩成两个核心部件：生成式智能体与环境。
> 2. 生成式智能体除了 LLM 本体，还需要画像、记忆、规划和行动模块。
> 3. 智能体之间的通信既可以追求协作，也可以追求共识。
> 4. 通信内容存在“自然语言 vs 自定义信号”两条路线，分别偏向可解释性与可优化性。
> 5. 环境由工具、规则与干预接口共同构成，决定了系统的行为边界。
> 6. 从这里开始，你已经能看出后面应用分类实际上是在不同环境里重组这些基础部件。

## 术语对照

| 英文术语 | 中文译法 | 本章语境说明 |
|---|---|---|
| generative agents | 生成式智能体 | 以 LLM 为核心控制器的 agent |
| profiling | 画像设定 | 用自然语言定义角色、任务和行为倾向 |
| memory | 记忆 | 存储并检索历史轨迹 |
| planning | 规划 | 为更长时间尺度制定行为策略 |
| action | 行动 | 智能体对环境发出的具体操作 |
| collaboration | 协作 | 通过通信整合多个 agent 的信息与能力 |
| consensus | 共识 | 让多个 agent 在策略或行为上趋同 |
| intervention | 干预 | 外部主体对系统进行读取或中断 |
| action space | 行动空间 | 某一环境中 agent 可执行动作的集合 |

## 思考题

### 题目 1：为什么环境在这篇论文里不是背景，而是核心组件？

> [!tip] 理解提示（非原文）
> 想一想：如果没有环境来定义动作、规则和干预，agent 的“能力”还能否真正落地。

> [!note] 译者注（非原文）
> 因为多智能体系统不是只靠 prompt 在空中推理。环境规定了行动空间、通信方式、约束规则和外部干预条件。离开环境，agent 的能力就无法转化为真实可观察的系统行为。

### 题目 2：自然语言通信和自定义通信分别适合什么场景？

> [!warning] 易混点（非原文）
> 不要只看“人能不能看懂”，还要看“系统能不能高效优化”。

> [!note] 译者注（非原文）
> 自然语言更适合需要解释、对齐和达成共识的系统；自定义信号更适合追求效率与算法优化的协作系统。二者的差异，本质上是“可解释性”与“可优化性”的取舍。

## 相关页面

- [[raw/lessons/Multi-Agent-Survey/02-recent-advances/11-recent-advances-and-new-frontiers-introduction]]
- [[raw/lessons/Multi-Agent-Survey/02-recent-advances/13-recent-advances-and-new-frontiers-solving-complex-tasks]]
- [[raw/paper/multi-agent/02-a-survey-on-llm-based-multi-agent-system-recent-advances-and-new-frontiers-in-application-2024.pdf]]

## 下一步学习

- 上一篇：[[raw/lessons/Multi-Agent-Survey/02-recent-advances/11-recent-advances-and-new-frontiers-introduction|11-introduction]]
- 下一篇：[[raw/lessons/Multi-Agent-Survey/02-recent-advances/13-recent-advances-and-new-frontiers-solving-complex-tasks|13-solving-complex-tasks]]
