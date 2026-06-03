---
type: lesson
tags: [G2CP, 协议设计, 知识图谱, 形式化方法, 操作语义, 承诺语义, FIPA-ACL, 论文翻译]
created: 2026-05-16
updated: 2026-05-16
sources:
  - arXiv:2602.13370
difficulty: advanced
prerequisites:
  - "[[raw/lessons/G2CP/02-related-work|G2CP 02-相关工作]]"
  - "[[wiki/concepts/knowledge-graph]]"
topic: G2CP 协议定义翻译
status: completed
series:
  name: G2CP
  part: 3
---

# G2CP 论文翻译 — 第3节 The G²CP Protocol（协议定义）

> G²CP 的数学基础：知识图上的确定性图操作，配合 7 种语言行为和完整的社会承诺语义。

## 正文翻译

### 3.1 Formal Definition（形式化定义）

设 $G = (V, E, \Lambda, \Psi)$ 为一个异构有向知识图，其中：

- $V = \{v_1, \ldots, v_n\}$ 为顶点（实体）集合
- $E \subseteq V \times V$ 为有向边（关系）集合
- $\Lambda = \{\lambda_1, \ldots, \lambda_k\}$ 为节点类型集合
- $\Psi = \{\psi_1, \ldots, \psi_m\}$ 为边类型集合

每个节点 $v_i$ 具有：
- 类型 $\lambda(v_i) \in \Lambda$
- 稠密向量嵌入 $\mathbf{x}_i \in \mathbb{R}^d$（通过句子编码器对节点文本描述计算，用于语义相似性搜索）
- 属性 $A_i$（键值字典，存储如 `{name: "Bearing B-4521", serial: "SN-9042", install_date: "2021-03-15"}` 等结构化属性）

每条边 $e_{ij} = (v_i, v_j)$ 具有：
- 类型 $\psi(e_{ij}) \in \Psi$
- 权重 $w_{ij} \in [0, 1]$
- 时间戳 $t_{ij}$

> [!tip] 理解提示（非原文）
> 注意每个节点都有向量嵌入，这至关重要。它使得 G²CP 可以进行语义搜索（如"找到与'压力下降'最相似的节点"），而不仅仅依赖精确匹配。嵌入是 G²CP 连接符号操作和神经检索的桥梁。

一条 **G²CP 消息**是一个元组：

$$m = \langle sender, receiver, perf, op, ctx \rangle \tag{1}$$

其中：
- $sender, receiver \in \mathcal{A}$（智能体标识符）
- $perf \in \mathcal{P}$（语言行为，来自表 2）
- $op$ 是图操作（定义 3.1）
- $ctx = (conv\_id, G_{focus})$，其中 $G_{focus} \subseteq G$ 是当前对话子图

> [!note] 译者注（非原文）
> $G_{focus}$（对话焦点子图）是一个精妙的设计。随着对话进行，焦点子图逐步扩展，使后续操作可以在已有结果上增量构建，避免从头遍历。这类似于人类对话中的"共同基础（common ground）"概念。

**直觉**：一条 G²CP 消息是两个共享同一数据库的同事之间的指令条。语言行为说明正在执行什么类型的言语行为（询问、告知、提议），操作说明涉及哪些确切数据。因为两个智能体看到同一个图，所以内容误解的空间为零。

**定义（图操作）**：图操作 $op$ 有以下两种形式之一：

**(1) 遍历（Traversal）**：

$$op = \texttt{TRAVERSE}(V_s, \Psi_f, h, ret)$$

其中：
- $V_s \subseteq V$：源节点集
- $\Psi_f \subseteq \Psi$：边类型过滤器
- $h \in \mathbb{N} \cup \{\infty\}$：跳数深度
- $ret \in \{\texttt{SUBGRAPH}, \texttt{PATHS}, \texttt{LEAVES}\}$：返回类型

**(2) 更新（Update）**：

$$op = \texttt{UPDATE}(\Delta G)$$

其中 $\Delta G = (\Delta V^+, \Delta V^-, \Delta E^+, \Delta E^-)$ 指定要添加/删除的节点和边。

### 3.2 Operational Semantics（操作语义）

**直觉**：遍历从一组源节点出发，沿指定类型的边向外"行走"，收集给定跳数内可达的所有内容。返回格式控制智能体获得的是完整的探索子图、仅端点还是具体的遍历路径。

遍历操作的形式化语义：

$$\mathcal{T}: V_s \times \Psi_f \times \mathbb{N} \times \{\texttt{SUBGRAPH}, \texttt{PATHS}, \texttt{LEAVES}\} \rightarrow 2^{V \times E} \tag{2}$$

递归计算如下：

$$\mathcal{T}(V_s, \Psi_f, h, ret) = \begin{cases} extract(V_s, \emptyset) & \text{if } h = 0 \\ \mathcal{T}(V_s \cup N(V_s, \Psi_f), \Psi_f, h-1, ret) & \text{otherwise} \end{cases} \tag{3}$$

其中过滤邻域函数：

$$N(V_s, \Psi_f) = \{v_j \mid \exists v_i \in V_s, (v_i, v_j) \in E, \psi(v_i, v_j) \in \Psi_f\}$$

计算从源节点集沿指定类型边可达的邻居节点，`extract` 函数根据 `ret` 格式化结果。

> [!tip] 理解提示（非原文）
> 这个递归定义非常优雅，实质上是广度优先搜索（BFS）：
> - 基础情况 $h = 0$：不遍历，只返回源节点本身
> - 递归情况：将当前边界扩展一跳（$V_s \cup N(V_s, \Psi_f)$），然后递归
>
> 关键点：边类型过滤 $\Psi_f$ 使得遍历只沿特定类型的关系进行，这是智能体专业化的基础。

更新操作变换图：

$$G' = \texttt{UPDATE}(G, \Delta G) = (V', E', \Lambda, \Psi) \tag{4}$$

其中：
- $V' = (V \cup \Delta V^+) \setminus \Delta V^-$
- $E' = (E \cup \Delta E^+) \setminus \Delta E^-$

> [!warning] 易混点（非原文）
> 注意 $\Lambda$ 和 $\Psi$ 在 UPDATE 操作中不变——更新操作不改变节点类型集和边类型集，只改变具体的节点和边实例。这是一个安全性设计：图的模式（schema）是不可变的，只有数据是可变的。

### 3.3 Protocol Properties（协议属性）

**定理（确定性，Determinism）**：对于固定的图状态 $G$ 和操作 $op$，执行 $op$ 的结果是确定性的，不依赖于智能体实现。

*证明概要*：遍历操作是具有确定性邻域函数的递归集合扩展。更新操作是集合论变换。两者都不涉及随机过程。

**定理（可审计性，Auditability）**：任何通过 G²CP 可达的智能体结论，都可以通过在图上重放消息序列来验证。

*证明概要*：每条消息指定了显式的图操作。给定初始状态 $G_0$ 和消息序列 $\langle m_1, \ldots, m_k \rangle$，我们可以重构图状态 $G_0, G_1, \ldots, G_k$ 并验证一致性。

**定理（完备性，Completeness）**：G²CP 可以表达任何通过图遍历和检索增强生成可回答的查询。

*证明概要*：任何基于图的推理任务分解为：节点选择、边遍历和子图检索——每个都映射到遍历操作。复杂查询通过消息序列组合。

> [!tip] 理解提示（非原文）
> 三个属性的实际含义：
> - **确定性**：同样的图 + 同样的操作 = 同样的结果，不管谁来执行。这消除了 LLM 的非确定性问题。
> - **可审计性**：给定完整的消息日志，任何第三方都可以独立验证每一步推理是否正确。这在安全关键领域至关重要。
> - **完备性**：G²CP 不是只能做简单查询——理论上，任何可通过图遍历完成的任务都可用 G²CP 表达。

**实际意义**：对于安全关键应用（工业维护、医疗保健、航空），这些属性意味着每个系统推荐都可以通过重放产生它的精确图操作来独立审计。与自由文本多智能体系统（中间推理是不透明的）不同，G²CP 提供了从用户查询到最终答案的完整、确定性、可验证的审计轨迹。

### 3.4 Message Syntax（消息语法）

G²CP 消息的具体序列化格式：

```
<sender:agent_id> TO <receiver:agent_id>
PERFORMATIVE: <perf>
CONVERSATION: <conv_id>
OPERATION:
  TRAVERSE
    FROM: <node_selector>
    VIA: <edge_types>
    DEPTH: <hop_count>
    RETURN: <format>
  [CONSTRAINTS: <additional_filters>]
```

节点选择器支持：
- **显式 ID**：`{node_123, node_456}`
- **类型选择**：`{type:Fault, type:Component}`
- **属性过滤**：`{Symptom WHERE severity>0.8}`
- **上下文引用**：`{CURRENT_FOCUS}`

> [!note] 译者注（非原文）
> 消息语法的设计非常务实。四种节点选择方式覆盖了从精确指定到模糊搜索的各种需求。`{CURRENT_FOCUS}` 引用对话子图中的当前焦点节点，使得多轮对话中的增量推理变得自然。

### 3.5 Relationship to FIPA-ACL（与 FIPA-ACL 的关系）

G²CP 与 FIPA-ACL 操作在相同的**通信层**——它定义智能体如何交换消息，而非任务如何编排。架构对比：

| 层 | FIPA-ACL | G²CP |
|---|---------|------|
| 语言行为 | INFORM, REQUEST, ... | INFORM, REQUEST, ... |
| 内容语言 | SL / KIF 谓词 | **图操作** |
| 本体 | 智能体特定 | **共享图模式** |
| 传输 | ACL 信封 | **Kafka + 签名** |

两个框架都使用语言行为来表达语内行为力（illocutionary force）。关键区别在于**内容语言**：

- FIPA 使用逻辑谓词，如 `(price (good g1) 50)`
- G²CP 使用图操作，如 `TRAVERSE FROM {g1} VIA {priced_at} DEPTH 1`

这种替换为 LLM 智能体带来两个优势：
1. 图操作无需定理证明器即可直接执行
2. 结果（子图）通过节点嵌入自然集成到神经检索管线中

> [!tip] 理解提示（非原文）
> 这个对比清楚地揭示了 G²CP 的定位：它不是要替代 FIPA-ACL 的通信框架，而是替换其内容语言。performatives（语言行为）保留不变，但消息的"载荷"从逻辑谓词变成了图操作。这使得 G²CP 可以继承 FIPA 几十年的形式化成果，同时与现代 LLM 系统兼容。

### 3.6 Commitment Semantics（承诺语义）

遵循 Singh [singh1998semantics] 和 Fornara、Colombetti [fornara2004communicative] 的工作，作者形式化了每种 G²CP 语言行为所创建的社会承诺。记 $C(x, y, p)$ 表示"智能体 $x$ 向智能体 $y$ 承诺条件 $p$ 成立"。

#### REQUEST(A, B, TRAVERSE(...))

- **前置条件**：A 相信 B 可以执行该遍历（即 B 的角色覆盖所需边类型）
- **后置条件**：B 承诺返回遍历结果或结构化 ERROR
- **社会承诺**：$C(B, A, \text{execute\_and\_return}(op))$
- **验证方式**：A 可以在 $G$ 上重放 $op$ 并检查结果一致性

#### INFORM(A, B, subgraph)

- **前置条件**：A 已执行产生 subgraph 的操作
- **后置条件**：A 断言 $subgraph \subseteq G$
- **社会承诺**：$C(A, B, \text{grounded}(subgraph, G))$
- **验证方式**：B 通过重新执行源操作检查 $subgraph \subseteq G$

#### QUERY(A, B, TRAVERSE(...))

- **前置条件**：A 寻求存在性信息
- **后置条件**：B 承诺提供真实的布尔响应
- **社会承诺**：$C(B, A, \text{truthful\_response}(op))$
- **验证方式**：A 重放 $op$ 并检查非空性

#### PROPOSE(A, B, op)

- **前置条件**：A 认为 $op$ 是相关的
- **后置条件**：B 承诺评估并以 CONFIRM 或 REJECT 回应
- **社会承诺**：$C(B, A, \text{evaluate\_and\_respond}(op))$

#### CONFIRM(A, B, result)

- **前置条件**：A 已验证 B 的先前结果
- **后置条件**：A 认可 result 为正确
- **社会承诺**：$C(A, B, \text{verified}(result))$

#### REJECT(A, B, op)

- **前置条件**：A 检测到 B 的操作或结果中的约束违反
- **后置条件**：B 不得对被拒绝的结果采取行动
- **社会承诺**：$C(A, B, \text{violated}(op, constraint))$

#### UPDATE(A, B, ΔG)

- **前置条件**：A 被授权进行图修改
- **后置条件**：B 承诺在验证后应用 ΔG
- **社会承诺**：$C(B, A, \text{apply\_if\_valid}(\Delta G))$
- **验证方式**：A 查询更新后的图确认变更

> [!tip] 理解提示（非原文）
> 承诺语义的核心价值在于：每条消息都建立了可公开验证的社会义务。这不是"我建议你做 X"这样的模糊表达，而是明确的前置条件→后置条件→验证机制三元组。这种形式化使得自动化的协议合规性检查成为可能。

所有承诺通过审计日志公开可观察，满足 Singh [singh1998semantics] 的标准：意义应锚定在社会事实中而非私有心智状态中。

## 关键要点回顾

1. G²CP 消息由发送者、接收者、语言行为、图操作和上下文五元组构成
2. 图操作分为 TRAVERSE（递归集合扩展遍历）和 UPDATE（集合论增量更新）两种
3. 三大核心属性：确定性（结果不依赖实现）、可审计性（可重放验证）、完备性（可表达任何图查询）
4. 消息语法支持四种节点选择方式：显式 ID、类型选择、属性过滤、上下文引用
5. 与 FIPA-ACL 的关键区别在于内容语言：逻辑谓词 → 图操作
6. 七种语言行为各有完整的承诺语义：前置条件、后置条件、社会承诺、验证方式

## 下一步学习

- [[raw/lessons/G2CP/04-architecture|04-多智能体架构]]：了解协议如何在实际系统中实现
