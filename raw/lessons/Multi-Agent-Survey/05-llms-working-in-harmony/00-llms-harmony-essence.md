---
type: lesson
tags: [多智能体, multi-agent, survey, 技术支柱, 架构, 规划, 记忆, 论文精华, 学习笔记]
created: 2026-05-15
updated: 2026-05-15
topic: LLM 多智能体综述中文精读（第 5 篇-00）论文精华总览
difficulty: intermediate
prerequisites:
  - [[raw/lessons/Multi-Agent-Survey/03-collaboration-mechanisms/00-collaboration-mechanisms-essence]]
  - [[raw/lessons/Multi-Agent-Survey/04-trust-worthiness/00-trust-essence]]
sources:
  - https://arxiv.org/abs/2504.01963
status: completed
series:
  name: "Multi-Agent Survey｜LLMs Working in Harmony（2025）"
  part: 0
paper_meta:
  paper_id: "multi-agent-survey-05"
  title: "LLMs Working in Harmony: A Survey on the Technological Aspects of Building Effective LLM-Based Multi Agent Systems"
  authors: "RM Aratchige, Dr. WMKS Ilmini"
  year: "2025"
  link: "https://arxiv.org/abs/2504.01963"
  venue: "arXiv preprint (arXiv:2504.01963)"
  section: "essence-summary"
---

# LLM 多智能体综述中文精读（第 5 篇-00）：论文精华总览

> 这篇论文的最大价值，是以**四大技术支柱**为主线，系统拆解构建有效 LLM 多智能体系统需要掌握的核心技术组件。

## 这篇论文在整条阅读主线里的位置

前几篇综述分别从不同角度覆盖了 LLM-MAS：

- 第 1 篇搭骨架：LLM-MAS 的基本组成与挑战；
- 第 2 篇看应用：LLM-MAS 的应用版图扩张；
- 第 3 篇讲协作：协作机制的五维分析框架；
- 第 4 篇论可信：安全、隐私、鲁棒性视角；
- **第 5 篇则聚焦技术实现层：用 Architecture、Planning、Memory、Frameworks 四个支柱，把"怎么做"讲清楚。**

换句话说，这篇论文最适合在你知道"多智能体有哪些协作关系"之后，进一步追问"具体用什么技术把这些协作落地"。

## 论文基本信息

| 项目 | 内容 |
|------|------|
| **标题** | LLMs Working in Harmony: A Survey on the Technological Aspects of Building Effective LLM-Based Multi Agent Systems |
| **作者** | RM Aratchige, Dr. WMKS Ilmini |
| **单位** | General Sir John Kotelawala Defence University, Sri Lanka |
| **发表时间** | 2025 年 3 月 |
| **发表平台** | arXiv 预印本 (arXiv:2504.01963) |
| **核心问题** | 构建高效 LLM 多智能体系统需要哪些基础技术？哪些技术在实践中最有效？ |
| **研究方法** | 系统文献综述，聚焦 Architecture / Planning / Memory / Technologies and Frameworks 四大支柱 |
| **所用数据/文献来源** | Google Scholar, IEEE Xplore, arXiv；筛选与 LLM 多智能体直接相关的论文 |
| **核心结论** | MoA 架构在多智能体协作中表现最优；ReAct 在规划-行动整合上效果最好；记忆架构和框架选择高度依赖应用场景 |
| **不足与局限** | 文献覆盖以 arXiv 预印本为主，部分论文尚未经过同行评审；对四大支柱之间的交叉联动分析较浅；未提供可量化的基准评测比较 |

## 论文最核心的四柱框架

作者把 LLM-MAS 的技术全景归纳为四个关键维度：

| 维度 | 它回答的问题 | 核心代表工作 |
|------|------------|------------|
| Architecture | agent 怎么组织在一起？ | CMD, Chain-of-Agents, Agent Forest, MoA |
| Planning | agent 怎么思考和决策？ | AdaPlanner, ChatCoT, KnowAgent, RAP, ToT, ReAct |
| Memory | agent 怎么记住和检索信息？ | VectorDB, RAG, ChatDB, MemoryBank, RET-LLM, SCM |
| Technologies/Frameworks | 用什么工程工具落地？ | AutoGen, CAMEL, CrewAI, MetaGPT, LangGraph |

> [!tip] 理解提示（非原文）
> 前几篇综述常把 planning、memory、architecture 混在"agent 能力"里讲；这篇的贡献是把它们拆成独立的技术支柱，每个支柱都有独立的对比分析。

## 如果只记住 7 个精华判断

### 1. 构建 LLM-MAS 不是拼积木，而是搭四根柱子
只用一个大 prompt + 多个 agent 实例，不是真正的多智能体系统。这篇论文的主张是：你必须同时考虑架构拓扑、规划策略、记忆机制和工程框架，缺一不可。

### 2. MoA（Mixture-of-Agents）是当前架构设计的集大成者
作者对四种架构做了横向对比，认为 MoA 的 proposer + aggregator 分层设计最优雅：proposer 负责产生多样性，aggregator 负责质量整合。但它也有 bug——不是所有模型在两个角色上都同样好，WizardLM 擅长提议但不擅长聚合。

### 3. ReAct 是规划-行动整合的最优范式
将 reasoning 和 acting 交替进行，同时更新思维链和行动序列。作者对六个规划框架的对比中，ReAct 因其简单有效、可解释性强、幻觉控制好而排在第一推荐位。

### 4. 记忆架构没有"万能药"，而是"看菜吃饭"
作者明确指出：短期记忆适合快速响应场景，长期记忆适合需要跨对话保持上下文的场景。关键不是选哪个好，而是你的应用需要哪种记忆能力。

### 5. 框架选择取决于工程需求，而非技术优越性
AutoGen、CAMEL、CrewAI、MetaGPT、LangGraph 各有侧重：AutoGen 擅长对话编程，MetaGPT 擅长 SOP 驱动的复杂协作，LangGraph 擅长图结构工作流。没有哪个全面碾压，只有哪个更匹配你的场景。

### 6. 规模化困境是真实存在的技术瓶颈
Agent Forest 实验表明，增加 agent 数量确实提升性能，但收益递减；MoA 增加 aggregator 层数也引入管理复杂度。更多 agent 不等于更强系统。

### 7. 这篇论文填补了"技术选型指南"的空白
之前的综述更多回答"what"和"why"，这篇回答的是"which"和"how"——当你想搭一个 LLM-MAS 时，每个技术维度上该优先考虑什么方案。

## 一张技术选型速查表

| 你的需求 | 推荐技术 | 理由 |
|---------|---------|------|
| 多 agent 协同推理 | MoA 架构 | proposer+aggregator 分层，兼顾多样性与质量 |
| 长文本处理 | Chain-of-Agents | 分段处理 + 管理 agent 汇总，突破单模型 token 限制 |
| 辩论式决策 | CMD | 模拟人类辩论，多轮讨论聚合观点 |
| 规划-行动整合 | ReAct | 推理与行动交替，可解释性强，幻觉控制好 |
| 自适应规划 | AdaPlanner | 闭环反馈，动态调整计划 |
| 知识增强规划 | RAP / KnowAgent | 结合外部知识库提升规划准确性 |
| 思维树搜索 | Tree of Thoughts | 需要探索多条推理路径的复杂问题 |
| 短期记忆 | 向量数据库 + RAG | 快速检索，动态知识注入 |
| 长期记忆 | MemoryBank / SCM | 跨对话保持、遗忘曲线管理 |
| 符号化精确记忆 | ChatDB / RET-LLM | SQL 操作精确操控、三元组知识存储 |
| 快速原型 | AutoGen | 对话编程，灵活组合 |
| 角色扮演协作 | CAMEL | 角色启始提示，自主合作 |
| SOP 驱动开发 | MetaGPT | 编码标准化操作流程，减少幻觉级联 |
| 图结构工作流 | LangGraph | 循环图工作流，有状态管理 |

## 这篇论文与前几篇的互补关系

| 论文 | 主问题 | 你读完后的收获 |
|------|--------|--------------|
| 第 1 篇 | LLM-MAS 是什么 | 搭骨架 |
| 第 2 篇 | LLM-MAS 能做什么 | 看应用地图 |
| 第 3 篇 | LLM-MAS 怎么协作 | 学机制语言 |
| 第 4 篇 | LLM-MAS 安全可信吗 | 建立审稿人批判视角 |
| 第 5 篇 | LLM-MAS 用什么技术落地 | 掌握技术选型框架 |

> [!note] 译者注（非原文）
> 如果你未来要自己动手搭一个 LLM-MAS 系统，第 5 篇是最直接的"施工手册"。它不像第 3 篇那样高度抽象，而是给了每个技术维度上的具体选项、优劣对比和选型建议。

## 这篇论文最值得抄走的技术评估框架

当你以后评估一篇新的多智能体方法论文时，可以强制自己回答四个问题：

1. **架构**：agent 之间的组织拓扑是什么？是否有中心调度者？通信模式是怎样的？
2. **规划**：agent 如何制定和执行计划？是静态预设还是动态调整？推理和行动是分离还是整合？
3. **记忆**：系统如何存储和检索信息？是参数记忆还是外部记忆？短期还是长期？
4. **工程框架**：用什么工具实现的？是否支持扩展和集成外部数据源？

如果这四个问题答不清，你其实还没有真正理解这篇方法论文的技术方案。

## 本篇分篇导航

| part | 文件 | 覆盖内容 |
|------|------|---------|
| 00 | [[raw/lessons/Multi-Agent-Survey/05-llms-working-in-harmony/00-llms-harmony-essence\|00-essence]] | 论文精华、技术选型速查、研究价值 |
| 01 | [[raw/lessons/Multi-Agent-Survey/05-llms-working-in-harmony/01-llms-harmony-introduction-and-architecture\|01-introduction-and-architecture]] | 导论 + 架构篇（CMD, CoA, Agent Forest, MoA） |
| 02 | [[raw/lessons/Multi-Agent-Survey/05-llms-working-in-harmony/02-llms-harmony-planning\|02-planning]] | 规划篇（AdaPlanner, ChatCoT, KnowAgent, RAP, ToT, ReAct） |
| 03 | [[raw/lessons/Multi-Agent-Survey/05-llms-working-in-harmony/03-llms-harmony-memory\|03-memory]] | 记忆篇（VectorDB, RAG, ChatDB, MemoryBank, RET-LLM, SCM） |
| 04 | [[raw/lessons/Multi-Agent-Survey/05-llms-working-in-harmony/04-llms-harmony-frameworks-and-conclusion\|04-frameworks-and-conclusion]] | 框架篇 + 方法论 + 讨论 + 结论 |

## 本篇小结

> [!note] 译者注（非原文）
> 这篇"精华总览"最重要的作用，是让你先记住一句话：
> **LLM 多智能体系统的技术落地，不是靠一个银弹框架，而是靠 Architecture + Planning + Memory + Frameworks 四根柱子的协同设计。**
>
> 只要你记住"架构怎么组织、规划怎么思考、记忆怎么存储、框架怎么落地"这四个问题，后续读任何 LLM-MAS 方法论文都会思路更清晰。

## 相关页面

- [[raw/lessons/Multi-Agent-Survey/03-collaboration-mechanisms/00-collaboration-mechanisms-essence]]
- [[raw/lessons/Multi-Agent-Survey/04-trust-worthiness/00-trust-essence]]
- [[raw/lessons/Multi-Agent-Survey/05-llms-working-in-harmony/01-llms-harmony-introduction-and-architecture]]

## 下一步学习

- 上一篇：[[raw/lessons/Multi-Agent-Survey/04-trust-worthiness/00-trust-essence]]
- 下一篇：[[raw/lessons/Multi-Agent-Survey/05-llms-working-in-harmony/01-llms-harmony-introduction-and-architecture\|01-introduction-and-architecture]]
