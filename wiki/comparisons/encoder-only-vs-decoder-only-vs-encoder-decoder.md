---
type: comparison
tags: [深度学习, Transformer, BERT, GPT, T5, 架构对比]
created: 2026-04-09
updated: 2026-04-09
sources:
  - raw/lessons/Transformer/08-beyond-original.md
---

# Encoder-Only vs Decoder-Only vs Encoder-Decoder 架构对比

> 三种 Transformer 变体在设计哲学上截然不同：Encoder-Only 牺牲生成能力换取双向理解，Decoder-Only 拥抱单向约束实现自回归生成，Encoder-Decoder 则保留完整能力但增加架构复杂度。

## 概述

2017 年原始 Transformer 同时包含编码器和解码器，用于机器翻译。研究者很快发现并非所有任务都需要完整架构，由此演化出三种主流变体。三种架构的根本分歧在于**注意力方向**——即每个 token 可以"看到"哪些其他 token。这一看似简单的设计选择，深刻影响了模型的预训练方式、擅长任务和工程特性。

核心设计哲学差异：

- **Encoder-Only**：理解优先。让每个 token 同时看到左右两侧的上下文，追求对输入的深度理解。
- **Decoder-Only**：生成优先。强制从左到右的单向注意力，天然适配自回归生成过程。
- **Encoder-Decoder**：理解与生成兼顾。编码器负责深度理解源输入，解码器负责自回归生成目标输出，两者通过交叉注意力连接。

## 核心对比表

| 对比维度 | Encoder-Only (BERT) | Decoder-Only (GPT) | Encoder-Decoder (T5) |
|----------|---------------------|---------------------|----------------------|
| **注意力方向** | 双向（可见全部上下文） | 单向因果（仅可见左侧） | 编码器双向 + 解码器单向 |
| **预训练任务** | MLM（遮蔽语言模型）+ NSP | CLM（下一个 token 预测） | 跨度破坏（Span Corruption） |
| **生成能力** | 不支持自回归生成 | 天然支持自回归生成 | 编码后自回归解码 |
| **理解能力** | 最强（双向上下文） | 较弱（单向限制） | 强（编码器双向） |
| **典型模型** | BERT, RoBERTa, DeBERTa | GPT-2/3/4, LLaMA, Claude | T5, BART, UL2 |
| **典型任务** | 分类、NER、问答、语义相似度 | 对话、续写、代码生成 | 翻译、摘要、改写 |
| **参数规模** | 1 亿 ~ 3 亿 | 70 亿 ~ 1.8 万亿 | 1 亿 ~ 110 亿 |
| **推理方式** | 一次前向传播输出 | 逐 token 自回归 | 编码一次 + 逐 token 解码 |
| **训练稳定性** | 高 | 中等（长序列梯度） | 中等（双模块协调） |

## 各架构的优势与局限

### Encoder-Only（以 BERT 为代表）

**优势：**
- 双向注意力让每个位置都拥有完整的上下文信息，在理解类任务上精度最高
- 推理速度快——一次前向传播即可输出全部结果，无需逐 token 生成
- 参数效率高，亿级参数即可达到优秀的理解任务表现
- 微调成本低，少量标注数据即可适配下游任务

**局限：**
- 无法进行自回归生成，不适用于对话、续写等开放生成场景
- MLM 预训练任务与下游任务存在差异，需要微调才能适配
- 架构天花板较低，难以通过规模扩展获得涌现能力

### Decoder-Only（以 GPT 为代表）

**优势：**
- 架构简洁统一，仅用单向注意力即可完成所有操作
- 随规模增长涌现出 In-Context Learning、复杂推理等高级能力
- 统一的"下一个 token 预测"预训练目标与生成任务天然匹配
- 一个模型可以同时处理多种任务，通过提示词即可切换

**局限：**
- 单向注意力在理解任务上不如双向注意力精确
- 推理延迟高，必须逐 token 生成，长输出时尤为明显
- 大规模部署的推理成本极高
- 理解类任务（如分类、NER）可能"杀鸡用牛刀"

### Encoder-Decoder（以 T5 为代表）

**优势：**
- 兼具深度理解（编码器）和高质量生成（解码器）能力
- 天然适配 seq2seq 任务（翻译、摘要），输入和输出可完全不同
- 编码器只需运行一次，生成时解码器通过交叉注意力复用源信息

**局限：**
- 架构最复杂，实现和维护成本高
- 需要同时训练两个模块，超参数调优难度大
- 生态和工具链不如 Decoder-Only 丰富
- 在纯理解或纯生成任务上，通常不如专用架构高效

## 架构选型指南

### 按任务类型选择

| 任务类型 | 推荐架构 | 代表模型 | 选型理由 |
|----------|----------|----------|----------|
| 文本分类（情感、主题） | Encoder-Only | RoBERTa, DeBERTa | 快速精准，资源消耗低 |
| 命名实体识别（NER） | Encoder-Only | BERT-Chinese | 细粒度序列标注，双向理解 |
| 问答系统（抽取式） | Encoder-Only | DeBERTa | 精确定位答案 span |
| 对话 / 聊天 | Decoder-Only | GPT-4, LLaMA | 开放式自回归生成 |
| 文本续写 / 创作 | Decoder-Only | GPT-4, Claude | 流畅自然的生成能力 |
| 代码生成 | Decoder-Only | Codex, CodeGen | 长序列自回归生成 |
| 机器翻译 | Encoder-Decoder | T5, NLLB | 需深度理解源语言后生成目标语言 |
| 文章摘要 | Encoder-Decoder | BART, Pegasus | 理解全文后压缩生成 |
| 文本改写 / 风格转换 | Encoder-Decoder | T5 | 保留原意的同时改变表达 |

### 决策原则

1. **只需理解，不需生成** → Encoder-Only
2. **需要开放式生成** → Decoder-Only
3. **需要理解原文后生成新文本（输入与输出对应）** → Encoder-Decoder
4. **资源受限时** → Encoder-Only（亿级参数即可胜任理解任务）
5. **追求通用能力** → Decoder-Only（通过规模获得涌现能力，一个模型处理多种任务）

### 2024-2026 年的趋势

Decoder-Only 架构凭借规模优势成为主流，LLaMA、Qwen 等开源模型均为 Decoder-Only。但 Encoder-Only 在需要高精度理解的垂直场景（医疗、法律 NER）中仍有不可替代的优势。Encoder-Decoder 在翻译和多语言任务上保持竞争力。

## 相关页面

- [[wiki/entities/BERT|BERT 实体页]]
- [[wiki/entities/GPT|GPT 实体页]]
- [[wiki/entities/T5|T5 实体页]]
- [[wiki/concepts/encoder-decoder|编码器-解码器架构]]
- [[wiki/lessons/Transformer/08-beyond-original|超越原始 Transformer 课程]]
- [[wiki/comparisons/dense-vs-moe|Dense vs MoE 对比]]
