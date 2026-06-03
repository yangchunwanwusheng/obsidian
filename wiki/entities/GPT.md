---
type: entity
tags: [深度学习, NLP, GPT, OpenAI, 大语言模型]
created: 2026-04-09
updated: 2026-04-09
sources:
  - raw/lessons/Transformer/08-beyond-original.md
---

# GPT

> 通过 Decoder-Only 架构和规模法则（Scaling Law）验证了"模型越大，能力越强"的路径，涌现出 In-Context Learning 等突破性能力。

## 基本信息

| 字段 | 内容 |
|------|------|
| 全称 | Generative Pre-Trained Transformer |
| 系列论文 | GPT-1 (2018) / GPT-2 (2019) / GPT-3 (2020) / GPT-4 (2023) |
| 团队 | OpenAI |
| 起始年份 | 2018 |
| 架构类型 | Decoder-Only（单向 / Causal Masked Self-Attention） |

## 概述

GPT 只保留 Transformer 的解码器部分，采用因果掩码（Causal Mask）使每个 token 仅能关注自身及左侧上下文。预训练目标为预测下一个 token（自回归语言模型）。GPT 系列的核心叙事是**规模法则**——参数量从亿级增长到万亿级，模型能力随之涌现质变。

## GPT 系列演进

| 版本 | 参数量 | 核心突破 | 涌现能力 |
|------|--------|----------|----------|
| GPT-1 | 1.17 亿 | 首个"预训练+微调"范式 | 基础文本补全 |
| GPT-2 | 15 亿 | 更大数据集，多任务零样本 | 零样本任务迁移 |
| GPT-3 | 1750 亿 | In-Context Learning | 无需微调，通过提示直接执行任务 |
| GPT-4 | ~1.8 万亿（推测） | 多模态、复杂推理 | 复杂逻辑推理、指令遵循 |
| GPT-4o | — | 原生多模态 | 实时语音/视频交互 |

## 核心创新

### 规模法则（Scaling Law）

参数量增长带来能力质变：1 亿参数可做简单分类和补全，15 亿涌现零样本学习，1750 亿涌现 In-Context Learning 和复杂推理。从量变到质变，不是线性提升而是"涌现"。

### In-Context Learning (ICL)

GPT-3 最重要的发现——模型在**不更新参数**的情况下，通过 few-shot 示例学习新任务。本质是利用注意力机制在上下文中匹配任务描述与示例，从预训练知识中提取相关信息，而非真正更新权重。

## 关键架构选择

- **Causal Masked Self-Attention**：保证自回归生成的一致性
- **无编码器**：Decoder-Only 架构更简单，训练更稳定
- **Weight Tying**：嵌入层与输出层共享权重，节省参数

## 与 BERT 的关键区别

| 维度 | GPT | BERT |
|------|-----|------|
| 架构 | Decoder-Only | Encoder-Only |
| 注意力 | 单向（仅可见左侧） | 双向（可见全部上下文） |
| 预训练任务 | 下一个 token 预测 | MLM + NSP（完形填空） |
| 核心能力 | 生成、续写、对话 | 理解、分类、定位 |
| 参数规模 | 70 亿 ~ 1.8 万亿 | 1 亿 ~ 3 亿 |

## 相关页面

- [[wiki/entities/BERT]] — Encoder-Only 对应架构
- [[wiki/entities/T5]] — Encoder-Decoder 对应架构
- [[wiki/lessons/Transformer/08-beyond-original|超越原始 Transformer]] — 三种架构的系统对比
