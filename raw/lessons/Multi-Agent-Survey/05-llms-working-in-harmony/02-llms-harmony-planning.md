---
type: lesson
tags: [多智能体, multi-agent, survey, 规划, AdaPlanner, ChatCoT, KnowAgent, RAP, Tree-of-Thoughts, ReAct, 论文翻译, 学习笔记]
created: 2026-05-15
updated: 2026-05-15
topic: LLM 多智能体综述中文精读（第 5 篇-02）规划框架
difficulty: intermediate
prerequisites:
  - [[raw/lessons/Multi-Agent-Survey/05-llms-working-in-harmony/00-llms-harmony-essence]]
  - [[raw/lessons/Multi-Agent-Survey/05-llms-working-in-harmony/01-llms-harmony-introduction-and-architecture]]
sources:
  - https://arxiv.org/abs/2504.01963
status: completed
series:
  name: "Multi-Agent Survey｜LLMs Working in Harmony（2025）"
  part: 2
paper_meta:
  paper_id: "multi-agent-survey-05"
  title: "LLMs Working in Harmony: A Survey on the Technological Aspects of Building Effective LLM-Based Multi Agent Systems"
  authors: "RM Aratchige, Dr. WMKS Ilmini"
  year: "2025"
  link: "https://arxiv.org/abs/2504.01963"
  section: "Section II-B"
---

# LLM 多智能体综述中文精读（第 5 篇-02）：规划框架

> 本篇覆盖 **Section II-B Planning**。核心任务是理解六种规划框架的设计哲学、优劣对比，以及为什么作者最终推荐 ReAct 作为首选方案。

## 本章导读

> [!note] 译者注（非原文）
> 规划（Planning）是 LLM agent 从"会说话"到"会做事"的关键一步。本章覆盖的六个框架可以分为三类：
> - **自适应规划**（AdaPlanner）：让 agent 能根据环境反馈动态调整计划
> - **知识增强规划**（ChatCoT、KnowAgent、RAP）：通过工具或知识库增强推理能力
> - **推理-行动整合**（ToT、ReAct）：重新思考推理和行动的关系
>
> 其中 ReAct 被作者选为最优方案，ToT 则在需要探索式搜索的场景中独树一帜。

## 正文翻译

> [!warning] 易混点（非原文）
> 从这里开始进入原文翻译。所有补充解释都会单独标成非原文。

### 原文标题：II-B Planning

在 LLM 多智能体系统中，规划涉及 agent 为实现目标而采用的推理和行动策略。这些系统必须在短期和长期推理之间取得平衡，同时响应环境反馈。作为自主决策者，LLM agent 基于初始目标生成行动序列，但通常需要自适应规划来应对现实问题的复杂性。

有效的 LLM 多智能体规划框架通常包含**反馈机制**，根据不断变化的环境条件来修正或重新校准行动，以避免幻觉和计划过度简化等问题。

> [!tip] 理解提示（非原文）
> 论文在这里建立了一个关键判断：规划 ≠ 一次性生成一个静态计划。好的规划框架必须能够根据环境的实时反馈进行调整。这个"闭环"特征是区分好规划框架和差规划框架的核心标准。

#### II-B.1 AdaPlanner

Sun 等人提出的 AdaPlanner 框架引入了一种新方法：让 LLM agent 能够**根据实时环境反馈来修改和调整计划**。这与传统的静态规划系统形成了鲜明对比——传统系统通常遵循固定的行动序列。

AdaPlanner 的闭环系统能够进行动态调整，为处理复杂的、长期的任务提供了关键的灵活性——在这些任务中，静态计划往往因意外变化而失败。

**两种优化策略：**
1. **计划内优化（in-plan refinement）**：agent 修改现有计划的特定部分以响应即时反馈
2. **计划外优化（out-of-plan refinement）**：当遇到意外情况时，创建新的行动方案

**其他关键机制：**
- **代码风格提示（code-style prompting）**：减少生成计划中的歧义，提升跨任务一致性
- **技能发现（skill discovery）**：允许 agent 将过去的成功计划作为 few-shot 示例复用于未来问题，提高适应性和效率

**局限：**
- 对更复杂任务依赖 few-shot 专家演示，未来需减少或消除这一依赖
- 虽然 ALFWorld 和 MiniWoB++ 环境中的表现令人振奋，但需在更多样化领域进一步测试

> [!tip] 理解提示（非原文）
> AdaPlanner 的核心贡献在于"闭环自适应"——让你的 agent 不是一次性把计划写完就执行到底，而是像人类一样边执行边调整。代码风格提示和技能发现是两个很实用的工程技巧：前者用代码形式约束计划输出以减少歧义，后者把成功经验存下来作为未来参考。

#### II-B.2 ChatCoT

Chen 等人提出的 ChatCoT 框架旨在改进 LLM 处理复杂多步推理任务的能力。它采用**工具增强的思维链（tool-augmented chain-of-thought）推理方法**，专为聊天式交互（如 ChatGPT）设计。

ChatCoT 允许 LLM 在工具操作和推理动作之间交替切换，形成迭代式的、逐步的推理过程，将工具使用与 CoT 推理无缝整合。

**实验结果：**
在 MATH 和 HotpotQA 等数据集上，ChatCoT 比现有方法实现了 **7.9% 的相对性能提升**。

**局限：**
- 尚未用 GPT-4 测试（因访问限制），可能影响发现的泛化性
- 针对聊天式 LLM 优化，可能限制与其他架构的兼容性
- 高计算需求（尤其是 GPU 资源）对广泛部署构成挑战

> [!tip] 理解提示（非原文）
> ChatCoT 的思路是把"使用工具"融入"思维链推理"中——不是在思考完之后才用工具，而是在思考过程中随时可以调用工具来验证或获取信息。7.9% 的提升虽然不算惊艳，但展示了工具增强推理的可行性方向。

#### II-B.3 KnowAgent

Zhu 等人提出的 KnowAgent 框架通过**整合行动知识库（action knowledge base）和自我学习策略**来增强 LLM 的规划能力。LLM 虽然强大，但常因缺乏内在的行动知识而在生成连贯行动序列和与环境交互时遇到困难。

KnowAgent 利用结构化的行动知识来优化规划路径，确保 LLM 生成更合理、更可执行的行动轨迹。它将复杂的行动数据转换为 LLM 可理解的格式，显著提高了规划准确性。

**实验结果：**
在 HotpotQA 和 ALFWorld 等数据集上，KnowAgent 不仅匹敌甚至超越了当前最先进的性能，同时有效减少了规划幻觉。

**局限：**
- 主要在常识问答和家庭任务上评估，未来可扩展至医疗推理、算术、网页浏览等领域
- 目前仅支持单 agent 应用，探索多智能体系统可进一步增强其效用
- 行动知识库的手动设计是劳动密集型的挑战，需自动化解决方案

> [!tip] 理解提示（非原文）
> KnowAgent 解决了一个很根本的问题：LLM 本身没有关于"在环境中可以执行哪些动作"的结构化知识。它想出来的"计划"可能包含不合理甚至不可能的动作。KnowAgent 的做法是显式告诉 agent"在什么情况下，你能做哪些事"，这本质上是一种"约束空间内的规划"。

#### II-B.4 RAP: Retrieval Augmented Processing

RAP 引入了一个突破性的框架：通过将**检索增强技术与上下文记忆**相结合来增强 LLM 的规划能力。随着 LLM 越来越多地被用作机器人和游戏领域的自主 agent，如何将过去经验融入当前决策仍是一个重大挑战。

RAP 的设计亮点是**多功能性**——它能在纯文本和多模态环境中无缝运行。它允许 agent 动态利用与当前上下文相关的过往经验来指导后续行动。

**实验结果：**
在文本场景中取得了最先进性能，在多模态具身任务中显著增强了 LLM agent 的能力。

> [!tip] 理解提示（非原文）
> RAP 的核心思想是"让 agent 有记忆，并在做规划时主动检索相关经验"。这和 AdaPlanner 的"技能发现"有相似之处，但 RAP 更系统化——它把经验检索作为规划流程中内置的一环，而不是事后可选的功能。类似于 RAG 对语言生成的作用，RAP 是对 agent 规划的"知识增强"。

#### II-B.5 Tree of Thoughts (ToT)

Tree of Thoughts (ToT) 框架为语言模型的推理能力引入了全新的范式。传统 LM 在推理时采用 token 级别、从左到右的决策方式，这在需要探索、策略性前瞻和重视初始决策的任务中表现不佳。

ToT 拓展了流行的 Chain of Thought 提示技术，允许 LM 探索被称为"thoughts"（思维）的连贯文本单元，作为问题解决的中间步骤。ToT 让 LM 能够通过**评估多条推理路径并做出选择**来进行精心决策，同时具备前瞻或回溯能力。

**实验突破：**
- 在 Game of 24 任务中：GPT-4 + CoT 仅 4% 成功率；ToT 将其提升至 **74%**
- 在 Creative Writing 和 Mini Crosswords 任务上也表现显著提升

**局限：**
- 对 GPT-4 已经表现良好的许多任务来说，搜索方法可能不是必需的
- 初步探索限于三个相对简单的任务，需在编码、数据分析和机器人等更复杂场景测试
- 搜索方法相比采样方法资源密集度更高，但 ToT 的模块化灵活性允许用户定制性能-成本权衡

> [!tip] 理解提示（非原文）
> ToT 是本章中思路最独特的一个框架。它把推理问题变成了**树搜索问题**——不再是从头到尾线性思考，而是在每个决策点探索多条路径，评估、剪枝、回溯。4% → 74% 的跳跃极为惊人。但它的代价也很明显：计算成本高，且很多常规任务用不着这么复杂的搜索。

#### II-B.6 ReAct

ReAct 框架代表了 LLM 中**推理与行动整合**的重大进步。虽然 LLM 在语言理解和交互决策方面展现出卓越能力，但推理（如 chain-of-thought）和行动（如 action planning）的传统分离限制了它们在复杂任务中的有效性。

ReAct 通过**交替生成推理轨迹和任务特定行动**来解决这一限制，增强了两者之间的协同效应：

- **推理轨迹**帮助模型归纳、追踪和更新行动计划，同时管理异常
- **行动**使模型能够与外部知识库或环境进行交互

**实验表现：**
在问答（HotpotQA）和事实验证（Fever）等任务中，ReAct 有效缓解了 chain-of-thought 推理中常见的幻觉问题和错误传播——通过调用简单的 Wikipedia API 进行外部交互。这产生了更加人性化的、可追溯的任务解决轨迹。

**局限：**
- 虽然其简洁性是优势，但也意味着需要更多演示才能有效学习大动作空间中的复杂任务
- 初步实验表明，在特定任务上微调（如 HotpotQA）可进一步提升性能——特别是通过引入高质量人工标注
- 与强化学习等互补范式的整合有望增强 agent 鲁棒性

> [!tip] 理解提示（非原文）
> ReAct 是作者明确定义为"最推荐"的规划框架。它的核心洞见是：推理和行动不应该分两步做，而应该交替进行——想一步、做一步、看结果、再想下一步。这很像人类解决复杂问题时的"边想边做"模式。而且因为推理轨迹是可见的，整个决策过程可解释、可追踪、可调试。

## 规划框架对比总览（Table II 翻译）

| 框架 | 关键特征 | 优势 | 局限 |
|------|---------|------|------|
| AdaPlanner | 根据实时反馈动态调整计划；计划内/计划外优化；代码风格提示减少歧义 | 在复杂长周期任务中灵活性强；通过技能发现提高适应性 | 依赖 few-shot 专家演示；需在更多领域测试 |
| ChatCoT | 工具增强思维链推理；迭代式多轮对话 | 复杂任务相对提升 7.9%；增强推理适应性 | 未用 GPT-4 测试；高计算需求；针对聊天式模型优化 |
| KnowAgent | 整合行动知识库与自学习；优化规划路径形成连贯行动序列 | 匹敌或超越 SOTA 性能；有效减少规划幻觉 | 评估任务有限；仅支持单 agent；行动知识库手动设计工作量大 |
| RAP | 检索增强 + 上下文记忆；文本和多模态环境通用 | 多项基准测试 SOTA；利用过往经验增强决策 | 多模态任务适配需仔细实现；检索过程管理复杂 |
| ToT | 探索多条推理路径；评估选择 + 前瞻回溯；将推理化为树搜索 | Game of 24 成功率 4%→74%；模块化灵活 | 搜索方法资源密集；对简单任务非必需 |
| ReAct | 推理与行动交替生成；增强可解释性和可信度 | 多项任务性能优越；有效缓解幻觉和错误传播；**作者首选推荐** | 简洁性可能限制复杂场景处理；大动作空间需更多演示 |

## 本章小结

> [!note] 译者注（非原文）
> 本章建立了以下关键判断：
> 1. 好的规划框架必须支持反馈闭环——静态计划无法应对真实世界的动态变化。
> 2. 六个规划框架可以分为三族：自适应族（AdaPlanner）、知识增强族（ChatCoT、KnowAgent、RAP）、推理-行动整合族（ToT、ReAct）。
> 3. ReAct 被作者选为最推荐方案，原因有三：简洁、可解释、幻觉控制好。
> 4. ToT 在需要探索式搜索的场景中独一无二，但成本高、适用面窄。
> 5. 知识增强（RAP、KnowAgent）和推理-行动整合（ReAct）不是互斥的，未来方向可能是两者的融合。

## 术语对照

| 英文术语 | 中文译法 | 本章语境说明 |
|----------|---------|-------------|
| planning hallucination | 规划幻觉 | agent 生成不可执行或不合理的行动计划 |
| in-plan refinement | 计划内优化 | 在现有计划框架内微调特定步骤 |
| out-of-plan refinement | 计划外优化 | 创建全新行动方案应对意外 |
| skill discovery | 技能发现 | 将过去的成功计划复用为 few-shot 示例 |
| code-style prompting | 代码风格提示 | 使用代码形式约束 LLM 输出减少歧义 |
| action knowledge base | 行动知识库 | 结构化描述 agent 可执行行动的知识库 |
| reasoning trace | 推理轨迹 | agent 逐步推理过程的可视化记录 |
| retrieval-augmented planning | 检索增强规划 | 利用过往经验辅助当前规划决策 |

## 思考题

### 题目 1：为什么 ReAct 的"推理-行动交替"比"先推理再行动"更有效？

> [!tip] 理解提示（非原文）
> 想一想：如果你先写一个完整的计划再去执行，中途发现计划有误怎么办？

> [!note] 译者注（非原文）
> "先推理再行动"的分离模式面临两个致命问题：一是推理时缺少实际执行反馈，可能生成不切实际的计划；二是执行中遇到意外时，没有机制来修正后续计划。ReAct 的交替模式解决了这两个问题：每一步行动后的观察结果直接被纳入下一步推理，形成持续的计划修正循环。这也解释了为什么 ReAct 能有效减少幻觉——因为外部知识源的反馈被实时注入推理过程。

### 题目 2：ToT 的树搜索和 Agent Forest 的多数投票，有什么本质区别？

> [!tip] 理解提示（非原文）
> 树搜索和投票都是在"多选一"，但它们的决策机制完全不同。

> [!note] 译者注（非原文）
> Agent Forest 的多数投票是"事后聚合"：多个 agent 各自独立生成完整回答，然后统计哪个回答最受欢迎。ToT 的树搜索是"过程探索"：在每个决策节点，agent 主动生成多个候选"思维"、评估每个候选的质量、选择最有希望的路径继续探索。区别在于：投票是横向的、一层的、无反馈的；树搜索是纵向的、多层的、带评估反馈的。后者更适合需要"几步之后才能看出好坏"的策略性推理任务。

## 相关页面

- [[raw/lessons/Multi-Agent-Survey/05-llms-working-in-harmony/00-llms-harmony-essence]]
- [[raw/lessons/Multi-Agent-Survey/05-llms-working-in-harmony/01-llms-harmony-introduction-and-architecture]]
- [[raw/lessons/Multi-Agent-Survey/05-llms-working-in-harmony/03-llms-harmony-memory]]

## 下一步学习

- 上一篇：[[raw/lessons/Multi-Agent-Survey/05-llms-working-in-harmony/01-llms-harmony-introduction-and-architecture\|01-introduction-and-architecture]]
- 下一篇：[[raw/lessons/Multi-Agent-Survey/05-llms-working-in-harmony/03-llms-harmony-memory\|03-memory]]
