---
type: lesson
tags: [多智能体, multi-agent, survey, 创造力, 创意生成, 论文精华, 学习笔记]
created: 2026-05-15
updated: 2026-05-15
topic: LLM 多智能体综述中文精读（第 6 篇-00）论文精华总览
difficulty: intermediate
prerequisites:
  - [[raw/paper/multi-agent/00-reading-manual]]
  - [[raw/lessons/Multi-Agent-Survey/01-progress-and-challenges/00-progress-and-challenges-essence]]
sources:
  - https://aclanthology.org/2025.emnlp-main.1403/
  - "DOI: 10.18653/v1/2025.emnlp-main.1403"
status: completed
series:
  name: "Multi-Agent Survey｜Creativity in LLM-based MAS（2025）"
  part: 0
paper_meta:
  paper_id: "multi-agent-survey-06"
  title: "Creativity in LLM-based Multi-Agent Systems: A Survey"
  authors: "Yi-Cheng Lin, Kang-Chieh Chen, Zhe-Yan Li, Tzu-Heng Wu, Tzu-Hsuan Wu, Kuan-Yu Chen, Hung-yi Lee, Yun-Nung Chen"
  year: "2025"
  venue: "EMNLP 2025"
  link: "https://aclanthology.org/2025.emnlp-main.1403/"
  section: "essence-summary"
---

# LLM 多智能体综述中文精读（第 6 篇-00）：论文精华总览

> 这篇论文的最大价值，是第一次把"创造力"单独抽出来作为 LLM 多智能体系统的核心分析维度。之前的综述都在讲协作、讲架构、讲应用，但没有人系统回答：**多个 LLM agent 一起工作，到底能不能产生比单 agent 更有创造力的输出？如果能，是靠哪些机制？**

## 这篇论文在整条阅读主线里的位置

前几篇综述各司其职：
- 第 1 篇搭骨架，讲 LLM-MAS 的基本构成与挑战；
- 第 2 篇画应用地图，展示 MAS 在各个领域的扩张；
- 第 3 篇聚焦协作机制，把 actor / type / structure / strategy / coordination 五个维度讲透；
- 第 4 篇探讨 agent 间的"超越自说自话"的高级交互；
- 第 5 篇关注 agent 和谐协作的系统设计；
- **第 6 篇则打开了一个全新视角：创造力**。

换句话说，这篇论文最适合你在已经理解"多智能体怎么协作"之后，进一步追问：

1. 多 agent 系统能不能比单 agent 更有创造力？
2. 创造力在 MAS 中通过哪些技术机制实现？
3. 不同类型、不同粒度的 agent persona 如何影响创意输出？
4. 创造力的评估到底怎么做——能不能既客观又可解释？

## 论文最核心的四维框架

作者从四个维度构建了创造力 MAS 的分析框架：

| 维度 | 它回答的问题 | 核心内容 |
|------|------------|---------|
| Workflow & Proactivity | 多 agent 的创作流程怎么组织？ | 三阶段流程（Planning/Process/Decision Making）+ 主动性谱系（reactive → proactive） |
| Techniques | 创造力通过什么技术机制产生？ | Divergent Exploration / Iterative Refinement / Collaborative Synthesis |
| Persona & Profile | agent 的"人设"怎么影响创意？ | 三层粒度（Coarse/Medium-Coarse/Fine）+ 三种生成方式 |
| Evaluation | 怎么衡量创造力？ | 客观指标（Distinct-n, Self-BLEU, FID 等）+ 主观评估（TTCT 改编）+ 交互评估 |

> [!tip] 理解提示（非原文）
> 前几篇综述很少触及"agent 的人格粒度"和"创造力评估指标"这些维度。这篇论文的独特性在于：它把创造力从"应用场景"升级为"系统能力维度"。

## 如果只记住 8 个精华判断

### 1. 这是第一篇专门聚焦"创造力"的 LLM-MAS 综述
之前的综述都在关注协作架构、通信协议、任务分工。但创造力——作为多智能体系统区别于单体的一个关键增益——从未被系统性地综述过。这篇论文填补了这个空白。

### 2. 创造力不是"多个 agent 各想各的"，而是可设计的机制
作者将创造力技术归纳为三类，每一类都对应一种可工程化的创意生产模式：
- **发散探索（Divergent Exploration）**：让 agent 朝不同方向生成候选，扩大创意空间；
- **迭代精炼（Iterative Refinement）**：通过多轮反馈逐步提升创意质量；
- **协作合成（Collaborative Synthesis）**：将不同 agent 的产出整合为统一的创意作品。

### 3. agent 的主动性（proactivity）是一个被低估的维度
作者区分了从 reactive（被动响应）到 proactive（主动发起）的谱系。这个维度直接决定了多 agent 系统是"只会回答问题的工具"还是"能主动提出创意的伙伴"。

### 4. Persona 设计有三层粒度，越细越"像人"但成本越高
| 粒度 | 描述 | 典型方式 |
|------|------|---------|
| Coarse | 大类角色（如"诗人""程序员"） | 一个 prompt 定义 |
| Medium-Coarse | 带背景故事的角色 | 少量属性描述 |
| Fine | 高度个性化的角色 | 从数据中提取或模型生成 |

### 5. 创造力评估是多维的，不能只看一个指标
作者系统整理了客观指标（Distinct-n、Entropy-n、Self-BLEU、SBERT、FID、TIE）和主观指标（改编自 TTCT 的 Fluency / Flexibility / Originality / Elaboration），还有交互层面的评估。这种多维评估框架是对该领域方法论的重要贡献。

### 6. 发散探索 + 迭代精炼 + 协作合成的组合最有效
单独使用任何一种技术都有局限：发散探索可能生成大量低质量内容；迭代精炼可能过早收敛；协作合成可能丢失多样性。最佳实践是把三者组合成流水线。

### 7. 创造力 MAS 面临独特的伦理和公平挑战
与通用 MAS 不同，创造力 MAS 会面临：
- 创意归属（谁才是"作者"？）
- 文化偏见（训练数据的文化不平衡导致创意倾向西方化）
- 创意冲突（agent 之间的审美分歧如何解决）

### 8. 统一的评估框架是该领域最迫切的需求
作者明确指出，目前创造力 MAS 最大的障碍之一是缺乏统一的评估基准。不同工作用不同指标、不同任务、不同评判标准，导致很难横向比较。

## 一张研究地图：从"创造力 MAS"往外能长出哪些方向

| 研究方向 | 这篇论文给出的切口 |
|---------|-------------------|
| 创意生成技术 | Divergent Exploration + Iterative Refinement + Collaborative Synthesis 的组合设计 |
| Persona 工程 | 三层粒度的 persona 设计 + 三种生成方式的最优组合 |
| 创意评估 | 客观指标 + 主观评估 + 交互评估的三维体系 |
| 多模态创意 | 从文本扩展到图像、音乐、视频等多模态创意生成 |
| 主动性设计 | reactive 到 proactive 的谱系中寻找最佳平衡点 |
| 创意归因 | 多 agent 协作产出中的知识产权与作者归属 |
| 文化公平性 | 非西方文化背景下的创造力 MAS 表现 |
| 人机共创 | 人类与 MAS 协作创作的工作流设计 |

## 这篇论文与前几篇的互补关系

| 论文 | 主问题 | 你读完后的收获 |
|------|--------|--------------|
| 第 1 篇 | LLM-MAS 是什么 | 搭骨架 |
| 第 2 篇 | LLM-MAS 能做什么 | 看应用地图 |
| 第 3 篇 | LLM-MAS 怎么协作 | 学机制语言 |
| 第 4 篇 | agent 间的高级交互 | 理解深度交互 |
| 第 5 篇 | agent 和谐协作 | 掌握系统设计 |
| 第 6 篇 | LLM-MAS 怎么创造 | 建立创造力评估框架 |

> [!note] 译者注（非原文）
> 如果你未来想做"创意 AI""多 agent 创意协作""创意质量自动评估""人格驱动的创意生成"等研究，第 6 篇是最直接的理论基础，因为它已经把创造力在 MAS 中的位置形式化了。

## 这篇论文最值得抄走的分析框架

当你以后读一篇创造力相关的 multi-agent 方法论文时，可以强制自己回答四个问题：

1. **Workflow**：创作流程是几阶段？agent 的主动性在什么水平？
2. **Technique**：用了哪种创意技术（发散/精炼/合成）？是否组合使用？
3. **Persona**：agent 的人格设计在什么粒度？是怎么生成的？
4. **Evaluation**：用了哪些指标？客观和主观评估是如何结合的？

如果这四个问题答不清，你其实还没有真正读懂这篇创造力 MAS 论文。

## 本篇分篇导航

| part | 文件 | 覆盖内容 |
|------|------|---------|
| 00 | [[raw/lessons/Multi-Agent-Survey/06-creativity/00-creativity-mas-essence\|00-essence]] | 论文精华、阅读地图、研究价值 |
| 01 | [[raw/lessons/Multi-Agent-Survey/06-creativity/01-creativity-mas-introduction-and-workflow\|01-introduction-workflow]] | Section 1（Introduction）+ Section 2（Workflow and Proactivity） |
| 02 | [[raw/lessons/Multi-Agent-Survey/06-creativity/02-creativity-mas-techniques\|02-techniques]] | Section 3（MAS Techniques for Creativity） |
| 03 | [[raw/lessons/Multi-Agent-Survey/06-creativity/03-creativity-mas-persona\|03-persona]] | Section 4（Persona and Agent Profile） |
| 04 | [[raw/lessons/Multi-Agent-Survey/06-creativity/04-creativity-mas-evaluation\|04-evaluation]] | Section 5（Evaluation） |
| 05 | [[raw/lessons/Multi-Agent-Survey/06-creativity/05-creativity-mas-challenges-and-conclusion\|05-challenges-conclusion]] | Section 6（Challenges）+ Section 7（Conclusion）+ Limitations |

## 本篇小结

> [!note] 译者注（非原文）
> 这篇"精华总览"最重要的作用，是让你先记住一句话：
> **多智能体系统的创造力，不是 agent 多了就自然有的，而是需要精心设计的 workflow、technique、persona 和 evaluation 共同作用的结果。**
>
> 只要你抓住"三阶段流程 + 三种创意技术 + 三层 persona 粒度 + 三维评估体系"这个框架，后续读任何创造力 MAS 论文都会更加游刃有余。

## 相关页面

- [[raw/paper/multi-agent/00-reading-manual]]
- [[raw/lessons/Multi-Agent-Survey/01-progress-and-challenges/00-progress-and-challenges-essence]]
- [[raw/lessons/Multi-Agent-Survey/02-recent-advances/10-recent-advances-and-new-frontiers-overview]]
- [[raw/lessons/Multi-Agent-Survey/03-collaboration-mechanisms/00-collaboration-mechanisms-essence]]

## 下一步学习

- 上一篇：[[raw/lessons/Multi-Agent-Survey/05-llms-working-in-harmony/05-working-in-harmony-conclusion]]
- 下一篇：[[raw/lessons/Multi-Agent-Survey/06-creativity/01-creativity-mas-introduction-and-workflow|01-introduction-workflow]]
