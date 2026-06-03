---
type: lesson
tags:
  - 深度学习
  - Transformer
  - 训练
  - 推理
  - Label Smoothing
  - Warmup
  - Beam Search
  - KV Cache
created: 2026-04-09
updated: 2026-04-09
topic: 训练与推理——从理论到实践的桥梁
difficulty: advanced
prerequisites:
  - "[[06-encoder-decoder|编码器-解码器架构]]"
status: completed
series:
  name: Transformer 深度学习
  part: 7
---

# 训练与推理：从理论到实践的桥梁

> 本章将揭示 Transformer 如何从"搭建好的架构"变成"可用的模型"——包括训练阶段的损失函数、学习率调度、正则化策略，以及推理阶段的解码策略、采样方法和 KV Cache 加速技术。阅读完成后，你将理解为什么同样的架构，不同的训练策略会导致截然不同的效果。

## 上下文：完整的 Transformer 架构

在第 6 章中，我们已经组装了完整的 Transformer 架构：编码器负责理解输入序列，解码器负责生成输出序列。两者之间通过 Cross-Attention 机制连接，让解码器的每一层都能"看到"编码器的输出。

架构组件包括：
- **Multi-Head Self-Attention**：编码器内并行理解输入
- **Masked Self-Attention**：解码器内防止看到未来位置
- **Cross-Attention**：解码器"查阅"编码器信息
- **FFN**：逐位置的非线性变换
- **残差连接 + LayerNorm**：稳定训练、梯度流动

但架构只是骨架，**训练**赋予它灵魂，**推理**让它产生价值。本章我们将深入训练与推理的工程细节。

---

## 第一部分：训练阶段

### 1.1 损失函数：交叉熵

#### 直观理解

想象你在教一个小朋友认识动物。你拿出一张狗的图片说"这是狗"，但小朋友回答"这是猫"。你会怎么评价这个回答？

显然，小朋友答错了，但"猫"比"石头"要接近正确答案。**交叉熵损失**就是衡量这种"距离"的方式——它不仅惩罚错误答案，还根据错误答案与正确答案的相似度给予不同力度的惩罚。

在语言模型中，词汇表里的每个词都可以看作一个"类别"。给定前文，模型对下一个词的打分（logit）经过 Softmax 后变成概率分布。交叉熵损失衡量的是：模型预测的概率分布与真实标签（one-hot 分布）之间的距离。

#### 数学形式

对于一个真实的下一个词 $w_t$，模型的交叉熵损失为：

$$L_{CE} = -\log p_\theta(w_t | w_{1:t-1})$$

其中 $p_\theta$ 是模型参数 $\theta$ 下的条件概率。**注意**：这里是对"真实词"的概率取负对数，所以当模型对真实词预测的概率越高，损失越小。

#### 技术细节

> [!example]- 代码实现：交叉熵损失
> ```python
> import torch
> import torch.nn.functional as F
>
> # 假设 vocab_size = 10000
> # 模型对下一个词的 logit 输出
> logits = torch.randn(32, 10000)  # batch_size=32, seq_len 已在前面处理
>
> # 真实标签 (batch_size,)
> targets = torch.randint(0, 10000, (32,))
>
> # PyTorch 的交叉熵损失（内置 Softmax）
> loss = F.cross_entropy(logits, targets)
> print(f"平均损失: {loss.item():.4f}")
> # 对应 -log(p(真实词))，如果损失=2.3，意味着 log(p) ≈ -2.3，p ≈ 0.1
> ```

**在 Transformer 训练中，我们通常对整个序列计算交叉熵损失**：将目标序列平移一位作为标签，计算每个位置的损失后取平均。

---

### 1.2 Label Smoothing：为什么 $\epsilon = 0.1$？

#### 核心思想

传统交叉熵损失假设真实标签是 **100% 确定的 one-hot 向量**：是狗就是 100% 狗，绝不可能是猫。但现实世界是模糊的——"狗"和"狼"在某些特征上非常相似，"猫"和"老虎"也是。

Label Smoothing 的核心思想是：**不再要求模型对真实标签 100% 置信，而是允许一点不确定性**。具体做法是将 one-hot 标签软化为：

$$q'_i = (1 - \epsilon) \cdot q_i + \epsilon \cdot \frac{1}{K}$$

其中 $q_i$ 是原始 one-hot 分布（只有真实类为 1），$K$ 是类别总数，$\epsilon$ 是平滑参数（通常设为 0.1）。

#### 直觉类比

想象一个学生在考试。传统训练像是一个**只认满分的老师**：99 分和 0 分一样糟糕。Label Smoothing 则像一个**宽容的老师**：90 分以上都是好学生，85 分也勉强接受。

这种宽容带来一个好处：模型不会过度"自信"。一个 99% 置信说"这是狗"的模型，其实更容易在遇到模糊样本时犯错；而一个 80% 置信说"这大概是狗"的模型，反而更有可能正确处理边缘情况。

#### 效果对比

| 配置 | 训练集准确率 | 测试集准确率 | 泛化能力 |
|------|------------|------------|---------|
| 无 Label Smoothing | 很高 | 较低 | 容易过拟合 |
| Label Smoothing (ε=0.1) | 略低 | 更高 | 更好的泛化 |

#### 公式拆解

以 $\epsilon = 0.1$、词汇表大小 $K = 10000$ 为例：

原始 one-hot（假设真实词是第 5 个词）：
$$q = [0, 0, 0, 0, 1, 0, ..., 0]$$

平滑后：
$$q'_5 = (1 - 0.1) \cdot 1 + 0.1 \cdot \frac{1}{10000} = 0.9 + 0.00001 = 0.90001$$

$$q'_i = 0 + 0.1 \cdot \frac{1}{10000} = 0.00001 \quad (i \neq 5)$$

所以模型的目标从"P(第5词)=1.0"变成了"P(第5词)=0.9, P(其他词)≈0.00001"。

---

### 1.3 学习率调度：Warmup + Inverse Square Root

#### 为什么需要学习率调度？

**直觉角度**：想象你在下山。山路陡峭时（训练初期），步子太大会错过最低点；山路平缓时（训练后期），步子太小又走不动。理想的做法是**初期小步探路，后期大步下山**。

**技术角度**：Transformer 的训练动态非常不稳定。在早期，模型参数剧烈变化，梯度方向不稳定，此时使用大学习率会导致训练崩溃。后期模型趋于收敛，需要更小的学习率来精细调整。

#### Transformer 的学习率公式

原版论文《Attention Is All You Need》提出的学习率调度公式为：

$$lr = d_{model}^{-0.5} \cdot \min\left(step^{-0.5}, step \cdot warmup^{-1.5}\right)$$

这个公式看起来复杂，但我们来逐步拆解。

#### 逐步拆解

**第一部分：$d_{model}^{-0.5}$**

这是一个与模型维度相关的缩放因子。$d_{model}$ 是模型的隐藏层维度（通常为 512）。这个因子相当于一个归一化，让学习率与模型规模适配。

- 如果 $d_{model} = 512$，则 $512^{-0.5} = \frac{1}{\sqrt{512}} \approx \frac{1}{22.6} \approx 0.044$

**第二部分：$\min(step^{-0.5}, step \cdot warmup^{-1.5})$**

这部分控制学习率随训练步数的动态变化：

- **在 warmup 阶段**（$step < warmup$）：学习率随步数线性增加
  - $step \cdot warmup^{-1.5}$ 与步数成正比，斜率为 $warmup^{-1.5}$
  - 假设 $warmup = 4000$，则斜率 $= 4000^{-1.5} = \frac{1}{4000^{1.5}} = \frac{1}{4000 \cdot \sqrt{4000}} \approx 4e-6$

- **在 warmup 之后**（$step \geq warmup$）：学习率随步数衰减
  - $step^{-0.5}$ 与步数的平方根成反比，即学习率 $\propto \frac{1}{\sqrt{step}}$
  - 这是标准的 **inverse square root decay**

#### 数值示例

假设 $d_{model} = 512$，$warmup = 4000$，基础学习率因子 $= 1.0$

| Step | 计算过程 | 学习率 |
|------|---------|--------|
| 0 | 线性区初始 | ~0 |
| 1000 | $1 \cdot 1000 \cdot 4000^{-1.5} \approx 0.00125$ | $0.044 \times 0.00125 \approx 5.5e-5$ |
| 2000 | $1 \cdot 2000 \cdot 4000^{-1.5} \approx 0.0025$ | $0.044 \times 0.0025 \approx 1.1e-4$ |
| 4000 | 线性区末尾，与衰减区交汇 | $0.044 \times 0.0044 \approx 2.0e-4$ |
| 8000 | 衰减区：$8000^{-0.5} \approx 0.0112$ | $0.044 \times 0.0112 \approx 4.9e-4$ |
| 16000 | 衰减区：$16000^{-0.5} \approx 0.0079$ | $0.044 \times 0.0079 \approx 3.5e-4$ |
| 50000 | 衰减区：$50000^{-0.5} \approx 0.0045$ | $0.044 \times 0.0045 \approx 2.0e-4$ |

#### 为什么这样设计？

1. **Warmup 阶段（0 → warmup）**：从零开始，逐步增加学习率，让模型有时间适应梯度的方向变化，避免早期的不稳定更新破坏已学到的有用表示。

2. **Inverse Square Root 衰减（warmup 之后）**：比简单的线性衰减或常数衰减更平滑，符合训练后期"精细调整"的需求。

#### PyTorch 实现

> [!example]- 代码实现：学习率调度器
> ```python
> from torch.optim.lr_scheduler import LambdaLR
> import math
>
> def get_lr_scheduler(optimizer, d_model, warmup_steps):
>     def lr_lambda(step):
>         # d_model^{-0.5} * min(step^{-0.5}, step * warmup^{-1.5})
>         return d_model ** (-0.5) * min(
>             step ** (-0.5),
>             step * (warmup_steps ** (-1.5))
>         )
>     return LambdaLR(optimizer, lr_lambda)
>
> # 使用示例
> optimizer = torch.optim.Adam(model.parameters(), lr=0)
> scheduler = get_lr_scheduler(optimizer, d_model=512, warmup_steps=4000)
>
> # 训练循环
> for step in range(50000):
>     train_step()
>     scheduler.step()
>     if step % 1000 == 0:
>         print(f"Step {step}: lr = {scheduler.get_last_lr()[0]:.6f}")
> ```

---

### 1.4 正则化：Dropout

#### Dropout 的直观理解

Dropout 就像是在训练时**随机让一些神经元"请假"**。每个神经元有 $p$ 的概率被临时关闭（输出置零），每次训练迭代都是一次不同的"随机Subnet"参与计算。

**为什么这样做？** 如果模型对某个神经元的依赖过重，这个神经元就成了"单点故障"。Dropout 强制模型学习**冗余的表示**——即使某些神经元失效，关键信息依然能被其他神经元传递。

#### 在 Transformer 中的位置

原版 Transformer 在以下位置使用 Dropout（$p_{drop} = 0.1$）：

1. **Multi-Head Attention 输出的残差连接之前**：每个注意力头的输出
2. **FFN 输出的残差连接之前**：FFN 的最终输出
3. **注意力权重的 Softmax 之后**（可选）：某些实现会在此处也加 Dropout
4. **Embedding 层**（某些变体）：在输入的 embedding 上应用

#### 技术细节

> [!example]- Dropout 在 PyTorch 中的使用
> ```python
> import torch.nn as nn
>
> class TransformerLayer(nn.Module):
>     def __init__(self, d_model, nhead, dim_feedforward, dropout=0.1):
>         super().__init__()
>         self.self_attn = nn.MultiheadAttention(d_model, nhead, dropout=dropout)
>         self.ffn = nn.Sequential(
>             nn.Linear(d_model, dim_feedforward),
>             nn.GELU(),
>             nn.Dropout(dropout),
>             nn.Linear(dim_feedforward, d_model),
>         )
>         self.norm1 = nn.LayerNorm(d_model)
>         self.norm2 = nn.LayerNorm(d_model)
>         self.dropout = nn.Dropout(dropout)  # 注意力后的 Dropout
>
>     def forward(self, x, mask=None):
>         # Self-attention with residual
>         attn_out, _ = self.self_attn(x, x, x, attn_mask=mask)
>         x = self.norm1(x + self.dropout(attn_out))
>
>         # FFN with residual
>         ffn_out = self.ffn(x)
>         x = self.norm2(x + self.dropout(ffn_out))
>
>         return x
> ```

**重要**：Dropout 在训练时启用，在推理时关闭。推理时所有神经元都参与计算，输出需要乘以 $(1-p)$ 或在训练时对输出进行缩放以保持期望一致。

---

## 第二部分：推理阶段

### 2.1 贪心解码 vs Beam Search

#### 贪心解码（Greedy Decoding）

**原理**：每一步都选择概率最高的词。

$$\hat{w}_t = \arg\max_{w} P(w | w_{1:t-1})$$

**优点**：
- 速度快：只需一次 Softmax 计算
- 实现简单：直接的 argmax 操作

**缺点**：
- **局部最优陷阱**：每步最优不等于全局最优。例如：
  - 第一步选择 "a"（概率 0.6），放弃 "the"（概率 0.4）
  - 第二步 "a" 后跟 "dog"（概率 0.9）→ 联合概率 0.54
  - 但 "the" 后跟 "dog"（概率 0.95）→ 联合概率 0.38

  看起来贪心赢了，但如果考虑第三步：
  - "a dog runs"（0.8）→ 0.432
  - "the dog runs"（0.95）→ 0.361

  还是贪心赢。但如果 "the dog" 后跟 "is" 的概率是 0.99，而 "a dog" 后跟 "is" 的概率只有 0.5：
  - "a dog is" → 0.54 × 0.5 = 0.27
  - "the dog is" → 0.38 × 0.99 = 0.376

  这时贪心就错了！**早期的次优选择可能让后续序列更优**。

#### Beam Search

**原理**：维护 $k$ 个"候选序列"（称为 beam），每一步扩展每个候选的所有可能，然后保留概率最高的 $k$ 个。

**直观理解**：Beam Search 像是一个**有远见的探索者**。不是每次都走看起来最好的路，而是同时保留几条有希望的路，看谁能走得更远。

#### 算法步骤

假设 beam size = 3，要生成最多 10 个词：

```
Step 0: 从 <s> 开始
  候选1: <s> the    (log_prob = -0.4)
  候选2: <s> a      (log_prob = -0.5)
  候选3: <s> some   (log_prob = -0.7)

Step 1: 扩展每个候选
  候选1: <s> the dog    (log_prob = -0.4 + -0.2 = -0.6)
  候选1: <s> the cat    (log_prob = -0.4 + -0.3 = -0.7)
  候选2: <s> a dog      (log_prob = -0.5 + -0.1 = -0.6)
  候选2: <s> a cat      (log_prob = -0.5 + -0.4 = -0.9)
  候选3: <s> some dog   (log_prob = -0.7 + -0.2 = -0.9)
  ...

  取 top-3: <s> the dog, <s> a dog, <s> the cat

Step 2+: 依此类推...
```

#### 贪心 vs Beam Search 对比

| 特性 | 贪心解码 | Beam Search (k=5) |
|------|---------|------------------|
| 时间复杂度 | O(T) | O(k × T × V) |
| 空间复杂度 | O(1) | O(k × T) |
| 找到全局最优概率 | 不保证 | 更可能（但不保证） |
| 生成多样性 | 低 | 中等 |
| 适合场景 | 实时系统、对话 | 机器翻译、摘要 |

#### Length Penalty 修正

Beam Search 有一个常见问题：**倾向于生成短句子**。因为对数概率是负数，序列越长，累积 log_prob 越负。可以通过 Length Penalty 修正：

$$\text{score} = \frac{\log P(w_{1:T})}{HP^{\alpha}(T)}$$

其中 $HP(T) = \frac{5 + T}{6}$ 是长度惩罚项，$\alpha$ 通常设为 0.6 或 0.7。分母增大短序列的惩罚，多用于机器翻译。

---

### 2.2 Teacher Forcing：训练与推理的差异

#### 问题背景

训练时我们有**真实标签**（ground truth），可以强制让模型学习正确的前文→词的关系：

```
训练: "我 爱 [你]" → 让模型预测 "你"，前文是 "我 爱"
```

但推理时，模型只能用自己的输出作为下一步的输入：

```
推理: <s> → 预测 "我" → 用 "我" 预测 "爱" → 用 "我 爱" 预测 "你" → ...
```

这就是 **Teacher Forcing 问题**：训练和推理时，前文的来源不一致。

#### Teacher Forcing 的策略

**策略 1：Free Running（自由运行）**
- 训练时也用模型自己的预测作为下一步输入
- 优点：训练和推理行为一致
- 缺点：训练初期，错误会累积，导致学习效率低

**策略 2：Always Teacher Forcing（总是强制）**
- 训练时始终使用真实标签作为下一步输入
- 优点：训练效率高，梯度稳定
- 缺点：**曝光偏差（Exposure Bias）**：模型没学过如何从自己的错误中恢复

**策略 3：Scheduled Sampling（计划采样）**
- 训练时以一定概率 $p$ 使用真实标签，$(1-p)$ 使用模型预测
- $p$ 随训练进度逐渐降低（如从 1.0 降到 0.5）
- 优点：缓解曝光偏差
- 缺点：混合训练目标，训练不稳定

**策略 4：No Teacher Forcing + 课程学习**
- 早期用 Teacher Forcing 快速入门
- 后期切换到 Free Running
- 现代大型模型（如 GPT 系列）通常直接用 Free Running，因为模型足够强大，能自己修正错误

---

### 2.3 Sampling 策略：Top-k 和 Top-p

#### 朴素采样的问题

如果直接按模型输出的概率分布采样，**高概率词会主导生成**，但偶尔也会采样到低概率词。这可能导致：

- 生成不连贯的词（如突然跳到一个无关主题）
- 生成重复或无意义的短语

#### Top-k Sampling

**原理**：只从概率最高的 $k$ 个词中采样，其余词概率置零后重新归一化。

```
原始分布: P(狗)=0.4, P(猫)=0.3, P(狼)=0.15, P(狐狸)=0.1, P(其他)=0.05
Top-k=2:  只从 {狗, 猫} 采样
新分布:   P(狗)=0.4/0.7≈0.57, P(猫)=0.3/0.7≈0.43
```

**问题**：$k$ 是固定值，可能不适用所有上下文。某些上下文需要更大的 $k$ 来保持多样性，某些则需要更小的 $k$ 来保持一致性。

#### Top-p (Nucleus Sampling)

**原理**：动态选择概率之和达到 $p$ 的最小词集合，然后在此集合内采样。

```
词汇按概率排序: 狗(0.4), 猫(0.3), 狼(0.15), 狐狸(0.1), 鸟(0.03), ...
累积概率:       0.4,    0.7,    0.85,     0.95,    0.98, ...

如果 p=0.9，则选择累积概率首次超过 0.9 的最小集合：{狗, 猫, 狼, 狐狸}
```

#### 对比

| 策略 | 优点 | 缺点 |
|------|------|------|
| 贪心 (argmax) | 确定性强 | 缺乏多样性，容易重复 |
| Top-k | 控制多样性，简单直观 | k 是超参数，不自适应 |
| Top-p | 自适应词汇量，适应性更强 | 某些极端分布可能退化为贪心 |
| Temperature | 控制随机性 | 需要调参，可能过度平滑 |

#### Temperature 采样

Temperature $T$ 修改 Softmax 的温度：

$$P_T(w_i) = \frac{\exp(z_i / T)}{\sum_j \exp(z_j / T)}$$

- $T = 1$：标准 Softmax
- $T > 1$：分布更平坦（更高多样性）
- $T < 1$：分布更尖锐（更确定性）

> [!example]- 完整采样代码
> ```python
> import torch
> import torch.nn.functional as F
>
> def sample_with_top_p(logits, p=0.9, temperature=1.0):
>     """Top-p (nucleus) sampling with temperature"""
>     if temperature != 1.0:
>         logits = logits / temperature
>
>     # 将 logits 转换为概率分布
>     probs = F.softmax(logits, dim=-1)
>
>     # 按概率降序排序
>     sorted_probs, sorted_indices = torch.sort(probs, descending=True)
>
>     # 计算累积概率
>     cumulative_probs = torch.cumsum(sorted_probs, dim=-1)
>
>     # 找到最小集合使累积概率超过 p
>     mask = cumulative_probs > p
>     # 第一个超过 p 的位置设为 False（保留）
>     mask[..., 1:] = mask[..., :-1].clone()
>     mask[..., 0] = True  # 至少保留一个
>
>     # 将不在集合中的词概率置零
>     sorted_probs[~mask] = 0
>
>     # 重新归一化
>     filtered_probs = sorted_probs / sorted_probs.sum(dim=-1, keepdim=True)
>
>     # 从过滤后的分布中采样
>     sample_idx = torch.multinomial(filtered_probs, 1)
>
>     # 映射回原始词表索引
>     return sorted_indices.gather(-1, sample_idx)
>
> # 使用示例
> logits = torch.randn(10000)  # 模型输出的原始分数
> next_word = sample_with_top_p(logits, p=0.9, temperature=0.8)
> ```


---

### 2.4 KV Cache：推理加速的关键优化

#### 自回归生成的问题

在推理时，Transformer 需要**逐词生成**。每生成一个词，都要重新计算整个序列的注意力：

```
生成第1词: 输入 [<s>]
生成第2词: 输入 [<s>, w1]
生成第3词: 输入 [<s>, w1, w2]  ← 需要重新计算 <s> 和 w1 的注意力！
生成第4词: 输入 [<s>, w1, w2, w3]  ← 需要重新计算 <s>, w1, w2 的注意力！
```

这是一个 **$O(T^2)$ 的问题**：序列长度变为 2 倍，计算量变为 4 倍。

#### KV Cache 的核心思想

**关键观察**：生成第 $t$ 个词时，只需要计算与 $w_t$ 的注意力，而之前的词 $w_{1:t-1}$ 的 Key 和 Value 向量**在每一步都不会改变**。

因此，我们可以**缓存**这些 Key-Value 对，每次只需要计算新词的 Query，以及从缓存中获取之前所有词的 Key 和 Value。

#### 图示理解

```
无 KV Cache:
  Step 1: Q=[w1], K=[<s>,w1], V=[<s>,w1]  → 计算注意力
  Step 2: Q=[w2], K=[<s>,w1,w2], V=[<s>,w1,w2]  → 重新计算 w1 的注意力！
  Step 3: Q=[w3], K=[<s>,w1,w2,w3], V=[<s>,w1,w2,w3]  → 重新计算 w1, w2 的注意力！

有 KV Cache:
  Step 1: Q=[w1], K=[<s>], V=[<s>]  → 计算注意力，缓存 K,V
  Step 2: Q=[w2], K=[<s>,w1], V=[<s>,w1]  → 从缓存获取 w1，只计算 w2 的注意力
  Step 3: Q=[w3], K=[<s>,w1,w2], V=[<s>,w1,w2]  → 从缓存获取 w1,w2，只计算 w3 的注意力
```

#### 计算节省

| 序列长度 T | 无 KV Cache | 有 KV Cache | 节省比例 |
|-----------|------------|------------|---------|
| 10 | 100 | 55 | 45% |
| 50 | 2500 | 1275 | 49% |
| 100 | 10000 | 5050 | 49.5% |
| 500 | 250000 | 125250 | 49.9% |

KV Cache 将复杂度从 $O(T^2)$ 降低到约 $O(T)$，因为每一步只需要计算当前位置的 Q-K-V 乘法。

#### KV Cache 在 Multi-Head Attention 中的实现

> [!example]- KV Cache 代码示例
> ```python
> class TransformerDecoderLayer(nn.Module):
>     def __init__(self, d_model, nhead, dropout=0.1):
>         super().__init__()
>         self.self_attn = nn.MultiHeadAttention(d_model, nhead, dropout=dropout)
>         self.norm1 = nn.LayerNorm(d_model)
>
>     def forward_with_cache(self, x, kv_cache=None):
>         if kv_cache is None:
>             attn_out, _ = self.self_attn(x, x, x)
>             new_k = attn_out
>             new_v = attn_out
>         else:
>             cached_k = kv_cache['k']
>             cached_v = kv_cache['v']
>             attn_out, _ = self.self_attn(x, cached_k, cached_v)
>             new_k = torch.cat([cached_k, x], dim=0)
>             new_v = torch.cat([cached_v, x], dim=0)
>
>         new_cache = {'k': new_k, 'v': new_v}
>         return self.norm1(x + attn_out), new_cache
> ```

**实际工业实现**（如 FasterTransformer、vLLM）会使用更复杂的显存管理策略，包括：
- **Paged Attention**：将 KV Cache 分页管理，减少显存碎片
- **连续批处理**：合并多个请求的 KV Cache 操作
- **动态申请**：按需分配显存

---

## 第三部分：完整训练流程与评估

### 3.1 训练一个翻译模型的完整流程

#### 阶段 1：数据准备

```
原始平行语料:
  "Hello, world!" → "你好，世界！"
  "The cat sat on the mat." → "猫坐在垫子上。"

分词 (Tokenization):
  英文: ["Hello", ",", "world", "!", "<eos>"]
  中文: ["你", "好", "，", "世", "界", "！", "<eos>"]

构建词表 (Vocabulary):
  英文词表: 32000 tokens
  中文词表: 25000 tokens

数据转换为 token IDs:
  src = [234, 12, 1893, 5, 1]
  tgt = [102, 893, 4, 567, 23, 8, 1]
  label = [893, 4, 567, 23, 8, 1, 2]
```

#### 阶段 2：训练配置

> [!example]- 训练配置示例
> ```python
> training_config = {
>     'd_model': 512,
>     'nhead': 8,
>     'num_encoder_layers': 6,
>     'num_decoder_layers': 6,
>     'dim_feedforward': 2048,
>     'dropout': 0.1,
>     'label_smoothing': 0.1,
>     'optimizer': 'Adam',
>     'lr': 0,
>     'beta1': 0.9,
>     'beta2': 0.98,
>     'eps': 1e-9,
>     'warmup_steps': 4000,
>     'max_steps': 100000,
>     'gradient_clip': 1.0,
>     'batch_size': 4096,
> }
> ```

#### 阶段 3：训练循环

```python
model = build_transformer(config)
optimizer = torch.optim.Adam(model.parameters(), lr=0)
scheduler = get_lr_scheduler(optimizer, config['d_model'], config['warmup_steps'])

for step in range(config['max_steps']):
    src, tgt = next(data_loader)
    src = src.to(device)
    tgt = tgt.to(device)
    logits = model(src, tgt)
    loss = compute_loss(logits, tgt_labels, label_smoothing=0.1)
    loss.backward()
    torch.nn.utils.clip_grad_norm_(model.parameters(), 1.0)
    optimizer.step()
    scheduler.step()
    optimizer.zero_grad()
    if step % 5000 == 0:
        val_loss = evaluate(model, val_data)
        bleu = evaluate_bleu(model, test_data)
        print(f"Step {step}: loss={loss:.4f}, val_loss={val_loss:.4f}, BLEU={bleu:.2f}")
```

#### 阶段 4：评估指标

##### BLEU 分数简介

**BLEU (Bilingual Evaluation Understudy)** 是机器翻译最常用的评估指标之一，通过计算生成译文与参考译文的 **n-gram 重叠度**来衡量翻译质量。

**核心思想**：一个好的翻译应该与参考译文在词序和短语上有一定的重叠。BLEU 分数越高，说明翻译越接近人类译文。

**计算公式**：

$$\text{BLEU} = \text{BP} \cdot \exp\left(\sum_{n=1}^{N} w_n \log p_n\right)$$

其中：
- $p_n$ 是 n-gram 精确度
- $w_n = \frac{1}{N}$ 是均匀权重（通常 $N=4$）
- BP 是简短惩罚因子

**BP 公式**：

$$\text{BP} = \begin{cases} 1 & \text{if } c > r \ e^{(1 - r/c)} & \text{if } c \le r \end{cases}$$

其中 $c$ 是生成译文长度，$r$ 是参考译文长度。

**直观理解**：
- BLEU 分数范围 0~100
- > 40：可接受
- > 60：很好
- < 20：质量较差

**BLEU 的局限性**：
- 只关注表面重叠，不理解语义
- 对同义词不友好
- 无法评估句子结构的多样性

---

## 常见误区

| 误区 | 正确理解 |
|------|---------|
| Label Smoothing 会让模型预测变得模糊 | Label Smoothing 的目标是**正则化**，让模型不过度依赖单一路径。预测时依然取概率最高的词，只是概率分布更平滑 |
| 学习率 Warmup 是为了让模型热身 | Warmup 的真正目的是**稳定性**：早期参数随机，梯度方向不稳定，大学习率会导致训练崩溃 |
| Beam Search 的 k 越大越好 | k 增大会提升质量，但边际效益递减，且计算成本线性增长。通常 k=3~5 是合理的折中 |
| KV Cache 会影响模型输出的准确性 | KV Cache **只改变计算方式，不改变模型本身**。只要正确实现，输出完全一致 |
| Teacher Forcing 是一种强制学习方法 | Teacher Forcing 是**训练技巧**，不是模型必须遵守的约束。Free Running 在足够大的模型上同样有效 |
| BLEU 分数越高，翻译质量越好 | BLEU 只是**表面指标**，无法捕捉语义正确性、流畅度等问题。高 BLEU 不等于高质量翻译 |

---

## 思考题

> [!hint]- 基础题 1：Label Smoothing 的概率分布
> 如果词汇表大小 $K=5000$，Label Smoothing 参数 $\epsilon=0.2$，真实词的概率平滑后是多少？

> [!success]- 答案
> 平滑后真实词的概率为 $(1-\epsilon) + \epsilon/K = 0.8 + 0.2/5000 = 0.8 + 0.00004 = 0.80004$。每个非真实词的概率为 $\epsilon/K = 0.00004$。

---

> [!hint]- 基础题 2：贪心解码的问题
> 假设模型对马的下一个词的概率分布如下：
> - 是 (0.5), 有 (0.3), 在 (0.1), 跑 (0.05), 其他 (0.05)
> 贪心解码会选择哪个词？为什么这不是全局最优的可能？

> [!success]- 答案
> 贪心解码选择是（概率最高）。但如果有之后跟跑的概率是 0.9，而是之后跟跑的概率只有 0.1，那么有马跑的联合概率 (0.27) 会高于是马跑的联合概率 (0.05)。

---

> [!hint]- 基础题 3：KV Cache 的作用
> 为什么推理时需要 KV Cache？它节省了什么计算？

> [!success]- 答案
> 自回归生成时，每步都需要重新计算之前所有词的注意力。KV Cache 通过缓存之前词的 Key 和 Value，避免重复计算。对于长度为 T 的序列，复杂度从 $O(T^2)$ 降低到约 $O(T)$。

---

> [!hint]- 进阶题：学习率调度分析
> 假设 $d_{model}=256$，$warmup=2000$。请计算 step=1000 和 step=8000 时的学习率，并解释为什么 warmup 之后的学习率反而更高？

> [!success]- 答案
> - Step 1000 (warmup 阶段)：$lr = 256^{-0.5} \times 1000 \times 2000^{-1.5} \approx 0.0625 \times 0.00112 \approx 7e-5$
> - Step 8000 (衰减阶段)：$lr = 256^{-0.5} \times 8000^{-0.5} \approx 0.0625 \times 0.0112 \approx 7e-4$
>
> 确实 step=8000 的学习率比 step=1000 高 10 倍。这是因为 warmup 阶段的学习率是**线性增长**到峰值，而衰减阶段遵循 inverse square root 规律。Step=4000 时达到峰值，之后才逐渐下降。所以 warmup 之后、学习率开始下降之前，是学习率最高的阶段。

---

## 关键要点回顾

1. **交叉熵损失**是语言模型的标配损失函数，等价于最大化真实词的概率（等价于最小化负对数似然）

2. **Label Smoothing (ε=0.1)** 通过软化 one-hot 标签的正则化技术，提升模型的泛化能力，防止过度自信

3. **Transformer 的学习率调度**采用 Warmup + Inverse Square Root 策略：前期线性增加学习率避免训练不稳定，后期逐渐衰减实现精细调优

4. **Dropout** 在 Transformer 的多个位置使用（注意力输出、FFN 输出），通过随机丢弃神经元强制学习冗余表示

5. **贪心解码**简单快速但可能陷入局部最优；**Beam Search** 通过维护多条候选路径找到更优解，但计算成本也更高

6. **Teacher Forcing** 解决了训练与推理输入分布不一致的问题，但可能导致曝光偏差；现代大模型通常直接使用 Free Running

7. **Top-p (Nucleus) Sampling** 相比 Top-k 能自适应调整候选词集合，在多样性和质量间取得更好平衡

8. **KV Cache** 通过缓存已计算的 Key-Value 向量，将推理复杂度从 $O(T^2)$ 降低到 $O(T)$

9. **BLEU** 通过 n-gram 重叠度评估翻译质量，虽是行业标准但有语义盲区的局限性

---

## 扩展阅读

### 论文

- **《Attention Is All You Need》** (Vaswani et al., 2017) — 原版 Transformer 论文，包含详细的训练和推理配置
- **《BLEU: a Method for Automatic Evaluation of Machine Translation》** (Papineni et al., 2002) — BLEU 分数的原始论文
- **《Scheduled Sampling for Sequence Prediction with Recurrent Neural Networks》** (Bengio et al., 2015) — Teacher Forcing 和 Scheduled Sampling 的经典论文
- **《Generating Sequences With Neural Networks》** (Sutskever et al., 2014) — Sequence-to-Sequence 模型和 Beam Search 的早期工作

### 博客与教程

- **《The Illustrated Transformer》** (Jay Alammar) — 图解 Transformer，清晰易懂
- **《Neural Machine Translation by Jointly Learning to Align and Translate》** (Bahdanau Attention) — 注意力机制的早期形式
- **《How to Generate Text: Using Different Decoding Methods for Language Models》** (Hugging Face Blog) — 各种解码策略的对比分析

### 视频课程

- **Stanford CS224N Lecture 9-10** — Neural Machine Translation and Sequence-to-Sequence Models
- **3Blue1Brown Neural Networks Series** — 帮助建立对深度学习的直观理解

---

## 下一步学习

恭喜完成 Transformer 训练与推理的学习！

接下来可以继续：

- **[[08-beyond-original|第 8 篇：超越原始 Transformer]]** — 探索 Transformer 的进化之路：Layer Normalization 的位置、Pre-LN vs Post-LN、位置编码的改进、Parallel Transformer、Linear Attention 等前沿技术

或者深入研究：

- **[[wiki/concepts/LLM-training|大型语言模型训练]]** — 从 Transformer 到 GPT 系列的训练策略演变
- **[[wiki/concepts/RLHF|RLHF (人类反馈强化学习)]]** — 如何用强化学习进一步优化模型输出
