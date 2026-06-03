---
type: lesson
tags:
  - 深度学习
  - Transformer
  - Multi-Head Attention
  - 多头注意力
created: 2026-04-09
updated: 2026-04-09
topic: Multi-Head Attention——多视角并行关注
difficulty: intermediate
prerequisites:
  - "[[02-self-attention|Self-Attention]]"
status: completed
series:
  name: Transformer 深度学习
  part: 3
---

# Multi-Head Attention：多视角并行关注

> Multi-Head Attention 让 Transformer 能够同时从多个角度理解序列——有些头关注语法结构，有些头捕捉语义关系，还有些头追踪指代关系。就像一个团队从不同专业视角审视同一份文档，汇总后得到更全面的理解。

## 学习目标

- [ ] 理解单头 Self-Attention 的局限性
- [ ] 掌握 Multi-Head Attention 的核心思想：将注意力拆分为多个并行头
- [ ] 能徒手推导 Multi-Head Attention 的维度变化
- [ ] 理解"多头"与"多个独立 Self-Attention"的本质区别
- [ ] 掌握 PyTorch 中 `nn.MultiheadAttention` 的使用方法

## 前置知识

| 知识点 | 要求 | Wiki 参考 |
|--------|------|-----------|
| Self-Attention 机制 | 熟练掌握 Q/K/V 的含义和计算流程 | [[02-self-attention]] |
| 矩阵乘法 | 能做矩阵乘法并追踪维度 | — |
| Softmax 函数 | 知道它把向量变成概率分布 | — |
| 分块矩阵的概念 | 知道如何将大矩阵拆分为小矩阵 | — |

---

## 直观理解：一个学生 vs 一个专家组

### 类比一：单头 Attention 像一个学生在阅读

想象一个学生阅读"苹果"这个词：
- 作为 Query，学生可能在问："这是一种水果还是公司？"
- 作为 Key，学生需要判断这个词的特征
- 最后学生得到一个综合理解

**问题**：这个学生只能给出一个答案。但"苹果"可能是水果（ripe apple）、公司（Apple Inc.）、也可能与"牛顿"关联。单一视角容易产生偏见。

### 类比二：Multi-Head Attention 像一个专家组讨论

专家组包含：植物学家、商业分析师、物理老师、历史学家。

当看到"苹果"时：
- **植物学家**问："这是什么品种的水果？生长在哪里？"
- **商业分析师**问："这是哪家公司的产品？市场份额如何？"
- **物理老师**问："苹果落地遵循什么力学原理？"
- **历史学家**问："亚当夏娃的苹果是什么寓意？"

每个专家从自己的专业视角给出关注重点，最后汇总所有人的意见，得到对"苹果"更全面的理解。

> [!success] 核心直觉
> Multi-Head Attention 的本质是：**让多个"专家"同时工作，每个专家关注不同类型的相关性**。有的头捕捉语法关系，有的捕捉语义相似度，有的捕捉指代链。汇总后，模型能同时考虑多种类型的依赖关系。

### 类比三：人类视觉的多尺度处理

人类的视觉系统同时处理多个尺度的信息：
- 有人关注整体轮廓（大尺度）
- 有人关注纹理细节（小尺度）
- 有人关注颜色对比
- 有人关注运动方向

Multi-Head Attention 类似于让模型同时从多个"尺度"理解序列信息。

---

## 一、单头的局限：为什么一个头不够？

### 1.1 单头只能捕捉一种"关注模式"

回顾 Self-Attention 的计算：

$$\text{Attention}(Q, K, V) = \text{softmax}\left(\frac{QK^T}{\sqrt{d_k}}\right)V$$

在这个公式中，**一组 $W_Q, W_K, W_V$ 投影定义了一种类型的"查询-键匹配"**。

举例来说，在句子"The cat sat on the mat because it was tired"中：
- "it" 应该高度关注 "cat"（指代消解）
- 这需要一种特定的 Q/K 匹配模式

但如果模型同时需要捕捉：
- "sat" 和 "on" 的介词关系（语法）
- "tired" 和 "cat" 的语义关联（属性修饰）

**单一投影矩阵无法同时最优地支持所有这些模式**。不同的注意力模式需要不同的 Q/K/V 投影。

### 1.2 直观例子：同义词与多义性

考虑句子"The bank by the river is nice"和"The bank account is empty"。

- 第一句的"bank"指河岸（地理实体）
- 第二句的"bank"指银行（金融机构）

**单头 Attention** 在处理这两个句子时，如果投影矩阵偏向于"地理"语义，第一句学得好但第二句差，反之亦理。**多头 Attention** 可以让一个头专门捕捉"金融机构"相关的模式，另一个头捕捉"地理实体"相关的模式。

### 1.3 数学直觉：低维投影的信息瓶颈

假设 $d_{model} = 512$，而我们只使用一组 $d_k = 64$ 的 Q/K/V 投影。

这意味着：
- 每个 token 的信息被压缩到 64 维空间
- 某些 64 维子空间可能只保留"语法"信息
- 另一些 64 维子空间可能只保留"语义"信息
- **不可能在一个 64 维空间中同时最优地保留所有类型的信息**

通过将 512 维分成 8 个 64 维的头，每个头可以专门处理一种类型的信息，汇总后恢复完整的 512 维表示。

---

## 二、多头机制：拆解与重组

### 2.1 核心思想

Multi-Head Attention 的公式：

$$\text{MultiHead}(Q, K, V) = \text{Concat}(head_1, head_2, \ldots, head_h) \cdot W^O$$

其中：

$$head_i = \text{Attention}(QW_i^Q, KW_i^K, VW_i^V)$$

这个公式在说：
1. **拆分**：将 512 维拆成 8 份，每份 64 维
2. **独立计算**：每份独立计算一个 Attention（"头"）
3. **拼接**：将 8 个头的输出拼接成 512 维
4. **线性变换**：通过 $W^O$ 做最终投影

### 2.2 直观理解：分组处理

想象一个 12 人的篮球队：
- **单头 Attention**：5 人上场，必须同时兼顾进攻、防守、组织——结果每样都不精
- **多头 Attention**：分成 3 组：进攻组、防守组、组织组，每组 4 人专注各自任务，最后汇总成完整战术

### 2.3 与"8 个独立 Attention"的区别

> [!important] 关键区别
> 如果用 8 个完全独立的 Self-Attention（各自有完整的 $W_Q, W_K, W_V, W^O$），参数量的确相似，但有一个关键区别：**输出维度不同**。
>
> - 8 个独立 Attention：每个输出 64 维，拼接后 512 维，各自为战，没有信息交互
> - Multi-Head Attention：在拼接后通过 $W^O$ 做**跨头的信息混合**，让不同头学到的信息可以相互影响
>
> 更重要的是，Multi-Head Attention 是**在同一个序列上做多次不同视角的注意力**，而不是 8 次独立的不相干的操作。

---

## 三、公式拆解：每一步在做什么

### 3.1 输入投影

$$Q_i = Q W_i^Q, \quad K_i = K W_i^K, \quad V_i = V W_i^V$$

**这句话在说什么**：对于第 $i$ 个头，我们有一组独立的投影矩阵 $W_i^Q, W_i^K, W_i^V$。每个头用自己的投影矩阵将输入映射到自己的子空间。

维度追踪：
- $Q \in \mathbb{R}^{n \times d_{model}}$
- $W_i^Q \in \mathbb{R}^{d_{model} \times d_k}$
- $Q_i \in \mathbb{R}^{n \times d_k}$

其中 $d_k = d_{model} / h$。在标准 Transformer 中，$d_{model} = 512, h = 8, d_k = 64$。

### 3.2 计算单个头的 Attention

$$head_i = \text{softmax}\left(\frac{Q_i K_i^T}{\sqrt{d_k}}\right) V_i$$

**这句话在说什么**：在第 $i$ 个头的子空间内，标准 Self-Attention 流程——点积、缩放、Softmax、加权求和。

### 3.3 拼接多个头

$$\text{Concat}(head_1, head_2, \ldots, head_h) \in \mathbb{R}^{n \times (h \cdot d_k)}$$

**这句话在说什么**：将 $h$ 个头的输出在**最后一个维度**上拼接。由于每个头输出 $d_k$ 维，$h$ 个头拼接后得到 $h \cdot d_k = d_{model}$ 维。

### 3.4 最终输出投影

$$\text{MultiHead}(Q, K, V) = \text{Concat}(head_1, \ldots, head_h) W^O$$

其中 $W^O \in \mathbb{R}^{(h \cdot d_k) \times d_{model}}$。

**这句话在说什么**：拼接后的 $h \cdot d_k$ 维向量通过一个线性变换投影回 $d_{model}$ 维。这个 $W^O$ 矩阵允许模型学习如何**跨头组合信息**——比如把"语法头"的输出和"语义头"的输出做加权融合。

---

## 四、维度追踪：512 如何变成 8×64 再变回 512

### 4.1 完整维度流程图

```
输入 X: (n, d_model=512)
       │
       ├─→ W_Q^1 ... W_Q^8 ──────────────────┐
       │                                      │
       │                                      ▼
       │                              Q_1...(n, 64)  每个头独立计算 Q
       │                                      │
       ├─→ W_K^1 ... W_K^8 ──────────────────┼──→ K_1...(n, 64)
       │                                      │
       │                                      │
       ├─→ W_V^1 ... W_V^8 ──────────────────┼──→ V_1...(n, 64)
       │                                      │
       ▼                                      ▼
    Attention 计算 ──────────────────→ head_1...(n, 64) 每个头独立输出
       │                                      │
       └──────────────────────────────────────┤
                                            ▼
                                  Concat ──→ (n, 8×64) = (n, 512)
                                            │
                                            ▼
                                       W^O ──→ (n, d_model=512)
```

### 4.2 逐层维度表

| 操作 | 输入维度 | 输出维度 |
|------|---------|---------|
| $QW_i^Q$ | $(n, 512) \times (512, 64)$ | $(n, 64)$ |
| 单头 Attention | $(n, 64) \times (n, 64) \times (n, 64)$ | $(n, 64)$ |
| Concat 8 个头 | $(n, 64) \times 8$ | $(n, 512)$ |
| $W^O$ 投影 | $(n, 512) \times (512, 512)$ | $(n, 512)$ |

### 4.3 为什么保持总维度不变？

每个头的维度 $d_k = d_{model} / h$：
- $h = 8, d_{model} = 512 \Rightarrow d_k = 64$
- $h = 4, d_{model} = 512 \Rightarrow d_k = 128$
- $h = 16, d_{model} = 512 \Rightarrow d_k = 32$

**设计哲学**：Multi-Head Attention **不改变总计算量**（相对于单头 512 维），而是将计算分散到多个低维子空间中并行处理。这样既有多样性（多视角），又有效率（总维度不变）。

---

## 五、数值示例：2 个头的完整计算

> [!important] 简化维度示例
> 为便于手算，这里使用 $d_{model}=4, h=2, d_k=2$ 的极简配置。原理与标准 512/8/64 完全一致。

### 5.1 场景设定

输入句子："猫 坐"（2 个 token），$d_{model}=4$，$h=2$，$d_k=2$。

假设输入嵌入矩阵：

$$X = \begin{bmatrix} 1.0 & 0.0 & 0.5 & 0.3 \\ 0.2 & 0.8 & 0.1 & 0.5 \end{bmatrix} \begin{matrix} \text{猫} \\ \text{坐} \end{matrix}$$

两个头的投影矩阵：

$$W_1^Q = W_1^K = W_1^V = \begin{bmatrix} 1 & 0 \\ 0 & 1 \end{bmatrix} \quad \text{（单位矩阵，头1保留前2维）}$$

$$W_2^Q = W_2^K = W_2^V = \begin{bmatrix} 0 & 1 \\ 1 & 0 \end{bmatrix} \quad \text{（交换矩阵，头2保留后2维）}$$

### 5.2 第一步：两个头的 Q/K/V 投影

**头1（Q₁, K₁, V₁）**：

$$Q_1 = X W_1^Q = \begin{bmatrix} 1.0 & 0.0 \\ 0.2 & 0.8 \end{bmatrix}, \quad K_1 = X W_1^K = \begin{bmatrix} 1.0 & 0.0 \\ 0.2 & 0.8 \end{bmatrix}, \quad V_1 = X W_1^V = \begin{bmatrix} 1.0 & 0.0 \\ 0.2 & 0.8 \end{bmatrix}$$

**头2（Q₂, K₂, V₂）**：

$$Q_2 = X W_2^Q = \begin{bmatrix} 0.0 & 1.5 \\ 0.8 & 0.1 \end{bmatrix}, \quad K_2 = X W_2^K = \begin{bmatrix} 0.0 & 1.5 \\ 0.8 & 0.1 \end{bmatrix}, \quad V_2 = X W_2^V = \begin{bmatrix} 0.0 & 1.5 \\ 0.8 & 0.1 \end{bmatrix}$$

### 5.3 第二步：各自计算 Attention

**头1的Attention**（$d_k = 2, \sqrt{d_k} \approx 1.414$）：

$$S_1 = Q_1 K_1^T = \begin{bmatrix} 1.0 & 0.0 \\ 0.2 & 0.8 \end{bmatrix} \begin{bmatrix} 1.0 & 0.2 \\ 0.0 & 0.8 \end{bmatrix} = \begin{bmatrix} 1.00 & 0.20 \\ 0.20 & 0.68 \end{bmatrix}$$

$$S_{1,scaled} = \frac{S_1}{1.414} = \begin{bmatrix} 0.707 & 0.141 \\ 0.141 & 0.481 \end{bmatrix}$$

对每行 Softmax：

$$\text{head}_1 = \text{softmax}(S_{1,scaled}) V_1 = \begin{bmatrix} 0.568 & 0.432 \\ 0.432 & 0.568 \end{bmatrix} \begin{bmatrix} 1.0 & 0.0 \\ 0.2 & 0.8 \end{bmatrix} = \begin{bmatrix} 0.586 & 0.346 \\ 0.546 & 0.454 \end{bmatrix}$$

**头2的Attention**：

$$S_2 = Q_2 K_2^T = \begin{bmatrix} 0.0 & 1.5 \\ 0.8 & 0.1 \end{bmatrix} \begin{bmatrix} 0.0 & 0.8 \\ 1.5 & 0.1 \end{bmatrix} = \begin{bmatrix} 2.25 & 0.15 \\ 0.15 & 0.65 \end{bmatrix}$$

$$S_{2,scaled} = \frac{S_2}{1.414} = \begin{bmatrix} 1.591 & 0.106 \\ 0.106 & 0.460 \end{bmatrix}$$

对每行 Softmax：

$$\text{head}_2 = \text{softmax}(S_{2,scaled}) V_2 = \begin{bmatrix} 0.823 & 0.177 \\ 0.397 & 0.603 \end{bmatrix} \begin{bmatrix} 0.0 & 1.5 \\ 0.8 & 0.1 \end{bmatrix} = \begin{bmatrix} 0.142 & 1.269 \\ 0.482 & 0.969 \end{bmatrix}$$

### 5.4 第三步：拼接并投影

$$\text{Concat} = [\text{head}_1 | \text{head}_2] = \begin{bmatrix} 0.586 & 0.346 & 0.142 & 1.269 \\ 0.546 & 0.454 & 0.482 & 0.969 \end{bmatrix}$$

假设 $W^O = I_{4 \times 4}$（单位矩阵），则最终输出即拼接结果：

$$\text{MultiHead}(Q, K, V) = \begin{bmatrix} 0.586 & 0.346 & 0.142 & 1.269 \\ 0.546 & 0.454 & 0.482 & 0.969 \end{bmatrix}$$

> [!success] 计算完成
> 观察输出：每个 token 现在是 4 维向量，其中**前 2 维来自头1（原始空间）**，**后 2 维来自头2（交换空间）**。不同的头捕捉了不同类型的依赖关系。训练过程中，模型会学习让不同的头关注不同类型的信息。

---

## 六、为什么拆成多个低维头而不是用一个完整的高维头？

### 6.1 信息多样性的来源

假设用一组 $d_{model} = 512$ 的 Q/K/V（而不是 8 组 64 维）：

- 模型仍然可以学到多种注意力模式
- 但这些模式都在**同一个 512 维空间**中相互竞争

Multi-Head Attention 的优势在于：**将问题分解到多个独立的低维空间**，每个空间专门处理一种模式，减少干扰。

### 6.2 计算效率的考量

虽然 Multi-Head Attention 的参数总量与单头 512 维相近，但：

| 方案 | 单头 Q/K/V 投影 | 多头 Q/K/V 投影 |
|------|----------------|-----------------|
| 矩阵维度 | $3 \times (512 \times 512)$ | $8 \times 3 \times (64 \times 64)$ |
| 计算量 | $O(n \cdot d_{model}^2)$ | $O(n \cdot h \cdot d_k^2) = O(n \cdot d_{model} \cdot d_k)$ |
| 实际效率 | 较低（需要大矩阵乘法） | 较高（多个小矩阵可并行） |

### 6.3 论文原文的解释

Vaswani 等人在原论文中这样描述：

> "Multi-head attention allows the model to jointly attend to information from different representation subspaces at different positions."

翻译：多头注意力允许模型在**不同的表示子空间**中**同时关注**来自不同位置的信息。

---

## 七、代码视角：PyTorch 实现

> [!example]- 🔧 PyTorch nn.MultiheadAttention（点击展开）
>
> ```python
> import torch
> import torch.nn as nn
>
> # 标准配置：d_model=512, h=8, d_k=64
> d_model = 512
> num_heads = 8
> d_k = d_model // num_heads  # 64
>
> # 创建 Multi-Head Attention 层
> mha = nn.MultiheadAttention(
>     embed_dim=d_model,    # 输入维度
>     num_heads=num_heads,   # 头的数量
>     dropout=0.1,           # 可选：dropout 比率
>     bias=True,            # 可选：是否使用偏置
>     add_bias_kv=False,    # 可选：是否给 K/V 加偏置
> )
>
> # 输入格式: (seq_len, batch, d_model)
> seq_len, batch_size = 10, 2
> x = torch.randn(seq_len, batch_size, d_model)
>
> # 使用示例
> output, attn_weights = mha(x, x, x)
> # output: (seq_len, batch, d_model)
> # attn_weights: (batch, num_heads, seq_len, seq_len)
>
> print(f"Output shape: {output.shape}")          # (10, 2, 512)
> print(f"Attention shape: {attn_weights.shape}") # (2, 8, 10, 10)
> ```
>
> **关键点**：
> 1. PyTorch 的 `nn.MultiheadAttention` 默认 `kdim=vdim=d_model`（即 key 和 query 维度相同）
> 2. `attn_weights` 的形状是 `(batch, num_heads, seq_len, seq_len)`，可以可视化每个头关注什么
> 3. 可以通过 `output_weights` 或检查 `attn_weights` 分析不同头的学到的注意力模式

> [!example]- 🔧 从底层看 Multi-Head Attention 的实现
>
> ```python
> import torch
> import torch.nn as nn
> import torch.nn.functional as F
> import math
>
> class MultiHeadAttention(nn.Module):
>     def __init__(self, d_model, num_heads, d_k=None):
>         super().__init__()
>         self.num_heads = num_heads
>         self.d_k = d_k if d_k else d_model // num_heads
>
>         # 投影矩阵
>         self.W_Q = nn.Linear(d_model, self.d_k * num_heads, bias=False)
>         self.W_K = nn.Linear(d_model, self.d_k * num_heads, bias=False)
>         self.W_V = nn.Linear(d_model, self.d_k * num_heads, bias=False)
>         self.W_O = nn.Linear(self.d_k * num_heads, d_model, bias=False)
>
>     def split_heads(self, x):
>         """将 batch 维度的 hidden 拆成多个头"""
>         batch_size, seq_len, _ = x.shape
>         x = x.view(batch_size, seq_len, self.num_heads, self.d_k)
>         return x.permute(0, 2, 1, 3)  # (batch, heads, seq_len, d_k)
>
>     def forward(self, Q, K, V):
>         batch_size = Q.size(0)
>
>         # 1. 线性投影
>         Q = self.W_Q(Q)
>         K = self.W_K(K)
>         V = self.W_V(V)
>
>         # 2. 拆分多头
>         Q = self.split_heads(Q)  # (batch, heads, seq_len, d_k)
>         K = self.split_heads(K)
>         V = self.split_heads(V)
>
>         # 3. 计算注意力
>         d_k = Q.size(-1)
>         scores = torch.matmul(Q, K.transpose(-2, -1)) / math.sqrt(d_k)
>         attn_weights = F.softmax(scores, dim=-1)
>         context = torch.matmul(attn_weights, V)
>
>         # 4. 合并多头并输出投影
>         context = context.permute(0, 2, 1, 3).contiguous()
>         context = context.view(batch_size, -1, self.num_heads * self.d_k)
>         output = self.W_O(context)
>
>         return output, attn_weights
> ```

---

## 常见误区

| 误解                         | 正确理解                                                                                |
| -------------------------- | ----------------------------------------------------------------------------------- |
| "头数越多越好"                   | 头数过多会导致每个头的维度太小（$d_k = d_{model}/h$），学不到足够的语义信息。标准 Transformer 用 8 个头是经验值           |
| "多头就是多个独立的 Self-Attention" | Multi-Head Attention 在 Concat 后还有 $W^O$ 投影，实现跨头信息融合；且共享总维度，单头高维空间中学不到的多样性被分散到多个低维空间 |
| "每个头学到的内容是确定的"             | 不同头的功能（如语法、语义）是**训练出来的**，不是预先指定的。某些头可能学到相似的模式（冗余），某些头可能几乎不激活                        |
| "头1关注语法，头2关注语义"            | 这种划分是可能的，但不是必然的。不同头学到的模式取决于任务和数据，可能出现跨类型的混合模式                                       |
| "多头的输出是多个头的平均值"            | 多头输出是**拼接**（Concat）而非平均。拼接保留了每个头的独立信息，最后通过 $W^O$ 学习最优组合                             |

---

## 思考题

> [!hint]- 💡 思考题 1（基础）：头数的极端情况
> 如果 $h = 1$（只有一个头），Multi-Head Attention 会退化成什么？如果 $h = d_{model}$（每个头只有 1 维）呢？

> [!success]- ✅ 参考答案
> - **$h = 1$**：退化为标准的单头 Self-Attention（因为拼接后维度仍为 $d_{model}$，$W^O$ 只是额外线性变换）
> - **$h = d_{model}$**：每个头的维度 $d_k = 1$，此时每个头做的是标量点积注意力，拼接后维度仍是 $d_{model}$。但维度太低可能严重限制表达能力

> [!hint]- 💡 思考题 2（基础）：多头的参数效率
> 假设 $d_{model} = 512, h = 8, d_k = 64$，计算 Multi-Head Attention 的参数量，并与单头 $d_k = 512$ 的 Self-Attention 比较。

> [!success]- ✅ 参考答案
> - **Multi-Head Attention**：
>   - $W_Q^h, W_K^h, W_V^h$：每个 $8 \times (512 \times 64) = 8 \times 32768$
>   - $W^O$：$8 \times 64 \times 512 = 262144$
>   - 总计：$3 \times 262144 + 262144 = 1048576$（约 1M 参数）
> - **单头 Self-Attention**（$d_k = 512$）：
>   - $W_Q, W_K, W_V$：各 $512 \times 512 = 262144$
>   - 总计：$3 \times 262144 = 786432$（约 0.8M 参数）
> - 多头 Attention 参数略多（约 1M vs 0.8M），但**计算量相近**，主要是并行度不同

> [!hint]- 💡 思考题 3（进阶）：可视化与分析
> 假设你训练好了一个 Transformer 模型，如何分析不同头学到了什么类型的注意力模式？提出你的分析方案。

> [!success]- ✅ 参考答案
> 1. **提取注意力权重**：运行模型，收集 `attn_weights`，形状为 `(batch, num_heads, seq_len, seq_len)`
>
> 2. **头间相关性分析**：计算不同头之间的注意力权重相关性（余弦相似度），识别冗余头
>
> 3. **特定位置分析**：
>    - 对于指代消解任务（如"it"指向"cat"），检查哪些头的注意力权重在该位置有正确的指代映射
>    - 对于语法任务，检查是否有的头对介词关系（如"sat on"）有高注意力权重
>
> 4. **头的功能聚类**：对所有头的注意力模式做聚类，识别功能相似的头组
>
> 5. **头的消融实验**：逐头遮蔽（mask），观察对下游任务的影响，量化每个头的贡献

---

## 关键要点回顾

1. **单头局限**：单一 Q/K/V 投影只能在同一个语义空间中捕捉一种注意力模式，无法同时最优处理语法、语义、指代等多种关系
2. **多头思想**：将 $d_{model}$ 拆分为 $h$ 个 $d_k = d_{model}/h$ 维子空间，每个子空间独立计算 Attention
3. **公式结构**：$head_i = \text{Attention}(QW_i^Q, KW_i^K, VW_i^V)$ → $\text{Concat}(head_1, \ldots, head_h) W^O$
4. **信息汇总**：拼接保持了每个头的独立信息，$W^O$ 投影学习跨头信息融合
5. **设计哲学**：多头不增加总计算量（相对于单头 512 维），而是将计算分散到多个低维子空间，提高表达多样性
6. **与独立 Attention 的区别**：不是 8 个独立不相关的操作，而是通过拼接后的 $W^O$ 实现信息交互

---

## 扩展阅读

- 📄 Vaswani et al. (2017) — *"Attention Is All You Need"* 第 3.2 节（Multi-Head Attention 的原始定义）
- 📝 Jay Alammar — [The Illustrated Transformer](https://jalammar.github.io/illustrated-transformer/)（多头注意力的最佳可视化教程）
- 📝 Lilian Weng — [Multi-Head Attention](https://lilianweng.github.io/posts/2018-06-24-attention/)（多头注意力的详细解释）
- 📝 Sofia Neote — [What Does BERT's Multi-Head Attention Learn?](https://pair.withgoogle.com/)（分析 BERT 中不同头学到的模式）
- 🎬 StatQuest with Josh Starmer — [Multi-Head Attention](https://www.youtube.com/watch?v=23C07Zt8C88)（YouTube 视频讲解）

---

## 相关页面

- [[02-self-attention|第 2 章：Self-Attention]]
- [[04-positional-encoding|第 4 章：位置编码]]
- [[05-ffn-residual-layernorm|第 5 章：FFN、残差连接与 Layer Normalization]]
- [[06-encoder-decoder|第 6 章：编码器-解码器架构]]

## 下一步学习

➡️ [[04-positional-encoding|第 4 章：位置编码]] — Transformer 没有顺序感？位置编码来帮忙
