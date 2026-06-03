---
type: concept
tags: [深度学习, Transformer, Masked Attention, 因果遮罩, 自回归]
created: 2026-04-09
updated: 2026-04-09
sources:
  - raw/lessons/Transformer/06-encoder-decoder.md
---

# Masked Self-Attention（掩码自注意力）

> 通过遮蔽未来位置的信息，确保解码器在生成第 $t$ 个 token 时只能看到位置 $1$ 到 $t$ 的内容，维持自回归的因果性。

## 概述：为什么解码器需要遮罩？

在 [[wiki/concepts/encoder-decoder|编码器-解码器]] 架构中，编码器可以同时看到完整的输入序列（因为输入已经完整给出），但解码器的工作方式完全不同——它是**逐 token 生成**的。

问题在于：训练时为了效率，解码器会一次性接收完整的目标序列（Teacher Forcing）。如果不加限制，第 1 个位置在 [[wiki/concepts/self-attention|Self-Attention]] 计算时就能"看到"第 2、3、4... 个位置的内容——这相当于考试时提前看到答案。模型会走捷径，直接从未来位置复制信息，而无法学会真正根据已有上下文进行预测。

**Masked Self-Attention 的职责**：在注意力计算中屏蔽"未来"位置，强制每个位置只能关注自身及之前的位置。

## 因果遮罩的实现：上三角 $-\infty$ 技巧

实现方式非常简洁——在 Softmax 之前，将注意力分数矩阵的上三角部分替换为 $-\infty$：

```
原始分数矩阵:                  因果遮罩 (Mask):
  我   爱   人工  智能
[0.3  0.5  0.2  0.4]         [  0  -∞  -∞  -∞ ]
[0.1  0.6  0.3  0.5]    →    [  0   0  -∞  -∞ ]
[0.2  0.4  0.5  0.3]         [  0   0   0  -∞ ]
[0.3  0.3  0.4  0.6]         [  0   0   0   0 ]

Masked + Softmax 后:
  我    爱    人工  智能
[1.00  0.00  0.00  0.00]    ← "我" 只看自己
[0.35  0.65  0.00  0.00]    ← "爱" 看 "我" 和自己
[0.25  0.38  0.37  0.00]    ← "人工" 看前面三个
[0.20  0.28  0.25  0.27]    ← "智能" 看全部
```

**为什么用 $-\infty$ 而不是 0？** 因为 $e^{-\infty} = 0$，经过 Softmax 后权重为零，完全屏蔽。若用 0，则 $e^0 = 1$，未来位置仍有非零贡献，遮罩失效。

## 训练时 vs 推理时的差异

| 阶段 | 输入 | Mask 的作用 | 并行性 |
|------|------|------------|--------|
| **训练** | 完整目标序列（右移一位） | 必须——防止偷看未来位置 | 所有位置可并行计算（因序列已知） |
| **推理** | 逐步增长的序列（自回归） | 形式上存在——但每步只有已生成的 token | 串行——每步生成一个 token |

训练时，Mask 是效率与正确性的保障：既允许一次性处理整个序列（高效），又确保不泄露未来信息（正确）。推理时，Mask 的约束自然满足——因为未来 token 本身还不存在，但代码中仍保留 Mask 以保持一致性。

## 要点

- **因果性保障**：Mask 的本质是保证因果关系——"未来不能影响过去"，这是自回归生成的基石。
- **训练效率的关键**：没有 Mask 就无法用 Teacher Forcing 并行训练，只能像推理一样逐步串行，训练速度将慢几个数量级。
- **Decoder-Only 模型也用它**：GPT 等模型虽然没有编码器，但生成时同样需要因果遮罩，Masked Self-Attention 是其唯一的注意力机制。
- **与编码器 Self-Attention 的对比**：编码器处理完整输入，不需要 Mask，所有位置互相可见；解码器需要 Mask，只能看到过去。

## 相关页面

- [[wiki/concepts/encoder-decoder|编码器-解码器架构]]
- [[wiki/concepts/self-attention|Self-Attention]]
- [[wiki/concepts/cross-attention|Cross-Attention]]
- [[wiki/concepts/multi-head-attention|Multi-Head Attention]]
