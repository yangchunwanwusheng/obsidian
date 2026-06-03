---
type: lesson
tags: [多智能体, multi-agent, survey, 协作机制, orchestration, 论文翻译, 学习笔记]
created: 2026-05-08
updated: 2026-05-08
topic: LLM 多智能体综述中文精读（第 3 篇-06）协调编排与经验总结
difficulty: intermediate
prerequisites:
  - [[raw/lessons/Multi-Agent-Survey/03-collaboration-mechanisms/05-collaboration-mechanisms-structures]]
sources:
  - [[raw/paper/multi-agent/03-multi-agent-collaboration-mechanisms-a-survey-of-llms-2025.pdf]]
  - https://arxiv.org/abs/2501.06322
status: completed
series:
  name: "Multi-Agent Survey｜Collaboration Mechanisms（2025）"
  part: 6
paper_meta:
  paper_id: "multi-agent-survey-03"
  title: "Multi-Agent Collaboration Mechanisms: A Survey of LLMs"
  authors: "Khanh-Tung Tran, Dung Dao, Minh-Duong Nguyen, Quoc-Viet Pham, Barry O’Sullivan, Hoang D. Nguyen"
  year: "2025"
  link: "https://arxiv.org/abs/2501.06322"
  section: "Section 4.5 + 4.6"
---

# LLM 多智能体综述中文精读（第 3 篇-06）：协调编排与经验总结

> 本篇覆盖 **Section 4.5 Coordination and Orchestration** 与 **Section 4.6 Summary and Lessons Learned**。这一节开始真正触及“系统怎么跑起来”。

## 本章导读

> [!note] 译者注（非原文）
> 如果说前面几节分别回答了：
> - 什么关系；
> - 采用什么策略；
> - 使用什么拓扑；
>
> 那这一节就在回答：
> **这些东西最后怎么被编排成一个真实可运行系统？**

## 正文翻译

### 原文标题：4.5 Coordination and Orchestration

作者指出，协调与编排关注的不是单条 collaboration channel 自身，而是**多个 channel 之间的关系与互动**。它们规定：
- collaboration channels 如何被创建；
- 以什么顺序发生；
- 各自具有什么特征；
- 如何共同构成整体多智能体交互流程。

根据设计方式，作者把 coordination / orchestration 分成两类：
1. **Static architecture**
2. **Dynamic architecture**

#### 4.5.1 Static Architecture

静态架构依赖领域知识与预定义规则来建立 collaboration channels。它的目标是让交互严格贴合领域要求，并使用先验知识优化系统性能。

作者举出的典型模式是：**顺序串联（sequential chaining）**。
- 一个 agent 的输出喂给下一个 agent；
- 不同 agent 分别承担 expert、summarizer 等职责；
- 通过预先设计好的流程解决复杂问题。

例如：
- 三个 LLM agent 顺序连接，前两个提供互补视角，第三个做总结；
- MapCoder 中 recall、planning、code generation、debugging 等 agent 的协作流程是显式写定的；
- MACRec 在推荐任务中预设 Manager、User/Item Analyst、Reflector 等角色及其互动顺序；
- 文学翻译多 agent 协作也沿用传统出版工作流映射成预定义 channel。

作者认为静态架构的优点是：
- 流程清晰；
- 易于控制与解释；
- 适合已有强领域知识的任务。

但它的问题是：
- 泛化能力有限；
- 面对新任务或新环境适应性较弱；
- 流程一旦设计不佳，很难自动补救。

#### 4.5.2 Dynamic Architecture

动态架构则面向变化环境与变化任务。它依赖管理型 agent 或自适应机制，在运行时决定：
- 谁该参与；
- 角色如何分配；
- channel 如何生成；
- 哪些步骤并行，哪些串行。

作者举出的两个代表例子：

1. **Solo Performance Prompting (SPP)**
   - 根据输入动态识别 relevant personas；
   - 由管理 agent 生成带定制 system prompt 的多个 agent；
   - 让这些 agent 协作 brainstorm 和 refinement。

2. **图编排机制 / DAG orchestration**
   - Orchestrator 根据用户输入动态构建 DAG；
   - 节点表示任务；
   - 边表示依赖与 collaboration channels；
   - Delegator 汇总所有完成结果形成最终响应。

作者认为动态架构的核心价值在于：
- 能针对任务即时重构协作网络；
- 更适合复杂多步任务；
- 能提升并行性与可扩展性。

但代价是：
- 系统更难设计；
- 需要更强的 runtime 控制；
- 错误恢复和稳定性问题更突出。

### 原文标题：4.6 Summary and Lessons Learned

作者总结说，LLM-based multi-agent collaborative systems 的兴起，来自 LLM 作为 central processing brain 的成功，以及对人类协作方式的借鉴。

这些系统通常通过以下方式工作：
- 将复杂任务分解成子任务；
- 给 agent 指定角色，例如 software engineer；
- 借助 collaboration channels 实现规划、协调与执行。

作者再次重申，协作通道由以下维度刻画：
- actors
- type
- structure
- strategy

并指出这些 channel 有时还能展现高级行为，例如 Theory of Mind。

但作者也明确提醒：
- LLM 原本是为 standalone 使用训练的；
- 它们并不是专门为 collaborative tasks 训练；
- 因此当前很多多智能体机制仍然不清晰，既给理论研究留下问题，也让实际系统更难解释与预测。

作者给出的主要经验包括：

1. **Effective Collaboration Channels**
   建立清晰可靠的协作通道极其关键。好的 collaboration mechanism 能让 MAS 超过单 agent；差的设计反而会输给单 agent。

2. **Collective Domain Knowledge**
   领域知识对于设计协作架构和 crafting system prompts 很重要。很多 channel 都是围绕领域知识显式预定义的。

3. **Adaptive Role and Channel Assignment**
   某些任务中，让系统动态分配角色和 channel 比固定分配更灵活更有效。

4. **Optimal Collaborative Strategy**
   - 规则驱动适合高一致性、强规范任务；
   - 角色驱动适合工作分工明确的任务；
   - 模型驱动适合不确定、动态环境。

5. **Scalability Considerations**
   agent 数量增加后，协调复杂度显著上升，需要更可扩展的架构与算法。

6. **Ethical and Safety Considerations**
   agent 必须在伦理边界内行动，否则协作系统会放大错误与风险。

## 本章小结

> [!note] 译者注（非原文）
> 这节最关键的工程直觉是：
> - **静态编排**更像写死的 pipeline；
> - **动态编排**更像运行时生成 workflow；
> - 真正好的系统，不只是“有多个 agent”，而是有好的编排机制。
>
> 作者最后的经验总结也非常实在：多智能体系统成败，往往不取决于 agent 数量，而取决于 channel 设计、角色分配、策略匹配和扩展控制。

## 术语对照

| 英文术语 | 中文译法 | 本章语境说明 |
|---|---|---|
| coordination | 协调 | 多个 channel/agent 如何配合 |
| orchestration | 编排 / 调度 | 流程如何被组织与推进 |
| static architecture | 静态架构 | 流程预定义 |
| dynamic architecture | 动态架构 | 流程运行时生成或调整 |
| sequential chaining | 顺序串联 | 一个 agent 的输出接到下一个 |
| DAG orchestration | DAG 编排 | 用依赖图组织多步多 agent 流程 |

## 思考题

### 题目 1：为什么很多早期工程系统偏爱静态编排？

> [!tip] 理解提示（非原文）
> 想想：当你首先追求“能跑、能控、能调试”时，哪种方案更现实？

> [!note] 译者注（非原文）
> 因为静态流程更好解释、更容易验证，也更利于在已有领域知识下快速做出稳定原型，所以往往是工程起点。

### 题目 2：为什么动态编排被视为更有前途？

> [!warning] 易混点（非原文）
> 动态编排不是“更炫”，而是更贴近真实复杂任务需求。

> [!note] 译者注（非原文）
> 因为真实世界任务往往不完全可预先写死。动态编排允许系统根据任务内容、依赖结构和 agent 能力实时重组工作流，更可能在复杂场景下取得更高上限。

## 相关页面

- [[raw/lessons/Multi-Agent-Survey/03-collaboration-mechanisms/05-collaboration-mechanisms-structures]]
- [[raw/lessons/Multi-Agent-Survey/03-collaboration-mechanisms/07-collaboration-mechanisms-applications]]
- [[raw/paper/multi-agent/03-multi-agent-collaboration-mechanisms-a-survey-of-llms-2025.pdf]]

## 下一步学习

- 上一篇：[[raw/lessons/Multi-Agent-Survey/03-collaboration-mechanisms/05-collaboration-mechanisms-structures|05-structures]]
- 下一篇：[[raw/lessons/Multi-Agent-Survey/03-collaboration-mechanisms/07-collaboration-mechanisms-applications|07-applications]]
