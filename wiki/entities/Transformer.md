---
type: entity
tags: [深度学习, Transformer, 注意力机制, 序列建模]
created: 2026-04-09
updated: 2026-04-09
sources:
  - raw/lessons/Transformer/01-why-transformer.md
  - raw/lessons/Transformer/06-encoder-decoder.md
  - raw/lessons/Transformer/08-beyond-original.md
---

# Transformer

> 完全基于注意力机制的序列建模架构，彻底取代了 RNN 的顺序依赖，开启了深度学习的新纪元。

## 基本信息

| 项目 | 内容 |
|------|------|
| **名称** | Transformer |
| **论文** | *"Attention Is All You Need"* |
| **作者** | Ashish Vaswani, Noam Shazeer, Niki Parmar, Jakob Uszkoreit, Llion Jones 等 |
| **年份** | 2017 |
| **机构** | Google Brain, Google Research |
| **核心贡献** | 首次提出完全抛弃递归结构、仅依赖注意力机制的序列转换架构 |

## 概述

Transformer 是 2017 年由 Google 团队提出的深度学习架构。其核心洞察是：既然 Attention 机制已经实现了序列元素之间的全局信息交互，RNN 的"逐步传递信息"功能就变得冗余。论文标题 *"Attention Is All You Need"* 一语道破核心思想——只用 Attention，不用递归。

原始 Transformer 采用编码器-解码器（Encoder-Decoder）架构，各由 N=6 层堆叠而成。编码器负责理解输入序列，通过 Multi-Head Self-Attention 让每个位置融合全局上下文信息；解码器负责自回归生成输出，通过 Masked Self-Attention 防止偷看未来，并通过 Cross-Attention 从编码器输出中按需取用信息。

Transformer 的诞生标志着深度学习从"序列处理"范式转向"集合处理"范式，直接催生了 BERT、GPT 等大语言模型，成为现代 AI 的基础设施。

## 核心组件

| 组件 | 说明 | 详细概念 |
|------|------|----------|
| **Self-Attention** | 序列中每个位置直接与所有其他位置交互，计算复杂度为 O(1) 跳 | [[wiki/concepts/self-attention]] |
| **Multi-Head Attention** | 将注意力拆分为多个头并行计算，让模型同时关注不同子空间的信息 | [[wiki/concepts/multi-head-attention]] |
| **Positional Encoding** | 通过正弦/余弦函数为无序的注意力机制注入位置信息 | [[wiki/concepts/positional-encoding]] |
| **Feed-Forward Network** | 对每个位置独立施加两层线性变换+ReLU，提供非线性建模能力 | [[wiki/concepts/feed-forward-network]] |
| **残差连接** | 将子层输入直接加到输出上，缓解深层网络的退化问题 | [[wiki/concepts/residual-connection]] |
| **Layer Normalization** | 对每个样本的特征维度做归一化，稳定深层网络的训练 | [[wiki/concepts/layer-normalization]] |

## 三大革命

| 革命 | RNN 体系 | Transformer |
|------|----------|-------------|
| **并行计算** | 必须逐 token 顺序处理，GPU 利用率低 | 所有位置同时计算，矩阵运算天然并行 |
| **全局感受野** | 信息需经 T 步传递，长距离衰减严重 | 每个位置一步直达所有其他位置，距离无关 |
| **梯度路径** | 路径长度 = 序列长度，易梯度消失 | 路径长度 = 常数，与序列长度无关 |

> 核心代价：注意力矩阵为 n x n，计算复杂度 O(n^2)，但 GPU 并行的优势在常规序列长度下远大于此代价。

## 架构变体

原始 Transformer 发表后，研究者根据任务特性发展出三条路线：

| 变体 | 代表模型 | 注意力方向 | 适用场景 |
|------|----------|------------|----------|
| **Encoder-Only** | BERT, RoBERTa, DeBERTa | 双向（可见全部上下文） | 理解任务：分类、NER、问答 |
| **Decoder-Only** | GPT-2/3/4, LLaMA, Claude | 单向（仅左侧上下文） | 自回归生成：续写、对话、代码 |
| **Encoder-Decoder** | T5, BART, UL2 | 编码器双向 + 解码器单向 | Seq2Seq：翻译、摘要 |

- [[wiki/entities/BERT]] — Encoder-Only 路线的开创者，通过 MLM 实现双向预训练
- [[wiki/entities/GPT]] — Decoder-Only 路线的代表，验证了规模法则的惊人力量
- [[wiki/entities/T5]] — Encoder-Decoder 路线的标杆，将所有 NLP 任务统一为文本到文本

## 关键超参数

原始论文（base 模型）的配置：

| 超参数 | 符号 | 值 | 说明 |
|--------|------|-----|------|
| 模型维度 | d_model | 512 | 嵌入和隐藏层的维度 |
| 注意力头数 | h | 8 | Multi-Head Attention 的头数 |
| 每头维度 | d_k = d_v | 64 | 512 / 8 |
| FFN 中间层维度 | d_ff | 2048 | 4 x 512 |
| 编码器/解码器层数 | N | 6 | 两侧各堆叠 6 层 |

## 相关页面

### 概念页

- [[wiki/concepts/self-attention]]
- [[wiki/concepts/multi-head-attention]]
- [[wiki/concepts/positional-encoding]]
- [[wiki/concepts/feed-forward-network]]
- [[wiki/concepts/residual-connection]]
- [[wiki/concepts/layer-normalization]]

### 实体页

- [[wiki/entities/BERT]]
- [[wiki/entities/GPT]]
- [[wiki/entities/T5]]

### 学习笔记

- [[wiki/lessons/Transformer/01-why-transformer|为什么需要 Transformer]]
- [[wiki/lessons/Transformer/06-encoder-decoder|编码器-解码器架构]]
- [[wiki/lessons/Transformer/08-beyond-original|超越原始 Transformer]]
