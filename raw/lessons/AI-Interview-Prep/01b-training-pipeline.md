---
type: lesson
tags: [面试, 预训练, SFT, RLHF, DPO, LoRA, 数据工程]
created: 2026-05-20
updated: 2026-05-20
difficulty: advanced
prerequisites: [Transformer基础, 深度学习训练]
topic: LLM训练流程深入
status: in-progress
series: {name: "AI Interview Prep", part: "1b"}
---

# LLM 训练流程深入

> 从预训练数据清洗到 DPO 对齐，覆盖 LLM 训练全链路的核心技术和面试高频考点。

## 学习目标

- [ ] 能推导 Next Token Prediction 的交叉熵损失
- [ ] 掌握预训练数据工程全流程：去重、质量过滤、配比设计
- [ ] 理解 LoRA 的低秩分解原理和内在维度假说
- [ ] 能推导 RLHF 的 PPO 目标函数和 KL 惩罚
- [ ] 能从 RLHF 推导出 DPO 的闭环解
- [ ] 掌握数据污染检测和合成数据生成的利弊

---

## 一、预训练深入

### 1.1 Next Token Prediction 的数学推导

给定训练语料 $x_1, x_2, ..., x_T$，语言模型的目标是最大化对数似然：

$$\mathcal{L} = -\sum_{t=1}^{T} \log P(x_t | x_{<t}; \theta)$$

**展开交叉熵损失**：模型输出 logits $z_t \in \mathbb{R}^V$，经过 softmax 得到概率：

$$P(x_t = v | x_{<t}) = \frac{\exp(z_{t,v})}{\sum_{j=1}^V \exp(z_{t,j})}$$

对于真实 token $x_t = v^*$，损失为：

$$\ell_t = -z_{t,v^*} + \log\sum_{j=1}^V \exp(z_{t,j})$$

**信息论视角——为什么自回归训练能学会语言**：

根据 Shannon 的信息论，语言存在统计规律（冗余度约 50%）。自回归模型本质是在学习 $P(x_t | x_{<t})$，即基于上下文对下一个 token 的最优预测。当模型能精确预测下一个 token 时，它必然已经捕获了语言的语法、语义和世界知识。这是自监督信号与下游任务高度一致的根本原因。

> [!hint]- 面试题：预训练loss从3.0降到2.0和从1.5降到1.4，哪个提升更有意义？
> 在 log-likelihood 空间，同样的数值改善对应不同的信息增益。从 3.0 降到 2.0 意味着困惑度从 $e^{3.0} \approx 20$ 降到 $e^{2.0} \approx 7.4$，不确定性减少了 63%。从 1.5 降到 1.4 意味着困惑度从 4.48 降到 4.06，不确定性减少了 9.4%。前期下降对模型能力提升更显著。但后期下降往往更难获得，可能反映模型学到了更深层的语言规律。

### 1.2 数据工程全流程

```
原始数据 → 去重 → 质量过滤 → 有害内容过滤 → 数据配比 → 训练
   │          │         │            │              │
  来源      MinHash   分类器       规则+分类器     领域权重
  CC/书      +LSH     打分          +人工审核      搜索调优
  /代码                                         /训练曲线
```

**数据来源**：
- CommonCrawl：量大但质量参差，需大量清洗
- 书籍：高质量长文本，训练连贯性
- 代码：训练推理和结构化能力
- Wikipedia：高质量知识，但量小
- 论文/数学：STEM 能力

**MinHash + LSH 去重原理**：

MinHash 将文档压缩为一组签名（哈希值的最小值），局部敏感哈希（LSH）将相似签名映射到同一个桶。

1. 将文档分词后生成 n-gram 集合（通常 5-gram）
2. 用 $k$ 个哈希函数计算每个 n-gram 的 MinHash 签名
3. 将签名分成 $b$ 个 band，每个 band $r = k/b$ 行
4. 如果两个文档在任意 band 完全匹配，就是候选重复
5. 参数选择：$P(\text{detect}|J \geq t) = 1 - (1 - t^r)^b$

**数值示例**：设 $k=128, b=16, r=8$，Jaccard 相似度阈值 $t=0.8$：
检测概率 $= 1 - (1 - 0.8^8)^{16} \approx 0.996$，几乎不会漏掉高相似度重复。

> [!warning] 易混点：去重的粒度
> 文档级去重（整篇相同）是最低标准。段落级去重（重复段落）和 n-gram 级去重（长重复片段）对减少记忆化（memorization）更有效。但过度去重会损失多样性。

### 1.3 训练稳定性技巧

**梯度裁剪**：$\|\nabla\| > \theta$ 时，$\nabla \leftarrow \theta \cdot \nabla / \|\nabla\|$。防止梯度爆炸，通常 $\theta = 1.0$。

**Cosine with Warmup**：

$$\eta_t = \eta_{\min} + \frac{1}{2}(\eta_{\max} - \eta_{\min})(1 + \cos(\pi \cdot \frac{t - t_{warmup}}{T - t_{warmup}}))$$

Warmup 阶段（前 0.1~2% 步数）线性增加学习率，让模型参数从随机初始化平滑过渡到合理区域，避免初期大梯度破坏训练。

**长文本训练挑战**：
- 显存：序列长度翻倍，Attention 计算量翻 4 倍（$O(n^2)$）
- 解决方案：Flash Attention（IO 感知的精确注意力，节省 HBM 访问）、Ring Attention（跨设备分布序列维度）、序列并行

> [!hint]- 面试题：Flash Attention 为什么能加速？
> Flash Attention 不是近似算法，计算结果与标准 Attention 完全相同。加速来自减少 HBM（高带宽内存）访问次数：标准 Attention 需要读写 $O(n^2)$ 的中间矩阵 $S = QK^T$ 到 HBM。Flash Attention 将 Q/K/V 分块（tiling），在 SRAM（片上缓存）中完成分块内的 softmax 和矩阵乘，只写回最终输出。用更多计算换更少 IO，在现代 GPU 上 IO 是瓶颈。

---

## 二、SFT 深入

### 2.1 指令数据构造方法

**Self-Instruct**：用种子指令集（175 个任务描述），让 LLM 生成新指令、输入和输出。通过过滤（ROUGE 去重、质量分类器）获得数万条指令数据。Alpaca 是 Stanford 基于此方法用 GPT-3.5 生成的 52K 数据集。

**Evol-Instruct（WizardLM）**：对现有指令进行"进化"——添加约束、加深复杂度、扩展主题，逐步生成更难的指令。比随机生成更能覆盖能力边界。

**数据质量 > 数量**：LIMA 论文证明仅 1000 条高质量指令数据就能训练出接近 GPT-4 的对话能力。关键不是数据的量，而是覆盖度和质量——每条数据都要教会模型一个有用的行为模式。

### 2.2 LoRA 的完整数学原理

**核心思想**：微调时不修改原始权重 $W \in \mathbb{R}^{d \times d}$，而是添加一个低秩增量 $\Delta W = BA$：

$$y = (W + BA)x, \quad B \in \mathbb{R}^{d \times r}, \quad A \in \mathbb{R}^{r \times d}$$

参数量从 $d^2$ 降到 $2rd$（$r \ll d$）。如 $d=4096, r=16$，参数量减少 128 倍。

**为什么低秩分解有效——内在维度假说**：

Aghajanyan et al. (2021) 发现，预训练模型的微调过程具有"低内在维度"特性：虽然模型参数在高维空间，但有效的微调更新实际上发生在低维子空间中。直觉上，预训练已经学到了通用的表示结构，微调只需要在这个结构的低维"方向"上做调整。

**LoRA 超参数详解**：

| 参数 | 含义 | 典型值 | 影响 |
|------|------|--------|------|
| `r` (rank) | 低秩矩阵的秩 | 8, 16, 32, 64 | 越大表达力越强，参数越多 |
| `alpha` | 缩放系数 | 通常 = 2×r | 控制更新的"学习率"，实际缩放 = alpha/r |
| `dropout` | LoRA 层的 dropout | 0.05~0.1 | 防过拟合 |
| `target_modules` | 应用 LoRA 的层 | q_proj, v_proj 或全部 | 应用越多参数越多，效果可能更好 |

> [!warning] 易混点：alpha 和 r 的关系
> LoRA 的实际缩放是 $\Delta W = \frac{\alpha}{r} BA$。$\alpha$ 的作用是解耦学习率和秩的选择——当你增大 r 时，如果不调 alpha，更新幅度会自动减小。设置 $\alpha = 2r$ 是常见做法，保证不同 rank 下更新幅度一致。

### 2.3 QLoRA 和 LoRA 合并

**QLoRA**：在 LoRA 基础上，将原始模型权重量化到 4-bit（NF4 格式），训练时反量化到 bf16 做前向传播，梯度只更新 LoRA 参数。这让 65B 模型能在单张 A100 上微调。

**LoRA 合并**：推理时将 $\Delta W = BA$ 加到 $W$ 上：$W' = W + \frac{\alpha}{r}BA$。合并后没有额外的推理开销，与全量微调的推理速度完全相同。

### 2.4 实际代码：PEFT/LoRA 微调

```python
from peft import LoraConfig, get_peft_model
from transformers import AutoModelForCausalLM

model = AutoModelForCausalLM.from_pretrained("meta-llama/Llama-2-7b-hf")

lora_config = LoraConfig(
    r=16,
    lora_alpha=32,           # alpha = 2 * r
    target_modules=["q_proj", "k_proj", "v_proj", "o_proj",
                     "gate_proj", "up_proj", "down_proj"],
    lora_dropout=0.05,
    task_type="CAUSAL_LM",
)

model = get_peft_model(model, lora_config)
# 可训练参数：~0.5% 的总参数
model.print_trainable_parameters()
# trainable: 13M / total: 6.7B = 0.19%
```

> [!hint]- 面试题：LoRA 应该应用到哪些层？只加 Attention 还是全加？
> 早期做法只加 Q/V 投影（QLoRA 论文推荐），理由是 Attention 层对下游任务适应最关键。但 Hu et al. (2024) 的实验表明，同时加到 Attention 和 FFN 的所有线性层效果最好，且参数增量仍然很小（<1%）。Llama 实践中通常加 q/k/v/o/gate/up/down 七个矩阵。

---

## 三、RLHF 深入

### 3.1 Bradley-Terry 模型

人类偏好建模的基础。给定两个回答 $y_1, y_2$，人类偏好 $y_1 \succ y_2$ 的概率：

$$P(y_1 \succ y_2) = \frac{\exp(r(x, y_1))}{\exp(r(x, y_1)) + \exp(r(x, y_2))} = \sigma(r(x, y_1) - r(x, y_2))$$

其中 $r(x, y)$ 是 Reward Model 给出的标量奖励值，$\sigma$ 是 sigmoid 函数。

**训练 Reward Model**：从人类标注的偏好对 $(y_w, y_l)$（$y_w$ 胜出）训练：

$$\mathcal{L}_{RM} = -\mathbb{E}\left[\log \sigma(r(x, y_w) - r(x, y_l))\right]$$

### 3.2 PPO 在 RLHF 中的完整流程

目标函数：

$$\max_\pi \mathbb{E}_{x \sim D, y \sim \pi(\cdot|x)}\left[r(x, y) - \beta \cdot D_{KL}(\pi(\cdot|x) \| \pi_{ref}(\cdot|x))\right]$$

**KL 惩罚的作用**：无约束优化会让模型"reward hacking"——生成高分但无意义的文本（如重复安全陈述）。KL 惩罚约束策略不偏离参考模型（SFT 模型）太远，确保生成的多样性和自然度。

**PPO 流程四步**：

1. **采样**：从当前策略 $\pi_\theta$ 对 prompt $x$ 生成回答 $y$
2. **评分**：用 RM 计算 $r(x, y)$，用参考模型 $\pi_{ref}$ 计算 KL 惩罚，得到总奖励 $\hat{r} = r - \beta \cdot D_{KL}$
3. **优势估计**：用 Generalized Advantage Estimation (GAE) 计算每步的优势
4. **策略更新**：用 clipped surrogate objective 更新：

$$\mathcal{L}_{PPO} = \mathbb{E}\left[\min\left(\rho_t \hat{A}_t, \text{clip}(\rho_t, 1 \pm \epsilon)\hat{A}_t\right)\right]$$

其中 $\rho_t = \pi_\theta(y_t|x, y_{<t}) / \pi_{old}(y_t|x, y_{<t})$ 是重要性采样比。

> [!hint]- 面试题：RLHF 的训练不稳定性来自哪里？怎么解决？
> 三大不稳定源：(1) Reward Model 本身不完美，高分区域可能不可靠；(2) KL 惩罚难以平衡——太小则 reward hacking，太大则学不动；(3) PPO 的多个超参数（学习率、clip ratio、GAE lambda）敏感。解决方案：使用 reward normalization（运行均值标准化）、adaptive KL controller（动态调整 $\beta$）、value function pre-training、以及更小的学习率配合更多更新步数。

### 3.3 Constitutional AI (RLAIF)

用 AI 模型替代人类做偏好标注：

1. 让模型生成回答
2. 让模型对自己的回答做"宪法评审"（依据一组规则）
3. 让模型生成修订版本
4. 用原始回答 vs 修订版本作为偏好对训练 RM

优势：标注成本极低，可大规模扩展。劣势：AI 偏好可能存在系统性偏差（如过于保守）。

---

## 四、DPO 深入

### 4.1 从 RLHF 到 DPO 的推导

RLHF 的优化目标（带 KL 约束）：

$$\max_\pi \mathbb{E}_{y \sim \pi}[r(x,y)] - \beta D_{KL}(\pi \| \pi_{ref})$$

**关键洞察**：这个优化问题有闭式解：

$$\pi^*(y|x) = \frac{1}{Z(x)} \pi_{ref}(y|x) \exp\left(\frac{1}{\beta}r(x,y)\right)$$

其中 $Z(x)$ 是配分函数（与 $y$ 无关的归一化常数）。从上式反解 reward：

$$r(x,y) = \beta \log \frac{\pi^*(y|x)}{\pi_{ref}(y|x)} + \beta \log Z(x)$$

代入 Bradley-Terry 偏好模型，$Z(x)$ 项在两个回答的差中被消去：

$$P(y_w \succ y_l) = \sigma\left(\beta \log \frac{\pi(y_w|x)}{\pi_{ref}(y_w|x)} - \beta \log \frac{\pi(y_l|x)}{\pi_{ref}(y_l|x)}\right)$$

**DPO 的损失函数**：

$$\mathcal{L}_{DPO} = -\mathbb{E}\left[\log \sigma\left(\beta \log \frac{\pi_\theta(y_w|x)}{\pi_{ref}(y_w|x)} - \beta \log \frac{\pi_\theta(y_l|x)}{\pi_{ref}(y_l|x)}\right)\right]$$

> [!warning] 易混点：DPO 不需要 Reward Model
> DPO 直接优化策略网络 $\pi_\theta$，损失函数中 policy log-prob 替代了显式的 reward。这并不意味着"没有 reward"，而是 reward 隐含在策略与参考策略的 log-ratio 中。DPO 仍需要偏好对数据。

### 4.2 DPO 变体对比

| 方法 | 核心改动 | 优势 | 适用场景 |
|------|---------|------|---------|
| DPO | 闭环解直接优化策略 | 简单稳定 | 标准对齐 |
| IPO | 用平方损失替代 log-sigmoid | 对偏好噪声更鲁棒 | 噪声标注 |
| KTO | 只需好/坏标签，不需要配对 | 数据更容易获取 | 大规模标注 |
| ORPO | 将对齐目标融入 SFT 阶段 | 无需额外对齐阶段 | 资源受限 |
| Online DPO | 训练中在线采样偏好对 | 避免离线数据分布偏移 | 有 RM 或 AI 评审时 |

### 4.3 DPO 训练代码

```python
import torch
import torch.nn.functional as F

def dpo_loss(policy_chosen_logps, policy_rejected_logps,
             ref_chosen_logps, ref_rejected_logps, beta=0.1):
    """DPO 损失函数的核心实现"""
    # log-ratio: log pi(y|x) - log pi_ref(y|x)
    chosen_rewards = beta * (policy_chosen_logps - ref_chosen_logps)
    rejected_rewards = beta * (policy_rejected_logps - ref_rejected_logps)

    # Bradley-Terry 偏好模型
    loss = -F.logsigmoid(chosen_rewards - rejected_rewards).mean()

    # 可选：添加 accuracy 指标
    accuracy = (chosen_rewards > rejected_rewards).float().mean()

    return loss, accuracy, chosen_rewards.mean(), rejected_rewards.mean()
```

> [!hint]- 面试题：DPO 的 reference model 能不能用同一个模型？
> 不行。Reference model $\pi_{ref}$ 必须固定不变（通常是 SFT 后的模型），用于计算"奖励"。如果 reference 也是当前模型，log-ratio 恒为零，梯度为零，训练无效。实践中需要同时加载两个模型（policy 和 reference），显存需求约为 SFT 的两倍。可以用 LoRA 减少 policy 的显存，reference 模型可以量化到 4-bit。

---

## 五、数据工程深入

### 5.1 数据污染检测

预训练数据中混入测试集会导致模型"记住"答案，评估指标虚高。

**检测方法**：

1. **n-gram 重叠**：检查测试样本的 13-gram（或更长）是否出现在训练语料中
2. **会员推理攻击（Membership Inference）**：训练一个分类器判断某个样本是否在训练集中
3. ** perplexity 检测**：模型对训练集样本的 perplexity 显著低于新样本

```python
def ngram_contamination_check(test_text, train_data, n=13):
    """检查测试文本的 n-gram 是否出现在训练数据中"""
    from collections import Counter
    test_ngrams = set()
    tokens = test_text.split()
    for i in range(len(tokens) - n + 1):
        test_ngrams.add(tuple(tokens[i:i+n]))

    contamination_count = 0
    for doc in train_data:
        doc_tokens = doc.split()
        doc_ngrams = set()
        for i in range(len(doc_tokens) - n + 1):
            doc_ngrams.add(tuple(doc_tokens[i:i+n]))
        overlap = test_ngrams & doc_ngrams
        contamination_count += len(overlap)

    return contamination_count
```

### 5.2 合成数据生成的利弊

**优势**：
- 大规模生成特定领域数据（代码、数学、推理）
- 成本远低于人工标注
- 可控的数据分布和难度梯度

**劣势**：
- **模型坍缩（Model Collapse）**：用 AI 生成数据训练下一代 AI，多样性逐代递减
- **幻觉传播**：错误信息被"固化"到训练数据中
- **风格趋同**：生成文本缺乏人类的多样性和创造力

**最佳实践**：
1. 合成数据占比不超过 50%
2. 混合多源数据保持多样性
3. 用强模型（GPT-4 级别）生成，弱模型验证
4. 对合成数据做严格的质量过滤

### 5.3 数据混合策略

不同领域数据的配比对模型能力有显著影响。Llama 3 的数据配比（近似）：

| 数据域 | 占比 | 作用 |
|--------|------|------|
| 网页文本 | ~50% | 通用知识和语言能力 |
| 代码 | ~30% | 推理和结构化能力 |
| 书籍 | ~10% | 长文本连贯性 |
| STEM | ~5% | 科学和数学 |
| 其他 | ~5% | 补充多样性 |

**配比搜索方法**：
1. 小模型（如 1B）上做配比网格搜索
2. 用验证集困惑度和下游任务指标评估
3. 最优配比迁移到大模型

> [!hint]- 面试题：给你一个从网上爬取的 100GB 脏数据集，你怎么清洗成可用的预训练数据？
> 分步清洗流程：
> 1. **格式提取**：从 HTML 提取正文（trafilatura/Readability），去除导航/广告/侧边栏
> 2. **语言识别**：fastText 语言分类，过滤目标语言
> 3. **去重**：MinHash+LSH 文档级去重 → 后缀数组精确去重
> 4. **质量过滤**：训练一个二分类器（高质量 vs 低质量），或用规则过滤（太短/重复/乱码/特殊字符过多）
> 5. **有害内容过滤**：毒性分类器 + PII 检测（去电话/邮箱/身份证号）
> 6. **去污染**：n-gram 检查排除测试集泄漏
> 7. **分词和配比**：按领域分桶，调整配比，混合采样
> 关键指标：清洗后数据量（通常保留 10~30%）、去重率、质量分布。每一步都要做统计和可视化检查。

## 关键要点回顾

1. **预训练核心是数据工程**：70% 的时间在处理数据，模型架构只占 30%
2. **LoRA 有效是因为内在低维度**：微调不需要在高维空间搜索，低秩方向足够
3. **RLHF 的关键是稳定性**：KL 惩罚、reward normalization、adaptive controller
4. **DPO 是 RLHF 的闭式解**：不需要训练 RM，直接用偏好对优化策略
5. **数据污染是隐形杀手**：必须在预训练前做检测，否则评估结果不可信
6. **合成数据要克制**：比例过高会导致模型坍缩，必须与真实数据混合

## 扩展阅读

- Ouyang et al., "Training language models to follow instructions with human feedback" (InstructGPT, 2022)
- Hu et al., "LoRA: Low-Rank Adaptation of Large Language Models" (2021)
- Rafailov et al., "Direct Preference Optimization" (DPO, 2023)
- Dettmers et al., "QLoRA: Efficient Finetuning of Quantized LLMs" (2023)
- Touvron et al., "Llama 3" (2024)

## 下一步学习

- [[01a-transformer-deep|Transformer核心原理深度]] — 架构层面的技术细节
