---
type: comparison
tags: [深度学习, 归一化, LayerNorm, BatchNorm, Transformer]
created: 2026-04-09
updated: 2026-04-09
sources:
  - raw/lessons/Transformer/05-ffn-residual-layernorm.md
---

# LayerNorm vs BatchNorm：归一化方法的路线分野

> BatchNorm 沿 Batch 方向归一化，LayerNorm 沿特征方向归一化——这个看似微小的差异决定了它们各自在 CV 和 NLP 中的主导地位。

## 核心对比

| 维度 | Batch Normalization | Layer Normalization |
|------|--------------------|--------------------|
| **归一化方向** | 沿 Batch 方向（对所有样本的同一特征） | 沿特征方向（对同一样本的所有特征） |
| **统计量计算** | 跨样本，每个特征维度独立计算均值和方差 | 逐样本，单个样本所有特征联合计算均值和方差 |
| **Batch Size 依赖** | 强依赖（小 batch 统计不稳定） | 无依赖（每个样本独立归一化） |
| **训练/推理一致性** | 不一致（训练用 batch 统计，推理用滑动平均） | 完全一致（逐样本计算，无状态差异） |
| **适用架构** | CNN、图像模型（ResNet、EfficientNet） | Transformer、RNN（GPT、BERT、LLaMA） |
| **变长序列支持** | 差（padding 影响统计量，不同长度混合不稳定） | 好（每个 token 独立归一化，不受 padding 影响） |
| **参数量** | $2 \times d$（每特征一对 $\gamma, \beta$） | $2 \times d$（每特征一对 $\gamma, \beta$） |
| **计算位置** | 通常在卷积层之后、激活函数之前 | 在 Transformer 中作为 Add & Norm 的一部分 |

## 归一化方向的可视化

假设输入张量为 $(B, T, d)$——Batch 为 $B$，序列长度 $T$，特征维度 $d$：

```
BatchNorm（沿 Batch 方向归一化）：
  对每个特征维度 j，计算 B 个样本的均值和方差
  → 统计量 shape: (d,)
  → 依赖 batch 中其他样本

LayerNorm（沿特征方向归一化）：
  对每个样本，计算 d 个特征维度的均值和方差
  → 统计量 shape: (B, T)
  → 完全独立于其他样本
```

## 为什么 Transformer 选 LayerNorm 而不是 BatchNorm

### 原因一：变长序列问题

NLP 中句子长度差异大，batch 内通常需要 padding 到统一长度。BatchNorm 跨样本统计时：

- padding 位置的"噪声"会污染真实 token 的均值和方差
- 不同长度的句子混合时，统计量极不稳定

LayerNorm 对每个 token 独立归一化，只依赖自身 512 维特征，完全不受 padding 和其他样本影响。

### 原因二：Batch Size 敏感

大模型训练时 batch size 可能很小（受显存限制），BatchNorm 在小 batch 下统计量方差大、效果差。LayerNorm 不依赖 batch 维度，batch size 为 1 也能正常工作。

### 原因三：训练/推理行为一致性

BatchNorm 训练时用当前 batch 统计量，推理时用训练期间累积的滑动平均——两者存在 gap。LayerNorm 每次都是逐样本实时计算，训练和推理完全一致，无状态维护负担。

### 原因四：与残差连接的协同

Transformer 的 Add & Norm 结构中，残差连接将 $x + Sublayer(x)$ 传给 LayerNorm。LayerNorm 对单个 token 向量做归一化，天然适配这种"对每个位置独立操作"的模式。

> 类比：如果每个 token 是一个人，LayerNorm 让每个人用自己的健康指标评价自己，BatchNorm 是用群体平均值评价——前者更个人化、不受他人影响。

## Pre-LN vs Post-LN：归一化位置的演进

原始 Transformer 使用 Post-LN（归一化在残差之后），现代模型几乎统一采用 Pre-LN（归一化在子层之前）。

| 特性 | Post-LN（原始设计） | Pre-LN（现代主流） |
|------|-------------------|-------------------|
| **公式** | $LayerNorm(x + Sublayer(x))$ | $x + Sublayer(LayerNorm(x))$ |
| **训练稳定性** | 较差（需要 warm-up 阶段） | 更稳定（无需 warm-up 或更短） |
| **梯度流** | 子层输出到 Add 位置梯度分散 | 梯度直接通过残差分支流向所有子层 |
| **归一化对象** | 残差和 $(x + Sublayer(x))$ | 子层输入 $x$ |
| **使用现状** | 原始 Transformer、早期 BERT | GPT-2/3/4、LLaMA、几乎所有现代模型 |
| **理论性能上界** | 可能更高（归一化在更"下游"） | 略低但训练更充分后实际更好 |

**直觉理解**：Pre-LN 相当于把"格式统一"工作提前——在信息混合之前就做好标准化，让后续处理更稳定。这就像开会前先把所有报告统一格式，而不是开完会再整理。

## 发展趋势

```
BatchNorm (2015) —→ CV 领域标配
     │
     │ NLP 需要：不依赖 batch、支持变长序列
     ▼
LayerNorm (2016) —→ NLP/Transformer 领域标配
     │
     ├── Post-LN (2017) — 原始 Transformer
     │
     └── Pre-LN (2020+) — 现代大模型统一方案
          │
          └── 新趋势：RMSNorm（LayerNorm 简化版）
              — LLaMA 等模型采用
              — 去掉均值中心化，只做缩放
              — 计算更快，效果相当
```

## 相关页面

- [[wiki/concepts/layer-normalization]] — LayerNorm 概念详解
- [[wiki/summaries/05-ffn-residual-layernorm]] — FFN、残差连接与 LayerNorm 学习笔记
- [[wiki/comparisons/rnn-vs-transformer]] — RNN vs Transformer 架构对比
