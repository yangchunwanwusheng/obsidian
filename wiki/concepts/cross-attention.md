---
type: concept
tags: [深度学习, Transformer, Cross-Attention, 交叉注意力, 编码器-解码器]
created: 2026-04-09
updated: 2026-04-09
sources:
  - raw/lessons/Transformer/06-encoder-decoder.md
---

# Cross-Attention（交叉注意力）

> 连接编码器与解码器的桥梁——让解码器在生成每个 token 时，能够"回头参考"编码器对输入序列的理解。

## 概述

Cross-Attention 是 [[wiki/concepts/encoder-decoder|Encoder-Decoder]] 架构中最核心的通信机制。它出现在解码器的第二个子层，负责将解码器的生成状态与编码器的输入表示进行对齐。与 Self-Attention 不同，Cross-Attention 的 Query 和 Key/Value 来自不同的序列，因此它本质上是一个**跨序列的信息检索过程**。

形象地说，解码器在输出第 $t$ 个词时，通过 Cross-Attention 向编码器"提问"："我当前应该关注输入序列的哪个位置？"编码器则通过 Key/Value "回答"："你应该关注这里的信息。"这正是翻译模型能够实现词对齐的底层机制。

## 核心设计：Q 来自解码器，K/V 来自编码器

Cross-Attention 的公式与标准注意力相同，但参与运算的矩阵来源不同：

$$\text{CrossAttn}(Q_d, K_e, V_e) = \text{softmax}\left(\frac{Q_d \cdot K_e^T}{\sqrt{d_k}}\right) V_e$$

其中：
- $Q_d$ 来自解码器，形状 $\mathbb{R}^{n_{dec} \times d_k}$（$n_{dec}$ 为解码器序列长度）
- $K_e, V_e$ 来自编码器输出，形状 $\mathbb{R}^{n_{enc} \times d_k}$ 和 $\mathbb{R}^{n_{enc} \times d_v}$

**关键洞察**：注意力矩阵 $A \in \mathbb{R}^{n_{dec} \times n_{enc}}$ 不是方阵。每一行代表解码器某个位置对编码器所有位置的关注分布，意味着解码器的每个 token 可以独立决定"看输入的哪里"。

## Cross-Attention vs Self-Attention

| 维度 | Self-Attention | Cross-Attention |
|------|---------------|-----------------|
| Q 来源 | 同一序列自身 | 解码器序列 |
| K 来源 | 同一序列自身 | **编码器输出** |
| V 来源 | 同一序列自身 | **编码器输出** |
| 注意力矩阵 | $n \times n$ 方阵 | $n_{dec} \times n_{enc}$ 矩形矩阵 |
| 信息方向 | 序列内部互通 | 跨序列信息流动 |
| 所在位置 | 编码器 + 解码器 | 仅解码器第二个子层 |

## 要点

- **信息桥梁**：Cross-Attention 是编码器信息传递到解码器的唯一通道，没有它解码器将完全不知道输入内容。
- **灵活对齐**：解码器的每个位置独立学习关注编码器的哪些位置，天然支持软对齐（soft alignment），无需手动设计对齐规则。
- **与多头的协同**：结合 [[wiki/concepts/multi-head-attention|Multi-Head Attention]]，不同的头可以关注输入的不同方面（如一个头关注语法对齐，另一个头关注语义对齐）。
- **Decoder-Only 模型没有它**：GPT 等 Decoder-Only 模型去掉了编码器，自然也不需要 Cross-Attention，仅依赖 [[wiki/concepts/masked-attention|Masked Self-Attention]]。

## 相关页面

- [[wiki/concepts/encoder-decoder|编码器-解码器架构]]
- [[wiki/concepts/self-attention|Self-Attention]]
- [[wiki/concepts/masked-attention|Masked Self-Attention]]
- [[wiki/concepts/multi-head-attention|Multi-Head Attention]]
