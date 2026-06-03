---
type: lesson
tags: [MUG, 翻译笔记, 相关工作, 多模态推理, 多智能体系统, 反事实推理]
created: 2026-05-16
updated: 2026-05-16
sources:
  - MUG_MultiAgentUndercoverGaming_2025.pdf
difficulty: advanced
prerequisites:
  - "[[raw/lessons/MUG/00-essence]]"
topic: MUG — 相关工作
status: completed
---

# MUG — 相关工作 (Related Work)

> 从多模态推理到多智能体系统再到反事实推理，MUG 的理论根基横跨三大研究方向。

## 正文翻译

### 多模态推理

早期的多模态推理工作聚焦于对齐视觉和文本表示，这推动了多模态大语言模型（Multimodal LLMs, MLLMs）的发展 (Liu et al. 2024; IJCAI 2025)，以及上下文学习（In-Context Learning, ICL）(Min et al. 2022) 和思维链（Chain-of-Thought, CoT）推理 (Wei et al. 2022) 的采用。

这些技术使模型能够将复杂任务分解为可解释的中间步骤，增强了透明度和性能。

CoT 向多模态上下文的演进，被称为多模态思维链（Multimodal Chain-of-Thought, MCoT）推理，产生了一系列架构，从线性表示 (Wei et al. 2022; ChongwahNgo 2008) 到基于图的表示 (Besta et al. 2024; Yuan et al. 2023)，以及针对不同模态的专门方法，如 Multimodal-CoT (Zhang et al. 2024)、MVoT (Li et al. 2025) 和 Video-of-Thought (Fei et al. 2024)。

> [!tip] 理解提示（非原文）
> 多模态推理的演进路径：从简单的视觉-文本对齐 → CoT（让模型"一步一步想"）→ MCoT（把 CoT 扩展到多模态场景）。这条发展线说明了一个趋势：研究者越来越关注如何让模型在多模态输入下进行可解释的逐步推理，而非端到端的"黑箱"判断。

### 多智能体系统

多智能体系统（Multi-Agent Systems, MASs）(Wooldridge 2009) 的范式已经获得广泛关注，作为克服单个 LLM 内在局限性（如幻觉和有限推理深度）的手段。

MAS 利用多个基于 LLM 的智能体的集体智慧，实现分布式知识保留 (Zheng et al. 2024)、长期规划 (Torreno et al. 2017) 和通过协作问题解决的专业化 (Lu et al. 2023; Wang et al. 2025)。

然而，现有的 MAD 框架经常会过早收敛（premature convergence），压制少数观点（suppressing minority viewpoints），并且无法模拟现实世界中的对抗性推理条件。

> [!note] 译者注（非原文）
> "过早收敛"是多智能体辩论的经典问题：当多个 LLM 智能体辩论时，它们往往会快速达成一致（尤其是使用相同基础模型时），但这种"共识"不一定正确——可能只是模型偏差导致的集体幻觉。MUG 通过引入信息不对称（给不同智能体不同的图像）来打破这种过早收敛，迫使智能体基于不同的视觉证据进行推理。

### 反事实推理

反事实推理（Counterfactual Reasoning）植根于结构因果模型（Structural Causal Models）和 do-演算（do-calculus）(Pearl 2009; ChongwahNgo 2007)，对于理解在假设干预下结果如何变化至关重要。

在 AI 中，反事实通过使模型能够生成替代场景并在不确定性下评估决策，增强了可解释性和鲁棒性 (Guidotti et al. 2024; Wang et al. 2024)。

最近的工作已经将反事实推理扩展到 LLM，方法涵盖从基于常识的场景生成 (Zhang et al. 2024; Chatzi et al. 2025) 到使用外部工具的基于图的因果推断，如 CausalCoT (Jin et al. 2023) 和 CausalTool (Hua et al. 2024)。

虽然像 Causal Agent (Han et al. 2024) 这样的基于智能体的框架将 LLM 与因果工具集成用于干预分析，但它们通常缺乏显式的反事实推理，限制了在复杂、高维设置中的适用性。

### 本文的独特贡献

**本文通过将反事实学习直接嵌入多智能体协作过程中来区别于现有工作，增强了多模态推理的鲁棒性和可解释性。**

> [!warning] 易混点（非原文）
> 需要区分"反事实推理"在因果推断和 MUG 中的不同含义：
> - 在因果推断中，"反事实"指的是"如果 X 没有发生，Y 会怎样"——一种关于因果关系的推理
> - 在 MUG 中，"反事实"指的是"如果图像中的某个元素被修改了，智能体的回答会怎样"——一种通过修改输入来测试智能体可靠性的手段
> 两者有联系但不完全相同：MUG 更侧重于利用反事实作为测试工具，而非严格的因果推断
