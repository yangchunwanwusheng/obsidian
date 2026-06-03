---
type: lesson
tags:
  - 深度学习
  - Transformer
  - 编码器
  - 解码器
  - Encoder-Decoder
  - Cross-Attention
  - Masked Attention
created: 2026-04-09
updated: 2026-04-09
topic: 编码器-解码器架构——从零件到整机
difficulty: intermediate
prerequisites:
  - "[[02-self-attention|Self-Attention]]"
  - "[[03-multi-head-attention|Multi-Head Attention]]"
  - "[[04-positional-encoding|位置编码]]"
  - "[[05-ffn-residual-layernorm|FFN、残差连接与 Layer Normalization]]"
status: completed
series:
  name: Transformer 深度学习
  part: 6
---

# 编码器-解码器架构：从零件到整机

> 把前四章的所有零件——Self-Attention、Multi-Head Attention、位置编码、FFN、残差连接、LayerNorm——组装成完整的 Transformer 架构。本章你将看到，一句话从输入到输出的完整数据流。

## 学习目标

- [ ] 能画出完整的 Transformer 编码器-解码器架构图
- [ ] 理解编码器和解码器的区别（Cross-Attention 是关键）
- [ ] 能解释 Masked Self-Attention 为什么是必要的
- [ ] 能追踪一句话从输入到输出的完整数据流
- [ ] 理解自回归生成的过程

## 前置知识

| 知识点 | 要求 | Wiki 参考 |
|--------|------|-----------|
| Self-Attention | 理解 Q/K/V 和缩放点积 | [[02-self-attention]] |
| Multi-Head Attention | 理解多头拆分和拼接 | [[03-multi-head-attention]] |
| 位置编码 | 理解正弦编码的作用 | [[04-positional-encoding]] |
| FFN + 残差 + LayerNorm | 理解 Add & Norm 流程 | [[05-ffn-residual-layernorm]] |

---

## 直观理解：翻译流水线

> [!tip] 类比：双团队协作翻译
> 想象一个翻译公司有两个团队：
>
> **理解团队（编码器）**：阅读原文，逐句理解含义，输出一份"理解报告"。团队中有多个专员（6 层编码器），每人都在前一位的理解基础上加深认识。
>
> **翻译团队（解码器）**：根据理解团队的报告，逐字写出翻译。但翻译团队有一个严格规则——**不能偷看后面还没翻到的字**。翻译过程中，他们会不断回头参考理解团队的报告（Cross-Attention）。
>
> 这就是 Transformer 的核心范式：**编码器理解输入 → 解码器生成输出**。

---

## 一、Transformer 全景架构

### 1.1 完整架构图

```
输入："I love AI"                              输出："我爱人工智能"
    │                                               ↑
    ▼                                               │
┌─────────────────────┐                   ┌─────────────────────┐
│   Input Embedding   │                   │   Output Embedding  │
│   + Positional Enc  │                   │   + Positional Enc  │
└─────────┬───────────┘                   └──────────┬──────────┘
          │                                          │
          ▼                                          ▼
┌─────────────────────┐                   ┌─────────────────────┐
│  Encoder Layer ×N   │                   │  Decoder Layer ×N   │
│  ┌───────────────┐  │                   │  ┌───────────────┐  │
│  │ Multi-Head    │  │                   │  │ Masked MHA     │  │
│  │ Self-Attention│  │                   │  │ (自注意力)     │  │
│  │ + Add & Norm  │  │                   │  │ + Add & Norm   │  │
│  ├───────────────┤  │                   │  ├───────────────┤  │
│  │ Feed Forward  │  │◄──── Cross-Attn ──│  │ Cross-Attention│  │
│  │ + Add & Norm  │  │                   │  │ + Add & Norm   │  │
│  └───────────────┘  │                   │  ├───────────────┤  │
└─────────┬───────────┘                   │  │ Feed Forward  │  │
          │                               │  │ + Add & Norm   │  │
          │                               │  └───────────────┘  │
          │                               └──────────┬──────────┘
          │                                          │
          └──────────────┐                           │
                         ▼                           ▼
                   ┌───────────┐              ┌───────────┐
                   │  编码器   │              │  Linear   │
                   │  输出     │              │  + Softmax│
                   └───────────┘              └───────────┘
                                                        │
                                                        ▼
                                                  词概率分布
```

### 1.2 两个关键数字

原始 Transformer 的超参数：

| 超参数 | 值 | 含义 |
|--------|-----|------|
| $N$ | 6 | 编码器和解码器各堆叠 6 层 |
| $d_{model}$ | 512 | 模型的隐藏维度 |
| $h$ | 8 | 多头注意力的头数 |
| $d_k = d_v$ | 64 | 每个头的维度（512 / 8） |
| $d_{ff}$ | 2048 | FFN 中间层维度（4 × 512） |

---

## 二、编码器：理解输入的专家

### 2.1 单层编码器的结构

编码器的每一层包含两个子层：

```
输入
 │
 ▼
┌─────────────────────────┐
│  Multi-Head Self-Attention │  ← 所有位置互相看
│  + Add & Norm              │
├─────────────────────────┤
│  Position-wise FFN         │  ← 每个位置独立加工
│  + Add & Norm              │
└─────────────────────────┘
 │
 ▼
输出（传递给下一层）
```

用公式表示：

$$\text{Sublayer}_1(x) = \text{LayerNorm}(x + \text{MultiHead}(x, x, x))$$

$$\text{Sublayer}_2(x) = \text{LayerNorm}(x + \text{FFN}(x))$$

> [!note] 注意力类型
> 编码器中使用的是**标准的 Self-Attention**——每个位置可以看到所有其他位置（包括"未来"的位置）。因为编码器是在处理完整的输入句子，不存在"偷看未来"的问题。

### 2.2 堆叠 $N=6$ 层

为什么需要堆叠多层？

| 层级 | 可能学到的模式 |
|------|---------------|
| 第 1-2 层 | 基础词法特征（词性、词间共现） |
| 第 3-4 层 | 句法结构（主谓宾、修饰关系） |
| 第 5-6 层 | 语义特征（指代消解、语义角色） |

每一层都在前一层输出的基础上，提取更抽象的特征——类似 CNN 从边缘→纹理→部件→物体的层级提取。

### 2.3 数据流追踪（编码器）

输入句子 "I love AI"：

```
Step 1: Token Embedding
  "I" → [0.1, 0.3, ..., 0.2]    (512维向量)
  "love" → [0.5, -0.1, ..., 0.4]
  "AI" → [-0.2, 0.6, ..., 0.3]

Step 2: + Positional Encoding
  "I" → [0.1+PE(0), 0.3+PE(0), ...]    (加上位置0的编码)
  "love" → [0.5+PE(1), -0.1+PE(1), ...]
  "AI" → [-0.2+PE(2), 0.6+PE(2), ...]

Step 3: Layer 1
  Multi-Head Self-Attention → 每个词融合了其他词的上下文信息
  Add & Norm → 保留原始信息 + 归一化
  FFN → 对每个位置的非线性变换
  Add & Norm → 保留信息 + 归一化

Step 4-8: Layer 2-6
  重复 Step 3 的结构，逐层提取更抽象的特征

输出: 3 个 512 维的向量，每个都融合了完整的上下文信息
```

**关键洞察**：编码器的输出中，"I" 的向量不再只是"I"的含义——它已经融合了 "love" 和 "AI" 的上下文信息。这是一个**上下文化的表示（contextualized representation）**。

---

## 三、解码器：带着约束的生成者

### 3.1 与编码器的区别

解码器的每一层包含**三个子层**（比编码器多一个）：

```
输入（已生成的 token）
 │
 ▼
┌─────────────────────────┐
│  Masked Multi-Head       │  ← 只看"过去"，不看"未来"
│  Self-Attention          │
│  + Add & Norm            │
├─────────────────────────┤
│  Multi-Head              │  ← Q 来自解码器，K/V 来自编码器
│  Cross-Attention         │
│  + Add & Norm            │
├─────────────────────────┤
│  Position-wise FFN       │  ← 每个位置独立加工
│  + Add & Norm            │
└─────────────────────────┘
 │
 ▼
输出
```

### 3.2 Masked Self-Attention：为什么不能偷看？

在训练时，解码器的输入是完整的目标序列（如 "我 爱 人工智能"）。但如果不加限制，"我"在 Self-Attention 时就能看到 "爱"、"人工智能"——这等于考试时偷看答案。

**解决方案**：在计算注意力分数后，将"未来位置"的分数设为 $-\infty$，这样 Softmax 后这些位置的权重趋近于 0。

```
注意力分数矩阵（未 Mask）:        Mask 矩阵:
  我   爱   人工  智能              0   -∞   -∞   -∞
[0.3  0.5  0.2  0.4]             0    0   -∞   -∞
[0.1  0.6  0.3  0.5]      →      0    0    0   -∞
[0.2  0.4  0.5  0.3]             0    0    0    0
[0.3  0.3  0.4  0.6]

Masked + Softmax 后:
  我    爱    人工  智能
[1.00  0.00  0.00  0.00]    ← "我" 只看自己
[0.35  0.65  0.00  0.00]    ← "爱" 看 "我" 和自己
[0.25  0.38  0.37  0.00]    ← "人工" 看前面三个
[0.20  0.28  0.25  0.27]    ← "智能" 看全部
```

> [!warning] 为什么是 $-\infty$？
> 因为 $e^{-\infty} = 0$，Softmax 后权重为零。不能用 0，因为 $e^0 = 1$，仍然有贡献。

### 3.3 Cross-Attention：连接编码器和解码器的桥梁

这是解码器最关键的子层。Cross-Attention 的特殊之处：

| | Self-Attention | Cross-Attention |
|---|---|---|
| Q 来源 | 解码器自身 | 解码器自身 |
| K 来源 | 解码器自身 | **编码器输出** |
| V 来源 | 解码器自身 | **编码器输出** |

公式：

$$\text{CrossAttn}(Q_d, K_e, V_e) = \text{softmax}\left(\frac{Q_d \cdot K_e^T}{\sqrt{d_k}}\right) V_e$$

> [!tip] 类比：翻译员的目光
> Cross-Attention 就像翻译员在输出"爱"这个词时，回头看向编码器对 "love" 的表示——解码器在**问**"我现在要输出什么？"，编码器在**答**"你应该关注输入的这个位置"。

**维度追踪**：
- $Q_d \in \mathbb{R}^{n_{dec} \times d_k}$（$n_{dec}$ 是解码器序列长度）
- $K_e \in \mathbb{R}^{n_{enc} \times d_k}$（$n_{enc}$ 是编码器序列长度）
- $V_e \in \mathbb{R}^{n_{enc} \times d_v}$
- 注意力矩阵 $A \in \mathbb{R}^{n_{dec} \times n_{enc}}$（不是方阵！）
- 输出 $\in \mathbb{R}^{n_{dec} \times d_v}$

---

## 四、完整数据流：一句话的旅程

让我们追踪 "I love AI" → "我爱人工智能" 的完整翻译过程：

### 4.1 编码阶段

```
"I love AI"
    │
    ▼ [Tokenization]
["I", "love", "AI"]
    │
    ▼ [Embedding + Positional Encoding]
X_enc = [[emb(I)+PE₀], [emb(love)+PE₁], [emb(AI)+PE₂]]   ← 3×512 矩阵
    │
    ▼ [Encoder Layer 1]
    Multi-Head Self-Attention → Add & Norm → FFN → Add & Norm
    │
    ▼ [Encoder Layer 2-6]  (重复 5 次)
    │
    ▼
H_enc = [[h_I], [h_love], [h_AI]]   ← 3×512 矩阵（上下文化的编码器输出）
```

### 4.2 解码阶段

```
训练时输入: "我爱人工智能"（右移一位，开头加 <BOS>）
    │
    ▼ [Tokenization + Embedding + PE]
X_dec = [[emb(<BOS>)+PE₀], [emb(我)+PE₁], [emb(爱)+PE₂], ...]
    │
    ▼ [Decoder Layer 1]
    │
    ├─→ Masked Self-Attention（只看已生成的位置）→ Add & Norm
    │
    ├─→ Cross-Attention（Q=解码器, K/V=编码器输出 H_enc）→ Add & Norm
    │
    └─→ FFN → Add & Norm
    │
    ▼ [Decoder Layer 2-6]  (重复 5 次)
    │
    ▼ [Linear + Softmax]
P(下一个词) = [0.01, 0.85, 0.02, ..., 0.05]
              ↑ "我" 的概率最高
```

### 4.3 自回归生成

训练时可以一次处理整个序列（因为 Teacher Forcing），但**推理时是逐 token 生成的**：

```
Step 1: 输入 <BOS>          → 模型预测 "我"   (概率 0.85)
Step 2: 输入 <BOS> 我       → 模型预测 "爱"   (概率 0.72)
Step 3: 输入 <BOS> 我 爱    → 模型预测 "人工" (概率 0.68)
Step 4: 输入 <BOS> 我 爱 人工 → 模型预测 "智能" (概率 0.81)
Step 5: 输入 <BOS> 我 爱 人工 智能 → 模型预测 <EOS> (结束)
```

这就是**自回归生成（autoregressive generation）**——每一步的输出成为下一步的输入，直到生成结束符 `<EOS>`。

---

## 五、Embedding 与 Softmax 的权重共享

原始 Transformer 的一个设计细节：**输入 Embedding 矩阵、输出 Embedding 矩阵、以及 Softmax 前的 Linear 层共享同一个权重矩阵**。

$$\text{共享权重} \in \mathbb{R}^{vocab\_size \times d_{model}}$$

> [!note] 为什么共享？
> 1. **参数量减少**：不共享时 Embedding + Linear 共需 $2 \times vocab\_size \times d_{model}$ 参数；共享后减半
> 2. **语义一致性**：输入时 "猫" 映射到某个向量，输出时这个向量也对应 "猫"——映射方向一致
> 3. **注意**：Embedding 权重需要乘以 $\sqrt{d_{model}}$，因为正弦位置编码的值域大约在 $[-1, 1]$，而 Embedding 的值域取决于初始化

---

## 六、代码视角：完整 Transformer 模型

> [!example]- 🔧 PyTorch 完整实现（点击展开）
>
> ```python
> import torch
> import torch.nn as nn
> import math
>
> class Transformer(nn.Module):
>     def __init__(self, vocab_size, d_model=512, nhead=8,
>                  num_encoder_layers=6, num_decoder_layers=6,
>                  dim_feedforward=2048, dropout=0.1):
>         super().__init__()
>         self.d_model = d_model
>
>         # Embedding 层
>         self.embedding = nn.Embedding(vocab_size, d_model)
>         self.pos_encoder = PositionalEncoding(d_model, dropout)
>
>         # 编码器
>         encoder_layer = nn.TransformerEncoderLayer(
>             d_model, nhead, dim_feedforward, dropout, batch_first=True)
>         self.encoder = nn.TransformerEncoder(encoder_layer, num_encoder_layers)
>
>         # 解码器
>         decoder_layer = nn.TransformerDecoderLayer(
>             d_model, nhead, dim_feedforward, dropout, batch_first=True)
>         self.decoder = nn.TransformerDecoder(decoder_layer, num_decoder_layers)
>
>         # 输出层
>         self.fc_out = nn.Linear(d_model, vocab_size)
>
>     def forward(self, src, tgt, src_mask=None, tgt_mask=None):
>         # 编码器
>         src_emb = self.pos_encoder(self.embedding(src) * math.sqrt(self.d_model))
>         memory = self.encoder(src_emb, src_mask)
>
>         # 解码器
>         tgt_emb = self.pos_encoder(self.embedding(tgt) * math.sqrt(self.d_model))
>         output = self.decoder(tgt_emb, memory, tgt_mask=tgt_mask)
>
>         # 输出
>         return self.fc_out(output)
>
> class PositionalEncoding(nn.Module):
>     def __init__(self, d_model, dropout=0.1, max_len=5000):
>         super().__init__()
>         self.dropout = nn.Dropout(dropout)
>         pe = torch.zeros(max_len, d_model)
>         position = torch.arange(0, max_len).unsqueeze(1).float()
>         div_term = torch.exp(
>             torch.arange(0, d_model, 2).float() * (-math.log(10000.0) / d_model))
>         pe[:, 0::2] = torch.sin(position * div_term)
>         pe[:, 1::2] = torch.cos(position * div_term)
>         self.register_buffer('pe', pe.unsqueeze(0))
>
>     def forward(self, x):
>         return self.dropout(x + self.pe[:, :x.size(1)])
> ```
>
> PyTorch 已经内置了完整的 Transformer 模块。核心代码不到 30 行。

---

## 常见误区

| 误解 | 正确理解 |
|------|---------|
| "编码器和解码器的结构完全相同" | 解码器多一个 Cross-Attention 子层和 Mask 机制 |
| "Masked Attention 只在推理时使用" | 训练时也需要 Mask，否则"偷看"未来位置 |
| "编码器的输出只有一个向量" | 编码器输出 $n$ 个向量（每个输入 token 一个），全部传递给解码器 |
| "解码器只看编码器的最后一个位置" | Cross-Attention 让解码器的每个位置都能关注编码器的所有位置 |
| "Transformer 只能做翻译" | 编码器-解码器架构可用于任何 seq2seq 任务（摘要、对话、代码生成等） |

---

## 思考题

> [!hint]- 💡 思考题 1（基础）：为什么编码器不需要 Mask？
> 编码器处理的是完整的输入句子，它需要在所有位置之间做 Self-Attention。为什么不像解码器那样加 Mask？

> [!success]- ✅ 参考答案
> 因为编码器的目标是**理解完整的输入**——它同时处理所有输入 token，不存在"还没看到后面的内容"的问题。输入句子在编码器处理前就已经完整可见，所以不需要 Mask。而解码器需要**逐 token 生成**，不能在生成第 $t$ 个 token 时看到第 $t+1$ 个及之后的 token。

> [!hint]- 💡 思考题 2（基础）：Cross-Attention 的维度
> 如果编码器输入有 5 个 token，解码器当前有 3 个 token，Cross-Attention 的注意力矩阵是什么形状？这说明了什么？

> [!success]- ✅ 参考答案
> 注意力矩阵 $A \in \mathbb{R}^{3 \times 5}$。3 行对应解码器的 3 个位置，5 列对应编码器的 5 个位置。每一行表示解码器的某个位置对编码器所有位置的"关注分布"。这说明**解码器的每个位置可以独立地决定关注编码器的哪些位置**。

> [!hint]- 💡 思考题 3（进阶）：Decoder-Only 的可能
> GPT 系列模型只使用解码器，不需要编码器。如果去掉编码器，Cross-Attention 子层也需要去掉。那么一个 Decoder-Only 模型的结构是什么？它适合做什么任务？

> [!success]- ✅ 参考答案
> Decoder-Only 模型每层只有两个子层：Masked Self-Attention + FFN（没有 Cross-Attention）。由于没有编码器提供输入序列的表示，输入和输出都通过同一个序列处理。这种架构特别适合**语言建模**任务——给定前面的 token，预测下一个 token。GPT 系列就是基于这个架构。缺点是无法直接处理"输入-输出"对的任务（如翻译），但可以通过 Prompt 的方式间接实现。

---

## 关键要点回顾

1. **Transformer** 由编码器（理解输入）和解码器（生成输出）组成，各 6 层
2. **编码器**每层包含：Multi-Head Self-Attention + FFN，各配 Add & Norm
3. **解码器**每层多一个 **Cross-Attention**——Q 来自解码器，K/V 来自编码器
4. **Masked Self-Attention** 防止解码器在生成时"偷看未来"
5. **自回归生成**：推理时逐 token 生成，每步输出成为下步输入
6. **Embedding 权重共享**：输入/输出 Embedding + Softmax 前的 Linear 层共享权重

---

## 扩展阅读

- 📄 Vaswani et al. (2017) — *"Attention Is All You Need"* Section 3（完整架构定义）
- 📝 Harvard NLP — [The Annotated Transformer](https://nlp.seas.harvard.edu/2018/04/03/attention.html)（代码级注释，逐行解读论文）
- 📝 Jay Alammar — [The Illustrated Transformer](https://jalammar.github.io/illustrated-transformer/)（最佳可视化教程）
- 📝 Peter Bloem — [Transformers from Scratch](https://peterbloem.nl/blog/transformers)（从零实现）

---

## 相关页面

- [[01-why-transformer|第 1 章：为什么需要 Transformer]]
- [[02-self-attention|第 2 章：Self-Attention]]
- [[03-multi-head-attention|第 3 章：Multi-Head Attention]]
- [[04-positional-encoding|第 4 章：位置编码]]
- [[05-ffn-residual-layernorm|第 5 章：FFN、残差与 LayerNorm]]

## 下一步学习

➡️ [[07-training-inference|第 7 章：训练与推理]] — 从理论到实践的桥梁
