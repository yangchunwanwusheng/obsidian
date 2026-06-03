---
type: lesson
tags: [多智能体, multi-agent, survey, 协作机制, communication-structure, 论文翻译, 学习笔记]
created: 2026-05-08
updated: 2026-05-08
topic: LLM 多智能体综述中文精读（第 3 篇-05）通信结构
difficulty: intermediate
prerequisites:
  - [[raw/lessons/Multi-Agent-Survey/03-collaboration-mechanisms/04-collaboration-mechanisms-strategies]]
sources:
  - [[raw/paper/multi-agent/03-multi-agent-collaboration-mechanisms-a-survey-of-llms-2025.pdf]]
  - https://arxiv.org/abs/2501.06322
status: completed
series:
  name: "Multi-Agent Survey｜Collaboration Mechanisms（2025）"
  part: 5
paper_meta:
  paper_id: "multi-agent-survey-03"
  title: "Multi-Agent Collaboration Mechanisms: A Survey of LLMs"
  authors: "Khanh-Tung Tran, Dung Dao, Minh-Duong Nguyen, Quoc-Viet Pham, Barry O’Sullivan, Hoang D. Nguyen"
  year: "2025"
  link: "https://arxiv.org/abs/2501.06322"
  section: "Section 4.4"
---

# LLM 多智能体综述中文精读（第 3 篇-05）：通信结构

> 本篇覆盖 **Section 4.4 Communication Structures**。这一节讨论的不是“关系类型”，而是 agent 之间**怎么连、怎么传、谁控谁**。

## 本章导读

> [!note] 译者注（非原文）
> 很多读者会把“合作 / 竞争”和“中心化 / 去中心化”混为一谈。其实它们不是一个维度：
> - 前者是关系；
> - 后者是拓扑。

## 正文翻译

### 原文标题：4.4 Communication Structures

作者将多智能体协作中的通信结构主要划分为三类：
1. **Centralized**
2. **Decentralized / Distributed**
3. **Hierarchical**

作者用 Figure 5 和 Table 4 对这些结构做了对比。

### 原文标题：4.4.1 Centralized Structure

**中心化结构**（也称 star structure）指所有 agent 都连接到一个中心 agent。这个中心 agent 负责：
- 管理；
- 控制；
- 协调其余 agent 的交互与协作。

作者指出，联邦学习可以看作一种典型中心化协作：多个 agent / client 把局部学习结果交给中心端聚合。进入 LLM 时代后，这种思路也被用于分布式 LLM 训练与定制。

除了 FL，作者还列举了更多“中心 hub”例子：
- **LLM-Blender**：收集多个 LLM 输出并用排序聚合；
- **AgentCoord**：帮助用户设计多 LLM 协调策略；
- 一些图构建与知识综合系统：由中心节点反复 query、search、answer。

中心化的优点：
- 设计与实现简单；
- 资源分配高效；
- 汇总和控制更直接。

缺点：
- 中心节点是单点故障；
- 韧性较弱；
- 易形成瓶颈。

### 原文标题：4.4.2 Decentralized and Distributed Structure

**去中心化 / 分布式结构**将控制与决策分散到多个 agent 上。每个 agent 依赖局部信息，并可能与其他 agent 有限通信。

作者指出，这种结构常用于：
- swarm robotics；
- 网络化系统；
- distributed AI；
- world simulation；
- 多轮 debate；
- 动态 reasoning 图结构。

典型特点是：
- peer-to-peer 直接通信；
- 无中心绝对控制者；
- 不同任务与 agent 组合下，最优通信结构可能不同。

作者列举的例子包括：
- 多个 LLM 通过固定轮数 debate 增强 factuality 与 reasoning；
- 动态 DAG reasoning；
- MedAgent、MetaGPT、ChatDev、MARG、AutoAgents 等多专业 agent 协作；
- AgentCF 把 user 和 item 都当成 agent；
- Generative Agents 构建 agent society 进行模拟；
- OpenAgents 提供多 agent 野外开放平台。

作者认为这类结构的优点是：
- 鲁棒；
- 可扩展；
- agent 自主性强；
- 更适合适应变化。

缺点则是：
- 通信开销高；
- 资源分配可能低效；
- 协调难度更大。

### 原文标题：4.4.3 Hierarchical Structure

**层级化结构**让 agent 处在分层系统中，不同层级承担不同功能，主要与同层或相邻层交互。

作者提到的例子包括：
- AgentVerse 的分层协作；
- DyLAN 的多层前馈 agent network；
- CAMEL、ChatEval、ChatDev 等层级化或群组组织；
- 多种总线 / 星型 / 环型 / 树型通信范式。

以 DyLAN 为例：
- 第一阶段是 Team Optimization，筛选高贡献 agent；
- 第二阶段是 Task Solving，由筛出的精英 agent 合作；
- 中间还可能通过 ranker 关停低效 agent。

作者认为层级化的优势包括：
- 分层资源分配较高效；
- 可以降低纯中心化的单点瓶颈；
- 适合复杂工作流与组织式任务。

缺点是：
- 架构更复杂；
- 延迟更高；
- 边缘层或关键层一旦失效，影响严重。

## 本章小结

> [!note] 译者注（非原文）
> 这一节最重要的结论是：
> - centralized 像“总控台”；
> - decentralized 像“平权协作网络”；
> - hierarchical 像“组织架构树”。
>
> 它们没有绝对优劣，关键看任务：
> - 想要可控、简单、易聚合 → centralized；
> - 想要鲁棒、扩展、自主 → decentralized；
> - 想兼顾专业分层与复杂流程 → hierarchical。

## 术语对照

| 英文术语 | 中文译法 | 本章语境说明 |
|---|---|---|
| centralized | 中心化 | 所有协作围绕中心 agent |
| decentralized | 去中心化 | 决策在多个 agent 之间分散 |
| distributed | 分布式 | 控制与执行分散在各处 |
| hierarchical | 层级化 | 分层结构与分级 authority |
| peer-to-peer | 点对点 | agent 直接互相通信 |
| bottleneck | 瓶颈 | 中心节点或关键路径拥塞 |

## 思考题

### 题目 1：为什么中心化常常更容易做出稳定原型？

> [!tip] 理解提示（非原文）
> 想想：如果所有消息都经过一个 orchestrator，调试是不是更容易？

> [!note] 译者注（非原文）
> 因为中心化更容易统一控制流程、汇总结果和限制行为，所以在工程实现上通常更容易快速成型。但代价是脆弱性和扩展瓶颈。

### 题目 2：为什么去中心化更强韧，却往往更难设计？

> [!warning] 易混点（非原文）
> “没有中心”不代表“更简单”。

> [!note] 译者注（非原文）
> 因为控制被分散后，系统要靠局部规则和通信协议自发协调。你省掉了中心节点，却把复杂性转移到了各 agent 的互动设计上。

## 相关页面

- [[raw/lessons/Multi-Agent-Survey/03-collaboration-mechanisms/04-collaboration-mechanisms-strategies]]
- [[raw/lessons/Multi-Agent-Survey/03-collaboration-mechanisms/06-collaboration-mechanisms-coordination-and-lessons]]
- [[raw/paper/multi-agent/03-multi-agent-collaboration-mechanisms-a-survey-of-llms-2025.pdf]]

## 下一步学习

- 上一篇：[[raw/lessons/Multi-Agent-Survey/03-collaboration-mechanisms/04-collaboration-mechanisms-strategies|04-strategies]]
- 下一篇：[[raw/lessons/Multi-Agent-Survey/03-collaboration-mechanisms/06-collaboration-mechanisms-coordination-and-lessons|06-coordination-and-lessons]]
