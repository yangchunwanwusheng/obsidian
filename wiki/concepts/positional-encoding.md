---
type: concept
tags: [深度学习, Transformer, 位置编码, Positional Encoding, 正弦编码, RoPE]
created: 2026-04-09
updated: 2026-04-09
sources:
  - raw/lessons/Transformer/04-positional-encoding.md
---

# 位置编码（Positional Encoding）

> 通过向词嵌入注入基于数学函数的序列位置信号，解决 Self-Attention 置换等变性问题，使 Transformer 能够区分"谁对谁做了什么"。

## 概述

Self-Attention 本身是**置换等变**的——打乱输入顺序，输出只是重排版本。这意味着对 Attention 而言，"狗咬人"和"人咬狗"完全等价。位置编码的职责就是在进入 Attention 之前，将"你在序列中的位置"这一信息注入每个 token 的表示中。

词嵌入告诉模型"这是什么"，位置编码告诉模型"在哪里"——两者缺一不可。

## 正弦位置编码核心公式

原始 Transformer 论文（Vaswani et al., 2017）使用固定公式生成位置编码：

$$PE_{(pos, 2i)} = \sin\left(\frac{pos}{10000^{2i/d_{model}}}\right)$$

$$PE_{(pos, 2i+1)} = \cos\left(\frac{pos}{10000^{2i/d_{model}}}\right)$$

**公式解读**：偶数维度用 sin，奇数维度用 cos。维度索引 $i$ 越大，频率 $\omega_i = 1/10000^{2i/d_{model}}$ 越低——从 1 递减到 1/10000，形成对数均匀的几何级数。高频分量精确区分相邻 token，低频分量捕捉大尺度位置关系。

**物理类比**：像乐队中的低音鼓和高音哨——低频告诉你"已经进行到歌曲的哪一段"，高频告诉你"当前在哪个节拍"。

## 关键性质

### 线性性质（泛化的关键）

存在线性变换（旋转矩阵）$M$ 使得 $PE_{pos+k} \approx M^k \cdot PE_{pos}$。这意味着从任意位置的编码可以线性推算出其他位置的编码。这正是正弦编码外推能力的来源——不需要查表，用连续函数即可生成任意位置的编码。

### 外推能力

模型在训练时见过位置 0~1000，推理时遇到位置 1200，正弦编码仍能给出合理的向量；而可学习编码对未见位置只能靠插值，效果差。

## 位置编码方案对比

| 方案 | 年份 | 核心思想 | 参数量 | 外推能力 | 代表模型 |
|------|------|---------|--------|---------|---------|
| **正弦编码** | 2017 | 三角函数生成固定编码 | $O(1)$ | 强 | 原始 Transformer |
| **可学习编码** | 2018 | 为每个位置学一个向量 | $O(n_{max} \cdot d)$ | 弱 | BERT、GPT-2 |
| **RoPE** | 2021 | 对 Q/K 做旋转变换，使 attention score 只依赖相对位置 | $O(1)$ | 强 | LLaMA、GLM-4 |
| **ALiBi** | 2022 | 在 attention score 上加线性距离偏置 | $O(1)$ | 强 | BLOOM |

**实践选择**：训练数据充足时，可学习编码往往表现更好（BERT、GPT 系列）；需要长序列外推时，正弦编码或 RoPE/ALiBi 更优。

## 要点

- Self-Attention 置换等变，本身不含位置信息——位置编码是必需的输入增强，不是 Attention 的一部分
- 正弦编码的频率几何级数设计，使其能同时捕捉近处和远处的位置关系
- 线性性质（旋转矩阵变换）赋予正弦编码天然的外推能力，不受训练长度限制
- RoPE 是当前主流方案，将位置信息融入 Q/K 旋转而非简单加到嵌入上
- 位置编码不仅编码"第几个"，还编码位置间的相对距离关系——这对 Attention 计算至关重要

## 相关页面

- [[wiki/concepts/self-attention]] — Self-Attention 的置换等变性是位置编码存在的原因
- [[wiki/concepts/feed-forward-network]] — 位置编码与词嵌入相加后，经 Attention 再进入 FFN
- [[02-self-attention|第 2 章：Self-Attention]] — 详细课程
- [[04-positional-encoding|第 4 章：位置编码]] — 完整课程笔记
