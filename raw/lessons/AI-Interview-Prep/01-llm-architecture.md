---
type: lesson
tags: [面试, LLM, Transformer, 注意力机制, 推理优化, 量化]
created: 2026-05-19
updated: 2026-05-19
difficulty: advanced
prerequisites: [深度学习基础, 神经网络基础]
topic: LLM架构深度
status: in-progress
series: {name: "AI Interview Prep", part: 1}
---

# LLM 架构深度——从 Transformer 到推理优化

> 面试必考核心：理解大语言模型从底层原理到工程优化的完整链路，能讲清楚每一个"为什么"。

## 学习目标

- [ ] 能完整画出 Transformer 架构图并逐层解释
- [ ] 理解 MHA/GQA/MQA 等注意力变体的区别与取舍
- [ ] 掌握 LLM 训练三阶段（预训练→SFT→RLHF/DPO）的原理
- [ ] 能解释 KV Cache、Flash Attention、量化等推理优化技术
- [ ] 能对比 GPT / Llama / GLM / DeepSeek 等主流架构差异

## 一、Transformer 架构——面试第一关

### 1.1 整体架构

```
Input: "我 喜欢 人工智能"
          │
          ▼
    ┌─────────────┐
    │ Tokenizer   │  ← 分词：文本 → token IDs
    └──────┬──────┘
           │ [12, 456, 789]
           ▼
    ┌─────────────┐
    │ Embedding   │  ← Token Embedding + Position Embedding
    └──────┬──────┘
           │ [seq_len, d_model]
           ▼
    ┌─────────────────────────────────────────┐
    │          Transformer Block × N          │
    │  ┌─────────────────────────────────┐    │
    │  │     Multi-Head Attention        │    │
    │  │     + Add & Norm (残差+LayerNorm)│    │
    │  ├─────────────────────────────────┤    │
    │  │     Feed-Forward Network (FFN)  │    │
    │  │     + Add & Norm               │    │
    │  └─────────────────────────────────┘    │
    └─────────────────────────────────────────┘
           │
           ▼
    ┌─────────────┐
    │ LM Head     │  ← 线性层 + Softmax → 词表概率分布
    └─────────────┘
```

> [!tip] 理解提示（非原文）
> 面试中最常问的就是这张图。不要死记硬背，要从"为什么需要它"的角度理解每一个组件。

### 1.2 自注意力机制（Self-Attention）—— 核心中的核心

**面试必问：请解释 Self-Attention 的计算过程**

直觉理解：每个词都要"看一眼"句子中的所有其他词，决定该关注谁。

$$\text{Attention}(Q, K, V) = \text{softmax}\left(\frac{QK^T}{\sqrt{d_k}}\right)V$$

逐步拆解：
1. **Q（Query）**：当前词发出的"查询"——"我在找什么信息？"
2. **K（Key）**：每个词提供的"键"——"我有什么信息"
3. **V（Value）**：每个词提供的"值"——"我的具体内容"
4. **$QK^T$**：计算当前词与所有词的相似度（注意力分数）
5. **$/\sqrt{d_k}$**：缩放——防止点积值过大导致 softmax 梯度消失
6. **softmax**：归一化为概率分布（所有注意力权重之和为 1）
7. **×V**：按注意力权重加权求和，得到当前词的新表示

> [!hint]- 面试场景：为什么要除以 $\sqrt{d_k}$？
> 当 $d_k$ 很大时（如 64、128），点积结果会很大，softmax 输出接近 one-hot（某个值接近1，其余接近0），梯度趋近于 0，训练不稳定。除以 $\sqrt{d_k}$ 将方差控制回 1，保持梯度健康流动。

### 1.3 Multi-Head Attention（MHA）

```
输入 X [seq_len, d_model]
         │
    ┌────┼────┐
    ▼    ▼    ▼
  Head1 Head2 ... Head_h     ← h 个头，每个维度 d_k = d_model / h
    │    │         │
    ▼    ▼         ▼
  Attn  Attn ... Attn        ← 每个头独立计算 Attention
    │    │         │
    └────┼────┘    │
         ▼         │
       Concat      │
         │         │
         ▼         │
    Linear(W_o) ←──┘        ← 输出投影矩阵
         │
         ▼
    输出 [seq_len, d_model]
```

**多头的意义**：不同头关注不同类型的关系——有的关注语法关系，有的关注语义关系，有的关注远距离依赖。

### 1.4 注意力变体（面试高频）

| 变体 | Q/K/V 共享方式 | 优点 | 缺点 | 代表模型 |
|------|--------------|------|------|---------|
| **MHA** (Multi-Head) | 每个头独立 Q/K/V | 表达力最强 | KV Cache 大 | GPT-3/4, 原始 Transformer |
| **MQA** (Multi-Query) | Q 多头，K/V 共享 1 组 | 推理快，KV Cache 小 | 表达力略降 | PaLM, StarCoder |
| **GQA** (Grouped-Query) | Q 分组，每组共享 K/V | 性能与速度平衡 | - | Llama-2/3, Mistral |
| **MLA** (Multi-Head Latent) | 低秩压缩 KV | KV Cache 极小 | 实现复杂 | DeepSeek-V2/V3 |

> [!warning] 易混点（面试陷阱）
> MQA ≠ KV 共享给所有头。MQA 是所有头共享**同一组** K 和 V 投影，但 Q 仍然是多头的。GQA 是折中——将 Q 分成若干组，每组共享一组 K/V。

**面试场景题**：

> [!success]- 场景：你的 LLM 推理服务在处理长文本时显存不够，如何优化？
> 1. 首选 GQA/MQA 架构——减少 KV Cache 显存占用（可能节省 50-70%）
> 2. 启用 KV Cache 量化（FP16 → INT8/INT4）
> 3. 使用 Flash Attention 减少 HBM 访问
> 4. 采用 PagedAttention（vLLM）管理 KV Cache 内存碎片
> 5. Sliding Window Attention（如 Mistral）限制注意力范围

### 1.5 位置编码

**为什么需要位置编码？** Self-Attention 是置换不变的（permutation invariant），打乱词序结果不变——这显然不对。

| 编码方式 | 原理 | 优点 | 缺点 |
|---------|------|------|------|
| 正弦位置编码 | 固定三角函数 | 可泛化到更长序列 | 外推能力有限 |
| 可学习位置编码 | 随模型训练 | 灵活 | 无法外推 |
| RoPE（旋转位置编码） | 用旋转矩阵编码相对位置 | 天然编码相对位置，外推好 | 实现稍复杂 |
| ALiBi | 直接在注意力分数加线性偏置 | 简单，外推好 | 表达力略弱 |

**RoPE 的直觉**：把位置信息融入 Q 和 K 的计算，使得 $q_m \cdot k_n$ 的结果自然包含 $m-n$ 的相对位置信息。Llama/GLM/Qwen 等主流模型都用 RoPE。

## 二、LLM 训练三阶段

### 2.1 阶段概览

```
阶段一：预训练（Pre-training）
  目标：学会语言的统计规律
  数据：万亿 token（网页/书籍/代码）
  方法：Next Token Prediction（自回归）
  产出：Base Model（如 Llama-3-8B-base）
         │
         ▼
阶段二：有监督微调（SFT, Supervised Fine-Tuning）
  目标：学会按指令回答
  数据：数万条高质量指令-回答对
  方法：继续自回归训练
  产出：Chat Model（如 Llama-3-8B-instruct）
         │
         ▼
阶段三：对齐训练（Alignment）
  目标：让回答更安全、更有用、更符合人类偏好
  方法：RLHF / DPO / RLAIF
  产出：最终发布的模型
```

### 2.2 预训练（Pre-training）

**核心任务：Next Token Prediction**

给定前文 $x_1, x_2, ..., x_{t-1}$，预测 $x_t$：

$$L = -\sum_{t=1}^{T} \log P(x_t | x_1, ..., x_{t-1}; \theta)$$

**面试场景题**：

> [!hint]- 面试官问：预训练的数据质量有多重要？数据配比怎么设计？
> **数据质量决定上限**。关键策略：
> 1. **去重**：MinHash/LSH 去重，避免模型记忆重复内容
> 2. **质量过滤**：用小模型打分，过滤低质量内容
> 3. **数据配比**：Llama-3 的配比参考——网页 50%、代码 20%、科学 15%、书籍 15%
> 4. **课程学习**：先学高质量数据，再混入更多来源
> 5. **多语言**：中文 LLM 需要大量中文数据（Wudao/SkyPile 等）

### 2.3 SFT（Supervised Fine-Tuning）

**关键：高质量数据 >> 大量数据**

```python
# SFT 数据格式示例
{
    "instruction": "解释什么是机器学习",
    "input": "",
    "output": "机器学习是人工智能的一个分支..."
}
```

**LoRA（Low-Rank Adaptation）**—— 参数高效微调：

$$W' = W + \Delta W = W + BA$$

其中 $B \in \mathbb{R}^{d \times r}$, $A \in \mathbb{R}^{r \times d}$, $r \ll d$

- 原始参数 W 冻结不更新
- 只训练低秩矩阵 A 和 B
- 训练参数量减少到 0.1%-1%
- 推理时可合并：$W' = W + BA$，零额外推理成本

> [!hint]- 面试场景：LoRA 的 rank（r）怎么选？为什么不能太大？
> - r 通常取 8-64，取决于任务复杂度
> - r 太大 → 参数量接近全量微调，容易过拟合（尤其是数据量小时）
> - r 太小 → 表达力不够，无法学好目标任务
> - 实践建议：从 r=16 开始，观察 loss 曲线，逐步调整
> - 多个 LoRA adapter 可以共享同一个 base model，方便切换

### 2.4 RLHF / DPO 对齐训练

**RLHF（Reinforcement Learning from Human Feedback）流程**：

```
1. 收集人类偏好数据
   prompt: "写一首关于春天的诗"
   回答A: [好的回答] ← 人类标注：更好
   回答B: [差的回答]

2. 训练 Reward Model（奖励模型）
   学习人类偏好：好的回答 → 高分，差的回答 → 低分

3. PPO 强化学习
   用 Reward Model 的得分作为奖励信号
   让模型学习生成高分回答
   加入 KL 散度惩罚，防止偏离 SFT 模型太远
```

**DPO（Direct Preference Optimization）**——RLHF 的简化替代：

$$L_{DPO} = -\mathbb{E}\left[\log \sigma\left(\beta \log \frac{\pi_\theta(y_w|x)}{\pi_{ref}(y_w|x)} - \beta \log \frac{\pi_\theta(y_l|x)}{\pi_{ref}(y_l|x)}\right)\right]$$

- **不需要 Reward Model**，直接从偏好数据优化策略模型
- 训练更简单、更稳定
- 效果接近甚至超过 RLHF
- DeepSeek/Qwen 等最新模型都使用 DPO 或其变体

> [!success]- 面试场景：RLHF vs DPO，怎么选？
> **DPO 适用场景**：
> - 计算资源有限（不需要训练额外的 RM）
> - 数据量适中（10K-100K 偏好对）
> - 对齐目标相对简单
>
> **RLHF 适用场景**：
> - 需要更精细的奖励信号（如多维度评分）
> - 在线学习（模型不断生成→人类标注→更新）
> - 需要处理复杂的对抗性场景
>
> **趋势**：工业界越来越多地采用 DPO + 在线变体（如 online DPO），兼顾简洁性和效果。

## 三、推理优化——工程面试重点

### 3.1 KV Cache

**核心思想**：生成第 t 个 token 时，前 t-1 个 token 的 K/V 已经计算过，缓存下来复用。

```
无 KV Cache:
  生成 token1: 计算 K1,V1
  生成 token2: 重新计算 K1,V1,K2,V2  ← 重复计算！
  生成 token3: 重新计算 K1,V1,K2,V2,K3,V3  ← 更浪费！

有 KV Cache:
  生成 token1: 计算 K1,V1 → 缓存
  生成 token2: 只计算 K2,V2 → 与缓存拼接  ← 省了！
  生成 token3: 只计算 K3,V3 → 与缓存拼接  ← 更省！
```

**KV Cache 显存计算**：

$$\text{KV Cache} = 2 \times n_{layers} \times n_{heads} \times d_{head} \times seq\_len \times \text{dtype\_size}$$

例如 Llama-3-8B（FP16，seq_len=4096）：
- 2 × 32 × 32 × 128 × 4096 × 2 bytes ≈ **2 GB** 仅用于 KV Cache！

> [!hint]- 面试场景：用户反馈"长文本生成很慢"，你怎么排查？
> 1. **检查 KV Cache 显存**：长序列 KV Cache 可能 OOM
> 2. **检查 batch 效率**：是否有足够请求填满 batch？
> 3. **量化推理**：FP16 → INT8/INT4 减少显存和计算
> 4. **Flash Attention**：减少 HBM 读写次数
> 5. **Speculative Decoding**：用小模型猜多个 token，大模型并行验证
> 6. **Prefix Caching**：相同前缀的请求复用 KV Cache

### 3.2 量化（Quantization）

| 方法 | 精度 | 显存节省 | 精度损失 | 速度 |
|------|------|---------|---------|------|
| FP16 | 16bit | 基准 | 无 | 基准 |
| BF16 | 16bit | 基准 | 极小 | 基准 |
| INT8 (PTQ) | 8bit | ~50% | 小 | 快 |
| INT4 (GPTQ/AWQ) | 4bit | ~75% | 中 | 最快 |
| GGUF (llama.cpp) | 2-8bit | 可调 | 可调 | CPU友好 |

**PTQ（Post-Training Quantization）**——训后量化：
- GPTQ：基于近似二阶信息的逐层量化
- AWQ：基于激活感知的权重量化（保护重要权重）
- SmoothQuant：将激活的难度迁移到权重

### 3.3 Flash Attention

**核心思想**：利用 GPU SRAM（快速缓存）减少对 HBM（高带宽内存）的访问次数。

```
标准 Attention:
  Q,K,V 在 HBM → 计算 QK^T 写回 HBM → softmax 读回 → 写回 HBM → 乘V → 写回
  HBM 访问次数: O(N²d)，N为序列长度

Flash Attention:
  分块计算（Tiling），在 SRAM 中完成 softmax + 乘V
  HBM 访问次数: O(N²d²/M)，M 为 SRAM 大小
  实际加速: 2-4x，且数值精确（非近似）
```

### 3.4 Speculative Decoding（投机解码）

```
                Draft Model（小模型）
                快速生成 K 个候选 token
                         │
                         ▼
            ┌────────────────────────┐
            │   Target Model（大模型）│
            │   并行验证 K 个 token   │
            └─────┬──────────────────┘
                  │
          ┌───────┼───────┐
          ▼       ▼       ▼
        接受 n 个 token，拒绝后续
        （n 可能为 0 到 K）
```

- 小模型（如 7B）猜，大模型（如 70B）验证
- 接受的 token 不需要重新计算
- 典型加速 2-3x，输出分布不变（精确等价）

## 四、主流 LLM 架构对比

| 模型 | 参数量 | 注意力 | 位置编码 | 特点 |
|------|--------|--------|---------|------|
| GPT-4 | ~1.8T (MoE) | GQA | RoPE | MoE 架构，多专家路由 |
| Llama-3 | 8B/70B/405B | GQA | RoPE | 开源标杆，高质量数据 |
| GLM-4 | 9B | GQA | RoPE | 中英双语优秀 |
| DeepSeek-V3 | 671B (MoE) | MLA | RoPE | 极低 KV Cache，性价比高 |
| Qwen-2.5 | 7B/72B | GQA | RoPE | 阿里开源，中文强 |
| Mistral | 7B/8x22B | GQA | RoPE | Sliding Window Attention |

> [!tip] 面试加分项
> 能说出这些模型之间的**关键差异**（如 MLA vs GQA、MoE 路由方式、训练数据策略），而不是笼统地"它们都是 Transformer"。

## 五、面试高频问题速查

> [!success]- Q1：Transformer 为什么比 RNN 好？
> 1. **并行计算**：RNN 必须顺序处理，Transformer 所有位置同时计算
> 2. **长距离依赖**：RNN 梯度消失/爆炸，Transformer 自注意力直接建模任意距离的关系
> 3. **可扩展性**：Transformer 可以轻松扩展到千亿参数
> 4. 但 Transformer 的注意力的计算复杂度是 O(N²)，长序列有瓶颈

> [!success]- Q2：Decoder-Only vs Encoder-Only vs Encoder-Decoder 怎么选？
> | 架构 | 代表 | 适用场景 |
> |------|------|---------|
> | Decoder-Only | GPT/Llama | 生成任务（对话、写作、代码） |
> | Encoder-Only | BERT | 理解任务（分类、NER、检索） |
> | Encoder-Decoder | T5/BART | 翻译、摘要 |
> 当前趋势：**Decoder-Only 一统天下**，GPT/Llama/GLM/Qwen 都是 Decoder-Only

> [!success]- Q3：什么是 MoE（Mixture of Experts）？为什么用 MoE？
> - 将 FFN 层替换为多个"专家"网络
> - 每个 token 只激活部分专家（如 8 选 2），计算量大幅减少
> - 671B 的 DeepSeek-V3 实际每次只激活 ~37B 参数
> - 用更少的算力获得更强的模型能力
> - 挑战：负载均衡（防止某些专家过载/空闲）、路由策略设计

## 关键要点回顾

1. **Transformer = 自注意力 + FFN + 残差 + LayerNorm**，逐层理解是面试基础
2. **注意力变体 MHA→MQA→GQA→MLA** 是推理优化的核心，理解取舍
3. **训练三阶段**：预训练（学语言）→ SFT（学指令）→ RLHF/DPO（学偏好）
4. **推理优化三板斧**：KV Cache（缓存）+ 量化（减精度）+ Flash Attention（减IO）
5. **LoRA 是参数高效微调的标配**，理解低秩分解的原理

## 深度扩展阅读

本篇为基础概览，以下三篇深度扩展笔记覆盖了面试所需的全部细节：

| 扩展笔记 | 核心内容 | 建议时间 |
|----------|---------|---------|
| [[raw/lessons/AI-Interview-Prep/01a-transformer-deep\|01a Transformer 核心原理深度]] | Self-Attention 完整数学推导、FFN/SwiGLU、LayerNorm/RMSNorm/Pre-Norm vs Post-Norm、RoPE 完整推导与外推方案、注意力变体 MHA/MQA/GQA/MLA 对比、采样策略数学公式 | 2h |
| [[raw/lessons/AI-Interview-Prep/01b-training-pipeline\|01b 训练流程深入]] | 预训练数据工程（MinHash+LSH去重）、SFT（Self-Instruct/Evol-Instruct）、LoRA 完整数学原理（内在维度假说）、QLoRA、RLHF（Bradley-Terry/PPO）、DPO 完整数学推导、数据污染检测 | 2h |
| [[raw/lessons/AI-Interview-Prep/01c-inference-optim\|01c 推理优化与部署]] | KV Cache 精确计算、GQA/MQA 显存节省分析、量化完整链路（GPTQ/AWQ/SmoothQuant）、Flash Attention Tiling 策略、Speculative Decoding 等价性证明、vLLM/TensorRT-LLM 框架对比 | 2h |

## 参考资源

- [The Illustrated Transformer](https://jalammar.github.io/illustrated-transformer/) — Jay Alammar
- [Let's build GPT from scratch](https://www.youtube.com/watch?v=kCc8FmEb1nY) — Andrej Karpathy
- Llama 3 技术报告 / DeepSeek-V3 技术报告

## 导航

← [[raw/lessons/AI-Interview-Prep/00-overview|总览页]]
→ [[raw/lessons/AI-Interview-Prep/02-ai-agent|第2站：AI Agent 系统]]
→ [[raw/lessons/AI-Interview-Prep/03-rag|第3站：RAG 检索增强生成]]
