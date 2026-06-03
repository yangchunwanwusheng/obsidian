---
type: paper-note
tags: [LLM-Agent, Memory-Mechanism, Survey, Agent-Memory, Reflection, Retrieval]
created: 2026-04-18
updated: 2026-04-18
sources: [https://arxiv.org/abs/2404.13501]
---

# A Survey on the Memory Mechanism of LLM-based Agents

> 这篇论文是第一篇系统性综述 LLM-based Agent（基于大型语言模型的智能体）记忆机制的文献，被 ACM TOIS (Transactions on Information Systems) 接收，对于理解 Agent 如何存储、检索和利用历史信息具有重要参考价值。

## 论文基本信息

| 字段 | 内容 |
|------|------|
| 标题 | A Survey on the Memory Mechanism of Large Language Model based Agents |
| 作者 | Zeyu Zhang, Xiaohe Bo, Chen Ma, Rui Li, Xu Chen, Quanyu Dai, Jieming Zhu, Zhenhua Dong, Ji-Rong Wen |
| 发表 | arXiv 2024.04, ACM TOIS (Transactions on Information Systems) |
| 链接 | https://arxiv.org/abs/2404.13501 |
| GitHub | https://github.com/nuster1128/LLM_Agent_Memory_Survey |

## 一句话核心贡献

本文提出了一个**三维框架（Three-Dimensional Framework）**来系统性地分类和理解 LLM Agent 的记忆机制，从 Memory Sources（记忆来源）、Memory Forms（记忆形式）、Memory Operations（记忆操作）三个维度进行了全面综述。

---

## 背景与动机

### 1.1 为什么记忆对 LLM Agent 至关重要

大型语言模型（Large Language Model, LLM）本身是无状态的（stateless），每次交互都是独立的上下文窗口（context window）。然而，一个真正有效的 Agent 需要：

- **保持一致性（Consistency）**：在多轮对话中记住用户之前的需求和偏好
- **积累经验（Experience）**：从历史任务执行中学习，避免重复错误
- **跨会话持久化（Cross-session Persistence）**：在对话结束后保留关键信息
- **上下文理解（Context Understanding）**：理解当前任务与历史任务的关联

这使得记忆机制成为 LLM Agent 架构中的核心组件。

### 1.2 现有研究的不足

在本文之前，缺乏对 LLM Agent 记忆机制的**系统性分类框架**，研究者难以：
- 理解不同记忆方法之间的联系与区别
- 识别当前研究的空白和未来方向
- 复现和比较不同方法的性能

---

## 三维分类框架详解

本文的核心贡献是提出了 Memory Sources × Memory Forms × Memory Operations 的三维框架。下面逐一详解：

### 维度一：Memory Sources（记忆来源）

记忆来源定义了 Agent 可以从哪些渠道获取信息来构建记忆。

| 来源类型 | 英文 | 中文解释 | 示例 |
|----------|------|----------|------|
| **Task History** | 任务历史 | Agent 在当前会话或跨会话中执行的具体任务记录 | 对话历史、工具调用序列、任务完成状态 |
| **Task-type History** | 同类任务历史 | 相似类型任务的模式和规律 | "用户每次都让我写代码" 的模式识别 |
| **External Knowledge** | 外部知识 | Agent 训练数据之外的知识库 | 知识图谱、向量数据库、实时搜索结果 |

**详细解读**：

#### 1. Task History（任务历史）

这是最直接的记忆来源，包括：
- **对话历史（Dialogue History）**：用户与 Agent 的完整交流记录
- **工具调用历史（Tool Execution History）**：Agent 调用了哪些工具、执行顺序、返回结果
- **任务完成状态（Task Completion Status）**：哪些任务成功、哪些失败、失败原因

#### 2. Task-type History（同类任务历史）

这是更高级的记忆来源，需要 Agent 进行**模式识别（Pattern Recognition）**和**泛化（Generalization）**：
- 从多个具体任务中提取共同的**意图模式（Intent Pattern）**
- 识别用户的**习惯性行为（Habitual Behavior）**
- 建立**任务分类体系（Task Taxonomy）**

#### 3. External Knowledge（外部知识）

当 Agent 需要回答超出其参数化知识（Parametric Knowledge）的问题时，需要从外部获取：
- **知识图谱（Knowledge Graph）**：结构化的实体关系数据
- **向量数据库（Vector Database）**：嵌入后的文档片段
- **实时信息（Real-time Information）**：搜索引挚返回的最新结果

---

### 维度二：Memory Forms（记忆形式）

记忆形式定义了信息在记忆中是如何组织和表示的。

| 形式类型 | 英文 | 中文解释 | 特点 |
|----------|------|----------|------|
| **Complete Info** | 完整信息 | 未经压缩的原始记录 | 保真度高，但存储成本高 |
| **Recent Info** | 最近信息 | 限定时间窗口内的信息 | 简洁，但可能丢失重要历史 |
| **Retrieved Info** | 检索信息 | 通过检索获取的相关片段 | 动态匹配，但依赖检索质量 |
| **Tool Info** | 工具信息 | 工具使用相关的模式和结果 | 特定于工具调用场景 |

**详细解读**：

#### 1. Complete Info（完整信息）

Agent 保留尽可能完整的历史记录：
- 优点：信息不丢失，可回溯任何细节
- 缺点：随时间增长，上下文窗口会被撑爆（Context Overflow）
- 适用场景：短会话、关键任务记录

#### 2. Recent Info（最近信息）

采用**滑动窗口（Sliding Window）**策略，只保留最近 N 轮对话：
- 常见实现：保留最近 K 个 token 或 N 轮对话
- 优点：固定存储成本，控制上下文长度
- 缺点：**近期偏差（Recency Bias）**，早期重要信息可能被遗忘
- 典型方法：对话摘要（Dialogue Summarization）

#### 3. Retrieved Info（检索信息）

通过**检索增强生成（Retrieval-Augmented Generation, RAG）**机制获取：
- **向量检索（Vector Retrieval）**：将记忆编码为向量，查询时计算相似度
- **稀疏检索（Sparse Retrieval）**：如 BM25，基于词项频率
- **稠密检索（Dense Retrieval）**：基于嵌入模型（Embedding Model）
- 优点：按需获取，只加载相关内容
- 缺点：检索质量直接影响记忆可用性

#### 4. Tool Info（工具信息）

专门针对**工具调用（Tool Calling）**场景的记忆：
- 记录哪些工具被成功调用过
- 记录工具调用的参数和结果
- 建立工具使用的**最佳实践（Best Practice）**

---

### 维度三：Memory Operations（记忆操作）

记忆操作定义了 Agent 如何**主动管理和优化**其记忆。

| 操作类型 | 英文 | 中文解释 | 目的 |
|----------|------|----------|------|
| **Reflection** | 反思 | 对历史经验进行抽象和归纳 | 提取高层知识，丢弃细节 |
| **Merging** | 合并 | 将相似或相关的记忆整合 | 减少冗余，统一表示 |
| **Forgetting** | 遗忘 | 主动删除或压缩低价值记忆 | 节省空间，提高效率 |

**详细解读**：

#### 1. Reflection（反思）

Reflection 是让 Agent 从具体经验中**抽象出一般性知识**的过程：

**核心思想**：
```
具体经历 → 抽象总结 → 高层知识
例子：多次成功执行 Python 任务 → 总结"写 Python 的通用步骤"
```

**Reflection 的典型流程**：
1. **回顾（Review）**：Agent 审视最近的任务执行历史
2. **归纳（Generalize）**：提炼出通用的模式或规则
3. **存储（Store）**：将归纳结果存入长期记忆

**相关工作**：
- **Self-Refine**：通过迭代反馈改进输出
- **Generative Agent**：模拟"反思"行为来更新信念

#### 2. Merging（合并）

Merging 是将**相似或相关的记忆条目**整合为统一表示：

**合并的触发条件**：
- 两条记忆描述同一实体或事件
- 记忆存在逻辑矛盾，需要消解
- 多个记忆片段可以组合成更完整的知识

**合并的挑战**：
- **实体消解（Entity Resolution）**：判断两条记录是否指向同一实体
- **冲突解决（Conflict Resolution）**：当信息矛盾时如何处理
- **粒度控制（Granularity Control）**：合并后信息是否过于笼统

#### 3. Forgetting（遗忘）

遗忘并不是记忆的缺陷，而是**主动的信息管理策略**：

**遗忘的类型**：
- **主动遗忘（Active Forgetting）**：Agent 主动决定丢弃某些信息
- **被动遗忘（Passive Forgetting）**：通过存储限制自然淘汰旧信息
- **选择性遗忘（Selective Forgetting）**：根据重要性/相关性筛选

**遗忘的评估指标**：
- **相关性（Relevance）**：与当前任务的相关程度
- **重要性（Importance）**：对长期目标的价值
- **新颖性（Novelty）**：是否提供了新信息

---

## 评估方法

论文系统性地介绍了当前评估 LLM Agent 记忆能力的指标和方法。

### 评估维度

| 评估维度 | 英文 | 中文解释 | 关键指标 |
|----------|------|----------|----------|
| **准确性** | Accuracy | 记忆检索和使用的正确性 | 精确率、召回率、F1 |
| **一致性** | Consistency | 跨会话/跨时间的一致性 | 信念一致性得分 |
| **效率** | Efficiency | 记忆存储和检索的资源消耗 | 延迟、内存占用 |
| **适应性** | Adaptability | 对新任务和新环境的适应 | 少样本学习性能 |

### 常用评估基准

1. **单代理任务基准**：评估单个 Agent 在特定任务（如购物、旅行规划）上的记忆表现
2. **多代理协作基准**：评估多个 Agent 共享记忆时的协作效率
3. **长期记忆基准**：评估 Agent 在长时程任务中的记忆保持能力

---

## 应用场景

### 1. 对话 Agent（Conversational Agent）

对话 Agent 需要在多轮对话中保持**用户画像（User Profile）**和**对话上下文（Dialogue Context）**：

**典型问题**：
- 如何平衡短期上下文和长期用户偏好？
- 当用户改变意图时，Agent 如何"忘记"旧意图？

**代表系统**：ReAct、Reflexion、Self-Refine

### 2. 模拟 Agent（Simulated Agent）

在沙盒环境（如虚拟城市）中模拟智能体行为：

**核心需求**：
- 保持**身份一致性（Identity Consistency）**：智能体是谁，价值观是什么
- 积累**社交经验（Social Experience）**：与虚拟世界中其他智能体的交互历史
- 模拟**记忆衰减（Memory Decay）**：像人类一样遗忘不重要的细节

**代表系统**：Generative Agent (Stanford)、Smallville

### 3. 任务执行 Agent（Task-Oriented Agent）

专注于完成特定任务（如代码生成、数据分析）：

**核心需求**：
- 记住**成功的执行模式（Successful Execution Patterns）**
- 记住**失败教训（Failure Lessons）**，避免重蹈覆辙
- 在**任务切换（Task Switching）**时快速加载相关记忆

**代表系统**：AutoGPT、GPTEngineer、Devin

---

## 未来研究方向

### 1. 记忆架构设计（Memory Architecture Design）

如何设计更高效的**层次化记忆架构（Hierarchical Memory Architecture）**：
- 感官记忆（Sensory Memory）→ 工作记忆（Working Memory）→ 长期记忆（Long-term Memory）
- 类比人类记忆系统的分层设计

### 2. 评估标准化（Evaluation Standardization）

当前缺乏统一的评估基准来衡量不同记忆机制的性能：
- 需要设计**综合基准（Comprehensive Benchmark）**
- 需要**细粒度指标（Fine-grained Metrics）**来区分不同方面的表现

### 3. 多模态记忆（Multimodal Memory）

扩展记忆机制以处理**多模态信息（Multimodal Information）**：
- 图像记忆（Image Memory）
- 视频记忆（Video Memory）
- 音频记忆（Audio Memory）
- 具身感知记忆（Embodied Perception Memory）

### 4. 个性化记忆（Personalized Memory）

为每个用户建立**个性化记忆模型（Personalized Memory Model）**：
- 学习用户的偏好和习惯
- 适应用户的沟通风格
- 预测用户的潜在需求

### 5. 安全与隐私（Security and Privacy）

记忆机制带来的**安全挑战（Security Challenges）**：
- 敏感信息如何安全存储
- 记忆如何防止被恶意篡改
- 用户如何管理和删除自己的记忆数据

---

## 核心要点回顾

1. **三维框架是本文的核心贡献**：Memory Sources × Memory Forms × Memory Operations 构成了理解记忆机制的完整视角

2. **记忆不是静态存储，而是动态过程**：Reflection、Merging、Forgetting 都是主动的记忆管理操作

3. **不同应用场景需要不同的记忆策略**：对话 Agent、模拟 Agent、任务执行 Agent 的需求差异很大

4. **评估标准化是未来重要方向**：需要建立统一的基准来推动研究进展

5. **多模态和个性化是发展趋势**：未来的记忆机制将处理更丰富的信息类型

---

## 相关页面

- [[wiki/concepts/retrieval-augmented-generation]] - 检索增强生成，与记忆检索密切相关
- [[wiki/concepts/reflection-mechanism]] - 反思机制，记忆操作的核心组成
- [[wiki/entities/llm-agent]] - LLM Agent 实体页面
- [[wiki/concepts/working-memory]] - 工作记忆，认知科学中的重要概念

---

## 扩展阅读

- [GitHub: LLM Agent Memory Survey](https://github.com/nuster1128/LLM_Agent_Memory_Survey) - 官方代码仓库
- [arXiv: Generative Agents](https://arxiv.org/abs/2304.03442) - 生成式 Agent 经典论文
- [arXiv: ReAct](https://arxiv.org/abs/2210.03629) - 推理与行动结合的方法
