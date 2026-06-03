---
type: entity
tags: [深度学习, NLP, BERT, 预训练, Google]
created: 2026-04-09
updated: 2026-04-09
sources:
  - raw/lessons/Transformer/08-beyond-original.md
---

# BERT

> 首次在大规模上实现真正双向预训练的 Encoder-Only 模型，革新了 NLP 理解任务的范式。

## 基本信息

| 字段 | 内容 |
|------|------|
| 全称 | Bidirectional Encoder Representations from Transformers |
| 论文 | [BERT: Pre-training of Deep Bidirectional Transformers](https://arxiv.org/abs/1810.04805) |
| 团队 | Google |
| 年份 | 2018 |
| 架构类型 | Encoder-Only（双向注意力） |
| 典型参数量 | BERT-Base: 1.1 亿 / BERT-Large: 3.4 亿 |

## 概述

BERT 只保留 Transformer 的编码器部分，通过 **Masked Language Model (MLM)** 和 **Next Sentence Prediction (NSP)** 两个预训练任务，让模型同时学习左侧和右侧上下文信息，实现真正的双向理解。预训练完成后，通过微调即可适配多种下游任务。

## 核心创新

### Masked Language Model (MLM)

随机遮住输入中 15% 的 token，要求模型预测被遮住的内容。遮蔽策略为：80% 替换为 `[MASK]`，10% 替换为随机 token，10% 保持不变。这一设计强迫模型从双向上下文中推断缺失信息，而非仅依赖单侧。

### Next Sentence Prediction (NSP)

训练模型判断两个句子是否为连续文本，以学习句子间关系。后续研究（如 RoBERTa）发现 NSP 作用有限，直接去掉后通过更长序列训练反而效果更好。

## 典型应用场景

| 任务类型 | 输出方式 | 示例 |
|----------|----------|------|
| 序列标注 | 每个位置的隐藏状态 | 命名实体识别（NER）、词性标注 |
| 句子对分类 | `[CLS]` token 表示 | 自然语言推理、语义相似度 |
| 问答 | 预测 start/end 位置 | SQuAD 问答 |
| 单句分类 | `[CLS]` token 表示 | 情感分析、垃圾邮件检测 |

## 与 GPT 的关键区别

| 维度 | BERT | GPT |
|------|------|-----|
| 架构 | Encoder-Only | Decoder-Only |
| 注意力 | 双向（可见全部上下文） | 单向（仅可见左侧） |
| 预训练任务 | MLM + NSP（完形填空） | 下一个 token 预测 |
| 核心能力 | 理解、分类、定位 | 生成、续写、对话 |
| 生成方式 | 非自回归 | 自回归 |

BERT 是"阅读理解高手"——同时看到全文后回答问题；GPT 是"作家"——从左到右逐字续写。

## 相关页面

- [[wiki/entities/GPT]] — Decoder-Only 对应架构
- [[wiki/entities/T5]] — Encoder-Decoder 对应架构
- [[wiki/lessons/Transformer/08-beyond-original|超越原始 Transformer]] — 三种架构的系统对比
