---
type: concept
tags: [深度学习, Transformer, KV Cache, 推理优化, 注意力加速]
created: 2026-04-09
updated: 2026-04-09
sources:
  - raw/lessons/Transformer/07-training-inference.md
---

# KV Cache

> 通过缓存注意力中的 Key-Value 向量，避免 [[wiki/concepts/autoregressive-generation|自回归生成]] 中的重复计算，将推理复杂度从 $O(T^2)$ 降低到约 $O(T)$。

## 概述

在 [[wiki/concepts/autoregressive-generation|自回归生成]] 过程中，每生成一个新 token，模型都需要重新计算整个序列的 [[wiki/concepts/self-attention|Self-Attention]]。这意味着第 $t$ 步需要计算 $t$ 个 token 之间的注意力，而前 $t-1$ 个 token 的 Key 和 Value 向量其实与前一步完全相同——它们只是被重复计算了一遍。

KV Cache 的核心观察：**已生成 token 的 Key 和 Value 向量在后续步骤中不会改变**。因此可以将其缓存下来，每步只需计算新 token 的 Q、K、V，然后将新的 K、V 追加到缓存中。

## 加速原理

无 KV Cache 时，生成一个长度为 $T$ 的序列，第 $t$ 步需要计算 $t$ 个 token 的注意力，总计算量为 $\sum_{t=1}^{T} t = \frac{T(T+1)}{2} = O(T^2)$。

有 KV Cache 时，第 $t$ 步只计算 1 个新 token 的 Q、K、V 以及它与缓存的交互，总计算量降为 $\sum_{t=1}^{T} 1 = O(T)$。

```
无 KV Cache:
  Step 1: Q=[w1], K=[<s>,w1], V=[<s>,w1]         → 全部重新计算
  Step 2: Q=[w2], K=[<s>,w1,w2], V=[<s>,w1,w2]   → w1 重新计算！
  Step 3: Q=[w3], K=[<s>,w1,w2,w3], ...           → w1, w2 重新计算！

有 KV Cache:
  Step 1: 计算 w1 的 K,V → 写入缓存
  Step 2: 计算 w2 的 K,V → 追加缓存，从缓存读取 w1
  Step 3: 计算 w3 的 K,V → 追加缓存，从缓存读取 w1, w2
```

## 工作流程

1. **初始步骤**：计算第一个 token 的 Q、K、V，将 K 和 V 存入缓存
2. **后续每步**：
   - 只计算新 token 的 Q、K、V
   - 将新的 K、V 追加到缓存末尾
   - 用新 token 的 Q 与缓存中所有 K 计算注意力分数
   - 用注意力权重与缓存中所有 V 加权求和，得到输出
3. **生成结束**：丢弃缓存，释放内存

## 内存代价

KV Cache 以显存换取速度。缓存的显存占用为：

$$\text{显存} = 2 \times n_{layers} \times n_{heads} \times d_{head} \times T \times \text{batch\_size} \times \text{dtype\_size}$$

以 GPT-3（175B）为例：96 层、96 头、128 维 head、FP16 精度、序列长度 2048，单条请求的 KV Cache 约需 **2.4 GB** 显存。当 batch size 增大或序列更长时，KV Cache 的显存消耗会成为瓶颈。

## 与现代优化的关系

KV Cache 的朴素实现会预分配最大长度的连续显存，导致严重的显存碎片和浪费。工业界的解决方案包括：

- **PagedAttention**（vLLM）：将 KV Cache 按固定大小的"页"管理，按需分配和释放，类似操作系统的虚拟内存分页机制
- **连续批处理（Continuous Batching）**：多个请求共享 GPU 计算，每个请求独立管理自己的 KV Cache 生命周期
- **KV Cache 量化**：将缓存的 K、V 从 FP16 压缩到 INT8 甚至 INT4，以精度换显存
- **滑动窗口注意力**：只缓存最近 W 个 token 的 KV，超出的丢弃，以长度换显存

## 要点

- KV Cache 利用了自回归生成中已计算 token 的 K、V 不变性
- 将推理复杂度从 $O(T^2)$ 降至 $O(T)$，是 Transformer 推理的标配优化
- KV Cache 不改变模型输出，只避免重复计算
- 显存占用随序列长度、模型规模、batch size 线性增长
- PagedAttention 等现代技术解决了 KV Cache 的显存管理瓶颈

## 相关页面

- [[wiki/concepts/autoregressive-generation|自回归生成]] — KV Cache 所优化的生成范式
- [[wiki/concepts/self-attention|Self-Attention]] — KV Cache 缓存的 K、V 向量来源
- [[wiki/concepts/beam-search|Beam Search 与解码策略]] — 解码策略与 KV Cache 的配合使用
