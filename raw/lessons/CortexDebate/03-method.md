---
type: lesson
tags: [CortexDebate, 翻译笔记, 稀疏辩论图, 方法论]
created: 2026-05-16
updated: 2026-05-16
sources:
  - CortexDebate_2025.pdf
difficulty: advanced
prerequisites:
  - "[[raw/lessons/CortexDebate/00-essence]]"
topic: CortexDebate — 稀疏辩论图方法
status: completed
---

# CortexDebate — 稀疏辩论图方法 (Section 3 & 4)

> 本篇翻译论文第 3 节 Preliminaries 和第 4 节 Methodology 的核心内容，包括问题定义、整体框架和稀疏辩论图的构建方法。

## 正文翻译

### 3.1 问题定义

CortexDebate 在 n 个 LLM 智能体之间建立一个有向辩论图 $\mathcal{G} = (\mathcal{A}, \mathcal{E})$，其中：

- **$\mathcal{A} = \{A_i\}_{i=1}^n$** 是顶点集，代表参与的 LLM 智能体
- **$\mathcal{E} = \{E_{i \to j}\}_{i,j \in [1,2,...,n]}$** 是有向边集，代表信息传输

每条有向边 $E_{i \to j}$ 被分配一个权重 $W_{i \to j}$，表示通过辩论智能体 $A_i$，智能体 $A_j$ 的性能预期提升程度。所有权重 $\{W_{i \to j}\}$ 在辩论过程中动态优化。

给定一个问题 $Q$，智能体 $\{A_i\}_{i=1}^n$ 进行 D 轮辩论。在第 d 轮辩论中，每个 LLM 智能体 $A_i$ 审视与其相连的 LLM 智能体的输出，然后生成自己的输出 $O_i^d$ 以及自置信度分数 $H_i^d$。之后，该辩论轮次的最终答案 $F_d$ 通过多数投票获得。

> [!tip] 理解提示（非原文）
> 这里的关键区分是"有向边"——$E_{i \to j}$ 表示 $A_i$ 对 $A_j$ 有帮助，但反过来 $A_j$ 对 $A_i$ 不一定有帮助。所以辩论图是有向的，边的权重也是不对称的。这意味着 $A_i$ 可能需要辩论 $A_j$ 的观点，但 $A_j$ 不一定需要辩论 $A_i$ 的观点。

### 4.1 Phase 1: 初始答案生成

给定问题 $Q$，CortexDebate 让每个 LLM 智能体 $A_i$ 独立生成初始输出 $O_i^0$ 和自置信度分数 $H_i^0$。为缓解过度自信，CortexDebate 采用重校准 (recalibration) 策略，该策略在先前工作中已被证明有效 (Chen et al., 2023)：

$$H_i^0 = \begin{cases} 0.8, & H_i^0 \geq 0.8 \\ 0.6, & 0.6 \leq H_i^0 < 0.8 \\ H_i^0, & 0.3 \leq H_i^0 < 0.6 \\ 0.3, & H_i^0 < 0.3 \end{cases}$$

> [!note] 译者注（非原文）
> 这个重校准策略的本质是"压缩置信度的高位和低位"：超过 0.8 的全部压到 0.8，低于 0.3 的全部抬到 0.3，中间区间的保持不变。这样防止某个智能体给出 0.99 的极端高置信度来主导辩论。

### 4.2 Phase 2: 多轮辩论

CortexDebate 进入辩论阶段，智能体集合 $\{A_i\}$ 进行 D 轮辩论。在第 d 轮辩论中，CortexDebate 包含四个步骤。

#### Step 1: 边权重优化

MDM 基于 Credibility、Reliability、Intimacy 和 Self-orientation 四个方面计算边权重。每个方面的具体计算如下：

**Credibility ($C_d$)** — 专业能力

由于 LLM 的 Scaling Law (Hoffmann et al., 2022) 可以评估一个 LLM 智能体的能力，我们使用它来计算 Credibility。Scaling Law 的公式为：

$$\mathcal{L}(N, M) = \frac{406.4}{N^{0.34}} + \frac{410.7}{M^{0.28}} + 1.69$$

其中 $N$、$M$ 和 $\mathcal{L}$ 分别表示模型的参数量、预训练数据的 token 数量和预训练损失。损失值越小表示模型能力越好，因此：

$$C_d = \frac{1}{\mathcal{L}(N, M)}$$

> [!warning] 易混点（非原文）
> 这里的 Scaling Law 公式来自 Chinchilla 论文 (Hoffmann et al., 2022)，是一个经验公式。它根据模型的参数量 N 和预训练数据量 M 来估计预训练损失。注意：Credibility 在辩论过程中是固定的（因为模型参数和预训练数据不变），不像其他三个因素会随辩论进行而更新。

**Reliability ($R_d$)** — 表现稳定性

代表 $A_i$ 在前 d-1 轮中对自己答案的平均置信度分数：

$$R_d = \frac{R_{d-1} \times (d-1) + H_i^{d-1}}{d}$$

**Intimacy ($I_d$)** — 观点差异程度

代表 $A_i$ 和 $A_j$ 在前 d-1 轮中观点差异的平均程度。MDM 首先使用余弦相似度计算 $O_i^{d-1}$ 和 $O_j^{d-1}$ 之间的文本相似度。然后计算平均观点相似度：

$$\overline{Sim}_d = \frac{\overline{Sim}_{d-1} \times (d-1) + cos(O_i^{d-1}, O_j^{d-1})}{d}$$

由于 $I_d$ 代表差异程度，因此：

$$I_d = 1 - \overline{Sim}_d$$

> [!tip] 理解提示（非原文）
> Intimacy 这个名字可能有些误导。在社会学原始公式中，Intimacy 指的是"亲密程度"——你和某人越亲密，越信任他。但在 MAD 适配中，论文将其重新定义为"观点差异程度"——两个智能体观点越不同，"碰撞"越多，辩论越有价值。这是 Intimacy 概念的一个巧妙转换。

**Self-Orientation ($S_d$)** — 自我导向程度

基于"较少参与群体讨论意味着更自私"的事实，MDM 使用 $A_i$ 在前 d-1 轮中与其他智能体辩论的次数 $P_d$ 来间接反映自我导向：

$$S_d = (d-1) \times (n-1) - P_d$$

其中 $(d-1) \times (n-1)$ 表示一个智能体在前 d-1 轮中最多可以与其他智能体辩论的次数。

**综合边权重计算**

遵循 McKinsey Trust Formula，边 $E_{i \to j}$ 的权重计算为：

$$W_{i \to j}^d = \frac{C_d \times R_d \times I_d}{S_d}$$

#### Step 2: 稀疏图建立

对于 $A_j$，它可以与其他 n-1 个 LLM 智能体辩论。CortexDebate 根据指向它的边的权重 $\{W_{i \to j}^d\}_{i=1, i \neq j}^n$ 来确定辩论对手集合。

首先，计算这些边的平均权重：

$$\overline{W}_j^d = \frac{1}{n-1} \sum_{i(i \neq j)} W_{i \to j}^d$$

其次，权重低于 $\overline{W}_j^d$ 的边被移除，形成稀疏辩论图：

$$W_{i \to j}^d = \begin{cases} 1, & W_{i \to j} \geq \overline{W}_j^d \\ 0, & W_{i \to j} < \overline{W}_j^d \end{cases}$$

因此，$A_j$ 的辩论对手集合为：

$$Deb_j^d = \{A_i \mid W_{i \to j}^d = 1, i \neq j\}$$

> [!note] 译者注（非原文）
> 稀疏化的策略非常简洁：对每个智能体，只保留权重高于平均值的入边。这意味着每个智能体平均只与约一半的其他智能体辩论。随着辩论进行，如果某些智能体持续表现不佳或对某个智能体没有帮助，它们之间的连接会被逐步剪除，使图越来越稀疏。

#### Step 3: 答案再生成

LLM 智能体 $A_j$ 接收 $Deb_j^d$ 中智能体在第 (d-1) 轮生成的答案，审视这些答案后生成新的答案 $O_j^d$ 和自置信度分数 $H_j^d$。输入提示可以表示为：

$$Prompt_j^d = [Ins, Q, \{O_k^{d-1}\}]$$

其中 $Ins$ 表示指示 $A_j$ 重新生成答案的指令，$\{O_k^{d-1}\}$ 表示 $A_j$ 接收到的答案集合。

#### Step 4: 辩论终止

所有 LLM 智能体生成答案后，CortexDebate 检查是否所有智能体达成共识（即所有智能体同意同一个答案）或辩论达到最大轮数。如果满足条件，辩论过程立即结束。

### 4.3 Phase 3: 最终答案生成

辩论过程结束后，CortexDebate 通过对最后一轮辩论中所有生成的答案进行多数投票来生成最终答案：

$$O_{final} = \arg\max_o \sum_i \mathbb{1}(O_i = o)$$

其中 $o$ 表示任一 LLM 智能体生成的不同答案。如果辩论后所有生成的答案都不同，我们将最终结果视为不正确。通过排除后备策略 (fallback strategies)，我们可以将观察到的性能清晰地归因于辩论机制本身。

> [!note] 译者注（非原文）
> "排除后备策略"意味着如果所有智能体的答案都不同，论文不采用额外策略（如随机选择、选择最自信的答案等），而是直接记为错误。这是一个严格但公平的评估方式，确保性能提升确实来自辩论本身。

### McKinsey Trust Formula 引入 (Section 3.2)

McKinsey Trust Formula (Lamarre et al., 2012) 在社会学中被广泛用于评估一个人在群体中的可信度水平。该公式可表示为：

$$T = \frac{C \times R \times I}{S}$$

其中 C、R、I 和 S 分别表示 Credibility（可信度）、Reliability（可靠性）、Intimacy（亲密性）和 Self-orientation（自我导向）。其中：

- **Credibility** 衡量专业能力
- **Reliability** 衡量任务表现的稳定性
- **Intimacy** 衡量与被评估者的关系
- **Self-orientation** 衡量被评估者在群体中的自我导向水平

在 MDM 模块中，我们将这四个因素适配到 MAD 场景：

| 因素 | MAD 适配含义 |
|------|-------------|
| Credibility | 评估 $A_i$ 的专业能力 |
| Reliability | $A_i$ 对自己答案的平均置信度分数，代表在当前问题上的表现可靠性 |
| Intimacy | $A_i$ 和 $A_j$ 在历史辩论中的观点差异平均程度，因为不同观点的碰撞可以增强辩论效果 (Xiong et al., 2023) |
| Self-orientation | $A_i$ 在辩论中的参与水平（参与水平越低，自我导向越高） |

## 关键要点回顾

- 辩论图是有向的，边权重反映"一个智能体对另一个智能体的预期帮助程度"
- 置信度重校准策略防止极端高/低置信度
- MDM 模块通过 McKinsey Trust Formula 的四个因素综合评估边权重
- 稀疏化策略简洁有效：移除权重低于平均值的边
- 辩论终止条件：共识达成或达到最大轮数

## 下一步学习

- [[raw/lessons/CortexDebate/04-trust-formula|04 - McKinsey Trust Formula 与 MDM 模块详解]]
- [[raw/lessons/CortexDebate/05-experiments|05 - 实验]]
