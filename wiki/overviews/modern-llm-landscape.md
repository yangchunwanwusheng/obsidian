---
type: overview
tags: [深度学习, 大语言模型, BERT, GPT, T5, 现代LLM, 格局概述]
created: 2026-04-09
updated: 2026-04-09
sources:
  - raw/lessons/Transformer/08-beyond-original.md
---

# 现代 LLM 格局概述

> 从 2017 年的原始 Transformer 到 2026 年的多模态大模型，一幅完整的技术演进全景图。

## 概述

2017 年 Transformer 论文发表后，研究社区在这个通用框架上进行了大量创新，分化出三条主要技术路线：Encoder-Only（以 [[wiki/entities/BERT|BERT]] 为代表）、Decoder-Only（以 [[wiki/entities/GPT|GPT]] 为代表）和 Encoder-Decoder（以 [[wiki/entities/T5|T5]] 为代表）。同时，[[wiki/concepts/scaling-law|规模法则]] 的发现推动了模型参数从亿级向万亿级迈进，[[wiki/concepts/mixture-of-experts|MoE]]、Flash Attention 等效率优化技术让大规模训练和部署成为可能。本页串联这一技术演化的完整脉络。

## 三条技术路线

### Encoder-Only：理解优先

[[wiki/entities/BERT|BERT]]（2018）开创了双向预训练的范式，通过 Masked Language Model（完形填空式训练）让模型同时理解左右上下文。它专注于文本理解任务——分类、命名实体识别、问答、语义相似度——而非文本生成。

代表模型：BERT、RoBERTa、DeBERTa。参数规模通常在 1 亿~3 亿级别。优势是训练效率高、推理速度快，在理解任务上至今仍有竞争力。

### Decoder-Only：生成的王者

[[wiki/entities/GPT|GPT]] 系列选择了单向自回归的路线——从左到右预测下一个 token。这种看似"受限"的设计，配合 [[wiki/concepts/scaling-law|规模法则]]，在千亿参数级别涌现出 In-Context Learning、复杂推理、代码生成等惊人能力，成为当前大语言模型的主流架构。

发展脉络：GPT-1（1.17 亿参数，预训练+微调）→ GPT-2（15 亿，零样本迁移）→ GPT-3（1750 亿，In-Context Learning）→ GPT-4（~1.8 万亿，多模态+复杂推理）→ GPT-4o（原生多模态）。

### Encoder-Decoder：双能力兼具

[[wiki/entities/T5|T5]]（2019）保留了原始 Transformer 的完整架构，将所有 NLP 任务统一为"Text-to-Text"格式。翻译、摘要、问答都转化为输入文本→输出文本的形式。

代表模型：T5、BART、UL2。适合需要同时理解源文本并生成新文本的任务（如翻译、摘要）。参数规模通常在 1 亿~110 亿级别。

### 三种架构的选型原则

```
你的任务是什么？
        │
        ▼
  需要生成新文本？
    │
  Yes───No
    │       │
    ▼       ▼
  生成类   理解/分类类
    │       │
    ▼       ▼
  需要参考原文？  →  Encoder-Only (BERT系)
    │
  Yes───No
    │       │
    ▼       ▼
Encoder-Decoder  Decoder-Only
  (T5/BART)      (GPT/LLaMA)
```

## 效率优化技术

随着模型规模扩大，效率优化成为 Transformer 实用化的关键。

### Flash Attention：硬件感知加速

Flash Attention 不改变注意力的数学定义，而是优化其在 GPU 上的计算方式。核心创新是使用 tiling 技术，让中间结果始终保留在 SRAM（快速缓存）中，避免反复读写 HBM（高带宽显存）。效果是速度提升 2-4 倍，内存使用减少。Flash Attention 2 进一步优化了线程分配和并行策略。

### 混合专家（MoE）：以小博大的策略

[[wiki/concepts/mixture-of-experts|MoE]] 架构通过稀疏激活实现"大参数量、小计算量"的目标。以 Mixtral 8x7B 为例：总参数 46.7B（8 个 expert），但每个 token 只激活 top-2 expert（约 12B 参数），实际计算量接近 12B 的 Dense 模型。GPT-4 被推测可能使用了 MoE 架构。

### 其他优化方向

- **量化（Quantization）**：将模型权重从 FP16 降至 INT8 或 INT4，大幅减少显存和推理延迟，精度损失可控。
- **稀疏注意力（Longformer/BigBird）**：将全局注意力改为局部窗口 + 少量全局位置，复杂度从 $O(n^2)$ 降至 $O(n)$。
- **Mamba / 状态空间模型**：用递归隐藏状态压缩序列信息，在长序列任务上实现 $O(n)$ 复杂度，是对 Transformer 架构本身的挑战。

## 发展脉络（2017-2026）

| 年份 | 里程碑事件 | 核心突破 |
|------|-----------|----------|
| 2017 | Transformer 论文发表 | 抛弃 RNN，纯注意力架构 |
| 2018 | BERT 发布 | 双向预训练，刷新 11 项 NLP 基准 |
| 2018 | GPT-1 发布 | 预训练+微调范式 |
| 2019 | GPT-2 / T5 | 零样本迁移、Text-to-Text 统一 |
| 2020 | GPT-3 发布 | 1750 亿参数，涌现 In-Context Learning |
| 2022 | ChatGPT 爆火 | RLHF 对齐，Decoder-Only 成为共识 |
| 2023 | LLaMA 开源 | 开源模型追赶闭源，MoE 开始应用 |
| 2023 | GPT-4 发布 | 多模态、复杂推理、MoE 架构（推测） |
| 2024 | GPT-4o / Claude 3.5 | 原生多模态、长上下文 |
| 2025 | 小型化+专用化 | Phi、Mistral、Qwen 等小模型崛起 |
| 2026 | 多模态融合成为默认 | 视觉、语音、文本统一模型成为主流 |

## 架构选型指南

| 场景 | 推荐架构 | 代表模型 | 原因 |
|------|----------|----------|------|
| 情感分类 | Encoder-Only | RoBERTa, DeBERTa | 快速、准确、可微调 |
| 命名实体识别 | Encoder-Only | BERT-Chinese | 细粒度序列标注 |
| 机器翻译 | Encoder-Decoder | T5, NLLB | 源语言理解+目标语言生成 |
| 文章摘要 | Encoder-Decoder | BART, Pegasus | 理解长文+生成摘要 |
| 对话/续写 | Decoder-Only | GPT-4, LLaMA | 自回归生成 |
| 代码生成 | Decoder-Only | Codex, CodeGen | 强生成能力+逻辑推理 |
| 长文档问答 | Encoder+稀疏注意力 | Longformer-BERT | O(n) 复杂度处理长文 |

## 核心实体与概念索引

### 实体页

| 实体 | Wiki 页面 | 定位 |
|------|-----------|------|
| Transformer | [[wiki/entities/Transformer]] | 一切的原点 |
| BERT | [[wiki/entities/BERT]] | Encoder-Only 路线的开创者 |
| GPT | [[wiki/entities/GPT]] | Decoder-Only 路线的引领者 |
| T5 | [[wiki/entities/T5]] | Encoder-Decoder 路线的代表 |

### 概念页

| 概念 | Wiki 页面 | 与本页的关系 |
|------|-----------|-------------|
| 规模法则 | [[wiki/concepts/scaling-law]] | 理解 GPT 系列从量变到质变的关键 |
| 混合专家 | [[wiki/concepts/mixture-of-experts]] | 效率优化的核心策略 |
| 自回归生成 | [[wiki/concepts/autoregressive-generation]] | Decoder-Only 架构的生成机制 |
| KV Cache | [[wiki/concepts/kv-cache]] | 大模型推理的必备优化 |

### 对比分析

| 对比 | Wiki 页面 | 焦点 |
|------|-----------|------|
| 三种架构变体 | [[wiki/comparisons/encoder-only-vs-decoder-only-vs-encoder-decoder]] | 设计哲学与适用场景 |
| Dense vs MoE | [[wiki/comparisons/dense-vs-moe]] | 激活方式与计算效率 |

## 相关页面

- [[wiki/overviews/transformer-architecture-overview|Transformer 架构概述]] — 一切技术的架构基础
- [[wiki/overviews/transformer-training-inference-overview|Transformer 训练与推理概述]] — 从架构到可用模型的桥梁
