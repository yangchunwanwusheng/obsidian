---
type: concept
tags: [深度学习, Transformer, 层归一化, Layer Normalization, LayerNorm]
created: 2026-04-09
updated: 2026-04-09
sources:
  - raw/lessons/Transformer/05-ffn-residual-layernorm.md
---

# Layer Normalization（层归一化）

> 对单个样本的所有特征维度做归一化，将每层输出稳定到均值 0、方差 1 的分布，为深层网络提供数值稳定性。

## 概述

Layer Normalization（LayerNorm）是 Transformer 中的"格式统一员"。在每一层输出经过 [[residual-connection|残差连接]] 的加法操作后，数值范围可能发生漂移。LayerNorm 将其重新标准化到稳定分布，确保后续层能在一致的数值空间中工作。

与 BatchNorm 跨样本归一化不同，LayerNorm 逐样本归一化——每个 token 独立地根据自身向量的统计量做标准化，不依赖 batch 中其他样本。这一特性使它天然适配变长序列，成为 NLP 领域的首选归一化方法。

## 核心公式

对单个样本向量 $x \in \mathbb{R}^{d_{model}}$：

$$LayerNorm(x) = \gamma \cdot \frac{x - \mu}{\sqrt{\sigma^2 + \epsilon}} + \beta$$

逐步拆解：
1. $\mu$ — 沿特征维度计算均值，反映该 token 表示的"整体大小"
2. $\sigma^2$ — 沿特征维度计算方差，反映内部数值的一致性程度
3. $(x - \mu) / \sqrt{\sigma^2 + \epsilon}$ — 标准化到均值 0、方差 1，消除量纲差异
4. $\gamma \cdot \hat{x} + \beta$ — 可学习的仿射变换，让模型保留调整尺度和平移的自由度

其中 $\epsilon$（通常 $10^{-6}$）是防止除零的小常数，$\gamma$ 和 $\beta$ 是可学习参数。

## 与 BatchNorm 的关键区别

| 特性 | Batch Normalization | Layer Normalization |
|------|--------------------|--------------------|
| 归一化方向 | 沿 Batch 方向（跨样本的同一特征） | 沿特征方向（同一样本的所有特征） |
| 统计量依赖 | 依赖当前 Batch 的均值和方差 | 仅依赖当前单个样本 |
| Batch Size 影响 | 强依赖，小 batch 统计不稳定 | 完全无关 |
| 训练/推理行为 | 训练用 batch 统计，推理用滑动平均 | 完全一致 |
| 对变长序列 | 欠佳（padding 影响统计量） | 天然适配（逐 token 独立归一化） |
| 适用领域 | 计算机视觉 | 自然语言处理 |

NLP 选择 LayerNorm 的核心原因：不同句子的长度差异导致 batch 内 padding 不一致，BatchNorm 的跨样本统计会被 padding 噪声污染；LayerNorm 对每个 token 独立操作，完全规避了这个问题。

## Pre-LN vs Post-LN

原始 Transformer 使用 Post-LN（后置归一化），现代模型普遍采用 Pre-LN（前置归一化）。

- **Post-LN**：`x -> Sublayer -> Add -> LayerNorm`（归一化在残差分支末端）
- **Pre-LN**：`x -> LayerNorm -> Sublayer -> Add`（归一化在残差分支开头）

Pre-LN 的优势在于梯度流更均衡——归一化在子层输入端完成，梯度可以通过残差路径无损回传到所有层。Post-LN 虽然理论上表达能力更强，但需要 warm-up 等技巧才能稳定训练。GPT-2 及后续的大语言模型几乎全部采用 Pre-LN。

## 要点

- LayerNorm 逐样本、逐特征维度归一化，不依赖 batch 统计，训练与推理行为一致
- 核心作用是稳定深层网络的数值分布，配合 [[residual-connection|残差连接]] 构成 "Add & Norm"
- 相比 BatchNorm，天然适配变长序列，不受 padding 和 batch size 影响
- $\gamma$ 和 $\beta$ 是可学习参数，让模型在标准化的基础上保留尺度调整的自由度
- Pre-LN 是现代主流，训练稳定性显著优于原始论文的 Post-LN 设计

## 相关页面

- [[wiki/concepts/residual-connection|残差连接]] — 与 LayerNorm 配合构成 "Add & Norm" 结构
- [[wiki/concepts/feed-forward-network|Feed Forward Network]] — LayerNorm 作用的关键位置之一
- [[wiki/concepts/self-attention|Self-Attention]] — LayerNorm 作用的另一个关键位置
