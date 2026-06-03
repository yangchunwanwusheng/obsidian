---
type: concept
tags: [深度学习, Transformer, 自回归生成, Autoregressive, Teacher Forcing]
created: 2026-04-09
updated: 2026-04-09
sources:
  - raw/lessons/Transformer/06-encoder-decoder.md
  - raw/lessons/Transformer/07-training-inference.md
---

# 自回归生成

> 语言模型逐 token 生成文本的核心范式：每一步的输出成为下一步的输入，直到生成结束符。

## 概述

自回归生成（Autoregressive Generation）是 Transformer 解码器在推理阶段的工作方式。模型不是一次性输出完整序列，而是**逐个 token 生成**：给定已生成的全部前文，预测下一个 token 的概率分布，从中选择一个 token 追加到序列末尾，然后重复这一过程，直到生成结束符 `<EOS>` 或达到最大长度。

具体过程如下：

```
Step 1: 输入 <BOS>                → 预测 "我"   (概率 0.85)
Step 2: 输入 <BOS> 我             → 预测 "爱"   (概率 0.72)
Step 3: 输入 <BOS> 我 爱          → 预测 "人工" (概率 0.68)
Step 4: 输入 <BOS> 我 爱 人工     → 预测 "智能" (概率 0.81)
Step 5: 输入 <BOS> 我 爱 人工 智能 → 预测 <EOS> (结束)
```

这种"条件链式生成"的本质是**分解联合概率**：$P(w_1, w_2, ..., w_T) = \prod_{t=1}^{T} P(w_t | w_{<t})$。每一步只负责一个条件概率，但整个序列的质量取决于每一步的准确性。

## 训练与推理的差异

自回归生成引入了一个核心矛盾：**训练和推理时前文的来源不同**。

- **训练时**：使用 [[wiki/concepts/autoregressive-generation|Teacher Forcing]]，将真实标签（ground truth）作为每一步的输入。模型可以一次性处理整个目标序列（通过 [[wiki/concepts/self-attention|Masked Self-Attention]] 防止偷看未来位置），效率很高。
- **推理时**：模型只能用自己的预测结果作为下一步的输入。如果某一步生成了错误的 token，后续所有步骤都会受到这个错误的影响。

这种差异被称为**曝光偏差（Exposure Bias）**——模型在训练时从未见过自己犯错的输出，推理时却必须从自己的错误中继续生成。

## Teacher Forcing 策略

| 策略 | 做法 | 优点 | 缺点 |
|------|------|------|------|
| **Always Teacher Forcing** | 始终用真实标签 | 训练效率高、梯度稳定 | 曝光偏差严重 |
| **Free Running** | 始终用模型自己的预测 | 训练推理一致 | 训练初期错误累积 |
| **Scheduled Sampling** | 以概率 $p$ 用真实标签，$p$ 随训练递减 | 缓解曝光偏差 | 混合目标，训练不稳定 |
| **课程学习** | 早期 Teacher Forcing，后期 Free Running | 兼顾效率和一致性 | 需要调切换时机 |

现代大型语言模型（GPT 系列）由于规模足够大、能力足够强，通常直接使用 Free Running 训练，模型能够自行从错误中恢复。

## 要点

- 自回归生成是条件概率的链式分解，每步预测一个 token
- 训练时通过 Masked Self-Attention 并行处理整个序列（Teacher Forcing）
- 推理时逐 token 串行生成，每步输出成为下步输入
- 曝光偏差是 Teacher Forcing 的固有缺陷，Scheduled Sampling 和 Free Running 是常见缓解方案
- 自回归生成是所有 GPT 类模型的统一范式，也是推理速度瓶颈的根源

## 相关页面

- [[wiki/concepts/self-attention|Self-Attention]] — 注意力机制，自回归生成中每步依赖的核心计算
- [[wiki/concepts/kv-cache|KV Cache]] — 针对自回归生成的推理加速优化
- [[wiki/concepts/beam-search|Beam Search 与解码策略]] — 自回归生成中选择下一个 token 的策略
