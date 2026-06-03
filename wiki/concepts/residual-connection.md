---
type: concept
tags: [深度学习, Transformer, 残差连接, Residual Connection, Skip Connection]
created: 2026-04-09
updated: 2026-04-09
sources:
  - raw/lessons/Transformer/05-ffn-residual-layernorm.md
---

# 残差连接（Residual Connection）

> 一条绕过非线性变换的"梯度高速公路"，让深度网络的每一层只需学习相对输入的微小偏差，而非完整的映射函数。

## 概述

残差连接（又称跳跃连接，Skip Connection）是深度学习中最重要的技巧之一。在 Transformer 中，每个子层（Self-Attention 或 FFN）的输出都与原始输入相加，形成 `output = x + Sublayer(x)` 的结构。这条"短路"路径确保了：

1. **梯度可以直接回传**——反向传播时，Add 节点的偏导数为 1，梯度无需经过子层的复杂变换即可流向浅层。
2. **原始信息不丢失**——即使子层训练不充分，输入仍能完整传递到下游。
3. **深层网络可训练**——没有残差，层数越多性能反而下降（如 110 层 ResNet 无残差时差于 20 层）。

残差连接最早由 He et al. (2016) 在 ResNet 中提出，用于解决图像领域的梯度消失问题。Transformer 直接借鉴了这一思想。

## 核心公式

Transformer 中残差连接与 LayerNorm 配合使用，构成 "Add & Norm" 结构：

$$Output = LayerNorm(x + Sublayer(x))$$

其中 $Sublayer(x)$ 可以是 Self-Attention 或 FFN。

## 关键直觉："Easy to Learn" 原则

假设网络的最优映射接近恒等映射 $H(x) = x$：

- **无残差**：网络需要直接拟合 $F(x) = x$，在多维空间中精确学到恒等映射很困难。
- **有残差**：网络只需拟合 $F(x) = H(x) - x$，即"相对于输入的偏差"。当最优解接近恒等时，让 $F(x) \approx 0$ 远比让 $F(x) \approx x$ 容易。

这就是"easy to learn"原则——学习残差（偏差）比学习完整映射简单得多。

## 与 ResNet 的关系

两者本质相同：提供一条绕过非线性变换的梯度高速公路。

- **ResNet**：`输入 x -> Linear -> ReLU -> Linear -> F(x) + x`（用于图像分类）
- **Transformer**：`输入 x -> Sublayer(x) -> F(x) + x`（用于序列建模）

Transformer 将 ResNet 的残差思想从卷积网络迁移到了注意力架构中，并用 [[layer-normalization]] 替代了 BatchNorm 来配合残差使用。

## 要点

- 残差连接让网络学习"偏差"而非"完整映射"，大幅降低了深层网络的训练难度
- 反向传播时 Add 节点提供梯度为 1 的直通路径，有效缓解梯度消失
- 去掉残差会导致训练不稳定、深层无效、信息丢失三重问题
- 残差连接与 [[layer-normalization]] 组成 "Add & Norm"，是 Transformer 的标志性结构
- 现代趋势采用 Pre-LN（先归一化再做子层），梯度流更均衡，训练更稳定

## 相关页面

- [[wiki/concepts/layer-normalization|Layer Normalization]] — 与残差连接配合的归一化方法
- [[wiki/concepts/feed-forward-network|Feed Forward Network]] — 残差连接包裹的子层之一
- [[wiki/concepts/self-attention|Self-Attention]] — 残差连接包裹的另一个子层
