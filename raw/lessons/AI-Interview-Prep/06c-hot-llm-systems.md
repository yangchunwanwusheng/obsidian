---
type: lesson
tags: [面试, DeepSeek, MLA, MoE, SGLang, RadixAttention, 推理模型, FP8]
created: 2026-05-20
updated: 2026-05-20
difficulty: advanced
prerequisites: [Transformer架构, KV-Cache, Attention变体, 量化基础]
topic: 业界热点LLM架构
status: in-progress
series: {name: "AI Interview Prep", part: "06c"}
---

# 业界热点 LLM 架构深度分析——DeepSeek-V3 到 SGLang

> 掌握 DeepSeek-V3 的 MLA+MoE、SGLang 的 RadixAttention 等最新架构创新，是面试中技术深度的最佳证明。

## 学习目标

- [ ] 能深入分析 DeepSeek-V3 的 MLA、MoE、MTP 三大创新及数学原理
- [ ] 掌握 SGLang 的 RadixAttention 和推理框架选型逻辑
- [ ] 理解推理模型（o1/R1）的架构创新
- [ ] 能在面试中对比 MLA vs GQA、SGLang vs vLLM 等关键选择

## 一、为什么面试要考察 LLM 热点架构？

```
面试考察逻辑：

基础理论（Transformer、Attention）
    ↓ 知道"是什么"
工程优化（KV Cache、量化、Flash Attention）
    ↓ 知道"怎么优化"
热点架构（DeepSeek MLA、SGLang）
    ↓ 知道"业界最新在做什么"

→ 面试官通过热点架构考察你的技术视野和工程深度
→ "你了解 DeepSeek-V3 的 MLA 吗" ≈ "你关注最新技术发展吗"
```

## 二、DeepSeek-V3 架构深度分析

### 2.1 整体架构概览

```
┌─────────────────────────────────────────────────────────────┐
│                    DeepSeek-V3 架构                          │
│                                                             │
│  基本参数：                                                  │
│  ┌─────────────────────────────────────────────┐            │
│  │ 总参数：671B（6710亿）                       │            │
│  │ 激活参数：37B（每 token 激活 370亿）         │            │
│  │ 训练数据：14.8T tokens                      │            │
│  │ 训练成本：$5.5M（≈ 2048×H800 × 2个月）     │            │
│  │ 上下文长度：128K tokens                     │            │
│  └─────────────────────────────────────────────┘            │
│                                                             │
│  三大核心创新：                                              │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐     │
│  │  MLA         │  │  DeepSeekMoE │  │  MTP         │     │
│  │  多头潜在    │  │  细粒度 MoE  │  │  多 Token    │     │
│  │  注意力      │  │  256个专家    │  │  预测        │     │
│  └──────────────┘  └──────────────┘  └──────────────┘     │
│        ↓                  ↓                  ↓              │
│   KV Cache 压缩     计算效率提升      训练+推理加速         │
│                                                             │
│  训练效率创新：                                              │
│  ┌──────────┐  ┌──────────┐  ┌──────────────┐             │
│  │ FP8 混合 │  │ DualPipe │  │ 无辅助损失   │             │
│  │ 精度训练 │  │ 计算通信 │  │ 负载均衡     │             │
│  │          │  │ 重叠     │  │              │             │
│  └──────────┘  └──────────┘  └──────────────┘             │
└─────────────────────────────────────────────────────────────┘
```

### 2.2 MLA（Multi-Head Latent Attention）深度解析

#### 问题动机

```
标准 MHA 的 KV Cache 内存：

假设模型配置：
  - n_kv_heads = 128（KV 头数）
  - head_dim = 128（每个头的维度）
  - seq_len = 128K（上下文长度）
  - num_layers = 61（层数）
  - dtype = FP16（2 bytes）

每 token 的 KV Cache 大小：
  = 2（K+V）× n_kv_heads × head_dim × num_layers × 2 bytes
  = 2 × 128 × 128 × 61 × 2
  = 4,128,768 bytes ≈ 4 MB / token

128K 上下文的 KV Cache：
  = 4 MB × 128K = 512 GB ← 完全不可行！
```

#### 核心思想：将 KV 压缩到低维潜在空间

```
标准 MHA:
  输入 X → K = X·W_k [seq_len, n_kv_heads, head_dim]
         → V = X·W_v [seq_len, n_kv_heads, head_dim]
  缓存：K 和 V 完整存储

MLA:
  输入 X → c_kv = X·W_dkv [seq_len, kv_lora_rank]  ← 压缩！
         → K = c_kv·W_uk [seq_len, n_kv_heads, head_dim]  ← 上投影恢复
         → V = c_kv·W_uv [seq_len, n_kv_heads, head_dim]  ← 上投影恢复
  缓存：只存 c_kv ← 维度远小于原始 KV！
```

#### 数学推导

```python
# MLA 的核心数学

# 标准 MHA
# K = X · W_k,  shape: [batch, seq_len, n_kv_heads, head_dim]
# V = X · W_v,  shape: [batch, seq_len, n_kv_heads, head_dim]
# Cache 大小 = 2 × n_kv_heads × head_dim × num_layers × 2 bytes

# MLA
# Step 1: 压缩（训练和推理都做）
#   c_kv = X · W_dkv    shape: [batch, seq_len, kv_lora_rank]
#   其中 kv_lora_rank << n_kv_heads × head_dim

# Step 2: 上投影恢复（推理时在需要时才做）
#   K = c_kv · W_uk     shape: [batch, seq_len, n_kv_heads, head_dim]
#   V = c_kv · W_uv     shape: [batch, seq_len, n_kv_heads, head_dim]

# Step 3: 注意力计算（与标准 MHA 相同）
#   Attention(Q, K, V) = softmax(Q·K^T / √d) · V

# Cache 大小对比：
#   MHA:  2 × n_kv_heads × head_dim × L = 2 × 128 × 128 × 61 = 2,064,384
#   MLA:  kv_lora_rank × L = 512 × 61 = 31,232
#   压缩比：≈ 66x！

# 以 DeepSeek-V3 671B 为例：
#   kv_lora_rank = 512
#   Cache 大小: 2 × 512 × 61 × 2 bytes ≈ 125 KB / token
#   vs 标准 MHA ≈ 4 MB / token
```

#### MLA vs GQA vs MQA 对比

| 维度 | MHA | GQA | MQA | MLA |
|------|-----|-----|-----|-----|
| **KV 头数** | = Q 头数 | Q 头分组共享 | 1 个 KV 头 | 压缩到低维向量 |
| **压缩方式** | 无 | 头维度共享 | 头维度共享 | 学习到的低秩压缩 |
| **信息损失** | 无 | 头间信息共享 | 头间信息共享 | 可学习的最小损失 |
| **压缩比** | 1x | 4-8x | 128x | ~66x |
| **表达能力** | 最高 | 中 | 低 | 接近 MHA |
| **训练代价** | 低 | 低 | 低 | 额外投影矩阵 |
| **代表模型** | 原始 Transformer | Llama-3 | PaLM | DeepSeek-V2/V3 |

```
关键区别：MLA 是"学习到的压缩"，GQA/MQA 是"结构化共享"

GQA: 多个 Q 头共享同一个 KV 头
     → 信息在"头"维度被硬性共享
     → 损失：不同 Q 头无法关注不同的 K/V 子空间

MLA: KV 被压缩到低维，推理时再上投影恢复
     → 压缩是模型自己学到的（低秩近似）
     → 损失：只有"不能被低秩近似"的高频信息
     → 表达能力远优于 GQA
```

> [!hint]- 面试场景：MLA 和 GQA 的本质区别是什么？各有什么优缺点？
> **本质区别**：
> - GQA 是"结构化共享"（多个 Q 头硬性共享 KV 头）
> - MLA 是"学习到的压缩"（KV 被低秩压缩，推理时恢复）
>
> **MLA 优点**：
> 1. 表达能力接近完整 MHA（因为压缩是学到的最优低秩近似）
> 2. Cache 压缩比高（~66x）
> 3. 推理时可以按需上投影（灵活性高）
>
> **MLA 缺点**：
> 1. 训练时需要额外计算投影矩阵
> 2. 实现复杂度高于 GQA
> 3. 上投影计算增加推理延迟（虽然 cache 小了）
>
> **GQA 优点**：
> 1. 实现极其简单
> 2. 无额外训练代价
> 3. 兼容性好（大多数框架直接支持）
>
> **GQA 缺点**：
> 1. 头共享导致信息损失
> 2. 压缩比有限（通常 4-8x）

### 2.3 DeepSeekMoE 架构

```
DeepSeekMoE 的关键设计：

┌──────────────────────────────────────────────────┐
│              DeepSeekMoE 层结构                    │
│                                                  │
│  输入 Token X                                     │
│       │                                          │
│       ▼                                          │
│  ┌──────────────┐                                │
│  │  Router      │  计算 Token → Expert 的分配    │
│  │  (Gate)      │                                │
│  └──────┬───────┘                                │
│         │                                        │
│    Top-K=8 选择                                  │
│    （从 256 个路由专家中选 8 个）                  │
│         │                                        │
│  ┌──────┴──────────────────────────────┐         │
│  │                                     │         │
│  │  ┌─────────┐  ┌─────────┐          │         │
│  │  │ 共享专家 │  │ 路由专家 │ ×256    │         │
│  │  │ (1个)   │  │ (选8个) │          │         │
│  │  │         │  │         │          │         │
│  │  │ 始终    │  │ 按需    │          │         │
│  │  │ 激活    │  │ 激活    │          │         │
│  │  └────┬────┘  └────┬────┘          │         │
│  │       │             │               │         │
│  │       └──────┬──────┘               │         │
│  │              ▼                      │         │
│  │       加权求和输出                   │         │
│  └─────────────────────────────────────┘         │
│                                                  │
│  关键参数：                                       │
│  - 总路由专家：256 个（细粒度）                    │
│  - 共享专家：1 个                                 │
│  - Top-K：8（每个 token 选 8 个路由专家）          │
│  - 跨节点：最多路由到 4 个节点                     │
│  - 每个专家 FFN 隐藏层：1408 维                   │
└──────────────────────────────────────────────────┘
```

**为什么用 256 个细粒度专家而非 8 个大专家？**

```
Mixtral 方案：8 个大专家，Top-2
  → 每个专家容量大，但组合数少：C(8,2) = 28 种组合
  → 知识容易冗余（大专家内部功能重叠）

DeepSeek 方案：256 个小专家，Top-8
  → 每个专家容量小但专精，组合数极多：C(256,8) ≈ 4.6×10¹³
  → 知识更分散、更精确、更少冗余
  → 类比：256 个专科医生 vs 8 个全科医生

共享专家的作用：
  → 捕捉通用知识（语法、常见模式）
  → 路由专家不需要重复学习通用知识
  → 提高路由专家的"专业化"程度
```

#### 无辅助损失负载均衡

```
传统 MoE 问题：Router 可能只选少数几个专家（"赢者通吃"）
传统解决方案：加 auxiliary loss（辅助损失）惩罚不均衡
  → 但 auxiliary loss 会干扰主训练目标

DeepSeek 创新方案：无辅助损失的负载均衡
  核心思想：为每个专家维护一个可学习的 bias
  - Router 计算得分后，加上 expert bias
  - Bias 通过简单的动态调整实现均衡：
    - 被选太多的专家 → 降低 bias
    - 被选太少的专家 → 提高 bias
  - 不需要修改损失函数，不影响训练目标
```

### 2.4 MTP（Multi-Token Prediction）

```
传统：每次预测 1 个 token
  P(x_t | x_<t)

MTP：同时预测未来多个 token
  P(x_t, x_{t+1}, ..., x_{t+k} | x_<t)

训练好处：
  1. 前馈规划：模型被迫"想清楚"未来几步再说
  2. 更好的表征：学到的 hidden state 包含更多未来信息
  3. 训练效率：一次前向传播获得多个 token 的损失信号

推理好处：
  1. 可作为 Speculative Decoding 的 Draft Model
  2. 一次生成 k 个候选 token → 验证 → 接受或拒绝
  3. 大幅提升推理速度（无需额外训练 Draft Model）
```

### 2.5 训练效率创新

| 技术 | 作用 | 效果 |
|------|------|------|
| **FP8 混合精度** | 部分计算用 FP8，关键层保留 FP16/FP32 | 训练速度翻倍，成本减半 |
| **DualPipe** | 计算与通信并行（MoE 跨节点通信时同时计算） | 几乎零通信开销 |
| **无辅助损失均衡** | 不用 auxiliary loss 也能均衡专家负载 | 训练更稳定 |
| **数据并行 + 专家并行** | 2048 张 H800 的极致并行 | 2个月完成 14.8T 训练 |

> [!hint]- 面试场景：DeepSeek-V3 为什么训练成本这么低？
> 1. **FP8 混合精度**：相比 BF16 减半的计算量和显存
> 2. **DualPipe**：MoE 的跨节点通信几乎被计算完全掩盖
> 3. **MoE 稀疏激活**：671B 总参数但只激活 37B，计算量远小于同参数 Dense 模型
> 4. **MLA**：KV Cache 小 → 推理成本低 → 训练时 Attention 内存小
> 5. **工程优化**：极致的并行策略和数据管道
> **核心结论**：不是单一优化，而是"MLA + MoE + FP8 + DualPipe"的系统性优化

## 三、SGLang 推理框架深度分析

### 3.1 定位与动机

```
问题：Agent / RAG 场景下的推理效率瓶颈

传统推理框架（vLLM）优化目标：
  → 单轮高吞吐（尽可能多地服务独立请求）

Agent 场景的特点：
  → 多轮对话（同一用户的连续请求）
  → System Prompt 重复（每次都传相同的工具定义）
  → 上下文不断增长（对话历史 + 工具结果）

浪费：
  → 每次 Agent 调用都重新计算 system prompt 的 KV Cache
  → 99% 的 KV 计算是重复的！

SGLang 的目标：
  → 专为 Agent / RAG / 多轮对话场景优化
  → 核心创新：RadixAttention（前缀 KV Cache 复用）
```

### 3.2 RadixAttention 核心创新

```
RadixAttention 的工作原理：

┌──────────────────────────────────────────────────────┐
│                 Radix Tree 结构                       │
│                                                      │
│  Root                                                │
│   │                                                  │
│   ├── "You are a helpful assistant"  ← System Prompt │
│   │       │                                          │
│   │       ├── "User: 天气怎么样"     ← 第1轮          │
│   │       │       │                                  │
│   │       │       └── "AI: 今天晴天" ← 回答1          │
│   │       │               │                          │
│   │       │               └── "User: 明天呢" ← 第2轮  │
│   │       │                       │                  │
│   │       │                       └── [待计算]        │
│   │       │                                          │
│   │       └── "User: 写个排序算法"  ← 另一个对话      │
│   │               │                                  │
│   │               └── [待计算]                        │
│                                                      │
│  每个节点存储：                                       │
│  - Token 序列的 KV Cache                             │
│  - 引用计数（多少请求在用这个前缀）                   │
│                                                      │
│  新请求到达时：                                       │
│  1. 在 Radix Tree 中匹配最长公共前缀                  │
│  2. 复用已有 KV Cache                                │
│  3. 只计算新 token 的 KV                             │
│  4. 将新的 KV Cache 插入 Radix Tree                  │
└──────────────────────────────────────────────────────┘

性能提升示例：
  System Prompt: 2000 tokens
  对话历史: 每轮 500 tokens

  无 RadixAttention:
    第1轮: 计算 2000 + 500 = 2500 tokens 的 KV
    第2轮: 计算 2000 + 500 + 500 = 3000 tokens 的 KV
    第3轮: 计算 2000 + 500 + 500 + 500 = 3500 tokens 的 KV
    总计: 9000 tokens 的 KV 计算

  有 RadixAttention:
    第1轮: 计算 2500 tokens 的 KV（正常）
    第2轮: 复用 2000（system prompt）+ 500（第1轮）= 只计算新 500 tokens
    第3轮: 复用 2000 + 1000 = 只计算新 500 tokens
    总计: 2500 + 500 + 500 = 3500 tokens 的 KV 计算

  节省: (9000 - 3500) / 9000 = 61%！
```

### 3.3 压缩有限状态机（CFSM）

```
Structured Output 的问题：
  Agent 输出 JSON → 需要保证格式正确
  传统方法：逐 token 检查是否合法 → 效率低

SGLang 的 CFSM 解决方案：

  JSON Schema / Regex
         │
         ▼
  ┌──────────────────┐
  │ 编译为压缩有限    │
  │ 状态机 (CFSM)    │
  │                  │
  │ 状态0 → 状态1 → 瀛态2 → ...
  │ 每个状态定义合法 token 集合
  └──────────────────┘
         │
         ▼
  推理时：
  1. 确定当前状态
  2. 只采样合法 token（logits mask）
  3. 状态转移
  4. 继续直到完成

优势：
  - 编译一次，重复使用（JSON Schema → CFSM 只编译一次）
  - 精确引导（保证 100% 格式正确）
  - 高效（状态机很小，mask 计算快）
```

### 3.4 SGLang vs vLLM 详细对比

| 维度 | vLLM | SGLang |
|------|------|--------|
| **核心创新** | PagedAttention（虚拟内存式 KV Cache 管理） | RadixAttention（前缀 KV Cache 复用） |
| **最佳场景** | 单轮高吞吐（独立请求批量处理） | 多轮对话 / Agent / RAG |
| **KV Cache 管理** | 物理块管理（类似虚拟内存分页） | Radix Tree + 前缀共享 |
| **Structured Output** | 支持但效率一般 | CFSM 压缩有限状态机（高效） |
| **量化支持** | GPTQ / AWQ / FP8 | GPTQ / AWQ / FP8 / FP4 |
| **Speculative Decoding** | 支持 | 支持 + MTP 利用 |
| **生态成熟度** | 更成熟，社区更大 | 快速增长，Agent 场景领先 |
| **多模态** | 支持主流 VLM | 支持主流 VLM |
| **部署复杂度** | 简单（pip install） | 简单（pip install） |

> [!hint]- 面试场景：Agent 推理服务，选 vLLM 还是 SGLang？
> **选 SGLang 的场景**：
> 1. Agent 应用（多轮对话、工具调用、system prompt 固定）
> 2. RAG 服务（长 prompt + 短生成，前缀复用率高）
> 3. Structured Output 需求强（JSON/Regex 约束）
>
> **选 vLLM 的场景**：
> 1. 纯聊天机器人（简单多轮，无复杂 system prompt）
> 2. 批量推理（离线处理大量独立请求）
> 3. 需要更成熟的生态和社区支持
>
> **关键决策因子**：前缀复用率
> - 复用率高（>30%）→ SGLang 优势明显
> - 复用率低（<10%）→ vLLM 和 SGLang 差异不大

## 四、推理模型架构分析

### 4.1 OpenAI o1/o3

```
推理模型的核心思想：

传统模型：
  用户输入 → LLM 直接输出答案
  → 快速但可能不准确

推理模型：
  用户输入 → LLM "思考"（生成隐藏的推理链） → 输出答案
  → 慢但更准确

o1/o3 的关键特征：
  1. 思考 Token（Reasoning Token）
     - 模型内部生成推理步骤
     - 用户看不到思考过程
     - 但消耗 Token 和时间

  2. 训练方法（推测）：
     - RL + 过程奖励模型（Process Reward Model）
     - 不只是对最终答案给奖励
     - 对推理过程每一步都给奖励

  3. 与 CoT Prompting 的区别：
     - CoT：用户手动要求 "请一步步思考"
     - o1：模型自动决定思考多少步
     - o1 的思考深度随问题难度自适应
```

### 4.2 DeepSeek-R1

```
DeepSeek-R1 的核心创新：纯 RL 训练推理能力

训练流程：
  Base Model（DeepSeek-V3-Base）
       │
       ▼
  GRPO 强化学习（Group Relative Policy Optimization）
  - 不需要 SFT 阶段
  - 直接用 RL 训练推理能力
  - 奖励信号：答案正确性
       │
       ▼
  R1 模型
  - 自发学会"重新思考"（Aha Moment）
  - 会自我纠错、回溯、尝试不同策略

"Aha Moment" 现象：
  训练过程中，模型自发出现：
  "等等，这个推理不对，让我重新想一下..."
  → 这种自我反思不是被训练的，而是 RL 自然涌现的

R1 的蒸馏能力：
  R1 的推理模式可以蒸馏到小模型
  - 1.5B / 7B / 14B / 32B 等小模型
  - 蒸馏后的小模型也能展现推理能力
  → 推理能力不只是大模型的专利
```

### 4.3 推理模型对工程的影响

| 维度 | 传统模型 | 推理模型 |
|------|---------|---------|
| **延迟** | 低（直接输出） | 高（思考后再输出） |
| **成本** | 低（只计费输出 token） | 高（思考 token 也计费） |
| **准确率** | 对简单问题够用 | 复杂推理显著提升 |
| **适用场景** | 简单问答、格式转换 | 数学、编程、逻辑推理 |
| **什么时候不该用** | 需要低延迟的实时场景 | 不需要深度推理的任务 |

> [!hint]- 面试场景：推理模型和传统模型在架构上有什么区别？
> **架构层面**：
> 1. 基础架构相同（都是 Transformer）
> 2. 训练方法不同：推理模型用 RL 训练推理过程，传统模型用 SFT 训练输出
> 3. 推理时的行为不同：推理模型会生成隐藏的"思考 token"
>
> **关键区别不在架构，在训练方法**：
> - 传统模型：Pretrain → SFT → RLHF（对齐人类偏好）
> - 推理模型：Pretrain → RL（训练推理能力）→ 可选 SFT
>
> **工程影响**：
> - Token 消耗更高（思考 token 可能是输出的 5-10 倍）
> - 延迟更高（思考需要时间）
> - 但复杂任务准确率显著提升

## 五、其他值得关注的 LLM 创新

### 5.1 推理框架新趋势

```
趋势1：Prefill/Decode 分离
  - Prefill（长输入处理）和 Decode（逐 token 生成）资源需求不同
  - Prefill 是计算密集型
  - Decode 是访存密集型
  → 分离到不同硬件/实例，各自优化

趋势2：分布式推理
  - 大模型太大，一张 GPU 放不下
  - Tensor Parallelism（层内切分）
  - Pipeline Parallelism（层间切分）
  - Expert Parallelism（MoE 专家分卡）

趋势3：Speculative Decoding 工程化
  - 不只用小模型做 Draft
  - MTP（DeepSeek）= 模型自带 Draft 能力
  - Medusa = 多头并行预测
  - 接受率提升到 80%+ → 2-3x 加速
```

## 六、面试高频问题汇总

> [!success]- Q1：DeepSeek-V3 为什么训练成本这么低？关键创新有哪些？
> 见 2.5 节详细分析。核心：MLA + MoE + FP8 + DualPipe 的系统性优化，不是单一突破。

> [!success]- Q2：MLA 相比 GQA/MQA 的优势是什么？代价是什么？
> 见 2.2 节的 MLA vs GQA vs MQA 对比表。核心：MLA 是"学习到的压缩"（低秩近似），表达力远优于 GQA 的"结构化共享"。

> [!success]- Q3：SGLang 的 RadixAttention 是怎么实现 KV Cache 复用的？
> 见 3.2 节的 Radix Tree 图示。核心：将 token 序列映射到 Radix Tree，新请求自动匹配最长公共前缀，复用已有 KV Cache。

> [!success]- Q4：DeepSeek 的 MoE 为什么用 256 个细粒度专家？
> 256 个小专家（Top-8）vs 8 个大专家（Top-2）：
> 1. 组合多样性：C(256,8) ≈ 10¹³ >> C(8,2) = 28
> 2. 知识更精确：每个专家更"专精"
> 3. 更少冗余：小专家内部功能不重叠
> 4. 加上共享专家处理通用知识

> [!success]- Q5：如果做 Agent 推理服务，选 vLLM 还是 SGLang？
> 见 3.4 节详细对比。核心决策因子：前缀复用率。Agent 场景复用率高 → SGLang 优势明显。

> [!success]- Q6：MTP 对训练和推理各有什么好处？
> **训练**：模型被迫学会"向前看"，hidden state 包含更多未来信息，表征更好
> **推理**：可直接用作 Speculative Decoding 的 Draft Model，无需额外训练小模型

> [!success]- Q7：FP8 训练的精度损失如何控制？
> 1. **选择性 FP8**：不是所有层都用 FP8，敏感层（如 Attention 分数）保留高精度
> 2. **动态缩放**：每层/每步动态调整 FP8 的缩放因子
> 3. **梯度累积用高精度**：梯度更新仍用 FP32，只有前向/反向的矩阵乘法用 FP8
> 4. **DeepSeek 实测**：FP8 训练的模型与 BF16 训练的模型精度差异 < 0.1%

> [!success]- Q8：推理模型（o1/R1）和传统模型的本质区别是什么？
> **不在架构，在训练方法**：
> - 传统：Pretrain → SFT → RLHF
> - 推理模型：Pretrain → RL（训练推理过程）→ 可选 SFT
> - R1 的 "Aha Moment" 证明：RL 可以自发涌现推理能力

## 关键要点回顾

1. **MLA** = KV 低秩压缩（学习到的压缩 vs GQA 的结构化共享），Cache 压缩 ~66x
2. **DeepSeekMoE** = 256 细粒度专家 + 1 共享专家 + 无辅助损失均衡
3. **SGLang** = RadixAttention（前缀 KV 复用）+ CFSM（Structured Output）
4. **推理模型** = RL 训练推理能力，不是架构创新而是训练范式创新
5. **训练效率** = FP8 + DualPipe + MoE 稀疏激活 = $5.5M 训练 671B 模型
6. **框架选型**：Agent/RAG → SGLang，批量推理 → vLLM

## 深度扩展阅读

| 关联笔记 | 核心内容 |
|----------|---------|
| [[raw/lessons/AI-Interview-Prep/01a-transformer-deep\|01a Transformer 深入]] | Self-Attention 数学 / Attention 变体对比 |
| [[raw/lessons/AI-Interview-Prep/01c-inference-optim\|01c 推理优化]] | KV Cache / PagedAttention / 量化 / Flash Attention |
| [[raw/lessons/AI-Interview-Prep/01b-training-pipeline\|01b 训练管线]] | Pre-training / SFT / RLHF / DPO |

## 导航

← [[raw/lessons/AI-Interview-Prep/06-hot-topics|06 业界热点总览]]
→ [[raw/lessons/AI-Interview-Prep/06a-hot-agent-systems|06a Agent 热点系统]]
→ [[raw/lessons/AI-Interview-Prep/06b-hot-memory-systems|06b Memory 热点系统]]
