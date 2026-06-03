---
type: lesson
tags:
  - 深度学习
  - Transformer
  - 位置编码
  - Positional Encoding
  - 正弦编码
created: 2026-04-09
updated: 2026-04-09
topic: 位置编码——让 Attention 知道"在哪里"
difficulty: intermediate
prerequisites:
  - "[[02-self-attention|Self-Attention]]"
status: completed
series:
  name: Transformer 深度学习
  part: 4
---

# 位置编码：让 Attention 知道"在哪里"

> 一句话摘要：位置编码通过向词嵌入注入基于数学函数的序列位置信号，解决了 Self-Attention 置换等变性问题，使 Transformer 能够区分"谁对谁做了什么"。

## 学习目标

- [ ] 理解 Self-Attention 置换等变性的本质含义及其与位置编码的关系
- [ ] 掌握正弦位置编码的公式、推导过程和直觉理解
- [ ] 能够手算任意位置的正弦位置编码向量
- [ ] 理解正弦编码中不同频率的物理意义
- [ ] 对比可学习位置编码与正弦编码的优劣
- [ ] 了解 RoPE、ALiBi 等现代位置编码技术的基本思想

---

## 前置知识

| 知识点 | 要求 | 参考 |
|--------|------|------|
| Self-Attention 计算流程 | 熟练掌握 | [[02-self-attention]] |
| 矩阵乘法 | 能做 | — |
| $\sin / \cos$ 三角函数基础 | 知道图像形状 | — |
| 指数函数 $e^x$ 和对数 | 知道基本性质 | — |

---

## 直观理解：两堂课的类比

### 类比一：一堆无编号的试卷

想象老师抱来一叠试卷，每张试卷的内容完全相同——同样的选择题、同样的作文题目。但老师没有在试卷上写姓名。

小明做完试卷后，小红把她的答案抄到了小明的试卷上。如果没有任何身份标记，你分得清哪张是小明的、哪张是小红的吗？

**Self-Attention 就是这样——它处理一组向量，但如果不告诉它"这是第 3 个词"、"这是第 7 个词"，它就无法区分顺序。**

### 类比二：盲人摸象团队

5 个盲人组成团队摸一头象。每个人负责摸象的一个部位（鼻子、耳朵、腿等）。他们可以交流各自摸到的信息，但没有人知道"象鼻在象的左边还是右边"。

如果把象的鼻子砍下来放在房间里，盲人知道"这是鼻子"，但不知道鼻子和耳朵谁在左谁在右。

**词嵌入就是"这是什么"的信息，位置编码就是"在哪里"的信息。没有位置信息，模型只知道词的性质，不知道词之间的关系取决于顺序。**

---

## 一、问题的核心：Self-Attention 的置换等变性

### 1.1 形式化定义

在上一章的思考题中，我们已经接触到置换等变性。严格地说：

设 $\sigma$ 是 $\{1, 2, ..., n\}$ 的一个排列（permutation），$X \in \mathbb{R}^{n \times d}$ 是输入序列的嵌入矩阵（$X_i$ 是第 $i$ 个 token 的嵌入向量）。

将 $X$ 按排列 $\sigma$ 重排得到 $X_\sigma$（即 $X_{\sigma,i} = X_i$）：
$$X_\sigma = \begin{bmatrix} x_{\sigma(1)} \\ x_{\sigma(2)} \\ \vdots \\ x_{\sigma(n)} \end{bmatrix}$$

如果一个函数 $f$ 满足：
$$f(X_\sigma) = (f(X))_\sigma$$

则 $f$ 是**置换等变**的——输出只是输入的置换，不包含输入顺序的绝对信息。

### 1.2 为什么 Self-Attention 是置换等变的？

回顾 Self-Attention 的计算：
$$\text{Attention}(Q, K, V) = \text{softmax}\left(\frac{QK^T}{\sqrt{d_k}}\right)V$$

其中 $Q = XW_Q$，$K = XW_K$，$V = XW_V$。

设输入被重排为 $X_\sigma$，则对应的 Q/K/V 也被重排为 $Q_\sigma, K_\sigma, V_\sigma$。

注意力分数矩阵：
$$Q_\sigma K_\sigma^T = (X_\sigma W_Q)(X_\sigma W_K)^T = X_\sigma W_Q W_K^T X_\sigma^T$$

注意到这等于对原始 $QK^T$ 按 $\sigma$ 进行行列重排。Softmax 对每行独立操作，所以：
$$\text{softmax}(Q_\sigma K_\sigma^T) = (\text{softmax}(QK^T))_\sigma$$

最终输出：
$$(\text{softmax}(QK^T))_\sigma \cdot V_\sigma = (A \cdot V)_\sigma$$

即输出也是原始输出的置换。**没有任何绝对位置信息被注入。**

### 1.3 置换等变性的实际影响

考虑两个句子：

**句子 A**："狗 咬 人"（3 个 token）
**句子 B**："人 咬 狗"（3 个 token）

假设词嵌入使得 `狗 = [1, 0]`，`咬 = [0, 1]`，`人 = [0.5, 0.5]`。

对句子 A 计算 Self-Attention（简化假设，不涉及实际训练权重）。如果输入变为句子 B（排列置换），输出只是置换版本——模型学到的是"狗和人在咬"，但分不清"谁咬谁"。

**结论**：没有位置信息，"狗咬人"和"人咬狗"在 Self-Attention看来是完全等价的。这对于语义理解是致命的。

> [!success] 一句话总结
> Self-Attention 是置换等变的——打乱输入顺序，输出只是重排。这说明 Self-Attention 本身不知道"谁在第几个位置"，必须靠额外的机制注入位置信息。

---

## 二、为什么用正弦函数？直觉与形式化

### 2.1 最简单的方案：独热位置编码

最直觉的做法是给每个位置一个唯一的向量：

$$PE_1 = [1, 0, 0, ..., 0], \quad PE_2 = [0, 1, 0, ..., 0], \quad ..., \quad PE_n = [0, 0, 0, ..., 1]$$

**问题**：
- 对于 maxlen = 2048 的模型，需要 2048 维的位置向量，维度爆炸
- 无法泛化到训练时没见过的位置长度
- 完全没有"相对位置"的概念——第 1 位和第 2 位的关系、第 3 位和第 4 位的关系，用独热编码表示毫无联系

### 2.2 正弦位置编码的直觉

正弦函数具有一个关键特性：**周期性**和**不同频率**。

想象一个乐队演奏：
- 低音鼓（低频）：节奏缓慢，传播远，能让人知道"这首歌已经进行了很久"
- 高音哨（高频）：声音尖锐，变化快，能让人知道"现在正处于副歌部分的某个节拍"

正弦位置编码就是用不同频率的正弦波来编码位置：
- **低频分量**编码"大尺度"位置——你在序列的前部、中部还是尾部
- **高频分量**编码"精细"位置——你具体在哪个 token

### 2.3 形式化定义

原始 Transformer 论文（Vaswani et al., 2017）使用的正弦位置编码：

$$PE_{(pos, 2i)} = \sin\left(\frac{pos}{10000^{2i/d_{model}}}\right)$$

$$PE_{(pos, 2i+1)} = \cos\left(\frac{pos}{10000^{2i/d_{model}}}\right)$$

其中：
- $pos$ 是位置（0, 1, 2, 3, ...）
- $i$ 是维度索引（0, 1, 2, ..., $d_{model}/2 - 1$）
- $d_{model}$ 是嵌入维度

**偶数维度用 sin，奇数维度用 cos。**

---

## 三、公式详解：逐步拆解与数值示例

### 3.1 频率的几何级数设计

公式中的分母 $10000^{2i/d_{model}}$ 看起来不直观。设：

$$\omega_i = \frac{1}{10000^{2i/d_{model}}}$$

当 $i = 0$ 时：
$$\omega_0 = \frac{1}{10000^0} = 1$$

当 $i = 1$ 时：
$$\omega_1 = \frac{1}{10000^{2/d_{model}}}$$

假设 $d_{model} = 512$：
$$\omega_1 = \frac{1}{10000^{1/256}} \approx \frac{1}{1.018} \approx 0.982$$

当 $i$ 增大时，$\omega_i$ 递减，形成一个从 1 到 $1/10000$ 的几何级数。

> [!success] 频率的几何级数意义
> 几何级数的设计确保了**每个频率之间等距**。从 $\omega_0 = 1$ 到 $\omega_{d_{model}/2-1} \approx 1/10000$，频率在对数尺度上均匀分布。这意味着编码位置信息的"分辨率"在对数尺度上是均匀的——能同时捕捉近处和远处的关系。

### 3.2 位置编码向量结构

对于位置 $pos$ 的编码是长度为 $d_{model}$ 的向量：

$$PE_{pos} = [\sin(\omega_0 \cdot pos), \cos(\omega_0 \cdot pos), \sin(\omega_1 \cdot pos), \cos(\omega_1 \cdot pos), ..., \sin(\omega_{d_{model}/2-1} \cdot pos), \cos(\omega_{d_{model}/2-1} \cdot pos)]$$

**每两个相邻维度是一对**（sin, cos），它们使用相同的频率 $\omega_i$。

### 3.3 数值示例：手算前几个位置

假设 $d_{model} = 8$（简化示例，实际通常是 512）。$i = 0, 1, 2, 3$（4 个频率对）。

**位置 0**：

$$\omega_0 = 1, \quad \omega_1 = \frac{1}{10000^{1/4}} = \frac{1}{10} = 0.1, \quad \omega_2 = \frac{1}{10000^{2/4}} = \frac{1}{100} = 0.01, \quad \omega_3 = \frac{1}{10000^{3/4}} = \frac{1}{1000} = 0.001$$

$$PE_0 = [\sin(0), \cos(0), \sin(0), \cos(0), \sin(0), \cos(0), \sin(0), \cos(0)] = [0, 1, 0, 1, 0, 1, 0, 1]$$

**位置 1**：

$$PE_1 = [\sin(1), \cos(1), \sin(0.1), \cos(0.1), \sin(0.01), \cos(0.01), \sin(0.001), \cos(0.001)]$$

计算（保留 4 位小数）：
- $\sin(1) \approx 0.8415$
- $\cos(1) \approx 0.5403$
- $\sin(0.1) \approx 0.0998$
- $\cos(0.1) \approx 0.9950$
- $\sin(0.01) \approx 0.0100$
- $\cos(0.01) \approx 0.9999$
- $\sin(0.001) \approx 0.0010$
- $\cos(0.001) \approx 1.0000$

$$PE_1 \approx [0.8415, 0.5403, 0.0998, 0.9950, 0.0100, 0.9999, 0.0010, 1.0000]$$

**位置 2**：

$$PE_2 = [\sin(2), \cos(2), \sin(0.2), \cos(0.2), \sin(0.02), \cos(0.02), \sin(0.002), \cos(0.002)]$$

- $\sin(2) \approx 0.9093$
- $\cos(2) \approx -0.4161$
- $\sin(0.2) \approx 0.1987$
- $\cos(0.2) \approx 0.9801$
- $\sin(0.02) \approx 0.0200$
- $\cos(0.02) \approx 0.9998$
- $\sin(0.002) \approx 0.0020$
- $\cos(0.002) \approx 0.9999$

$$PE_2 \approx [0.9093, -0.4161, 0.1987, 0.9801, 0.0200, 0.9998, 0.0020, 0.9999]$$

**观察**：
- 维度 0-1（高频 $\omega_0 = 1$）：值变化剧烈，能区分近距离位置
- 维度 2-3（中频 $\omega_1 = 0.1$）：值变化较缓
- 维度 4-5（低频 $\omega_2 = 0.01$）：值变化很慢
- 维度 6-7（超低频 $\omega_3 = 0.001$）：值几乎不变，能捕捉远距离位置

### 3.4 不同频率分量的物理意义

| 频率对 | $\omega_i$ | 周期 | 能捕捉的关系 |
|--------|-----------|------|-------------|
| 0 | 1.0000 | $2\pi \approx 6.28$ | 相邻 token |
| 1 | 0.1000 | $20\pi \approx 62.8$ | 10 个 token 内 |
| 2 | 0.0100 | $200\pi \approx 628$ | 100 个 token 内 |
| 3 | 0.0010 | $2000\pi \approx 6280$ | 1000 个 token 内 |

**低频分量（高维度索引）的周期很长**——即使位置变化 1000，低频分量才变化一周。这意味着低频分量能捕捉"大尺度"的位置关系（序列的前部 vs 尾部）。

**高频分量（低维度索引）的周期很短**——位置变化 1 就能看到明显差异，能精确定位近距离关系。

---

## 四、位置编码的线性性质：泛化的关键

### 4.1 核心性质

正弦位置编码有一个非常重要的性质：**位置编码可以通过线性变换转换为相对位置**。

具体来说，存在一个线性变换 $M$ 使得：
$$PE_{pos+k} \approx M^k \cdot PE_{pos}$$

这意味着**知道"位置 k"的编码，可以近似推算出"位置 pos+k"的编码**。

### 4.2 数学推导

令 $\omega = 2\pi / T$（T 为周期），考虑复数形式：
$$e^{i\omega \cdot pos} = \cos(\omega \cdot pos) + i\sin(\omega \cdot pos)$$

则：
$$e^{i\omega \cdot (pos+k)} = e^{i\omega \cdot pos} \cdot e^{i\omega \cdot k}$$

即**相对位置 k 的编码**是**绝对位置 pos 的编码**与一个**只依赖 k 的旋转矩阵**的乘积。

在实数形式下，对于一对 (sin, cos)：
$$\begin{bmatrix} \cos(\omega(pos+k)) \\ \sin(\omega(pos+k)) \end{bmatrix} = \begin{bmatrix} \cos\omega_k & -\sin\omega_k \\ \sin\omega_k & \cos\omega_k \end{bmatrix} \begin{bmatrix} \cos(\omega pos) \\ \sin(\omega pos) \end{bmatrix}$$

旋转矩阵 $R_k = \begin{bmatrix} \cos\omega_k & -\sin\omega_k \\ \sin\omega_k & \cos\omega_k \end{bmatrix}$ 只依赖相对距离 $k$，不依赖绝对位置 $pos$。

### 4.3 泛化能力的来源

这个线性性质给了正弦编码**外推能力**：

假设模型在训练时见过 position 0 到 position 1000。在推理时，模型需要处理一个测试句子有 1500 个 token。

对于位置 1200：
- 高频分量（如 $\omega_0 = 1$）：这些分量已经见过类似的值（因为周期短），能泛化
- 低频分量（如 $\omega_3 = 0.001$）：对于 pos=1200，$\omega_3 \cdot pos = 1.2$，对应的 sin/cos 值在训练时已经见过（因为训练时 pos=0 到 1000 已经覆盖了很多低频周期）

**正弦编码不依赖"查表"，而是用数学函数计算**，所以对任意位置都能生成编码。

> [!success] 为什么正弦编码比可学习编码泛化更好？
> 可学习编码在训练时为每个位置学到一个固定向量，位置 1001 的编码模型根本没学过，只能靠插值（效果差）。正弦编码用连续函数生成任意位置的编码，数学上保证平滑性，天然能外推。

---

## 五、可学习位置编码 vs 正弦位置编码

### 5.1 可学习位置编码

将位置编码当作可训练的参数 $P \in \mathbb{R}^{n_{max} \times d_{model}}$，通过反向传播学习。

**优点**：
- 理论上可以学到任意复杂的位置关系
- 在特定任务上可能达到更好的效果

**缺点**：
- 需要预先设定最大长度 $n_{max}$，无法处理超过训练长度的序列
- 参数量大：$n_{max} \times d_{model}$ 个参数
- 位置关系是离散的，泛化能力差
- 训练时见过的位置和未见过的位置有"鸿沟"

### 5.2 正弦位置编码

使用固定公式生成位置编码。

**优点**：
- 对任意长度都适用，无需重新训练
- 参数量为零（只有公式中的常数）
- 具有线性性质，泛化能力强
- 表达连续的位置关系

**缺点**：
- 位置关系受限于正弦函数的表达能力
- 无法针对特定任务调整

### 5.3 对比总结

| 特性 | 可学习位置编码 | 正弦位置编码 |
|------|--------------|-------------|
| 参数量 | $O(n_{max} \cdot d)$ | $O(1)$ |
| 外推能力 | 差（需要插值） | 强（数学函数） |
| 表达能力 | 可学任意复杂关系 | 受限于正弦函数 |
| 序列长度限制 | 需预先设定 | 任意长度 |
| 计算成本 | 查表 $O(1)$ | 三角函数 $O(d)$ |

**现代实践**：BERT、GPT 等大多使用**可学习位置编码**，因为：
1. 训练数据充足，很少遇到超长序列外推
2. GPU 计算三角函数的成本不可忽略
3. 可学习编码在实践中表现更好

但对于需要处理超长序列或训练数据不足的场景，正弦编码（或其变体）仍然是更好的选择。

---

## 六、现代位置编码技术

### 6.1 RoPE（Rotary Position Embedding）

RoPE 是 LLaMA、GLM-4 等模型采用的位置编码技术。

**核心思想**：不把位置编码加到词嵌入上，而是**对 Query 和 Key 旋转**。

对于位置 $pos$ 和维度 $d$，RoPE 定义旋转矩阵：
$$R_{pos} = \begin{bmatrix} \cos(pos\theta) & -\sin(pos\theta) & & \\ \sin(pos\theta) & \cos(pos\theta) & & \\ & & \ddots & \\ & & & \cos(pos\theta_i) & -\sin(pos\theta_i) \\ & & & \sin(pos\theta_i) & \cos(pos\theta_i) \end{bmatrix}$$

其中 $\theta_i = 10000^{-2i/d}$。

**关键性质**：RoPE 使得旋转后的 $Q$ 和 $K$ 的点积**只依赖相对位置**：
$$(R_{pos}^Q q)^T (R_{pos'}^K k) = q^T R_{pos-pos'} k$$

这意味着 attention score 只依赖于 $pos - pos'$，天然满足相对位置不变性。

### 6.2 ALiBi（Attention with Linear Biases）

ALiBi 是 Bloomberg 提出的方法，思路更简洁。

不修改嵌入，而是在 attention score 上加一个线性偏置：

$$score_{ij} = (q_i \cdot k_j) + |i - j| \cdot m$$

其中 $m$ 是一个预设的斜率（不同注意力头用不同的斜率）。

**优点**：
- 完全不需要位置编码
- 自然地编码相对距离
- 在长序列上泛化很好

### 6.3 总结对比

| 技术 | 年份 | 核心思想 | 外推能力 |
|------|------|---------|---------|
| 正弦编码 | 2017 | 数学函数生成 | 强 |
| 可学习 | 2018 | 查表 | 弱 |
| RoPE | 2021 | 旋转不变性 | 强 |
| ALiBi | 2022 | 线性偏置 | 强 |

---

## 七、代码实现：PyTorch

> [!example]- 🔧 PyTorch 实现正弦位置编码（点击展开）
>
> ```python
> import torch
> import math
>
> def sinusoidal_positional_encoding(max_len: int, d_model: int) -> torch.Tensor:
>     """
>     生成正弦位置编码
>
>     Args:
>         max_len: 最大序列长度
>         d_model: 嵌入维度
>
>     Returns:
>         (max_len, d_model) 的位置编码矩阵
>     """
>     pe = torch.zeros(max_len, d_model)
>
>     # 位置索引: (max_len, 1)
>     position = torch.arange(0, max_len).unsqueeze(1).float()
>
>     # 频率项: (d_model/2)
>     # div_term = 10000^(2i/d_model) for i in 0..d_model/2-1
>     div_term = torch.exp(
>         torch.arange(0, d_model, 2).float() *
>         (-math.log(10000.0) / d_model)
>     )
>
>     # 偶数维度: sin
>     pe[:, 0::2] = torch.sin(position * div_term)
>
>     # 奇数维度: cos
>     pe[:, 1::2] = torch.cos(position * div_term)
>
>     return pe
>
>
> # 使用示例
> max_len, d_model = 100, 512
> pe = sinusoidal_positional_encoding(max_len, d_model)
> print(f"Shape: {pe.shape}")  # (100, 512)
>
> # 可视化：位置 0 和位置 1 的编码差异
> diff = pe[1] - pe[0]
> print(f"位置1 - 位置0 的 L2 距离: {torch.norm(diff).item():.4f}")
>
> # 可视化：位置 100 和位置 200 的编码
> diff_100_200 = pe[200] - pe[100]
> print(f"位置200 - 位置100 的 L2 距离: {torch.norm(diff_100_200).item():.4f}")
>
> # 验证：不同位置的编码是否有足够的区分度
> # 计算所有位置对的余弦相似度矩阵
> cos_sim = torch.nn.functional.cosine_similarity(
>     pe.unsqueeze(1),  # (max_len, 1, d_model)
>     pe.unsqueeze(0),  # (1, max_len, d_model)
>     dim=2
> )
> print(f"余弦相似度矩阵 shape: {cos_sim.shape}")
> ```
>
> **关键实现细节**：
> 1. `div_term = exp(-log(10000) * 2i/d_model)` 等价于 $10000^{2i/d_{model}}$ 的倒数，避免直接计算大指数
> 2. 偶数维度用 sin，奇数维度用 cos，成对出现
> 3. 位置编码通常加到词嵌入上：`input_embeds + pe[:len(input)]`

---

## 八、与词嵌入的结合

完整的 Transformer 输入构建：

```python
# 伪代码
word_embedding = embedding(input_ids)  # (batch, seq_len, d_model)
pos_encoding = sinusoidal_positional_encoding(max_seq_len, d_model)
pos_encoding = pos_encoding[:seq_len]   # (seq_len, d_model)

# 广播相加
input_repr = word_embedding + pos_encoding  # (batch, seq_len, d_model)
```

**注意**：也有工作（如 BERT）使用可学习的位置编码，此时 `pos_encoding` 是可训练的参数。

> [!warning] 加法 vs 连接
> 原版 Transformer 使用**加法**（addition）结合位置编码。也有工作探索**连接**（concatenation），但加法更省参数，且实践中效果相当。

---

## 常见误区

| 误解 | 正确理解 |
|------|---------|
| "位置编码是 Transformer 的一部分" | 位置编码是**输入增强**，不是 Attention 本身的一部分 |
| "Self-Attention 知道 token 的顺序" | Self-Attention 是置换等变的，不知道绝对位置或相对位置 |
| "正弦编码比可学习编码更好" | 在长序列外推场景正弦更好；在充足训练数据下可学习编码往往更好 |
| "位置编码只是为了让模型知道'第几个'" | 位置编码还编码了位置之间的**关系**（相对距离），这对 Attention 至关重要 |
| "正弦编码的频率选择无所谓" | 频率决定了能捕捉的位置关系范围——太高无法捕捉远距离，太低无法精确区分近距离 |

---

## 思考题

### 基础理解

**Q1. 为什么 Self-Attention 本身无法区分"狗咬人"和"人咬狗"？**

> [!hint]- 💡 提示
> 考虑 Self-Attention 的置换等变性。如果把输入序列 [狗, 咬, 人] 重排为 [人, 咬, 狗]，Attention 的输出会怎么变化？

> [!success]- ✅ 参考答案
> Self-Attention 的输出满足 $f([狗, 咬, 人]) = [输出_{狗}, 输出_{咬}, 输出_{人}]$，而 $f([人, 咬, 狗])$ 只是这个输出的一个置换（[输出_{人}, 输出_{咬}, 输出_{狗}]）。"狗咬人"和"人咬狗"在 Self-Attention 看来只是不同的输入排列，模型学不到"谁咬谁"的区别，因为缺少位置信息来表达主语和宾语的关系。

**Q2. 正弦位置编码中，为什么用 10000 作为基数？换成其他数可以吗？**

> [!hint]- 💡 提示
> 基数影响频率的范围。考虑 $d_{model} = 512$，最小的频率是 $1/10000^{2/(d-1)}$。如果换成 100 或 100000 会怎样？

> [!success]- ✅ 参考答案
> 10000 决定了频率的范围（从 1 到 $1/10000$）。理论上可以用任何正数，但：
> - 太小（如 100）：低频分量周期太小，无法捕捉长距离位置关系
> - 太大（如 100000）：高频分量周期太大，区分近距离位置的能力下降
> 10000 是经验值，在 $d_{model}=512$ 时提供合适的位置分辨率（能区分 1-1000 个 token 内的位置关系）。

### 深入思考

**Q3. 假设 $d_{model}=4$，手算位置 0、1、2 的正弦编码（只算前 4 维，即 2 个频率对）。然后计算位置 0 和位置 2 的编码的内积，并与位置 0 和位置 1 的内积比较。这说明了什么？**

> [!hint]- 💡 提示
> $PE_{(pos, 2i)} = \sin(pos / 10000^{2i/d})$，$PE_{(pos, 2i+1)} = \cos(pos / 10000^{2i/d})$。对于 $d=4$，$i=0$ 时分母为 $10000^0=1$，$i=1$ 时分母为 $10000^{1/2}=100$。

> [!success]- ✅ 参考答案
> $d=4$，所以 $i=0, 1$（2个频率对）。
> $\omega_0 = 1/10000^0 = 1$，$\omega_1 = 1/10000^{1/2} = 1/100 = 0.01$
>
> $PE_0 = [\sin(0), \cos(0), \sin(0), \cos(0)] = [0, 1, 0, 1]$
> $PE_1 = [\sin(1), \cos(1), \sin(0.01), \cos(0.01)] \approx [0.84, 0.54, 0.01, 1.00]$
> $PE_2 = [\sin(2), \cos(2), \sin(0.02), \cos(0.02)] \approx [0.91, -0.42, 0.02, 1.00]$
>
> $PE_0 \cdot PE_1 \approx 0 \times 0.84 + 1 \times 0.54 + 0 \times 0.01 + 1 \times 1.00 = 1.54$
> $PE_0 \cdot PE_2 \approx 0 \times 0.91 + 1 \times (-0.42) + 0 \times 0.02 + 1 \times 1.00 = 0.58$
>
> 这说明**位置越远，内积越小（相似度越低）**。这符合直觉——位置 0 和位置 2 的距离是 2，位置 0 和位置 1 的距离是 1，距离越远相关性越低。正弦编码天然编码了相对位置的距离衰减特性。

---

## 关键要点回顾

1. **Self-Attention 是置换等变的**——打乱输入顺序，输出只是重排。这说明它本身不包含位置信息。
2. **位置编码是必需的**——要让模型理解"谁在第几位"和"谁离谁更近"，必须注入位置信号。
3. **正弦位置编码**使用不同频率的正弦/余弦函数：高频区分近距离，低频区分远距离。
4. **正弦编码具有线性性质**——$PE_{pos+k}$ 可以通过旋转矩阵从 $PE_{pos}$ 得到，这赋予了它强大的泛化/外推能力。
5. **可学习位置编码**在数据充足时通常表现更好，但正弦编码在长序列外推场景有优势。
6. **现代位置编码**（RoPE、ALiBi）进一步改进了相对位置编码和长序列泛化能力。

---

## 扩展阅读

- 📄 Vaswani et al. (2017) — *"Attention Is All You Need"* 第 3.5 节（位置编码的原始定义）
- 📝 Jay Alammar — [The Illustrated Transformer](https://jalammar.github.io/illustrated-transformer/)（位置编码的可视化解释）
- 📝 Lilian Weng — [Positional Encoding in Transformers](https://lilianweng.github.io/posts/2023-01-10-positional-encoding/)（全面的位置编码综述）
- 📝 Su et al. (2022) — *[RoFormer: Enhanced Pretraining with Rotary Position Embedding](https://arxiv.org/abs/2104.09864)*（RoPE 论文）
- 📝 Press et al. (2022) — *[Train Short, Test Long: Attention with Linear Biases](https://arxiv.org/abs/2108.12409)*（ALiBi 论文）
- 🎬 StatQuest — [Positional Encoding in Transformers (Natasha's Just Law)](https://www.youtube.com/watch?v=1biZf7C09gA)（YouTube 视频讲解）

---

## 相关页面

- [[02-self-attention|第 2 章：Self-Attention]]
- [[03-multi-head-attention|第 3 章：Multi-Head Attention]]
- [[05-ffn-residual-layernorm|第 5 章：FFN、残差连接与 Layer Normalization]]

## 下一步学习

➡️ [[05-ffn-residual-layernorm|第 5 章：FFN、残差连接与 Layer Normalization]] — Attention 之外的另一半：前馈网络与训练稳定性

或者：

➡️ [[06-encoder-decoder|第 6 章：编码器-解码器架构]] — 完整 Transformer 的整体结构
