---
type: research-topic
title: Closed-loop Safe Multi-Agent Reasoning — 基于 G2CP 与 GUARDIAN 的统一研究设计
status: 选定
tags: [multi-agent, hallucination, closed-loop-control, communication-protocol, anomaly-detection, reinforcement-learning]
created: 2026-05-17
updated: 2026-05-17
target_conference: NeurIPS / ICLR / ACL
project_hub: "[[../closed-loop-safe-multi-agent-reasoning/00-hub|Project Hub]]"
sources:
  - raw/lessons/G2CP/00-essence.md
  - raw/lessons/GUARDIAN/00-essence.md
  - raw/lessons/MARCH/00-essence.md
  - research/02-papers/multi-agent-hallucination-reading-guide.md
---

# Closed-loop Safe Multi-Agent Reasoning

> 将多智能体推理中的错误传播建模为一个**部分可观测的动态风险控制问题**，统一处理预防、检测、恢复与学习。

## 当前结论

- **主路线**：风险感知闭环控制器（risk-aware closed-loop controller）
- **增强建模**：引入 latent contamination belief，把异常检测信号从“观测”提升为“控制状态”
- **不采用的路线**：不做大规模端到端 LLM 训练；不做纯 benchmark 工程；不做简单模块拼装
- **当前目标场景**：通用问答 / 推理多智能体
- **资源约束**：中成本，方法为主，评测仅做轻量闭环扩展

## 一句话描述

> 如何让多智能体推理系统在协作过程中**自适应地预防错误传播、检测污染状态、执行恢复干预，并从历史轨迹中学习更优干预策略**？

## 研究意义

### 学术价值

当前多智能体 LLM 研究对“协作有效性”关注较多，但对“协作过程中错误如何传播、何时该介入、介入后如何恢复”缺乏统一建模。已有工作大多分散在：

- **预防类**：通过通信约束降低歧义与幻觉传播
- **检测类**：通过异常检测发现可疑 agent / message / interaction pattern
- **恢复类**：通过 heuristics 做回滚、重试或重新辩论
- **学习类**：通过自检或 RL 优化局部推理能力

但这些工作尚未把它们统一成一个闭环控制问题。因此，本方向的学术价值不在于“再加一个模块”，而在于提出一个统一抽象：

> **多智能体推理中的错误传播，是一个动态风险控制问题，而不是单点纠错问题。**

### 应用价值

- **高风险问答系统**：事实核查、法律问答、医学辅助分析
- **复杂推理系统**：多跳问答、数学推理、长期协作式问题分解
- **企业级 agent 工作流**：多个 agent 共同处理知识检索、审查、汇总与决策
- **可审计协作 AI**：要求解释性、可回放、可定位失败源头的实际系统

## 核心基础论文与启发来源

| 工作 | 核心角色 | 对本研究的启发 | 局限 |
|---|---|---|---|
| [[raw/lessons/G2CP/00-essence\|G2CP]] | 预防 | 用结构化图操作替代自由文本通信，显著降低语义漂移与级联幻觉 | 更像静态防护，缺少何时切换、何时放宽的动态控制 |
| [[raw/lessons/GUARDIAN/00-essence\|GUARDIAN]] | 检测 | 用时序属性图建模多智能体交互，检测异常节点 / 边 / 传播模式 | 能发现问题，但不回答“发现后怎么恢复、是否值得恢复” |
| [[raw/lessons/MARCH/00-essence\|MARCH]] | 学习 / 自检 | 用多智能体自检与强化学习提升事实一致性 | 重点是 self-check，不是统一预防—检测—恢复闭环 |
| [[research/02-papers/multi-agent-hallucination-reading-guide\|多智能体幻觉导读]] | 文献地图 | 给出检测、协议设计、辩论优化、模型异质性等技术族谱 | 仍需将多个分支统一到单一问题表述中 |

## 研究问题与主叙事

### 核心问题表述

不把问题写成“如何把 G2CP、GUARDIAN 和 RL 拼起来”，而写成：

> **多智能体推理中的错误传播，本质上是一个动态风险控制问题。**
> 系统需要在协作过程中持续判断：当前通信是否安全、哪些 agent 或 message 可能被污染、何时应限制通信、何时应执行恢复，以及如何从历史交互中学习更优干预策略。

### 现有工作的统一缺口

#### A. 预防类缺口
G2CP 证明了结构化通信可以显著降低歧义和级联幻觉，但它默认“强约束协议始终是更优选择”。实际系统中，通信约束过强会带来覆盖面损失、灵活性下降和额外成本，因此需要回答：

- 何时应该自由通信？
- 何时应该切到结构化通信？
- 是否存在按风险自适应切换协议的更优策略？

#### B. 检测类缺口
GUARDIAN 能定位异常节点、异常边与传播模式，但其输出是风险信号，不是恢复策略本身。系统仍需决定：

- 异常是否已经严重到需要干预？
- 应该隔离、回滚、重检索还是重辩论？
- 在预算有限时，哪种恢复动作最划算？

#### C. 学习类缺口
MARCH 等方法把学习引入 factual consistency，但更多聚焦 self-check，而不是把**通信约束、异常观测、恢复决策、长期策略改进**统一成一个闭环问题。

### 研究问题（RQs）

#### RQ1
**Can adaptive intervention outperform static communication or static defense strategies in multi-agent reasoning?**

自适应干预是否优于固定通信协议或固定防御策略？

#### RQ2
**Can a latent contamination-aware state representation improve the timing and quality of interventions?**

引入潜在污染状态（latent contamination belief）是否能改善干预时机与恢复质量？

#### RQ3
**Can a learned control policy achieve a better accuracy–safety–cost trade-off than heuristic recovery rules?**

学习到的控制策略，能否比手工 heuristics 在正确率、安全性与成本之间取得更优权衡？

### 论文核心贡献雏形

1. **提出一个风险感知闭环框架**，统一多智能体协作中的预防、检测、恢复与学习
2. **提出 contamination-aware state 表示**，把异常检测信号转化为控制决策可用的风险状态
3. **提出自适应恢复策略学习机制**，动态选择通信约束与修复动作
4. **在通用问答 / 推理场景中验证**：相比静态协议、静态检测、固定恢复规则，取得更优的 accuracy–safety–efficiency trade-off

### 电梯陈述

> We frame error propagation in LLM multi-agent reasoning as a closed-loop risk control problem, where the system learns when to prevent, detect, and recover from contamination during collaboration.

## 系统架构

### 四层闭环结构

#### 第 1 层：任务协作层（Task Collaboration Layer）
基础多智能体推理系统，负责问题分解、候选答案生成、中间推理交换与最终汇总。该层提供协作能力，但不天然保证安全。

#### 第 2 层：通信控制层（Communication Control Layer）
负责控制 agent 之间的通信模式，而不是默认采用固定协议。支持三类模式：

1. **自由文本通信**：灵活、便宜，但语义漂移风险高
2. **轻结构化通信**：claim / evidence / confidence 槽位化通信
3. **强结构化通信（G2CP 风格）**：以图操作、关系查询、结构化更新进行通信

这里 G2CP 的角色是：

> **一种高约束、低歧义的通信动作 / 通信模式，而不是整个系统本身。**

#### 第 3 层：风险观测层（Risk Observation Layer）
继承 GUARDIAN 的核心思想，将协作过程建模为**时序交互图**：

- **节点**：某一轮中的 agent 输出、claim 或 message state
- **边**：影响、引用、反驳、支持等通信关系
- **属性**：语义 embedding、evidence support、contradiction、confidence、provenance 等

风险观测器输出：

- node anomaly score
- edge contamination score
- subgraph inconsistency score
- propagation risk score

这里 GUARDIAN 的角色是：

> **风险观测器 / contamination sensor，而不是恢复器。**

#### 第 4 层：闭环控制与恢复层（Closed-loop Control & Recovery Layer）
这是论文最核心的层，包含两项职责：

1. **belief update**：维护当前污染状态的风险 belief
2. **intervention policy**：根据 belief 选择通信约束和恢复动作

恢复动作不再是固定 heuristics，而是一组受风险状态驱动的控制动作。

### 每一轮的闭环流程

```text
用户问题 q
   ↓
多智能体生成 / 交换中间结果
   ↓
构建当前时序交互图 G_t
   ↓
风险观测器产生 o_t
   ↓
belief updater 估计 b_t
   ↓
控制器选择动作 a_t
   ↓
执行干预（限流 / 结构化通信 / 隔离 / 回滚 / 重检索 / 重辩论）
   ↓
进入下一轮或终止输出
```

### 统一抽象

这套框架的统一性来自以下映射：

- **协议约束** = preventive action
- **异常检测** = observation mechanism
- **恢复操作** = control action
- **策略改进** = policy learning

因此论文不再是“协议 + 检测器 + 恢复器 + RL”的松散组合，而是：

> **LLM 多智能体协作中的错误传播，是一个部分可观测的动态风险控制问题。**

## 状态、动作、奖励与学习目标

### 3.1 隐状态：系统真正关心什么

定义不可完全观测的隐状态 $x_t$，表示当前协作过程的污染状态：

$$
x_t = \big(\mathbf{c}^{\mathrm{agent}}_t,\; \mathbf{c}^{\mathrm{edge}}_t,\; h_t,\; \rho_t\big)
$$

其中：

- $\mathbf{c}^{\mathrm{agent}}_t$：各 agent 的污染程度
- $\mathbf{c}^{\mathrm{edge}}_t$：各通信边的污染程度
- $h_t$：当前整体推理健康度（reasoning health）
- $\rho_t$：当前状态的可恢复性（recoverability）

直观上，系统真正想知道的不是“哪句话像错了”，而是：

> **当前协作过程是否已经进入 contamination regime，以及错误是否仍然可修复。**

### 3.2 观测：控制器实际能看到什么

系统通过当前时序交互图、语义一致性和预算信息形成观测 $o_t$：

$$
o_t = \phi(G_t, M_t, E_t, B_t)
$$

其中：

- $G_t$：当前时序交互图
- $M_t$：消息级语义与冲突特征
- $E_t$：证据支持与 provenance 特征
- $B_t$：预算与成本特征

建议观测包含四组特征：

1. **图结构风险特征**：node/edge anomaly、传播深度、异常子图聚集度
2. **语义与证据特征**：claim–evidence support、contradiction、citation consistency、retrieval agreement
3. **协作行为特征**：当前通信模式、轮数、主导 agent、异常快速收敛等
4. **成本特征**：token、latency、累计干预次数、剩余预算

### 3.3 belief state：把观测转成风险判断

由于 $x_t$ 不可直接观测，系统维护 belief state：

$$
b_t = p\big(x_t \mid o_{1:t}, a_{1:t-1}\big)
$$

这表示系统当前对污染程度与传播风险的判断。为保证中成本可实现，可使用工程化的 risk belief vector，而非完整贝叶斯推断。比如：

- $r_t^{\mathrm{agent}}$：top-k 可疑 agent 风险
- $r_t^{\mathrm{edge}}$：top-k 可疑传播边风险
- $r_t^{\mathrm{global}}$：全局污染概率
- $r_t^{\mathrm{drift}}$：问题漂移风险
- $r_t^{\mathrm{repair}}$：当前修复可行性分数

最终控制器使用的状态可写成：

$$
z_t = \big[b_t; c_t\big]
$$

其中 $c_t$ 表示轮次、协议模式、预算等协作上下文。

### 3.4 动作空间：控制器能做什么

建议采用**分层、离散、小而强**的动作空间。定义：

$$
a_t = (m_t, u_t)
$$

其中 $m_t$ 是通信模式控制，$u_t$ 是恢复 / 干预动作。

#### 通信模式动作 $m_t$

$$
m_t \in \{\text{FREE},\; \text{SEMI-STRUCTURED},\; \text{GRAPH-GROUNDED}\}
$$

- **FREE**：自由文本通信
- **SEMI-STRUCTURED**：claim / evidence / confidence 槽位化通信
- **GRAPH-GROUNDED**：G2CP 风格结构化图操作通信

#### 恢复动作 $u_t$

$$
u_t \in \{\text{CONTINUE},\; \text{ISOLATE}(i),\; \text{ROLLBACK}(\Delta),\; \text{REROUTE},\; \text{RETRIEVE-AGAIN},\; \text{RE-DEBATE},\; \text{STOP-AND-SYNTHESIZE}\}
$$

为防止动作空间爆炸，第一篇论文建议采用如下约束：

- 每轮只做一个高层动作
- 仅在 top-1 / top-2 可疑 agent 中选择隔离对象
- rollback 只允许回滚最近 1–2 轮

### 3.5 奖励设计：学会少出错、少浪费、少误杀

每一步奖励可写为：

$$
r_t = \lambda_1 R_{\mathrm{task},t} - \lambda_2 R_{\mathrm{prop},t} - \lambda_3 R_{\mathrm{cost},t} - \lambda_4 R_{\mathrm{over},t}
$$

其中：

- $R_{\mathrm{task},t}$：任务正确性收益
- $R_{\mathrm{prop},t}$：错误传播惩罚
- $R_{\mathrm{cost},t}$：token / latency / 干预成本惩罚
- $R_{\mathrm{over},t}$：过度干预或误杀惩罚

整条轨迹的优化目标为：

$$
J(\pi_\theta) = \mathbb{E}_{\pi_\theta}\Bigg[\sum_{t=1}^{T} r_t\Bigg]
$$

若要更贴合“安全控制”叙事，可采用约束优化版本：

$$
\max_{\pi_\theta} \; \mathbb{E}\big[R_{\mathrm{task}} - \alpha R_{\mathrm{cost}}\big]
$$

subject to

$$
\mathbb{E}\big[R_{\mathrm{unsafe\text{-}pass}}\big] \le \varepsilon
$$

其中 $R_{\mathrm{unsafe\text{-}pass}}$ 表示高风险状态下仍放行而导致错误的代价。

### 3.6 学习目标：学高层控制，而不是重训底层 LLM

推荐训练路线：

1. **规则策略 warm start**
   - anomaly 高则升级通信模式
   - propagation risk 高则 isolate / re-debate
   - evidence support 低则 re-retrieve

2. **离线策略学习（主实验）**
   - 从带污染注入的协作轨迹中收集 $(z_t, a_t, r_t)$
   - 采用 contextual bandit / offline RL / cost-sensitive policy learning 学高层控制器

3. **轻量在线改进（增强实验，可选）**
   - 在少量交互预算下做在线策略修正

该设计的关键点是：

> **不训练底层 LLM，只训练基于风险 belief 的高层控制器。**

这使得实验成本、归因清晰度和可解释性都更可控。

## 最小实验包与评测协议

### 4.1 评测总原则

本工作应坚持“**方法为主，评测扩展最小但足够说服人**”的原则：

1. **固定底层 agent scaffold**，避免把改进归因到额外 prompt 技巧
2. **固定 base model / agent 数 / 回合数 / 预算**，保证横向公平
3. **控制污染注入强度**，让检测率、恢复率和误杀率可比
4. **同时报告正确率、安全性、成本**，避免只靠 accuracy 单点取胜

### 4.2 任务集选择

建议使用“3 个主任务 + 1 个兼容验证任务”的设置。

| 类型 | 数据集 | 角色 | 选择理由 |
|---|---|---|---|
| 事实验证 | FEVER | 主任务 | 有明确证据支撑，非常适合观测错误传播与纠正 |
| 多跳问答 | HotpotQA 或 2WikiMultiHopQA | 主任务 | 容易出现跨 agent 的中间推理误传与问题漂移 |
| 数学推理 | MATH 或 GSM8K-hard | 主任务 | 适合测试错误步骤是否会在多轮协作中被放大 |
| 通用知识 | MMLU（子集） | 兼容验证 | 与 GUARDIAN 评测传统保持兼容，便于横向对比 |

### 4.3 固定 agent scaffold

为了把贡献归因于闭环控制器，而非 agent 角色工程，建议固定一个通用 4-agent scaffold：

1. **Solver**：给出初始解答或初始推理链
2. **Critic**：质疑与反驳关键步骤
3. **Verifier / Evidence Agent**：负责证据检索或步骤核查
4. **Synthesizer**：综合各方输出最终答案

对不同任务的适配：

- FEVER / HotpotQA：Verifier 偏 evidence-grounded
- MATH：Verifier 偏 step-level checking
- MMLU：Critic + Verifier 重点检验知识一致性与自信错误

### 4.4 三类核心污染场景

建议至少做三类污染 / 风险场景：

| 场景 | 构造方法 | 主要考察 | 是否需要显式标签 |
|---|---|---|---|
| S1: 内生幻觉放大 | 让一个上游 agent 在高熵 / 弱证据条件下先生成错误推理 | 系统能否尽早约束传播 | 是 |
| S2: Agent-targeted 证据污染 | 给某个 agent 注入错误检索证据或伪事实 | 检测器能否定位污染源、控制器能否隔离 | 是 |
| S3: Communication-targeted 篡改 | 在消息传递阶段篡改 claim / rationale / citation | 协议约束与恢复动作是否有效 | 是 |

可选第四类增强场景：

- **S4: Long-horizon drift**：增加辩论轮数，测试问题漂移与后期恢复能力

### 4.5 Baseline 设计

| Baseline | 描述 | 作用 |
|---|---|---|
| Single-Agent | 单体 CoT / RAG | 提供最基础下界 |
| MA-FreeText | 自由文本多智能体协作 | 检验自然语言通信下的传播问题 |
| MA-SemiStruct | 静态轻结构化通信 | 检验简单结构约束是否已足够 |
| G2CP-Static | 始终使用强结构化通信 | 检验“纯预防”路线 |
| GUARDIAN-Heuristic | 检测 + 固定 heuristics 恢复 | 检验“纯检测/修复”路线 |
| ClosedLoop-Heuristic | 本框架但不学习，只用 belief-based 规则 | 检验学习层的真实增益 |
| **Ours** | belief-guided learned controller | 完整方法 |

### 4.6 核心指标

#### 任务指标

- Accuracy / EM / task success
- FEVER label correctness
- MATH final answer correctness

#### 安全指标

- **Detection AUROC / F1**：识别污染节点 / 边的能力
- **Propagation Rate**：错误 claim 被多少 downstream agents 采纳
- **Recovery Success Rate**：污染 episode 被成功纠正的比例
- **Unsafe Pass Rate**：高风险状态下未干预而最终失败的比例
- **Over-intervention Rate**：在 clean episode 中不必要干预的比例

#### 成本指标

- token consumption
- latency
- intervention count
- graph-grounded mode usage ratio

建议论文主图用 **accuracy–safety–cost frontier** 呈现，而不是只报一张 accuracy 表。

### 4.7 关键消融实验

至少做以下消融：

1. **去掉 belief state**：直接用 anomaly score 做动作
2. **去掉学习层**：只保留 hand-crafted policy
3. **去掉 graph-grounded mode**：只允许 free / semi-structured
4. **去掉恢复动作**：只允许协议切换，不允许 isolate / rollback / re-debate
5. **去掉成本项**：看是否会出现过度保守、疯狂干预的策略

这些消融分别对应三条主张：

- belief 是否必要
- 学习是否必要
- G2CP 风格预防动作是否必要
- 恢复动作是否真正带来额外收益

### 4.8 Claim–Evidence 绑定（实验版）

| 核心 claim | 需要的证据 |
|---|---|
| 自适应闭环优于静态协议 / 静态防御 | Ours vs MA-FreeText / G2CP-Static / GUARDIAN-Heuristic |
| belief 表示改善干预时机 | Ours vs no-belief ablation |
| 学习策略优于 heuristics | Ours vs ClosedLoop-Heuristic |
| 闭环控制改善 accuracy–safety–cost trade-off | frontier 图 + 安全指标 + 成本指标联合报告 |

### 4.9 中成本执行建议

为了保证方案可落地，建议遵循以下预算控制：

- **主实验只保留 1 个强基座模型 + 1 个轻量泛化模型**
- **agent 数固定为 4**
- **最大交互轮数先固定为 3**
- **先跑小规模子集验证再全量扩展**
- **学习层优先使用离线策略学习，而非大规模在线 RL**

## 当前最大 reviewer 风险

1. **模块拼装质疑**：审稿人会问你的统一性在哪里，而不是接受“把三篇论文拼一起”
2. **学习增益不明显**：如果 learned controller 只比 heuristics 略好，会被质疑是否值得
3. **评测协议太人造**：如果污染注入过于人为，审稿人会怀疑泛化性
4. **恢复动作归因不清**：如果 isolate / rollback / re-debate 一起上，必须做足消融

## 当前建议的论文定位

> **A belief-guided high-level controller that adaptively selects communication constraints and recovery interventions under partial observability.**

更直接地说：

- 不训练底层 LLM
- 不做端到端多智能体 RL
- 只学习一个基于风险 belief 的高层控制器

这是当前资源条件下最稳、最可解释、也最容易做出清晰实验故事的版本。

## Reviewer Attack Map 与应对策略

### Attack 1：这是不是模块拼装，而不是统一方法？

**典型攻击**
- “你只是把 [[raw/lessons/G2CP/00-essence|G2CP]]、[[raw/lessons/GUARDIAN/00-essence|GUARDIAN]] 和一个 RL controller 拼在一起。”
- “统一性在哪里？为什么这不是 engineering composition？”

**应对策略**
1. 始终坚持主叙事：
   > 本文研究对象不是某个模块，而是**多智能体错误传播的闭环风险控制问题**。
2. 在方法定义层显式给出统一映射：
   - protocol constraint = preventive action
   - anomaly detection = observation mechanism
   - recovery = control action
   - learning = policy improvement
3. 做出两类关键消融：
   - 只用检测、不做控制
   - 只用协议切换、不做恢复
   证明单独模块不足以支撑完整收益

**最危险信号**
- 如果完整方法仅比“检测+heuristic 恢复”略好，审稿人会更倾向于认为这只是系统拼装

### Attack 2：learned controller 真的比 heuristics 强吗？

**典型攻击**
- “你训练了一个策略器，但简单规则也能做得差不多。”
- “RL / policy learning 是否只是把阈值调优自动化？”

**应对策略**
1. 设置强 heuristic baseline：ClosedLoop-Heuristic
2. 报告不仅是 accuracy，还包括：
   - unsafe pass rate
   - over-intervention rate
   - recovery success rate
   - frontier 曲线（accuracy–safety–cost）
3. 强调 learned policy 的价值在于：
   - 更好的介入时机
   - 更少不必要干预
   - 对不同污染场景的自适应切换

**最危险信号**
- 如果 learned controller 相比 heuristic 只在单一任务上小幅提升，而没有 trade-off 改善，则主张会变弱

### Attack 3：污染注入是不是太人造，不代表真实多智能体失败？

**典型攻击**
- “agent-targeted / communication-targeted 注入是 synthetic artifact。”
- “真实系统错误并不是这样发生的。”

**应对策略**
1. 明确三类场景分别对应真实 failure family：
   - 内生幻觉放大 → spontaneous hallucination amplification
   - agent-targeted 证据污染 → retrieval corruption / local misinformation
   - communication-targeted 篡改 → message distortion / relay corruption
2. 补一个**自然失败轨迹分析**：从 clean-free-text 多智能体协作中收集真实失败轨迹，标注其与三类注入场景的对应关系
3. 说明本工作目标不是穷尽现实攻击，而是覆盖最有代表性的传播机制

**最危险信号**
- 如果所有收益都只出现在强注入场景、而在自然失败轨迹上不成立，会被质疑外推性

### Attack 4：belief state 是否真的必要？

**典型攻击**
- “直接用 anomaly score 就能做决策，为什么要引入 latent contamination belief？”

**应对策略**
1. 做 no-belief ablation：直接 anomaly-to-action
2. 展示 belief 的价值不在“更复杂”，而在：
   - 结合历史观测与动作
   - 区分瞬时异常与持续污染
   - 更稳地判断是否值得恢复
3. 若可能，可给出一个 case study：
   - 单轮 anomaly 高但其实是 healthy disagreement
   - belief-based controller 选择继续观察
   - direct threshold controller 误触发恢复

**最危险信号**
- 如果 no-belief ablation 与完整方法几乎无差异，那么路线 2 的增强价值会被削弱

### Attack 5：结构化通信的收益是不是只是 prompt engineering 或上下文变短？

**典型攻击**
- “G2CP 模式变强只是因为 prompt 结构更清晰。”
- “节省 token 本身就会减少错误，不一定是协议本质作用。”

**应对策略**
1. 设立静态轻结构化 baseline：MA-SemiStruct
2. 证明 GRAPH-GROUNDED 模式在高风险阶段优于单纯槽位化格式
3. 分离两种收益：
   - token / context efficiency
   - semantic ambiguity reduction
4. 用传播率和恢复率指标证明，不只是 prompt 更整洁，而是传播机制被改变

**最危险信号**
- 如果 GRAPH-GROUNDED 和简单 JSON baseline 没显著差异，那么 protocol 层创新会被削弱

### Attack 6：恢复动作太多，归因不清楚

**典型攻击**
- “你同时用了 isolate、rollback、re-debate、re-retrieve，究竟是谁起作用？”

**应对策略**
1. 控制动作空间规模，第一版限制到小而强
2. 分层报告动作使用频率与成功率
3. 设计动作级消融：
   - 无 isolate
   - 无 rollback
   - 无 re-debate
4. 用场景-动作匹配表说明不同动作针对不同 failure mode

**最危险信号**
- 如果动作级消融没有显著分工特征，系统会显得像“动作堆叠”

## Claim–Evidence Sheet

| claim | direct evidence | supporting evidence | invalidating evidence | target experiment | target metric | likely reviewer attack |
|---|---|---|---|---|---|---|
| 自适应闭环优于静态协议或静态防御 | Ours vs MA-FreeText / MA-SemiStruct / G2CP-Static / GUARDIAN-Heuristic 主结果 | 不同污染场景下的稳定提升；frontier 曲线 | 仅在单一场景有效；成本显著更高 | 主实验 | Accuracy, Propagation Rate, Cost Frontier | “只是拼装，不是统一方法” |
| belief-guided 控制优于直接 anomaly-to-action | Ours vs no-belief ablation | case study；时序稳定性分析 | 两者几乎无差异 | belief 消融 | Recovery Success, Over-intervention Rate | “belief 没必要” |
| 学习策略优于 hand-crafted heuristics | Ours vs ClosedLoop-Heuristic | 干预时机更优；误杀更少 | heuristic 已基本达到同样效果 | policy 对比实验 | Unsafe Pass, Over-intervention, Accuracy–Cost trade-off | “RL 只是自动调阈值” |
| graph-grounded 高约束模式在高风险阶段是必要的 | Ours vs no-graph-mode / MA-SemiStruct | 在高污染场景中传播率显著下降 | 与简单结构化通信无显著差异 | protocol 消融 | Propagation Rate, Token Efficiency, Recovery Success | “只是 prompt engineering” |
| 恢复动作集合优于仅检测不恢复 | Ours vs detect-only baseline | 不同 failure mode 的动作分工统计 | 恢复动作几乎不被用到或收益不明显 | recovery 消融 | Recovery Success Rate, Final Accuracy | “检测就够了，恢复没贡献” |
| 轻量闭环评测协议能覆盖真实 failure family | 自然失败轨迹与三类污染注入的 failure mapping | 多任务一致性；失败案例分析 | 收益仅存在于人工注入，真实失败无效 | 自然失败补充实验 | Failure Coverage, Qualitative Alignment | “评测太人造，外推不足” |

## Baseline Ladder 与 Prompt / Protocol 具体定义

### 6.1 设计原则

为了避免审稿人质疑“结果只是 prompt engineering 或 scaffold 不公平”，所有 baseline 必须遵守四条硬约束：

1. **same data access**：所有方法看到同一问题输入、同一检索库、同一证据候选集
2. **same tool access**：若任务允许 retrieval / verifier tool，则所有可比方法都拥有相同工具能力
3. **matched budget**：控制总 token / 回合数 / agent 数；若存在额外开销，必须显式报告
4. **matched evaluation protocol**：污染注入位置、轮数上限、终止条件、打分方式统一

进一步地，整篇论文必须尽量保持以下固定不变：

- 同一 base model family（主实验）
- 同一 agent scaffold（Solver / Critic / Verifier / Synthesizer）
- 同一最大轮数
- 同一检索预算
- 同一最终答案格式

变化项只能来自：

- 通信协议形式
- 是否启用检测器
- 是否启用恢复动作
- 干预策略是 heuristic 还是 learned

### 6.2 Reviewer-credible baseline ladder

#### Lower-bound baseline

##### Single-Agent
单体 CoT / RAG。

**作用**：提供最基础下界，回答“多智能体是否真的有必要”。

**固定条件**：
- 与多智能体使用相同 base model
- 相同 retrieval access
- 相同最终预算上限（可按 token 归一化报告）

---

#### Standard baseline

##### MA-FreeText
标准自由文本多智能体协作。

**定义**：
- 4 个 agent 使用自然语言交换中间结果
- 无额外结构化槽位
- 无显式检测器
- 无恢复器
- 仅按固定轮数协作后由 Synthesizer 输出

**作用**：代表最常见的 LLM 多智能体协作范式，是“自然语言传播链”的标准基线。

---

#### Strong baseline

##### MA-SemiStruct
轻结构化多智能体协作。

**定义**：
- 与 MA-FreeText 完全相同的角色与轮数
- 只把消息格式改成固定槽位：claim / evidence / confidence / rationale
- 不允许 graph-grounded 操作
- 无显式检测器
- 无恢复器

**作用**：排除“只是 prompt 更整洁”的解释，证明强结构化通信的收益不等同于普通 JSON / slot formatting。

##### GUARDIAN-Heuristic
检测 + 固定恢复规则。

**定义**：
- 通信默认使用 free-text 或 semi-structured（需在主实验中固定一种）
- 启用 GUARDIAN 风格风险观测器
- 恢复策略为固定阈值 heuristics：
  - anomaly 高 → isolate 或 re-debate
  - evidence support 低 → re-retrieve
  - propagation risk 高 → 升级通信模式或回滚
- 不学习策略

**作用**：代表“有检测、有修复，但没有统一学习控制”的强基线。

---

#### Strongest published baseline

##### G2CP-Static
始终使用高约束 graph-grounded communication。

**定义**：
- 全程采用 G2CP 风格通信
- 不做动态模式切换
- 可无检测器，也可加固定验证器，但不能引入 learned controller
- 回合结构与 agent scaffold 固定

**作用**：代表“纯预防”路线的 strongest published-style baseline。

> 若后续实验主场景与原始 G2CP 工业设置不同，需要明确写成“G2CP-inspired static graph-grounded protocol adapted to QA/reasoning tasks”，避免夸大为原论文直接复现。

---

#### Budget-matched baseline

##### ClosedLoop-Heuristic
与 Ours 完全同 scaffold、同观测、同动作空间，但控制策略不用学习，而是手工规则。

**定义**：
- 与 Ours 使用相同 risk observer
- 使用相同动作空间：FREE / SEMI-STRUCTURED / GRAPH-GROUNDED + isolate / rollback / re-debate / re-retrieve 等
- 只把 policy 替换成 hand-crafted rules
- 使用相同预算限制

**作用**：这是最关键的 budget-matched baseline，用来回答：

> “如果系统已经有同样的检测器、同样的动作空间、同样的预算，学习策略是否仍然有价值？”

---

#### Optional oracle / upper-bound proxy

##### Oracle Controller（可选）

**定义**：
- 假设可访问真实污染标签或更强的后验信息
- 在该信息下选择最优或近似最优恢复动作

**作用**：不是正式 baseline，而是上界代理，用来回答：
- 当前 learned controller 距离理想干预还有多远
- 现阶段 bottleneck 在检测、belief 还是 policy

### 6.3 主结果表中建议保留的 baseline 集

主表不要过多塞 baseline，建议保留：

1. Single-Agent
2. MA-FreeText
3. MA-SemiStruct
4. G2CP-Static
5. GUARDIAN-Heuristic
6. ClosedLoop-Heuristic
7. **Ours**

如果版面紧张：
- Oracle Controller 放附录
- 其他 published debate-style baselines 可放补充实验或 related comparison table

### 6.4 各 baseline 的公平对照边界

#### Single-Agent vs Multi-Agent
可回答“多智能体值不值得”，但不能用来证明闭环控制优越性。

#### MA-FreeText vs MA-SemiStruct
可回答“收益是否只是更整洁的格式化 prompt”。

#### MA-SemiStruct vs G2CP-Static
可回答“强结构化 graph-grounded communication 是否比一般结构化槽位更强”。

#### GUARDIAN-Heuristic vs ClosedLoop-Heuristic
可回答“统一动作空间和协议切换是否必要，而不仅仅是检测+局部修复”。

#### ClosedLoop-Heuristic vs Ours
这是论文最关键的一组。

它回答：
> 在观测器、动作空间、预算都相同的前提下，learned high-level controller 是否比 heuristics 更优？

### 6.5 Prompt / protocol 固定模板原则

为了避免 prompt unfairness，建议所有 baseline 共享以下部分：

#### 共享部分
- task instruction
- role description（Solver / Critic / Verifier / Synthesizer）
- answer format
- tool usage rules
- max rounds
- stop condition

#### 允许变化的部分
- 消息表示格式
- 是否包含 graph operation schema
- 是否暴露 anomaly / risk signal 给 controller
- 是否允许触发恢复动作

也就是说：

> 不允许通过“给 Ours 更详细任务解释、给 baseline 更弱 prompt”来制造优势。

### 6.6 各 baseline 的最小 protocol 定义

#### Single-Agent
- 输入：问题 + 可选检索证据
- 输出：单次答案或 CoT 后答案
- 不存在 inter-agent communication

#### MA-FreeText
- 每个 agent 接收上轮其他 agent 的自然语言消息摘要
- 消息是开放文本
- 无固定字段要求

#### MA-SemiStruct
- 每条消息必须包含：
  - `claim`
  - `evidence`
  - `confidence`
  - `rationale`
- 但仍以自然语言槽位内容为主，不进行图操作

#### G2CP-Static
- 消息不再是自由文本，而是 graph-grounded operation / structured update
- 所有轮次都保持该模式
- 不根据风险动态切换

#### GUARDIAN-Heuristic
- 保持既定通信模式
- 每轮构建时序交互图
- 根据固定阈值触发动作

#### ClosedLoop-Heuristic
- 每轮：观测 → belief summary → heuristic action
- 与 Ours 唯一差别在于 policy 不是 learned

#### Ours
- 每轮：观测 → belief update → learned policy action
- 动态选择通信模式与恢复动作

### 6.7 审稿人最关心的公平性声明（正文应明确写出）

建议在实验节写一个短段落，明确声明：

1. 所有多智能体方法共享相同 agent scaffold
2. 所有方法共享相同 base model、检索接口和预算上限
3. 所有污染场景使用相同注入协议
4. Ours 的优势仅来自：
   - 自适应协议切换
   - contamination-aware belief
   - learned intervention policy

### 6.8 当前最推荐的主比较主线

如果只看论文最核心的叙事链，我建议你把结果组织成：

- **Step 1**：Single-Agent → MA-FreeText
  - 先说明多智能体带来了协作增益，也带来了传播风险
- **Step 2**：MA-FreeText → MA-SemiStruct → G2CP-Static
  - 说明通信约束确实重要
- **Step 3**：G2CP-Static / GUARDIAN-Heuristic → ClosedLoop-Heuristic
  - 说明统一闭环控制比单点预防或单点检测更强
- **Step 4**：ClosedLoop-Heuristic → Ours
  - 证明 learned controller 的必要性

## Belief Encoder 与 Policy Network 的最小实现方案

### 7.1 设计目标

这一部分的目标不是追求最强模型，而是给出一个**中成本、可解释、可稳定训练**的最小实现方案，使论文的核心贡献落在“闭环控制逻辑”而不是“大模型重训练”。

设计约束如下：

1. **不训练底层 LLM**
2. **不做 token-level policy learning**
3. **只做 round-level high-level control**
4. **state 以聚合特征为主，而非原始长文本**
5. **模型规模小，便于做消融与归因**

### 7.2 输入表示：controller 实际看到什么

控制器每一轮接收的不是完整对话，而是压缩后的协作状态 $z_t = [b_t; c_t]$。

其中建议拆成三层输入：

#### A. 风险观测特征（来自 risk observer）
- top-k node anomaly scores
- top-k edge contamination scores
- subgraph inconsistency score
- propagation depth / breadth
- disagreement intensity
- evidence support deficit

#### B. 协作上下文特征
- 当前轮次 $t$
- 当前通信模式（one-hot）
- 当前已使用干预次数
- 当前累计 token / latency
- 上一轮动作类型
- 当前参与 agent 数

#### C. 任务上下文摘要特征
- 任务类型（FEVER / HotpotQA / MATH / MMLU）
- 当前答案是否已收敛
- Verifier 的一致性分数
- 过去 1-2 轮答案变化幅度

这些特征构成一个定长向量，避免让控制器直接处理长文本。

### 7.3 belief encoder：最小可行实现

#### 推荐版本：GRU-based temporal belief encoder

最推荐的第一版实现是：

1. 对每轮观测特征做 MLP 投影
2. 用一个小型 GRU 聚合最近几轮观测历史
3. 输出 belief summary $b_t$

形式上可写成：

$$
h_t = \mathrm{GRU}(\psi(o_t), h_{t-1})
$$

$$
b_t = W_b h_t + b_b
$$

其中：

- $\psi(\cdot)$ 是观测特征投影 MLP
- $h_t$ 是时序隐状态
- $b_t$ 是 belief summary

#### 为什么优先选 GRU 而不是 Transformer

- 协作轮数本来就短（通常 2-4 轮）
- GRU 足够捕捉“异常是瞬时还是持续”的时间模式
- 参数更少，更容易训稳
- 更适合中成本 setting

#### 替代版本

##### 更简版本：MLP belief encoder
直接把最近两轮特征拼接后送入 MLP：

$$
b_t = \mathrm{MLP}([o_t; o_{t-1}; c_t])
$$

可作为低配消融。

##### 更强版本：Tiny Transformer encoder
若后续发现 3-5 轮上下文确实重要，可试一个 1-2 层小 Transformer。但不建议作为第一版主方法。

### 7.4 policy network：分层动作决策器

由于动作空间本身是分层的，policy network 也建议做成分层头，而不是单一 flat classifier。

#### 第一层：模式控制头
决定通信模式：

$$
\pi_m(m_t \mid z_t)
$$

输出三类动作概率：
- FREE
- SEMI-STRUCTURED
- GRAPH-GROUNDED

#### 第二层：恢复动作头
在给定当前 belief 和模式选择后，决定是否继续或恢复：

$$
\pi_u(u_t \mid z_t, m_t)
$$

输出动作概率：
- CONTINUE
- ISOLATE
- ROLLBACK
- REROUTE
- RETRIEVE-AGAIN
- RE-DEBATE
- STOP-AND-SYNTHESIZE

#### 第三层：目标选择头（可选）
仅当动作需要目标时启用，例如 ISOLATE(i)。

为控制复杂度，建议只在 top-k 可疑 agent 中选：

$$
\pi_g(g_t \mid z_t, u_t)
$$

其中 $g_t$ 表示目标 agent 或目标轮次。

### 7.5 最小网络结构建议

推荐一个非常保守、稳定的结构：

```text
Observation features
   ↓
MLP projection (2 layers, small hidden dim)
   ↓
GRU encoder (1 layer)
   ↓
Belief summary b_t
   ↓
Concat with context c_t
   ↓
Shared trunk MLP
   ├── Mode head
   ├── Action head
   └── Target head (optional)
```

建议超参数范围：

- observation projection dim: 64–128
- GRU hidden dim: 64–128
- shared trunk dim: 128–256
- 1-layer GRU 即可
- target head 仅在需要时启用

这能保证：
- 参数量小
- 训练稳定
- 易做消融
- 不会让 reviewer 觉得你靠大模型堆参数取胜

### 7.6 动作执行逻辑：policy 只做高层决策

必须强调：

> **policy network 不直接生成文本，也不直接修改 message 内容。**

它只做高层控制，例如：
- 当前轮是否升级到 GRAPH-GROUNDED
- 是否 re-debate
- 是否 isolate 当前最可疑 agent

具体文本生成仍由底层 LLM agents 完成。

这样做的好处：

1. 归因清晰：改进来自控制策略，而非 LLM 微调
2. 训练成本低：动作空间远小于 token 空间
3. 更符合论文叙事：high-level closed-loop controller

### 7.7 训练方式建议

#### 主推荐：offline policy learning

从带污染注入的轨迹中收集样本：

$$
\mathcal{D} = \{(z_t, a_t, r_t, z_{t+1})\}
$$

然后训练高层 policy。

适合的路线：

1. **Behavior cloning from heuristic teacher**
   - 先模仿一个强 heuristic controller
2. **Offline RL fine-tuning**
   - 再基于回报信号优化策略

这是最稳的中成本方案。

#### 备选：cost-sensitive contextual bandit

若想进一步简化，可把每轮决策近似成 contextual bandit：
- 输入当前 $z_t$
- 输出动作 $a_t$
- 以局部回报近似长期收益

这会损失部分长期时序性，但实现更简单，也适合作为附录对照。

### 7.8 关键消融对应的实现分支

为了让后续实验干净，建议一开始就按以下实现分支组织代码：

1. **No-Belief**
   - 去掉 GRU，仅用当前观测 + 上下文
2. **Heuristic Policy**
   - 保留同样输入与动作空间，但不用学习
3. **Flat Policy**
   - 不分模式头和动作头，直接单头分类
4. **No-Target Head**
   - 目标选择由 heuristic 决定

这会让消融实验在工程上很好落地。

### 7.9 明确不建议做的实现

以下实现虽然“看起来更强”，但当前阶段不建议做：

#### A. 端到端多智能体 RL
- 训练成本高
- credit assignment 极难
- 很容易训练不稳定
- 审稿时反而难归因

#### B. 让 controller 直接读原始长文本
- 输入太长
- 特征不稳定
- 训练更依赖 base model 表达
- 会弱化“显式风险状态”的贡献

#### C. 动作空间一开始就做太大
- 例如同时学 mode switch + fine-grained target + multi-step recovery chain
- 很容易变成难训、难解释、难消融

### 7.10 最小实现版本（建议写进论文的方法主线）

如果现在就锁定最稳版本，我建议论文主线写成：

> A lightweight GRU-based belief encoder summarizes temporal risk signals into a contamination-aware belief state, which is consumed by a hierarchical policy network to select communication modes and recovery interventions at the round level.

翻成更直接的话：

- belief encoder：小 GRU
- policy：分层动作头
- 学习：离线为主
- 控制粒度：轮级别
- 不训练底层 LLM

这就是当前资源条件下最稳的“最小可发表系统”。

## 论文图表清单（Figure / Table Plan）

### 8.1 图表规划原则

本工作不应把图表当作“实验做完再随手画出来的可视化”，而应把它们当作：

> **提前约束实验输出、直接承载论文 claim 的证据接口。**

因此每个图表都应明确：

1. 它支撑哪条核心 claim
2. 它最适合图还是表
3. 它在正文还是附录
4. 它最容易被误画成什么样

总原则：

- **主文只放高信息密度图表**，不堆表格
- **图负责模式感知与 trade-off 展示**
- **表负责精确数值、benchmark 主结果、消融细节**
- **每个主图都应能独立回答一个 reviewer 质疑**

### 8.2 正文主图清单

#### Figure 1. Closed-loop framework overview

**承载 claim**：本文不是模块拼装，而是统一的 closed-loop control framework。

**推荐形式**：系统框架图 / 流程图

**应包含元素**：
- task collaboration layer
- communication control layer
- risk observation layer
- belief state
- controller / recovery policy
- 回路箭头（feedback loop）

**图形角色**：整篇论文的“身份图”，最好放在方法部分第一页。

**常见陷阱**：
- 画成普通 pipeline，看不出闭环
- G2CP / GUARDIAN 被画成平级外挂模块，而不是 preventive action / observation mechanism

---

#### Figure 2. Temporal contamination and intervention timeline

**承载 claim**：系统处理的是动态风险传播，而不是单点纠错。

**推荐形式**：时间轴 + 传播图示意 / 多轮交互过程图

**应包含元素**：
- 多轮交互节点
- 错误传播路径
- risk signal 上升
- 某一轮触发协议升级或恢复动作
- 恢复后传播被切断或减弱

**图形角色**：帮助读者直观看到“错误如何扩散、控制器如何介入”。

**常见陷阱**：
- 只画静态图，不体现 time dimension
- 传播路径和恢复动作没有显式对应关系

---

#### Figure 3. Accuracy–Safety–Cost frontier

**承载 claim**：Ours 改善的是 trade-off，而不仅仅是单一准确率。

**推荐形式**：二维散点 / 气泡图 / 帕累托前沿图

**建议轴**：
- x 轴：cost（token 或 latency）
- y 轴：task accuracy
- 点颜色 / 形状：unsafe pass rate 或 propagation rate

**图形角色**：这是最重要的结果图之一，应当体现：
- Single-Agent
- MA-FreeText
- MA-SemiStruct
- G2CP-Static
- GUARDIAN-Heuristic
- ClosedLoop-Heuristic
- Ours

**常见陷阱**：
- 同时塞太多维度导致看不懂
- 只报平均值，不显示不同任务场景的变化
- 没有强调 Ours 是 Pareto-improving 而不是仅仅昂贵更强

---

#### Figure 4. Action usage and intervention composition

**承载 claim**：learned controller 不是黑箱乱动，而是学到了有结构的干预策略。

**推荐形式**：堆叠柱状图 / 分组柱状图

**应展示**：
- 不同任务或不同污染场景下
- FREE / SEMI-STRUCTURED / GRAPH-GROUNDED 的使用占比
- isolate / rollback / re-debate / re-retrieve 的触发频率

**图形角色**：解释 learned controller 的行为模式。

**常见陷阱**：
- 只给平均动作频率，看不出场景依赖
- 不区分 clean vs contaminated episodes

---

#### Figure 5. Case study: one failure trajectory and its recovery

**承载 claim**：belief-guided closed-loop intervention 在真实轨迹上可解释。

**推荐形式**：案例图（多 panel）

**建议 panel**：
- (a) free-text failure propagation
- (b) risk signal / belief over rounds
- (c) selected intervention
- (d) corrected final outcome

**图形角色**：给审稿人一个“具体发生了什么”的抓手。

**常见陷阱**：
- 案例太长、字太多
- 只展示成功案例，不展示为何基线失败

### 8.3 正文主表清单

#### Table 1. Main benchmark results

**承载 claim**：完整方法在主 benchmark 上整体最优或 trade-off 最优。

**推荐形式**：主结果表

**列建议**：
- 方法
- FEVER
- HotpotQA / 2Wiki
- MATH
- MMLU-subset
- Avg.
- Token cost
- Unsafe pass rate

**角色**：正文最核心表格，用于精确数值对比。

**常见陷阱**：
- 指标过多导致横向拥挤
- 将所有安全指标全塞进主表，破坏可读性

---

#### Table 2. Ablation study

**承载 claim**：belief、learning、graph-grounded mode、recovery action 各自都有必要性。

**推荐形式**：消融表

**建议行**：
- Ours
- w/o belief
- w/o learning
- w/o graph-grounded mode
- w/o recovery actions
- flat policy

**建议列**：
- Accuracy
- Propagation rate
- Recovery success
- Over-intervention
- Token cost

**角色**：证明方法不是“哪个模块都可以删”。

**常见陷阱**：
- 消融版本定义不精确
- 只报 accuracy，不报安全与成本

---

#### Table 3. Baseline fairness summary

**承载 claim**：比较是公平的，优势不来自 scaffold / prompt 偏置。

**推荐形式**：协议与预算对照表

**建议列**：
- Method
- Same agent scaffold?
- Same tools?
- Same budget cap?
- Communication type
- Detection?
- Recovery?
- Learned policy?

**角色**：这张表在正文或附录都很有价值，能大幅减少“unfair comparison”质疑。

**常见陷阱**：
- 只在文字里解释公平性，不给结构化表格

### 8.4 附录图表清单

#### Figure A1. Natural failure family mapping

把自然失败轨迹与三类污染注入场景对齐，回应“评测太人造”的攻击。

#### Figure A2. Belief vs anomaly over time

展示 belief state 与直接 anomaly score 的差异，支撑 latent contamination-aware 表示的必要性。

#### Figure A3. Per-task frontier plots

把主文中的综合 frontier 拆成按任务分图，避免主文过载。

#### Table A1. Prompt / protocol template summary

列出各 baseline 的 prompt/protocol 差异边界。

#### Table A2. Action success breakdown

统计不同恢复动作在不同污染场景中的成功率。

### 8.5 图表与 claim 的绑定关系

| Artifact | Primary claim supported |
|---|---|
| Figure 1 | 这是统一闭环框架，不是简单拼装 |
| Figure 2 | 错误传播是动态风险问题 |
| Figure 3 | Ours 优化的是 accuracy–safety–cost trade-off |
| Figure 4 | learned controller 学到了结构化干预模式 |
| Figure 5 | 方法在具体轨迹上可解释 |
| Table 1 | 主 benchmark 上整体有效 |
| Table 2 | 各组件各有必要 |
| Table 3 | 比较公平可信 |

### 8.6 当前最值得优先保证的数据产出

如果实验资源有限，优先确保能产出下列图表的数据：

1. **Table 1 主结果表**
2. **Figure 3 frontier 图**
3. **Table 2 消融表**
4. **Figure 5 案例研究图**

这是最小可发表核心包。

### 8.7 常见设计陷阱

#### 陷阱 1：所有东西都做成柱状图
会让论文显得平庸，也不利于展示 trade-off。

#### 陷阱 2：把精确数值和模式感知塞到同一个 artifact
结果通常是又难看又难读。应当图表分工明确。

#### 陷阱 3：只画“平均正确率”
这会掩盖论文真正的贡献：安全-成本-正确率三者之间的控制能力。

#### 陷阱 4：案例图没有基线对照
case study 若只展示 Ours，会像宣传图而不是科学证据。

#### 陷阱 5：消融图表不报告成本副作用
如果删掉模块后更便宜但略差，必须把 trade-off 讲清楚。

## 摘要 / 引言的 reviewer-facing 叙事版本

### 9.1 目标叙事风格

这篇论文更适合采用 **NeurIPS / ICLR 主线 + ACL 的 communication / reasoning 表达习惯** 的混合风格：

- 像 NeurIPS / ICLR 一样，把问题提升到**统一建模层**
- 像 ACL 一样，把多智能体通信、推理错误传播和具体 failure mode 讲清楚

因此摘要和引言的核心任务不是“介绍三个模块”，而是：

> **让审稿人先接受这是一个新的问题 framing，再接受你的方法。**

### 9.2 必须避免的叙事错误

#### 错误 1：以“我们结合了 A + B + C”开头
这会立刻触发模块拼装印象。

#### 错误 2：把重点写成“我们降低了 hallucination”
这太泛，无法体现你的真正创新——closed-loop risk control。

#### 错误 3：贡献点写成模块列表
例如：
- 我们用了 G2CP
- 我们用了 GUARDIAN
- 我们用了 RL

这会让 reviewer 默认你没有统一原理。

### 9.3 推荐的标题气质

标题应突出：
- closed-loop
- safe / risk-aware
- multi-agent reasoning / collaboration
- prevent / detect / recover 之一或其统一视角

候选风格：

1. **Closed-Loop Risk Control for Safe LLM Multi-Agent Reasoning**
2. **Learning to Prevent, Detect, and Recover from Error Propagation in LLM Multi-Agent Systems**
3. **Belief-Guided Closed-Loop Control for Safe Multi-Agent Reasoning**
4. **Adaptive Communication and Recovery Control for Safe LLM Multi-Agent Collaboration**

其中第 1 / 3 个最适合偏 NeurIPS / ICLR 的 framing。

### 9.4 摘要结构建议

推荐使用 5 句式结构：

#### 句 1：问题与重要性
指出多智能体系统的协作优势，同时点出新的 failure mode：错误会在 agent 间传播、放大、级联。

#### 句 2：现有方法的局限
说明现有工作分散在协议约束、异常检测或自检上，但缺少统一闭环控制。

#### 句 3：核心 framing
提出：
> 我们将错误传播建模为一个部分可观测的动态风险控制问题。

#### 句 4：方法概述
说明：
- 结构化通信用于预防
- 时序图观测器用于检测
- belief-guided controller 用于恢复与策略学习

#### 句 5：结果与价值
强调不是只提高 accuracy，而是改善 accuracy–safety–cost trade-off。

### 9.5 摘要草案（第一版）

> Large language model (LLM) multi-agent systems can improve problem solving through collaboration, but they also introduce a new class of failures: incorrect intermediate reasoning can propagate, amplify, and cascade across agents. Existing defenses are fragmented across communication constraints, anomaly detection, and self-checking, leaving the system without a unified mechanism for deciding when to prevent risky exchanges, when to intervene, and how to recover once contamination has spread. We frame error propagation in LLM multi-agent reasoning as a **partially observable closed-loop risk control problem**. Our framework combines adaptive communication constraints, temporal graph-based risk observation, and a belief-guided high-level controller that selects recovery interventions such as isolation, rollback, re-retrieval, and re-debate. Rather than optimizing only final-task accuracy, our method explicitly balances correctness, safety, and interaction cost. We show that this closed-loop formulation yields a stronger accuracy–safety–cost trade-off than static communication protocols, detection-only heuristics, and fixed recovery rules across multi-agent reasoning benchmarks.

这版摘要故意先强调 framing，不先强调模块来源。

### 9.6 引言 opening move 建议

推荐的 opening move 不是从 hallucination 一般问题讲起，而是从：

> **协作带来收益，但也带来传播性失败。**

一个更好的 opening 模式是：

1. 多智能体 LLM 系统越来越强
2. 但协作和单体不同，错误不再是局部的，而是会在交互网络中传播
3. 这使得问题从“如何让单个模型少幻觉”变成“如何控制协作过程中的风险传播”

这个 opening 的好处是：
- 直接建立你的问题独特性
- 自然引出 closed-loop control framing
- 避免掉入“又是一篇 hallucination mitigation paper”的印象

### 9.7 引言四段主线建议

#### Paragraph 1：问题提出
讲协作收益与传播风险并存。

目标句型：
- Multi-agent LLM systems can outperform single agents on complex tasks, but their gains come with a structural vulnerability: intermediate errors can spread through communication and distort downstream reasoning.

#### Paragraph 2：现有方法不足
把现有方法分成三类：
- communication constraint
- anomaly detection
- self-check / verification

然后指出：
- 它们各自解决一部分问题
- 但系统仍缺少一个统一机制来决定何时预防、何时恢复、何时继续协作

#### Paragraph 3：本文核心 idea
在这一段第一次明确提出：

> We cast multi-agent error propagation as a partially observable closed-loop risk control problem.

然后用一句话解释三层对应关系：
- protocol constraint as preventive action
- temporal graph observation as risk sensing
- learned intervention as recovery control

#### Paragraph 4：贡献概述
不要按模块列贡献，而要按研究价值列贡献。

### 9.8 推荐的 contribution 写法

建议写成下面这种 reviewer-friendly 形式：

1. **Problem reframing**
   - We formulate error propagation in LLM multi-agent reasoning as a partially observable closed-loop risk control problem.

2. **Unified framework**
   - We introduce a framework that unifies preventive communication constraints, temporal risk observation, and adaptive recovery interventions.

3. **Belief-guided controller**
   - We develop a lightweight belief-guided high-level controller that learns when to switch communication modes and when to trigger recovery actions.

4. **Empirical evidence**
   - We show that the resulting system improves the accuracy–safety–cost trade-off over static protocols, detection-only methods, and heuristic recovery strategies.

这样的好处是：
- 第一条先立问题
- 第二条立统一性
- 第三条立方法特色
- 第四条立实验价值

### 9.9 如何写 gap，避免“别人都做了”的感觉

错误写法：
- prior work has not combined G2CP and GUARDIAN

这太弱，因为“没组合过”不等于有研究价值。

推荐写法：

> Prior work addresses pieces of the problem—by constraining communication, detecting anomalous interactions, or improving self-verification—but still lacks a system-level mechanism that adaptively decides **when** to prevent risky communication, **when** to intervene, and **how** to recover under limited interaction budgets.

这个 gap 的好处：
- 不是组合缺口
- 而是**决策机制缺口**

### 9.10 引言第一版骨架

下面给出一版可直接后续扩写的 introduction skeleton：

#### P1
Large language model (LLM) multi-agent systems have emerged as a promising way to improve reasoning and decision making through collaboration. By decomposing tasks across specialized agents, such systems can outperform single-agent prompting on complex question answering, verification, and planning tasks. However, collaboration also introduces a structural vulnerability that is absent or weaker in single-agent settings: incorrect intermediate conclusions can propagate across agent interactions, distort downstream reasoning, and eventually lead to cascaded failure.

#### P2
Recent work has attacked this problem from several directions. Some methods reduce ambiguity by constraining how agents communicate, while others detect anomalous interaction patterns or strengthen self-checking and verification. These advances are important, but they remain fragmented. A multi-agent system still lacks a unified mechanism for deciding when communication should remain flexible, when it should be constrained, when suspicious trajectories require intervention, and which recovery action is most appropriate under limited interaction budgets.

#### P3
We argue that this is fundamentally a **closed-loop risk control** problem. In multi-agent reasoning, the system never observes contamination directly; instead, it must infer latent risk from interaction patterns, evidence consistency, and temporal disagreement signals, and then choose whether to continue, constrain communication, or trigger recovery. Based on this view, we formulate error propagation in LLM multi-agent reasoning as a **partially observable closed-loop control problem** and develop a framework that combines preventive communication constraints, temporal graph-based risk observation, and belief-guided adaptive recovery.

#### P4
Our method uses structured communication as a preventive action, temporal graph signals as a risk observation mechanism, and a lightweight high-level controller to select interventions such as isolation, rollback, re-retrieval, and re-debate. This design does not retrain the underlying LLM agents; instead, it learns a compact control policy over the collaboration process itself. Across multi-agent reasoning benchmarks, we show that this formulation improves the accuracy–safety–cost trade-off over static communication protocols, detection-only heuristics, and fixed recovery rules.

### 9.11 引言收束句风格建议

引言末尾的收束句建议避免写成：
- “The rest of the paper is organized as follows.”

更推荐 reviewer-facing 收束：

> Together, these results suggest that safe multi-agent reasoning should be studied not only as a matter of better prompts or stronger detectors, but as a problem of adaptive closed-loop control over collaborative inference.

这句话有助于把论文主张钉死。

## 下一步待补

- 第一版实验执行清单 / project plan

## 相关页面

- [[raw/lessons/G2CP/00-essence]]
- [[raw/lessons/GUARDIAN/00-essence]]
- [[raw/lessons/MARCH/00-essence]]
- [[research/02-papers/multi-agent-hallucination-reading-guide]]
- [[raw/lessons/Research-to-TopConf/05-direction-selection]]
