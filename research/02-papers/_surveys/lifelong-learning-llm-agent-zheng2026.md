---
type: paper-note
tags: [LLM-Agent, Lifelong-Learning, Memory-Module, Perception-Module, Action-Module, Continual-Learning]
created: 2026-04-18
updated: 2026-04-18
sources: [https://arxiv.org/abs/2501.07278]
---

# Lifelong Learning of LLM-based Agents

> 这是一篇关于 LLM-based Agent 终身学习的系统性综述论文，被 IEEE TPAMI (Transactions on Pattern Analysis and Machine Intelligence) 接收。论文提出了一个涵盖感知、记忆、动作三大模块的完整框架，为构建能够不断学习和适应的 Agent 系统提供了路线图。

## 论文基本信息

| 字段 | 内容 |
|------|------|
| 标题 | Lifelong Learning of Large Language Model based Agents: A Roadmap |
| 作者 | Junhao Zheng, Chengming Shi, Xidi Cai, Qiuke Li, Duzhen Zhang, Chenxing Li, Dong Yu, Qianli Ma |
| 发表 | IEEE TPAMI 2026 |
| 链接 | https://arxiv.org/abs/2501.07278 |
| GitHub | https://github.com/qianlima-lab/awesome-lifelong-llm-agent |

## 一句话核心贡献

本文提出了一个**模块化框架（Modular Framework）**，包含 Perception Module（感知模块）、Memory Module（记忆模块）、Action Module（动作模块），系统性地梳理了 LLM Agent 终身学习的理论基础、方法分类和未来挑战。

---

## 背景与动机

### 1.1 什么是终身学习

**终身学习（Lifelong Learning）**，又称**持续学习（Continual Learning）**或**增量学习（Incremental Learning）**，是指一个系统能够在不断变化的环境中**持续获取新知识**、**保留旧知识**、**适应新任务**的能力。

对于 LLM Agent 而言，终身学习意味着：
- **跨任务迁移（Cross-task Transfer）**：将在一个任务中学到的知识应用到新任务
- **灾难性遗忘防止（Catastrophic Forgetting Prevention）**：学习新知识时不忘记已学会的知识
- **自适应能力（Adaptive Capability）**：根据环境变化调整自己的行为策略

### 1.2 LLM Agent 终身学习的挑战

与传统机器学习不同，LLM Agent 的终身学习面临独特挑战：

| 挑战类型 | 英文 | 中文解释 |
|----------|------|----------|
| **灾难性遗忘** | Catastrophic Forgetting | 学习新任务时严重损害旧任务的性能 |
| **知识可塑性** | Knowledge Plasticity | 新知识如何有效融入现有知识体系 |
| **资源约束** | Resource Constraints | 无法无限扩展模型或记忆存储 |
| **任务边界模糊** | Task Boundary Ambiguity | 任务之间没有明确边界，难以区分 |
| **评估困难** | Evaluation Difficulty | 缺乏标准化的持续学习评估基准 |

### 1.3 本文与 Zhang 2024 的区别

| 维度 | Zhang 2024 (Memory Survey) | Zheng 2026 (Lifelong Learning) |
|------|---------------------------|-------------------------------|
| **焦点** | 记忆机制的分类与综述 | Agent 的持续学习与适应能力 |
| **框架** | 三维框架（来源/形式/操作） | 三大模块（感知/记忆/动作） |
| **目标** | 理解现有记忆方法 | 指导 Agent 如何不断进化 |
| **视角** | 静态分类 | 动态过程 |

---

## 模块化框架详解

### 模块一：Perception Module（感知模块）

**感知模块**负责将外部环境的信息转化为 Agent 可以处理的内部表示。

#### 1.1 单模态感知（Unimodal Perception）

处理单一类型的数据输入：

| 模态类型 | 英文 | 中文解释 | Agent 应用 |
|----------|------|----------|------------|
| **文本感知** | Text Perception | 处理和理解自然语言 | 对话理解、文档阅读 |
| **视觉感知** | Visual Perception | 处理图像和视频 | 视觉问答、图像描述 |
| **听觉感知** | Audio Perception | 处理声音和语音 | 语音助手、音乐理解 |
| **触觉感知** | Tactile Perception | 处理触觉信息 | 机器人操作 |

#### 1.2 多模态感知（Multimodal Perception）

整合多种感知通道的信息：

**多模态融合策略（Multimodal Fusion Strategy）**：

```
早期融合（Early Fusion）
  ↓
原始特征 → 联合表示 → 统一处理
  优点：保留细粒度信息
  缺点：融合难度大

晚期融合（Late Fusion）
  ↓
独立编码 → 决策层融合 → 最终输出
  优点：各模态独立处理
  缺点：可能丢失跨模态关联

中期融合（Intermediate Fusion）
  ↓
分层交互 → 逐步整合
  优点：平衡早期和晚期融合
  缺点：实现复杂度高
```

#### 1.3 感知记忆（Perceptual Memory）

感知模块与记忆模块的交互：

- **瞬时记忆（Iconic Memory）**：极短期的感知信息保留
- **工作记忆整合（Working Memory Integration）**：将感知信息组织成可理解的片段
- **选择性注意（Selective Attention）**：决定哪些感知信息值得进入记忆

---

### 模块二：Memory Module（记忆模块）

**记忆模块**是本文的核心，也是与 Zhang 2024 论文最相关的内容。本文将 Agent 记忆分为四种类型：

#### 2.1 Working Memory（工作记忆）

**定义**：工作记忆是**短期的、高频访问的**记忆形式，类似于人类认知中的"心理工作台"，用于临时存储和操作当前任务所需的信息。

**核心特征**：
| 特征 | 英文 | 中文解释 |
|------|------|----------|
| **容量有限** | Limited Capacity | 受限于上下文窗口大小，通常 4K-128K tokens |
| **短期性质** | Short-term Nature | 只在当前任务执行期间保持 |
| **高带宽** | High Bandwidth | 快速读写，与 LLM 直接交互 |
| **易失性** | Volatility | 对话结束或任务完成后可能被清除 |

**在 LLM Agent 中的实现**：
- **上下文窗口（Context Window）**：所有信息都在 Transformer 的注意力范围内
- **对话历史（Dialogue History）**：最近的交互记录
- **当前任务状态（Task State）**：正在执行的任务的中间结果

**典型代表**：
- **ChatGPT 的上下文机制**：将对话历史作为 context 传入
- **ReAct 的推理痕迹（Reasoning Trace）**：在推理过程中维护中间状态

#### 2.2 Episodic Memory（情景记忆）

**定义**：情景记忆存储**具体的经历和事件**，包括时间、地点、情感等上下文信息。类似于人类记忆中的"往事回忆"。

**核心特征**：
| 特征 | 英文 | 中文解释 |
|------|------|----------|
| **情境绑定** | Contextual Binding | 与特定时间、地点、情感相关联 |
| **自传体性质** | Autobiographical | 与 Agent 自身的"经历"相关 |
| **可回溯性** | Traceability | 可以检索特定事件的细节 |
| **时间结构** | Temporal Structure | 按时间顺序组织 |

**典型代表系统**：

| 系统名称 | 论文/项目 | 核心机制 |
|----------|----------|----------|
| **RET-LLM** | RET-LLM (2023) | 将对话历史转换为结构化的记忆表示 |
| **MemoChat** | MemoChat (2023) | 为对话 Agent 添加情景记忆，支持角色扮演 |
| **Generative Agent** | Stanford (2023) | 模拟 Agent 的"生活记忆"，记录一天中的事件 |

**情景记忆的挑战**：
- **记忆压缩（Memory Compression）**：如何高效存储大量经历
- **记忆检索（Memory Retrieval）**：如何快速找到相关的情景
- **记忆整合（Memory Integration）**：如何将新经历与旧经历关联

#### 2.3 Semantic Memory（语义记忆）

**定义**：语义记忆存储**一般性知识和概念**，不与特定情境绑定。类似于人类记忆中的"事实知识库"。

**核心特征**：
| 特征 | 英文 | 中文解释 |
|------|------|----------|
| **抽象性** | Abstractness | 脱离具体情境的一般性知识 |
| **概念化** | Conceptualization | 以概念和关系的形式组织 |
| **持久性** | Persistence | 长期存在，不易遗忘 |
| **可推理性** | Reasonability | 支持逻辑推理和知识推导 |

**在 LLM Agent 中的实现**：

| 实现方式 | 英文 | 中文解释 | 示例 |
|----------|------|----------|------|
| **知识图谱** | Knowledge Graph | 结构化的实体-关系表示 | ConceptNet, WordNet |
| **向量数据库** | Vector Database | 知识的密集向量表示 | ChromaDB, Pinecone |
| **外部知识库** | External Knowledge Base | 结构化的事实存储 | Wikipedia, 行业数据库 |
| **提示工程** | Prompt Engineering | 在 prompt 中注入知识 | Few-shot examples |

**典型代表系统**：

| 系统名称 | 核心机制 |
|----------|----------|
| **KonwAgent** | 使用知识图谱增强 Agent 的事实推理能力 |
| **ChatKBQA** | 基于知识图谱的问答 Agent |

#### 2.4 Parametric Memory（参数化记忆）

**定义**：参数化记忆是指**存储在模型参数中的知识**，通过模型编辑（Model Editing）技术来管理和更新。

**核心特征**：
| 特征 | 英文 | 中文解释 |
|------|------|----------|
| **隐式存储** | Implicit Storage | 知识编码在权重中，而非显式表示 |
| **大规模** | Large Scale | 可以存储海量知识 |
| **难更新** | Hard to Update | 修改参数可能影响其他能力 |
| **推理时访问** | Accessed at Inference | 在生成过程中被调用 |

**模型编辑技术详解**：

| 技术名称 | 英文全称 | 中文解释 | 特点 |
|----------|----------|----------|------|
| **LoRA** | Low-Rank Adaptation | 低秩适配 | 通过低秩矩阵更新，效率高 |
| **WISE** | Writes In Specialized Embeddings | 专门嵌入空间的写入 | 针对特定知识类型的定向编辑 |
| **GRACE** | Gradient-Based Knowledge Editing | 基于梯度的知识编辑 | 通过梯度信号精确定位和修改知识 |
| **ROME** | Rank-One Model Editing | 单秩模型编辑 | 将新知识作为秩一更新注入 |
| **FT** | Fine-Tuning | 微调 | 直接在目标任务上训练 |

**为什么需要参数化记忆？**

虽然 RAG（检索增强生成）可以提供外部知识，但参数化记忆有其独特价值：
1. **推理效率**：参数化知识不需要检索，直接通过注意力调用
2. **知识一致性**：同一知识在所有场景下表达一致
3. **隐式推理**：模型可以自发地组合多个知识片段

**参数化记忆的挑战**：

| 挑战 | 英文 | 中文解释 |
|------|------|----------|
| **知识定位** | Knowledge Localization | 确定哪个参数存储了特定知识 |
| **知识干扰** | Knowledge Interference | 修改一个知识可能影响其他知识 |
| **更新可逆性** | Update Reversibility | 如何在出错时撤销更新 |
| **可扩展性** | Scalability | 如何支持大量知识的存储和更新 |

#### 2.5 四种记忆类型的对比

| 维度 | Working Memory | Episodic Memory | Semantic Memory | Parametric Memory |
|------|----------------|-----------------|----------------|-------------------|
| **存储位置** | Context / RAM | Vector DB / Graph | Knowledge Graph | Model Weights |
| **容量** | 有限（上下文窗口） | 中等（可扩展） | 大（外部存储） | 极大（整个模型） |
| **更新速度** | 极快 | 快 | 中等 | 慢 |
| **遗忘风险** | 高 | 中 | 低 | 极低 |
| **访问方式** | 直接读取 | 检索 | 查询/推理 | 注意力 |
| **适用场景** | 当前任务 | 经验积累 | 事实推理 | 核心能力 |

---

### 模块三：Action Module（动作模块）

**动作模块**负责将决策转化为具体行为，并从执行结果中学习。

#### 3.1 Grounding Actions（接地动作）

**定义**：接地动作是将**抽象的语言输出**连接到**具体的外部环境或工具**的操作。

**核心问题**：LLM 输出的是文本（text），如何让文本"落地"到真实世界？

**典型接地动作**：
| 动作类型 | 英文 | 中文解释 | 示例 |
|----------|------|----------|------|
| **工具调用** | Tool Calling | 调用外部工具执行操作 | 调用 API、运行代码 |
| **环境交互** | Environment Interaction | 与物理或虚拟环境交互 | 操控机器人、点击界面 |
| **状态更新** | State Update | 更新外部系统的状态 | 写文件、发送消息 |
| **多模态生成** | Multimodal Generation | 生成非文本输出 | 生成图像、语音 |

**工具调用详解**：

```
LLM 输出：
"我需要查询北京的天气"

↓ 接地过程

工具调用：
function: get_weather
args: {"location": "北京"}

↓ 执行

外部 API 返回：
{"temperature": 22, "condition": "晴"}

↓ 结果注入

LLM 整合结果后输出：
"北京今天天气晴朗，气温22度..."
```

#### 3.2 Retrieval Actions（检索动作）

**定义**：检索动作是从**记忆系统中获取**相关信息来辅助决策。

**检索动作的类型**：

| 类型 | 英文 | 中文解释 | 使用场景 |
|------|------|----------|----------|
| **直接检索** | Direct Retrieval | 根据当前上下文直接匹配记忆 | 已知要找什么 |
| **推理检索** | Inference-based Retrieval | 通过推理确定需要什么记忆 | 不知道要找什么 |
| **相似性检索** | Similarity Retrieval | 基于向量相似度获取相关记忆 | 发现隐式关联 |
| **路径检索** | Path-based Retrieval | 在知识图谱中沿路径检索 | 关系推理 |

**检索与动作的结合**：
```
用户询问：
"上次我让你做的数据分析报告在哪里？"

↓ 检索动作

在 Episodic Memory 中检索：
- 找到"数据分析报告"相关的情景记忆
- 提取时间、地点、文件位置

↓ 接地动作

执行文件操作或返回引用
```

#### 3.3 Reasoning Actions（推理动作）

**定义**：推理动作是**利用记忆和知识进行逻辑推理**来解决问题。

**推理类型**：

| 类型 | 英文 | 中文解释 | 示例 |
|------|------|----------|------|
| **演绎推理** | Deductive Reasoning | 从一般到特殊的推理 | 已知"所有A是B"，推断"这个A是B" |
| **归纳推理** | Inductive Reasoning | 从特殊到一般的推理 | 从多个案例中总结规律 |
| **溯因推理** | Abductive Reasoning | 最佳解释推理 | 观察结果，推断最可能的原因 |
| **类比推理** | Analogical Reasoning | 基于相似性的推理 | "这类似于上次遇到的XX情况" |

**记忆增强推理**：

```
问题：
"为什么我的服务器响应变慢了？"

↓ 推理动作

Step 1: 检索相关记忆
- 历史日志（Episodic Memory）
- 性能基线（Semantic Memory）
- 类似案例（Parametric Memory）

Step 2: 组合推理
- 当前情况 + 历史模式 → 可能原因
- 排除法 → 最可能原因

Step 3: 生成建议
"根据历史记录，每次流量高峰后..."
```

---

## 三大模块的协同工作

### 信息流架构

```
外部环境
    ↓
┌─────────────────┐
│ Perception      │ ← 感知模块：信息获取与预处理
│ Module          │
└────────┬────────┘
         ↓
┌─────────────────┐
│ Working Memory  │ ← 工作记忆：信息临时存储与操作
│ (Short-term)    │
└────────┬────────┘
         ↓ (整合)
┌─────────────────┐
│ Memory Module   │ ← 记忆模块：多类型记忆的协调
│ - Episodic      │    • Episodic: 经验记录
│ - Semantic      │    • Semantic: 知识积累
│ - Parametric    │    • Parametric: 能力固化
└────────┬────────┘
         ↓
┌─────────────────┐
│ Action Module   │ ← 动作模块：决策与执行
│ - Grounding     │    • Grounding: 连接现实
│ - Retrieval     │    • Retrieval: 记忆获取
│ - Reasoning     │    • Reasoning: 逻辑思考
└────────┬────────┘
         ↓
外部环境 / 其他 Agent
```

### 模块间的信息反馈

```
Perception → Memory → Action → 结果评估 → Memory 更新
                              ↓
                        反思（Reflection）
                              ↓
                     记忆整合（Memory Integration）
```

---

## LifelongAgentBench：评估基准

本文提出了 **LifelongAgentBench**，一个综合评估 LLM Agent 终身学习能力的基准。

### 评估维度

| 维度 | 英文 | 中文解释 | 关键指标 |
|------|------|----------|----------|
| **持续性** | Continuity | 在连续任务中的性能保持 | 旧任务性能衰减率 |
| **适应性** | Adaptability | 对新任务的适应速度 | 新任务收敛速度 |
| **效率** | Efficiency | 学习新知识的资源消耗 | 训练时间、内存占用 |
| **泛化性** | Generalization | 知识迁移到新场景的能力 | 跨任务平均性能 |

### 基准任务设计

LifelongAgentBench 包含多层次的评估任务：

1. **任务序列（Task Sequences）**：按顺序执行的任务，测试记忆保持
2. **任务混合（Task Mixtures）：多个任务同时存在，测试选择性记忆
3. **任务演化（Task Evolution）**：任务定义随时间变化，测试适应能力
4. **跨域迁移（Cross-domain Transfer）**：领域切换，测试知识泛化

---

## 未来研究方向

### 1. 记忆模块的整合优化

如何让 Working Memory、Episodic Memory、Semantic Memory、Parametric Memory 更高效地协作：
- **动态记忆路由（Dynamic Memory Routing）**：根据任务类型自动选择记忆来源
- **记忆层次化（Memory Hierarchization）**：建立记忆的层次结构
- **跨记忆推理（Cross-memory Reasoning）**：在多种记忆类型间进行推理

### 2. 感知-记忆-动作的闭环

建立更紧密的反馈循环：
- **动作结果反馈（Action Feedback Loop）**：动作执行结果如何影响记忆更新
- **主动学习（Active Learning）**：Agent 主动选择要学习什么
- **元学习（Meta-learning）**：学习如何学习

### 3. 个性化与隐私的平衡

- 如何在不侵犯隐私的前提下建立个性化记忆
- 用户对记忆的**可控性（Controllability）**：查看、修改、删除记忆
- **联邦学习（Federated Learning）**：在不集中数据的情况下学习

### 4. 多模态终身学习

- 如何让 Agent 记住视觉、听觉、触觉等多模态信息
- **跨模态记忆关联（Cross-modal Memory Association）**：将"看到的东西"和"听到的描述"关联

### 5. 评估理论与基准

- 建立更完善的**终身学习评估理论**
- 设计更接近真实场景的**动态评估基准**

---

## 核心要点回顾

1. **三大模块框架**：Perception-Memory-Action 构成了 LLM Agent 的终身学习骨架

2. **四种记忆类型各有分工**：
   - Working Memory：短期操作
   - Episodic Memory：经验记录
   - Semantic Memory：知识存储
   - Parametric Memory：能力固化

3. **三种动作类型相互配合**：
   - Grounding：连接现实
   - Retrieval：获取记忆
   - Reasoning：逻辑思考

4. **模块间需要紧密协作**：感知获取信息，记忆存储信息，动作利用信息，三者形成闭环

5. **评估基准是研究关键**：LifelongAgentBench 为该领域提供了统一的评估标准

---

## 与 Zhang 2024 的关系

| 比较维度 | Zhang 2024 | Zheng 2026 |
|----------|------------|------------|
| **研究对象** | Memory Mechanism（记忆机制） | Lifelong Learning（终身学习） |
| **框架维度** | 3D: Sources × Forms × Operations | 3 Modules: Perception × Memory × Action |
| **记忆分类** | Forms: Complete/Recent/Retrieved/Tool | Types: Working/Episodic/Semantic/Parametric |
| **操作视角** | Memory Operations: Reflection/Merging/Forgetting | Memory Integration in Agent Lifecycle |
| **评估** | General evaluation methods | LifelongAgentBench |
| **发表** | ACM TOIS 2024 | IEEE TPAMI 2026 |

两篇论文互为补充：Zhang 2024 提供了记忆机制的静态分类，Zheng 2026 提供了 Agent 终身学习的动态视角。

---

## 相关页面

- [[wiki/concepts/agent-memory-survey-zhang2024]] - Zhang 2024 论文笔记
- [[wiki/concepts/retrieval-augmented-generation]] - RAG，与 Semantic Memory 相关
- [[wiki/concepts/continual-learning]] - 持续学习，机器学习领域相关概念
- [[wiki/entities/llm-agent]] - LLM Agent 实体页面

---

## 扩展阅读

- [GitHub: awesome-lifelong-llm-agent](https://github.com/qianlima-lab/awesome-lifelong-llm-agent) - 官方资源汇总
- [arXiv: LoRA](https://arxiv.org/abs/2106.09685) - 低秩适配技术
- [arXiv: ROME](https://arxiv.org/abs/2202.05262) - 模型编辑技术
- [IEEE TPAMI](https://www.computer.org/tpami/) - 期刊官网
