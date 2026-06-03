---
type: lesson
tags: [面试, Transformer, 自注意力, 位置编码, FFN, LayerNorm]
created: 2026-05-20
updated: 2026-05-20
difficulty: advanced
prerequisites: [深度学习基础, 神经网络]
topic: Transformer核心原理深度
status: in-progress
series: {name: "AI Interview Prep", part: "1a"}
---

# Transformer 核心原理深度

> 从数学推导到工程实现，覆盖资深面试官可能追问的每一个细节。

## 学习目标

- [ ] 能完整推导 Self-Attention 的每一步数学计算
- [ ] 理解缩放因子 $\sqrt{d_k}$ 的方差证明
- [ ] 区分 MHA/MQA/GQA/MLA 的设计动机和适用场景
- [ ] 掌握 RoPE 的旋转矩阵推导和外推问题
- [ ] 能解释 Pre-Norm vs Post-Norm 的梯度流动差异
- [ ] 理解 FFN 的"信息加工"角色和 SwiGLU 激活
- [ ] 掌握 LM Head 的采样策略及其数学公式

---

## 一、Self-Attention 机制的完整数学推导

### 1.1 Q/K/V 投影

输入序列 $X \in \mathbb{R}^{n \times d}$，其中 $n$ 为序列长度，$d$ 为模型维度。通过三个线性投影矩阵得到查询、键、值：

$$Q = XW_Q, \quad K = XW_K, \quad V = XW_V$$

其中 $W_Q, W_K \in \mathbb{R}^{d \times d_k}$，$W_V \in \mathbb{R}^{d \times d_v}$。

**直觉**：每个 token 都会生成三个向量——"我在找什么"（Q）、"我有什么信息"（K）、"我能提供什么内容"（V）。Attention 就是让每个 token 的 Q 去和所有 token 的 K 匹配，按匹配程度加权聚合 V。

### 1.2 缩放点积注意力

$$\text{Attention}(Q, K, V) = \text{softmax}\!\left(\frac{QK^T}{\sqrt{d_k}}\right)V$$

**为什么不缩放会出问题？方差证明。**

假设 $q_i, k_i$ 的各分量独立同分布，均值为 0，方差为 1。点积 $q \cdot k = \sum_{i=1}^{d_k} q_i k_i$，则：

$$\text{Var}(q \cdot k) = \sum_{i=1}^{d_k} \text{Var}(q_i k_i) = \sum_{i=1}^{d_k} E[q_i^2]E[k_i^2] = d_k$$

> [!warning] 易混点：缩放因子是 $\sqrt{d_k}$ 不是 $\sqrt{d}$
> 缩放的是 Q 和 K 的投影维度 $d_k$，不是模型总维度 $d$。在 Multi-Head Attention 中，$d_k = d / h$。

当 $d_k$ 很大时（如 64 或 128），点积的方差很大，导致 softmax 输入的数值范围极大。softmax 在大数值下梯度趋于 0（饱和区），造成梯度消失。除以 $\sqrt{d_k}$ 将方差归一化为 1，使 softmax 工作在有效梯度区间。

**数值示例**：$d_k = 64$ 时，不缩放点积标准差为 8，可能产生 ±25 的值；缩放后标准差为 1，值域回到 ±3 左右，softmax 梯度健康。

### 1.3 完整矩阵计算示例

设 3 个 token，$d_k = 2$：

```
Q = [[1, 0],      K = [[1, 1],      V = [[1, 0],
     [0, 1],           [0, 1],           [0, 1],
     [1, 1]]           [1, 0]]           [1, 1]]
```

Step 1 — 注意力分数：$S = QK^T / \sqrt{2}$

```
S = [[1,0]·[1,0]  [1,0]·[1,1]  [1,0]·[0,1]]     [[0.71, 0.71, 0.00]
     [0,1]·[1,0]  [0,1]·[1,1]  [0,1]·[0,1]  /√2=  [0.00, 0.71, 0.71]
     [1,1]·[1,0]  [1,1]·[1,1]  [1,1]·[0,1]]       [0.71, 1.41, 0.71]]
```

Step 2 — softmax 逐行归一化 → 得到权重矩阵 $A$

Step 3 — 输出 $O = AV$，每个 token 得到所有 V 的加权和。

### 1.4 Multi-Head Attention

$$\text{MultiHead}(Q,K,V) = \text{Concat}(\text{head}_1, ..., \text{head}_h)W_O$$

其中 $\text{head}_i = \text{Attention}(QW_Q^i, KW_K^i, VW_V^i)$，每个头的维度 $d_k = d_v = d/h$。

**为什么多头等价于全维度 Attention？** 因为 Concat 后乘 $W_O \in \mathbb{R}^{d \times d}$ 本质上是一个线性变换。多头的好处不是表达能力更强（单头全维度 + 足够宽的 FFN 理论上也够），而是提供了**不同的子空间注意力模式**——一个头可能关注语法关系，另一个关注语义关系，各自独立学习。

### 1.5 Causal Mask

```
Mask = [[  0, -∞, -∞],      Softmax后 → [[1.00, 0.00, 0.00],
         [  0,  0, -∞],                    [0.30, 0.70, 0.00],
         [  0,  0,  0]]                    [0.20, 0.30, 0.50]]
```

上三角填充 $-\infty$，softmax 后这些位置变为 0，确保 token 只能看到自己和之前的 token。

> [!hint]- 面试题：Causal Mask 在 KV Cache 中有什么影响？
> KV Cache 缓存已计算的 K 和 V，生成第 $t$ 个 token 时只需要计算 $q_t$ 与所有已缓存 $K_{1:t}$ 的注意力。Mask 是隐含的——因为 $q_t$ 只与 $K_{1:t}$ 相乘，天然满足因果约束。KV Cache 让推理复杂度从 $O(n^2)$ 降低为 $O(n)$（单步），代价是显存占用线性增长。

---

## 二、FFN（前馈网络）深入

### 2.1 FFN 的角色定位

Transformer 层有两个子层：Attention 负责"信息路由"（决定哪些 token 的信息流向哪里），FFN 负责"信息加工"（对路由过来的信息做非线性变换）。

**类比**：Attention 是快递分拣中心（决定包裹去哪），FFN 是加工车间（打开包裹、处理内容、重新打包）。

### 2.2 参数占比

标准 FFN：$\text{FFN}(x) = \text{ReLU}(xW_1 + b_1)W_2 + b_2$

其中 $W_1 \in \mathbb{R}^{d \times 4d}$，$W_2 \in \mathbb{R}^{4d \times d}$。FFN 参数量 $= 8d^2$，而 Attention 参数量 $\approx 4d^2$（Q/K/V/O 四个投影矩阵）。FFN 占总参数量约 **2/3**。

> [!hint]- 面试题：为什么 FFN 的中间维度是 4d？
> 这是从实验中得出的经验值。过小则容量不足，过大则参数浪费。Llama 系列中这个比例（intermediate_size / hidden_size）通常在 2.7~3.5 之间浮动，配合 SwiGLU 后往往略小于 4。

### 2.3 SwiGLU 激活函数

$$\text{SwiGLU}(x, W, V, b, c) = (\text{Swish}_\beta(xW + b) \otimes (xV + c))$$

其中 $\text{Swish}_\beta(x) = x \cdot \sigma(\beta x)$，$\sigma$ 是 sigmoid 函数，$\otimes$ 是逐元素乘法。

**为什么比 ReLU 好？**

1. **平滑性**：Swish 处处可导，ReLU 在 0 点不可导，梯度可能不平稳
2. **非单调性**：Swish 在负值区有轻微的负输出（自门控效应），有助于信息流动
3. **GLU 机制**：门控让网络自适应选择哪些信息通过，比固定激活更具表达力
4. **实验证据**：PaLM、Llama 等大模型一致表明 SwiGLU 优于 GELU/ReLU

**GLU 变体对比**：

| 变体 | 门控激活 | 非门控路径 | 代表模型 |
|------|---------|-----------|---------|
| GLU | $\sigma(xV)$ | $xW$ | 早期 GLU 论文 |
| SwiGLU | $\text{Swish}(xV)$ | $xW$ | Llama, PaLM |
| GeGLU | $\text{GELU}(xV)$ | $xW$ | Gemma |
| BinaryGLU | $\text{sigmoid}(xV)$ | $xW$ | 简化版本 |

> [!warning] 易混点：SwiGLU 使 FFN 参数增加
> 标准 FFN 有 $W_1, W_2$ 两个矩阵。SwiGLU 引入了额外的门控矩阵 $V$，参数量为 $3 \times d \times d_{ff}$（三个矩阵）。为保持总参数不变，实践中通常将 $d_{ff}$ 从 $4d$ 缩小到 $\frac{8d}{3}$。

---

## 三、LayerNorm 深入

### 3.1 为什么用 LayerNorm 不用 BatchNorm

BatchNorm 对 batch 维度归一化：$\hat{x}_i = (x_i - \mu_B) / \sigma_B$。问题在于：

1. **序列长度可变**：不同样本的 token 数不同，batch 统计量不稳定
2. **自回归生成**：推理时每次只有 1 个 token，无法计算 batch 统计量
3. **小 batch 场景**：大模型训练常用小 batch + 大 seq_len，batch 统计量噪声大

LayerNorm 对特征维度归一化：$\hat{x} = (x - \mu_L) / \sigma_L$，每个 token 独立归一化，与 batch 大小和序列长度无关。

### 3.2 RMSNorm vs LayerNorm

$$\text{RMSNorm}(x) = \frac{x}{\text{RMS}(x)} \cdot \gamma, \quad \text{RMS}(x) = \sqrt{\frac{1}{d}\sum_{i=1}^d x_i^2}$$

RMSNorm 去掉了减均值步骤，只做缩放。Llama 采用 RMSNorm 的原因：

1. **计算效率**：省去均值计算，约减少 7~10% 的归一化层计算
2. **效果相当**：实验表明去掉均值偏移对模型质量几乎无影响
3. **数值稳定性**：RMS 总是非负的，避免了方差接近零时的除零问题

### 3.3 Pre-Norm vs Post-Norm

```
Post-Norm:  x' = LayerNorm(x + Attention(x))     ← 原始 Transformer
Pre-Norm:   x' = x + Attention(LayerNorm(x))      ← GPT-2, Llama
```

**现代模型都用 Pre-Norm 的原因**：

Post-Norm 的残差连接在归一化之后，梯度需要穿过 LayerNorm 回传。深层网络中梯度信号容易衰减。Pre-Norm 的残差连接是"干净"的——梯度可以直接通过 $x$ 分支回传，LayerNorm 只在注意力分支内部。

**数学分析**：Post-Norm 的梯度路径 $\frac{\partial L}{\partial x} = \frac{\partial L}{\partial x'} \cdot \frac{\partial \text{LN}}{\partial x}$，LayerNorm 的 Jacobian 矩阵会缩放梯度。Pre-Norm 的梯度路径 $\frac{\partial L}{\partial x} = \frac{\partial L}{\partial x'} \cdot (I + \frac{\partial \text{Attn}}{\partial x})$，恒等项 $I$ 保证梯度至少不衰减。

> [!hint]- 面试题：Pre-Norm 有什么缺点？
> Pre-Norm 的残差分支恒等性更强，导致深层网络中各层的"有效深度"趋于一致——后面的层对输出的贡献越来越小，出现"表征塌缩"。一些工作（如 DeepNorm、Admin）尝试结合两者优势。但在工程实践中，Pre-Norm 的训练稳定性优势远大于其理论缺陷。

---

## 四、位置编码深入

### 4.1 RoPE 的完整推导

**核心思想**：用旋转矩阵编码位置，使得 $q_m$ 和 $k_n$ 的点积自然包含相对位置 $m - n$。

对于二维向量 $[x_1, x_2]$，位置 $m$ 的旋转编码：

$$\begin{pmatrix} x_1' \\ x_2' \end{pmatrix} = \begin{pmatrix} \cos m\theta & -\sin m\theta \\ \sin m\theta & \cos m\theta \end{pmatrix} \begin{pmatrix} x_1 \\ x_2 \end{pmatrix}$$

对于 $d$ 维向量，按相邻两两分组，每组的旋转角度不同：

$$\theta_i = 10000^{-2i/d}, \quad i = 0, 1, ..., d/2-1$$

**为什么点积包含相对位置**：

$$\langle f(q, m), f(k, n) \rangle = \text{Re}\left[\sum_{i} (q_{2i} + jq_{2i+1})(k_{2i} + jk_{2i+1})^* e^{j(m-n)\theta_i}\right]$$

结果只依赖 $m - n$，实现了相对位置编码。

**直觉**：把 token 的 embedding 想象成复平面上的向量，位置编码就是旋转这个向量。两个向量的点积（复数内积的实部）自然反映了它们之间的角度差——也就是位置差。

> [!warning] 易混点：RoPE 是乘性编码不是加性编码
> 正弦位置编码（原始 Transformer）是加法：$x + PE(pos)$。RoPE 是乘法：$R(pos) \cdot x$。这意味着 RoPE 不增加额外的参数维度，而是直接修改 Q 和 K 的方向。

### 4.2 RoPE 的外推问题

训练时见过的最大序列长度为 $L_{train}$，推理时超过 $L_{train}$ 会怎样？RoPE 的旋转角度在长序列上会超出训练分布，注意力模式崩溃。

**三种解决方案对比**：

| 方法 | 核心思想 | 优点 | 缺点 |
|------|---------|------|------|
| NTK-aware Scaling | 调整基频 $\theta$，使高频分量不受影响 | 简单有效 | 需要调参 |
| YaRN | 结合 NTK + 温度缩放 + 注意力权重调整 | 效果最好 | 实现复杂 |
| ALiBi | 直接在注意力分数上加线性偏置 | 无需修改 RoPE | 与 RoPE 不兼容 |

**NTK-aware Scaling 公式**：

$$\theta'_i = b' \cdot 10000^{-2i/d}, \quad b' = b \cdot s^{d/(d-2)}$$

其中 $s$ 是扩展比例，$b$ 是原始基频（10000）。直觉：低频分量（大的 $\theta_i^{-1}$）被拉伸更多，高频分量（小的 $\theta_i^{-1}$）保持不变，确保局部注意力模式不受影响。

### 4.3 RoPE 代码实现

```python
import torch
import torch.nn as nn
import math

class RotaryEmbedding(nn.Module):
    def __init__(self, dim, base=10000):
        super().__init__()
        inv_freq = 1.0 / (base ** (torch.arange(0, dim, 2).float() / dim))
        self.register_buffer("inv_freq", inv_freq)

    def forward(self, seq_len):
        t = torch.arange(seq_len, device=self.inv_freq.device).float()
        freqs = torch.outer(t, self.inv_freq)  # (seq_len, dim/2)
        return torch.cat([freqs, freqs], dim=-1)  # (seq_len, dim)

def rotate_half(x):
    x1, x2 = x.chunk(2, dim=-1)
    return torch.cat([-x2, x1], dim=-1)

def apply_rotary_emb(q, cos, sin):
    # q: (batch, heads, seq_len, dim)
    return q * cos + rotate_half(q) * sin
```

> [!hint]- 面试题：为什么 RoPE 只作用于 Q 和 K，不作用于 V？
> RoPE 的目的是让注意力权重反映相对位置关系——即"我关注谁"应该考虑距离。注意力权重由 Q 和 K 的点积决定，所以 RoPE 只需作用于 Q 和 K。V 代表的是"被关注的内容"，内容本身不需要位置调制。

---

## 五、注意力变体完整对比

### 5.1 架构对比图

```
MHA (Multi-Head Attention)  — 标准方案
┌─────────────────────────────────────┐
│  Q → [h heads × d_k]                │
│  K → [h heads × d_k]  ← 每个 K/V 独 │
│  V → [h heads × d_v]    立的投影矩阵 │
│  参数量: 4d² (Q,K,V,O)              │
└─────────────────────────────────────┘

MQA (Multi-Query Attention) — GQA的极端版
┌─────────────────────────────────────┐
│  Q → [h heads × d_k]                │
│  K → [1 head × d_k]  ← 所有 Q 头共  │
│  V → [1 head × d_v]    享同一组 K/V │
│  参数量: d² + 2d×d_k (省~2/3)       │
└─────────────────────────────────────┘

GQA (Grouped-Query Attention) — Llama 2/3
┌─────────────────────────────────────┐
│  Q → [h heads × d_k]                │
│  K → [g groups × d_k] ← g组, 每组   │
│  V → [g groups × d_v]   h/g个Q头共享│
│  参数量: d² + 2g×d×d_k/h            │
└─────────────────────────────────────┘
```

### 5.2 MLA（Multi-head Latent Attention）— DeepSeek

MLA 的核心是对 K 和 V 做低秩压缩，减少 KV Cache 的显存占用：

$$k = W_{DK} \cdot W_{UK} \cdot c, \quad v = W_{DV} \cdot W_{UV} \cdot c$$

其中 $c \in \mathbb{R}^{d_c}$ 是压缩后的 latent 向量（$d_c \ll d$），$W_{UK}, W_{UV}$ 是上投影矩阵。KV Cache 只需存储 $c$ 而非完整的 K 和 V，缓存大小从 $2 \times n_{layers} \times d$ 降到 $n_{layers} \times d_c$。

> [!hint]- 面试题：GQA 和 MLA 分别解决什么问题？如何选型？
> GQA 减少 K/V 的头数，降低 KV Cache 显存，适合通用推理加速。MLA 用低秩压缩进一步压缩 KV Cache，在超长序列（128K+）场景下优势更大。如果你的场景是长文本推理且显存受限，MLA 更优；如果是通用部署且需要兼顾训练效率，GQA 是更简单的选择。

---

## 六、Embedding 和 LM Head

### 6.1 Tied Embedding

Token Embedding 矩阵 $E \in \mathbb{R}^{V \times d}$（$V$ 是词表大小）和 LM Head 的输出投影 $W_{out} \in \mathbb{R}^{d \times V}$ 共享权重：$W_{out} = E^T$。

**为什么有效**：输入时 token $i$ 对应 $E$ 的第 $i$ 行；输出时预测 token $i$ 的概率等价于计算隐藏状态与 $E$ 第 $i$ 行的点积。语义上一致——"能理解一个词"和"能预测一个词"应该共享表示。

**为什么有些模型不用**：Llama 不 tie，将 Embedding 和 LM Head 解耦，给两个任务更大的自由度，在大规模训练中效果略好。

### 6.2 采样策略

**Temperature**：

$$p_i = \frac{\exp(z_i / T)}{\sum_j \exp(z_j / T)}$$

$T > 1$ 使分布更平坦（更随机），$T < 1$ 使分布更尖锐（更确定）。$T \to 0$ 退化为贪心解码。

**Top-k**：只保留概率最高的 $k$ 个 token，其余置零后重新归一化。

**Top-p（Nucleus Sampling）**：按概率从高到低累加，保留累计概率 $\leq p$ 的最小 token 集合。

```python
def top_p_sampling(logits, p=0.9, temperature=1.0):
    logits = logits / temperature
    probs = torch.softmax(logits, dim=-1)
    sorted_probs, sorted_indices = torch.sort(probs, descending=True)
    cumulative_probs = torch.cumsum(sorted_probs, dim=-1)
    # 移除累计概率超过 p 的 token
    remove_mask = cumulative_probs - sorted_probs > p
    sorted_probs[remove_mask] = 0
    sorted_probs /= sorted_probs.sum()
    # 从过滤后的分布中采样
    idx = torch.multinomial(sorted_probs, 1)
    return sorted_indices.gather(-1, idx)
```

**Beam Search**：维护 $b$ 个候选序列，每步扩展所有候选并保留 top-$b$。适合翻译、摘要等确定性任务，不适合开放式生成（输出重复、缺乏多样性）。

> [!hint]- 面试题：Top-k 和 Top-p 的区别是什么？各自在什么场景下更合适？
> Top-k 固定候选数量，无论分布的形状如何始终选 k 个。当分布平坦时（模型不确定），k 个 token 可能包含很多低质量选项。Top-p 自适应候选数量——分布集中时选很少的 token，分布分散时选更多。Top-p 在大多数生成场景下更优。但在代码生成等需要高确定性的场景，小的 Top-k（如 k=5）配合低 Temperature 效果可能更好。

## 关键要点回顾

1. **缩放因子 $\sqrt{d_k}$**：将点积方差归一化为 1，防止 softmax 梯度消失
2. **FFN 是参数大户**：占 Transformer 参数 2/3，SwiGLU 是当前最优激活
3. **Pre-Norm 胜在稳定**：梯度通过恒等路径直传，训练深层网络更容易
4. **RoPE 是乘性编码**：只作用于 Q/K，点积自然包含相对位置
5. **GQA/MLA 为推理服务**：减少 KV Cache 显存，GQA 更简单，MLA 压缩率更高
6. **采样策略无银弹**：Top-p 适合通用生成，Beam Search 适合确定性任务

## 扩展阅读

- Vaswani et al., "Attention Is All You Need" (2017)
- Shazeer, "GLU Variants Improve Transformer" (2020)
- Su et al., "RoFormer" (RoPE 原始论文, 2021)
- DeepSeek-AI, "DeepSeek-V2" (MLA, 2024)
- Touvron et al., "Llama 2" (GQA, 2023)

## 下一步学习

- [[01b-training-pipeline|LLM训练流程深入]] — 预训练/SFT/RLHF/DPO 完整链路
