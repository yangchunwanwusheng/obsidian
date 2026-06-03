---
type: entity
tags: [AI研究者, 注意力机制, Seq2Seq]
created: 2026-04-09
updated: 2026-04-09
sources:
  - raw/lessons/Transformer/01-why-transformer.md
---

# Dzmitry Bahdanau

> 注意力机制的先驱，2014 年提出 Bahdanau Attention，打破了 Seq2Seq 的信息瓶颈。

## 基本信息

- **身份**：AI 研究员
- **核心机构**：Jacobs University Bremen / Yoshua Bengio 实验室（论文发表时期）
- **研究领域**：机器翻译、序列建模、注意力机制

## 主要贡献

### Bahdanau Attention（2014）

Dzmitry Bahdanau 在论文 *"Neural Machine Translation by Jointly Learning to Align and Translate"* 中提出了一个简单而深刻的改进，彻底改变了序列建模的走向。

**核心思想**：不再用固定维度的上下文向量 c 压缩整个输入序列，而是在解码的每一步，动态"查看"输入序列的所有位置，计算一个加权的上下文向量。

这一想法打破了 Seq2Seq 模型的根本瓶颈——**信息压缩失真**。在传统 Seq2Seq 中，无论输入是 5 个词还是 500 个词，上下文向量 c 的维度都一样，长序列的信息必然严重丢失。Bahdanau 的方案让模型在每一步按需取用信息，无需将所有内容塞进固定向量。

Bahdanau Attention 证明了三个关键突破：

| 突破 | 含义 |
|------|------|
| **动态聚焦** | 每一步按需查看输入的不同位置，无需压缩 |
| **可解释性** | 注意力权重可视化后清晰展示了源语言与目标语言的对齐关系 |
| **长程直达** | 解码器可以直接访问编码器的任意位置，不受梯度传递长度限制 |

他采用的加性注意力（additive attention）计算方式为 $e_{ij} = v^T \tanh(W_q q_i + W_k k_j)$，表达力强但计算较慢。后来 Transformer 选择了更高效的缩放点积注意力。

## 影响

Bahdanau 的工作是 Attention 机制的诞生标志，为后来的 Transformer 铺平了道路。不过需要注意的是，Bahdanau Attention 仍然搭在 RNN 之上——编码器和解码器都是 RNN，顺序依赖问题依然存在。正是这个局限促使 Vaswani 等人在 2017 年进一步提出了彻底抛弃 RNN 的 Transformer 架构。

## 相关页面

- [[wiki/entities/Ashish Vaswani]] — 在 Bahdanau 基础上提出 Transformer
- [[wiki/entities/Jay Alammar]] — 可视化讲解 Attention 与 Transformer 的技术教育家
- [[wiki/concepts/self-attention]] — 从 Bahdanau Attention 演化而来的核心机制
- [[wiki/concepts/cross-attention]] — 与 Bahdanau Attention 一脉相承的跨序列注意力
- [[wiki/concepts/encoder-decoder]] — Bahdanau Attention 所服务的架构范式
