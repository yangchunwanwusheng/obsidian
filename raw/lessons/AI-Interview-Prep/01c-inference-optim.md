---
type: lesson
tags: [面试, 推理优化, KV-Cache, 量化, Flash-Attention, 部署, vLLM]
created: 2026-05-20
updated: 2026-05-20
difficulty: advanced
prerequisites: [Transformer基础, LLM训练]
topic: LLM推理优化与部署
status: in-progress
series: {name: "AI Interview Prep", part: "1c"}
---

# LLM 推理优化与部署

> 面向 AI 工程师面试的推理优化全景：从 KV Cache 到 Speculative Decoding，从量化到服务化部署。

## 学习目标

- [ ] 能精确计算任意模型配置下的 KV Cache 显存占用
- [ ] 理解量化从数学原理到工程实践的完整链路
- [ ] 掌握 Flash Attention 的 Tiling 策略和硬件友好的设计哲学
- [ ] 能为不同业务场景选型推理框架和部署方案

---

## 一、KV Cache 深入

### 1.1 为什么可以缓存 KV

标准 Multi-Head Attention 的计算公式：

$$\text{Attention}(Q, K, V) = \text{softmax}\left(\frac{QK^T}{\sqrt{d_k}}\right)V$$

自回归生成时，第 $t$ 步的 Query 是 $q_t$（仅第 $t$ 个 token 的向量），但 Key 和 Value 需要所有历史 token $[k_1, k_2, \ldots, k_t]$ 和 $[v_1, v_2, \ldots, v_t]$。核心观察是：**已计算的 $k_i$ 和 $v_i$ 在后续步骤中不会改变**——因为它们只依赖于位置 $i$ 的输入向量，不依赖于后续 token。

数学证明：$k_i = W_K \cdot x_i$，其中 $x_i$ 是第 $i$ 个 token 经过 embedding + position encoding 后的向量。在第 $t+1$ 步时，$x_i$ 没有任何变化，因此 $k_i$ 和 $v_i$ 可以复用。

**什么时候必须重新计算**：当序列的 token 发生变化时（比如 system prompt 变更、前缀修改），受影响位置之后的所有 KV 必须重新计算。

> [!hint]- 面试题：KV Cache 的 prefill 阶段和 decode 阶段有什么本质区别？
> **Prefill 阶段**（处理输入 prompt）：所有 token 的 KV 一次性并行计算，计算密集型（compute-bound），矩阵乘法规模大，GPU 利用率高。
>
> **Decode 阶段**（逐 token 生成）：每步只新增一个 token 的 KV，但需要读取全部历史 KV Cache，访存密集型（memory-bound）。这是推理性能的主要瓶颈。

### 1.2 KV Cache 显存精确计算

对于单次推理请求，KV Cache 的显存占用为：

$$\text{KV Cache Size} = 2 \times n_{\text{layers}} \times n_{\text{kv\_heads}} \times d_{\text{head}} \times s \times b_{\text{dtype}}$$

其中：
- $2$：Key 和 Value 各一份
- $n_{\text{layers}}$：Transformer 层数
- $n_{\text{kv\_heads}}$：KV 头数（GQA 时小于 attention 头数）
- $d_{\text{head}}$：每个头的维度
- $s$：序列长度
- $b_{\text{dtype}}$：每个元素的字节数（FP16 = 2, FP32 = 4）

**具体数值示例**：

| 模型 | 层数 | KV Heads | Head Dim | 每 token KV Cache | 2048 tokens | 8192 tokens |
|------|------|----------|----------|-------------------|-------------|-------------|
| Llama-3-8B | 32 | 8 (GQA) | 128 | 0.125 MB | 256 MB | 1 GB |
| Llama-3-70B | 80 | 8 (GQA) | 128 | 0.3125 MB | 640 MB | 2.5 GB |
| Qwen-2-72B | 80 | 8 (GQA) | 128 | 0.3125 MB | 640 MB | 2.5 GB |

以 Llama-3-8B 为例，单 token FP16 KV Cache：
$$2 \times 32 \times 8 \times 128 \times 2 \text{ bytes} = 131{,}072 \text{ bytes} = 128 \text{ KB}$$

> [!hint]- 面试题：A100 80GB 服务器上 Llama-3-70B 同时服务多少个 4096 长度请求？
> 模型权重（FP16）：$70 \times 2 = 140$ GB，需要 2 张 A100（TP=2）。
> 单卡权重占用 70 GB，剩余 10 GB。
> 单请求 KV Cache（4096 tokens）：$2 \times 80 \times 8 \times 128 \times 4096 \times 2 = 1.25$ GB。
> 还需要留约 2 GB 给激活值和临时缓冲区。
> 可用 KV 空间：$2 \times 10 - 4 = 16$ GB（两张卡合计）。
> 最大并发：$\lfloor 16 / 1.25 \rfloor = 12$ 个请求（非常有限，量化或 PagedAttention 可改善）。

### 1.3 GQA/MQA 的 KV Cache 节省

```
MHA (Multi-Head Attention):     32 heads Q, 32 heads KV
                                  ↓
MQA (Multi-Query Attention):    32 heads Q,  1 head  KV   → KV Cache ×(1/32)
                                  ↓
GQA (Grouped-Query Attention):  32 heads Q,  8 heads KV   → KV Cache ×(1/4)
```

GQA 的折中：比 MHA 省 4 倍 KV Cache（以 8 组为例），同时比 MQA 保留更好的模型质量。Llama-3 的实验表明 GQA 在推理速度和模型质量之间取得了良好平衡。

### 1.4 PagedAttention（vLLM）

核心思想：借鉴操作系统的虚拟内存分页管理。

```
传统 KV Cache（预分配连续内存）：
┌──────────────────────────┐
│ Request A: ████░░░░░░░░  │  ← 预分配最大长度，大量浪费
│ Request B: ██░░░░░░░░░░  │
│ 碎片化：无法合并利用      │
└──────────────────────────┘

PagedAttention（按需分配固定大小 Page）：
┌────┐ ┌────┐ ┌────┐ ┌────┐
│A-1 │ │A-2 │ │B-1 │ │A-3 │  ← 物理上不连续
│token│ │token│ │token│ │token│
│1-16│ │17-32│ │1-16│ │33-48│
└────┘ └────┘ └────┘ └────┘
  ↕ Page Table 映射
Request A: [Page 0] → [Page 1] → [Page 3]
Request B: [Page 2]
```

**PagedAttention 工作流程**：

1. **初始化**：为每个请求创建一个虚拟的 logical block 表，不预分配物理内存
2. **Prefill**：处理 prompt 时按需分配物理 block，记录 logical→physical 映射
3. **Decode**：每生成一个 token，写入当前 block 的下一个空位；block 满则分配新 block
4. **终止**：请求完成，释放所有 block 到全局 block pool
5. **Swap**：GPU 显存不足时，将冷 block 换出到 CPU 内存（类似 swap 到磁盘）

**关键优势**：显存浪费从传统方案的 60-80% 降低到 <4%，因为只按实际使用的 token 数分配。

> [!success]- PagedAttention 的性能数据
> vLLM 论文报告：在 ShareGPT 数据集上，相比原始 HuggingFace 实现，吞吐量提升 24x；相比 Orca（连续批处理），吞吐量提升 3.5x。核心收益来自近乎零的显存碎片浪费。

---

## 二、量化深入

### 2.1 量化的数学基础

**均匀量化**（最常用）将浮点数映射到整数：

$$x_q = \text{clamp}\left(\text{round}\left(\frac{x}{\Delta}\right) + z, q_{\min}, q_{\max}\right)$$

反量化：$\hat{x} = (x_q - z) \cdot \Delta$

其中 $\Delta$ 是 scale（步长），$z$ 是 zero-point。

**对称量化**：$z = 0$，量化范围关于零对称：

$$\Delta = \frac{\max(|x|)}{2^{b-1} - 1}$$

**非对称量化**：$z \neq 0$，适用于激活值分布不对称的情况：

$$\Delta = \frac{x_{\max} - x_{\min}}{2^b - 1}, \quad z = \text{round}\left(-\frac{x_{\min}}{\Delta}\right)$$

> [!hint]- 面试题：对称量化 vs 非对称量化，各适合什么场景？
> **对称量化**：适合权重（分布通常关于零近似对称）。硬件友好，反量化只需一次乘法。
> **非对称量化**：适合激活值（如 ReLU 后全为正数）。能更好地利用量化范围，减少精度损失。但需要额外的 zero-point 运算，开销稍大。

### 2.2 INT8 量化的完整流程

```python
import torch
import numpy as np

def calibrate(model, calibration_data, n_samples=128):
    """Step 1: 收集激活值统计信息"""
    activations = {}
    hooks = []

    def hook_fn(name):
        def fn(module, input, output):
            if name not in activations:
                activations[name] = []
            activations[name].append(input[0].detach().abs())
        return fn

    for name, module in model.named_modules():
        if isinstance(module, torch.nn.Linear):
            hooks.append(module.register_forward_hook(hook_fn(name)))

    with torch.no_grad():
        for i, batch in enumerate(calibration_data):
            if i >= n_samples:
                break
            model(batch)

    for h in hooks:
        h.remove()

    return activations

def compute_scale(activations, quant_type="symmetric"):
    """Step 2: 根据 calibration 数据计算 scale"""
    scales = {}
    for name, acts in activations.items():
        all_acts = torch.cat(acts, dim=0)
        if quant_type == "symmetric":
            max_val = all_acts.max().item()
            scales[name] = max_val / 127  # INT8 range [-128, 127]
        else:
            min_val, max_val = all_acts.min().item(), all_acts.max().item()
            scale = (max_val - min_val) / 255
            zero_point = round(-min_val / scale)
            scales[name] = (scale, zero_point)
    return scales
```

### 2.3 GPTQ 原理

GPTQ 的核心思想：**逐层、逐列量化，利用二阶信息（Hessian）补偿量化误差**。

关键公式：对权重矩阵 $W$ 的第 $i$ 列量化时，通过调整未量化的列来补偿误差：

$$\hat{w}_i = \text{quant}(w_i), \quad \delta_i = w_i - \hat{w}_i$$

补偿修正：$W_{:,i+1:} \leftarrow W_{:,i+1:} - \frac{\delta_i \cdot H_{i,i+1:}^{-1}}{H_{ii}^{-1}}$

其中 $H$ 是 Fisher 信息矩阵的近似（通过 calibration 数据计算）。

**为什么比简单 PTQ 好**：简单 PTQ 独立量化每个权重，误差累积不可控。GPTQ 利用 Hessian 矩阵衡量每个权重的重要性，量化重要权重时更谨慎，并通过误差补偿传播到后续权重。

### 2.4 AWQ 原理

AWQ（Activation-Aware Weight Quantization）的核心洞察：**不是所有权重同等重要，与激活值中大幅值通道对应的权重更为关键**。

```
权重通道重要性评估：
┌──────────────────────────────┐
│ 激活值通道幅度:  [0.1, 5.2, 0.3, 8.1, 0.2] │
│                   ↓              ↓       ↓
│ 重要通道索引:       1              3      │
│ 策略：对这些通道的权重量化时缩放保护  │
└──────────────────────────────────────────┘

缩放因子学习：
  x · W = x · (s · W/s)
  选择 s 使得量化误差最小
```

### 2.5 SmoothQuant

核心思想：**将量化难度从激活值迁移到权重**。

$$X \cdot W = (X \cdot \text{diag}(s)^{-1}) \cdot (\text{diag}(s) \cdot W)$$

通过一个逐通道的缩放因子 $s$，把激活值中的异常值（outlier）"平滑"到权重中。由于权重是静态的、可控的，量化权重比量化动态激活值容易得多。

> [!hint]- 面试题：INT4 vs INT8 vs FP8，如何为业务选择量化方案？
> - **INT4 (W4A16)**：最大加速和显存节省，适合吞吐优先、对精度略可容忍的场景（如批量推理、推荐系统排序）。GPTQ/AWQ 后通常掉点 1-3%。
> - **INT8 (W8A8)**：精度几乎无损（<1%），显存减半，推理速度提升显著。适合绝大多数线上服务。
> - **FP8 (E4M3/E5M2)**：硬件原生支持（H100/4090），动态范围优于 INT8，精度更好。新硬件上首选。
>
> **选型决策树**：
> 1. 硬件支持 FP8？→ 选 FP8
> 2. 精度要求极高？→ INT8 (W8A8)
> 3. 吞吐优先、显存紧张？→ INT4 (AWQ/GPTQ)

---

## 三、Flash Attention 深入

### 3.1 GPU 内存层次

```
┌─────────────────────────────────────────────┐
│ GPU 内存层次（以 A100 为例）                  │
│                                              │
│  SRAM (on-chip)     : 20 MB,  ~19 TB/s       │  ← 极快但极小
│  L2 Cache           : 40 MB,  ~3 TB/s        │
│  HBM (Global Memory): 80 GB,  ~2 TB/s        │  ← 大但相对慢
│                                              │
│  关键比率：SRAM/HBM 带宽比 ≈ 10x             │
│  关键比率：SRAM/HBM 容量比 ≈ 1/4000          │
└─────────────────────────────────────────────┘

性能优化核心洞察：
  减少对 HBM 的访问次数 >> 减少计算量
```

### 3.2 标准 Attention 的 HBM 访问分析

标准 Attention 计算 $S = QK^T$，然后 $P = \text{softmax}(S)$，最后 $O = PV$。

- 中间矩阵 $S$ 和 $P$ 的大小为 $N \times N$（$N$ 为序列长度）
- 写入和读取 $S, P$ 各需要 $O(N^2)$ 次 HBM 访问
- 对于 $N = 8192$，$S$ 矩阵仅存储就需要 256 MB（FP16）

总 HBM 访问：$O(N^2 d)$，其中 $d$ 是头维度。

### 3.3 Flash Attention 的 Tiling 策略

核心思想：**将 Q、K、V 分块加载到 SRAM 中，在 SRAM 内完成 softmax 计算，避免写出巨大的中间矩阵**。

```
Flash Attention Tiling 示意图：

Q 分块：Q₁, Q₂, Q₃, ...   每块大小 B_r × d
K 分块：K₁, K₂, K₃, ...   每块大小 B_c × d
V 分块：V₁, V₂, V₃, ...   每块大小 B_c × d

对外层循环 (Q 块 Qᵢ)：
  初始化：Oᵢ = 0, ℓᵢ = 0, mᵢ = -∞
  对内层循环 (KV 块 Kⱼ, Vⱼ)：
    Sᵢⱼ = Qᵢ · Kⱼᵀ          ← 在 SRAM 中计算
    m̃ᵢⱼ = rowmax(Sᵢⱼ)       ← 当前块最大值
    P̃ᵢⱼ = exp(Sᵢⱼ - m̃ᵢⱼ)    ← 在 SRAM 中做 softmax
    ℓ̃ᵢⱼ = rowsum(P̃ᵢⱼ)       ← 当前块归一化因子

    更新运行统计量：
    mᵢ_new = max(mᵢ, m̃ᵢⱼ)
    ℓᵢ_new = exp(mᵢ - mᵢ_new)·ℓᵢ + exp(m̃ᵢⱼ - mᵢ_new)·ℓ̃ᵢⱼ

    更新输出：
    Oᵢ = Oᵢ · (ℓᵢ · exp(mᵢ - mᵢ_new) / ℓᵢ_new)
         + P̃ᵢⱼ · Vⱼ / ℓᵢ_new
```

**HBM 访问降低到**：$O(N^2 d^2 / M)$，其中 $M$ 是 SRAM 大小。对于典型参数，这比标准 Attention 减少约 5-10 倍 HBM 访问。

### 3.4 Flash Attention 2 vs 1

| 维度 | Flash Attention 1 | Flash Attention 2 |
|------|-------------------|-------------------|
| 非矩阵乘法操作 | 较多 rescaling 操作 | 减少约 2x |
| 并行策略 | 按序列长度分块 | 按批量×头数分块（更好利用 GPU） |
| 工作粒度 | 每个 thread block 处理一个块 | 每个 warp 处理一行 Q |
| 实测加速 | ~2-3x vs 标准 | ~2x vs Flash Attention 1 |

### 3.5 实际性能数据

| 序列长度 | 标准 Attention | Flash Attention 1 | Flash Attention 2 |
|----------|---------------|-------------------|-------------------|
| 512 | 1.0x (baseline) | 1.8x | 3.2x |
| 2048 | 1.0x | 2.5x | 4.8x |
| 8192 | OOM | 3.0x | 5.5x |
| 32768 | OOM | 3.5x | 6.2x |

> [!hint]- 面试题：Flash Attention 能否用于推理阶段？有什么不同？
> Prefill 阶段：完全可以受益，因为计算模式与训练一致（矩阵乘法为主）。
> Decode 阶段：每步只有一个新 token 的 Q，KV Cache 很大。此时瓶颈是读取 KV Cache（memory-bound），Flash Attention 的 Tiling 策略收益有限。需要 **Flash Decoding**——将 KV Cache 分块并行处理，再 reduce 合并。Flash Decoding 在 decode 阶段可获得 2-4x 加速。

---

## 四、Speculative Decoding 深入

### 4.1 数学基础：为什么输出分布精确等价

设目标模型为 $M_p$（大模型），draft 模型为 $M_q$（小模型）。Draft 模型生成 $k$ 个 token：$x_1, x_2, \ldots, x_k$。

**接受/拒绝采样算法**：

```
对每个 draft token x_i (i = 1, 2, ..., k):
  计算接受概率: r = min(1, p(x_i) / q(x_i))
  以概率 r 接受:
    如果接受 → 继续检查下一个 token
    如果拒绝 → 从调整后的分布采样:
      修正分布: p'(x) = max(0, p(x) - q(x)) / Z
      其中 Z = Σ_x max(0, p(x) - q(x))
```

**精确等价性证明思路**：

对于任意 token $x$，它被选中的概率 = 直接被接受 + 拒绝后从修正分布采样：

$$P(\text{output} = x) = q(x) \cdot \min\left(1, \frac{p(x)}{q(x)}\right) + \left(1 - \sum_{x'} q(x')\min\left(1, \frac{p(x')}{q(x')}\right)\right) \cdot \frac{\max(0, p(x) - q(x))}{Z}$$

展开后可以证明这个概率精确等于 $p(x)$。因此，最终输出分布与目标模型完全一致——**零质量损失**。

### 4.2 Draft Model 的选择策略

| 策略 | 优点 | 缺点 |
|------|------|------|
| 同系列小模型（如 Llama-3-8B → Llama-3-70B） | 分布接近，接受率高 | 需要额外显存加载小模型 |
| N-gram / 小型 LM 头 | 几乎零额外开销 | 接受率较低 |
| Medusa 多头 | 并行猜测多个 token | 需要训练 Medusa 头 |
| Self-speculative（早退策略） | 不需要额外模型 | 实现复杂 |

> [!hint]- 面试题：Speculative Decoding 在什么场景下收益最大？
> 收益最大的条件：**draft 模型与大模型分布接近 + 生成任务相对确定性强**。
> - 代码生成、翻译：分布较集中，接受率可达 70-80%
> - 创意写作：分布分散，接受率可能只有 30-40%
> - 关键指标：接受率 $a$，猜测长度 $k$。预期每步生成 token 数 = $\frac{1 - a^{k+1}}{1 - a}$

### 4.3 Medusa：多头并行猜测

```
标准 Speculative Decoding (串行):
  Draft Model → t₁ → t₂ → t₃   ← 串行生成 3 个 token

Medusa (并行):
  ┌─ Head 0 → t₁ (top-1), t₁' (top-2), t₁'' (top-3)
  │
  │  Base Model 的最后一层
  │
  ├─ Head 1 → t₂ (基于 t₁)
  │
  └─ Head 2 → t₃ (基于 t₁)

  → 一次前向传播，猜测多个位置
  → 树状验证路径，取最长匹配
```

---

## 五、推理框架对比

### 5.1 vLLM 架构

```
┌────────────────────────────────────────────────────┐
│                    vLLM 架构                        │
│                                                     │
│  ┌──────────┐    ┌──────────────┐    ┌───────────┐ │
│  │ API Layer│───→│ Scheduler    │───→│ Worker    │ │
│  │ (FastAPI)│    │ (Batch调度)  │    │ (GPU执行) │ │
│  └──────────┘    └──────────────┘    └───────────┘ │
│       │                │                    │       │
│       │          ┌─────┴──────┐      ┌─────┴─────┐ │
│       │          │ BlockMgr   │      │ PagedAttn │ │
│       │          │ (显存管理)  │      │ (Kernel)  │ │
│       │          └────────────┘      └───────────┘ │
│       │                                              │
│  Continuous Batching + PagedAttention + Prefix Cache │
└────────────────────────────────────────────────────┘
```

**核心特性**：
- PagedAttention：零碎片 KV Cache 管理
- Continuous Batching：请求完成即刻替换新请求，不等待整批完成
- Prefix Caching：共享前缀（如 system prompt）的 KV Cache 跨请求复用

### 5.2 框架选型指南

| 框架 | 最佳场景 | 优势 | 限制 |
|------|----------|------|------|
| vLLM | 通用高吞吐服务 | 开源生态好，吞吐高 | 首次请求延迟一般 |
| TensorRT-LLM | NVIDIA GPU 极致性能 | kernel 级优化，延迟最低 | 仅 NVIDIA，工程复杂 |
| llama.cpp | CPU/消费级 GPU | 零门槛，GGUF 量化 | 吞吐受限 |
| TGI | HuggingFace 生态 | 集成度高，部署简单 | 定制性较弱 |

> [!hint]- 面试题：客服对话场景和代码补全场景，分别选什么推理框架？
> **客服对话场景**：请求量大、输出短（<200 tokens）、延迟敏感。选 **vLLM**（Continuous Batching 高吞吐 + 流式输出），量化选 INT8 或 FP8。
>
> **代码补全场景**：输入长（上下文可能上千行）、输出短（几行代码）、TTFT 要求极高。选 **TensorRT-LLM**（极致降低 prefill 延迟），配合 Prefix Caching 缓存文件前缀。

---

## 六、服务化部署

### 6.1 Continuous Batching vs Static Batching

```
Static Batching（等待最慢的请求完成）：
时间 →  t₁    t₂    t₃    t₄    t₅
Req A: [gen]  [gen]  [done] ░░░░  ░░░░     ← 等待中浪费
Req B: [gen]  [gen]  [gen]  [gen]  [done]
Req C: [gen]  [done] ░░░░  ░░░░  ░░░░     ← 大量浪费

Continuous Batching（完成即替换）：
时间 →  t₁    t₂    t₃    t₄    t₅
Req A: [gen]  [gen]  [done]
Req B: [gen]  [gen]  [gen]  [gen]  [done]
Req C: [gen]  [done]
Req D:                [gen]  [gen]  [done] ← C 完成后立即插入
Req E:                       [gen]  [gen]  ← A 完成后插入
```

**吞吐量对比**：Continuous Batching 在变长输出场景下，吞吐量提升 2-4x。

### 6.2 推理服务 SLO 设计

| 指标 | 全称 | 含义 | 典型目标 |
|------|------|------|----------|
| TTFT | Time To First Token | 首个 token 延迟 | < 200ms |
| TPOT | Time Per Output Token | 每 token 生成延迟 | < 50ms |
| Throughput | tokens/s | 每秒总生成 token 数 | 视业务定 |
| E2E Latency | End-to-End | 用户感知总延迟 | < 2s (短输出) |

**TTFT 主要由 prefill 时间决定**，TPOT 主要由 decode 的 KV Cache 读取和 attention 计算决定。两者优化的方向不同。

### 6.3 多 LoRA 服务

```python
# 多 LoRA 服务的核心思路
class MultiLoRAService:
    """共享基座模型，动态切换 LoRA adapter"""

    def __init__(self, base_model, lora_adapters):
        self.base = base_model              # 一份基座权重
        self.adapters = lora_adapters       # 多个 LoRA 权重
        # LoRA 权重很小（通常 < 100MB），可以全部常驻显存

    def forward(self, input_ids, adapter_id):
        # 1. 基座模型前向传播到目标层
        # 2. 从 adapter pool 取出对应 LoRA A, B 矩阵
        # 3. h = h + (x @ A) @ B  ← 仅一次额外矩阵乘
        # 4. 继续后续层
        pass
```

**显存开销**：基座模型权重 ×1 + LoRA 权重 ×N。LoRA 通常为 rank=16，权重仅为原模型的 0.1-1%，因此 10 个 LoRA 总增量 < 10%。

---

## 关键要点回顾

1. **KV Cache 是推理的显存瓶颈**，GQA/PagedAttention/量化是三大缓解手段
2. **量化选择**取决于硬件（FP8 > INT8 > INT4）和精度容忍度
3. **Flash Attention** 通过 SRAM Tiling 减少 HBM 访问，2-6x 加速
4. **Speculative Decoding** 用小模型猜测大模型验证，数学上保证输出分布不变
5. **Continuous Batching + PagedAttention** 是高吞吐推理服务的标配
6. 面试中最常考：KV Cache 计算、量化方案选型、推理框架对比

## 扩展阅读

- [vLLM: Efficient Memory Management for LLM Serving](https://arxiv.org/abs/2309.06180)
- [FlashAttention-2: Faster Attention with Better Parallelism](https://arxiv.org/abs/2307.08691)
- [GPTQ: Accurate Post-Training Quantization](https://arxiv.org/abs/2210.17323)
- [Speculative Decoding: Fast inference from large models](https://arxiv.org/abs/2211.17192)
- [SmoothQuant: Accurate and Efficient Post-Training Quantization for LLMs](https://arxiv.org/abs/2211.10438)

## 下一步学习

- [[raw/lessons/AI-Interview-Prep/02a-agent-paradigms|Agent 推理范式深度]]
- [[raw/lessons/AI-Interview-Prep/03-rag|RAG 系统设计]]
