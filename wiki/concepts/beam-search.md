---
type: concept
tags: [深度学习, Transformer, Beam Search, 解码策略, Top-k, Temperature]
created: 2026-04-09
updated: 2026-04-09
sources:
  - raw/lessons/Transformer/07-training-inference.md
---

# Beam Search 与解码策略

> 自回归生成中选择下一个 token 的方法：从贪心解码到 Beam Search，再到采样策略（Top-k、Top-p、Temperature），在确定性与多样性之间寻找平衡。

## 概述

在 [[wiki/concepts/autoregressive-generation|自回归生成]] 的每一步，模型输出的是词表上的一个概率分布。解码策略决定了如何从这个分布中选出下一个 token。不同的策略直接影响生成文本的质量、多样性和计算成本。

## 解码策略对比

| 策略 | 核心思想 | 多样性 | 计算成本 | 适用场景 |
|------|---------|--------|---------|---------|
| **贪心解码** | 每步选概率最高的词 | 极低 | O(T) | 实时系统、简单对话 |
| **Beam Search** | 同时维护 k 条候选路径 | 中等 | O(k x T x V) | 机器翻译、文本摘要 |
| **Top-k 采样** | 从概率最高的 k 个词中随机采样 | 较高 | O(T x V) | 开放式文本生成 |
| **Top-p 采样** | 从累积概率达 p 的最小词集中采样 | 较高（自适应） | O(T x V) | 通用文本生成 |
| **Temperature** | 调节分布的尖锐程度 | 可调 | O(T) | 配合其他策略使用 |

## 贪心解码

每一步取概率最大的 token：$\hat{w}_t = \arg\max_w P(w | w_{1:t-1})$。

**致命缺陷**：局部最优不等于全局最优。第一步选择了概率 0.6 的词 A 而放弃 0.4 的词 B，但如果词 B 后续路径的联合概率更高，贪心解码就会错过更好的结果。这就像下棋时只看一步的棋手，容易陷入局部陷阱。

## Beam Search

同时维护 $k$ 条候选序列（beam），每一步扩展所有候选的所有可能后续，保留联合概率最高的 $k$ 条继续前进。

- **beam size $k$**：通常取 3-5。$k$ 越大质量越好，但边际效益递减，计算成本线性增长
- **长度惩罚**：Beam Search 倾向生成短句（因为 log 概率累加越短越大），通过 Length Penalty 修正：$\text{score} = \frac{\log P(w_{1:T})}{HP^\alpha(T)}$，其中 $HP(T) = \frac{5+T}{6}$，$\alpha$ 通常取 0.6-0.7

## 采样策略

**Top-k 采样**：固定保留概率最高的 $k$ 个候选词，其余概率置零后重新归一化。缺点是 $k$ 值固定，无法适应不同上下文的分布形态——有些位置需要更多样的选择，有些则应更集中。

**Top-p 采样（Nucleus Sampling）**：动态选择累积概率首次超过 $p$ 的最小词集。分布集中时自动缩小候选集（接近贪心），分布平坦时自动扩大候选集（保持多样性），因此比 Top-k 更自适应。

**Temperature**：通过调节 Softmax 温度 $P_T(w_i) = \frac{\exp(z_i / T)}{\sum_j \exp(z_j / T)}$ 控制分布形状。$T < 1$ 使分布更尖锐（更确定），$T > 1$ 使分布更平坦（更多样），$T = 1$ 为标准分布。通常与其他策略组合使用。

## 要点

- 贪心解码简单快速，但容易陷入局部最优
- Beam Search 通过多路径搜索提升质量，是翻译和摘要任务的标准选择
- Top-p 采样比 Top-k 更自适应，是目前开放式生成的首选策略
- Temperature 不单独使用，通常与 Top-k/Top-p 组合控制生成风格
- 选择策略的核心权衡：确定性 vs 多样性、质量 vs 速度

## 相关页面

- [[wiki/concepts/autoregressive-generation|自回归生成]] — 解码策略所服务的生成范式
- [[wiki/concepts/kv-cache|KV Cache]] — 加速解码过程中每一步的注意力计算
