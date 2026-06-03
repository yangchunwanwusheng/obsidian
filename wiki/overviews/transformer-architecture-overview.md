---
type: overview
tags: [深度学习, Transformer, 架构概述, 全局视角]
created: 2026-04-09
updated: 2026-04-09
sources:
  - raw/lessons/Transformer/01-why-transformer.md
  - raw/lessons/Transformer/02-self-attention.md
  - raw/lessons/Transformer/06-encoder-decoder.md
---

# Transformer 架构全景概述

> 从输入到输出，一篇文章串联 Transformer 的所有核心组件与设计哲学。

## 概述

Transformer 是 2017 年由 Vaswani 等人在论文 *"Attention Is All You Need"* 中提出的深度学习架构。它彻底抛弃了 RNN 的递归结构，仅依赖注意力机制实现序列建模，带来了三大革命性优势：真正的并行计算、全局感受野（一步直达任意位置）、以及常数跳的梯度路径。本页从数据流角度，完整呈现 Transformer 从输入到输出的每一步，并揭示各组件之间的协作关系。

## 完整数据流

一段文字从进入 Transformer 到输出预测结果，经历以下关键阶段：

**第一步：分词与嵌入。** 输入文本首先被分词器拆分为 token 序列，每个 token 映射为一个 $d_{model}$ 维的嵌入向量。原始 Transformer 使用 $d_{model} = 512$。

**第二步：位置编码注入。** 由于 [[wiki/concepts/self-attention|Self-Attention]] 本身是置换等变的（不感知顺序），需要通过 [[wiki/concepts/positional-encoding|位置编码]] 为每个 token 注入位置信息。原始论文使用正弦-余弦函数生成位置编码，与嵌入向量逐元素相加。

**第三步：编码器堆叠处理。** 嵌入序列进入 $N=6$ 层编码器。每一层包含两个子层：[[wiki/concepts/self-attention|Multi-Head Self-Attention]] 让每个位置融合全局上下文信息，[[wiki/concepts/feed-forward-network|Position-wise FFN]] 对每个位置独立做非线性特征提取。两个子层都配有 [[wiki/concepts/residual-connection|残差连接]] 和 [[wiki/concepts/layer-normalization|Layer Normalization]]，保证梯度健康流通、训练稳定。编码器输出一组上下文化的向量表示，每个 token 的向量已融合完整输入序列的信息。

**第四步：解码器自回归生成。** 解码器同样是 $N=6$ 层，但每层包含三个子层。第一个子层是 [[wiki/concepts/masked-attention|Masked Self-Attention]]，只能看到已生成的 token，防止偷看未来信息。第二个子层是 [[wiki/concepts/cross-attention|Cross-Attention]]，Q 来自解码器，K/V 来自编码器输出——这是连接"理解"与"生成"的桥梁。第三个子层是与编码器相同的 FFN。解码器最终通过 Linear + Softmax 输出下一个 token 的概率分布。

**第五步：[[wiki/concepts/autoregressive-generation|自回归生成]]。** 推理时逐 token 生成：每步选出概率最高的 token（或按策略采样），将其追加到输入序列，继续下一步，直到生成结束符。

## 组件关系图

```
                    Transformer 完整架构
 ═══════════════════════════════════════════════════
 │                                                 │
 │  输入文本                                        │
 │    │                                             │
 │    ▼                                             │
 │  ┌──────────┐    ┌──────────────────┐            │
 │  │ Token    │    │ Positional       │            │
 │  │ Embedding│ +  │ Encoding         │            │
 │  └────┬─────┘    └──────────────────┘            │
 │       │                                          │
 │  ╔════╧════════════════════════════════╗         │
 │  ║        编码器 × 6 层               ║         │
 │  ║  ┌─────────────┐  ┌─────────────┐  ║         │
 │  ║  │ Multi-Head  │  │ FFN         │  ║         │
 │  ║  │ Self-Attn   │  │ (逐位置)    │  ║         │
 │  ║  │ + Add&Norm  │  │ + Add&Norm  │  ║         │
 │  ║  └─────────────┘  └─────────────┘  ║         │
 │  ╚══════════╤═════════════════════════╝         │
 │             │ 上下文化表示                       │
 │             │         ┌──────────────────┐       │
 │             └────────►│ Cross-Attention  │───────│──► K, V
 │                       │ (Q来自解码器)     │       │
 │  ╔════════════════════╧══════════════════╗       │
 │  ║        解码器 × 6 层               ║       │
 │  ║  ┌──────────┐ ┌──────────┐ ┌──────┐ ║       │
 │  ║  │ Masked   │ │ Cross    │ │ FFN  │ ║       │
 │  ║  │ Self-Attn│ │ Attn    │ │      │ ║       │
 │  ║  │ +Add&Norm│ │ +Add&Norm│ │+A&N  │ ║       │
 │  ║  └──────────┘ └──────────┘ └──────┘ ║       │
 │  ╚══════════════════╤═══════════════════╝       │
 │                     │                            │
 │                     ▼                            │
 │              Linear + Softmax                    │
 │                     │                            │
 │                     ▼                            │
 │              下一个 token 概率分布               │
 ═══════════════════════════════════════════════════
```

## 设计哲学

Transformer 的每一个设计选择都不是随意的，而是对 RNN 根本缺陷的精确回应：

**抛弃递归，拥抱矩阵乘法。** RNN 的顺序依赖使其无法并行。Transformer 用 Self-Attention 替代递归——通过 $QK^T$ 一次性计算所有 token 对之间的注意力分数，天然适配 GPU 并行计算。付出的代价是 $O(n^2)$ 的复杂度，但在 GPU 时代，并行矩阵运算的速度优势远大于复杂度劣势。

**残差连接 + LayerNorm：深层网络的"骨骼系统"。** 6 层堆叠意味着信息要经过大量非线性变换。[[wiki/concepts/residual-connection|残差连接]] 提供了梯度直达的"高速公路"，[[wiki/concepts/layer-normalization|LayerNorm]] 则保证每层的输出数值分布稳定。两者配合，让深层 Transformer 的训练变得可行。

**FFN：注意力之外的"思考层"。** 注意力负责信息融合（token 之间交流），[[wiki/concepts/feed-forward-network|FFN]] 负责信息加工（每个 token 独立变换）。有趣的是，FFN 占 Transformer 约 2/3 的参数量——注意力让信息流动，FFN 让信息变深。

**Mask 与 Cross-Attention的分工。** [[wiki/concepts/masked-attention|Masked Attention]] 保证生成的因果性（不偷看未来），[[wiki/concepts/cross-attention|Cross-Attention]] 则让生成有据可依（参考编码器的理解）。两者配合，实现"有约束的创造"。

## 核心组件索引

| 组件 | 概念页 | 核心作用 |
|------|--------|----------|
| Self-Attention | [[wiki/concepts/self-attention]] | 序列内部各位置互相关注，实现全局信息融合 |
| Multi-Head Attention | [[wiki/concepts/self-attention]] | 多头并行关注，捕捉不同类型的关系 |
| 位置编码 | [[wiki/concepts/positional-encoding]] | 注入位置信号，弥补注意力的置换等变性 |
| FFN | [[wiki/concepts/feed-forward-network]] | 逐位置非线性变换，占模型 2/3 参数 |
| 残差连接 | [[wiki/concepts/residual-connection]] | 梯度直达通道，让深层网络可训练 |
| Layer Normalization | [[wiki/concepts/layer-normalization]] | 归一化数值分布，稳定训练过程 |
| Masked Attention | [[wiki/concepts/masked-attention]] | 屏蔽未来位置，保证自回归因果性 |
| Cross-Attention | [[wiki/concepts/cross-attention]] | 连接编码器与解码器的信息桥梁 |
| 编码器-解码器 | [[wiki/concepts/encoder-decoder]] | "理解输入 + 生成输出"的核心架构范式 |
| 自回归生成 | [[wiki/concepts/autoregressive-generation]] | 逐 token 生成的核心范式 |

## 相关页面

- [[wiki/overviews/transformer-training-inference-overview|Transformer 训练与推理概述]] — 架构如何被训练和部署
- [[wiki/overviews/modern-llm-landscape|现代 LLM 格局概述]] — 从原始 Transformer 到 BERT/GPT 的演化
- [[wiki/comparisons/encoder-only-vs-decoder-only-vs-encoder-decoder|三种架构变体对比]] — Encoder-Only / Decoder-Only / Encoder-Decoder 的差异
- [[wiki/entities/Transformer|Transformer 实体页]]
