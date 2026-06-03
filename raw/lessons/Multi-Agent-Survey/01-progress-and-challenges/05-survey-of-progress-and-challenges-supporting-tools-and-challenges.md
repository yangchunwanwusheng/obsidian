---
type: lesson
tags: [多智能体, multi-agent, survey, 论文翻译, 学习笔记]
created: 2026-05-07
updated: 2026-05-07
topic: LLM 多智能体综述中文精读（五）框架、基准、挑战与结论
difficulty: intermediate
prerequisites:
  - [[raw/lessons/Multi-Agent-Survey/01-progress-and-challenges/04-survey-of-progress-and-challenges-applications]]
sources:
  - [[raw/paper/multi-agent/01-large-language-model-based-multi-agents-survey-of-progress-and-challenges-2024.pdf]]
  - https://arxiv.org/abs/2402.01680
status: completed
series:
  name: "Multi-Agent Survey｜Progress and Challenges（2024）"
  part: 5
paper_meta:
  paper_id: "multi-agent-survey-01"
  title: "Large Language Model based Multi-Agents: A Survey of Progress and Challenges"
  authors: "Taicheng Guo, Xiuying Chen, Yaqi Wang, Ruidi Chang, Shichao Pei, Nitesh V. Chawla, Olaf Wiest, Xiangliang Zhang"
  year: "2024"
  link: "https://arxiv.org/abs/2402.01680"
  section: "Section 5-7"
---

# LLM 多智能体综述中文精读（五）：框架、基准、挑战与结论

> 本篇覆盖原论文 **Section 5 Implementation Tools and Resources、Section 6 Challenges and Opportunities、Section 7 Conclusion**，并保留表 2。参考文献全文则单独放到下一篇。

## 本章导读

> [!note] 译者注（非原文）
> 前面几篇主要是在回答“LLM 多智能体是什么、怎么拆、用在哪里”；这一篇开始回答更现实的问题：
> - 如果我要自己做一个 LLM-MA 系统，能用什么框架？
> - 这个方向现在常用什么数据集和 benchmark？
> - 研究真正卡在哪里？
>
> 也可以说，这一篇把综述从“认知地图”推进到“研究入口与难点地图”。

## 正文翻译

> [!warning] 易混点（非原文）
> 从这里开始是原论文第 5-7 节的完整中文翻译；所有补充说明都用单独 callout 标出。

### 原文标题：5 Implementation Tools and Resources

### 原文标题：5.1 Multi-Agents Framework

作者详细介绍了三个开源多智能体框架：**MetaGPT**、**CAMEL** 和 **AutoGen**。它们都利用语言模型来完成复杂任务求解，并强调多智能体协作，但在方法与应用侧重点上有所不同。

#### MetaGPT

MetaGPT 的设计目标，是把人类工作流嵌入到语言模型智能体的运行过程中，从而减少复杂任务中常见的幻觉（hallucination）问题。它通过把标准操作流程（Standard Operating Procedures, SOPs）编码进系统，并采用“流水线”式方法，把不同角色分配给不同智能体。

#### CAMEL

CAMEL（Communicative Agent Framework）更偏向于促进智能体之间的自治合作。它使用一种名为 **inception prompting** 的新技术，引导对话型智能体朝着与人类目标一致的方向完成任务。该框架同时也可作为生成与研究对话数据的工具，帮助研究者理解可通信智能体的行为与交互方式。

#### AutoGen

AutoGen 是一个更加通用的框架，允许开发者借助语言模型来创建应用。它的独特之处在于具有很高的可定制性：开发者既可以用自然语言，也可以用代码来定义智能体如何交互。这种灵活性使它不仅可用于编码、数学等技术领域，也能应用于娱乐等面向消费者的场景。

作者还提到，较新的工作引入了面向**动态多智能体协作**的框架；另一些工作则提出了用于构建自治智能体的平台与库，强调其在任务求解与社会模拟中的可适应性。

> [!note] 译者注（非原文）
> 这一段的一个重要信号是：到 2024 年，这个领域已经不只是“论文原型”了，而是开始出现一批**可复用框架层**。
>
> 换句话说，multi-agent 正从“研究现象”走向“工程中间层”。这也是为什么后续几年会迅速出现更多 orchestration、framework、runtime 相关工作。

### 原文标题：5.2 Datasets and Benchmarks

作者在表 2 中总结了 LLM 多智能体研究中常用的数据集与基准。作者观察到：不同研究应用往往采用不同的数据集与 benchmark。

在**问题求解（Problem Solving）**场景中，大多数数据集与基准用于评估多个智能体通过合作或辩论所展现出的规划与推理能力。

在**世界模拟（World Simulation）**场景中，数据集与 benchmark 则主要用于评估：

- 模拟世界与真实世界之间的对齐程度；
- 不同智能体行为的分析质量。

然而，在一些研究应用中，例如科学团队执行实验（Science Team for Experiment Operations）与经济建模（economic modeling），目前仍缺乏足够完善的 benchmark。作者认为，这类 benchmark 的发展将大大增强人们衡量 LLM-MA 在这些复杂动态领域中成功程度与适用性的能力。

## 表 2：LLM-MA 常用数据集与基准（按原文重构）

> [!note] 译者注（非原文）
> 原表 2 同样是概览型表格。这里保留原有信息关系，并转换成适合笔记阅读的 Markdown 形式。

| 动机 | 领域 | 数据集 / 基准 | 使用工作 | 数据链接 |
|---|---|---|---|---|
| Problem Solving | Software Development | HumanEval | [Hong et al., 2023] | Link |
| Problem Solving | Software Development | MBPP | [Hong et al., 2023] | Link |
| Problem Solving | Software Development | SoftwareDev | [Hong et al., 2023] | Link |
| Problem Solving | Embodied AI | RoCoBench | [Mandi et al., 2023] | Link |
| Problem Solving | Embodied AI | Communicative Watch-And-Help (C-WAH) | [Zhang et al., 2023c] | Link |
| Problem Solving | Embodied AI | ThreeDWorld Multi-Agent Transport (TDW-MAT) | [Zhang et al., 2023c] | Link |
| Problem Solving | Embodied AI | HM3D v0.2 | [Yu et al., 2023] | Link |
| Problem Solving | Science Debate | MMLU | [Tang et al., 2023] | Link |
| Problem Solving | Science Debate | MedQA | [Tang et al., 2023] | Link |
| Problem Solving | Science Debate | PubMedQA | [Tang et al., 2023] | Link |
| Problem Solving | Science Debate | GSM8K | [Du et al., 2023] | Link |
| Problem Solving | Science Debate | StrategyQA | [Xiong et al., 2023] | Link |
| Problem Solving | Science Debate | Chess Move Validity | [Du et al., 2023] | Link |
| World Simulation | Society | SOTOPIA | [Zhou et al., 2023b] | / |
| World Simulation | Society | Gender Discrimination | [Gao et al., 2023a] | / |
| World Simulation | Society | Nuclear Energy | [Gao et al., 2023a] | / |
| World Simulation | Gaming | Werewolf | [Xu et al., 2023b] | / |
| World Simulation | Gaming | Avalon | [Light et al., 2023b] | / |
| World Simulation | Gaming | Welfare Diplomacy | [Mukobi et al., 2023] | / |
| World Simulation | Gaming | Layout in the Overcooked-AI environment | [Agashe et al., 2023] | / |
| World Simulation | Gaming | Chameleon | [Xu et al., 2023a] | Link |
| World Simulation | Gaming | Undercover | [Xu et al., 2023a] | Link |
| World Simulation | Psychology | Ultimatum Game TE | [Aher et al., 2023] | Link |
| World Simulation | Psychology | Garden Path TE | [Aher et al., 2023] | Link |
| World Simulation | Psychology | Wisdom of Crowds TE | [Aher et al., 2023] | Link |
| World Simulation | Recommender System | MovieLens-1M | [Zhang et al., 2023a] | Link |
| World Simulation | Recommender System | Amazon review dataset | [Zhang et al., 2023e] | / |
| World Simulation | Policy Making | Board Connectivity Evaluation | [Hua et al., 2023] | Link |

**表 2 原表题中文翻译：**
LLM-MA 研究中常用的数据集与基准。“/” 表示数据链接不可用。

> [!tip] 理解提示（非原文）
> 从表 2 里你能看出一个很现实的事实：
> 这个领域虽然热，但 benchmark 其实并不统一。
>
> 尤其是世界模拟那边，很多任务更像“研究性沙盒”，而不是标准化 leaderboard 任务，所以后面的评测问题自然会成为挑战。

### 原文标题：6 Challenges and Opportunities

对 LLM 多智能体框架与应用的研究正在快速推进，也因此带来了大量挑战与机会。作者识别出若干关键挑战以及未来可能的重要研究方向。

#### 原文标题：6.1 Advancing into Multi-Modal Environment

以往大多数 LLM-MA 研究都集中在**文本环境（text-based environments）**中，在文本处理与生成方面表现出色。然而，在多模态场景（multi-modal settings）中仍明显不足。在这些场景里，智能体需要与多种感知输入交互、理解来自不同模态的数据，并生成多种形式的输出，例如图像、音频、视频以及物理动作。

将 LLM 融入多模态环境，会带来额外挑战，例如：

- 如何处理多样数据类型；
- 如何让智能体彼此不仅能理解文本，还能理解其他信息形式并做出响应。

#### 原文标题：6.2 Addressing Hallucination

幻觉（hallucination）是 LLM 和单智能体 LLM 系统中的一个重要挑战，它指模型生成了事实错误的文本。然而在多智能体场景下，这个问题会进一步复杂化。

原因在于：**一个智能体的幻觉可能产生级联效应（cascading effect）**。由于多智能体系统本身是互联的，一个 agent 产生的错误信息可能被其他 agent 接受、转发，最终在系统中持续传播。

因此，在 LLM-MA 中检测并缓解幻觉，不仅重要，而且具有独特难度。这不仅要求修正单个智能体层面的错误，还要求对智能体之间的信息流进行管理，以防止错误在整个系统里扩散。

> [!warning] 易混点（非原文）
> 单 agent 幻觉更像“一个人说错话”；多 agent 幻觉则更像“错误在群体中被当成共识传开”。
>
> 所以多智能体中的 hallucination，已经不仅是生成问题，也是**传播问题**和**协作治理问题**。

#### 原文标题：6.3 Acquiring Collective Intelligence

在传统多智能体系统中，智能体通常借助强化学习，从离线训练数据集中学习。然而，LLM-MA 系统主要依赖即时反馈来学习，例如与环境或人类的交互反馈，这一点在第 3 节已经讨论过。

这种学习方式要求存在一个可靠的交互环境；而对于许多任务来说，设计这样一个交互环境并不容易，这会限制 LLM-MA 系统的可扩展性（scalability）。

此外，当前研究中主流的做法，是通过**记忆（Memory）**与**自演化（Self-Evolution）**来根据反馈调整智能体。虽然这些方法对单个智能体有效，但它们并没有充分利用 agent 网络作为整体所可能产生的**集体智能（collective intelligence）**。它们主要还是在孤立地调整单个 agent，而忽略了多智能体协同互动中可能涌现出的协同效应。

因此，如何**联合调整多个智能体**并实现真正意义上的最优集体智能，仍然是 LLM-MA 的关键挑战。

#### 原文标题：6.4 Scaling Up LLM-MA Systems

LLM-MA 系统由多个基于 LLM 的智能体构成，因此在智能体数量上会面临显著的可扩展性挑战。

从计算复杂度角度看，每个基于 LLM 的智能体通常都建立在 GPT-4 这一类大型语言模型之上，需要大量计算资源与内存。当 LLM-MA 系统中智能体数量增加时，资源需求也会显著上升。因此，在计算资源有限的场景下，开发这类系统会非常困难。

此外，随着系统中智能体数量增加，还会出现更多复杂性与研究机会，尤其是在：

- 高效智能体协调（efficient agent coordination）
- 通信（communication）
- 多智能体缩放规律（scaling laws of multi-agents）

等方面。

例如，agent 数量越多，确保它们有效协调与通信的难度就越高。作者引用相关工作指出，设计更先进的**智能体编排（Agents Orchestration）**方法会变得越来越重要。这类方法旨在优化：

- 智能体工作流
- 与不同 agent 相匹配的任务分配
- agent 之间的通信模式
- agent 之间的通信约束

有效的智能体编排有助于让多个 agent 和谐运行，减少冲突与冗余。此外，探索并定义控制多智能体系统在规模增大时行为与效率变化规律的 scaling laws，也是一项重要研究方向。

这些问题共同说明，需要新的创新方案来优化 LLM-MA 系统，使其既有效，又资源友好。

#### 原文标题：6.5 Evaluation and Benchmarks

作者已经在表 2 中总结了当前 LLM-MA 可用的数据集与 benchmark，但这还只是一个起点，远非完整。

作者识别出在评估 LLM-MA 系统和 benchmark 彼此性能时的两大挑战：

第一，现有大量研究主要关注在狭窄设定中评估**单个智能体的理解与推理**能力，而倾向于忽视多智能体系统中更广泛、更复杂的**涌现行为（emergent behaviors）**。

第二，在若干研究领域中，仍严重缺乏综合性的 benchmark，例如：

- 科学团队实验操作
- 经济分析
- 疾病传播模拟

这种缺口阻碍了人们准确评估与比较 LLM-MA 在这些关键领域中的真正能力。

#### 原文标题：6.6 Applications and Beyond

LLM-MA 系统的潜力远不止于目前已有应用。它在金融、教育、医疗、环境科学、城市规划等领域，都有希望推动更高级的计算型问题求解。

正如前文所讨论的，LLM-MA 系统既能处理复杂问题，也能模拟现实世界的多个方面。虽然当前 LLM 的角色扮演能力仍有限，但随着 LLM 技术持续进步，未来前景非常明亮。

人们可以预期，未来会出现：

- 更成熟的方法论
- 更多应用
- 更适合不同研究领域的数据集与 benchmark

此外，还可以从更多理论视角去研究 LLM-MA，例如：

- 认知科学（Cognitive Science）
- 符号人工智能（Symbolic Artificial Intelligence）
- 控制论（Cybernetics）
- 复杂系统（Complex Systems）
- 集体智能（Collective Intelligence）

这种多视角的理论探索，可能会帮助人们更全面地理解这一快速演进的领域，并激发新的创新应用。

### 原文标题：7 Conclusion

基于 LLM 的多智能体已经展现出令人鼓舞的集体智能，并迅速吸引了越来越多研究者的关注。

在这篇综述中，作者首先系统回顾了 LLM-MA 系统的发展，并从多个方面对其进行定位、区分和连接，包括：

- 智能体—环境接口
- 由 LLM 定义智能体画像的方式
- 智能体通信的管理策略
- 能力获取范式

作者还总结了 LLM-MA 在问题求解与世界模拟中的应用。

通过进一步整理常用数据集与 benchmark，并讨论挑战与未来机会，作者希望这篇综述能成为跨多个研究领域研究者的有用资源，激发未来研究去探索基于 LLM 的多智能体系统的潜力。

> [!note] 译者注（非原文）
> 这篇结论其实非常克制。作者没有夸张地说 multi-agent 已经成熟，而是更像在说：
> - 这个方向已经形成清晰研究版图；
> - 但真正困难的问题——多模态、幻觉传播、集体智能获取、扩展性、评测——才刚开始被认真对待。
>
> 这也是为什么这篇 2024 综述今天依然适合作为“入门总地图”。

## 本章小结

> [!note] 译者注（非原文）
> 这最后一篇正文可以总结成六个核心判断：
> 1. 到 2024 年，LLM 多智能体已经开始拥有可复用框架层，例如 MetaGPT、CAMEL、AutoGen。
> 2. benchmark 生态仍然分散，尤其在世界模拟方向上更明显。
> 3. 多模态环境是从“文本多 agent”走向“真实复杂世界多 agent”的关键门槛。
> 4. 幻觉在多智能体里会被放大成群体传播问题。
> 5. 真正的 collective intelligence 还远没有被充分解决，当前很多方法仍在“各调各的 agent”。
> 6. orchestration、evaluation、scaling law 很可能会成为后续几年最重要的主线问题之一。

## 术语对照

| 英文术语 | 中文译法 | 本章语境说明 |
|---|---|---|
| Multi-Agents Framework | 多智能体框架 | 用于构建和组织 multi-agent 系统的工程层工具 |
| inception prompting | 初始设定式提示 / inception prompting | CAMEL 中引导 agent 朝目标协作的技术 |
| hallucination | 幻觉 | 事实性错误及其传播问题 |
| cascading effect | 级联效应 | 一个 agent 的错误被整个系统放大 |
| collective intelligence | 集体智能 | 多个 agent 作为整体展现出的能力 |
| scaling laws | 缩放规律 | 系统规模增长时性能和效率如何变化 |
| emergent behaviors | 涌现行为 | 多主体互动后出现的整体层行为模式 |
| orchestration | 编排 | 对多个 agent 的任务、流程与通信进行组织 |

## 思考题

### 题目 1：为什么“评测”会成为 LLM 多智能体的核心难题？

> [!tip] 理解提示（非原文）
> 想想：单 agent 可以测答对率，但 multi-agent 可能还要测协作过程、传播机制、涌现行为。

> [!note] 译者注（非原文）
> 因为 multi-agent 的价值不只体现在最终答案上，还可能体现在过程组织、角色协作、通信效率、稳定性与涌现行为上。现有 benchmark 大多更适合评估单体推理，而不适合评估一个“群体系统”到底协作得好不好。

### 题目 2：为什么作者把 orchestration 提升为一个独立的重要方向？

> [!warning] 易混点（非原文）
> orchestration 不是简单的“多开几个 agent 并行”。

> [!note] 译者注（非原文）
> 因为当 agent 数量上升后，真正难的不是“有没有 agent”，而是“谁做什么、什么时候说话、谁可以影响谁、如何避免冲突和冗余”。这些都属于 orchestration。它本质上是多智能体系统的运行时治理问题。

## 相关页面

- [[raw/lessons/Multi-Agent-Survey/01-progress-and-challenges/04-survey-of-progress-and-challenges-applications]]
- [[raw/lessons/Multi-Agent-Survey/01-progress-and-challenges/06-survey-of-progress-and-challenges-references]]
- [[_templates/paper-translation-lesson]]

## 下一步学习

- 上一篇：[[raw/lessons/Multi-Agent-Survey/01-progress-and-challenges/04-survey-of-progress-and-challenges-applications|04-applications]]
- 下一篇：[[raw/lessons/Multi-Agent-Survey/01-progress-and-challenges/06-survey-of-progress-and-challenges-references|06-references]]
