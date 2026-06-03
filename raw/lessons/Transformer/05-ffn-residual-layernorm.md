---
type: lesson
tags:
  - 深度学习
  - Transformer
  - FFN
  - 残差连接
  - Layer Normalization
  - 前馈网络
created: 2026-04-09
updated: 2026-04-09
topic: FFN、残差连接与 Layer Normalization
difficulty: intermediate
prerequisites:
  - "[[02-self-attention|Self-Attention]]"
status: completed
series:
  name: Transformer 深度学习
  part: 5
---

# FFN、残差连接与 Layer Normalization

> 每个 token 的新表示是所有 token 的 Value 向量加权和——但注意力只是"收集情报"，真正的"深度分析"要靠 FFN；残差连接保证信息畅通无阻地流向深层，LayerNorm 则让每层输出保持稳定。

## 学习目标

- [ ] 理解 Position-wise FFN 的结构、升维降维的意义，以及 ReLU/GELU 的区别
- [ ] 掌握残差连接的"梯度高速公路"直觉，能解释为什么要 Add
- [ ] 熟练写出 LayerNorm 公式，能对比 BatchNorm/LayerNorm 的适用场景
- [ ] 能够画出一个完整 "Add & Norm" 子层的计算图
- [ ] 能从代码层面组装一个 Transformer 子层

## 前置知识

| 概念 | 来源 | 掌握程度 |
|------|------|----------|
| Self-Attention | [[02-self-attention]] | 理解 Q/K/V 的作用 |
| 矩阵乘法维度 | 线性代数 | 知道 $(B,T,d_{model})$ 的含义 |
| 激活函数 | 深度学习基础 | 知道 ReLU: $\max(0, x)$ |

## 直观理解

在 Transformer 的每一层中，Self-Attention 扮演着"情报收集员"的角色：它让每个 token 都能"看到"序列中所有其他 token 的信息，根据相关性加权求和。但收集情报不等于消化情报——一个人读了100份报告，如果不深入思考，认知不会提升。

FFN 就是那个"深度思考"的环节。它对每个位置的表示独立做非线性变换，把粗粒度的信息提炼成更抽象的特征。

残差连接则像会议室的"直达通道"：即使深度网络在分析过程中遗漏了某些关键信息，最原始的上下文还能通过这条捷径直接流到输出层，保证不会退步。

LayerNorm 是"格式统一员"：它把每一层的输出都标准化到均值为 0、方差为 1 的稳定分布，消除量纲差异，让后面的层能更稳定地学习。

## 1. Position-wise FFN

### 1.1 数学公式

$$FFN(x) = \max(0, xW_1 + b_1)W_2 + b_2$$

其中：
- $x \in \mathbb{R}^{d_{model}}$：单个 token 的输入向量（来自注意力层的输出）
- $W_1 \in \mathbb{R}^{d_{model} \times d_{ff}}$：第一层权重矩阵（升维）
- $W_2 \in \mathbb{R}^{d_{ff} \times d_{model}}$：第二层权重矩阵（降维）
- $d_{ff}$：FFN 中间层维度，通常设为 $4 \times d_{model}$

### 1.2 公式逐步拆解

**第一步：升维（信息扩展）**

$$h = xW_1 + b_1$$

把每个 token 的向量从 $d_{model}$ 维映射到 $d_{ff}$ 维。例如 GPT-2 中 $d_{model}=768$，升维后 $d_{ff}=768 \times 4 = 3072$。

为什么要升维？因为更高的维度意味着更大的"思考空间"——类似于把一句话扩展成一篇文章后，更容易找出重点。

**第二步：非线性激活**

$$h' = \max(0, h)$$

这是 ReLU 激活函数。它将负值置零，保留正值——可以理解为"筛选重要特征"：只有被判断为关键的信息才会保留下来。

**直觉理解**：ReLU 像一个"过滤器"，负值代表"噪声或无关信息"，正值代表"有意义的信号"。通过过滤，模型能聚焦于最显著的特征。

**第三步：降维（信息压缩）**

$$output = h'W_2 + b_2$$

把高维向量重新映射回原始维度 $d_{model}$。这是对"提炼后的信息"进行编码，使其能无缝衔接下一层。

### 1.3 为什么是 Position-wise？

"Position-wise"（逐位置）意味着：**同一个 FFN 网络独立地处理序列中的每一个位置**。

对输入 $X \in \mathbb{R}^{B \times T \times d_{model}}$：
- 不是把整个序列作为一个整体来处理
- 而是让每个时间步 $t$ 的向量 $x_t$ 独立通过同一个 FFN

这种设计的优点：
1. **计算可并行化**：各位置独立，同时计算效率高
2. **参数量少**：所有位置共享 $W_1, W_2$，不需要为每个位置学一套权重
3. **符合注意力输出的性质**：注意力层的输出已经是"每个位置的独立表示"（加权和信息已融合到各位置的向量中），FFN 只需在本地深化

> [!example]- PyTorch 实现
>
> ```python
> import torch
> import torch.nn as nn
>
> class FeedForward(nn.Module):
>     def __init__(self, d_model, d_ff, dropout=0.1):
>         super().__init__()
>         # 升维：d_model -> d_ff
>         self.linear1 = nn.Linear(d_model, d_ff)
>         # 降维：d_ff -> d_model
>         self.linear2 = nn.Linear(d_ff, d_model)
>         self.dropout = nn.Dropout(dropout)
>
>     def forward(self, x):
>         # x: (batch, seq_len, d_model)
>         x = self.linear1(x)      # (batch, seq_len, d_ff)
>         x = torch.relu(x)        # 非线性激活
>         x = self.dropout(x)
>         x = self.linear2(x)      # (batch, seq_len, d_model)
>         return x
> ```

### 1.4 激活函数：ReLU vs GELU

原始 Transformer（Vaswani et al., 2017）使用 **ReLU**。但现代大模型（如 BERT、GPT）更常用 **GELU**。

| 特性 | ReLU | GELU |
|------|------|------|
| 公式 | $\max(0, x)$ | $x \cdot \Phi(x) \approx 0.5x(1 + \tanh[\sqrt{2/\pi}(x + 0.044715x^3)])$ |
| 非线性类型 | 折线（不可导二阶） | 平滑曲线（任意阶可导） |
| 特性 | 稀疏激活（部分神经元恒为0） | 概率加权门控（依赖输入统计） |
| 计算效率 | 极快（比较运算） | 稍慢（需计算 $\tanh$） |
| 效果 | 强稀疏性 | 更细腻的特征建模 |

**GELU 的直觉**：GELU 相当于"软性 ReLU"，负值不是硬截断为 0，而是乘以一个小于 1 的系数——更像"概率性地衰减"而非"一刀切"。这让梯度流动更平滑。

BERT 实验显示：使用 GELU 在 MNLI 任务上比 ReLU 准确率提升约 0.5%。

### 1.5 FFN 的参数量

FFN 占据了 Transformer 的大部分参数。以 $d_{model}=512, d_{ff}=2048$ 为例：

- $W_1$: $512 \times 2048 \approx 100$ 万参数
- $W_2$: $2048 \times 512 \approx 100$ 万参数
- 总计约 200 万参数，占单层 Transformer 总参数（约 300 万）的 **约 65%**

这解释了为什么 FFN 被称为 Transformer 的"参数主力"。

## 2. 残差连接

### 2.1 公式

$$output = LayerNorm(x + Sublayer(x))$$

其中 $Sublayer(x)$ 可以是 Self-Attention，也可以是 FFN。残差连接将**原始输入** $x$ 与**子层的输出**相加。

### 2.2 为什么需要残差连接？

**问题：梯度消失**

在极深的网络中，反向传播时梯度需要层层回传。如果每层都对信号做大幅衰减，深层的梯度几乎为 0，导致前几层难以训练。

**残差的解决思路**

假设我们要学习一个恒等映射 $H(x) = x$。如果没有残差，网络需要直接拟合 $F(x) = x$；有了残差，网络只需拟合 $F(x) = H(x) - x$，即"相对于输入的偏差"。

当最优映射接近恒等时，让 $F(x) \approx 0$ 比让 $F(x) \approx x$ 容易得多——这叫**"easy to learn"** 原则。

**类比：会议记录的场景**

假设你是会议记录员，你需要记录原始发言（x），然后做摘要提炼（Sublayer(x)）。如果没有残差，你只能依赖摘要内容；如果有残差，你的最终笔记 = 原始笔记 + 摘要提炼——即使摘要有遗漏，原始内容也不会丢失。

### 2.3 ResNet 的类比

残差连接最早由 He et al. (2016) 在 ResNet 中提出，用于图像分类。Transformer 直接借鉴了这一思想。

ResNet 中的残差块：

```
输入 x → Linear → ReLU → Linear → 输出 F(x) + x
              ↑
           短路连接
```

Transformer 中的子层：

```
输入 x → Sublayer(x) → 输出 F(x) + x
        ↑
     短路连接
```

两者本质相同：提供一条绕过非线性变换的梯度高速公路。

### 2.4 如果去掉残差连接会怎样？

| 场景 | 现象 | 原因 |
|------|------|------|
| 训练不稳定 | 梯度爆炸/消失，loss 震荡 | 梯度路径变长，信号难以回传 |
| 深层无效 | 增加层数反而降低性能 | 深层的非线性变换难以训练 |
| 信息丢失 | 浅层的重要信息无法传递到深层 | 没有直接的"记忆通道" |

典型实验：对于 110 层的 ResNet，如果没有残差连接，训练误差比 20 层网络还高；但有残差连接，110 层反而比 20 层更优。

## 3. Layer Normalization

### 3.1 公式拆解

对单个样本 $x \in \mathbb{R}^{d_{model}}$，LayerNorm 的计算：

$$\mu = \frac{1}{d_{model}} \sum_{i=1}^{d_{model}} x_i \quad \text{(均值)}$$

$$\sigma^2 = \frac{1}{d_{model}} \sum_{i=1}^{d_{model}} (x_i - \mu)^2 \quad \text{(方差)}$$

$$\hat{x}_i = \frac{x_i - \mu}{\sqrt{\sigma^2 + \epsilon}} \quad \text{(标准化)}$$

$$y_i = \gamma_i \hat{x}_i + \beta_i \quad \text{(仿射变换)}$$

其中 $\gamma, \beta$ 是可学习的缩放和偏移参数，$\epsilon$（通常 $10^{-6}$）是防止除零的小常数。

**逐步拆解**：

1. **计算均值 $\mu$**：对向量所有维度求平均，得到一个标量——代表这个 token 表示的"整体大小"
2. **计算方差 $\sigma^2$**：衡量各维度偏离均值的程度——代表"内部不一致性"
3. **标准化 $\hat{x}$**：把每个维度都"居中"到均值为 0、方差为 1——消除量纲差异
4. **仿射变换 $y$**：允许模型学习最优的"重新缩放"和"平移"——因为标准化可能不是最优操作

### 3.2 LayerNorm vs BatchNorm

这是 NLP 和 CV 模型中最核心的归一化方法对比。

| 特性 | Batch Normalization | Layer Normalization |
|------|--------------------|--------------------|
| 归一化维度 | 沿 Batch 方向（对所有样本的同一特征） | 沿特征方向（对同一样本的所有特征） |
| 统计量来源 | 依赖当前 Batch 的均值和方差 | 仅依赖当前单个样本 |
| 输入依赖 | 训练时依赖 Batch size，推理时固定 | 每个样本独立，不依赖 Batch |
| 适用场景 | CV（图像，Batch 通常较大） | NLP（序列，长度可变） |
| 对序列建模 | 欠佳（padding、变长问题） | 自然适配（逐 token 归一化） |
| 训练/推理行为差异 | 训练用 Batch 统计，推理用滑动平均 | 完全一致 |

**NLP 为什么用 LayerNorm？**

考虑一个 batch 的句子："我 喜欢 学习"（3 tokens）和 "深度 学习 是 技术"（5 tokens，padding 到 5）。

用 BatchNorm：假设 embedding 维度是 512，那么每个 token 位置的均值/方差是从 batch 里所有样本的同一位置计算的。这意味着：

- padding 位置的"噪声"会影响真实 token 的归一化
- 不同长度的句子混合时，统计不稳定

用 LayerNorm：每个 token 的归一化只依赖自己向量内部的 512 维，完全独立于其他样本或其他 token。

**类比**：如果把每个 token 看作一个人，LayerNorm 就像让每个人用自己的身高、体重、年龄来评价自己的健康指标，而不是用群体平均值——个人化、且不受他人影响。

### 3.3 Post-LN vs Pre-LN

原始 Transformer 论文使用的是 **Post-LN**（后置 LayerNorm），后续研究发现 **Pre-LN**（前置 LayerNorm）训练更稳定。

**Post-LN（原始设计）**：

```
x → Attention → Add & Norm → FFN → Add & Norm
                     ↑               ↑
                  Post-LN         Post-LN
```

**Pre-LN（现代主流）**：

```
x → LayerNorm → Attention → Add → LayerNorm → FFN → Add
              ↑                                      ↑
           Pre-LN                                  Pre-LN
```

| 特性 | Post-LN | Pre-LN |
|------|---------|--------|
| 训练稳定性 | 较差（需要 warm-up） | 更稳定 |
| 梯度流 | 子层输出到 Add 位置梯度分散 | 梯度直接流向所有子层 |
| 理论解释 | 层归一化在残差分支末端 | 归一化在残差分支开头 |
| 使用现状 | 原始论文 | 绝大多数现代模型 |

**直觉理解**：Pre-LN 相当于把"格式统一"工作提前——在信息混合之前就做好标准化，让后续处理更稳定。

## 4. Add & Norm：三者协作

"Add & Norm" 是 Transformer 的标志性结构，本质上是残差连接 + LayerNorm 的组合。

### 4.1 数学形式

$$Output = LayerNorm(x + Sublayer(x))$$

这看似简单的一个公式，实际上是一个完整的"子层处理单元"：

1. **Add（残差连接）**：$x + Sublayer(x)$ — 保证梯度流通，保留原始信息
2. **Norm（LayerNorm）**：对相加结果做归一化 — 保证数值稳定

### 4.2 计算图视角

```
         x
         │
         ├──────────────────┐
         │                  │
         ▼                  │
    ┌─────────┐             │
    │Sublayer │             │
    │(Attn or │             │
    │  FFN)   │             │
    └────┬────┘             │
         │ Sublayer(x)      │
         │                  │
         ▼                  │
      ┌──────┐              │
      │ Add  │ ◄────────────┘
      └──┬──┘         x (skip connection)
         │
         ▼
  ┌─────────────┐
  │ LayerNorm   │
  └──────┬──────┘
         │
         ▼
      output
```

这条路径保证：**即使 Sublayer 训练得不好，信息仍然能通过残差连接流向深层，网络不会退化**。

### 4.3 完整的 Transformer 子层

把 Attention + FFN + Add & Norm 组装在一起：

> [!example]- 完整子层实现
>
> ```python
> import torch
> import torch.nn as nn
> import torch.nn.functional as F
> import math
>
> class TransformerSubLayer(nn.Module):
>     """一个完整的 Transformer 子层（Attention + FFN + Add&Norm）"""
>     def __init__(self, d_model, d_ff, n_heads, dropout=0.1):
>         super().__init__()
>         # Self-Attention
>         self.attention = nn.MultiheadAttention(d_model, n_heads, dropout=dropout, batch_first=True)
>         self.attn_norm = nn.LayerNorm(d_model)
>
>         # FFN
>         self.ffn = FeedForward(d_model, d_ff, dropout)
>         self.ffn_norm = nn.LayerNorm(d_model)
>
>     def forward(self, x, mask=None):
>         # Pre-LN 风格：先归一化再做子层
>         # Self-Attention 子层
>         x_norm = self.attn_norm(x)
>         attn_out, _ = self.attention(x_norm, x_norm, x_norm, attn_mask=mask)
>         x = x + attn_out  # 残差连接
>
>         # FFN 子层
>         x_norm = self.ffn_norm(x)
>         ffn_out = self.ffn(x_norm)
>         x = x + ffn_out  # 残差连接
>
>         return x
> ```

## 5. 常见误区

| 误区 | 正确理解 |
|------|----------|
| FFN 和 Self-Attention 可以并行 | 它们是串行执行的：Attention 先融合跨位置信息，FFN 再对每个位置做独立深化 |
| 残差连接是"加法"所以很简单 | 残差的"简单"在于它让网络学习"偏差"而非"完整映射"，大大降低了学习难度 |
| LayerNorm 和 BatchNorm 本质相同 | LayerNorm 逐样本归一化，适合变长序列；BatchNorm 跨样本归一化，适合固定结构 |
| Pre-LN 比 Post-LN 永远更好 | Pre-LN 训练更稳定，但 Post-LN 在某些任务上仍可达到更好性能 |
| FFN 中间维度越大越好 | 中间维度影响参数量和表达能力，需要在效率和效果间权衡（通常 $4 \times d_{model}$） |

## 6. 思考题

> [!hint]- 基础题 1：FFN 为什么叫 "Position-wise"？
>
> 提示：想想 FFN 是对 batch 中哪个维度独立操作的。

> [!success]- 基础题 1 答案
>
> "Position-wise" 是因为 FFN 对序列中**每个位置独立应用相同的变换**。对于输入 $X \in \mathbb{R}^{B \times T \times d_{model}}$，FFN 对每个 token 位置 $t$ 的向量 $x_t$ 独立通过相同的权重矩阵 $W_1, W_2$，而不是对整个序列做联合变换。同一 batch 中不同位置的 token 共享参数，这也是"position-wise"的另一层含义——位置间参数共享。

---

> [!hint]- 基础题 2：残差连接为什么能缓解梯度消失？
>
> 提示：从反向传播的链式法则角度思考，加入 Add 节点后梯度如何分流。

> [!success]- 基础题 2 答案
>
> 反向传播时，经过 Add 节点的梯度会**原封不动地传向上游**（因为 $\frac{\partial (x + F(x))}{\partial x} = 1$）。这提供了一条"梯度高速公路"，即使 $F(x)$ 的路径上梯度很小，输入 $x$ 仍然能收到有效的梯度更新。从信息流角度，残差连接让浅层参数能直接接收深层传回的监督信号，无需层层回传。

---

> [!hint]- 进阶题：Pre-LN 将 LayerNorm 放在残差连接内部，Post-LN 放在外部，这两者在数学上是否等价？
>
> 提示：考虑 $LayerNorm(x + Sublayer(x))$ vs $x + Sublayer(LayerNorm(x))$。

> [!success]- 进阶题 答案
>
> **不等价**。原因：
> 1. LayerNorm 不是线性操作，它会改变数据的尺度分布
> 2. Post-LN：归一化作用于 $(x + Sublayer(x))$，即对残差和做统一归一化
> 3. Pre-LN：归一化仅作用于 $x$ 和 $Sublayer(x)$ 的输入，让两者在归一化后的空间做加法
>
> 这种差异导致：Pre-LN 的梯度更均衡（每层都有归一化），训练更稳定；Post-LN 的表达能力可能更强（归一化在更"下游"），但训练困难。实践表明 Pre-LN 更适合深度 Transformer（20+ 层）。

## 7. 关键要点回顾

1. **FFN 是位置独立的非线性变换**：通过升维→激活→降维，对每个 token 做深度特征提取，中间层 $4 \times d_{model}$ 是经验最优配置

2. **残差连接 = 梯度高速公路**：让网络学习"偏差"而非"完整映射"，保证深层网络可训练，是深度学习最重要的技巧之一

3. **LayerNorm 是个性化归一化**：对单个样本的所有特征做归一化，不依赖 Batch 统计，自然适配变长序列，是 NLP 的首选归一化方法

4. **Add & Norm 是 Transformer 的稳定器**：残差保证信息流通，LayerNorm 保证数值稳定，两者缺一不可

5. **Pre-LN vs Post-LN 的权衡**：Pre-LN 训练更稳，Post-LN 理论表达更强；现代模型大多采用 Pre-LN

## 8. 扩展阅读

- **原论文**：[Attention Is All You Need](https://arxiv.org/abs/1706.03762) — Section 3.3（FFN）和 Section 5.4（Layer Normalization）
- **ResNet 残差连接**：[Deep Residual Learning for Image Recognition](https://arxiv.org/abs/1512.03385) — He et al., 2016
- **GELU 激活函数**：[Gaussian Error Linear Units (GELU)](https://arxiv.org/abs/1606.08415) — Hendrycks and Gimpel, 2016
- **LayerNorm 详细分析**：[On Layer Normalization in the Pre-Training Transformer](https://arxiv.org/abs/2002.04745) — 理论分析 Pre-LN 的有效性

## 9. 下一步学习

- [[06-encoder-decoder|编码器-解码器架构]]：了解完整的 Transformer 编码器-解码器结构，以及它们如何协作完成序列到序列的任务
- [[04-positional-encoding|位置编码]]：回顾 Transformer 如何将序列位置信息注入模型

---

**系列目录**：
- 第 1 篇：[[01-why-transformer|为什么需要 Transformer]]
- 第 2 篇：[[02-self-attention|Self-Attention]]
- 第 3 篇：[[03-multi-head-attention|Multi-Head Attention]]
- 第 4 篇：[[04-positional-encoding|位置编码]]
- **第 5 篇：本篇**
- 第 6 篇：[[06-encoder-decoder|编码器-解码器架构]]
