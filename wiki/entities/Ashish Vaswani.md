---
type: entity
tags: [AI研究者, Transformer, Google Brain]
created: 2026-04-09
updated: 2026-04-09
sources:
  - raw/lessons/Transformer/01-why-transformer.md
---

# Ashish Vaswani

> Transformer 架构论文的第一作者，提出了"只用 Attention，不用递归"的革命性思想。

## 基本信息

- **身份**：AI 研究员
- **核心机构**：Google Brain（论文发表时期）
- **研究领域**：序列建模、注意力机制、机器翻译

## 主要贡献

### Attention Is All You Need（2017）

Ashish Vaswani 作为第一作者，与 Noam Shazeer、Niki Parmar 等人共同发表了论文 *"Attention Is All You Need"*，提出了 **Transformer** 架构。这篇论文的核心洞察极为大胆：既然 Attention 已经是最有效的信息交互机制，为什么还要保留 RNN 这个"拖油瓶"？

论文标题一语道破核心思想——**只用 Attention，不用递归**。Transformer 带来了三大革命性优势：

1. **真正的并行化**：所有 token 同时处理，不再受顺序依赖束缚，可充分利用 GPU 算力
2. **全局感受野**：每个位置直接看到所有其他位置，信息交流成本与距离无关
3. **常数跳的梯度路径**：从根本上消除了梯度消失的土壤

Transformer 的核心操作——**Scaled Dot-Product Self-Attention**，通过矩阵乘法一次性计算所有 token 对之间的注意力分数，彻底改变了序列建模的范式。

## 影响

Transformer 的提出是深度学习历史上最重要的里程碑之一。它不仅取代了 RNN/LSTM 在 NLP 领域的统治地位，还成为 GPT、BERT、T5 等现代大语言模型的共同基础架构，并成功扩展到视觉（ViT）、语音、蛋白质结构预测等众多领域。

Vaswani 等人的工作揭示了一个关键 trade-off：**用 O(n²) 的计算复杂度，换取并行性、全局感受野和健康梯度流通**。在 GPU 时代，这个交换极为划算。

## 相关页面

- [[wiki/entities/GPT]] — 基于 Transformer Decoder 的自回归语言模型
- [[wiki/entities/BERT]] — 基于 Transformer Encoder 的双向预训练模型
- [[wiki/entities/T5]] — 基于 Transformer Encoder-Decoder 的统一生成模型
- [[wiki/entities/Dzmitry Bahdanau]] — Attention 机制的先驱，为 Transformer 铺路
- [[wiki/entities/Jay Alammar]] — Transformer 可视化教学先驱
- [[wiki/concepts/self-attention]] — Transformer 的核心机制
- [[wiki/concepts/encoder-decoder]] — Transformer 的架构范式
