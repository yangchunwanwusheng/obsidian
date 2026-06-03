---
type: lesson
tags: [多智能体, multi-agent, survey, 论文翻译, 学习笔记]
created: 2026-05-07
updated: 2026-05-07
topic: LLM 多智能体综述中文精读（三）系统拆解：环境接口、画像设定、通信与能力获取
difficulty: intermediate
prerequisites:
  - [[raw/lessons/Multi-Agent-Survey/01-progress-and-challenges/02-survey-of-progress-and-challenges-background]]
sources:
  - [[raw/paper/multi-agent/01-large-language-model-based-multi-agents-survey-of-progress-and-challenges-2024.pdf]]
  - https://arxiv.org/abs/2402.01680
status: completed
series:
  name: "Multi-Agent Survey｜Progress and Challenges（2024）"
  part: 3
paper_meta:
  paper_id: "multi-agent-survey-01"
  title: "Large Language Model based Multi-Agents: A Survey of Progress and Challenges"
  authors: "Taicheng Guo, Xiuying Chen, Yaqi Wang, Ruidi Chang, Shichao Pei, Nitesh V. Chawla, Olaf Wiest, Xiangliang Zhang"
  year: "2024"
  link: "https://arxiv.org/abs/2402.01680"
  section: "Section 3 Dissecting LLM-MA Systems"
---

# LLM 多智能体综述中文精读（三）：系统拆解

> 本篇覆盖原论文 **Section 3**，这是整篇综述最核心的骨架部分。作者把 LLM 多智能体系统拆成四个维度：**环境接口、画像设定、通信、能力获取**。

## 本章导读

> [!note] 译者注（非原文）
> 如果你只能从这篇论文里记住一个东西，那大概率就是这一节。
>
> 因为作者真正想做的不是“罗列一些案例”，而是给出一种统一看法：
> - agent 在什么环境里工作；
> - agent 被设定成什么角色；
> - agent 彼此如何交流；
> - agent 如何根据反馈不断调整自己。
>
> 这四个维度几乎可以当成后续阅读其他 multi-agent 论文时的分析模板。

## 正文翻译

> [!warning] 易混点（非原文）
> 从这里开始进入原文第 3 节的完整中文翻译；所有补充说明都会单独标记为非原文。

### 原文标题：3 Dissecting LLM-MA Systems: Interface, Profiling, Communication, and Capabilities

在这一节中，作者深入分析 LLM 多智能体系统（LLM-MA systems）的内部结构。在这些系统里，多个自治智能体会像人类群体在问题求解场景中的互动一样，开展协作活动。

作者关注的一个关键问题是：**这些 LLM 多智能体系统是如何与它们所处的运行环境，以及它们试图实现的集体目标相对齐的？**

为了解释这一点，作者在图 2 中给出了这类系统的一般性架构。作者从四个关键方面对其运行框架进行拆解：

1. 智能体—环境接口（agents-environment interface）
2. 智能体画像设定（agent profiling）
3. 智能体通信（agent communication）
4. 智能体能力获取（agent capability acquisition）

### 图 2：LLM-MA 系统架构（原图语义保留）

**图 2 原图题中文翻译：**
LLM-MA 系统的架构。

> [!note] 译者注（非原文）
> 原图的核心意思可以口语化理解为：
>
> - 外部有一个**环境（environment）**；
> - 中间有多个被设定了不同画像的 **agent**；
> - agent 之间会通过**通信**交换信息；
> - agent 会从环境或其他主体那里得到**反馈**；
> - 这些反馈再通过**记忆、自演化、动态生成**等机制，转化成能力提升。
>
> 也就是说，作者不是把 multi-agent 看成一堆 prompt，而是看成一个“**环境—角色—通信—反馈闭环**”系统。

### 原文标题：3.1 Agents-Environment Interface

运行环境（operational environments）定义了 LLM 多智能体系统被部署与交互的具体语境或场景。比如，这些环境可以是软件开发、游戏，或金融市场、社会行为建模等其他领域。

基于 LLM 的智能体在环境中进行感知与行动，而环境又会反过来影响它们的行为与决策。例如，在狼人杀（Werewolf Game）模拟中，沙盒环境（sandbox environment）规定了游戏框架，包括昼夜切换、讨论阶段、投票机制以及奖励规则。像狼人（werewolves）和预言家（Seer）这样的智能体会执行特定动作，例如击杀或查验身份。动作发生后，智能体会从环境中获得反馈，这些反馈告诉它们当前游戏状态如何变化。这些信息又会引导智能体随着博弈进程与其他智能体互动，不断调整自己的策略。

“智能体—环境接口（Agents-Environment Interface）”指的是：智能体如何与环境交互、如何感知环境。正是通过这个接口，智能体理解周围情境、做出决策，并从行动结果中学习。

作者将当前 LLM-MA 系统中的环境接口划分为三类：**沙盒（Sandbox）**、**物理环境（Physical）** 与 **无环境（None）**。

#### Sandbox（沙盒环境）

沙盒是由人类构造的模拟或虚拟环境，智能体能够在其中更自由地交互，并尝试各种行动与策略。这类接口广泛用于：

- 软件开发（把代码解释器当作模拟环境）
- 游戏（把游戏规则当作模拟环境）
- 其他类似可控交互场景

#### Physical（物理环境）

物理环境指真实世界环境，智能体需要与真实物体交互，并遵守现实物理规律与约束。在物理空间中，智能体通常需要采取会产生直接现实后果的动作。例如：扫地、做三明治、打包杂货、整理柜子等任务，都要求机器人智能体反复执行动作、观察物理环境，并持续修正自己的行为。

#### None（无特定外部环境）

有些场景中并不存在明确的外部环境，智能体也不会与环境交互。例如，许多工作会让多个智能体围绕一个问题展开辩论，以形成共识。这类应用主要关注智能体之间的通信，而不依赖外部环境。

> [!tip] 理解提示（非原文）
> 这三类接口其实非常重要，因为它决定了你的 multi-agent 系统“到底在对什么负责”：
> - **Sandbox**：更像可控试验场，便于观察与调试；
> - **Physical**：更接近真实机器人与具身任务，代价更高、约束更多；
> - **None**：说明系统重点不在“操作世界”，而在“集体推理/交流/辩论”。

### 原文标题：3.2 Agents Profiling

在 LLM 多智能体系统中，智能体由其**特征（traits）**、**行动（actions）** 与 **技能（skills）** 来定义，并且这些定义会围绕特定目标进行定制。

在不同系统中，智能体会承担不同角色，每个角色通常会有完整描述，涵盖其性格、能力、行为方式与约束条件。例如：

- 在游戏环境中，智能体可能被设定成具有不同身份和技能的玩家，各自以不同方式为游戏目标作出贡献；
- 在软件开发中，智能体可能扮演产品经理、工程师等角色，它们各自的职责与专长会引导开发流程；
- 在辩论平台中，智能体可能被设定成支持方、反对方或裁判，每种角色都有不同职责与策略。

这些画像（profiles）对于界定智能体在各自环境中的互动方式与发挥效果至关重要。原文表 1 列出了近期 LLM-MA 工作中的画像设定情况。

关于**智能体画像设定方法（Agent Profiling Methods）**，作者将其分为三类：

#### Pre-defined（预定义）

智能体画像由系统设计者显式写定。

#### Model-Generated（模型生成）

智能体画像由模型生成，例如由大型语言模型自动生成角色设定。

#### Data-Derived（数据导出）

智能体画像依据现成数据集构建。

> [!note] 译者注（非原文）
> 这里要特别注意：作者所谓的 “profile”，不是只有“你是程序员/你是产品经理”这种职业标签。
>
> 它更广泛地包括：
> - 角色身份
> - 行为偏好
> - 能力边界
> - 任务职责
> - 约束规则
>
> 所以，画像设定本质上是在定义“这个 agent 应该以什么样的方式存在于系统里”。

### 原文标题：3.3 Agents Communication

在 LLM 多智能体系统中，智能体之间的通信（communication）是支撑**集体智能（collective intelligence）**的关键基础设施。作者从三个角度拆解智能体通信：

1. **通信范式（Communication Paradigms）**：智能体之间采用什么样的互动风格与方法；
2. **通信结构（Communication Structure）**：多智能体系统内部通信网络的组织方式与架构；
3. **通信内容（Communication Content）**：智能体之间交换的具体内容。

#### Communication Paradigms（通信范式）

当前 LLM-MA 系统中的通信主要采用三种范式：**协作式（Cooperative）**、**辩论式（Debate）** 与 **竞争式（Competitive）**。

- **Cooperative（协作式）**：多个智能体为了共享目标或共同目标而一起工作，通常会交换信息以改进集体解决方案。
- **Debate（辩论式）**：智能体通过论证性交互来推进任务。它们会提出并捍卫自己的观点或答案，同时批评其他智能体的方案。这种范式适合于形成共识，或提炼出更精细的答案。
- **Competitive（竞争式）**：每个智能体都在追求自己的目标，而这些目标可能与其他智能体的目标发生冲突。

#### Communication Structure（通信结构）

图 3 展示了 LLM-MA 系统中四种典型通信结构。

- **Layered communication（分层通信）**：通信采用层级化组织。处于不同层的智能体拥有不同角色，主要在本层或相邻层之间互动。
- **Decentralized communication（去中心化通信）**：通信采用点对点网络，智能体彼此直接交流。这种结构常用于世界模拟类应用。
- **Centralized communication（中心化通信）**：存在一个中心智能体或一组中心智能体来协调整个系统中的通信，其他智能体主要通过这个中心节点互动。
- **Shared Message Pool（共享消息池）**：MetaGPT 提出的一种结构。系统维护一个共享消息池，智能体往里面发布消息，并根据自己的画像订阅相关消息，从而提升通信效率。

作者举例说，[Liu et al., 2023] 提出 Dynamic LLM-Agent Network（DyLAN），用多层前馈网络方式组织智能体，并引入推理时的 agent 选择与提前停止（early-stopping）机制，以提高协作效率。

#### Communication Content（通信内容）

在 LLM-MA 系统中，通信内容通常表现为**文本（text）**。但具体内容高度依赖于应用场景。例如：

- 在软件开发中，智能体之间可能围绕代码片段展开交流；
- 在狼人杀这类游戏模拟中，智能体可能交流自己的分析、怀疑对象或策略。

### 图 3：智能体通信结构（原图语义保留）

**图 3 原图题中文翻译：**
智能体通信结构。

> [!note] 译者注（非原文）
> 这张图最值得记的不是图长什么样，而是它提醒你：
>
> **通信不是“有没有消息传递”这么简单，而是“消息在什么拓扑里流动”。**
>
> 同样一组 agent：
> - 如果是去中心化，优点是灵活，缺点是容易混乱；
> - 如果是中心化，优点是可控，缺点是中心节点可能成为瓶颈；
> - 如果是共享消息池，优点是降低广播成本，但也依赖较好的消息筛选机制。

### 原文标题：3.4 Agents Capabilities Acquisition

“智能体能力获取（Agents Capabilities Acquisition）”是 LLM-MA 中的关键过程，它使智能体能够动态学习并演化。

在这个问题上，有两个基本概念：

1. 智能体应该从哪些类型的反馈中学习，以增强自己的能力；
2. 智能体应采用什么样的调整策略，才能更有效地解决复杂问题。

#### Feedback（反馈）

反馈指的是：智能体获得的、关于自己行动结果的关键信息。它帮助智能体理解自身行动可能带来的影响，并适应复杂、动态的问题环境。

在大多数研究中，反馈的形式是**文本**。根据反馈的来源，作者将其分为四类：

##### 1. Feedback from Environment（来自环境的反馈）

反馈来自现实世界环境或虚拟环境。这在大多数面向问题求解的 LLM-MA 中都很常见，例如：

- 软件开发场景中，智能体从代码解释器得到反馈；
- 具身多智能体系统中，机器人从真实或模拟环境中得到反馈。

##### 2. Feedback from Agents Interactions（来自智能体交互的反馈）

反馈来自其他智能体的判断，或来自它们之间的通信。这在问题求解与世界模拟场景中都很常见。例如：

- 在科学辩论中，智能体通过交流来学习如何批判性评估并修正结论；
- 在游戏模拟中，智能体会根据之前其他智能体之间的互动来调整策略。

##### 3. Human Feedback（人类反馈）

反馈直接来自人类，这对于让多智能体系统与人类价值观和偏好对齐至关重要。此类反馈在各种“human-in-the-loop（人在回路中）”应用中被广泛使用。

##### 4. None（无反馈）

在某些情况下，系统不会向智能体提供反馈。这通常出现在以分析模拟结果为主的世界模拟研究中，而非强调智能体规划能力的系统。例如，在传播模拟中，研究重点更偏向结果分析，因此反馈并不是系统组件的一部分。

#### Agents Adjustment to Complex Problems（面向复杂问题的智能体调整）

为了增强能力，LLM-MA 中的智能体可以通过三种主要方案进行适应：

##### 1. Memory（记忆）

大多数 LLM-MA 系统都会为智能体配备记忆模块，以帮助它们调整行为。智能体会把过去交互和反馈中的信息存入记忆。在执行动作时，它们可以检索与当前任务相关、且有价值的记忆，尤其是那些在相似过去目标中对应成功行为的记忆。这个过程能够帮助它们改进当前行动。

##### 2. Self-Evolution（自演化）

与仅依赖历史记录来决定下一步动作的记忆式方案不同，智能体还可以通过**修改自己本身**来动态自我演化，例如：

- 调整初始目标
- 调整规划策略
- 根据反馈或通信日志训练自己

作者举例说明：

- [Nascimento et al., 2023] 提出一种自我控制回路（self-control loop），使多智能体系统中的每个智能体都能进行自管理与自适应，从而提升协作效率；
- [Zhang et al., 2023b] 提出 ProAgent，它会预判队友决策，并根据智能体之间的通信日志动态调整各自策略，从而促进相互理解并增强协作规划能力；
- [Wang et al., 2023a] 提出 “Learning through Communication（LTC）” 范式，利用多智能体的通信日志生成数据集，用来训练或微调 LLM。LTC 使智能体能够通过与环境和其他智能体交互而持续适应与提升，突破了仅靠上下文学习或监督微调、却无法充分利用交互反馈的限制。

自演化的关键点在于：智能体不仅是在“利用历史”，而是在**主动改写自己的画像、目标或策略**。

##### 3. Dynamic Generation（动态生成）

在某些场景中，系统还可以在运行过程中**按需生成新的智能体**。这种能力使系统能够更有效地扩展与适配，因为它可以引入那些专门为当前需求与挑战而设计的新 agent。

随着 LLM-MA 系统中的智能体数量不断增加，如何管理各种类型的智能体所带来的复杂性，已经成为一个关键问题。“智能体编排（Agents Orchestration）”因此逐渐成为核心挑战，并开始得到研究关注。作者表示，他们会在第 6.4 节进一步讨论这一点。

> [!warning] 易混点（非原文）
> “记忆（memory）”与“自演化（self-evolution）”虽然都在利用历史信息，但不完全一样：
> - 记忆更像“查经验再做事”；
> - 自演化更像“根据经验改自己”。
>
> 这是一个很值得后续做研究时细分的边界。

## 表 1：LLM-MA 研究总结（按原文重构）

> [!note] 译者注（非原文）
> 原论文中的表 1 是一张跨页大表，信息密度很高。为了便于 Obsidian 阅读，这里按原列信息重构为 Markdown 表格。含义保持不变，但排版不是原 PDF 的视觉样式。

| 动机 | 研究领域与目标 | 工作 | 环境接口 | 画像方法 | 画像示例 | 通信范式 | 通信结构 | 反馈来源 | 能力调整 |
|---|---|---|---|---|---|---|---|---|---|
| Problem Solving | Software development | [Qian et al., 2023] | Sandbox | Pre-defined, Model-Generated | CTO, programmer | Cooperative | Layered | Environment, Agent interaction, Human | Memory, Self-Evolution |
| Problem Solving | Software development | [Hong et al., 2023] | Sandbox | Pre-defined | Product Manager, Engineer | Cooperative | Layered, Shared Message Pool | Environment, Agent interaction, Human | Memory, Self-Evolution |
| Problem Solving | Software development | [Dong et al., 2023b] | Sandbox | Pre-defined, Model-Generated | Analyst, coder | Cooperative | Layered | Environment, Agent interaction | Memory, Self-Evolution |
| Problem Solving | Embodied Agents / Multi-robot planning | [Chen et al., 2023d] | Sandbox, Physical | Pre-defined | Robots | Cooperative | Centralized, Decentralized | Environment, Agent interaction | Memory |
| Problem Solving | Embodied Agents / Multi-robot collaboration | [Mandi et al., 2023] | Sandbox, Physical | Pre-defined | Robots | Cooperative | Decentralized | Environment, Agent interaction | Memory |
| Problem Solving | Embodied Agents / Multi-agents cooperation | [Zhang et al., 2023c] | Sandbox | Pre-defined | Robots | Cooperative | Decentralized | Environment, Agent interaction | Memory |
| Problem Solving | Science Experiments / Optimization of MOF | [Zheng et al., 2023] | Physical | Pre-defined | Strategy planners, literature collector, coder | Cooperative | Centralized | Environment, Human | Memory |
| Problem Solving | Science Debate / Improving Factuality | [Du et al., 2023] | None | Pre-defined | Agents | Debate | Decentralized | Agent interaction | Memory |
| Problem Solving | Science Debate / Examining Inter-Consistency | [Xiong et al., 2023] | None | Pre-defined | Proponent, Opponent, Judge | Debate | Centralized, Decentralized | Agent interaction | Memory |
| Problem Solving | Science Debate / Evaluators for debates | [Chan et al., 2023] | None | Pre-defined | Agents | Debate | Centralized, Decentralized | Agent interaction | Memory |
| Problem Solving | Science Debate / Multi-Agents for Medication | [Tang et al., 2023] | None | Pre-defined | Cardiology, Surgery | Debate, Cooperative | Centralized, Decentralized | Agent interaction | Memory |
| World Simulation | Society / Modest Community (25 persons) | [Park et al., 2023] | Sandbox | Model-generated | Pharmacy, shopkeeper | - | - | Environment, Agent interaction | Memory |
| World Simulation | Society / Online community (1000 persons) | [Park et al., 2022] | None | Pre-defined, Model-generated | Camping, fishing | - | - | Agent interaction | Dynamic Generation |
| World Simulation | Society / Emotion propagation | [Gao et al., 2023a] | None | Pre-defined, Model-generated | Real-world user | - | - | Agent interaction | Memory |
| World Simulation | Society / Real-time social interactions | [Kaiya et al., 2023] | Sandbox | Pre-defined | Real-world user | - | - | Environment, Agent interaction | Memory |
| World Simulation | Society / Opinion dynamics | [Li et al., 2023a] | None | Pre-defined | NIN, NINL, NIL | - | - | Agent interaction | Memory |
| World Simulation | Gaming / Werewolf | [Xu et al., 2023b], [Xu et al., 2023c] | Sandbox | Pre-defined | Seer, werewolf, villager | Cooperative, Debate, Competitive | Decentralized | Environment, Agent interaction | Memory |
| World Simulation | Gaming / Avalon | [Light et al., 2023a], [Wang et al., 2023c] | Sandbox | Pre-defined | Servant, Merlin, Assassin | Cooperative, Debate, Competitive | Decentralized | Environment, Agent interaction | Memory |
| World Simulation | Gaming / Welfare Diplomacy | [Mukobi et al., 2023] | Sandbox | Pre-defined | Countries | Cooperative, Competitive | Decentralized | Environment, Agent interaction | Memory |
| World Simulation | Psychology / Human behavior simulation | [Aher et al., 2023] | Sandbox | Pre-defined | Humans | - | - | Agent interaction | Memory |
| World Simulation | Psychology / Collaboration exploring | [Zhang et al., 2023d] | None | Pre-defined | Agents | Cooperative, Debate | Decentralized | Agent interaction | Memory |
| World Simulation | Economy / Macroeconomic simulation | [Li et al., 2023e] | None | Pre-defined, Model-generated | Labor | Cooperative | Decentralized | Agent interaction | Memory |
| World Simulation | Economy / Information marketplaces | [Anonymous, 2023] | Sandbox | Pre-defined, Data-Derived | Buyer | Cooperative, Competitive | Decentralized | Environment, Agent interaction | Memory |
| World Simulation | Economy / Improving financial trading | [Li et al., 2023g] | Physical | Pre-defined | Trader | Debate | Decentralized | Environment, Agent interaction | Memory |
| World Simulation | Economy / Economic theories | [Zhao et al., 2023] | Sandbox | Pre-defined, Model-Generated | Restaurant, Customer | Competitive | Decentralized | Environment, Agent interaction | Memory, Self-Evolution |
| World Simulation | Recommender Systems / Simulating user behaviors | [Zhang et al., 2023a] | Sandbox | Data-Derived | Users from MovieLens-1M | - | - | Environment | Memory |
| World Simulation | Recommender Systems / Simulating user-item interactions | [Zhang et al., 2023e] | Sandbox | Pre-defined, Data-Derived | User Agents, Item Agents | Cooperative | Decentralized | Environment, Agent interaction | Memory |
| World Simulation | Policy Making / Public Administration | [Xiao et al., 2023] | None | Pre-defined | Residents | Cooperative | Decentralized | Agent interaction | Memory |
| World Simulation | Policy Making / War Simulation | [Hua et al., 2023] | None | Pre-defined | Countries | Competitive | Decentralized | Agent interaction | Memory |
| World Simulation | Disease / Human Behaviors to epidemics | [Ghaffarzadegan et al., 2023] | Sandbox | Pre-defined, Model-Generated | Conformity traits | Cooperative | Decentralized | Environment, Agent interaction | Memory |
| World Simulation | Disease / Public health | [Williams et al., 2023] | Sandbox | Pre-defined, Model-Generated | Adults aged 18 to 64 | Cooperative | Decentralized | Environment, Agent interaction | Memory, Dynamic Generation |

**表 1 原表题中文翻译：**
LLM-MA 研究工作总结。作者按照研究动机、研究领域与目标，对当前工作进行分类，并从智能体—环境接口、智能体画像设定、智能体通信和能力获取等方面进行细化描述。“-” 表示对应工作中未特别说明该元素。

## 本章小结

> [!note] 译者注（非原文）
> 这一章最关键的五个 take-away：
> 1. 作者把 LLM 多智能体理解为一个“环境—角色—通信—反馈”的闭环系统。
> 2. **环境接口**决定了 agent 到底是在操作世界、模拟世界，还是只做群体讨论。
> 3. **画像设定**决定了 agent 是“谁”、能干什么、受什么约束。
> 4. **通信**不仅是传消息，更是一个关于范式、拓扑和信息内容的系统设计问题。
> 5. **能力获取**不是只靠提示词，而是通过反馈、记忆、自演化、动态生成不断调整。
>
> 读完这一章后，你已经拿到了一套分析后续 multi-agent 论文的“通用问卷”。

## 术语对照

| 英文术语 | 中文译法 | 本章语境说明 |
|---|---|---|
| agents-environment interface | 智能体—环境接口 | agent 如何感知并作用于环境 |
| operational environment | 运行环境 | 系统被部署和交互的具体场景 |
| agent profiling | 智能体画像设定 | 对角色、能力、行为、约束的刻画 |
| communication paradigm | 通信范式 | 协作、辩论、竞争等交互风格 |
| communication structure | 通信结构 | 中心化、去中心化、分层、消息池等拓扑 |
| communication content | 通信内容 | agent 之间实际交换的信息 |
| feedback | 反馈 | 来自环境、他人或人的回路信号 |
| self-evolution | 自演化 | 根据反馈改写自身目标、策略或行为 |
| dynamic generation | 动态生成 | 系统运行时按需生成新 agent |
| orchestration | 编排 | 多 agent 的任务分配与协同管理 |

## 思考题

### 题目 1：作者为什么把“通信”单独拆成一个大维度，而不是把它当成环境接口的一部分？

> [!tip] 理解提示（非原文）
> 想想“看环境”和“看彼此”在多智能体里是不是两个完全不同的问题。

> [!note] 译者注（非原文）
> 因为在多智能体系统里，很多核心能力并不是 agent 单独从环境里学到的，而是通过**相互交流、相互评估、相互纠偏**产生的。环境接口描述的是 agent 和世界之间的关系；通信描述的是 agent 和 agent 之间的关系。后者是集体智能出现的关键。

### 题目 2：为什么“记忆”和“自演化”要区分开？

> [!warning] 易混点（非原文）
> 二者都在使用历史信息，但作用层次不一样。

> [!note] 译者注（非原文）
> 记忆主要是“保留和调取过去经验”，帮助当前决策；自演化则是“利用过去经验来改变自己”，包括重设目标、调整规划策略，甚至用通信日志来训练模型。前者偏检索与调用，后者偏结构性调整。

## 相关页面

- [[raw/lessons/Multi-Agent-Survey/01-progress-and-challenges/02-survey-of-progress-and-challenges-background]]
- [[raw/lessons/Multi-Agent-Survey/01-progress-and-challenges/04-survey-of-progress-and-challenges-applications]]
- [[raw/paper/multi-agent/01-large-language-model-based-multi-agents-survey-of-progress-and-challenges-2024.pdf]]

## 下一步学习

- 上一篇：[[raw/lessons/Multi-Agent-Survey/01-progress-and-challenges/02-survey-of-progress-and-challenges-background|02-background]]
- 下一篇：[[raw/lessons/Multi-Agent-Survey/01-progress-and-challenges/04-survey-of-progress-and-challenges-applications|04-applications]]
