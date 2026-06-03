---
type: lesson
tags: [MARCH, 多智能体强化学习, 自检机制, 幻觉检测, 论文笔记]
created: 2026-05-16
updated: 2026-05-16
sources:
  - MARCH_2026.pdf
difficulty: advanced
prerequisites:
  - "[[wiki/concepts/multi-agent-system]]"
topic: MARCH 论文精华总结
status: completed
---

# MARCH: Multi-Agent Reinforced Self-Check for LLM Hallucination — 精华总结

> 通过三智能体信息不对称流水线与多智能体强化学习，使 8B 参数模型在幻觉检测与事实对齐上达到闭源模型水平。

## 论文元信息

| 项目           | 内容                                                                                                                                    |
| ------------ | ------------------------------------------------------------------------------------------------------------------------------------- |
| **标题**       | MARCH: Multi-Agent Reinforced Self-Check for LLM Hallucination                                                                        |
| **作者**       | Zhuo Li*, Yupeng Zhang*, Pengyu Cheng*, Jiajun Song, Mengyu Zhou, Hao Li, Shujie Hu, Yu Qin, Erchao Zhao, Xiaoxi Jiang, Guanjun Jiang |
| **机构**       | Qwen Large Model Application Team, Alibaba; The Chinese University of Hong Kong, Shenzhen                                             |
| **会议/期刊**    | arXiv 预印本（尚未正式发表）                                                                                                                     |
| **发表时间**     | 2026 年 3 月 25 日                                                                                                                       |
| **arXiv ID** | 2603.24579v1                                                                                                                          |
| **代码链接**     | https://github.com/Qwen-Applications/MARCH                                                                                            |

## 核心问题

在 RAG（Retrieval-Augmented Generation）系统中，LLM 仍然会产生"上下文冲突型幻觉"——即生成的回答与检索到的文档证据相矛盾。现有的幻觉检测方法（如 LLM-as-a-judge）存在**确认偏误（Confirmation Bias）**问题：验证器在同时看到问题、文档和生成回答时，倾向于验证内部一致性而非基于源文档的客观性，从而不自觉地复现原始生成中的错误。

## 核心方法

### 三智能体流水线

MARCH 将一个共享策略模型 $\pi_\theta$ 实例化为三个功能不同的智能体：

1. **Solver（求解器）**：基于查询 $\mathbf{x}$ 和检索文档 $\mathbf{D}$ 生成初始 RAG 回答 $\mathbf{y}$
2. **Proposer（提议器）**：作为"回答原子化器"，将 Solver 的回答分解为 $n$ 个可验证的问答对 $\{(\mathbf{q}_i, \mathbf{a}_i)\}_{i=1}^n$
3. **Checker（检查器）**：作为"盲审员"，仅基于检索文档 $\mathbf{D}$ 独立回答所有问题 $\{\mathbf{q}_i\}$，**严格屏蔽 Solver 的原始输出和对应答案**

### 信息不对称（Information Asymmetry）

这是 MARCH 的关键创新。Checker 只看到分解后的原子问题和检索文档，**看不到 Solver 的原始输出**。这种精心设计的信息不对称打破了自确认偏误的循环，将验证过程从"对合理文本的循环确认"转变为"生成声明与事实证据之间的严格交叉检验"。

### 零容忍奖励（Zero-Tolerance Reward, ZTR）

奖励函数定义为二元"全有或全无"结构：

$$R(\{(\mathbf{a}_i, \hat{\mathbf{a}}_i)\}_{i=1}^n) = \begin{cases} 0, & \text{if every } \mathbf{a}_i = \hat{\mathbf{a}}_i \\ -1, & \text{otherwise} \end{cases}$$

任何一个声明不匹配，整条轨迹都会被惩罚。这防止模型优化部分正确性或风格上的合理性。

### 多智能体强化学习（MARL）

三个智能体共享同一个策略 $\pi_\theta$，使用 PPO 进行联合优化。每个训练样本同时贡献两条轨迹（Solver 的回答轨迹 $\mathbf{y}$ 和 Checker 的审计轨迹 $\boldsymbol{\lambda}$），通过统一梯度更新使模型同时进化为可靠的生成器和严格的审计器。

$$\nabla_\theta \mathcal{L}_{\text{MARCH}}(\theta) \approx \frac{1}{2|\mathcal{B}|}\sum_{\mathbf{x}\in\mathcal{B}}\Bigg[\sum_{t=1}^{|\mathbf{y}|}\hat{A}_{y}^{t}\nabla_\theta\log\pi_\theta(y^t|\cdot)+\sum_{t=1}^{|\boldsymbol{\lambda}|}\hat{A}_{\lambda}^{t}\nabla_\theta\log\pi_\theta(\lambda^t|*)-\beta\nabla_\theta\text{KL}[\pi_\theta\|\pi_\text{ref}]\Bigg]$$

## 实验设置

| 项目 | 详情 |
|------|------|
| **基座模型** | Meta-Llama3.1-8B-Instruct |
| **训练数据** | STEM（BioASQ，4721 样本）+ General（2WikiMultiHopQA + MuSiQue，各 4500 样本） |
| **训练框架** | VerL + FSDP + vLLM |
| **优化算法** | PPO，1 epoch，global batch size 32 |
| **评估基准** | RAGTruth, FaithBench, ContextualJudgeBench, Facts Grounding, HotpotQA, MuSiQue, 2WikiMultiHopQA |
| **评估方式** | Qwen3-235B-A22B 作为裁判模型，每个查询 8 次独立生成 + 多数投票 |
| **无需标注** | 训练仅使用查询和检索文档，无需人工标注答案 |

## 主要实验结果

### 幻觉检测（RAGTruth + FaithBench 一致性率 %）

| 模型 | Summary | Data2Txt | QA | Average |
|------|---------|----------|------|---------|
| Meta-Llama3.1-8B-Instruct | 71.33 | 48.67 | 63.31 | 55.20 |
| MARCH-STEM | 86.67 | **72.67** | 83.45 | **74.93** |
| MARCH-General | **92.67** | 72.00 | 83.45 | **75.23** |
| o4-mini-high | 76.67 | 64.00 | **82.01** | 67.13 |
| Qwen2.5-14B-Instruct | 82.00 | 51.33 | 93.53 | 68.17 |

MARCH-STEM 平均提升 +19.73，MARCH-General 提升 +20.03，8B 模型超越 14B 和闭源模型。

### 事实对齐（Facts Grounding）

- 基座模型：57.09%
- MARCH-STEM：**85.23%**（+28.14）
- MARCH-General：**80.12%**（+23.03）
- 接近 Gemini 2.5 Flash 水平，超越 GPT-4o

### 多跳 QA（MARCH-Best vs GPT-4o）

| 方法 | HotpotQA | MuSiQue | 2WikiMQA |
|------|----------|---------|----------|
| GPT-4o RAG-Standard | 64.0 | 29.8 | 57.8 |
| GPT-4o IRCoT | 66.4 | 44.2 | 78.0 |
| **MARCH-Best (8B)** | **73.6** | **40.8** | **69.4** |

8B 模型在 HotpotQA 上超越标准 GPT-4o RAG 近 10 个百分点。

### 消融实验关键发现

- **联合优化**（Solver + Checker）始终优于单独优化 Solver
- **ZTR (-1/0)** 优于 ERR（比例惩罚）和 ZTR (0/1)（激励式）
- 框架与 RLHF、Few-Shot、CoT **正交互补**，可叠加提升
- 在 Qwen3-8B 上同样有效（平均 +11%），证明模型无关性

## 论文不足与可改进之处

1. **奖励作弊倾向**：训练过程中 Proposer 倾向于减少提出的问题数量（"少说少错"策略），虽然论文提出了简单的指令约束缓解方案（要求至少提出 $k$ 个问题），但这是一种启发式修补，未从根本上解决奖励设计与信息密度之间的张力。

2. **仅关注数值型验证**：Proposer 的设计明确优先提取数值型声明（numbers, percentages），对定性描述、因果关系、推理链条等非数值型幻觉的检测能力未被直接评估。论文自身也承认"specifically prioritize numerical and quantitative verification"。

3. **依赖单一基座模型**：主要实验基于 Llama3.1-8B-Instruct，虽然在 Qwen3-8B 上做了补充验证，但两个模型都属于 8B 级别。框架在更大规模模型（70B+）或更小模型（1-3B）上的表现未知。

4. **Checker 的可靠性假设**：框架隐含假设 Checker 基于文档的回答是可靠的"伪真值"，但 Checker 本身也是同一个 8B 模型，其审计准确性有上限。论文采用多数投票缓解随机错误，但未分析系统性偏差。

5. **计算开销**：三智能体流水线 + 多次采样审计意味着每次训练步需要 3 倍的前向推理。虽然论文声称"不施加过多的计算负担"，但实际训练时间显著增加（见图 4e），部署时的推理成本也未讨论。

6. **评估依赖强裁判模型**：所有评估使用 Qwen3-235B-A22B 作为裁判，评估结果与裁判模型的能力和偏差耦合，缺乏更细粒度的人工评估。

7. **泛化性未充分验证**：训练数据仅涉及 STEM 和通用 QA 场景，对法律、金融、医疗等高风险领域的领域内泛化能力未做实验验证。

## 导航

- [[raw/lessons/MARCH/01-introduction|01 — 引言]]
- [[raw/lessons/MARCH/02-methodology|02 — 方法论]]
- [[raw/lessons/MARCH/03-experiments|03 — 实验]]
- [[raw/lessons/MARCH/04-related-work|04 — 相关工作]]
- [[raw/lessons/MARCH/05-conclusion|05 — 结论]]
