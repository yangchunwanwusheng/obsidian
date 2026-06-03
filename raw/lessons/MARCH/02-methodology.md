---
type: lesson
tags: [MARCH, 翻译笔记, 方法论, 多智能体强化学习, 信息不对称, PPO]
created: 2026-05-16
updated: 2026-05-16
sources:
  - MARCH_2026.pdf
difficulty: advanced
prerequisites:
  - "[[raw/lessons/MARCH/00-essence]]"
  - "[[raw/lessons/MARCH/01-introduction]]"
topic: MARCH — 方法论
status: completed
---

# MARCH — 2. 方法论 (Methodology)

> 三智能体流水线（Solver-Proposer-Checker）的数学形式化、信息不对称机制与联合 PPO 优化。

## 正文翻译

### 2.1 问题形式化与概述

给定输入查询 $\mathbf{x}\in\mathcal{X}$ 和一组检索文档 $\mathbf{D}=[\mathbf{d}_1, \mathbf{d}_2, \dots, \mathbf{d}_l]$，我们的目标是优化策略模型 $\pi_\theta(\mathbf{y}|\mathbf{x}, \mathbf{D})$，使其生成与 $\mathbf{D}$ 中证据最大程度事实对齐的回答 $\mathbf{y}$。

形式化地，假设我们可以使用函数 $\mathcal{A}(\cdot)$ 从回答 $\mathbf{y}$ 中提取所有事实陈述 $\{\mathbf{a}_1, \mathbf{a}_2, \dots, \mathbf{a}_n\}$，即 $\mathcal{A}(\mathbf{y})=\{\mathbf{a}_i\}_{i=1}^n$。那么，从带可验证奖励的强化学习（RLVR）角度，如果对每个原子陈述 $\mathbf{a}_i$ 都有一个理想的真值标注集合 $\mathcal{A}^*=\{\mathbf{a}^*_1, \dots, \mathbf{a}^*_n\}$，优化目标是最大化期望证据一致性：

$$\max_{\theta}\mathbb{E}_{\mathbf{x}\sim\mathcal{X},\mathbf{y}\sim\pi_\theta(\cdot|\mathbf{x},\mathbf{D})}[R(\{\mathbf{a}_i,\mathbf{a}^*_i\}_{i=1}^n)]$$

其中 $R(\cdot)$ 是事实检查函数，识别 $\mathcal{A}(\mathbf{y})=\{\mathbf{a}_i\}_{i=1}^n$ 与 $\{\mathbf{a}^*_i\}_{i=1}^n$ 之间的精确匹配。然而，$\mathcal{A}^*$ 在大多数应用领域中实际上是**不可获得的**，需要大量人工标注。

因此，我们将上述等式重构为一个**自包含的、可验证的目标**，将锚定过程分解为三个独立的智能体任务：

1. **回答生成**：Solver 智能体基于输入查询 $\mathbf{x}$ 和检索上下文 $\mathbf{D}$ 生成回答 $\mathbf{y}$。
2. **声明级 QA 提议**：Proposer 智能体将回答 $\mathbf{y}$ 分解为 $n$ 个离散事实声明，以问答（QA）格式呈现：$\mathcal{Q}(\mathbf{y})=\{(\mathbf{q}_i, \mathbf{a}_i)\}_{i=1}^n$。
3. **事实验证**：Checker 智能体回答所有原子问题 $\{\mathbf{q}_i\}_{i=1}^n$，得到 $\{\hat{\mathbf{a}}_i\}_{i=1}^n$，**仅提供检索文档 $\mathbf{D}$，不接触原始回答 $\mathbf{y}$ 和提取的答案 $\{\mathbf{a}_i\}_{i=1}^n$**。

通过用多智能体流水线的锚定结果 $\hat{\mathbf{a}}_i$ 替代不可达的真值 $\mathbf{a}^*_i$，我们将事实性优化近似为：

$$\max_{\theta}\mathbb{E}_{\mathbf{x}\sim\mathcal{X},\mathbf{y}\sim\pi_\theta(\cdot|\mathbf{x},\mathbf{D})}\left[R(\{(\mathbf{a}_i,\hat{\mathbf{a}}_i)\}_{i=1}^n)\right]$$

通过优化此等式，我们强制 $\pi_\theta$ 内化严格的验证逻辑，确保生成内容严格锚定在源证据中，以增强整体事实准确性。

> [!tip] 理解提示（非原文）
> 核心思想：用"另一个智能体基于文档的独立回答"替代"人类标注的真值"。这本质上是**自博弈**（self-play）思想在事实对齐中的应用——让同一个模型扮演不同角色，通过内部对抗来提升事实性。

### 2.2 MARCH 建模

如图 1 所示，我们将上述功能原语实例化为三个从同一基座策略 $\pi_\theta$ 派生的专门智能体：Solver、Proposer 和 Checker。它们被条件化为表现出不同的认知行为，有效地将 RAG 任务转化为一个**结构化的非对称合作博弈**。

**Solver 策略：**

$$\nu_{\text{solve}}(\cdot|\mathbf{x},\mathbf{D})=\pi_\theta(\cdot|\mathbf{x},\mathbf{D},\mathbf{s}_{\text{solve}})$$

其中 $\mathbf{s}_{\text{solve}}$ 是 Solver 的系统提示，诱导策略模型 $\pi_\theta$ 基于检索文档 $\mathbf{D}$ 和输入 $\mathbf{x}$ 生成综合回答 $\mathbf{y}$。

**Proposer 策略：**

$$\nu_{\text{propose}}(\cdot|\mathbf{y})=\pi_\theta(\cdot|\mathbf{y},\mathbf{s}_{\text{propose}})$$

Proposer 作为"回答原子化器"，将连续回答 $\mathbf{y}$ 解耦为可验证组件。$\nu_{\text{propose}}$ 将回答分解为一组自包含的事实问答对 $\mathcal{Q}(\boldsymbol{\tau})=\{(\mathbf{q}_i, \mathbf{a}_i)\}_{i=1}^n$，其中每个 $\mathbf{q}_i$ 代表一个针对性查询，每个 $\mathbf{a}_i$ 表示 $\mathbf{y}$ 中主张的对应声明。

**Checker 策略：**

$$\nu_{\text{check}}(\cdot|\{\mathbf{q}_i\}_{i=1}^n,\mathbf{D})=\pi_\theta(\cdot|\{\mathbf{q}_i\},\mathbf{D},\mathbf{s}_{\text{check}})$$

Checker 作为"盲审员"，是非对称合作的**关键枢纽**。为消除确认偏误并打破信息对称性，当回答 $\mathbf{q}_i$ 时，$\nu_{\text{check}}$ 严格对 Solver 的回答 $\mathbf{y}$ 和对应声明 $\mathbf{a}_i$ **保持盲态**。Checker 仅基于 RAG 文档 $\mathbf{D}$ 回复所有查询 $\{\mathbf{q}_i\}_{i=1}^n$。

> [!warning] 易混点（非原文）
> 注意 Proposer 不参与 RL 梯度更新——它只在训练中做推理（生成 QA 对），其输出被用作 Checker 和 Solver 之间的桥梁。实际上只有 Solver 和 Checker 的轨迹被纳入 PPO 梯度计算。

**整体目标。** 通过整合 Solver、Proposer 和 Checker 的条件依赖关系，MARCH 定义了一个统一的非对称合作博弈目标，聚焦于 Solver 的回答 $\mathbf{y}$ 与 Checker 的独立锚定轨迹 $\boldsymbol{\lambda}$ 之间的证据同一性，由 Proposer 的原子声明分解作为中介：

$$\max_{\pi_\theta}\ \mathbb{E}_{\mathbf{x}\sim\mathcal{X},\mathbf{y}\sim\nu_{\text{solve}}(\cdot|\mathbf{x},\mathbf{D})}\left[R(\{(\mathbf{a}_i,\hat{\mathbf{a}}_i)\}_{i=1}^n)\right]$$

$$\text{s.t. } \{(\mathbf{q}_i,\mathbf{a}_i)\}_{i=1}^n=\mathcal{Q}(\boldsymbol{\tau}),\ \boldsymbol{\tau}\sim\nu_{\text{propose}}(\cdot|\mathbf{y})$$

$$\{\hat{\mathbf{a}}_i\}_{i=1}^n=\mathcal{A}(\boldsymbol{\lambda}),\ \boldsymbol{\lambda}\sim\nu_{\text{check}}(\cdot|\{\mathbf{q}_i\}_{i=1}^n,\mathbf{D})$$

其中 $R(\cdot)$ 衡量每个主张声明 $\mathbf{a}_i$ 与独立锚定 $\hat{\mathbf{a}}_i$ 之间的证据同一性。通过优化此目标，策略 $\pi_\theta$ 被迫内化严格的验证逻辑，确保其叙述合成严格锚定在可验证证据中，从而增强整体事实准确性。

### 2.3 MARCH 实现

在我们的实现中，我们特别优先考虑**数值和定量验证**，这是基于以下观察：在数据密集型 RAG 场景中，LLM 经常遭受引用数字错误或在上下文整合过程中改变数值等幻觉问题。通过将验证循环锚定在这些高保真标记上，MARCH 为锚定长形式推理提供了更严格的证据基线。

**零容忍奖励（Zero-Tolerance Reward, ZTR）。** 为强制绝对事实锚定，我们定义了一个二元零容忍奖励函数。在 RAG 上下文中，我们主张回答 $\mathbf{y}$ 中任何单个幻觉声明 $\mathbf{a}_i$ 都会使整个推理链失效。因此，只有当 Solver 主张的每个声明 $\mathbf{a}_i$ 与 Checker 提取的共识答案 $\hat{\mathbf{a}}_i$ 达到同一性匹配时，轨迹才被赋予成功分数：

$$R(\{(\mathbf{a}_i,\hat{\mathbf{a}}_i)\}_{i=1}^n)=\begin{cases}0, & \text{if every } \mathbf{a}_i = \hat{\mathbf{a}}_i \\ -1, & \text{otherwise}\end{cases}$$

这种"全有或全无"的奖励结构防止策略 $\pi_\theta$ 优化部分正确性或风格上的合理性，强制模型优先考虑严格的证据同一性。此外，为确保 Checker 提供可靠的审计基线并减少奖励信号的方差，我们在交叉验证阶段采用**多样本策略**。对于 Proposer 生成的每个问题 $\mathbf{q}_i$，$\nu_{\text{check}}$ 生成多个独立审计轨迹。我们应用多数投票来确定最终共识答案 $\hat{\mathbf{a}}_i$，有效缓解了 Checker 单次独立审计中的随机错误。

> [!note] 译者注（非原文）
> ZTR 的设计思想类似于软件工程中的"fail-fast"原则——一个 bug 就让整个测试失败，而非按比例扣分。这确保模型不能通过"大部分正确"来获得高奖励。

**策略优化。** 我们使用近端策略优化（PPO）来优化三个智能体的共享策略 $\pi_\theta$。关键的是，我们将推理路径 $\mathbf{y}$ 和审计轨迹 $\boldsymbol{\lambda}$ 都纳入优化循环，确保 $\pi_\theta$ 同时作为可靠的生成器和严格的审计器进化。

因此，训练批次 $\mathcal{B}$ 中的每个原始查询 $\mathbf{x}_i$ 贡献两条不同的奖励信号给策略，允许 $\pi_\theta$ 在统一训练批次内学习叙述合成与独立文档锚定之间的细粒度对应关系。虽然两条轨迹的终端奖励来自相同的证据同一性匹配，但对应的优势 $\hat{A}$ 和 KL 散度惩罚是独立估计的。联合梯度计算为所有单个轨迹梯度的总和：

$$\nabla_\theta \mathcal{L}_{\text{MARCH}}(\theta) \approx \frac{1}{2|\mathcal{B}|}\sum_{\mathbf{x}\in\mathcal{B}}\Bigg[\sum_{t=1}^{|\mathbf{y}|}\hat{A}_{y}^{t}\nabla_\theta\log\pi_\theta(y^t|\cdot)+\sum_{t=1}^{|\boldsymbol{\lambda}|}\hat{A}_{\lambda}^{t}\nabla_\theta\log\pi_\theta(\lambda^t|*)-\beta\nabla_\theta\text{KL}[\pi_\theta\|\pi_\text{ref}]\Bigg]$$

其中 $\hat{A}_{\lambda}^{t}$ 和 $\hat{A}_{y}^{t}$ 是 Checker 输出 $\boldsymbol{\lambda}$ 和 Solver 输出 $\mathbf{y}$ 的第 $t$ 个 token 对应的优势，通过广义优势估计（GAE）估计。

$(\cdot)$ 和 $(*)$ 封装了各自的上下文：$(\mathbf{x},\mathbf{D},\mathbf{y}^{<t})$ 和 $(\{\mathbf{q}_i\},\mathbf{D},\boldsymbol{\lambda}^{<t})$ 分别对应 Solver 和 Checker 的生成前缀。此公式确保 $\pi_\theta$ 同时精炼其叙述生成和证据精度，内化一个自包含的验证循环，其中生成性声明本质上与策略自身的审计能力相系。

### Algorithm 1: MARCH 训练过程

```
输入: LLM 策略 π_θ; 训练批次 B; Solver, Proposer, Checker 系统提示

// Phase 1: 多智能体协作与展开
for 每个 (x, D) ∈ B do
    // Step 1: Solver 生成推理路径 y
    y ~ ν_solve(·|x, D) = π_θ(·|x, D, s_solve)

    // Step 2: Proposer 原子化回答 y，解析 QA 对
    τ ~ ν_propose(·|y) = π_θ(·|y, s_propose)
    提取 {(q_i, a_i)} = Q(τ)

    // Step 3: Checker 生成审计轨迹 λ
    λ ~ ν_check(·|{q_i}, D) = π_θ(·|{q_i}, D, s_check)
    解析重新检查的答案 {â_i} = A(λ)

    // Step 4: 统一奖励标注
    计算 ZTR R_i（基于 a 与 â 的匹配）

    // Step 5: 存储推理和审计两条轨迹
    将 y_i 和 λ_i 添加到统一批次轨迹集
    两条轨迹都关联终端奖励 R_i
end for

// Phase 2: 联合策略优化
for 每条轨迹 τ' ∈ T_B = {y_i, λ_i} do
    基于 R_i 使用 GAE 估计优势 Â_τ'
    计算每 token 的 KL(π_θ || π_ref)
end for
根据联合梯度公式更新 π_θ
```

> [!tip] 理解提示（非原文）
> 注意训练过程的两个阶段：Phase 1 是多智能体协作（每个样本跑三次前向推理），Phase 2 是统一梯度更新（将 Solver 和 Checker 的轨迹一起优化）。关键设计是同一奖励 $R_i$ 同时反传给两条轨迹，使得 Solver 学会"生成可验证的回答"，Checker 学会"严格基于文档验证"。

## 关键要点

1. **问题重构**：将"有真值标注的事实对齐"转化为"自包含的三智能体验证"——用 Checker 的独立回答替代不可获得的真值
2. **信息不对称是核心创新**：Checker 严格看不到 Solver 的输出，打破确认偏误循环
3. **ZTR 的"全有或全无"设计**：任何声明不匹配即惩罚整条轨迹，防止部分正确性优化
4. **双轨迹联合优化**：同一奖励信号同时反传给 Solver 和 Checker，实现生成与审计能力的共同进化
5. **Proposer 只做推理**：只有 Solver 和 Checker 的轨迹进入 PPO 梯度，Proposer 作为桥梁角色

## 相关页面

- [[raw/lessons/MARCH/00-essence|MARCH 精华总结]]
- [[raw/lessons/MARCH/01-introduction|01 — 引言]]
- [[raw/lessons/MARCH/03-experiments|03 — 实验]]
