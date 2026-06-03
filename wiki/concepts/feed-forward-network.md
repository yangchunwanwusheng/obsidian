---
type: concept
tags: [深度学习, Transformer, FFN, 前馈网络, Position-wise FFN]
created: 2026-04-09
updated: 2026-04-09
sources:
  - raw/lessons/Transformer/05-ffn-residual-layernorm.md
---

# 前馈网络（Feed-Forward Network）

> Position-wise FFN 是 Transformer 中对每个 token 独立做非线性深度特征提取的模块，通过升维-激活-降维的三步变换，承担了 Transformer 约 2/3 的参数量。

## 概述

在 Transformer 的每一层中，Self-Attention 是"情报收集员"——让每个 token 看到序列中所有其他 token 的信息并加权融合。但收集情报不等于消化情报。FFN 就是那个"深度思考"的环节：它对 Attention 输出的每个 token 表示独立做非线性变换，把粗粒度的融合信息提炼成更抽象的特征。

如果说 Attention 负责"看全局"，FFN 就负责"想深层"。两者串行协作，缺一不可。

## 核心公式

$$FFN(x) = W_2 \cdot \sigma(W_1 \cdot x + b_1) + b_2$$

其中 $\sigma$ 为激活函数（ReLU 或 GELU）。

**逐步拆解**：

1. **升维**：$h = W_1 x + b_1$，将 $d_{model}$ 维映射到 $d_{ff}$ 维（通常 $d_{ff} = 4 \times d_{model}$），扩展"思考空间"
2. **激活**：$h' = \sigma(h)$，引入非线性，筛选关键特征
3. **降维**：$output = W_2 h' + b_2$，将高维表示压缩回 $d_{model}$ 维，衔接下一层

## 关键设计

### 升维 4x 的意义

GPT-2 中 $d_{model}=768$，升维后 $d_{ff}=3072$。更高的维度意味着更大的特征空间——类比把一句话扩展成一段论述后，更容易提炼出关键论点。$4\times$ 是经验最优配置，在表达力和效率间取得平衡。

### Position-wise 的含义

"Position-wise"指同一个 FFN 网络独立处理序列中的每个位置——各 token 的向量互不影响，且共享权重。这带来三个好处：计算可并行化、参数量少（所有位置共享 $W_1, W_2$）、符合 Attention 输出的性质（融合后的信息已在各位置向量中）。

## 激活函数对比

| 特性 | ReLU | GELU |
|------|------|------|
| 公式 | $\max(0, x)$ | $x \cdot \Phi(x)$（近似为 $0.5x(1+\tanh[\sqrt{2/\pi}(x+0.044715x^3)])$） |
| 非线性类型 | 硬截断（折线） | 平滑曲线（任意阶可导） |
| 对负值 | 直接置零 | 概率性衰减（软性截断） |
| 梯度流 | 0 点处不可导 | 处处平滑，梯度流更稳定 |
| 使用者 | 原始 Transformer | BERT、GPT 等现代模型 |

GELU 的直觉：相当于"软性 ReLU"，负值不是一刀切为零，而是乘以一个小于 1 的系数，更像是"概率性地衰减"。这让梯度流动更平滑，建模更细腻。

## FFN 占 Transformer 参数量的比例

以 $d_{model}=512, d_{ff}=2048$ 为例：

- $W_1$：$512 \times 2048 \approx 100$ 万参数
- $W_2$：$2048 \times 512 \approx 100$ 万参数
- FFN 总计约 200 万参数，占单层 Transformer 总参数的 **约 65%**

FFN 是 Transformer 的"参数主力"。这也解释了为什么模型压缩和加速的工作常从 FFN 入手（如 MoE 稀疏激活）。

## 要点

- FFN 是位置独立的非线性变换：Attention 融合跨位置信息，FFN 对每个位置独立深化——两者角色互补
- 升维 4x → 激活 → 降维的三步结构是经典设计，中间维度提供充足的"思考空间"
- GELU 替代 ReLU 是现代趋势：平滑的非线性带来更稳定的梯度流和更细腻的特征建模
- FFN 占单层 Transformer 约 2/3 参数，是模型的参数主体
- Position-wise 设计保证了完全可并行化，且各位置共享权重，参数高效

## 相关页面

- [[wiki/concepts/self-attention]] — Attention 与 FFN 串行协作，共同构成 Transformer 层
- [[wiki/concepts/positional-encoding]] — 位置编码与词嵌入相加后进入 Attention-FFN 流水线
- [[05-ffn-residual-layernorm|第 5 章：FFN、残差连接与 Layer Normalization]] — 完整课程笔记（含残差连接和 LayerNorm 的详细内容）
