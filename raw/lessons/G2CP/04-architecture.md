---
type: lesson
tags: [G2CP, 多智能体架构, Neo4j, LLM, 动态操作选择, 工业诊断, 论文翻译]
created: 2026-05-16
updated: 2026-05-16
sources:
  - arXiv:2602.13370
difficulty: advanced
prerequisites:
  - "[[raw/lessons/G2CP/03-protocol|G2CP 03-协议定义]]"
topic: G2CP 多智能体架构翻译
status: completed
series:
  name: G2CP
  part: 4
---

# G2CP 论文翻译 — 第4节 Multi-Agent Architecture（多智能体架构）

> 四个专用智能体通过 G²CP 图操作协调，LLM 仅用于用户界面层和动态操作选择。

## 正文翻译

### 4.1 Agent Roles and Specializations（智能体角色与专业化）

G²CP 在一个工业知识管理系统中实例化了四个专用智能体：

**诊断智能体（Diagnostic Agent, $A_D$）**：通过遍历症状到故障的关系执行根因分析。
- 专业化边类型：$\Psi_{diag} = \{causes, indicates, correlates\_with\}$

**流程智能体（Procedural Agent, $A_P$）**：通过遍历故障到操作的关系检索维修程序。
- 专业化边类型：$\Psi_{proc} = \{addressed\_by, requires, precedes\}$

**综合智能体（Synthesis Agent, $A_S$）**：通过历史数据遍历发现模式。
- 专业化边类型：$\Psi_{synth} = \{occurred\_in, replaced\_in, failed\_after\}$

综合智能体通过**时间图遍历**发现新关系。具体来说，$A_S$ 计算故障事件与上下文条件之间的共现频率。对于每一对 $(f, c)$（其中 $f$ 是故障节点，$c$ 是条件节点如传感器读数或环境因素），$A_S$ 遍历所有包含 $f$ 的历史工单，检查 $c$ 在可配置时间窗口（默认 48 小时）内是否时间共现。如果共现频率超过阈值（例如 $f = $ "Part X failure" 和 $c = $ "temp_anomaly" 在 20 个案例中有 15 个共现），$A_S$ 通过 UPDATE 消息提出置信度 $15/20 = 0.75$ 的新边。这种机制使知识图能够从运营数据中持续丰富。

> [!tip] 理解提示（非原文）
> 综合智能体的共现发现机制是一个非常有价值的设计。它将"知识发现"也纳入了 G²CP 框架——新知识通过 UPDATE 操作添加到图中，整个流程可审计。这意味着系统不是静态的，而是可以持续学习的。

**摄取智能体（Ingestion Agent, $A_I$）**：基于新信息更新图，使用 UPDATE 操作维护 $G$。

> [!note] 译者注（非原文）
> 四个智能体的专业化方式值得注意：不是通过不同的提示词或系统提示来区分，而是通过**允许访问的边类型**来区分。这是一种"结构化权限"设计——每个智能体只能遍历其专业范围内的关系。这既限制了错误传播范围，又使权限管理形式化。

### 4.2 Coordination Protocol（协调协议）

典型的复杂诊断查询交互序列分为三个阶段：

**阶段一：查询分解（Query Decomposition）**

接收到用户查询 $q$ 后，Dispatcher 使用 LLM 提取实体、分类意图并选择主智能体。对于诊断查询，它提取症状 $S = \{s_1, \ldots, s_k\}$ 并构造：

```
REQUEST TRAVERSE
  FROM: {s_1, ..., s_k}
  VIA: {causes, indicates}
  DEPTH: 2
  RETURN: SUBGRAPH
```

**阶段二：迭代精化（Iterative Refinement）**

智能体可以互相 REQUEST 操作。$A_D$ 识别故障 $f^*$ 后，向 $A_P$ 发送：

```
REQUEST TRAVERSE
  FROM: {f*}
  VIA: {addressed_by}
  DEPTH: 1
  RETURN: SUBGRAPH
```

**阶段三：知识整合（Knowledge Integration）**

当 $A_S$ 发现新模式时，它广播：

```
UPDATE APPLY
  ADD_EDGE: {
    from: Part:X,
    to: Sensor:temp_anomaly,
    type: risk_indicator,
    confidence: 0.89
  }
```

### 4.2.1 Complete Worked Example（完整工作示例）

跟踪查询 **"What causes pressure drops in hydraulic circuit HC-3?"** 的完整消息交换：

**Step 1.** Dispatcher → $A_D$：

```
Dispatcher TO A_D
PERFORMATIVE: REQUEST
CONVERSATION: conv_042
OPERATION:
  TRAVERSE
    FROM: {Component:HC-3}
    VIA: {has_symptom}
    DEPTH: 1
    RETURN: SUBGRAPH
```

**Step 2.** $A_D$ 在 Neo4j 上执行遍历，检索与 HC-3 关联的症状：`Symptom:pressure_drop`、`Symptom:flow_reduction`。然后 $A_D$ 执行第二次遍历查找根因：

```
A_D TO Dispatcher
PERFORMATIVE: INFORM
CONVERSATION: conv_042
OPERATION:
  RESULT SUBGRAPH:
    Nodes: {Fault:seal_degradation,
            Fault:valve_blockage,
            Fault:pump_cavitation}
    Edges: {seal_degradation -[causes]-> pressure_drop,
            valve_blockage -[causes]-> flow_reduction,
            pump_cavitation -[causes]-> pressure_drop}
    Confidence: [0.91, 0.84, 0.72]
```

> [!tip] 理解提示（非原文）
> 注意 INFORM 消息返回的是子图而非自然语言描述。子图中包含节点、边和置信度分数。接收方可以直接基于这个子图进行下一步操作，无需"理解"自然语言。

**Step 3.** Dispatcher → $A_P$（针对排名最高的故障）：

```
Dispatcher TO A_P
PERFORMATIVE: REQUEST
CONVERSATION: conv_042
OPERATION:
  TRAVERSE
    FROM: {Fault:seal_degradation}
    VIA: {addressed_by, requires_part}
    DEPTH: 1
    RETURN: SUBGRAPH
```

**Step 4.** $A_P$ → $A_S$（检查历史频率）：

```
A_P TO A_S
PERFORMATIVE: QUERY
CONVERSATION: conv_042
OPERATION:
  TRAVERSE
    FROM: {Fault:seal_degradation, Component:HC-3}
    VIA: {occurred_in}
    DEPTH: 2
    RETURN: LEAVES
```

**Step 5.** $A_S$ 返回历史发生数据（18 个月内 7 次事件）。$A_P$ 将此整合到程序推荐中。Dispatcher 从收集的子图生成最终自然语言响应。

**总 token 数**：189（对比 FTMA 估计 1,400+）。

> [!note] 译者注（非原文）
> 这个完整示例非常有价值。5 步消息交换完成了一个涉及诊断→程序检索→历史验证的复杂查询，总共只用 189 个 token。对比 FTMA 的 1,400+ token，效率提升约 7.4 倍。更重要的是，每一步都是可验证的——任何人都可以重放这些图操作检查结果。

### 4.3 Implementation Details（实现细节）

每个智能体维护：
- **本地图缓存**：$G$ 的物化视图，针对其边类型专业化优化
- **消息队列**：带优先级排序的待处理 G²CP 消息
- **执行引擎**：基于 Cypher 的遍历执行器
- **LLM 接口**：仅用于解析用户查询和格式化最终响应，**不用于智能体间通信**

系统使用 **Neo4j** 进行图存储，自定义遍历 API。消息传递通过 **Apache Kafka** 实现可靠性和可审计性。

### 4.4 G²CP Runtime Engine（G²CP 运行时引擎）

每个智能体运行一个 G²CP 解析器，实现三阶段管线：

**Algorithm 1: G²CP Message Parsing and Execution（消息解析与执行）**

```
输入：原始消息 m_raw
输出：执行结果 R 或错误

// 阶段 1：解析并验证语法
m ← ParseMessage(m_raw)
if m = INVALID then return ERROR("Malformed message")

// 阶段 2：安全验证
if ¬AuthorizeAgent(m.sender, m.op) then
    return ERROR("Unauthorized operation")

// 阶段 3：执行操作
if m.op is TRAVERSE then
    V_s ← ResolveNodeSelector(m.op.from)
    R ← ExecuteTraversal(V_s, m.op.via, m.op.depth)
else if m.op is UPDATE then
    R ← ApplyUpdate(G, m.op.ΔG)

// 阶段 4：记录审计日志
AuditLog.append(m, R, timestamp)
return R
```

**节点解析（Node Resolution）**使用优先级系统：
1. 显式 ID：通过索引查找
2. 类型过滤器：通过 Cypher `MATCH (n:Type) RETURN n`
3. 属性过滤器：编译为参数化 Cypher 并进行消毒处理
4. 上下文引用：从对话状态解析

**遍历执行器（Traversal Executor）**使用 Neo4j 模式匹配：

```cypher
MATCH path = (start)-[r*1..{depth}]->(end)
WHERE start IN {node_set}
  AND type(r) IN {edge_filter}
RETURN path
```

对于大规模遍历（$h > 2$），使用广度优先边界扩展，在 1000 个节点处提前终止。

**错误处理**：超时保护（30秒）、结果大小限制（5000 节点）、畸形操作报告、图不一致处理（重试或升级到人工）。

> [!tip] 理解提示（非原文）
> 运行时引擎的三阶段管线（解析→安全验证→执行）是经典的安全设计模式。注意安全验证在执行之前——即使一条格式正确的消息，如果发送者没有相应权限，也会被拒绝。审计日志在最后记录，确保每条消息的完整生命周期可追踪。

### 4.5 LLM-Based Operation Selection（基于 LLM 的动态操作选择）

关键澄清：**智能体不是预脚本化的**。每个智能体使用基于 LLM 的推理在运行时动态选择图操作。只有图模式（节点和边类型）是预定义的——所有基线共享相同的图模式。Algorithm 2 详述了选择过程：

**Algorithm 2: LLM-Based Graph Operation Selection（基于 LLM 的图操作选择）**

```
输入：用户查询 q 或上游 G²CP 消息 m
输出：G²CP 操作 op

// Step 1: 通过 LLM 提取实体
entities ← LLM(EXTRACT_PROMPT, q)
// 例如 {"bearing B-4521", "grinding noise"}

// Step 2: 实体链接到图节点
V_s ← ∅
for each e ∈ entities do
    v ← FuzzyMatch(e, V, threshold=0.85)
    if v ≠ NULL then
        V_s ← V_s ∪ {v}

// Step 3: 意图分类
intent ← LLM(INTENT_PROMPT, q)
// 四种之一: diagnostic, procedural, predictive, factoid

// Step 4: 基于意图选择边类型
Ψ_f ← EDGE_MAP[intent]
// 例如 diagnostic → {causes, indicates}

// Step 5: 深度估计
h ← LLM(DEPTH_PROMPT, q, |V_s|, |Ψ_f|)
// 通常 1-3，基于查询复杂度

// Step 6: 组装 G²CP 操作
op ← TRAVERSE(V_s, Ψ_f, h, ret)
return op
```

**示例提示词**：

实体提取提示词：
> Given the query: "{query}"
> Extract all named entities that correspond to industrial equipment, symptoms, faults, or parts. Return as JSON: {"entities": [...]}

意图分类提示词：
> Classify the query intent into one of: diagnostic (symptom → cause), procedural (fault → fix), predictive (pattern → forecast), factoid (direct lookup). Query: "{query}" Intent:

深度估计提示词：
> Given {n_entities} source entities and {n_edge_types} edge types, estimate the number of traversal hops (1-3) needed. Simple lookups: 1. Causal chains: 2. Complex multi-factor analysis: 3. Query: "{query}" Depth:

> [!warning] 易混点（非原文）
> Algorithm 2 揭示了一个重要细节：LLM 在 G²CP 系统中有三个用途（查询理解、实体提取、意图分类→操作选择），但**不用于智能体间通信**。LLM 的非确定性和幻觉风险被限制在"用户→系统"的边界内，而"系统内部"的智能体间通信完全由确定性图操作控制。这是一个精心设计的安全边界。

## 关键要点回顾

1. 四个专用智能体通过边类型专业化分工：诊断（causes/indicates）、流程（addressed_by/requires）、综合（occurred_in/replaced_in）、摄取（UPDATE）
2. 协调三阶段：查询分解→迭代精化→知识整合
3. 完整工作示例：HC-3 压力下降查询仅用 189 token 完成诊断→程序检索→历史验证
4. 运行时引擎：三阶段管线（解析→安全验证→执行）+ 审计日志
5. LLM 的角色被限定在用户界面层：实体提取、意图分类、深度估计——不参与智能体间通信

## 下一步学习

- [[raw/lessons/G2CP/05-experiments|05-实验评估]]：了解实验设计和性能数据
