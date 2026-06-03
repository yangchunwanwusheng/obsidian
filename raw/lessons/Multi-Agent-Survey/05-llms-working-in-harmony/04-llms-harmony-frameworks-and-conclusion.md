---
type: lesson
tags: [多智能体, multi-agent, survey, 框架, AutoGen, CAMEL, CrewAI, MetaGPT, LangGraph, 方法论, 结论, 论文翻译, 学习笔记]
created: 2026-05-15
updated: 2026-05-15
topic: LLM 多智能体综述中文精读（第 5 篇-04）技术框架 + 方法论 + 讨论 + 结论
difficulty: intermediate
prerequisites:
  - [[raw/lessons/Multi-Agent-Survey/05-llms-working-in-harmony/00-llms-harmony-essence]]
  - [[raw/lessons/Multi-Agent-Survey/05-llms-working-in-harmony/03-llms-harmony-memory]]
sources:
  - https://arxiv.org/abs/2504.01963
status: completed
series:
  name: "Multi-Agent Survey｜LLMs Working in Harmony（2025）"
  part: 4
paper_meta:
  paper_id: "multi-agent-survey-05"
  title: "LLMs Working in Harmony: A Survey on the Technological Aspects of Building Effective LLM-Based Multi Agent Systems"
  authors: "RM Aratchige, Dr. WMKS Ilmini"
  year: "2025"
  link: "https://arxiv.org/abs/2504.01963"
  section: "Section II-D + Section III + Section IV + Section V"
---

# LLM 多智能体综述中文精读（第 5 篇-04）：技术框架 + 方法论 + 讨论 + 结论

> 本篇覆盖 **Section II-D Technologies/Frameworks**、**Section III Methodology**、**Section IV Discussion** 和 **Section V Conclusion**。核心任务是理解五个主流工程框架的设计理念，以及作者的最终结论——哪些技术在实践中真正有效。

## 本章导读

> [!note] 译者注（非原文）
> 这是本论文的最后一章，也是信息密度很高的一章。五个工程框架各有其设计哲学：AutoGen 强调对话编程，CAMEL 注重角色扮演，CrewAI 侧重流程挖掘，MetaGPT 引入 SOP，LangGraph 用图结构建模。作者在最后的 Discussion 中给出了明确的结论：**MoA 是架构首选，ReAct 是规划首选，记忆和框架选择需根据应用场景决定。**

## 正文翻译

> [!warning] 易混点（非原文）
> 从这里开始进入原文翻译。所有补充解释都会单独标成非原文。

### 原文标题：II-D Technologies / Frameworks

LLM 多智能体系统的开发高度依赖各种技术和框架，这些工具为 agent 的高效协作和任务执行提供了必要的基础设施。这些框架提供了构建、部署和管理多智能体环境的基本工具，实现 agent 间的无缝通信和协调。

像 AutoGen 和 MetaGPT 这样的技术使 agent 能够基于实时数据和交互生成动态响应和解决方案。CAMEL 和 CrewAI 等框架提供了集成环境，简化了多智能体系统的设计和编排，增强了可扩展性和灵活性。

#### II-D.1 AutoGen

Wu 等人提出的 AutoGen 是一个开源框架，旨在通过**多 agent 对话**来促进 LLM 应用的开发。这个创新框架允许多个 agent 通过可定制、可对话的 agent 在多种模式下交互、协作和完成任务。

AutoGen 支持 LLM、人类输入和各种工具的组合，使开发者能够灵活定义 agent 的交互行为。通过使用自然语言和计算机代码来编程对话模式，AutoGen 作为一个通用框架适用于广泛的应用，包括数学、编程、问答、运筹学和在线决策。

**核心设计原则：**
- **可对话 agent（conversable agent）**：每个 agent 可以发送和接收消息，同时维护自己的内部上下文
- **对话编程（conversation programming）**：包含两个关键概念
  - **计算（computation）**：agent 在多 agent 对话中执行的行动
  - **控制流（control flow）**：规定这些行动发生的顺序或条件

**局限：**
- 仍处于早期开发阶段
- 现有 agent 实现的整合需要进一步研究
- 安全挑战——尤其是应用复杂度增加时的自动化与人类控制之间的平衡

> [!tip] 理解提示（非原文）
> AutoGen 的核心卖点是"对话即编程"——你不用写复杂的编排逻辑，而是定义 agent 之间的对话规则，让它们自己协商、合作。这大大降低了多智能体系统的开发门槛。但它仍处于早期阶段，文档和社区生态还在成长中。

#### II-D.2 CAMEL

Li 等人提出的 CAMEL 是一个旨在增强聊天式语言模型中**通信 agent 自主协作**的新框架。随着这些模型的不断进化，其有效性往往依赖于人类输入来引导对话——这可能是一项艰巨且耗时的工作。

CAMEL 采用**角色扮演方法**和**启始提示（inception prompting）**，使 agent 能够在保持与人类意图一致的同时协作完成任异。该框架不仅促进了对话数据的生成，还作为探索 agent 群体行为和能力的宝贵资源。

**核心挑战与应对：**
- 对话偏离（conversation deviation）
- 角色翻转（role flipping）
- 终止条件的定义

开源的 CAMEL 库包含各种 agent 的实现、数据生成管道和分析工具。

> [!tip] 理解提示（非原文）
> CAMEL 的独特之处在于它把"角色扮演"作为多 agent 协作的核心机制。不是给 agent 分配功能模块，而是给它们分配"身份"——你是医生、我是程序员、他是项目经理。这种设计使 agent 之间的协作更像人类的团队合作流程。

#### II-D.3 CrewAI

Berti 等人提出的 CrewAI 框架是实现基于 AI 的 agent 工作流（AI-Based Agents Workflow, AgWf）范式的关键组件，旨在通过整合 LLM 来增强流程挖掘（Process Mining）任务。

**核心概念：**
- **AI-based agents**：将 LLM 与定制化的系统提示（system prompts）结合，通过角色提示将模型行为与特定角色对齐
- **AI-based tasks**：通过链接到 agent 的文本指令来定义，明确职责划分
- **Tools**：作为 Python 类或函数实现，可根据文档字符串（包括输入参数和输出类型）集成到任务中

**执行模式：**
- 传统的顺序执行
- 通过层级流程实现更复杂的并发执行（仍需进一步发展）

> [!tip] 理解提示（非原文）
> CrewAI 的设计理念偏向"工程实用主义"——它不追求学术上的新颖性，而是把 LLM agent 以"团队"（Crew）的形式组织起来，每人有角色、有任务、有工具。它特别适合需要结构化工作流的场景，如流程挖掘、数据分析等。

#### II-D.4 MetaGPT

Hong 等人提出的 MetaGPT 是一个创新的**元编程框架**，旨在增强基于 LLM 的多智能体系统之间的协作。现有的 LLM 多智能体系统在简单对话任务上表现出色，但在复杂场景中因逻辑不一致和级联幻觉而挣扎——这些问题源于简单地将 LLM 链接在一起。

MetaGPT 通过**将人类高效工作流程编码为结构化提示序列**——即标准化操作流程（Standardized Operating Procedures, SOPs）——来解决这些挑战。

**核心特征：**
- **装配线范式**：为不同 agent 高效分配不同角色，将复杂任务分解为可管理的子任异
- **角色专业化**：通过角色专精和结构化通信，增强 agent 交互和信息共享的能力
- **结构化通信协议**：包含结构化接口和发布-订阅机制，agent 可以从其他角色和环境中获取相关信息
- **可执行反馈机制**：运行时的实时调试和代码执行，显著提升代码生成质量

**实验表现：**
在 HumanEval 和 MBPP 等基准测试上表现令人印象深刻。

> [!tip] 理解提示（非原文）
> MetaGPT 是本章中设计理念最完整的框架。它直接把软件开发公司的 SOP（需求分析→架构设计→编码→测试）编码进了多 agent 的工作流中。这使得 agent 不再是"自由聊天"，而是按照既定的、经过人类验证的高效流程来协作。这也是它控制级联幻觉的核心手段——每一步都有标准化输出，作为下一步的标准化输入。

#### II-D.5 LangGraph

LangGraph 框架是开发高级 RAG 系统的强大工具，特别适用于基于知识的问答应用。与传统的 RAG 模型不同——后者常因依赖静态预加载知识而导致准确性下降——LangGraph 利用**图技术**来增强信息检索过程。

通过实现高效的搜索和对检索数据可靠性的评估，LangGraph 显著提高了生成回答的上下文理解和准确性。这种方法不仅规避了现有 RAG 模型的局限，还实现了与实时数据的集成。

**核心特点：**
- **有状态的多 actor 应用环境**：专为 LLM 设计
- **循环图结构**：允许开发者定义复杂的工作流并控制应用状态
- **LangGraph Conversational Retrieval Agent**：结合语言处理、AI 模型集成和图数据管理

> [!tip] 理解提示（非原文）
> LangGraph 的独特之处在于用"图"来建模 agent 工作流——节点是处理步骤，边是数据/控制流，而且支持循环。这意味着你可以用 LangGraph 构建比线性流水线更复杂的工作流（如带反馈循环的自适应系统）。它和 LangChain 生态深度绑定，适合已经在用 LangChain 的团队。

## 技术框架对比总览（Table IV 翻译）

| 框架 | 关键特征 | 优势 | 局限 |
|------|---------|------|------|
| AutoGen | 多 agent 对话开源框架；支持多种交互模式；组合 LLM、人类输入和工具 | 增强多 agent 合作；灵活定义 agent 行为；超越 SOTA 方法 | 仍处于早期开发；现有 agent 集成需进一步研究 |
| CAMEL | 角色扮演 + 启始提示；促进通信 agent 任务完成；生成对话数据 | 自主合作减少人类输入；对话挑战的可扩展解决方案；开源实现 | 对话偏离和角色翻转挑战；需在多样化上下文中评估 |
| CrewAI | AI agent 工作流框架；整合 LLM 与 AI 任务和工具；支持顺序和并发执行 | 增强 LLM 推理能力；将复杂任务分解为可管理工作流；促进高质量输出 | 并发执行需进一步发展；手动设计 agent 可能复杂 |
| MetaGPT | 元编程框架整合 SOP；为 agent 分配角色并增强结构化通信 | 通过类人验证减少错误；改善协作和方案生成；基准测试表现显著 | 实现复杂度高；对结构化提示的依赖可能限制灵活性 |
| LangGraph | 用图技术增强 RAG 系统；允许 agent 应用循环工作流 | 提高准确性和上下文理解；实现实时数据集成；支持 agent 间协作交互 | 循环工作流管理复杂；图结构依赖可能有学习曲线 |

### 原文标题：III Methodology

本综述旨在系统评估和综合 LLM 多智能体系统的现有研究，特别关注直接影响其应用和可扩展性的技术方面。

**研究方法：**

1. **研究问题定义**：首先明确研究范围——仅关注**专为 LLM 多智能体系统设计的技术**，而非独立讨论 LLM 和多智能体系统。这一选择确保了对两者交集的深入审视。

2. **四个主要技术维度**：Architecture, Memory, Planning, Technologies/Frameworks——每个维度都针对 LLM 多智能体系统设计和运行中的核心组件。

3. **文献检索**：跨多个受认可的学术来源（Google Scholar, IEEE Xplore, arXiv），使用与四个主题相关的目标关键词。其中 **arXiv 被证明是最有价值的存储库**——提供了高浓度的相关论文，详细描述了 LLM 多智能体系统的最新发展和实验应用。

4. **论文筛选**：每篇论文根据其与已确定主题的相关性进行评估，优先考虑来自可信作者和著名会议/期刊的出版物。

5. **内容分析**：对每篇入选论文进行详细分析，特别关注方法、架构和实验设置的描述。记录每种方法的优点和局限性。

> [!tip] 理解提示（非原文）
> 这篇论文的方法论很标准，但有一个值得注意的点：作者明确指出 arXiv 是最高产的相关论文来源。这反映了一个现实——LLM 多智能体领域的发展速度远远超过了传统同行评审的周期，最前沿的工作往往最先出现在 arXiv 上。但这也意味着，综述中引用的许多论文尚未经过同行评审，这是读者需要心中有数的。

### 原文标题：IV Discussion

#### IV-A Key Findings

**架构方面：**
Mixture of Agents (MoA) 被确定为最高效的架构设计。其分层模型将 agent 分为 proposer 和 aggregator，通过角色专业化最大化不同 LLM 的优势。MoA 在 AlpacaEval 2.0 和 MT-Bench 上展现出色性能。但其局限在于：并非所有 LLM 在两个角色上表现相同（如 WizardLM 擅长提议但不擅长聚合），且扩展 agent 数量会引入管理复杂度。

**记忆方面：**
分析发现，**不同的记忆方案根据具体用例各有其适用性**。短期记忆模型在 agent 需要快速访问近期信息时表现出色，但不需要大量历史上下文。长期记忆模型则适用于需要在长交互中保持深入信息的应用场景。记忆架构的选择应与系统的预期功能对齐——这在高响应性或需要细致历史回忆的场景中尤为关键。

**规划方面：**
**ReAct 框架被确定为 LLM 多智能体系统中整合推理和行动规划的首选方法**。通过交替生成推理轨迹和任务特定行动，ReAct 有效应对了复杂任务处理的传统局限。它在多跳问答和事实验证等多样化任务中展现优势——同时缓解幻觉和错误传播。

**技术/框架方面：**
框架的选择往往取决于应用需求，而非某一个框架的固有优越性。**易集成性、特定编程语言支持、可扩展性和与外部数据源的兼容性**等因素在决定最优框架时起关键作用。随着领域的发展，框架对新进展的适应性和与新工具的兼容性将是持续实用的关键。

#### IV-B Future Directions

本综述为 LLM 多智能体系统的未来发展提供了路线图：

1. **架构**：MoA 已验证有效，但需要更先进的编排技术来管理更大规模的 agent 交互
2. **规划**：ReAct 是当前最优，但与强化学习等多任务训练的整合可增强其鲁棒性
3. **记忆**：记忆架构需与应用目标对齐；未来需要更智能的自动记忆管理方案
4. **框架**：框架的适应性比具体特性更重要——需要能跟上 LLM 技术快速进化的可扩展基础设施

### 原文标题：V Conclusion

LLM 多智能体系统的发展标志着实现复杂协作式 AI 应用的重要一步。本综述确定了增强系统有效性的核心框架和方法：

- **MoA** 用于结构化 agent 协作
- **ReAct** 用于整合推理和行动
- **记忆架构**保持多样化，由应用特定需求决定短期或长期数据保留的最合适方案
- **技术选择**取决于应用需求，偏好能集成外部数据并支持可扩展性的适应性框架

尽管这些系统展现出巨大潜力，但持续存在的挑战——包括计算成本、通信优化和角色专业化——必须在更广泛适用性实现之前得到解决。未来的研究和实践进展对于进化这些框架至关重要，从而推动构建稳健、灵活的 LLM 多智能体系统，以支持多样化领域中的复杂 AI 驱动工作流。

> [!tip] 理解提示（非原文）
> 结论中的一句话特别值得注意："Memory architectures remain diverse, with application-specific requirements determining the most suitable approach." 作者在四大支柱中，只有对记忆没有给出最优推荐。这本身也是一个重要信号——当前记忆方案还没有出现像 MoA 之于架构、ReAct 之于规划那样的通用性最优方案。

## 核心发现速查卡

| 技术维度 | 首选方案 | 作者的理由 | 需要注意的陷阱 |
|---------|---------|-----------|-------------|
| Architecture | **MoA** | proposer-aggregator 分层，最大化不同 LLM 优势 | 并非所有模型都胜任两种角色 |
| Planning | **ReAct** | 推理-行动交替，可解释性强，幻觉控制好 | 大动作空间的复杂任务需要额外增强 |
| Memory | **无单一最优** | 场景高度依赖 | 短期 vs 长期、向量 vs 符号的权衡需仔细评估 |
| Frameworks | **场景依赖** | 需考虑集成性、扩展性、数据源兼容性 | 框架的生态系统质量比功能列表更重要 |

## 本章小结

> [!note] 译者注（非原文）
> 本章作为本论文的收尾，给出了整篇综述最核心的四个结论：
> 1. MoA 是架构维度的最优选择——分层设计 proved effective。
> 2. ReAct 是规划维度的最优选择——简洁、可靠、可解释。
> 3. 记忆和框架没有唯一最优解——不是因为它们不重要，而是因为它们高度场景依赖。
> 4. 剩余的关键挑战——计算成本、通信优化、角色专业化——仍然是该领域普遍的开放问题。
>
> 这篇论文最有价值的遗产，可能是它建立的"四柱评估框架"。以后再读任何一个新的 LLM-MAS 方法，你都可以问：它在架构、规划、记忆、工程框架这四个维度上做了什么贡献？哪个维度是它的特色，哪个维度是它借用的？

## 五大工程框架选型速查

| 你的需求 | 推荐框架 | 核心理由 |
|---------|---------|---------|
| 快速搭建多 agent 对话系统 | AutoGen | 对话编程理念，最小化开发成本 |
| 角色驱动的自主协作 | CAMEL | 角色扮演 + 启始提示，减少人类干预 |
| 结构化的多 step 工作流 | CrewAI | agent + task + tool 三层设计 |
| 需要类人 SOP 的复杂开发 | MetaGPT | SOP 编码化，装配线范式 |
| 图结构的复杂工作流 | LangGraph | 循环图 + 有状态管理 |

## 术语对照

| 英文术语 | 中文译法 | 本章语境说明 |
|----------|---------|-------------|
| conversable agent | 可对话 agent | AutoGen 的基础 agent 抽象 |
| conversation programming | 对话编程 | 通过定义对话模式来编排 agent |
| inception prompting | 启始提示 | CAMEL 中启动角色扮演的提示技术 |
| role flipping | 角色翻转 | 多 agent 对话中角色意外切换的问题 |
| Standardized Operating Procedures (SOPs) | 标准化操作流程 | MetaGPT 中将人类工作流程编码为提示序列 |
| assembly line paradigm | 装配线范式 | MetaGPT 的任务分解与分配模式 |
| cyclic graph structure | 循环图结构 | LangGraph 中支持反馈循环的工作流设计 |
| execution-based feedback | 可执行反馈 | MetaGPT 的运行时调试机制 |

## 思考题

### 题目 1：MetaGPT 的 SOP 方法为什么能减少级联幻觉？

> [!tip] 理解提示（非原文）
> 想一想：如果每个 agent 的输出都必须符合预定义的格式，下一个 agent 的输入就会更可预测。

> [!note] 译者注（非原文）
> 级联幻觉的根本原因在于信息传递中的熵增——一个 agent 的输出中的小错误被下一个 agent 放大、曲解或扩散。MetaGPT 的 SOP 方案通过两个机制控制这个问题：(1) 标准化输出格式——每个阶段的输出必须符合预设的结构，减少了"意外信息"被传入下一阶段的可能性；(2) 中间验证——在装配线上的每个节点，输出都可以被检查是否符合规范。这本质上是一种"结构化信息约束"，类似通信系统中的纠错编码。

### 题目 2：作者为什么得出结论"记忆和框架没有唯一最优解"，而架构和规划却有？

> [!tip] 理解提示（非原文）
> 想一想 MoA 和 ReAct 的共同特征是什么？

> [!note] 译者注（非原文）
> MoA 和 ReAct 的共同特征是它们在各自维度上解决了一个**通用性问题**：MoA 解决了"如何让多个 agent 的输出有效整合"（proposer-aggregator 分层），ReAct 解决了"如何让推理和行动有效协同"（交替生成）。这些问题的答案是跨场景通用的——无论是编码、问答还是决策，这些模式都适用。
>
> 但记忆和框架面临的是**场景特定问题**：一个需要快速对话响应的客服系统和一个需要长期个性化记忆的心理陪伴系统，对记忆的要求完全不同；一个快速原型和一个生产级系统，对框架的要求也完全不同。当问题的答案本身就是场景依赖的时候，就不存在跨场景的"最优解"。

## 相关页面

- [[raw/lessons/Multi-Agent-Survey/05-llms-working-in-harmony/00-llms-harmony-essence]]
- [[raw/lessons/Multi-Agent-Survey/05-llms-working-in-harmony/01-llms-harmony-introduction-and-architecture]]
- [[raw/lessons/Multi-Agent-Survey/05-llms-working-in-harmony/02-llms-harmony-planning]]
- [[raw/lessons/Multi-Agent-Survey/05-llms-working-in-harmony/03-llms-harmony-memory]]

## 下一步学习

- 上一篇：[[raw/lessons/Multi-Agent-Survey/05-llms-working-in-harmony/03-llms-harmony-memory\|03-memory]]
- 返回系列入口：[[raw/lessons/Multi-Agent-Survey/05-llms-working-in-harmony/00-llms-harmony-essence\|00-essence (系列总览)]]
