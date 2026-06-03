---
type: lesson
tags: [多智能体, multi-agent, survey, 创造力, agent人格, persona, 角色设计, 论文翻译, 学习笔记]
created: 2026-05-15
updated: 2026-05-15
topic: LLM 多智能体综述中文精读（第 6 篇-03）Persona 与 Agent 画像
difficulty: intermediate
prerequisites:
  - [[raw/lessons/Multi-Agent-Survey/06-creativity/02-creativity-mas-techniques]]
sources:
  - https://aclanthology.org/2025.emnlp-main.1403/
  - "DOI: 10.18653/v1/2025.emnlp-main.1403"
status: completed
series:
  name: "Multi-Agent Survey｜Creativity in LLM-based MAS（2025）"
  part: 3
paper_meta:
  paper_id: "multi-agent-survey-06"
  title: "Creativity in LLM-based Multi-Agent Systems: A Survey"
  authors: "Yi-Cheng Lin, Kang-Chieh Chen, Zhe-Yan Li, Tzu-Heng Wu, Tzu-Hsuan Wu, Kuan-Yu Chen, Hung-yi Lee, Yun-Nung Chen"
  year: "2025"
  venue: "EMNLP 2025"
  link: "https://aclanthology.org/2025.emnlp-main.1403/"
  section: "Section 4"
---

# LLM 多智能体综述中文精读（第 6 篇-03）：Persona 与 Agent 画像

> 本篇覆盖 **Section 4: Persona and Agent Profile**。核心任务是理解：agent 的"人设"（persona）是如何设计的？不同粒度的 persona 如何影响创造力输出？以及 persona 有哪些生成方式？

## 本章导读

> [!note] 译者注（非原文）
> 在创造力 MAS 中，persona 可能是最被低估但实际上影响最大的维度。想想看：两个"诗人 agent"，一个是"浪漫派"、一个是"现代派"，它们产出的诗必然截然不同。Persona 决定了 agent 的创作风格、审美偏好、甚至"想问题的角度"。
>
> 作者在这一章做了两件事：第一，建立 persona 粒度的三层分类（从粗到细）；第二，整理了 persona 的三种生成方式。这个框架不仅对研究有用，对实际构建创造力 MAS 的工程师也直接可操作。

## 正文翻译

> [!warning] 易混点（非原文）
> 从这里开始进入原文翻译。所有补充解释都会单独标成非原文。

### 原文标题：4 Persona and Agent Profile

作者指出，在创造力 MAS 中，agent 的 persona（人格/角色画像）是一个核心设计维度。Persona 直接决定了 agent 的"创意风格"——同样的任务，具有不同 persona 的 agent 会给出截然不同的输出。因此，理解 persona 设计的原则和方法，对于构建有效的创造力 MAS 至关重要。

#### 4.1 Persona 粒度的三层分类

作者将 persona 设计按粒度（granularity）从粗到细分为三个层次：

##### 4.1.1 Coarse-Grained Persona（粗粒度人格）

**定义**：粗粒度人格只定义了 agent 的最基本角色标识——通常是一个大类标签，如"诗人""小说家""广告文案写手""音乐评论家"等。

**特征**：
- 信息量极少，通常只用一到两个词描述；
- 不同 agent 之间的差异性大但内部同质化程度高；
- 实现简单，一个 prompt 即可定义。

**典型使用场景**：
- 需要快速区分 agent 的基本功能；
- agent 数量多、无需精细角色区分时；
- 作为其他设计元素的附带属性，而非核心变量。

**局限性**：
- 创意输出的风格局限于大类特征，缺乏个性化差异；
- 同一类角色下的 agent 输出容易同质化；
- 不适合需要微妙风格差异的创意任务。

##### 4.1.2 Medium-Coarse-Grained Persona（中等粗粒度人格）

**定义**：中等粗粒度人格在基本角色标签之外，增加了若干具体属性，如创作风格偏好、知识背景、审美倾向等。

**特征**：
- 包含 3-10 个关键属性来描述一个 agent；
- 不同 agent 之间既有大类差异，也有属性层面的微调；
- 需要在 prompt 或系统配置中进行中等复杂度的定义。

**典型描述示例**：
- "一位受海子影响的乡土诗人，擅长用朴素语言表达深刻情感"
- "一位偏好极简主义的广告文案写手，信仰'少即是多'的设计哲学"
- "一位受过古典音乐训练的爵士钢琴家，喜欢在不和谐音中寻找新的和声可能"

**使用场景**：
- 需要 agent 之间有可辨识但可控的风格差异；
- 创意任务本身对风格偏好有特定要求；
- 希望模拟真实创作群体中的专家多样性。

**优势**：
- 在丰富度与可控性之间取得良好平衡；
- 属性可组合，产生有意义的 agent 间差异；
- 与人类创作团队中的"个人风格"概念自然对应。

##### 4.1.3 Fine-Grained Persona（细粒度人格）

**定义**：细粒度人格是对 agent 进行高度详细和个性化的人设定义，可能包含完整的背景故事、成长经历、价值观体系、创作哲学、语言习惯等深度信息。

**特征**：
- 信息丰富，可能包含数页的描述；
- 每个 agent 都是高度独特、立体的"人格"；
- 通常需要从真实人物数据中提取或通过复杂建模生成。

**典型描述示例**：
不仅包含基本角色和风格偏好，还包括：
- 完整的生平背景（出生地、教育经历、关键人生事件）；
- 核心价值观和创作信念；
- 特定的语言习惯和表达模式；
- 情感倾向和情绪反应模式；
- 与其他风格/流派的关系定位。

**使用场景**：
- 社会模拟类应用（需要高度真实的 agent 行为）；
- 创意写作中的角色扮演和叙事生成；
- 研究人格与创造力之间关系的学术场景。

**局限性与挑战**：
- 设计成本极高——手工创建需要大量专业知识；
- 自动生成的质量难以保证一致性；
- 过于详细的 persona 可能限制而非增强创造力；
- 评估细粒度 persona 的效果缺乏统一标准。

> [!tip] 理解提示（非原文）
> 三层粒度的选择本质上是一个"丰富度 vs. 可控性"的权衡。粗粒度简单易用但同质化严重；细粒度丰富但成本高且可能过拟合；中等粗粒度是实践中最常见的折中选择。这个框架的价值在于：给你一个语言来描述和比较不同系统中 agent 的"人格分辨率"。

#### 4.2 Persona 的三种生成/定义方式

作者进一步整理了 persona 的三种来源方式：

##### 4.2.1 Human-Defined Persona（人工定义）

**定义**：由人类设计师手动编写 agent 的 persona 描述。

**实现方式**：
- 在 system prompt 中直接描述；
- 通过角色卡片（character card）或 persona 文件定义；
- 使用结构化的 persona 模板（template-based）。

**优势**：
- 完全可控——设计师可以精确控制 agent 的行为边界；
- 可解释性强——每个 persona 特征都有明确的设计意图；
- 便于对齐——可以根据任务需求精确调整；
- 易于迭代——设计师可以快速测试和修改 persona 参数。

**局限**：
- 人力成本高——大规模 MAS 中需要定义大量 persona；
- 受限于设计师的想象力和领域知识；
- 可能存在无意识的偏见——设计师的文化背景会影响 persona 设计；
- 一致性难保证——不同设计师之间存在风格差异。

##### 4.2.2 Model-Generated Persona（模型生成）

**定义**：使用 LLM 自身来自动生成 agent 的 persona 描述。

**实现方式**：
- 给定高层描述（如"生成了一个浪漫派诗人的人格"），让 LLM 自动扩展为完整的 persona 描述；
- 基于特定风格/流派关键词，让 LLM 生成对应的 persona；
- 通过 few-shot 示例引导 LLM 生成特定类型的人格。

**优势**：
- 高效——可以快速生成大量 persona；
- 多样性——LLM 可以从训练数据中调取丰富的人格模式；
- 可扩展——适合大规模 MAS 或需要频繁更换 persona 的场景；
- 降低人力门槛——非领域专家也能生成相对合理的 persona。

**局限**：
- 质量不稳定——生成的 persona 可能包含矛盾、重复或不合理的设定；
- 风格趋同——同一 LLM 生成的 persona 可能带有隐式的"模型人格"；
- 文化偏差——模型训练数据中的文化偏见会被带入 persona 生成；
- 缺乏深度——自动生成的 persona 通常停留在表面特征，缺乏细粒度人格所需的深度。

> [!warning] 易混点（非原文）
> 模型生成的 persona 和 agent 本身的"模型能力"是两回事。前者是系统为 agent 设计的人设描述（"这个 agent 应该是什么样的人"），后者是 agent 作为 LLM 实例的基础能力。人设描述是输入的一部分，不改变底层模型能力。

##### 4.2.3 Data-Derived Persona（数据提取）

**定义**：从真实世界的数据中提取和构建 agent 的 persona。

**数据来源**：
- 真实人物的公开资料（作家、艺术家、音乐家等的访谈、作品、传记）；
- 社交媒体数据（特定人群的语言模式、兴趣偏好、表达风格）；
- 文学/艺术作品中的人物设定；
- 用户研究数据（通过调查、访谈等方式收集的真实人格数据）。

**实现方式**：
- 分析真实人物的创作语料，提取其语言风格、主题偏好、修辞特征；
- 从大量社交媒体文本中学习特定人群的语言模式；
- 通过 clustering 等方法从数据中自动发现不同的人格类型。

**优势**：
- 真实性——基于真实数据，persona 通常更立体、更少套路化；
- 多样性——真实世界天然包含丰富的人格多样性；
- 研究价值——可以用于验证人格与创造力关系的假设；
- 可复现——基于公开数据构建的 persona 可以被其他研究者复现。

**局限**：
- 数据获取困难——高质量的真实人格数据不容易获得；
- 隐私和伦理问题——从真实人物数据提取 persona 涉及隐私和知情同意问题；
- 数据偏见——来源数据的偏差会被带入 persona；
- 迁移难度——从一种创作形式（如诗歌）提取的 persona 不一定适合另一种形式（如广告文案）。

#### 4.3 Persona 对创造力的影响机制

作者讨论了 persona 如何影响 agent 的创造力输出，提出几个关键机制：

**机制一：风格约束引导创意方向**

Persona 通过设定风格边界，让 agent 的创意探索在一个有意义的方向上进行，而不是随机发散。例如，一个"极简主义诗人"的 persona 会引导 agent 在简约、意象密集的方向上探索，这比完全开放的"写一首诗"更有方向性。

**机制二：Persona 多样性促进整体创意多样性**

当系统中不同 agent 具有不同的 persona 时，整个系统的创意多样性自然会增加。这不仅是量的增加，更是质的多元化——不同 persona 的 agent 会在不同的"创意子空间"中探索。

**机制三：Persona 冲突产生创造性张力**

当具有对立或互补 persona 的 agent 进行交互时（如"保守派评论家"与"前卫艺术家"），persona 之间的张力本身可以成为创意的催化剂。这种张力迫使 agent 更深入地思考和辩护自己的创意选择。

**机制四：Persona 的元认知功能**

Persona 不仅影响 agent "做什么"，还影响 agent "怎么想"。一个具有"自我反思型评论家" persona 的 agent，不仅会评价他人的作品，还会评价自己的评价标准——这种元认知能力在创造力任务中非常重要。

> [!note] 译者注（非原文）
> Persona 这一章在整篇综述中起到"承上启下"的作用：它解释了第 2 章中的"工作流"中 agent 角色的来源，也为第 5 章的"评估"提供了评估对象的分析基础。理解 persona 设计，才能真正理解为什么不同创造力 MAS 的表现会有差异。

## 术语对照

| 英文术语 | 中文译法 | 本章语境说明 |
|---------|---------|------------|
| persona | 人格/角色画像 | agent 的人设定义，决定其创作风格 |
| granularity | 粒度 | persona 描述的精细程度 |
| coarse-grained | 粗粒度 | 仅定义大类角色的 persona |
| medium-coarse-grained | 中等粗粒度 | 包含若干关键属性的 persona |
| fine-grained | 细粒度 | 高度详细和个性化的 persona |
| human-defined | 人工定义 | 由人类设计师手动编写的 persona |
| model-generated | 模型生成 | 由 LLM 自动生成的 persona |
| data-derived | 数据提取 | 从真实世界数据提取的 persona |
| creative tension | 创造性张力 | persona 冲突激发的创意动力 |

## 思考题

### 题目 1：为什么细粒度人格不一定优于粗粒度人格？

> [!tip] 理解提示（非原文）
> 想想"信息越多越好"在创造力场景中为什么可能不成立？

> [!success]- 参考答案（非原文）
> 过于详细的 persona 可能产生"过度约束"效应——agent 的行为被过多的具体限定所束缚，失去了创意所需的灵活性和探索空间。此外，细粒度 persona 的设计成本和维护成本远高于粗粒度 persona，而边际收益可能递减。在某些场景中，一个简单的"浪漫派诗人"标签可能比一份三页的角色传记更有效地激发有意义的创意。

### 题目 2：三种 persona 生成方式能否组合使用？如果能，什么场景下适合组合？

> [!tip] 理解提示（非原文）
> 想想哪种方式适合"广度"（生成大量 persona），哪种适合"深度"（打造高质量 persona）。

> [!success]- 参考答案（非原文）
> 可以组合。一个典型的组合策略是：（1）用模型生成快速产生大量候选 persona（广度）；（2）用人工定义的方式精选和调整其中最有潜力的 persona（深度）；（3）如果可用，从真实数据中提取特征来增强 persona 的真实性（真实性）。这种组合适合需要同时满足"大规模"和"高质量"的场景，如构建用于研究人格-创造力关系的实验平台。

## 相关页面

- [[raw/lessons/Multi-Agent-Survey/06-creativity/02-creativity-mas-techniques]]
- [[raw/lessons/Multi-Agent-Survey/06-creativity/04-creativity-mas-evaluation]]
- [[raw/lessons/Multi-Agent-Survey/06-creativity/00-creativity-mas-essence]]

## 下一步学习

- 上一篇：[[raw/lessons/Multi-Agent-Survey/06-creativity/02-creativity-mas-techniques|02-techniques]]
- 下一篇：[[raw/lessons/Multi-Agent-Survey/06-creativity/04-creativity-mas-evaluation|04-evaluation]]
