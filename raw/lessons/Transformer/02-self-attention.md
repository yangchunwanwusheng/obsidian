---
type: lesson
tags:
  - 深度学习
  - Transformer
  - Self-Attention
  - 自注意力
  - QKV
  - 点积注意力
created: 2026-04-09
updated: 2026-04-09
topic: Self-Attention——序列自我审视的核心引擎
difficulty: intermediate
prerequisites:
  - "[[01-why-transformer|为什么需要 Transformer]]"
status: completed
series:
  name: Transformer 深度学习
  part: 2
---

# Self-Attention：序列自我审视的核心引擎

> 深入拆解 Scaled Dot-Product Self-Attention 的每一个细节——从直觉类比到完整的手推计算，从公式推导到代码实现。学完本章，你将真正理解 Transformer 最核心的公式。

## 学习目标

- [ ] 能解释 Self-Attention 与传统 Attention 的本质区别
- [ ] 理解 Query、Key、Value 三个角色的直觉含义与物理意义
- [ ] **能手推一个完整的 Self-Attention 计算示例**（从输入到最终输出）
- [ ] 理解缩放因子 $\sqrt{d_k}$ 的数学推导与必要性
- [ ] 能写出 Self-Attention 的 PyTorch 实现

## 前置知识

| 知识点 | 要求 | Wiki 参考 |
|--------|------|-----------|
| Transformer 的动机 | 理解为什么需要抛弃 RNN | [[01-why-transformer]] |
| 矩阵乘法 | 能做矩阵乘法运算 | — |
| Softmax 函数 | 知道它把向量变成概率分布 | — |
| 词嵌入 | 知道词可以表示为向量 | — |

---

## 直观理解：两个互补的类比

### 类比一：搜索引擎

你在 Google 搜索框中输入"Transformer 深度学习"——这就是 **Query**（你在找什么）。Google 的索引库中，每个网页都有一组标签和关键词——这就是 **Key**（网页能匹配什么）。当你搜索时，Google 把你的 Query 和所有网页的 Key 进行匹配，找到最相关的网页，然后返回它们的实际内容——这就是 **Value**（网页的内容）。

Self-Attention 做的就是这件事：**序列中的每个词都像搜索引擎一样，发出自己的 Query，与所有词的 Key 匹配，然后取回对应的 Value。**

### 类比二：小组讨论会

5 人小组讨论会中，每个人都可以向其他人提问（Query），每个人都有一个"标签"标明自己的专长（Key），以及实际掌握的知识（Value）。

当 Alice 发言时，她的 Query 与所有人的 Key 匹配——如果 Bob 的 Key 与 Alice 的 Query 高度相关，Alice 就会重点关注 Bob 提供的 Value。

**Self-Attention 就是让序列中的每个词像讨论会成员一样，自动找到与自己最相关的其他词，并加权整合它们的信息。**

> [!success] 两个类比的互补
> 搜索引擎类比强调"信息检索"——找最相关的内容。小组讨论类比强调"互相交流"——每个人同时是提问者和被提问者。Self-Attention 兼具两种特性。

---

## 一、从传统 Attention 到 Self-Attention

### 1.1 传统 Attention（回顾）

在 [[01-why-transformer|上一章]] 学到的 Bahdanau Attention 中：

- **Query**：来自解码器的隐藏状态 $s_{t-1}$
- **Key**：来自编码器的隐藏状态 $h_i$
- **Value**：也是 $h_i$

关键特征：Query 来自"外部"（解码器），Key/Value 来自编码器。这是一场**跨序列的对话**。

### 1.2 Self-Attention 的核心变化

Self-Attention 中，**Query、Key、Value 全部来自同一个序列**：

$$Q = X \cdot W_Q, \quad K = X \cdot W_K, \quad V = X \cdot W_V$$

其中 $X \in \mathbb{R}^{n \times d_{model}}$ 是输入序列的嵌入矩阵（$n$ 个 token，每个 $d_{model}$ 维），$W_Q, W_K, W_V$ 是三个可学习的投影矩阵。

> [!success] "Self" 的含义
> Self-Attention 之所以叫"Self"，是因为序列在"自我对齐"——每个位置通过学习到的投影矩阵，找到序列内部对自己最有价值的其他位置。没有外部的解码器参与，纯粹是序列内部的自我审视。

---

## 二、Q/K/V 的三个角色：深度理解

### 2.1 为什么需要三个角色？

Q/K/V 的设计借鉴了**信息检索系统**，但比检索系统更精巧：

| 角色 | 信息检索类比 | 在 Self-Attention 中的含义 | 直觉 |
|------|-------------|--------------------------|------|
| **Query (Q)** | 搜索关键词 | "我在找什么样的信息？" | 每个词发出的"提问" |
| **Key (K)** | 网页的标题/标签 | "我能提供什么样的信息？" | 每个词的"自我介绍" |
| **Value (V)** | 网页的实际内容 | "我携带的具体信息" | 每个词的"真实内容" |

### 2.2 投影矩阵的物理意义

为什么不直接用输入向量 $X$ 作为 Q/K/V？

$$Q = X W_Q, \quad K = X W_K, \quad V = X W_V$$

**原因**：投影矩阵将同一个输入 $X$ 映射到三个**不同的语义空间**：

- $W_Q$ 把 $X$ 映射到"**提问空间**"——"我在找什么"
- $W_K$ 把 $X$ 映射到"**被检索空间**"——"我是什么类型的"
- $W_V$ 把 $X$ 映射到"**内容空间**"——"我的实际内容"

这让模型可以学到：**同一个词在不同角色下有不同的表示**。例如，"苹果"作为 Query 可能在寻找"水果"相关信息，但作为 Key 可能在标示"科技公司"的身份。

### 2.3 维度追踪

设输入序列有 $n$ 个 token，嵌入维度为 $d_{model}$：

$$\underbrace{X}_{n \times d_{model}} \cdot \underbrace{W_Q}_{d_{model} \times d_k} = \underbrace{Q}_{n \times d_k}$$

$$\underbrace{X}_{n \times d_{model}} \cdot \underbrace{W_K}_{d_{model} \times d_k} = \underbrace{K}_{n \times d_k}$$

$$\underbrace{X}_{n \times d_{model}} \cdot \underbrace{W_V}_{d_{model} \times d_v} = \underbrace{V}_{n \times d_v}$$

在原始 Transformer 中，$d_k = d_v = d_{model} / h = 64$（$h=8$ 个头，$d_{model}=512$）。

---

## 三、完整手推：从输入到输出的每一步

> [!important] 核心章节
> 这是最重要的部分。我们将用一个具体的数值例子，从输入矩阵开始，一步一步算到最终输出矩阵——**不跳步、不断裂**。

### 3.0 场景设定

输入句子："猫 坐 在 垫子"（4 个 token），嵌入维度 $d_{model}=4$，投影维度 $d_k = d_v = 3$。

### 3.1 第一步：输入嵌入矩阵 $X$

假设已经完成词嵌入（暂不考虑位置编码）：

$$X = \begin{bmatrix} 1.0 & 0.0 & 0.5 & 0.3 \\ 0.2 & 0.8 & 0.1 & 0.5 \\ 0.4 & 0.3 & 0.9 & 0.2 \\ 0.6 & 0.1 & 0.3 & 0.7 \end{bmatrix} \begin{matrix} \text{猫} \\ \text{坐} \\ \text{在} \\ \text{垫子} \end{matrix}$$

### 3.2 第二步：生成 Q、K、V

假设学习到的投影矩阵：

$$W_Q = \begin{bmatrix} 0.1 & 0.3 & 0.5 \\ 0.2 & 0.1 & 0.4 \\ 0.3 & 0.5 & 0.2 \\ 0.4 & 0.2 & 0.3 \end{bmatrix}, \quad W_K = \begin{bmatrix} 0.5 & 0.1 & 0.3 \\ 0.2 & 0.4 & 0.1 \\ 0.3 & 0.2 & 0.5 \\ 0.1 & 0.3 & 0.4 \end{bmatrix}, \quad W_V = \begin{bmatrix} 0.3 & 0.2 & 0.5 \\ 0.1 & 0.4 & 0.3 \\ 0.5 & 0.1 & 0.2 \\ 0.2 & 0.3 & 0.4 \end{bmatrix}$$

**计算 $Q = X W_Q$**（以第 1 行"猫"为例）：

$$q_1 = \begin{bmatrix} 1.0 \times 0.1 + 0.0 \times 0.2 + 0.5 \times 0.3 + 0.3 \times 0.4 \\ 1.0 \times 0.3 + 0.0 \times 0.1 + 0.5 \times 0.5 + 0.3 \times 0.2 \\ 1.0 \times 0.5 + 0.0 \times 0.4 + 0.5 \times 0.2 + 0.3 \times 0.3 \end{bmatrix} = \begin{bmatrix} 0.37 \\ 0.61 \\ 0.69 \end{bmatrix}$$

类似地计算其余行和 $K, V$，得到：

$$Q = \begin{bmatrix} 0.37 & 0.61 & 0.69 \\ 0.38 & 0.30 & 0.49 \\ 0.49 & 0.56 & 0.41 \\ 0.38 & 0.37 & 0.62 \end{bmatrix}, \quad K = \begin{bmatrix} 0.70 & 0.33 & 0.62 \\ 0.31 & 0.48 & 0.33 \\ 0.56 & 0.28 & 0.55 \\ 0.47 & 0.32 & 0.51 \end{bmatrix}$$

$$V = \begin{bmatrix} 0.58 & 0.17 & 0.71 \\ 0.25 & 0.49 & 0.37 \\ 0.59 & 0.23 & 0.32 \\ 0.41 & 0.29 & 0.63 \end{bmatrix}$$

### 3.3 第三步：计算注意力分数 $S = QK^T$

$$S = QK^T = \begin{bmatrix} 0.37 & 0.61 & 0.69 \\ 0.38 & 0.30 & 0.49 \\ 0.49 & 0.56 & 0.41 \\ 0.38 & 0.37 & 0.62 \end{bmatrix} \begin{bmatrix} 0.70 & 0.31 & 0.56 & 0.47 \\ 0.33 & 0.48 & 0.28 & 0.32 \\ 0.62 & 0.33 & 0.55 & 0.51 \end{bmatrix}$$

以第 1 行"猫"为例，计算它对所有 token 的原始关注度：

$$s_{11} = 0.37 \times 0.70 + 0.61 \times 0.33 + 0.69 \times 0.62 = 0.259 + 0.201 + 0.428 = 0.888$$

$$s_{12} = 0.37 \times 0.31 + 0.61 \times 0.48 + 0.69 \times 0.33 = 0.115 + 0.293 + 0.228 = 0.636$$

$$s_{13} = 0.37 \times 0.56 + 0.61 \times 0.28 + 0.69 \times 0.55 = 0.207 + 0.171 + 0.380 = 0.757$$

$$s_{14} = 0.37 \times 0.47 + 0.61 \times 0.32 + 0.69 \times 0.51 = 0.174 + 0.195 + 0.352 = 0.721$$

类似地计算其余行，得到：

$$S = \begin{bmatrix} 0.888 & 0.636 & 0.757 & 0.721 \\ 0.672 & 0.418 & 0.586 & 0.549 \\ 0.743 & 0.524 & 0.649 & 0.611 \\ 0.739 & 0.468 & 0.639 & 0.603 \end{bmatrix} \begin{matrix} \text{猫} \\ \text{坐} \\ \text{在} \\ \text{垫子} \end{matrix}$$

**解读**：$S$ 的第 $(i, j)$ 元素表示第 $i$ 个 token 对第 $j$ 个 token 的"原始关注度"。注意 $S$ 是 $4 \times 4$ 的——每个 token 对所有 token（包括自己）都有一个分数。

### 3.4 第四步：缩放 $S_{scaled} = S / \sqrt{d_k}$

$d_k = 3$，所以 $\sqrt{d_k} = \sqrt{3} \approx 1.732$。

$$S_{scaled} = \frac{S}{1.732} = \begin{bmatrix} 0.513 & 0.367 & 0.437 & 0.416 \\ 0.388 & 0.241 & 0.338 & 0.317 \\ 0.429 & 0.302 & 0.375 & 0.353 \\ 0.427 & 0.270 & 0.369 & 0.348 \end{bmatrix}$$

### 3.5 第五步：Softmax 归一化

对 $S_{scaled}$ 的**每一行**独立应用 Softmax。以第 1 行"猫"为例：

$$\alpha_{1j} = \frac{e^{s_{1j}}}{\sum_k e^{s_{1k}}}$$

分母 $= e^{0.513} + e^{0.367} + e^{0.437} + e^{0.416} = 1.670 + 1.443 + 1.548 + 1.516 = 6.177$

$$\alpha_{11} = \frac{1.670}{6.177} = 0.270, \quad \alpha_{12} = \frac{1.443}{6.177} = 0.234$$

$$\alpha_{13} = \frac{1.548}{6.177} = 0.251, \quad \alpha_{14} = \frac{1.516}{6.177} = 0.245$$

验证：$0.270 + 0.234 + 0.251 + 0.245 = 1.000$ ✓

对所有行做 Softmax 后，得到注意力权重矩阵：

$$A = \begin{bmatrix} 0.270 & 0.234 & 0.251 & 0.245 \\ 0.278 & 0.246 & 0.269 & 0.262 \\ 0.271 & 0.248 & 0.266 & 0.259 \\ 0.275 & 0.243 & 0.267 & 0.260 \end{bmatrix}$$

**解读**：每一行是 Softmax 后的概率分布，和为 1。"猫"对自己的关注最高（0.270），但差距不大——因为在这个随机初始化的例子中，所有 token 的关注度比较均匀。训练后，这些权重会变得更有区分度。

### 3.6 第六步：加权求和 $\text{Output} = A \cdot V$

$$\text{Output} = A \cdot V = \begin{bmatrix} 0.270 & 0.234 & 0.251 & 0.245 \\ 0.278 & 0.246 & 0.269 & 0.262 \\ 0.271 & 0.248 & 0.266 & 0.259 \\ 0.275 & 0.243 & 0.267 & 0.260 \end{bmatrix} \begin{bmatrix} 0.58 & 0.17 & 0.71 \\ 0.25 & 0.49 & 0.37 \\ 0.59 & 0.23 & 0.32 \\ 0.41 & 0.29 & 0.63 \end{bmatrix}$$

以第 1 行"猫"的输出为例：

$$\text{output}_1 = 0.270 \times \begin{bmatrix} 0.58 \\ 0.17 \\ 0.71 \end{bmatrix} + 0.234 \times \begin{bmatrix} 0.25 \\ 0.49 \\ 0.37 \end{bmatrix} + 0.251 \times \begin{bmatrix} 0.59 \\ 0.23 \\ 0.32 \end{bmatrix} + 0.245 \times \begin{bmatrix} 0.41 \\ 0.29 \\ 0.63 \end{bmatrix}$$

$$= \begin{bmatrix} 0.157 \\ 0.046 \\ 0.192 \end{bmatrix} + \begin{bmatrix} 0.059 \\ 0.115 \\ 0.087 \end{bmatrix} + \begin{bmatrix} 0.148 \\ 0.058 \\ 0.080 \end{bmatrix} + \begin{bmatrix} 0.100 \\ 0.071 \\ 0.154 \end{bmatrix} = \begin{bmatrix} 0.464 \\ 0.290 \\ 0.513 \end{bmatrix}$$

对所有行计算，得到最终输出：

$$\text{Output} = \begin{bmatrix} 0.464 & 0.290 & 0.513 \\ 0.459 & 0.291 & 0.506 \\ 0.461 & 0.290 & 0.509 \\ 0.462 & 0.289 & 0.510 \end{bmatrix} \begin{matrix} \text{猫} \\ \text{坐} \\ \text{在} \\ \text{垫子} \end{matrix}$$

> [!success] 手推完成！
> 从输入嵌入 $X$ 出发，经过 Q/K/V 投影 → 点积 → 缩放 → Softmax → 加权求和，我们得到了每个 token 的新表示。**每个新表示都是所有 token 的 Value 向量的加权和，权重由注意力机制决定。**
>
> 在这个随机初始化的例子中，权重比较均匀，输出看起来很相似。但经过训练后，注意力权重会变得非常有区分度——"猫"可能会高度关注"坐"（因为猫是动作的主语），"坐"可能会高度关注"垫子"（因为垫子是动作的地点）。

---

## 四、为什么需要缩放？$\sqrt{d_k}$ 的数学推导

### 4.1 问题：点积的方差随维度增长

假设 $q$ 和 $k$ 的每个分量独立、均值为 0、方差为 1。那么点积：

$$q \cdot k = \sum_{i=1}^{d_k} q_i \cdot k_i$$

期望 $E[q \cdot k] = 0$，但**方差**为：

$$\text{Var}(q \cdot k) = \sum_{i=1}^{d_k} \text{Var}(q_i \cdot k_i) = d_k$$

因为每一项 $q_i k_i$ 的方差为 1（两个独立标准正态变量的乘积方差为 1），共 $d_k$ 项。

**后果**：当 $d_k = 64$ 时，点积的标准差为 8；当 $d_k = 512$ 时，标准差为 $\approx 22.6$。点积值的范围会随着维度大幅增长。

### 4.2 大方差如何毁掉 Softmax

Softmax 对输入值的**相对大小**极度敏感：

$$\text{softmax}([1, 2, 3]) = [0.09, 0.24, 0.67] \quad \text{← 温和分布}$$

$$\text{softmax}([10, 20, 30]) \approx [0.00, 0.00, 1.00] \quad \text{← 几乎 one-hot}$$

当 Softmax 输出接近 one-hot 时：
- **梯度趋近于 0**——Softmax 在输入值极端时的梯度几乎为零，模型学不到东西
- **注意力退化为只看一个位置**——失去了"加权整合"的核心优势

### 4.3 缩放的数学证明

除以 $\sqrt{d_k}$ 将方差标准化为 1：

$$\text{Var}\left(\frac{q \cdot k}{\sqrt{d_k}}\right) = \frac{d_k}{(\sqrt{d_k})^2} = \frac{d_k}{d_k} = 1$$

> [!success] 一句话总结
> 缩放因子的作用是**控制 Softmax 输入的数值范围**，使方差稳定在 1 附近，防止高维空间中点积过大导致 Softmax 梯度消失。这不是为了防止数值溢出（float32 可以表示很大的数），而是为了**保持梯度健康**。

---

## 五、为什么用点积？

Attention 有多种计算相似度的方式，最常见的有两种：

| 方法 | 公式 | 特点 |
|------|------|------|
| **加性注意力**（Bahdanau） | $e_{ij} = v^T \tanh(W_q q_i + W_k k_j)$ | 表达力强，但需要学习的参数多，计算慢 |
| **点积注意力**（Transformer） | $e_{ij} = q_i \cdot k_j$ | 表达力略弱，但计算极快（矩阵乘法天然并行） |

Transformer 选择点积的原因：**在大规模训练中，计算速度比微弱的表达力差异更重要**。加上缩放因子后，点积注意力在实践中表现与加性注意力相当。

---

## 六、矩阵形式：从标量到批量

所有手推步骤可以用一个简洁的矩阵公式概括：

$$\text{Attention}(Q, K, V) = \text{softmax}\left(\frac{QK^T}{\sqrt{d_k}}\right)V$$

维度追踪：

$$\underbrace{Q}_{n \times d_k} \cdot \underbrace{K^T}_{d_k \times n} = \underbrace{S}_{n \times n} \xrightarrow{\text{scale + softmax}} \underbrace{A}_{n \times n} \cdot \underbrace{V}_{n \times d_v} = \underbrace{\text{Output}}_{n \times d_v}$$

> [!note] 并行性的来源
> $QK^T$ 是一个 $n \times n$ 的矩阵乘法——**不需要逐 token 顺序计算**，整个序列的注意力分数可以一次性通过矩阵乘法得到。这就是 Transformer 并行性的根本来源。

---

## 七、代码视角：PyTorch 实现

> [!example]- 🔧 PyTorch 实现（点击展开）
>
> ```python
> import torch
> import torch.nn.functional as F
> import math
>
> def self_attention(X, W_Q, W_K, W_V):
>     """
>     X: (batch_size, seq_len, d_model)
>     W_Q, W_K, W_V: (d_model, d_k) 或 (d_model, d_v)
>     """
>     # 1. 投影到 Q, K, V
>     Q = X @ W_Q  # (batch, seq_len, d_k)
>     K = X @ W_K  # (batch, seq_len, d_k)
>     V = X @ W_V  # (batch, seq_len, d_v)
>
>     # 2. 计算注意力分数并缩放
>     d_k = Q.size(-1)
>     scores = Q @ K.transpose(-2, -1) / math.sqrt(d_k)  # (batch, seq_len, seq_len)
>
>     # 3. Softmax 归一化
>     attn_weights = F.softmax(scores, dim=-1)  # (batch, seq_len, seq_len)
>
>     # 4. 加权求和
>     output = attn_weights @ V  # (batch, seq_len, d_v)
>
>     return output, attn_weights
>
>
> # 使用示例
> batch_size, seq_len, d_model, d_k = 2, 4, 512, 64
> X = torch.randn(batch_size, seq_len, d_model)
> W_Q = torch.randn(d_model, d_k)
> W_K = torch.randn(d_model, d_k)
> W_V = torch.randn(d_model, d_k)
>
> output, weights = self_attention(X, W_Q, W_K, W_V)
> print(f"Output shape: {output.shape}")     # (2, 4, 64)
> print(f"Weight shape: {weights.shape}")    # (2, 4, 4)
> print(f"Weight sum per row: {weights[0, 0].sum():.4f}")  # ≈ 1.0
> ```
>
> 核心就是 4 行代码：投影 → 点积缩放 → Softmax → 加权求和。

---

## 八、Self-Attention 的计算复杂度

| 操作 | 复杂度 | 说明 |
|------|--------|------|
| $Q, K, V$ 投影 | $O(n \cdot d_{model} \cdot d_k)$ | 三个矩阵乘法 |
| $QK^T$ | $O(n^2 \cdot d_k)$ | 注意力分数矩阵 |
| Softmax | $O(n^2)$ | 逐行归一化 |
| $A \cdot V$ | $O(n^2 \cdot d_v)$ | 加权求和 |
| **总计** | $O(n^2 \cdot d + n \cdot d^2)$ | $d = d_k = d_v$ |

**核心 trade-off**：
- 当 $n < d$（短序列）：$n \cdot d^2$ 项主导，与 RNN 相当
- 当 $n > d$（长序列）：$n^2 \cdot d$ 项主导，复杂度高于 RNN

这也是 Flash Attention 等优化技术存在的意义——它们不改变理论复杂度，但通过**减少显存访问**大幅提升实际速度。

---

## 常见误区

| 误解                              | 正确理解                                                  |
| ------------------------------- | ----------------------------------------------------- |
| "Self-Attention 就是传统 Attention" | Self-Attention 的 Q/K/V 全部来自同一序列，传统 Attention 的 Q 来自外部 |
| "每个词只关注最相似的一个词"                 | 注意力是**加权平均**，会综合所有词的信息（除非 softmax 退化为 one-hot）        |
| "缩放是为了防止数值溢出"                   | 缩放主要是防止 Softmax 的**梯度消失**，而非数值溢出                      |
| "Self-Attention 可以完全取代 RNN"     | Self-Attention 本身是**置换等变**的，不包含位置信息，需要位置编码            |
| "注意力权重就是相似度"                    | 注意力权重是学习到的，不一定反映直觉上的"相似性"                             |

---

## 思考题

> [!hint]- 💡 思考题 1（基础）：Q/K/V 相同的后果
> 如果令 $W_Q = W_K = W_V$（即 Q、K、V 完全相同），Self-Attention 的输出会变成什么样？

> [!success]- ✅ 参考答案
> 如果 $Q = K$，则注意力分数矩阵 $S = QK^T = (XW)(XW)^T = XWW^TX^T$。问题有两个：
> 1. **对称性过强**：$s_{ij} = s_{ji}$，词 $i$ 对词 $j$ 的关注度和词 $j$ 对词 $i$ 的关注度相同——但"猫吃鱼"中，"鱼"应高度关注"吃"，"吃"不一定最关注"鱼"
> 2. **表达力下降**：无法学习"同一个词在不同角色下的不同表示"
> 3. 如果进一步 $W = I$（单位矩阵），则 $S = XX^T$（Gram 矩阵），注意力完全由输入的余弦相似度决定，无法学习

> [!hint]- 💡 思考题 2（基础）：缩放的必要性
> 假设 $d_k = 1$，还需要缩放因子吗？为什么？

> [!success]- ✅ 参考答案
> $d_k = 1$ 时，$\sqrt{d_k} = 1$，缩放因子没有实际效果。因为单个分量的方差本身就是 1，不存在"维度累加导致方差增长"的问题。缩放因子的意义在于抵消高维空间中**维度累加**导致的方差增长。

> [!hint]- 💡 思考题 3（进阶）：置换等变性
> 如果打乱输入序列的顺序，Self-Attention 的输出会怎么变化？这说明了什么？

> [!success]- ✅ 参考答案
> 如果输入变为 $X' = [x_3, x_1, x_2]^T$，则输出 $\text{Output}'$ 会是原始输出的**一个置换**——每个 token 对应的输出向量不变，只是顺序变了。
>
> 这说明 Self-Attention 是**置换等变（permutation equivariant）**的——它本身无法区分位置顺序。这就是为什么 Transformer 需要位置编码来注入顺序信息（下一篇的主题）。

> [!hint]- 💡 思考题 4（进阶）：Self-Attention 的感受野
> 在 RNN 中，一个 token 要"看到"10 步前的 token 需要经过 10 次传递。在单层 Self-Attention 中，一个 token 可以直接看到多远的 token？如果堆叠 $L$ 层 Self-Attention 呢？

> [!success]- ✅ 参考答案
> 单层 Self-Attention 中，每个 token 可以直接看到**所有**其他 token——无论距离多远，感受野是全局的。
>
> 堆叠 $L$ 层时，情况更有趣：第 1 层让每个 token 融合了直接邻居的信息；第 2 层让每个 token 融合了邻居已经融合过的信息（相当于间接看到了 2 跳之外的上下文）。但对于 Self-Attention，每一层本身就是全局感受野，堆叠的目的是**逐层提取更抽象的特征**，而非扩大感受野。这与 CNN 逐层扩大感受野的机制有本质区别。

---

## 关键要点回顾

1. **Self-Attention** 让序列中每个位置都能直接关注所有其他位置，Q/K/V 全部来自同一序列
2. **投影矩阵** $W_Q, W_K, W_V$ 将输入映射到三个不同语义空间，赋予每个词"提问"、"被检索"、"提供内容"三种角色
3. **计算流程**：$Q=XW_Q \to K=XW_K \to V=XW_V \to S=QK^T \to \text{scale} \to \text{softmax} \to AV$
4. **缩放因子** $\sqrt{d_k}$ 将点积方差标准化为 1，防止 Softmax 梯度消失
5. **矩阵形式**使整个计算高度并行化，这是 Transformer 高效训练的关键

---

## 扩展阅读

- 📄 Vaswani et al. (2017) — *"Attention Is All You Need"* 第 3.1 节（Scaled Dot-Product Attention 的原始定义）
- 📝 Jay Alammar — [The Illustrated Transformer](https://jalammar.github.io/illustrated-transformer/)（Self-Attention 最佳可视化教程）
- 📝 Lilian Weng — [Attention? Attention!](https://lilianweng.github.io/posts/2018-06-24-attention/)（Attention 机制综述）
- 📝 Peter Bloem — [Transformers from Scratch](https://peterbloem.nl/blog/transformers)（从零实现 Transformer）
- 🎬 3Blue1Brown — [Attention in transformers, visually explained](https://www.youtube.com/watch?v=eMlx5fFNoYw)（YouTube 视频讲解，强烈推荐）

---

## 相关页面

- [[01-why-transformer|第 1 章：为什么需要 Transformer]]
- [[03-multi-head-attention|第 3 章：Multi-Head Attention]]
- [[04-positional-encoding|第 4 章：位置编码]]

## 下一步学习

➡️ [[03-multi-head-attention|第 3 章：Multi-Head Attention]] — 为什么一个注意力头不够？
