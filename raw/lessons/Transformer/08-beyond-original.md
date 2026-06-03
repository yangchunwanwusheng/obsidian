---
type: lesson
tags:
  - 深度学习
  - Transformer
  - BERT
  - GPT
  - Flash Attention
  - MoE
  - 现代变体
created: 2026-04-09
updated: 2026-04-09
topic: 超越原始 Transformer——BERT、GPT 与现代变体
difficulty: advanced
prerequisites:
  - "[[06-encoder-decoder|编码器-解码器架构]]"
status: completed
series:
  name: Transformer 深度学习
  part: 8
---

# 超越原始 Transformer：BERT、GPT 与现代变体

> 当 2017 年 Transformer 论文发表时，它是一个优雅的通用架构。此后的七年里，研究者们在这个框架上进行了大量创新，发展出encoder-only、decoder-only、encoder-decoder等多种变体，每种都针对不同的任务场景优化。本章将系统梳理这些架构演化的脉络，帮助你在实际项目中做出正确的技术选型。

## 学习目标

- [ ] 理解三种架构变体的设计哲学与适用场景
- [ ] 掌握 BERT 的 MLM 预训练与双向注意力的核心思想
- [ ] 了解 GPT 系列从 1.17 亿到千亿参数的规模演进
- [ ] 熟悉 Flash Attention、MoE 等效率优化技术
- [ ] 能够根据任务类型选择合适的架构

## 前置知识

| 知识点 | 掌握程度 | 参考来源 |
|--------|----------|----------|
| 自注意力机制 | 理解 Q/K/V 计算过程 | [[wiki/lessons/Transformer/04-attention|注意力机制]] |
| 编码器-解码器架构 | 能画出完整结构图 | [[wiki/lessons/Transformer/06-encoder-decoder|编码器-解码器架构]] |
| 预训练-微调范式 | 了解迁移学习基本思想 | [[wiki/lessons/Transformer/07-training-inference|训练与推理]] |

## 1. 三种架构变体：设计与哲学

2017 年的原始 Transformer 同时包含编码器和解码器，但研究者很快发现：**不是所有任务都需要完整的架构**。根据任务特性，产生了三种主流变体：

### 1.1 架构对比

| 特性 | Encoder-Only (BERT) | Decoder-Only (GPT) | Encoder-Decoder (T5) |
|------|---------------------|--------------------|--------------------|
| **注意力范围** | 双向（可见全部上下文） | 单向（仅可见左侧） | 编码器双向，解码器单向 |
| **典型应用** | 理解任务：分类、NER、问答 | 自回归生成：续写、对话、代码 | seq2seq：翻译、摘要、生成 |
| **代表模型** | BERT, RoBERTa, DeBERTa | GPT-2/3/4, LLaMA, Claude | T5, BART, UL2 |
| **预训练任务** | MLM + NSP | 下一个token预测 | 跨度破坏（Span Corruption） |
| **生成方式** | 非自回归（一次输出） | 自回归（逐token生成） | 自回归（编码后解码） |
| **典型参数规模** | 1亿~3亿 | 70亿~1.8万亿 | 1亿~110亿 |

### 1.2 直观理解：三种架构的性格比喻

> **Encoder-Only (BERT)** 像一个**阅读理解高手**。当你给它一段文字时，它能同时看到全文，理解上下文关系，然后回答问题或做分类。它不会生成新文字，专注于理解。

> **Decoder-Only (GPT)** 像一个**作家**。它只能从左到右写作，每写一个字都要回顾之前的内容。这种"单向"特性使得它天然适合生成任务——作家也是在已有文字基础上续写。

> **Encoder-Decoder (T5)** 像一个**翻译专家**。先完整阅读原文（编码器），理解后再用目标语言表达（解码器）。它既有理解能力，又有生成能力。

### 1.3 图示：三种架构结构

```
Encoder-Only (BERT):
┌─────────────────────────────────────┐
│  Input: "The cat sat on the mat"   │
│         ↓                           │
│  ┌─────────────────────────┐         │
│  │  Shared Embedding       │         │
│  └─────────────────────────┘         │
│         ↓                           │
│  ┌─────────────────────────┐         │
│  │  N × Encoder Block      │ ← 双向注意力
│  │  (self-attention all)  │         │
│  └─────────────────────────┘         │
│         ↓                           │
│  ┌─────────────────────────┐         │
│  │  [CLS] token output     │ → 分类/序列标注
│  └─────────────────────────┘         │
└─────────────────────────────────────┘

Decoder-Only (GPT):
┌─────────────────────────────────────┐
│  Input: "The cat sat on"           │
│         ↓                           │
│  ┌─────────────────────────┐         │
│  │  Shared Embedding       │         │
│  └─────────────────────────┘         │
│         ↓                           │
│  ┌─────────────────────────┐         │
│  │  N × Decoder Block      │ ← 单向注意力（Masked）
│  │  (causal self-attention)│         │
│  └─────────────────────────┘         │
│         ↓                           │
│  ┌─────────────────────────┐         │
│  │  Language Modeling Head  │ → 预测下一个token
│  └─────────────────────────┘         │
│         ↓                           │
│  Output: "the" (概率分布)           │
└─────────────────────────────────────┘

Encoder-Decoder (T5):
┌─────────────────────────────────────────────────┐
│  Input: "Hello world" (源语言)                  │
│         ↓                                       │
│  ┌─────────────────────────┐                     │
│  │  Encoder                │ ← 双向注意力       │
│  │  (full context)         │                     │
│  └─────────────────────────┘                     │
│         ↓ (encoded representation)               │
│  ┌─────────────────────────┐                     │
│  │  Cross Attention         │ ← 看编码器输出     │
│  │  + Decoder Blocks        │ ← 单向注意力       │
│  └─────────────────────────┘                     │
│         ↓                                       │
│  Output: "你好世界" (目标语言)                   │
└─────────────────────────────────────────────────┘
```

## 2. BERT：双向理解的革命

### 2.1 BERT 的核心创新

2018 年，Google 发表了 BERT（Bidirectional Encoder Representations from Transformers），它的核心贡献是**首次在大规模上实现了真正的双向预训练**。

#### 直觉理解：BERT 的"完形填空"预训练

想象你学习英语时做的完形填空题：给你一句缺了几个词的句子，让你去猜空格里的词。BERT 的预训练就是这样——随机遮住 15% 的 token，然后让模型去预测被遮住的是什么。

```
原始句子: "The cat [MASK] on the mat"
                ↑
           被遮住的词

模型需要理解: "The cat ___ on the mat"
             → 填 "sat"
```

这种任务叫做 **Masked Language Model (MLM)**，它强迫模型同时学习左 context 和 right context 的信息——这才是真正的"双向"理解。

### 2.2 BERT 的两种预训练任务

#### Masked Language Model (MLM)

> **技术细节**：随机选择 15% 的 token 进行替换——其中 80% 替换为 `[MASK]`，10% 替换为随机 token，10% 保持不变。这样做是为了**让模型不仅记住 [MASK] token 的含义，还要学习处理"未见过的词"的能力**。

```
原句: "我的猫喜欢喝牛奶"

80% 情况: "我的猫喜欢喝 [MASK]"     → 预测 "牛奶"
10% 情况: "我的猫喜欢喝 河水"       → 预测 "牛奶"（噪声）
10% 情况: "我的猫喜欢喝 牛奶"       → 预测 "牛奶"（无变化）
```

#### Next Sentence Prediction (NSP)

> **设计动机**：很多任务需要理解句子之间的关系，比如问答和自然语言推理。NSP 训练模型判断两个句子是否为连续文本。

```
句子A: "小狗在公园里玩耍"
句子B: "它非常开心"

标签: IsNext（连续句子）

---

句子A: "小狗在公园里玩耍"
句子B: "今天天气很晴朗"

标签: NotNext（非连续）
```

> [!note]
> 后续研究发现 NSP 的作用有限，RoBERTa 等改进模型直接去掉了 NSP，改为更长的序列训练。

### 2.3 BERT 的应用场景

```python
# BERT 典型应用示例：文本分类
from transformers import BertTokenizer, BertForSequenceClassification
import torch

tokenizer = BertTokenizer.from_pretrained('bert-base-chinese')
model = BertForSequenceClassification.from_pretrained('bert-base-chinese', num_labels=2)

# 输入: "这个电影太好看了！"
# 输出: 正面/负面分类
inputs = tokenizer("这个电影太好看了！", return_tensors="pt")
logits = model(**inputs).logits
predicted_class = torch.argmax(logits, dim=-1).item()  # 1 (正面)
```

| 任务类型 | BERT 输出层 | 典型应用 |
|----------|-------------|----------|
| 序列标注 | 每个位置的隐藏状态 | 命名实体识别（NER）、词性标注 |
| 句子对分类 | `[CLS]` token | 自然语言推理、语义相似度 |
| 问答 | start/end positions | SQuAD 问答 |
| 单句分类 | `[CLS]` token | 情感分析、垃圾邮件检测 |

## 3. GPT 系列：规模的力量

### 3.1 GPT 的进化路径

GPT（Generative Pre-Trained Transformer）系列展示了**规模法则（Scaling Law）**的惊人力量：

| 版本 | 参数规模 | 关键创新 | 关键能力 |
|------|----------|----------|----------|
| GPT-1 | 1.17 亿 | 首个"预训练+微调"范式 | 简单生成 |
| GPT-2 | 15 亿 | 更大的训练数据、多任务零样本 | 零样本任务迁移 |
| GPT-3 | 1750 亿 | 涌现 In-Context Learning | 无需微调，直接提示 |
| GPT-4 | ~1.8 万亿（推测） | 多模态、复杂推理 | 复杂推理、指令遵循 |
| GPT-4o | - | 原生多模态 | 实时语音/视频交互 |

### 3.2 规模效应：从量变到质变

> **直觉理解**：把模型参数想象成大脑神经元。1 亿参数像老鼠的大脑，能完成简单任务；1750 亿参数像人类大脑，涌现出之前没有的推理能力。

#### 三个关键能力的涌现

```
参数规模增长 →
                    ┌─ 简单分类
                    │
  1亿参数 ──────────┤
  (GPT-1)           │  基础文本补全
                    │
                    └─ 简单问答
                           │
                           │  零样本学习
  15亿参数 ────────────────┤
  (GPT-2)                  │  多任务零样本
                           │
                           └─ 基础推理
                                  │
                                  │  In-Context Learning
  1750亿参数 ─────────────────────┤
  (GPT-3)                        │
                                  │  复杂推理、代码生成
                                  │  思维链 (Chain-of-Thought)
                                  │
                                         ┌─ 原生多模态
  GPT-4 ────────────────────────────────┤
                                         │  复杂逻辑推理
                                         │
                                         └─ 长程依赖
```

### 3.3 In-Context Learning：无需梯度的学习

**In-Context Learning (ICL)** 是 GPT-3 最重要的发现——模型能够在**不更新参数的情况下**，通过 few-shot 示例学习新任务。

```
Prompt:
"翻译成法语:
英语: Hello
法语: Bonjour

英语: Good morning
法语: Bonjour

英语: Thank you
法语:"
       ↑
       模型直接"推理"出答案，而不需要任何参数更新
```

> **技术解释**：ICL 的本质是让模型在推理时利用 Transformer 的注意力机制，**在上下文中找到"任务描述"和"示例"之间的对应关系**。这不同于传统微调——没有权重更新，没有梯度，但模型确实"学习"了。

> [!warning]- 常见误区
> **ICL 不是真正的学习**：模型没有更新权重，知识完全来自预训练。ICL 更像是一种"检索"——从模型参数中提取与当前任务相关的信息。

### 3.4 GPT 架构的技术细节

```python
# GPT 风格的自回归生成（伪代码）
def generate(model, tokenizer, prompt, max_new_tokens=50):
    """
    核心机制：每一步只预测下一个token，然后将其加入输入继续生成
    """
    input_ids = tokenizer.encode(prompt, return_tensors="pt")

    for _ in range(max_new_tokens):
        # 前向传播得到下一个token的logits
        outputs = model(input_ids)
        next_token_logits = outputs.logits[:, -1, :]  # 只取最后一个位置

        # 采样（temperature > 0 时加入随机性）
        probs = torch.softmax(next_token_logits / temperature, dim=-1)
        next_token = torch.multinomial(probs, num_samples=1)

        # 拼接，继续
        input_ids = torch.cat([input_ids, next_token], dim=-1)

    return tokenizer.decode(input_ids[0])
```

**GPT 的关键架构选择**：
- **Causal (单向) Masked Self-Attention**：每个 token 只能看到自己和之前的 token，保证生成的自回归性
- **没有编码器**：decoder-only 架构更简单，训练更稳定
- **Weight Tying**：embedding 层和输出层共享权重，节省参数量

## 4. 效率优化：让 Transformer 更快更强

### 4.1 稀疏注意力：打破 O(n²) 困局

标准自注意力的计算复杂度是 $O(n^2)$，其中 n 是序列长度。这意味着长文本会带来平方级增长的计算和内存成本。

#### Longformer：局部 + 全局的稀疏模式

> **核心思想**：不是每个 token 都要 attend 到所有其他 token。

```
标准注意力（全部 attend）:
Token 1: attend to [1, 2, 3, 4, 5, ..., n]
Token 2: attend to [1, 2, 3, 4, 5, ..., n]
...
Token n: attend to [1, 2, 3, 4, 5, ..., n]
         ↑
         O(n²) 复杂度

Longformer（局部窗口 + 少量全局）:
Token 1: attend to [1, 2, 3, 4, 5] + 全局位置
Token 2: attend to [1, 2, 3, 4, 5, 6] + 全局位置
...
         ↑
         O(n) 或 O(n log n) 复杂度
```

| 模型 | 注意力模式 | 复杂度 | 最大长度 |
|------|-------------|--------|----------|
| Standard Transformer | Full O(n²) | $O(n^2)$ | 512 |
| Longformer | Sliding + Global | $O(n)$ | 32,768 |
| BigBird | Sliding + Global + Random | $O(n)$ | 4,096 |

### 4.2 Flash Attention：硬件感知的革命

Flash Attention 不是改变理论复杂度，而是**让 O(n²) 的计算在现代 GPU 上跑得更快**。

> **直觉理解**：标准 Attention 实现需要先将注意力矩阵写入 HBM（高带宽显存），计算后再读回。Flash Attention 的创新在于**使用tiling技术，让中间结果始终保留在 SRAM（快速缓存）中，避免反复读写 HBM**。

```
标准实现流程:
1. 计算 QK^T → 写入 HBM (慢)
2. 从 HBM 读回 → softmax
3. 写入 HBM (慢)
4. 读回 → 乘以 V
5. 写入 HBM (慢)

Flash Attention 流程:
1. 分块加载 Q, K, V 到 SRAM
2. 在 SRAM 内计算局部注意力
3. 通过online softmax归一化
4. 直接写出最终结果到 HBM
   （只需一次写入！）

收益: 速度提升 2-4×，内存减少 O(n)
```

**Flash Attention 2** 进一步优化：
- 更好的线程分配（warp 级别并行）
- 支持序列长度大于 SRAM 能容纳的大小（通过递归分块）
- 推理时也可加速（用于 KV Cache 优化）

### 4.3 混合专家：稀疏激活的大模型

**Mixture of Experts (MoE)** 让大模型在**不增加实际计算量**的情况下获得更大容量。

> **类比理解**：MoE 就像一个大型医院的专家会诊。有 8 位不同科室的专家（8 个 expert），但每次问诊只叫一位（稀疏激活）。病人（输入 token）根据症状（router）被分配到最相关的专家。

```
传统 Dense 模型:
┌──────────────────────┐
│  Input Token         │
└──────────┬───────────┘
           ↓
┌──────────────────────┐
│  FFN (所有参数激活)   │ ← 100% 激活
└──────────────────────┘
           ↓
┌──────────────────────┐
│  Output              │
└──────────────────────┘

MoE 模型:
┌──────────────────────┐
│  Input Token         │
└──────────┬───────────┘
           ↓
┌──────────────────────┐
│  Router (MLP)        │ → 决定激活哪个 expert
└──────────┬───────────┘
           ↓
    ┌──────┼──────┐
    ↓      ↓      ↓
┌──────┐ ┌──────┐ ┌──────┐
│ E1   │ │ E2   │ │ E8   │  ← 只有 1-2 个 expert 激活
└──────┘ └──────┘ └──────┘
    │      │      │
    └──────┼──────┘
           ↓
┌──────────────────────┐
│  Weighted Sum        │
└──────────────────────┘
```

| 特性 | Dense Transformer | MoE (如 Mixtral 8×7B) |
|------|------------------|----------------------|
| 总参数量 | 70B | 46.7B (8×7B experts) |
| 激活参数/token | 70B | ~12B (top-2 of 8) |
| 实际计算量 | 高 | 低（接近小模型） |
| 内存需求 | 高 | 中等（所有 expert 需加载） |
| 吞吐量 | 低 | 高（稀疏激活） |

**为什么 MoE 如此重要**：可以用 1/6 的计算量训练出接近同等能力的大模型。GPT-4 被推测可能使用了 MoE 架构。

## 5. Transformer 之外：Mamba 与状态空间模型

### 5.1 为什么需要挑战 Transformer

Transformer 的核心瓶颈在于**长序列的自注意力**。尽管有各种优化，理论上 O(n) 的复杂度仍然是一个目标。

**状态空间模型 (State Space Models, SSM)** 提供了一条不同的思路：用一个**递归的隐藏状态**来压缩序列信息。

> **直觉理解**：Transformer 像是在做"全局透视"——每个位置都能看到所有其他位置。Mamba 则像是一个"记忆管理者"——用一个固定大小的隐藏状态来总结过去的所有信息。

### 5.2 Mamba：选择性状态空间

Mamba (2023) 是 SSM 领域最重要的进展。它的核心创新是**让状态转移具有选择性**——模型可以决定关注哪些历史信息。

```
传统 SSM (如 S4):
- 状态转移矩阵 A 是固定的
- 对所有 token 一视同仁
- 无法高效选择相关信息

Mamba (选择性 SSM):
┌─────────────────────────────────────┐
│  输入: x[t]                          │
│         ↓                           │
│  ┌─────────────────────────────┐     │
│  │  选择性扫描 (Selective Scan) │    │
│  │  - 决定跳过/保留哪些历史信息  │    │
│  │  - 动态调整 B, C, Δ 参数    │    │
│  └─────────────────────────────┘     │
│         ↓                           │
│  隐藏状态 h[t] = A·h[t-1] + B·x[t]  │
│         ↓                           │
│  输出 y[t] = C·h[t]                 │
└─────────────────────────────────────┘
```

| 特性 | Transformer | Mamba |
|------|-------------|-------|
| 复杂度（序列） | O(n²) | **O(n)** |
| 内存（序列） | O(n²) | **O(n)** |
| 并行训练 | 高效 | 需特殊并行策略 |
| 推理速度 | 慢（KV Cache） | **极快（递归）** |
| 长依赖建模 | 强 | 中等（受状态维度限制） |
| 实际效果 | SOTA | 有竞争力（尤其长序列） |

> [!note]
> Mamba 在基因序列、音频等长序列任务上表现优异，但在需要真正全局注意力的 NLP 任务上，Transformer 仍是主流。

## 6. 架构选型指南

### 决策流程图

```
你的任务是什么？
        │
        ▼
┌───────────────────┐
│ 需要生成新文本吗？  │
└────────┬──────────┘
         │
    Yes ─┴─ No
     │       │
     ▼       ▼
┌────────┐ ┌─────────────────┐
│ 生成类  │ │ 理解/分类类      │
└───┬────┘ └────────┬────────┘
    │               │
    ▼               ▼
┌───────────┐  ┌──────────────┐
│ 需要原文   │  │ 只需要理解？   │
│ 作为参考？ │  └────┬─────────┘
└───┬───────┘       │
    │         Yes ─┴─ No
    │          │       │
    ▼          ▼       ▼
┌────────┐ ┌──────┐ ┌──────────┐
│Encoder-│ │BERT  │ │ 序列标注  │
│Decoder │ │系    │ │(NER等)   │
│(T5等)  │ │      │ │          │
└────────┘ └──────┘ └──────────┘
    │          │
    ▼          ▼
┌────────┐ ┌──────────┐
│翻译/摘要│ │BERT系    │
│/改写   │ │(BERT/    │
│        │ │RoBERTa) │
└────────┘ └──────────┘
```

### 场景推荐表

| 场景 | 推荐架构 | 代表模型 | 原因 |
|------|----------|----------|------|
| 情感分类 | Encoder-Only | RoBERTa, DeBERTa | 快速、准确、需微调 |
| 命名实体识别 | Encoder-Only | BERT-Chinese | 细粒度序列标注 |
| 机器翻译 | Encoder-Decoder | T5, NLLB | 需要源上下文理解 |
| 文章摘要 | Encoder-Decoder | BART, Pegasus | 理解+生成 |
| 对话/续写 | Decoder-Only | GPT-4, LLaMA | 自回归生成 |
| 代码生成 | Decoder-Only | Codex, CodeGen | 强生成能力 |
| 长文档问答 | Encoder+稀疏注意力 | Longformer-BERT | O(n) 复杂度 |
| 实时语音流 | Decoder-Only | Whisper | 流式推理 |

### 2024-2026 年的格局变化

```
2020: BERT 统治 NLP 理解任务
2022: ChatGPT 爆火，Decoder-Only 开始流行
2023: LLaMA 开源，MoE 开始应用
2024: GPT-4o 原生多模态，Claude 强调安全
2025: 小型化 + 专用化（Phi, Mistral, Qwen）
2026: 多模态融合成为默认能力
```

## 7. 常见误区

| 误区 | 正确理解 |
|------|----------|
| "BERT 也可以生成文本" | BERT 预训练是 MLM，无法自回归生成。只能用于理解任务，生成需要额外训练（如 Seq2Seq BERT）。 |
| "模型越大效果越好" | 规模法则存在，但边际效益递减。同时要考虑推理成本、延迟、功耗。专用小模型有时优于通用大模型。 |
| "Attention 就是 O(n²) 无法优化" | Flash Attention 证明实现优化可以大幅降低实际开销。稀疏注意力可改变理论复杂度。 |
| "MoE 是一半参数一半计算" | MoE 参数量大，但稀疏激活使计算量接近小模型。内存占用仍是全部 expert 的权重。 |
| "Mamba 将取代 Transformer" | Mamba 在长序列上有优势，但在复杂推理任务上 Transformer 仍是主流。两者各有适用场景。 |

## 8. 思考题

> [!hint]- 基础题 1：架构选择
> 为什么问答系统（Question Answering）通常使用 Encoder-Only 架构（如 BERT）而不是 Decoder-Only（如 GPT）？
>
> *提示：考虑两种架构的输出形式和问答任务的特点。*

> [!hint]- 基础题 2：MLM vs CLM
> 解释 Masked Language Model（MLM）和 Causal Language Model（CLM）的区别。为什么 BERT 使用 MLM 而 GPT 使用 CLM？
>
> *提示：MLM 允许双向 attention，CLM 必须是单向。想想这与各自的使命（理解 vs 生成）有什么关系。*

> [!success]- 基础题 1 答案
> 问答系统需要从给定文本中**定位**答案的.span，然后输出。这是一种**序列标注**任务——给输入序列的每个 token 打标签（start/end 位置）。Encoder-Only 架构可以一次性看到完整上下文，每个位置都有对应的隐藏状态输出，非常适合这种任务。Decoder-Only 需要自回归生成答案，但问答不需要生成新文本，而是精准定位。
> [!success]- 基础题 2 答案
> **MLM（BERT）**：随机遮住 15% token，训练模型预测被遮住的词。遮住的 token 之间可以相互"看"到，因此是双向的。适合理解任务。
>
> **CLM（GPT）**：训练模型预测下一个 token。每个 token 只能看到左侧（之前的）token，因此是单向的。适合生成任务。
>
> BERT 的使命是"理解"，需要看到完整上下文；GPT 的使命是"生成"，必须从左到右写。架构与使命匹配。
>

> [!hint]- 进阶题：Scale Law 的局限性
> 如果 Scale Law（规模法则）继续有效，理论上 10 万亿参数的模型应该比现在的 GPT-4 更强。但实际上存在多个阻碍因素。请列举至少 3 个实际阻碍因素，并说明为什么仅仅增加参数不够。
>
> *提示：从(1) 数据质量 (2) 推理成本 (3) 训练稳定性 (4) 架构效率 等角度思考。*

## 9. 关键要点回顾

1. **三种架构变体**反映了对称性与生成性的权衡：Encoder-Only 放弃生成换来双向理解；Decoder-Only 拥抱单向实现自回归生成；Encoder-Decoder 保留完整能力但增加复杂度。

2. **BERT 的贡献**是证明了双向预训练的有效性。MLM 任务让模型同时理解左右上下文，这是理解任务的基础。

3. **GPT 系列验证了规模法则**：从 1 亿到千亿参数，模型能力出现"涌现"——In-Context Learning、复杂推理等能力在小模型上完全看不到。

4. **效率优化是让 Transformer 实用的关键**：Flash Attention 通过硬件感知优化打破内存瓶颈；MoE 通过稀疏激活在不增加计算量的情况下扩展容量。

5. **架构选择应基于任务**：理解任务用 Encoder-Only，生成任务用 Decoder-Only，需要同时理解源文本并生成新文本时用 Encoder-Decoder。

6. **Mamba 等新架构**在长序列任务上有潜力，但 Transformer 生态（工具、优化、社区）仍是主导力量。

## 10. 扩展阅读

### 论文

| 论文 | 年份 | 推荐理由 |
|------|------|----------|
| [BERT: Pre-training of Deep Bidirectional Transformers](https://arxiv.org/abs/1810.04805) | 2018 | BERT 原始论文，必读 |
| [Language Models are Few-Shot Learners (GPT-3)](https://arxiv.org/abs/2005.14165) | 2020 | ICL 的首次系统性验证 |
| [FlashAttention-2: Faster Attention with Better Parallelism](https://arxiv.org/abs/2307.08691) | 2023 | Flash Attention 核心改进 |
| [Mixtral 8x7B: Sparse Mixture of Experts](https://arxiv.org/abs/2401.04088) | 2024 | MoE 的优秀实践 |
| [Mamba: Linear-Time Sequence Modeling](https://arxiv.org/abs/2312.00752) | 2023 | 状态空间模型的重要突破 |

### 博客与教程

- [Jay Alammar: The Illustrated BERT](https://jalammar.github.io/illustrated-bert/) — BERT 的可视化解释
- [Lil'Log: Attention? Attention!](https://lilianweng.github.io/posts/2018-06-24-attention/) — 注意力机制的全面梳理
- [Karpathy: Let's Build GPT from Scratch](https://www.youtube.com/watch?v=kCc8FmEb1nY) — 从零实现 GPT 的视频教程

### 开源项目

- [Hugging Face Transformers](https://github.com/huggingface/transformers) — 所有主流模型的统一实现
- [Flash Attention](https://github.com/Dao-AILab/flash-attention) — 高效注意力实现
- [Mamba](https://github.com/state-spaces/mamba) — 状态空间模型实现

---

## 下一步学习

恭喜完成 Transformer 系列课程！以下是建议的后续学习方向：

- **[[wiki/lessons/Transformer/09-llm-pretraining|大型语言模型预训练]]** — 深入了解 GPT、LLaMA 等模型的预训练细节
- **[[wiki/concepts/RLHF|RLHF 与人类反馈强化学习]]** — 理解 ChatGPT 如何通过人类反馈微调
- **[[wiki/concepts/chain-of-thought|思维链 (Chain-of-Thought)]]** — 理解大模型推理能力的来源

---

*本笔记为 Transformer 系列最后一篇。全系列覆盖了从注意力机制到现代大模型的完整知识链路。*
