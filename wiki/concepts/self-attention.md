---
type: concept
tags: [深度学习, Transformer, 自注意力, Self-Attention, QKV, 点积注意力]
created: 2026-04-09
updated: 2026-04-09
sources:
  - raw/lessons/Transformer/02-self-attention.md
  - raw/lessons/Transformer/01-why-transformer.md
---

# Self-Attention（自注意力）

> 序列中每个位置同时作为"提问者"和"被提问者"，通过学习到的投影矩阵自动发现并整合序列内部最相关的信息——这就是 Transformer 的核心引擎。

## 概述

Self-Attention 是 Transformer 架构中最关键的机制。与传统 Attention 不同，它的 Query、Key、Value 全部来自**同一个输入序列**，使序列能够在内部完成"自我对齐"——每个 token 既是信息的检索者，也是信息的提供者。

其核心思想借鉴了信息检索系统：每个 token 通过一组可学习的投影矩阵 $W_Q, W_K, W_V$，将自身映射到三个不同的语义空间——"我在找什么"（Query）、"我是什么类型"（Key）、"我携带什么内容"（Value）。然后通过 Query 与所有 Key 的匹配来确定关注程度，最终用加权求和整合所有 Value。

整个计算可以用一次矩阵乘法完成（$QK^T$），这赋予了 Self-Attention 天然的并行性——无需像 RNN 那样逐步传递信息，所有位置同时计算。

## 核心公式

$$\text{Attention}(Q, K, V) = \text{softmax}\left(\frac{QK^T}{\sqrt{d_k}}\right)V$$

**自然语言解读**：将输入分别投影为 Q、K、V 三个矩阵；Q 与 K 的转置做点积得到"谁关注谁"的原始分数矩阵（$n \times n$）；除以 $\sqrt{d_k}$ 进行缩放以防梯度消失；对每行做 Softmax 归一化为概率分布；最后用注意力权重对 V 加权求和，得到每个位置融合了全局信息的输出。

**维度流转**：$(n \times d_k) \cdot (d_k \times n) \to (n \times n) \xrightarrow{\text{softmax}} (n \times n) \cdot (n \times d_v) \to (n \times d_v)$

## 要点

- **三角色投影**：$W_Q, W_K, W_V$ 将同一输入映射到"提问"、"被检索"、"内容"三个语义空间，使模型能学到同一个词在不同角色下的不同表示
- **缩放因子的本质**：$\sqrt{d_k}$ 的作用不是防止数值溢出，而是将点积方差标准化为 1，防止高维空间中 Softmax 梯度消失——当 $d_k=64$ 时，未缩放的点积标准差为 8，Softmax 输出会退化为 one-hot
- **全局感受野**：单层 Self-Attention 中每个 token 可以直接看到序列中所有其他 token，无需像 RNN 那样逐步传递
- **注意力是加权平均而非选择**：Softmax 输出是概率分布，会综合所有位置的信息，而非只看最相关的一个

## 与传统 Attention 的区别

| 特征 | 传统 Attention（Bahdanau） | Self-Attention |
|------|--------------------------|----------------|
| Q 的来源 | 解码器的隐藏状态（外部） | 同一输入序列 |
| K/V 的来源 | 编码器的隐藏状态 | 同一输入序列 |
| 对话模式 | 跨序列对话 | 序列内部自我审视 |
| 相似度计算 | 加性注意力（tanh + 线性） | 缩放点积（矩阵乘法） |
| 并行性 | 受限于解码顺序 | 完全并行 |

## 关键性质

**置换等变性**：如果打乱输入序列的顺序，输出只是对应顺序的置换——Self-Attention 本身无法区分位置顺序。这就是为什么 Transformer 需要位置编码（Positional Encoding）来注入顺序信息。

**并行性**：$QK^T$ 是一次矩阵乘法，所有位置的注意力分数同时计算，这是 Transformer 训练效率的根本来源。

**$O(n^2)$ 复杂度**：注意力分数矩阵是 $n \times n$ 的，序列长度翻倍则计算量翻四倍。这是 Self-Attention 处理长序列时的主要瓶颈，也是 Flash Attention 等优化技术存在的原因。

## 相关页面

- [[wiki/concepts/multi-head-attention|Multi-Head Attention]] — 将 Self-Attention 拆分为多个并行头，捕捉多种依赖关系
- [[wiki/concepts/cross-attention|Cross-Attention]] — Q 来自一个序列，K/V 来自另一个序列的注意力变体
- [[wiki/entities/Transformer|Transformer]] — 使用 Self-Attention 作为核心构件的架构
