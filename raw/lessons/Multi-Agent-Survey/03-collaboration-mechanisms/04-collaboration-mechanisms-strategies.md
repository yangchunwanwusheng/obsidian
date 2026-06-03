---
type: lesson
tags: [多智能体, multi-agent, survey, 协作机制, collaboration-strategy, 论文翻译, 学习笔记]
created: 2026-05-08
updated: 2026-05-08
topic: LLM 多智能体综述中文精读（第 3 篇-04）协作策略
difficulty: intermediate
prerequisites:
  - [[raw/lessons/Multi-Agent-Survey/03-collaboration-mechanisms/03-collaboration-mechanisms-types]]
sources:
  - [[raw/paper/multi-agent/03-multi-agent-collaboration-mechanisms-a-survey-of-llms-2025.pdf]]
  - https://arxiv.org/abs/2501.06322
status: completed
series:
  name: "Multi-Agent Survey｜Collaboration Mechanisms（2025）"
  part: 4
paper_meta:
  paper_id: "multi-agent-survey-03"
  title: "Multi-Agent Collaboration Mechanisms: A Survey of LLMs"
  authors: "Khanh-Tung Tran, Dung Dao, Minh-Duong Nguyen, Quoc-Viet Pham, Barry O’Sullivan, Hoang D. Nguyen"
  year: "2025"
  link: "https://arxiv.org/abs/2501.06322"
  section: "Section 4.3"
---

# LLM 多智能体综述中文精读（第 3 篇-04）：协作策略

> 本篇覆盖 **Section 4.3 Collaboration Strategies**。作者把 MAS 的协作策略分为三类：**规则驱动、角色驱动、模型驱动**。

## 本章导读

> [!note] 译者注（非原文）
> 如果“协作类型”回答的是 agent 关系是什么，那么“协作策略”回答的是：
> **它们按照什么逻辑来协作？**

## 正文翻译

### 原文标题：4.3 Collaboration Strategies

作者指出，MAS 中常见的三类协作策略是：
1. **Rule-based**
2. **Role-based**
3. **Model-based**

这些策略决定了 agent 如何组织行动、如何做决策、以及如何处理不确定性。

### 原文标题：4.3.1 Rule-based Protocols

在**规则驱动协议**中，agent 之间的交互被预定义规则严格控制。系统通过一组明确规则约束 collaboration channels，使 agent 按既定规则协调行动，而不是依赖概率判断或角色自发发挥。

作者给出的例子包括：
- 受社会心理学启发的辩论与多数投票规则；
- 基于事件触发条件的动态规则协议，用以减少不必要通信；
- 类似同行评审的规则：agent 互相 critique、revision、refine；
- consensus seeking 任务中，靠规则来协商并收敛到共同目标。

作者总结规则驱动的优点：
- 效率高；
- 可预测；
- 易于实现与调试；
- 在有明确程序和低变化任务里很有效；
- 通过给每个 agent 施加限制，也有助于公平性。

缺点是：
- 缺乏适应性；
- 面对意外情形时常常失效；
- 任务越复杂，规则数量越可能爆炸；
- 可扩展性与维护成本会变差。

### 原文标题：4.3.2 Role-based Protocols

在**角色驱动协议**中，每个 agent 被赋予明确预定义角色或工作分工，每个角色负责某一部分目标。作者指出，AgentVerse、MetaGPT、RoCo、BabyAGI 等都体现了 role-based 协作的有效性。

这类系统的典型做法是：
- 给不同 agent 设定 distinct role；
- 每个 role 绑定特定 expertise；
- 通过 SOP 或工作流把角色间依赖明确化；
- 让各 agent 在各自专业边界内主动协同，减少重复与冲突。

作者认为 role-based 的优势包括：
- 分工清晰；
- 模块化强；
- 组件可单独设计、实现、更新；
- 很适合模拟现实世界中专业岗位明确的环境，例如软件开发、企业协作、机器人分工。

但它的弱点在于：
- 如果角色定义不当，系统会僵硬；
- 角色之间的依赖会成为性能瓶颈；
- 沟通不畅或阻塞会严重影响整体效果。

### 原文标题：4.3.3 Model-based Protocols

**模型驱动协议**更强调在不确定环境中基于概率与推断做决策。作者指出，这类方法适合 perception 不确定、环境动态变化、需要持续更新判断的场景。

代表思路包括：
- 利用 **Theory of Mind** 推断其他 agent 的可能心理状态；
- 用概率逻辑推理推断人类目标或其他 agent 意图；
- 用 PGM、probabilistic timed automata 等模型处理不确定状态转移。

作者认为 model-based 的优势在于：
- 灵活；
- 对动态环境适应更强；
- 对噪声和意外有更高鲁棒性；
- 适合游戏、机器人、部分可观测环境等复杂场景。

缺点则是：
- 设计难；
- 部署复杂；
- 需要更复杂的环境与交互模型；
- 计算代价更高，实时性可能受限。

## 本章小结

> [!note] 译者注（非原文）
> 这节最值得带走的是一个选型直觉：
> - **规则驱动**：适合流程清晰、约束刚性的任务；
> - **角色驱动**：适合专业分工明确、工程上易落地的任务；
> - **模型驱动**：适合动态、不确定、需要持续推断的任务。
>
> 你完全可以把它理解成三种“系统脑回路”：
> - 按规章做事；
> - 按岗位做事；
> - 按环境推断做事。

## 术语对照

| 英文术语 | 中文译法 | 本章语境说明 |
|---|---|---|
| rule-based protocol | 规则驱动协议 | 依赖预定义规则进行协调 |
| role-based protocol | 角色驱动协议 | 依赖岗位分工和角色边界 |
| model-based protocol | 模型驱动协议 | 依赖概率推断与环境建模 |
| Theory of Mind | 心智理论 | 对其他 agent 心理状态的推断 |
| SOP | 标准操作流程 | 将角色间工作流程写死或写清 |
| probabilistic reasoning | 概率推理 | 在不确定环境里做决策 |

## 思考题

### 题目 1：为什么 role-based 在当前 LLM 多智能体系统里这么常见？

> [!tip] 理解提示（非原文）
> 想一想：给 agent 一句“你是 planner / coder / reviewer”，是不是比训练一个复杂协同控制器更容易落地？

> [!note] 译者注（非原文）
> 因为 role-based 最接近人类组织分工，解释性强、实现快、提示工程成本低，也便于模块化维护，所以成为当前工程系统最常见的范式。

### 题目 2：为什么 model-based 更强，但没有成为默认主流？

> [!warning] 易混点（非原文）
> “更强”不等于“更适合所有工程系统”。

> [!note] 译者注（非原文）
> 因为 model-based 的设计和部署成本更高，需要更复杂的推断结构与环境建模。很多实际系统优先追求可实现、可解释、可控，所以更常先用 rule-based 或 role-based。

## 相关页面

- [[raw/lessons/Multi-Agent-Survey/03-collaboration-mechanisms/03-collaboration-mechanisms-types]]
- [[raw/lessons/Multi-Agent-Survey/03-collaboration-mechanisms/05-collaboration-mechanisms-structures]]
- [[raw/paper/multi-agent/03-multi-agent-collaboration-mechanisms-a-survey-of-llms-2025.pdf]]

## 下一步学习

- 上一篇：[[raw/lessons/Multi-Agent-Survey/03-collaboration-mechanisms/03-collaboration-mechanisms-types|03-types]]
- 下一篇：[[raw/lessons/Multi-Agent-Survey/03-collaboration-mechanisms/05-collaboration-mechanisms-structures|05-structures]]
