---
type: summary
tags: [多智能体, multi-agent, llm-agent, survey, reading-guide]
created: 2026-05-07
updated: 2026-05-07
sources:
  - https://arxiv.org/abs/2402.01680
  - https://arxiv.org/abs/2412.17481
  - https://arxiv.org/abs/2501.06322
  - https://arxiv.org/abs/2502.14321
  - https://arxiv.org/abs/2504.01963
  - https://aclanthology.org/2025.emnlp-main.1403/
  - https://api.openalex.org/
---

# 多智能体综述论文阅读手册

> 目标：用一组“新 + 前沿 + 有引用 + 尽量高质量”的综述论文，帮你快速建立 **LLM 多智能体（LLM-based Multi-Agent Systems, LLM-MAS）** 的基本概念、主流框架、协作机制、通信范式、工程挑战与前沿应用视角。

## Goal
- 建立多智能体方向的基本地图：这个领域到底在研究什么。
- 先分清 **单 Agent**、**多 Agent**、**协作机制**、**通信机制**、**系统工程** 这几层。
- 形成一条从“概念入门”到“研究前沿”的阅读主线，而不是一上来就被零散论文淹没。
- 为后续做研究方向判断、读具体方法论文、设计自己的问题切入点打基础。

## 我这次的筛选原则

### 你要求的目标
你要的是：
- **有关多智能体的综述论文**
- **高质量**
- **前沿、比较新**
- **尽量有引用/有含金量**
- **有助于建立基础认知，而不是只看某个很窄的子问题**

### 我实际采用的筛选标准
1. **优先 general survey，而不是单一垂域论文**
   - 比如自动驾驶、多机器人、某个垂类应用的 survey，我这次没有放进主线。
2. **优先 2024-2025 的新综述**
   - 因为 LLM 多智能体变化很快，2023 年很多综述已经有点旧。
3. **保留 1 篇高引用“奠基型综述”**
   - 即便年份略早，也必须有一篇帮助你搭骨架。
4. **优先能回答“这个领域整体长什么样”**
   - 而不是只讲 creativity、robotics、workflow 等单一分支。
5. **引用数作为参考，不作为唯一标准**
   - 新论文天然引用少，所以我同时考虑：新颖度、覆盖面、框架感、是否适合建立认知。

### 本次未纳入主线但值得知道的材料
这些我认为“有价值，但不适合作为你的主线第一批”：
- **Game-Theoretic Lens on LLM-based Multi-Agent Systems (2026)**：更像理论视角/分析框架，不是你当前最需要的入门主线。
- **A Survey on Agent Workflow -- Status and Future (2025)**：更偏 workflow / orchestration，和 multi-agent 有交叉，但不够聚焦“多智能体综述”主线。
- 各类 **multi-robot / autonomous driving / embodied** survey：质量不差，但太垂直，不利于你先搭整体认知。

## 推荐阅读顺序

> 核心原则：**先搭地图，再看机制；先看 general survey，再看特化视角。**

| 顺序 | 文件 | 角色定位 | 为什么现在读 | 引用情况（OpenAlex，检索于 2026-05-07） |
|---|---|---|---|---|
| 1 | [[01-large-language-model-based-multi-agents-survey-of-progress-and-challenges-2024.pdf]] | 奠基综述 | 引用最高，最适合先搭骨架 | 56 |
| 2 | [[02-a-survey-on-llm-based-multi-agent-system-recent-advances-and-new-frontiers-in-application-2024.pdf]] | 新版全景图 | 用更新视角刷新应用版图与问题定义 | 6 |
| 3 | [[03-multi-agent-collaboration-mechanisms-a-survey-of-llms-2025.pdf]] | 协作机制主线 | 适合把“多智能体到底怎么协作”读清楚 | 15 |
| 4 | [[04-beyond-self-talk-a-communication-centric-survey-of-llm-based-multi-agent-systems-2025.pdf]] | 通信机制主线 | 从 communication 角度切开，是很好的研究视角 | 8 |
| 5 | [[05-llms-working-in-harmony-a-survey-on-the-technological-aspects-of-building-effective-llm-based-multi-agent-systems-2025.pdf]] | 工程技术视角 | 重点看 architecture / memory / planning / framework | 1 |
| 6 | [[06-creativity-in-llm-based-multi-agent-systems-a-survey-2025.pdf]] | 专题延伸 | 不是基础主线，但能让你看到高层次应用与评测问题 | 2 |

## 最建议的阅读节奏

### Phase 1：先建立全局地图（1 → 2）
目标：回答下面 4 个问题
- 什么叫 LLM-based Multi-Agent System？
- 它和单 Agent 的本质差别是什么？
- 这个领域的主要应用场景有哪些？
- 当前研究的共性挑战是什么？

### Phase 2：再看“协作是怎么发生的”（3 → 4）
目标：回答下面 4 个问题
- Agent 之间有哪些协作关系？
- 任务如何分解、角色如何分配？
- 通信协议、拓扑结构、中心化/去中心化各有什么利弊？
- 为什么 communication 会成为多智能体系统的关键瓶颈？

### Phase 3：最后看“系统怎么落地”（5）
目标：回答下面 3 个问题
- 如果真的要搭一个多智能体系统，工程上最关键的模块是什么？
- memory / planning / framework / orchestration 分别扮演什么角色？
- 当前系统真正卡在 scalability、coordination、benchmarking 的哪一层？

### Phase 4：看一个高质量专题外延（6）
目标：回答下面 3 个问题
- 多智能体不只是“做任务”，还可以怎样提升创造性输出？
- 当任务目标不是 correctness，而是 creativity 时，评测会怎么变？
- 多智能体系统的价值，是不是在某些任务上比单 Agent 更明显？

## 每篇论文该怎么读

---

## 1. Large Language Model based Multi-Agents: A Survey of Progress and Challenges (2024)
- 文件：[[01-large-language-model-based-multi-agents-survey-of-progress-and-challenges-2024.pdf]]
- 原文：<https://arxiv.org/abs/2402.01680>
- 定位：**奠基综述 / 主线第一篇**
- 引用：**56**（本批里最高）

### 为什么它排第一
这篇最大的价值不是“最新”，而是 **最适合先搭认知骨架**：
- 它是较早系统总结 LLM 多智能体的综述之一；
- 已经有一定引用，说明社区认可度不错；
- 它会帮助你先把“多智能体系统的全景图”建立起来。

### 读这篇时重点抓什么
1. **定义**：作者怎么定义 LLM-based multi-agent systems。
2. **应用场景**：复杂任务、world simulation、自动化问题求解等。
3. **系统要素**：agent profile、communication、planning、benchmark。
4. **挑战**：协作为什么难，瓶颈在哪里。

### 读完后你应该能回答
- 多智能体为什么不是“多开几个 prompt”这么简单？
- 多智能体系统最小要素有哪些？
- 为什么这个领域从单 Agent 走向多 Agent？

### 我给你的建议
这篇不要细抠所有例子，重点做 **框架型笔记**。
你要把这篇当成“总目录”，后面很多新论文其实都在细化它里面的某几个维度。

---

## 2. A Survey on LLM-based Multi-Agent System: Recent Advances and New Frontiers in Application (2024)
- 文件：[[02-a-survey-on-llm-based-multi-agent-system-recent-advances-and-new-frontiers-in-application-2024.pdf]]
- 原文：<https://arxiv.org/abs/2412.17481>
- 定位：**新版全景图 / 主线第二篇**
- 引用：**6**

### 为什么它排第二
这篇比第 1 篇新，适合在你已有基本骨架后，用更近一轮的视角去更新认知。
它的优势在于：
- 时间更近；
- 更明确地强调了 **applications**；
- 比较适合回答“这个方向近一年到底长成什么样了”。

### 这篇要抓什么
1. **LLM-MAS 的定义边界**：哪些系统算，哪些不太算。
2. **应用分层**：
   - 复杂任务求解
   - 特定场景模拟
   - generative agents 评估/模拟
3. **未来方向**：作者认为新 frontier 在哪里。

### 读完后你应该能回答
- LLM-MAS 的主流应用面已经扩展到了哪些地方？
- “做任务”和“做模拟”这两类系统的差别是什么？
- 如果要选一个研究切口，哪些方向已经拥挤，哪些方向还有空间？

### 和第 1 篇的关系
- **第 1 篇**：像“老地图”，骨架稳定。
- **第 2 篇**：像“新版地图”，补近一年的增长区域。

两篇一起读，你会明显感觉这个领域是怎么从“概念兴起”走到“应用分化”的。

---

## 3. Multi-Agent Collaboration Mechanisms: A Survey of LLMs (2025)
- 文件：[[03-multi-agent-collaboration-mechanisms-a-survey-of-llms-2025.pdf]]
- 原文：<https://arxiv.org/abs/2501.06322>
- 定位：**协作机制核心综述 / 主线第三篇**
- 引用：**15**

### 为什么它很重要
如果说前两篇在回答“这个领域是什么”，那这篇开始真正回答：

**“多智能体系统里的协作，到底是怎么设计出来的？”**

这对你后面想读更研究化、更方法化的论文非常关键。

### 这篇的独特价值
根据摘要，它把 collaboration 拆成了若干关键维度：
- actors
- collaboration types
- structures
- strategies
- coordination protocols

这说明它不是泛泛讲应用，而是在试图建立 **协作机制 taxonomy**。

### 读这篇时要重点记录
1. **协作类型**：cooperation / competition / coopetition。
2. **组织结构**：peer-to-peer / centralized / distributed。
3. **协作策略**：role-based、model-based 等。
4. **协调协议**：任务如何交接、冲突如何处理、结果如何汇总。

### 读完后你应该能回答
- 多智能体系统常见的拓扑结构有哪些？
- 一个系统为什么要选择 centralized，而不是 decentralized？
- “角色分工”到底是 prompt design，还是系统设计？

### 我给你的建议
这篇非常适合做 **表格笔记**。
建议你自己整理一张表：

| 维度 | 常见选项 | 优点 | 缺点 | 适合场景 |
|---|---|---|---|---|
| 结构 | centralized / distributed / peer-to-peer |  |  |  |
| 协作关系 | cooperation / competition / coopetition |  |  |  |
| 策略 | role-based / model-based / planner-led |  |  |  |

---

## 4. Beyond Self-Talk: A Communication-Centric Survey of LLM-Based Multi-Agent Systems (2025)
- 文件：[[04-beyond-self-talk-a-communication-centric-survey-of-llm-based-multi-agent-systems-2025.pdf]]
- 原文：<https://arxiv.org/abs/2502.14321>
- 定位：**通信视角核心综述 / 主线第四篇**
- 引用：**8**

### 为什么这篇值得单独拿出来
很多综述会把 communication 当成多智能体里的一个子模块，但这篇直接把它提升成 **主视角**。
这很有研究味道，因为真正复杂的多智能体系统，问题经常不只是“单个 agent 能力不够”，而是：
- 消息怎么传？
- 什么时候传？
- 传什么？
- 传多少？
- 谁决定停？

### 这篇最值得看的点
1. **communication-centric framework**：把系统层和系统内部层分开看。
2. **交互机制**：agents 如何 interact / negotiate / coordinate。
3. **挑战**：
   - efficiency
   - security vulnerabilities
   - benchmarking
   - scalability

### 读完后你应该能回答
- 为什么 communication 不是“附属模块”，而是核心瓶颈？
- 多智能体系统的失败，很多时候是不是通信失败？
- 如果以后你做论文，communication 有没有可能成为一个很好的切入点？

### 我给你的建议
如果你以后想往：
- 动态拓扑
- 协作效率
- 多 agent 一致性
- 安全/鲁棒性

这些方向走，这篇必须认真读。

---

## 5. LLMs Working in Harmony: A Survey on the Technological Aspects of Building Effective LLM-Based Multi Agent Systems (2025)
- 文件：[[05-llms-working-in-harmony-a-survey-on-the-technological-aspects-of-building-effective-llm-based-multi-agent-systems-2025.pdf]]
- 原文：<https://arxiv.org/abs/2504.01963>
- 定位：**工程技术栈视角 / 主线第五篇**
- 引用：**1**

### 为什么它虽然引用低，但仍然值得保留
它不一定是你这批里“最硬核”的综述，但它有个价值：

**它试图用系统工程视角去回答：一个有效的 LLM-MAS 到底由哪些技术模块拼起来。**

这对于你后面如果要真正搭系统、看框架、做 benchmark，会很有用。

### 重点看什么
根据摘要，这篇聚焦四个技术区：
- Architecture
- Memory
- Planning
- Technologies / Frameworks

### 读完后你应该能回答
- 多智能体系统的工程骨架有哪些层？
- memory 和 planning 在 multi-agent 里为什么比单 agent 更复杂？
- framework 帮你解决了什么，又遮蔽了什么？

### 我的建议
这篇**不要当第一篇看**。
因为如果没有前四篇的认知骨架，你会容易把它看成“工具/框架盘点”。
但如果你已经读完 1-4，这篇就会帮你把概念落到系统搭建层。

---

## 6. Creativity in LLM-based Multi-Agent Systems: A Survey (2025)
- 文件：[[06-creativity-in-llm-based-multi-agent-systems-a-survey-2025.pdf]]
- 原文：<https://aclanthology.org/2025.emnlp-main.1403/>
- 定位：**高质量专题延伸 / 主线第六篇（可选强化）**
- 引用：**2**
- 备注：**EMNLP 2025 论文集正式发表版本**

### 为什么它被放在最后
它不是 general survey，而是一个 **专题型 survey**。
但它有两个优点：
1. 是正式会议论文，不只是 arXiv 预印本；
2. 让你看到多智能体不只是在“任务完成”层面有价值，还能在 **创造性生成** 上展现独特优势。

### 适合你什么时候读
当你已经知道：
- 什么是多智能体
- 它怎么协作
- 它怎么通信
- 它怎么搭系统

然后你想看：
**“这个框架在一个高阶任务领域里到底能带来什么额外价值？”**

### 重点抓什么
1. creativity 在 multi-agent 中是如何定义的。
2. 为什么多 agent 比单 agent 更可能提升创意性输出。
3. 评价 creative MAS 的 benchmark / metric 为什么更难。

### 这篇的作用
它会让你从“系统构建者视角”再往前一步，进入：
- **能力边界**
- **评测难题**
- **应用价值**

这对建立研究直觉很有帮助。

## 一句话版阅读路线

如果你时间不多，先按这个最小闭环走：

**1 → 2 → 3 → 4**

如果你想把工程落地也纳入：

**1 → 2 → 3 → 4 → 5**

如果你还想看一个专题高阶应用：

**1 → 2 → 3 → 4 → 5 → 6**

## 建议你读完后自己整理的 8 个核心问题

1. LLM-MAS 的最小定义是什么？
2. 多智能体相对单智能体的真正增益在哪里？
3. 常见协作结构有哪些？
4. communication 在系统里扮演什么地位？
5. memory / planning / role assignment 各自怎么影响协作质量？
6. 当前最典型的评测短板是什么？
7. 当前这个方向最值得做研究的问题在哪：效率、鲁棒性、安全、拓扑、自组织、评测，还是别的？
8. 如果要自己做一个研究问题，我会选“机制问题”还是“应用问题”？

## 我对这批论文的总判断

### 最适合建立基础认知的
- [[01-large-language-model-based-multi-agents-survey-of-progress-and-challenges-2024.pdf]]
- [[02-a-survey-on-llm-based-multi-agent-system-recent-advances-and-new-frontiers-in-application-2024.pdf]]

### 最适合理解“多智能体为什么复杂”的
- [[03-multi-agent-collaboration-mechanisms-a-survey-of-llms-2025.pdf]]
- [[04-beyond-self-talk-a-communication-centric-survey-of-llm-based-multi-agent-systems-2025.pdf]]

### 最适合从工程系统视角补全认知的
- [[05-llms-working-in-harmony-a-survey-on-the-technological-aspects-of-building-effective-llm-based-multi-agent-systems-2025.pdf]]

### 最适合看高阶专题延伸的
- [[06-creativity-in-llm-based-multi-agent-systems-a-survey-2025.pdf]]

## Sources
- [Paper] Large Language Model based Multi-Agents: A Survey of Progress and Challenges — https://arxiv.org/abs/2402.01680
- [Paper] A Survey on LLM-based Multi-Agent System: Recent Advances and New Frontiers in Application — https://arxiv.org/abs/2412.17481
- [Paper] Multi-Agent Collaboration Mechanisms: A Survey of LLMs — https://arxiv.org/abs/2501.06322
- [Paper] Beyond Self-Talk: A Communication-Centric Survey of LLM-Based Multi-Agent Systems — https://arxiv.org/abs/2502.14321
- [Paper] LLMs Working in Harmony: A Survey on the Technological Aspects of Building Effective LLM-Based Multi Agent Systems — https://arxiv.org/abs/2504.01963
- [Paper] Creativity in LLM-based Multi-Agent Systems: A Survey — https://aclanthology.org/2025.emnlp-main.1403/
- [Metadata] OpenAlex citation data — https://api.openalex.org/

## Open Questions
- 2026 年会不会出现一篇真正“统合 2024-2025 全部主线”的新总综述？
- communication / topology / self-organization 哪个会成为下一阶段最有研究价值的核心问题？
- 多智能体系统的评测，会不会逐渐从任务成功率转向更细的协作质量指标？

## Next Step
- 先按顺序读 **1 和 2**，然后我可以继续帮你做第二步：
  1. 为每篇论文写中文精读笔记；
  2. 提炼成一张“多智能体研究地图”；
  3. 结合你的目标，进一步筛选最适合做研究切口的 5-10 篇方法论文。
