---
type: lesson
tags: [多智能体, multi-agent, survey, 创造力, 评估方法, 客观指标, 主观评估, 论文翻译, 学习笔记]
created: 2026-05-15
updated: 2026-05-15
topic: LLM 多智能体综述中文精读（第 6 篇-04）创造力评估
difficulty: intermediate
prerequisites:
  - [[raw/lessons/Multi-Agent-Survey/06-creativity/03-creativity-mas-persona]]
sources:
  - https://aclanthology.org/2025.emnlp-main.1403/
  - "DOI: 10.18653/v1/2025.emnlp-main.1403"
status: completed
series:
  name: "Multi-Agent Survey｜Creativity in LLM-based MAS（2025）"
  part: 4
paper_meta:
  paper_id: "multi-agent-survey-06"
  title: "Creativity in LLM-based Multi-Agent Systems: A Survey"
  authors: "Yi-Cheng Lin, Kang-Chieh Chen, Zhe-Yan Li, Tzu-Heng Wu, Tzu-Hsuan Wu, Kuan-Yu Chen, Hung-yi Lee, Yun-Nung Chen"
  year: "2025"
  venue: "EMNLP 2025"
  link: "https://aclanthology.org/2025.emnlp-main.1403/"
  section: "Section 5"
---

# LLM 多智能体综述中文精读（第 6 篇-04）：创造力评估

> 本篇覆盖 **Section 5: Evaluation**。核心任务是理解：多智能体系统的创造力怎么衡量？有哪些客观指标和主观评估方法？以及为什么交互评估是创造力 MAS 特有的评估维度？

## 本章导读

> [!note] 译者注（非原文）
> 评估是创造力 MAS 最"硬核"但也最"棘手"的一章。说它硬核，因为作者系统整理了多个可量化的客观指标；说它棘手，因为创造力本身就是一个高度主观的概念——什么叫"更有创造力"本身就没有统一答案。
>
> 这一章的核心价值在于：作者没有回避这个困难，而是给出了一个三维评估框架——客观指标、主观评估、交互评估——让研究者至少知道该从哪里入手。特别是作者将经典创造力测试（TTCT）的维度改编到 MAS 评估中，这是一个很有启发性的思路。

## 正文翻译

> [!warning] 易混点（非原文）
> 从这里开始进入原文翻译。所有补充解释都会单独标成非原文。

### 原文标题：5 Evaluation

作者在开篇就指出，创造力评估是 MAS 创造力研究中最具挑战性的问题之一。与分类准确率、BLEU 分数等有明确 ground truth 的传统 NLP 指标不同，创造力缺乏单一的、广泛接受的客观度量标准。因此，作者提出了一个三维评估框架，涵盖客观测量、主观评估和交互评估。

#### 5.1 Objective Measurements（客观指标）

客观指标的优势在于可自动化计算、可复现、可大规模使用。作者将现有的客观指标分为以下几类：

##### 5.1.1 多样性指标（Diversity Metrics）

多样性是创造力的基础——如果所有输出看起来都一样，那就谈不上创造力。作者整理了以下多样性指标：

**（1）Distinct-n**

Distinct-n 计算生成的 n-gram 中不重复比例。例如 Distinct-2 = 不重复的 bigram 数 / 总 bigram 数。Distinct-n 值越高，说明用词越多样化。

- 公式：Distinct-n = |unique n-grams| / |total n-grams|
- 适用场景：评估文本生成中的词汇多样性
- 优点：计算简单，直观易懂
- 局限：只看词汇层面，不看语义层面。两段话可能用完全不同的词汇表达相同的平庸内容。

**（2）Entropy-n**

Entropy-n 计算 n-gram 分布的熵值。熵越高，n-gram 分布越均匀（即多样性越高）。

- 公式：Entropy-n = -Sum(p(ngram) * log(p(ngram)))
- 适用场景：评估生成的 n-gram 分布的均匀程度
- 优点：考虑了分布的概率特性，比 Distinct-n 更精细
- 局限：同样只关注词汇层面

##### 5.1.2 自相似性指标（Self-Similarity Metrics）

通过衡量同一系统不同输出之间的相似度来间接度量多样性——相似度越低，多样性越高。

**（3）Self-BLEU**

Self-BLEU 将一个生成输出视为"参考"，用另一个生成输出作为"候选"计算 BLEU。Self-BLEU 越低，说明输出之间越不相似——即多样性越高。

- 原理：对一个系统的 N 个输出，计算所有配对的 BLEU，取平均
- 适用场景：评估生成文本集合的内部多样性
- 优点：基于成熟的 BLEU 计算框架，广泛使用
- 局限：BLEU 本身偏重 n-gram 精确匹配，可能低估语义相似但用词不同的情况

**（4）SBERT Similarity（基于 SBERT 的语义相似度）**

使用 Sentence-BERT 计算输出之间的语义相似度。与 Self-BLEU 相比，SBERT 能捕捉语义层面的相似性（而非仅词汇层面）。

- 原理：用 SBERT 将每个输出编码为向量，计算成对余弦相似度
- 适用场景：评估语义层面的多样性
- 优点：比 n-gram 方法更能反映真正的语义差异
- 局限：依赖于 SBERT 模型的训练数据和领域匹配度

##### 5.1.3 生成质量指标（Generation Quality Metrics）

除了多样性，还需要评估生成质量。作者整理了以下常用于创造力评估的质量指标：

**（5）FID（Frechet Inception Distance）**

FID 最初用于评估生成图像的质量和多样性。在文本领域，可以用文本嵌入替代图像嵌入来计算 FID。

- 原理：比较生成样本分布与参考分布（通常是高质量人类创作）之间的 Frechet 距离
- 适用场景：评估生成文本的整体质量和多样性
- 优点：同时考虑质量和多样性
- 局限：需要有高质量的参考分布；在文本领域的使用不如图像领域成熟

**（6）TIE（Text Inception Embedding）**

类似于 FID 但专为文本设计，使用文本嵌入空间中的分布距离来衡量生成质量。

- 适用场景：文本生成质量的整体评估
- 优点：专为文本设计
- 局限：仍处于研究早期，标准化程度不高

> [!tip] 理解提示（非原文）
> 这些客观指标的一个共同局限是：它们衡量的是"统计特征"（多样性、分布距离），而不是"创意质量"。一个系统完全可能生成大量语法正确、词汇多样、但毫无新意的文本，并在这些指标上获得高分。因此，客观指标必须与主观评估结合使用。

#### 5.2 Subjective Assessments（主观评估）

客观指标无法完全替代人类对创造力的判断，因此作者介绍了主观评估方法，特别强调了改编自经典创造力心理学测试的评估维度。

##### 5.2.1 TTCT 改编框架

作者将 Torrance Tests of Creative Thinking（TTCT，托兰斯创造力思维测试）的维度改编到 MAS 创造力评估中。TTCT 是心理学领域最广泛使用的创造力测试之一。改编后的四个核心维度：

**（1）Fluency（流畅性）**

定义：生成大量想法的能力。在 MAS 评估中，流畅性衡量的是系统在给定任务中能产生多少个不同的、相关的创意输出。

- 评估方式：计数有效输出的数量
- 重要性：流畅性是创造力的基础——必须先"有想法"，才能谈"想法好不好"

**（2）Flexibility（灵活性）**

定义：从不同角度、不同类别思考问题的能力。在 MAS 评估中，灵活性衡量的是生成输出在不同维度（如主题、风格、方法）上的多样性。

- 评估方式：对输出进行类别标注，计算覆盖的类别数量
- 重要性：灵活性反映了系统是否有能力跳出固定思维模式

**（3）Originality（原创性）**

定义：产生独特、不常见想法的能力。在 MAS 评估中，原创性衡量的是生成输出与常见输出的差异程度。

- 评估方式：通过统计稀有度（相对于参考语料库）或人工评分的统计稀有度
- 重要性：原创性是创造力的核心——没有原创性，就没有真正的"创造"

**（4）Elaboration（精细度）**

定义：在基本想法之上添加细节、丰富内容的能力。在 MAS 评估中，精细度衡量的是输出中细节的丰富程度和思考的深度。

- 评估方式：评估输出中包含的细节数量和质量（如具体的意象、精确的描述、逻辑的层次）
- 重要性：精细度区分了"一个有潜力的粗略想法"和"一个经过充分发展的创意作品"

##### 5.2.2 人工评估的实施方式

作者讨论了两种主要的人工评估实施方式：

**（1）专家评估（Expert Evaluation）**

由领域专家（如诗人、文学评论家、广告创意总监）根据上述维度对 MAS 输出进行评分。

- 优势：专业判断更准确，能捕捉细微的创意质量差异
- 局限：成本高、难以规模化、专家之间可能存在分歧

**（2）众包评估（Crowdsourced Evaluation）**

通过众包平台（如 Amazon Mechanical Turk）招募大量评估者进行评分。

- 优势：成本相对较低、可规模化、统计效力更高
- 局限：评估者质量参差不齐、需要精心设计质量控制机制

##### 5.2.3 主观评估的主要挑战

作者总结了主观评估面临的几个关键挑战：

- **评分者一致性（Inter-rater Reliability）**：不同评估者对"什么是好的创意"可能有截然不同的标准；
- **文化偏见（Cultural Bias）**：评估者的文化背景会影响其对创造力的判断；
- **顺序效应（Order Effects）**：评估顺序可能影响评分（如前面的输出被记住得更清楚）；
- **疲劳效应（Fatigue Effects）**：长时间评估后，评估者的判断标准可能漂移。

#### 5.3 Interaction Evaluation（交互评估）

作者指出，创造力 MAS 的评估还需要考虑一个传统单 agent 系统不需要考虑的维度：**交互质量**。因为创造力 MAS 的产出不仅依赖于单个 agent 的能力，还依赖于 agent 之间的交互过程。

##### 5.3.1 交互评估的维度

**（1）交互效率（Interaction Efficiency）**

衡量 agent 之间的信息交换是否高效。包括：
- 交互轮次（turns）：达到目标输出所需的交互轮次数；
- 信息密度（information density）：每轮交互中传递的有效信息量；
- 冗余度（redundancy）：交互中重复或无用的信息比例。

**（2）交互质量（Interaction Quality）**

衡量 agent 之间交互的"创意价值"。包括：
- 观点碰撞度（perspective collision）：交互是否产生了不同观点的有意义的碰撞；
- 协同进化度（co-evolution）：agent 的输出是否在交互中逐渐提升（而非原地打转）；
- 互补性（complementarity）：不同 agent 的贡献是否形成了有效的互补。

**（3）交互鲁棒性（Interaction Robustness）**

衡量交互过程在面对不良输入或异常情况时的稳定性。包括：
- 冲突处理能力：当 agent 之间出现意见分歧时，系统是否能有效解决；
- 错误恢复能力：当某个 agent 产生低质量输出时，系统是否能自行纠正；
- 循环避免能力：系统是否能避免陷入无意义的重复交互循环。

##### 5.3.2 交互评估的方法

作者指出，交互评估仍处于早期阶段，目前主要有以下几种方法：

- **人工观察与评分**：人类评估者观察 agent 之间的完整交互日志并评分；
- **自动化代理指标**：使用交互轮次、token 消耗量等作为交互效率的代理指标；
- **对比实验**：对比不同交互设计（如不同 communication topology、不同 role assignment）下的系统输出质量。

> [!tip] 理解提示（非原文）
> 交互评估是这篇文章最有前瞻性的部分之一。大多数现有工作只关注最终输出的质量，但作者提醒我们：**一个好的创意 MAS 的输出质量，是交互过程的产物**。如果不评估交互过程本身，就无法真正理解为什么某些系统比另一些系统更有创造力。

#### 5.4 评估框架的整体结构

作者将上述三个维度整合为一个完整的评估框架：

```
创造力 MAS 评估
├── 客观指标（Objective）
│   ├── 多样性：Distinct-n, Entropy-n
│   ├── 自相似性：Self-BLEU, SBERT Similarity
│   └── 生成质量：FID, TIE
├── 主观评估（Subjective）
│   ├── Fluency（流畅性）
│   ├── Flexibility（灵活性）
│   ├── Originality（原创性）
│   └── Elaboration（精细度）
└── 交互评估（Interaction）
    ├── 交互效率
    ├── 交互质量
    └── 交互鲁棒性
```

> [!note] 译者注（非原文）
> 这个三维评估框架是这篇综述对领域最重要的方法论贡献之一。它清晰地区分了"输出层面的衡量"（客观+主观）和"过程层面的衡量"（交互），为后续研究提供了一个可操作的评估蓝图。

## 术语对照

| 英文术语 | 中文译法 | 本章语境说明 |
|---------|---------|------------|
| objective measurement | 客观指标 | 可自动化计算的量化指标 |
| subjective assessment | 主观评估 | 由人类评估者进行的创造力判断 |
| interaction evaluation | 交互评估 | 评估 agent 之间交互过程的质量 |
| Distinct-n | Distinct-n | n-gram 不重复比例（多样性指标） |
| Entropy-n | Entropy-n | n-gram 分布的熵值 |
| Self-BLEU | Self-BLEU | 同一系统不同输出间的 BLEU 相似度 |
| SBERT similarity | SBERT 相似度 | 基于 Sentence-BERT 的语义相似度 |
| FID | Frechet Inception Distance | 生成分布与参考分布的距离 |
| TIE | Text Inception Embedding | 文本嵌入的分布距离 |
| TTCT | 托兰斯创造力思维测试 | 经典创造力心理学测试 |
| fluency | 流畅性 | 生成大量想法的能力 |
| flexibility | 灵活性 | 从不同角度思考的能力 |
| originality | 原创性 | 产生独特想法的能力 |
| elaboration | 精细度 | 添加细节和丰富内容的能力 |
| inter-rater reliability | 评分者一致性 | 不同评估者的评分一致程度 |

## 思考题

### 题目 1：为什么多样性指标高不等于创造力高？

> [!tip] 理解提示（非原文）
> 假设一个系统每次都输出随机采样的词汇——多样性一定很高，但它有创造力吗？

> [!success]- 参考答案（非原文）
> 多样性指标只衡量"有多不同"，不衡量"有多好"或"有多新"。一个系统完全可能生成高度多样化但质量低劣的内容（如随机拼接），也可能生成词汇丰富但毫无新意的内容（如用不同方式重复陈词滥调）。创造力的核心是"新颖且有价值"，多样性只是新颖性的必要条件，而非充分条件。

### 题目 2：为什么交互评估是创造力 MAS 特有的需求？

> [!tip] 理解提示（非原文）
> 想想单 agent 系统和多 agent 系统在评估上的根本区别——什么信息在多 agent 系统中有但单 agent 系统中没有？

> [!success]- 参考答案（非原文）
> 因为创造力 MAS 的最终输出是 agent 间交互过程的产物。在单 agent 系统中，输入→输出的映射是黑箱；但在 MAS 中，交互日志是可见的、可分析的。交互评估的意义在于：（1）诊断——如果输出不好，是哪个 agent 或哪种交互模式出了问题？（2）优化——知道交互瓶颈在哪里，才能有针对性地改进系统设计；（3）公平比较——不同系统的最终输出可能相似，但交互效率可能天差地别。

## 相关页面

- [[raw/lessons/Multi-Agent-Survey/06-creativity/03-creativity-mas-persona]]
- [[raw/lessons/Multi-Agent-Survey/06-creativity/05-creativity-mas-challenges-and-conclusion]]
- [[raw/lessons/Multi-Agent-Survey/06-creativity/02-creativity-mas-techniques]]

## 下一步学习

- 上一篇：[[raw/lessons/Multi-Agent-Survey/06-creativity/03-creativity-mas-persona|03-persona]]
- 下一篇：[[raw/lessons/Multi-Agent-Survey/06-creativity/05-creativity-mas-challenges-and-conclusion|05-challenges-conclusion]]
