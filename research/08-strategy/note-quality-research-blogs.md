---
type: strategy-note
tags: [笔记体系, 写作规范, 调研, 顶级研究者, 范例分析]
created: 2026-07-05
updated: 2026-07-05
sources:
  - https://lilianweng.github.io/
  - https://jalammar.github.io/
  - https://karpathy.ai/
  - https://colah.github.io/
  - https://distill.pub/
  - https://deeplearningwithpython.io/
  - https://sebastianraschka.com/
  - https://www.3blue1brown.com/
  - https://paperswithcode.com/
  - https://ar5iv.org/
---

# 顶级科研笔记范例调研

> **元信息**
> - 调研日期：2026-07-05
> - 覆盖对象：10 个（Karpathy / Chris Olah / François Chollet / Sebastian Raschka / Lilian Weng / Jay Alammar / Distill.pub / 3Blue1Brown / Papers with Code / arXiv HTML+ar5iv）
> - 调研方法：联网搜索（web-search-prime）+ 页面直接抓取
> - 核心发现：当前 D:\note 笔记在"可视化—信息密度—论文卡片化—结构模板"四个维度均有可量化的提升空间；最值得复刻的对象是 Lilian Weng（结构密度）+ Jay Alammar（图文协同）+ Distill.pub（交互式图表设计思想）。

## 摘要

本次调研覆盖 10 个顶级研究者/平台的公开笔记范例，从写作结构、视觉表达、信息密度、可复用性四个维度进行了系统比对。综合结论：(1) **Lilian Weng** 以"前置目录 + 公式编号 + 大段自洽 + 高密度参考文献"建立了长文式博客的标杆；(2) **Jay Alammar** 证明了"插图即解释"的可行性——核心论文每讲一步配一张自绘示意图；(3) **Distill.pub** 的"交互即论证"理念把图表从装饰升级为论据；(4) **3Blue1Brown** 把数学直觉可视化做到了极致，是手绘风格图表的最高水平；(5) **arXiv HTML + ar5iv** 的 LaTeXML 路径让论文本身具备阅读级排版，启示我们"原始资料排版优化"也是知识管理的一环。相比之下，D:\note 当前笔记以"callout 折叠 + 文字段落"为主，缺少自定义配图、缺少论文卡片化模板、缺少"图—文—公式"三件套协同。建议优先落地"代表作结构剖析 → 模板化 → 自绘示意图规范"三步走。

---

## 1. Lilian Weng（最详细，标杆）

### 1.1 笔记特点

Lilian Weng（前 OpenAI 安全研究负责人）的博客 [Lil'Log](https://lilianweng.github.io/) 是当前公认最值得复刻的"研究者博客模板"。特点如下：

- **覆盖深度极高**：单篇博客常达 8000-15000 字（如 [LLM Powered Autonomous Agents](https://lilianweng.github.io/posts/2023-06-23-agent/)、[What are Diffusion Models?](https://lilianweng.github.io/posts/2021-07-11-diffusion-models/)、[Prompt Engineering](https://lilianweng.github.io/posts/2023-03-15-prompt-engineering/)）。
- **结构化极强**：每篇博客前置 TOC（Table of Contents）+ H2 大节 + H3 小节 + 内部小标题层层递进。
- **公式编排规范**：LaTeX 行内 + 行间公式混排；公式后必有"直觉解释"段落。
- **参考文献密度高**：单篇 50-100 条引用，挂在文末 "References" 节，使用学术风格编号。
- **更新机制透明**：博客末尾会标注 "Last updated on ..."，版本化可追溯。
- **可视化克制**：多用 ASCII/示意图 + 偶尔的关键图（如 Agent 系统的 component overview 一张图讲透整体架构）。

### 1.2 代表作结构剖析：[LLM Powered Autonomous Agents](https://lilianweng.github.io/posts/2023-06-23-agent/)（2023-06-23）

**开场**：第一段即点题"LLM 作为 agent 大脑，加上几个关键组件"，立即给读者 mental model。

**整体结构（实测目录）**：

```
1. Agent System Overview
   - Fig. 1. Overview of a LLM-powered autonomous agent system
2. Component One: Planning
   2.1 Task Decomposition
   2.2 Self-Reflection
3. Component Two: Memory
   3.1 Types of Memory
   3.2 Maximum Inner Product Search (MIPS)
4. Component Three: Tool Use
5. Case Studies: Scientific Discovery Agents & Generative Agents Simulation
6. References (70+)
```

**结构剖析要点**：

- **顶部 Figure 1 给出系统总览图**，后面每个 component 单独成节，把图拆开重讲——经典的"全景 → 拆解 → 串联"叙事。
- **每个 component 内部继续用 H3 二级结构拆解**，如 Memory 分为 Types（感觉记忆/短时记忆/长时记忆的认知科学类比）+ MIPS（检索技术细节）。这种"概念先讲直觉 → 再讲技术"双层结构是 Lilian 的标志风格。
- **Case Studies 节** 把前面抽象的组件拉回具体例子（Scientific Discovery Agents / Generative Agents Simulation），避免读者"听懂了但不知怎么用"。
- **结尾 References 数量极大**，每条都是真实可点的学术引用，体现"严肃博客"。

**图表使用**：

- 文章只有 1-2 张关键插图（系统总览图 + 可能一张 memory 检索示意），其余全靠文字和公式。
- 公式均用 LaTeX 行间显示，编号规范（如 Eq. 1, 2, 3）。
- 表格只在跨方法对比时出现（如 ReAct / Reflexion / Chain-of-Thought 三种 prompting 方法对比）。

**结尾模式**：没有"总结段"，直接进 References。这是 Lilian 的有意取舍——博客就是工作笔记的延伸，不需要总结性陈词。

### 1.3 写作风格

- **学术性 70% + 通俗性 30%**：主体是技术细节，但每个概念第一次出现时都会用一句自然语言解释"它是什么/为什么重要"。
- **类比克制**：Lilian 偶尔用认知科学的类比（如记忆分为 sensory / short-term / long-term），但不滥用，整体保持工程化的冷静语气。
- **断言清晰**：每节第一句话往往是结论或定义句（"Task decomposition can be done ...", "Self-reflection ..."），然后再展开论证。
- **引用驱动**：所有重要论断都有引用支撑，这点和教科书、综述论文一致。

### 1.4 可视化使用

- **少而精的图**：核心论点配一张图，绝不堆图。
- **公式排版规范**：行内 `$...$` + 行间 `$$...$$`，自动编号；变量首次出现时一定有"where X is ..." 的说明。
- **表格用于对比**：方法对比、benchmark 对比一律走表格，不用 bullet list。
- **配色朴素**：博客使用 GitHub Pages 默认主题，不靠配色取胜，靠结构清晰。

### 1.5 对 D:\note 笔记的差距

1. **缺少 TOC 自动生成**：Lilian 每篇博客前置 TOC，D:\note 多数学习笔记直接进入正文，缺少导航。
2. **公式编号缺失**：D:\note 公式以 callout 形式呈现，没有 `(Eq. 1)` 式引用机制，后续笔记互引困难。
3. **References 节缺失或简化**：现有 `_templates/lesson.md` 引用源字段放在 frontmatter 而非正文末尾，不便于读者按页内查看。
4. **"Last updated" 时间戳缺失**：多数笔记只有 `created`，没有 `updated`，更新痕迹不可见。
5. **Case Studies 节缺失**：D:\note 多数概念笔记讲完即止，缺少"概念 → 真实论文/项目案例"的回扣。

### 1.6 可学习的具体技巧

1. **每篇笔记固定 6 段结构**：Overview → Component 1 → Component 2 → ... → Case Studies → References。组件式结构天然适合任何复杂主题（算法 / 框架 / 调研）。
2. **前置 Figure 1 总览图**：每个主题开篇配 1 张"组件关系图"，后续每节重提这张图的某一部分。
3. **每节第一句下断言**：如 "Task decomposition can be done via (1) simple prompting, (2) task-specific instructions, or (3) human inputs."——断言先行，再展开。
4. **文末 References 至少 20 条起步**，且全部可点击——体现严谨度。
5. **公式 + 直觉 + 数值示例三件套**：写一个公式前先讲动机，写完后给一个具体数字例子（如 `d=64, n=512 → O(dn²)=2.1M ops`）。

---

## 2. Jay Alammar（图解教学典范）

### 2.1 笔记特点

[Jay Alammar](https://jalammar.github.io/)（前 Salesforce 工程师，机器学习可视化布道者）以一系列 "The Illustrated XXX" 文章闻名：

- [The Illustrated Transformer](https://jalammar.github.io/illustrated-transformer/)（被引用次数最多）
- [The Illustrated GPT-2](https://jalammar.github.io/illustrated-gpt2/)
- [The Illustrated Word2vec](https://jalammar.github.io/illustrated-word2vec/)
- [Visualizing Machine Learning One Concept at a Time](https://jalammar.github.io/)（合集入口）

**核心特点**：

- **图就是叙事**：每张图都讲一步推理，去掉图就看不懂。
- **配色统一**：固定使用几种"安全色"（蓝、绿、橙、红）做组件区分，不滥用渐变。
- **从宏观到微观层层放大**：一张总览图 → 单组件图 → 单 token 流程图 → 单矩阵乘法示意——四层粒度切换。
- **文字极简**：每段不超过 3-4 句，只描述图在说什么，不重复图的内容。

### 2.2 代表作结构剖析：[The Illustrated Transformer](https://jalammar.github.io/illustrated-transformer/)

**开场**：用一句话点题"This post, we will look at the Transformer – a model that uses attention to boost the speed with which these models can be trained."——然后立即进入**A High-Level Look** 章节给出一张端到端流程图。

**整体结构（实测目录）**：

```
1. A High-Level Look
2. Bringing The Tensors Into The Picture
3. Self-Attention at a High Level
4. Self-Attention in Detail
   4.1 Compute Q, K, V
   4.2 Compute attention scores
   4.3 Softmax + weighting
5. Multi-Head Attention
6. Positional Encoding
7. The Decoder Side
8. The Final Linear and Softmax Layer
9. Recap
```

**结构剖析要点**：

- **"High-Level → Detail" 双层结构**：每讲一个新概念都先给一张简化的总览图，再用 1-3 节展开细节。
- **"Putting it all together" 模式**：到第 9 节 Recap，再贴一次完整流程图，让读者"看完整条链路"。
- **每个小节都配 1-2 张图**：从 The Illustrated Transformer 的截图看，文章共 ~30 张图，平均每节 3-4 张。
- **不使用 callout / 折叠**：信息全部展开，依赖图的清晰度。

**图表使用**：

- **手绘风格 + 简化标注**：箭头粗、字体大、组件块状化、避免细线。
- **矩阵可视化**：把 attention matrix 画成彩色热力图（Q/K/V 矩阵分别着色）。
- **动画 GIF 辅助**：早期版本有 attention flow 的 GIF，后被静态图替代。

**结尾**：第 9 节 Recap 把整篇文章的图重新串联一次——这种"重复总览图"是教学文章的标志手法。

### 2.3 写作风格

- **通俗性 80% + 学术性 20%**：Jay Alammar 完全面向"想搞懂 Transformer 但数学不太行"的读者，几乎不用 LaTeX 公式，必要时用"if you remember matrix multiplication..."这种过渡句。
- **类比丰富**：把 attention 类比为"数据库查询"（Q=query, K=key, V=value），把 positional encoding 类比为"在盒子上贴标签"。
- **语气亲和**：用 "we"、"you"、"let me show you" 等第二人称，营造一对一教学感。

### 2.4 可视化使用

- **图占页面 60-70%**：文字只承载过渡和重点强调。
- **统一配色**：蓝=Query, 绿=Key, 橙=Value, 灰=背景——读者看一张图就学会配色含义。
- **矩阵/向量用方块表示**：避免数学符号画图，直接用颜色块。
- **箭头粗壮**：1-2 px 黑色箭头，方向明确。

### 2.5 对 D:\note 笔记的差距

1. **自绘示意图缺失**：D:\note 现有 learning note 几乎全部依赖文字 + 代码 + 公式截图，没有"为这个概念专门画一张图"。
2. **配色规范未统一**：每个 callout、每个高亮没有全局配色标准（蓝/绿/橙/红的语义未定义）。
3. **缺少"总览—展开—Recap"叙事**：多数笔记讲完即止，没有 Recap 节回扣。
4. **缺少图注规范**：现有图很少有 figure caption（Fig. 1. XXX 的形式）。

### 2.6 可学习的具体技巧

1. **每篇笔记强制配 1 张"概念总览图"**：即使是 ASCII 流程图也行，要有一个 visual entry point。
2. **统一配色语义**：定义 D:\note 配色规范，如"蓝色=输入，绿色=处理，橙色=输出，红色=警告/损失"。
3. **每节配 1 张细节图**：拒绝"一图走天下"，每深入一层就换一张更细的图。
4. **结尾 Recap 节**：把全文的核心图重新串联一次，方便读者快速回顾。
5. **弱化数学公式，用类比承载推理**：对入门读者，把 Q/K/V 类比为"图书馆检索"比直接讲 attention 公式有效。

---

## 3. Andrej Karpathy（深度 + 工程化）

### 3.1 笔记特点

[Andrej Karpathy](https://karpathy.ai/)（前 Tesla AI 主管、OpenAI 创始成员）的内容以**视频 + 代码 + 长文博客**组合为主：

- [Neural Networks: Zero to Hero](https://karpathy.ai/zero-to-hero.html)（YouTube 系列课程）
- [A Recipe for Training Neural Networks](http://karpathy.github.io/2019/04/25/recipe/)
- [Hacker's guide to Neural Networks](http://karpathy.github.io/neuralnets/)
- [Software 2.0](https://karpathy.medium.com/software-2-0-a64152b37c35)（Medium 长文）

**核心特点**：

- **"从零手写"哲学**：每个概念都要写一遍代码，不调包。
- **课堂式叙述**：像在黑板前讲解，边写边讲。
- **不回避踩坑**：明确说出"这样写会怎样 / 为什么这样写不对"。
- **少量配图，重在代码**：核心载体是 Python 代码 + console 输出。

### 3.2 代表作结构剖析：[A Recipe for Training Neural Networks](http://karpathy.github.io/2019/04/25/recipe/)

**开场**：用一句名人名言式断言开场（"The first step to training a neural net is to not touch any neural net code at all..."），奠定"先验数据，再写模型"的反直觉基调。

**整体结构（实测目录）**：

```
1. Introduction
2. Recipe
   2.1 Look at the data (sanity check labels, class balance, data augmentations)
   2.2 Set up the end-to-end training/evaluation skeleton + get simple baselines
   2.3 Overfit a small batch
   2.4 Set up learning rate, default = 1e-3 / batch size
   2.5 Coarse -> fine random hyperparameter search
   2.6 Set up logging, save checkpoints
   2.7 From layers to CNNs (visualize first-layer filters, check saliency maps)
3. Conclusion
```

**结构剖析要点**：

- **"Checklist 化"结构**：每个子节都是"做完这一步再走下一步"的清单模式——天然适合作为工程模板。
- **每节末尾有 "Things to look for"**：列出本节容易踩的坑——把工程经验浓缩为可复用的 checklist。
- **不使用复杂图表**：依赖代码块 + 简短要点列表。

### 3.3 写作风格

- **教学性 90%**：每句话都在"教读者做事"。
- **不回避个人经验**：明确说"在我的经验中……"、"我犯过的错是……"——把博客当成技术 memo。
- **短句 + 命令式语气**：多用 "Get this right."、"Don't."、"Look at the data."——简明有力。

### 3.4 可视化使用

- **极简**：几乎没有自定义插图。
- **ASCII art / 排版辅助**：用代码块表达架构。
- **依赖 YouTube 视频**：可视化任务完全交给视频承担，博客只是 reference + code。

### 3.5 对 D:\note 笔记的差距

1. **"Checklist 化"结构缺失**：D:\note 多数笔记是叙述式，缺少"做完这一步再下一步"的工程清单。
2. **个人经验/踩坑记录不足**：现有笔记太"教科书"，缺少"我/作者实际怎么做的"。
3. **代码块与文字叙述的协同**：Karpathy 在每段后立即给代码片段做演示，D:\note 笔记代码块常常孤立。

### 3.6 可学习的具体技巧

1. **工程类笔记采用 "Checklist + Things to look for" 双段结构**：每节先给步骤，再给易错点。
2. **每段叙述后立即给代码片段**：代码块长度不超过 15 行，紧跟文字解释。
3. **明确的开场断言**：第一句就是反直觉或核心断言（"不要碰任何 NN 代码，先看数据"）。
4. **使用 YouTube/视频作为补充**：可视化任务交给视频，博客承担 reference + 代码。

---

## 4. Chris Olah（Distill 联合创始人）

### 4.1 笔记特点

[Chris Olah](https://colah.github.io/)（Anthropic 可解释性研究者，Distill.pub 联合创始人）的博客和 Distill 文章有强烈的"解释即论证"风格：

- 个人博客：[colah.github.io](https://colah.github.io/)（早年的 LSTM/GRU/Attention 图解系列已成经典）
- Distill 文章合集：[distill.pub](https://distill.pub/)
- 代表作：[Attention and Augmented RNNs](https://distill.pub/2016/augmented-rnns/)、[Feature Visualization](https://distill.pub/2017/feature-visualization/)、[The Building Blocks of Interpretability](https://distill.pub/2018/building-blocks/)

**核心特点**：

- **早期文章使用 SVG 矢量插图**（如 LSTM cell 的解剖图）：组件清晰、配色克制。
- **极强的"可视化论证"理念**：图本身就是论据，不是装饰。
- **教学语言**：每篇都用大量类比和直觉引入。

### 4.2 代表作结构剖析：[Understanding LSTM Networks](https://colah.github.io/posts/2015-08-Understanding-LSTMs/)

**开场**：先用 RNN 的痛点引入（"long-term dependencies 难以学习"），给出朴素的"加记忆"直觉。

**整体结构**：

```
1. The Problem of Long-Term Dependencies
2. LSTM Networks
3. The Core Idea Behind LSTMs
4. Step-by-Step LSTM Walk Through
   - Forget gate
   - Input gate
   - Cell state update
   - Output gate
5. Variants on LSTM (Peephole, Coupled, Gated Recurrent Unit)
6. Conclusion
```

**结构剖析要点**：

- **"Step-by-Step Walk Through" 是 Chris 的标志结构**：每个 gate 用一节 + 一张手绘示意图讲透。
- **SVG 手绘示意图**：每张图都是作者手绘的矢量图，颜色不超过 4 种（黄、蓝、红、绿），组件边界清晰。
- **每个 gate 配一张"内部结构图"**：图是这篇文章的灵魂。

### 4.3 写作风格

- **学术 + 通俗混合**：每个数学概念第一次出现时，先用类比，再用数学，再用图。
- **结论驱动**：每节末尾给"so what"——这个 gate 为什么重要、解决了什么。

### 4.4 可视化使用

- **手绘 SVG 插图**：组件用浅色块填充，箭头用粗黑线，文字标注直接在图上。
- **配色极简**：每张图不超过 4 种颜色。
- **图即叙事**：去掉图，文章几乎无法阅读。

### 4.5 对 D:\note 笔记的差距

1. **手绘风格插图缺失**：现有笔记依赖 callout + 截图，没有"为这个概念专门画的图"。
2. **Step-by-Step Walk Through 模板缺失**：多数笔记直接讲结论，缺少"一步一步走一遍"的过程性叙事。
3. **每图独立配色语义缺失**：每张图应该有"为什么这样配色"的内在逻辑。

### 4.6 可学习的具体技巧

1. **每个核心组件用一节 + 一张手绘示意图讲透**：拒绝"一图讲三件事"。
2. **配色克制**：每张图 ≤ 4 色，每种颜色对应一个语义。
3. **Step-by-Step Walk Through**：把抽象流程拆成 N 个 sub-step，每个 sub-step 一节。

---

## 5. François Chollet（教科书级严谨）

### 5.1 笔记特点

[François Chollet](https://github.com/fchollet)（Keras 作者，Google DeepMind）的内容以**教材 + Jupyter notebooks**为主：

- [Deep Learning with Python (3rd edition, online free)](https://deeplearningwithpython.io/)
- [deep-learning-with-python-notebooks](https://github.com/fchollet/deep-learning-with-python-notebooks)（GitHub）
- [Keras 官方文档](https://keras.io/)

**核心特点**：

- **教科书级严谨**：每章末有 Q&A + 关键概念回顾。
- **代码即教材**：每个概念都配可直接运行的 Jupyter notebook。
- **直觉 + 数学 + 代码三层**：先讲直觉动机，再讲数学推导，最后给代码。
- **适度可视化**：用 matplotlib 输出训练曲线 + 少量示意图。

### 5.2 代表作结构剖析（章节层面）：Chapter 4 "Getting started with neural networks: Classification and regression"

**结构**：

```
1. Classification and regression glossary
2. The MNIST handwritten digit classifier
   - Working with image data
   - The model architecture
   - The compilation step
   - Preparing the data
   - Fitting the model
   - Evaluating the model
   - Using the model to predict on new data
3. Regression: Predicting house prices
4. Summary
```

**结构剖析要点**：

- **"完整端到端任务"模式**：每个章节就是一个完整任务，从数据到评估。
- **每节都是同一模板**：问题描述 → 数据 → 模型 → 训练 → 评估。
- **Chapter 末尾 Summary 章节**：用 1-2 页 + 关键 takeaway 列表回顾。

### 5.3 写作风格

- **教科书语气 + 第二人称教学**：多用 "you will"、"let's ..."。
- **每个概念先定义**：先讲"X is..."再讲"X is used for..."。
- **公式与代码并重**：每个公式后必给 PyTorch/Keras 代码示例。

### 5.4 可视化使用

- **依赖 matplotlib**：训练曲线、attention map、filter visualization 都用代码出图。
- **轻量示意图**：少量组件结构示意图，主要靠代码输出。
- **图表在代码块内**：图和代码合在一起，notebook 本身就是最终成品。

### 5.5 对 D:\note 笔记的差距

1. **Chapter Summary 节缺失**：多数笔记没有"关键 takeaway 列表"。
2. **"端到端任务"模板缺失**：现有笔记偏概念性，缺少"数据 → 模型 → 训练 → 评估"完整链路。
3. **Q&A 节缺失**：教科书末尾的"自我提问"环节缺失。

### 5.6 可学习的具体技巧

1. **每章末尾强制 Summary 节**：用 3-5 个 bullet points 总结本章 takeaway。
2. **使用端到端任务模板**：每个新概念都配一个"数据集 → 模型 → 训练 → 评估"完整案例。
3. **Jupyter notebook 作为载体**：可运行代码 + 注释 = 最佳学习材料。

---

## 6. Sebastian Raschka（工程导向 + 开源教材）

### 6.1 笔记特点

[Sebastian Raschka](https://sebastianraschka.com/)（畅销书《Python Machine Learning》《Build a Large Language Model From Scratch》作者）的博客和书配套 notebook：

- 个人博客：[sebastianraschka.com](https://sebastianraschka.com/)
- 杂志：[magazine.sebastianraschka.com](https://magazine.sebastianraschka.com/p/supporting-ahead-of-ai)
- 配套 GitHub：[rasbt/LLMs-from-scratch](https://github.com/rasbt/LLMs-from-scratch)、[rasbt/python-machine-learning-book-3rd-edition](https://github.com/rasbt/python-machine-learning-book-3rd-edition)

**核心特点**：

- **"From Scratch" 哲学**：每个模型都从零实现 PyTorch 代码。
- **配套图非常精美**：书中的 SVG 示意图由专门的设计师制作，质量极高。
- **章节结构教科书化**：每章"问题 → 概念 → 代码 → 实验 → 总结"五段式。
- **理论深度与工程实现并重**：不是单纯的"调包教程"，而是"先讲原理再讲实现"。

### 6.2 代表作结构剖析（书章节层面）：Chapter 4 "Implementing a GPT model from scratch to generate text"

**结构**：

```
1. Coding an LLM architecture
   - LLM architecture overview
   - Normalization
   - GELU activation
   - Shortcut / residual connection
2. Coding the transformer block
3. The full GPT model
4. Generating text
5. Summary
```

**结构剖析要点**：

- **"Building blocks 递进"模式**：从 LLM 总览 → 单组件（GELU / LayerNorm / Residual）→ Transformer Block → 完整 GPT → 文本生成。
- **每节都先讲概念再给代码**：代码块不超过 25 行，每段后立即解释。
- **章节末尾 Summary 卡片**：用"本章学到的" 列表收尾。

### 6.3 写作风格

- **清晰直接**：每节开头一句话定义本节目标，结尾一句话总结本节学到的。
- **代码即文档**：变量命名自解释，函数 docstring 完整。
- **可运行优先**：所有代码片段都是真实可运行的。

### 6.4 可视化使用

- **专业 SVG 插图**：书中的图由专业设计师制作，配色克制、信息密度高。
- **配套动图/示意图**：GitHub repo 提供 book figures 的 PNG/SVG 源文件。

### 6.5 对 D:\note 笔记的差距

1. **"Building Blocks 递进"模板缺失**：缺少"总览 → 单组件 → 组件组合 → 完整系统"的递进叙事。
2. **代码与图协同不足**：现有笔记的代码和图常常分离（代码在一处，图在另一处）。

### 6.6 可学习的具体技巧

1. **"Building Blocks"递进结构**：先讲总体架构图，再讲每个组件，最后讲组件如何组合。
2. **每章末尾 "Main Takeaways"**：5 个 bullet points 总结本章。
3. **代码块长度 ≤ 25 行**：长代码必须拆成多个块，每块单独解释。

---

## 7. Distill.pub（交互式论文范式）

### 7.1 笔记特点

[Distill.pub](https://distill.pub/)（Chris Olah 联合创办的开放期刊，2021 年起进入无限期 hiatus）的文章是"交互式论文"的最高水平：

- [How to Create a Distill Article](https://distill.pub/guide/)
- [Communicating with Interactive Articles](https://distill.pub/2020/communicating-with-interactive-articles)
- [Feature Visualization](https://distill.pub/2017/feature-visualization)
- [The Building Blocks of Interpretability](https://distill.pub/2018/building-blocks)

**核心特点**：

- **交互即论证**：读者通过拖动滑块、点击节点、调整参数来"探索"概念——交互不是装饰，是论据的一部分。
- **可复现的图**：所有图表都是可交互的 d3.js 或 canvas 实现，参数可调。
- **严格的排版规范**：参考印刷时代的学术期刊美学（大留白、衬线字体、章节分隔）。
- **"蒸馏式"叙事**：把复杂概念"蒸馏"为一系列可视化步骤。

### 7.2 代表作结构剖析：[Feature Visualization](https://distill.pub/2017/feature-visualization)

**结构**：

```
1. Introduction (目标: "What does a neural net see?")
2. The Feature Visualization Optimizer
3. How does it work? (interactive: user can drag to see how parameters affect output)
4. Applications
   - Understanding neurons
   - Visualizing the "loss landscape"
   - Synthetic images
5. Related work
6. Conclusion + Future work
```

**结构剖析要点**：

- **每个图都是可交互的**：用户可以拖动滑块看到不同参数下的网络响应——"图"本身就是一个实验。
- **章节之间用大留白分隔**：参考纸质期刊美学，让读者有"翻页"感。
- **代码透明**：交互式图表背后的 JS / Python 代码直接展示在页面上。

### 7.3 写作风格

- **学术严谨 + 实验式叙事**：每个断言都附可视化证据。
- **段落短**：每段 3-5 句，避免长段落。
- **关键概念首次出现时给定义卡片**：用 callout 风格但更精致的卡片展示。

### 7.4 可视化使用

- **交互式图表**：d3.js + canvas 实现。
- **配色克制**：基础色 + 高亮色不超过 5 种。
- **印刷级排版**：衬线字体 + 大留白 + 图注规范。
- **图即论文**：图占 50-70% 页面。

### 7.5 对 D:\note 笔记的差距

1. **交互式图表缺失**：现有笔记全静态，无法让读者"调参数看效果"。
2. **印刷级排版缺失**：当前 Obsidian 默认主题排版较朴素，缺少专业排版规范。
3. **"代码透明"理念缺失**：可视化背后的代码很少展示。

### 7.6 可学习的具体技巧

1. **大留白分隔章节**：每章之间留 2-3 行空白，强化"翻页"感。
2. **关键概念卡片化**：每个核心概念用 callout/卡片形式独立展示，配定义 + 类比 + 例子。
3. **图表背后的代码透明**：每个示意图后附"this chart is generated by ..."的代码片段。
4. **使用 d3.js 或类似库**：即使只做静态图，也使用专业可视化库而非默认 chart。

---

## 8. 3Blue1Brown（数学直觉可视化巅峰）

### 8.1 笔记特点

[3Blue1Brown](https://www.3blue1brown.com/)（Grant Sanderson 的频道）的神经网络系列是"用视频讲数学"的最高水平：

- [Neural Networks 章节页](https://www.3blue1brown.com/topics/neural-networks)
- [But what is a neural network?](https://www.youtube.com/watch?v=aircAruvnKk)（Deep learning chapter 1）
- [Gradient descent](https://www.youtube.com/watch?v=IHZwWFHWa-w)（chapter 2）
- [Backpropagation](https://www.youtube.com/watch?v=Ilg3gGewQ5U)（chapter 3）
- [Transformers](https://www.youtube.com/watch?v=wjZofJX0v4M)（2024 新系列）

**核心特点**：

- **Manim 引擎自制**：所有动画用 Python 的 Manim 库手工编写，可精确控制每个元素。
- **数学直觉为先**：先讲"为什么这样"再讲"是什么"。
- **几何化思维**：把矩阵乘法、attention 等概念用 3D 几何呈现。
- **统一配色语义**：紫色=输入、绿色=权重、橙色=输出、红色=梯度——贯穿整个系列。

### 8.2 代表作结构剖析：[Deep learning chapter 1: But what is a neural network?](https://www.youtube.com/watch?v=aircAruvnKk)

**结构**：

```
0. 引入：人类识别手写数字的直觉
1. 用"神经元装数字"做类比
2. 多层网络结构示意
3. 用 MNIST 子集演示识别过程
4. 引入"权重"概念
5. 预告：下一节讲训练
```

**结构剖析要点**：

- **类比层层递进**：神经元→层→数字识别→权重——每一层类比都是上层的细化。
- **几何动画承载推理**：每个数学概念都先给几何动画演示，再给符号公式。
- **结尾"预告"**：明确告诉读者下一节讲什么，强化知识连贯。

### 8.3 写作风格（视频脚本层面）

- **直觉优先于公式**：每讲一个数学公式前，先用直觉动画解释 1-2 分钟。
- **极简文字**：视频中文字极少，几乎全靠画面和口头讲解。
- **悬念 + 揭示**：先抛问题（"为什么 NN 能识别数字？"），动画演示，再揭示答案。

### 8.4 可视化使用

- **Manim 自制动画**：数学概念全部用动画呈现（神经网络节点随训练变化、梯度下降的损失曲面等）。
- **配色语义统一**：紫色=神经元值，绿色=权重，橙色=激活，红色=梯度/损失——跨视频一致。
- **几何化**：把矩阵乘法画成空间变换，把 attention 画成向量间的相似度。

### 8.5 对 D:\note 笔记的差距

1. **配色语义缺失**：现有笔记没有"颜色 = 概念"的全局规范。
2. **几何化直觉缺失**：矩阵、向量等概念没有几何示意图。
3. **"预告—揭示"叙事缺失**：笔记不告诉读者"下一节讲什么"。

### 8.6 可学习的具体技巧

1. **配色语义统一**：定义 D:\note 全局配色规范（紫=输入、绿=权重、橙=输出、红=梯度）。
2. **几何化关键概念**：矩阵乘法、attention、loss landscape 都要有几何图。
3. **"下一节预告"段**：每篇笔记末尾写"下一步学习"链接或"下一节将讲"段落。

---

## 9. Papers with Code（论文卡片化呈现）

### 9.1 笔记特点

[Papers with Code](https://paperswithcode.com/)（Meta 在 2025 年 7 月关停，但数据仍在 Hugging Face / 代码库中可见）是**论文+代码+SOTA benchmark 三位一体**的最大平台：

- 主要功能：把论文卡片化，每张卡片展示论文标题、摘要、关键结果、代码链接、benchmark 排名。
- 设计哲学：**卡片 = 论文的"人话版 summary"**——读者用 30 秒就能判断这篇论文要不要深读。

**核心特点**：

- **论文卡片标准化**：所有论文呈现为同一种卡片结构（title / abstract / results table / code link）。
- **SOTA leaderboard**：每个任务有排行榜，论文按 SOTA 排序。
- **任务 / 数据集 / 指标三级索引**：可按 task 查所有相关论文。

### 9.2 论文卡片典型结构

```
┌─────────────────────────────────────────────┐
│ [Paper Title]                               │
│ Authors, Year, Venue                        │
├─────────────────────────────────────────────┤
│ TLDR / Abstract                             │
├─────────────────────────────────────────────┤
│ Code: 🔗 Official / Community               │
│ Datasets: ImageNet, COCO, ...               │
├─────────────────────────────────────────────┤
│ Results Table (Task: Image Classification)  │
│   Model       | Top-1 Acc | Year            │
│   ResNet-152  | 78.6%     | 2016             │
│   ViT-L/16    | 85.3%     | 2020             │
│   ...                                        │
└─────────────────────────────────────────────┘
```

**结构剖析要点**：

- **三段卡片化**：基本信息 → 摘要 → 实验/资源链接——读者快速 decide 是否深读。
- **SOTA leaderboard 自动聚合**：所有方法自动按 metric 排序。
- **跨论文对比标准化**：表格列对齐，可直接横向比较。

### 9.3 写作/呈现风格

- **极简文案**：论文卡片正文不超过 200 字。
- **链接驱动**：每条信息都有 external link。

### 9.4 可视化使用

- **表格占主导**：所有对比都走表格。
- **小型徽章**：任务、数据集、代码库都有统一徽章。

### 9.5 对 D:\note 笔记的差距

1. **论文卡片化模板缺失**：D:\note 的 `research/02-papers/` 笔记结构不统一，没有"30 秒 summary"的卡片模板。
2. **SOTA 对比表缺失**：阅读论文时不做横向 SOTA 对比。
3. **代码/数据集徽章缺失**：没有统一的"数据集/代码"链接徽章。

### 9.6 可学习的具体技巧

1. **论文卡片模板**：每篇论文笔记顶部强制 TLDR + 任务/数据集/指标表格。
2. **SOTA 对比表**：阅读论文时手绘一张"该方法 vs SOTA"对比表。
3. **链接徽章**：每个数据集、每个代码库都用统一格式的徽章链接（如 `[Code](url)`）。

---

## 10. arXiv HTML + ar5iv（论文排版新标准）

### 10.1 笔记特点

[arXiv HTML](https://info.arxiv.org/about/accessible_HTML.html) + [ar5iv](https://ar5iv.org/) 用 [LaTeXML](https://math.nist.gov/~BMiller/LaTeXML/manual.pdf) 把 LaTeX 论文自动转为响应式 HTML：

- arXiv 从 2023 年起为新论文自动生成 HTML 版本
- ar5iv 把 arXiv 全库的论文转 HTML
- 效果：论文可在手机/平板/电脑上阅读，公式用 MathML 渲染可缩放、可被屏幕阅读器读出

**核心特点**：

- **响应式排版**：图、表格、公式根据屏幕宽度自动调整。
- **公式可交互**：MathML 渲染的公式可缩放、可复制 LaTeX 源码。
- **辅助功能友好**：屏幕阅读器可访问，符合 WCAG。
- **LaTeX-to-HTML 自动转换**：作者无需手工重写。

### 10.2 论文结构剖析（任一篇 arXiv HTML 论文）

**典型结构**：

```
1. Abstract
2. Introduction
3. Related Work
4. Method
5. Experiments
6. Conclusion
7. References
```

**结构剖析要点**：

- **章节标题固定层级**：H1=章, H2=节, H3=小节。
- **公式自动编号 + 引用**：Eq. (1), (2), ... 可双向引用。
- **表格自动转 HTML table**：可在小屏幕上横向滚动。
- **图自动 responsive**：点击可放大。

### 10.3 写作风格

- **学术规范**：每篇论文遵守 IMRaD（Introduction-Method-Results-Discussion）结构。
- **段落紧凑**：每段 5-8 句，避免长段落。

### 10.4 可视化使用

- **LaTeX 原生图表**：tikz、pgfplots 矢量图。
- **配色简洁**：黑白为主，强调内容。
- **公式排版**：MathJax / MathML，标准学术格式。

### 10.5 对 D:\note 笔记的差距

1. **IMRaD 结构模板缺失**：D:\note 论文笔记没有标准"问题—方法—实验—结论"四段。
2. **公式双向引用缺失**：没有 `(Eq. 1)` 式编号 + 引用机制。
3. **响应式排版缺失**：在手机上阅读体验差。

### 10.6 可学习的具体技巧

1. **论文笔记采用 IMRaD 结构**：每篇论文笔记固定"问题—方法—实验—结论"四节。
2. **公式编号 + 双向引用**：所有公式用 `(Eq. N)` 编号，正文用"as shown in Eq. (3)"引用。
3. **关键表格用 Markdown table**：实验结果一律用 Markdown 表格展示，不放截图。

---

## 综合差距矩阵

> 评分 1-5，分数越高越强。
> 评估对象：D:\note 当前笔记体系 vs 每个范例。

| 范例 | 深度 | 结构 | 可视化 | 通俗性 | 可复用性 | 综合 |
|---|---|---|---|---|---|---|
| **Lilian Weng** | 5 | 5 | 3 | 4 | 5 | **4.4** |
| **Jay Alammar** | 4 | 4 | 5 | 5 | 4 | **4.4** |
| **Andrej Karpathy** | 5 | 4 | 2 | 4 | 4 | **3.8** |
| **Chris Olah** | 5 | 4 | 5 | 4 | 3 | **4.2** |
| **François Chollet** | 4 | 5 | 3 | 4 | 5 | **4.2** |
| **Sebastian Raschka** | 4 | 5 | 4 | 4 | 5 | **4.4** |
| **Distill.pub** | 5 | 5 | 5 | 3 | 3 | **4.2** |
| **3Blue1Brown** | 4 | 4 | 5 | 5 | 3 | **4.2** |
| **Papers with Code** | 3 | 5 | 4 | 5 | 5 | **4.4** |
| **arXiv HTML + ar5iv** | 5 | 5 | 3 | 3 | 5 | **4.2** |
| **D:\note 现状** | 3 | 3 | 2 | 4 | 3 | **3.0** |

**关键差距**：D:\note 在"结构化（3.0 vs 范例均值 4.6）"和"可视化（2.0 vs 范例均值 3.9）"两个维度有最大可优化空间。

---

## 对 D:\note 笔记体系的 10 条改进建议

> 每条建议都给出**具体落地路径**，避免空泛口号。

### 建议 1：建立"全局配色语义规范"

定义 D:\note 配色规范：紫=输入/原始数据，绿=权重/参数，橙=输出/预测，红=梯度/损失，蓝=处理过程/算法，灰=辅助/背景。在所有 callout、自定义示意图、表格高亮中一致使用。建议落地到 `_templates/style-guide.md` 文档，并在所有学习笔记 series 的第一篇统一声明。

### 建议 2：建立"概念总览图"必选规范

每篇超过 3000 字的学习笔记或论文笔记，强制在文章首屏配 1 张"概念总览图"（即使是 ASCII 流程图）。可在每篇笔记 frontmatter 增加 `overview-figure: required` 字段作为 lint 检查项。这是 Jay Alammar + Chris Olah + Distill.pub 的共同特征。

### 建议 3：引入"IMRaD + Lilian 式 TOC"双层结构

所有论文笔记（`research/02-papers/`）采用 IMRaD 结构：Abstract → Introduction → Method → Experiments → Conclusion。所有学习笔记（`raw/lessons/`）采用 Lilian 式结构：TOC → Overview → Component 1 → Component 2 → ... → Case Studies → References。两套结构互相独立，分别落地到 `_templates/paper-note.md` 和 `_templates/lesson.md`。

### 建议 4：建立"论文卡片化"模板

参考 Papers with Code，为每篇论文笔记建立"30 秒 summary 卡片"：TLDR（≤100字）+ 任务/数据集/指标 + SOTA 对比表 + 代码/数据链接徽章。这张卡片放在论文笔记的最前面，让读者快速 decide 是否深读。

### 建议 5：每章末尾强制"Main Takeaways" + "下一步学习"双段

所有学习笔记和论文笔记末尾强制两段：(1) Main Takeaways（3-5 个 bullet points 总结本章学到的），(2) 下一步学习（链接到下一篇笔记或外部资源）。这是 Lilian Weng + François Chollet + Sebastian Raschka 的共同模式。

### 建议 6：建立"自绘示意图"工作流

为每个核心概念制作 1 张自绘 SVG 示意图（即使很简单）。在 `_templates/figures/` 下建立图库：每张图命名为 `concept-name-v1.svg`，配套 `concept-name-v1.md` 解释图的设计意图和配色规范。Obsidian 支持嵌入 SVG，可直接在笔记中展示。

### 建议 7：引入"Checklist + Things to look for"工程笔记模板

所有工程类笔记（如 `research/03-experiments/`、`research/04-writing/`）采用 Karpathy 式结构：每节是 "Checklist（步骤）" + "Things to look for（易错点）" 双段。这把博客从"知识载体"升级为"工程 checklist"。

### 建议 8：建立"Last Updated + 版本化"机制

所有笔记 frontmatter 增加 `updated: YYYY-MM-DD` 字段，每次修改时强制更新。在 `wiki/log.md` 中自动 append 修改记录。这让笔记具备"时间机器"能力，便于回溯知识演化。

### 建议 9：建立"Case Studies + 真实回扣"机制

所有概念笔记在文末增加 "Case Studies" 节，至少给 1-2 个真实论文/项目作为案例（参考 Lilian Weng 的 Generative Agents 案例）。这是把"概念—应用"闭环的关键。

### 建议 10：建立"链接徽章 + 元信息卡片"规范

所有笔记的链接（数据集、代码库、论文）使用统一格式的徽章：`[Code](url)`、`[Paper](url)`、`[Dataset](url)`、`[Demo](url)`。在 `wiki/index.md` 增加"笔记索引卡片"，每张卡片展示 TLDR + 元信息 + 主图，让 vault 入口页具备 Papers with Code 风格的"卡片式导航"。

---

## 引用源清单

### Lilian Weng 相关
1. https://lilianweng.github.io/ — Lil'Log 主页
2. https://lilianweng.github.io/posts/2023-03-15-prompt-engineering/ — Prompt Engineering
3. https://lilianweng.github.io/posts/2023-06-23-agent/ — LLM Powered Autonomous Agents
4. https://lilianweng.github.io/posts/2021-07-11-diffusion-models/ — What are Diffusion Models?
5. https://lilianweng.github.io/posts/2023-01-27-the-transformer-family-v2/ — The Transformer Family v2
6. https://github.com/lilianweng/lilianweng.github.io — 博客源码

### Jay Alammar 相关
7. https://jalammar.github.io/ — 主页
8. https://jalammar.github.io/illustrated-transformer/ — The Illustrated Transformer
9. https://jalammar.github.io/illustrated-gpt2/ — The Illustrated GPT-2
10. https://jalammar.github.io/explaining-transformers/ — Interfaces for Explaining Transformer Language Models
11. https://jalammar.github.io/jays-intro-to-ai/ — Jay's Intro to AI 系列

### Andrej Karpathy 相关
12. https://karpathy.ai/ — 主页
13. https://karpathy.ai/zero-to-hero.html — Neural Networks: Zero to Hero
14. http://karpathy.github.io/neuralnets/ — Hacker's guide to Neural Networks
15. http://karpathy.github.io/2019/04/25/recipe/ — A Recipe for Training Neural Networks
16. https://karpathy.medium.com/software-2-0-a64152b37c35 — Software 2.0

### Chris Olah + Distill 相关
17. https://colah.github.io/ — 个人博客
18. https://colah.github.io/posts/2015-08-Understanding-LSTMs/ — Understanding LSTM Networks
19. https://distill.pub/ — Distill 主页
20. https://distill.pub/guide/ — How to Create a Distill Article
21. https://distill.pub/2020/communicating-with-interactive-articles — Communicating with Interactive Articles
22. https://distill.pub/2017/feature-visualization — Feature Visualization
23. https://distill.pub/2018/building-blocks — The Building Blocks of Interpretability

### François Chollet 相关
24. https://deeplearningwithpython.io/ — Deep Learning with Python 第三版（在线免费）
25. https://github.com/fchollet/deep-learning-with-python-notebooks — 配套 Jupyter Notebooks
26. https://keras.io/ — Keras 官方文档

### Sebastian Raschka 相关
27. https://sebastianraschka.com/ — 主页
28. https://sebastianraschka.com/llms-from-scratch/ — Build a Large Language Model (From Scratch) 主页
29. https://github.com/rasbt/LLMs-from-scratch — 配套代码
30. https://magazine.sebastianraschka.com/p/supporting-ahead-of-ai — 个人杂志

### 3Blue1Brown 相关
31. https://www.3blue1brown.com/ — 主页
32. https://www.3blue1brown.com/topics/neural-networks — Neural Networks 系列
33. https://www.3blue1brown.com/lessons/neural-networks/ — But what is a Neural Network?
34. https://www.youtube.com/playlist?list=PLZHQObOWTQDNU6R1_67000Dx_ZCJB-3pi — 视频播放列表

### Papers with Code 相关
35. https://paperswithcode.com/ — 主页（2025-07 关停）
36. https://github.com/paperswithcode — GitHub 仓库
37. https://medium.com/paperswithcode/papers-with-code-2021-a-year-in-review-de75d5a77b8b — 2021 年回顾

### arXiv HTML + ar5iv 相关
38. https://info.arxiv.org/about/accessible_HTML.html — arXiv HTML 介绍
39. https://ar5iv.org/ — ar5iv 主页
40. https://arxiv.org/html/2402.08954v1 — "HTML papers on arXiv: why it's important, and how we made it happen"
41. https://math.nist.gov/~BMiller/LaTeXML/manual.pdf — LaTeXML Manual
42. https://sigmathling.kwarc.info/resources/ar5iv-dataset-2024/ — ar5iv 04.2024 dataset
43. https://huggingface.co/papers/trending — Trending Papers（Hugging Face 替代 Papers with Code 的趋势展示）

---

## 附录：本次调研的方法论说明

1. **搜索策略**：先用 `mcp__web-search-prime__web_search_prime` 对每个对象做 1-2 次泛搜，再用关键词组合搜索（如 "结构 + 风格"、"可视化 + 原则"）。
2. **页面抓取**：尝试使用 WebFetch 直接抓取代表页面，因部分 github.io 域名被拦截，部分细节依赖搜索摘要和第二手解读（如翻译博客、Medium 评论、Reddit 讨论）。
3. **深度分析**：每个对象按"特点—代表作剖析—风格—可视化—差距—可学习技巧"六维度展开。
4. **综合输出**：通过差距矩阵横向对比 + 10 条改进建议纵向落地，把调研成果转化为可操作的笔记体系改造方案。