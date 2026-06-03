---
type: concept
tags: [深度学习, Transformer, 编码器, 解码器, Encoder-Decoder]
created: 2026-04-09
updated: 2026-04-09
sources:
  - raw/lessons/Transformer/06-encoder-decoder.md
---

# 编码器-解码器架构（Encoder-Decoder）

> Transformer 的核心范式——编码器负责理解输入，解码器负责生成输出，两者通过 Cross-Attention 通信。

## 概述：理解与生成的分工

编码器-解码器（Encoder-Decoder）是 Transformer 原始论文中提出的完整架构，用于解决序列到序列（seq2seq）任务，如机器翻译、文本摘要、代码生成等。其核心思想是**分工协作**：

- **编码器（Encoder）**：阅读并理解完整的输入序列，输出一组上下文化的向量表示。每个输入 token 都会被编码为一个融合了全局上下文信息的向量。
- **解码器（Decoder）**：基于编码器的理解，逐 token 生成输出序列。生成过程中受到两个约束——不能偷看未来位置（Masked Self-Attention），且必须参考编码器的输出（Cross-Attention）。

原始 Transformer 中，编码器和解码器各堆叠 6 层（$N=6$），隐藏维度 $d_{model}=512$，多头注意力头数 $h=8$。

## 编码器栈 vs 解码器栈

| 维度 | 编码器（Encoder） | 解码器（Decoder） |
|------|------------------|------------------|
| 核心职责 | 理解完整输入序列 | 自回归地生成输出序列 |
| 每层子层数 | 2 个 | 3 个 |
| 注意力类型 | [[wiki/concepts/self-attention\|Self-Attention]] | [[wiki/concepts/masked-attention\|Masked Self-Attention]] + [[wiki/concepts/cross-attention\|Cross-Attention]] |
| 是否需要 Mask | 不需要（输入完整可见） | 需要（不能偷看未来） |
| 信息来源 | 仅自身输入 | 自身 + 编码器输出 |
| 典型变体 | BERT（Encoder-Only） | GPT（Decoder-Only） |

## 数据流概览

一条完整的数据流路径：

```
输入序列 → Token Embedding + 位置编码
        → 编码器 ×6 层（Self-Attention + FFN）
        → 编码器输出（上下文化表示）

目标序列 → Token Embedding + 位置编码
        → 解码器 ×6 层：
            ① Masked Self-Attention（自注意力，只看过去）
            ② Cross-Attention（Q=解码器, K/V=编码器输出）
            ③ FFN（位置独立变换）
        → Linear + Softmax
        → 词概率分布 → 采样/取 argmax → 下一个 token
```

编码器的输出被解码器的每一层通过 Cross-Attention 引用，是两个模块间信息传递的唯一通道。

## 三种注意力在架构中的位置

| 注意力类型 | 位置 | 作用 |
|-----------|------|------|
| [[wiki/concepts/self-attention\|Self-Attention]] | 编码器每个子层 | 输入序列内部的全局信息融合 |
| [[wiki/concepts/masked-attention\|Masked Self-Attention]] | 解码器第一个子层 | 已生成序列内部的因果信息融合 |
| [[wiki/concepts/cross-attention\|Cross-Attention]] | 解码器第二个子层 | 跨序列的信息对齐与传递 |

三者共享相同的缩放点积计算公式，区别仅在于 Q/K/V 的来源和是否施加因果遮罩。[[wiki/concepts/multi-head-attention|Multi-Head Attention]] 机制对三者均适用，允许模型从多个表示子空间并行地捕获不同类型的依赖关系。

## 要点

- **分工明确**：编码器做"理解"，解码器做"生成"，各司其职，比单一模块处理两种任务更高效。
- **层级的抽象递进**：6 层编码器/解码器逐层提取更抽象的特征——底层学词法，中层学句法，高层学语义。
- **架构变体丰富**：Encoder-Only（BERT）擅长理解任务，Decoder-Only（GPT）擅长生成任务，完整 Encoder-Decoder（T5、BART）擅长 seq2seq 任务。
- **Embedding 权重共享**：原始设计中输入 Embedding、输出 Embedding 和 Softmax 前的 Linear 层共享同一权重矩阵，减少参数量并保持语义一致性。

## 相关页面

- [[wiki/concepts/cross-attention|Cross-Attention（交叉注意力）]]
- [[wiki/concepts/masked-attention|Masked Self-Attention（掩码自注意力）]]
- [[wiki/concepts/self-attention|Self-Attention（自注意力）]]
- [[wiki/concepts/multi-head-attention|Multi-Head Attention（多头注意力）]]
- [[wiki/concepts/positional-encoding|位置编码]]
- [[wiki/concepts/feed-forward-network|前馈网络]]
- [[wiki/concepts/residual-connection|残差连接]]
- [[wiki/concepts/layer-normalization|层归一化]]
