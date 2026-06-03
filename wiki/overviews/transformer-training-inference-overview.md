---
type: overview
tags: [深度学习, Transformer, 训练, 推理, 概述]
created: 2026-04-09
updated: 2026-04-09
sources:
  - raw/lessons/Transformer/07-training-inference.md
---

# Transformer 训练与推理全景概述

> 从"搭好架构"到"真正可用"——训练赋予模型灵魂，推理让模型产生价值。

## 概述

[[wiki/overviews/transformer-architecture-overview|Transformer 架构]] 定义了模型的骨架，但让这个骨架学会理解语言、生成文本，需要精心设计的训练流程；让训练好的模型高效地为用户服务，需要优化的推理策略。本页从训练和推理两个维度，呈现 Transformer 从理论到实践的完整图景。

## 训练流程关键步骤

### 损失函数：交叉熵

训练的核心目标是让模型对"正确的下一个 token"赋予尽可能高的概率。交叉熵损失衡量模型预测的概率分布与真实 one-hot 分布之间的距离：

$$L_{CE} = -\log p_\theta(w_t \mid w_{1:t-1})$$

对整个目标序列的每个位置计算交叉熵后取平均，作为训练的优化目标。直觉上，模型预测越准确（对真实词的概率越高），损失越小。

### Label Smoothing：防止过度自信

原版 Transformer 使用 $\epsilon = 0.1$ 的 Label Smoothing，将 one-hot 标签软化为 $(1-\epsilon)$ 给真实词、$\epsilon/K$ 给其他词的概率分布。这防止模型对训练标签过度自信，提升泛化能力。效果是训练集准确率略降，但测试集表现更好。

### 学习率调度：Warmup + Inverse Square Root

Transformer 的训练动态非常不稳定，原版论文设计了特殊的学习率公式：

$$lr = d_{model}^{-0.5} \cdot \min\left(step^{-0.5},\ step \cdot warmup^{-1.5}\right)$$

- **Warmup 阶段**（前 4000 步）：学习率从零线性增加，让模型有时间适应梯度方向变化，避免早期不稳定更新破坏已学到的表示。
- **衰减阶段**（4000 步之后）：学习率按 $1/\sqrt{step}$ 缓慢下降，实现精细调优。

### 正则化：多层 Dropout

原版 Transformer 在多个位置使用 Dropout（$p=0.1$）：注意力输出残差连接之前、FFN 输出残差连接之前、以及注意力权重 Softmax 之后。Dropout 通过随机让部分神经元"请假"，强制模型学习冗余表示，降低对单一路径的依赖。

### 梯度裁剪

训练中使用梯度范数裁剪（阈值 1.0），防止梯度爆炸导致训练崩溃。这是 Transformer 训练稳定性的重要保障。

## 推理流程关键步骤

### 解码策略

[[wiki/concepts/beam-search|解码策略]]决定如何从模型输出的概率分布中选择下一个 token：

- **贪心解码**：每步选概率最高的词。速度快，但可能陷入局部最优——早期次优选择可能让后续序列更优。
- **Beam Search**：维护 $k$ 条候选路径，每步扩展后保留概率最高的 $k$ 条。更可能找到全局较优序列，但计算量增加 $k$ 倍。适合翻译、摘要等需要确定性输出的任务。
- **Top-p (Nucleus Sampling)**：动态选择累积概率达到 $p$ 的最小词集合，在集合内采样。自适应控制多样性，是目前生成类任务的主流策略。
- **Temperature**：通过温度系数 $T$ 调节分布尖锐程度。$T<1$ 更确定，$T>1$ 更随机。

### KV Cache：推理加速核心

[[wiki/concepts/autoregressive-generation|自回归生成]]每一步都需要对已生成的完整序列重新计算注意力。[[wiki/concepts/kv-cache|KV Cache]] 的关键洞察是：已生成 token 的 Key 和 Value 向量在后续步骤中不会改变，因此可以缓存复用。

KV Cache 将推理复杂度从 $O(T^2)$ 降低到约 $O(T)$，是工业部署的标配优化。现代实现（如 vLLM）进一步使用 Paged Attention 分页管理 KV Cache，减少显存碎片。

### Teacher Forcing 与曝光偏差

训练时使用真实标签作为下一步输入（Teacher Forcing），推理时使用模型自己的预测。这种输入分布的不一致被称为**曝光偏差（Exposure Bias）**——模型没学过如何从自己的错误中恢复。解决方案包括 Scheduled Sampling（混合真实标签和模型预测）和课程学习（后期切换为 Free Running）。现代大型模型（GPT 系列）由于规模足够大，通常直接使用 Free Running。

## 训练 vs 推理核心差异

| 维度 | 训练阶段 | 推理阶段 |
|------|----------|----------|
| **输入来源** | 真实标签（Teacher Forcing） | 模型自身预测（自回归） |
| **解码方式** | 一次处理完整序列 | 逐 token 串行生成 |
| **Dropout** | 开启（随机丢弃神经元） | 关闭（全部神经元参与） |
| **计算并行性** | 高（矩阵运算可高度并行） | 低（自回归依赖上一步结果） |
| **注意力 Mask** | 训练用固定 Mask | 推理用因果 Mask |
| **KV Cache** | 不使用（Teacher Forcing 一次处理） | 使用（避免重复计算） |
| **优化目标** | 最小化交叉熵损失 | 选择合理的解码策略 |
| **瓶颈** | 训练时间和数据质量 | 生成速度和显存占用 |
| **Batch Size** | 尽量大（充分利用 GPU） | 通常为 1（单用户请求） |

## 核心概念索引

| 概念 | Wiki 页面 | 在训练/推理中的角色 |
|------|-----------|---------------------|
| 交叉熵损失 | — | 训练的优化目标 |
| Label Smoothing | — | 训练正则化，防止过度自信 |
| 学习率 Warmup | — | 训练稳定性保障 |
| Dropout | — | 训练正则化，推理时关闭 |
| 自回归生成 | [[wiki/concepts/autoregressive-generation]] | 推理的核心生成范式 |
| 解码策略 | [[wiki/concepts/beam-search]] | 推理时选择 token 的策略集合 |
| KV Cache | [[wiki/concepts/kv-cache]] | 推理加速的核心优化技术 |
| Transformer 架构 | [[wiki/overviews/transformer-architecture-overview]] | 训练和推理的基础架构 |

## 相关页面

- [[wiki/overviews/transformer-architecture-overview|Transformer 架构概述]] — 训练和推理的架构基础
- [[wiki/overviews/modern-llm-landscape|现代 LLM 格局概述]] — 现代大模型的训练规模与格局
- [[wiki/concepts/autoregressive-generation|自回归生成]] — 逐 token 生成的核心机制
- [[wiki/concepts/beam-search|解码策略]] — 贪心、Beam Search、Top-k、Top-p 详解
- [[wiki/concepts/kv-cache|KV Cache]] — 推理加速的核心技术
