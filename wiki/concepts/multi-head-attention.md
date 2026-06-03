---
type: concept
tags: [深度学习, Transformer, 多头注意力, Multi-Head Attention]
created: 2026-04-09
updated: 2026-04-09
sources:
  - raw/lessons/Transformer/03-multi-head-attention.md
---

# Multi-Head Attention（多头注意力）

> 将单个高维 Self-Attention 拆分为多个低维并行头，每个头在独立的子空间中捕捉不同类型的依赖关系，拼接后再通过线性投影融合——让模型同时拥有多个"专家视角"。

## 概述

Multi-Head Attention 是对 [[wiki/concepts/self-attention|Self-Attention]] 的直接扩展。单头 Self-Attention 只有一组 Q/K/V 投影矩阵，只能捕捉一种"关注模式"——它必须在语法关系、语义关联、指代消解等多种依赖类型之间妥协。Multi-Head Attention 的解决方案是：将总维度 $d_{model}$ 平均拆分给 $h$ 个头，每个头拥有独立的投影矩阵，在自己的低维子空间中专注一种模式。

标准配置（$d_{model}=512, h=8, d_k=64$）下，8 个头各自计算 64 维的 Attention，拼接后恢复为 512 维，再通过输出投影矩阵 $W^O$ 做跨头信息融合。这种设计的精妙之处在于：总计算量与单头 512 维 Self-Attention 相近，但表达力显著增强。

## 核心公式

$$\text{MultiHead}(Q, K, V) = \text{Concat}(head_1, \ldots, head_h) \cdot W^O$$

$$head_i = \text{Attention}(QW_i^Q, \; KW_i^K, \; VW_i^V)$$

**自然语言解读**：第 $i$ 个头用自己的投影矩阵 $W_i^Q, W_i^K, W_i^V$ 将输入映射到自己的 $d_k$ 维子空间，然后执行标准的缩放点积注意力。$h$ 个头的输出在特征维度上拼接（每个 $d_k$ 维，共 $h \cdot d_k = d_{model}$ 维），最后通过 $W^O$ 做线性变换，学习如何最优地组合不同头的信息。

**维度流转**：输入 $(n, d_{model}) \to$ 每个头投影为 $(n, d_k) \to$ 各自计算 Attention 输出 $(n, d_k) \to$ 拼接为 $(n, h \cdot d_k) = (n, d_{model}) \to W^O$ 投影为 $(n, d_{model})$。

## 关键设计决策

**为什么拆分成多个低维头？** 一个 512 维的单头空间中，语法信息、语义信息、指代信息必须竞争同一个投影子空间。将 512 维拆成 8 个 64 维的独立子空间后，每个头可以专注于一种类型的信息而互不干扰——实验表明 Transformer 确实学到了功能特化的头（部分头关注语法，部分头关注指代等）。

**总维度不变的哲学**：$h$ 个 $d_k$ 维头的总维度 $h \cdot d_k = d_{model}$，Multi-Head Attention 不增加总计算量（相对于单头 $d_{model}$ 维），而是将同样的计算预算分配到更多样化的子空间中。这是"用相同成本换取表达多样性"的工程智慧。

**$W^O$ 的作用**：拼接只是简单地把 8 个头的输出堆叠在一起，$W^O$ 才是让不同头信息真正融合的关键——它学习如何对多个头的输出做加权组合，而非简单的平均。

## 与单头 Self-Attention 的区别

| 特征 | 单头 Self-Attention | Multi-Head Attention |
|------|--------------------|--------------------|
| 投影矩阵 | 一组 $W_Q, W_K, W_V$ | $h$ 组独立的 $W_i^Q, W_i^K, W_i^V$ |
| Q/K/V 维度 | $d_{model}$ | $d_k = d_{model}/h$ |
| 关注模式 | 只能捕捉一种 | 多种并行 |
| 输出融合 | 直接输出 | Concat + $W^O$ 跨头融合 |
| 参数量 | $3 \times d_{model}^2$ | $4 \times d_{model}^2$（略多，因多了 $W^O$） |
| 本质区别 | 单一视角审视序列 | 多视角并行审视，最后综合 |

## 要点

- **单头局限**：一组投影矩阵无法同时最优地支持语法、语义、指代等多种依赖关系——这是多头的根本动机
- **拆分而非堆叠**：Multi-Head 不是 8 个独立的 Self-Attention 各自为战，而是通过 Concat + $W^O$ 实现跨头信息交互
- **功能特化是涌现的**：不同头关注不同类型的信息是训练中自然涌现的，并非预先指定；部分头可能冗余或几乎不激活
- **头数并非越多越好**：$h$ 过大会使每个头的维度 $d_k$ 过小，反而限制表达能力；标准 Transformer 用 $h=8$ 是经验最优值
- **输出是拼接而非平均**：拼接保留了每个头的独立信息，$W^O$ 负责学习最优组合方式

## 相关页面

- [[wiki/concepts/self-attention|Self-Attention]] — Multi-Head Attention 的基础构件
- [[wiki/entities/Transformer|Transformer]] — 以 Multi-Head Attention 为核心的完整架构
- [[wiki/concepts/encoder-decoder|编码器-解码器]] — Multi-Head 在编码器（Self-Attention）和解码器（Masked Self-Attention + Cross-Attention）中的不同用法
