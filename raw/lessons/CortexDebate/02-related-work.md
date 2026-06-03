---
type: lesson
tags: [CortexDebate, 翻译笔记, 多智能体辩论, 相关工作]
created: 2026-05-16
updated: 2026-05-16
sources:
  - CortexDebate_2025.pdf
difficulty: advanced
prerequisites:
  - "[[raw/lessons/CortexDebate/00-essence]]"
topic: CortexDebate — 相关工作
status: completed
---

# CortexDebate — 相关工作 (Section 2: Related Work)

> 本篇翻译论文第 2 节 Related Work 的内容。

## 正文翻译

在 MAD 系统中，每个 LLM 智能体提出自己的观点，并在多轮辩论中审视其他 LLM 智能体的观点 (Sun et al., 2024a)。总结来说，现有的 MAD 方法可以分为两类，即**顺序辩论 (Sequential Debate)** 和**并行辩论 (Parallel Debate)**。

### 顺序辩论 (Sequential Debate)

在这些方法中 (Hu et al., 2025; Brown-Cohen et al., 2023; Michael et al., 2023; Wang et al., 2025; He et al., 2024)，LLM 智能体按顺序生成观点。每个 LLM 智能体只能获取其前面的 LLM 智能体的观点。

例如，Liang et al. (2023) 要求两个 LLM 轮流反驳对方。除了辩手之外，Guan et al. (2025) 还添加了额外角色，如法官和评论家。法官在辩手之前发言解释任务，评论家最后发言总结辩论。

然而，在顺序辩论系统中，每个 LLM 智能体必须等待前面的智能体完成推理后才能开始。这导致辩论时间随 LLM 智能体数量线性增长，造成低效率，限制了可扩展性。

> [!tip] 理解提示（非原文）
> 顺序辩论就像"轮流发言"——A 说完 B 说，B 说完 C 说。虽然信息传递清晰（每个人都能看到前面所有人的发言），但效率很低：5 个智能体需要 5 倍的串行时间。这也是为什么 CortexDebate 选择并行辩论框架。

### 并行辩论 (Parallel Debate)

在这些方法中 (Pham et al., 2023; Yin et al., 2023; Chern et al., 2024; Khan et al., 2024; Liang et al., 2024; Li et al., 2024a; Hegazy, 2024; Zhang et al., 2024b)，所有 LLM 智能体基于上一辩论轮次中其他智能体的观点，同时生成自己的观点。

例如，Chan et al. (2023) 要求 LLM 智能体批判上一辩论轮次中生成的所有答案，并在每一轮同时更新自己的答案。除了上一轮生成的答案外，Sun et al. (2024b) 还为每个 LLM 智能体提供从网络检索到的与任务相关的信息。此外，一些方法 (Duan and Wang, 2024; Yoffe et al., 2024) 尝试调整每个 LLM 智能体的辩论影响力以提高辩论效果。例如，Chen et al. (2023) 要求每个 LLM 智能体为自己的答案生成置信度分数，然后将该分数与答案一起输入给其他 LLM 智能体。

### CortexDebate 与现有方法的区别

由于顺序辩论系统面临上述低效率问题，我们的 CortexDebate 遵循并行辩论框架。与要求每个 LLM 智能体在每轮中与所有其他智能体辩论的现有并行辩论方法相比，CortexDebate 通过在所有参与的 LLM 智能体之间建立**稀疏辩论图**来动态决定必要的辩论对手，从而缩短每个智能体的输入上下文。

这也有别于 Liu et al. (2024b) 和 Li et al. (2024b) 中的方法，后者的辩论对手是**固定的**。此外，与仅根据自身置信度确定每个 LLM 智能体辩论影响力的先前方法不同，我们引入了 McKinsey Trust Formula，使得每个 LLM 智能体的置信度和对其辩论组件的有用性都能得到评估。

> [!note] 译者注（非原文）
> 这里有一个重要的三分法对比：
> - **全辩论**（MLD, RECONCILE, ChatEval, PRD）：每个智能体与所有其他智能体辩论 → 连接太密集，上下文过长
> - **固定部分辩论**（GD, ND）：辩论对手是预先固定的 → 无法根据实际表现动态调整
> - **动态稀疏辩论**（CortexDebate）：通过 MDM 模块动态评估并选择最有价值的辩论对手 → 兼顾效率与效果

### 现有 MAD 方法分类汇总

| 类别 | 方法 | 特点 | 局限 |
|------|------|------|------|
| 顺序辩论 | Liang et al. (2023) | 两个 LLM 轮流反驳 | 时间随智能体数线性增长 |
| 顺序辩论 | Guan et al. (2025) | 添加法官和评论家角色 | 同上 |
| 并行辩论 | Chan et al. (2023) ChatEval | 每轮同时批判所有答案并更新 | 全连接 → 上下文过长 |
| 并行辩论 | Chen et al. (2023) RECONCILE | 生成置信度分数辅助辩论 | 仅依赖自身置信度 → 过度自信 |
| 并行辩论 | Liu et al. (2024b) GD | 分组辩论 | 对手固定，不能动态调整 |
| 并行辩论 | Li et al. (2024b) ND | 仅与邻居辩论 | 拓扑固定，缺乏优化 |
| **本文** | **CortexDebate** | **动态稀疏辩论图 + McKinsey Trust Formula** | **—** |

## 关键要点回顾

- MAD 方法分为顺序辩论和并行辩论两类
- 顺序辩论效率低，因此 CortexDebate 采用并行辩论框架
- 现有并行辩论方法要么全连接（上下文过长），要么固定拓扑（缺乏优化）
- CortexDebate 的核心优势：动态稀疏 + 可信度评估

## 下一步学习

- [[raw/lessons/CortexDebate/03-method|03 - 稀疏辩论图方法]]
