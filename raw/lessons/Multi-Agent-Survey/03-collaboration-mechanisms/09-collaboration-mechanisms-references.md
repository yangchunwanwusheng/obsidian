---
type: lesson
tags: [多智能体, multi-agent, survey, 协作机制, references, 学习笔记]
created: 2026-05-08
updated: 2026-05-08
topic: LLM 多智能体综述中文精读（第 3 篇-09）参考文献索引
difficulty: intermediate
prerequisites:
  - [[raw/lessons/Multi-Agent-Survey/03-collaboration-mechanisms/08-collaboration-mechanisms-open-problems-and-conclusion]]
sources:
  - [[raw/paper/multi-agent/03-multi-agent-collaboration-mechanisms-a-survey-of-llms-2025.pdf]]
  - https://arxiv.org/abs/2501.06322
status: completed
series:
  name: "Multi-Agent Survey｜Collaboration Mechanisms（2025）"
  part: 9
paper_meta:
  paper_id: "multi-agent-survey-03"
  title: "Multi-Agent Collaboration Mechanisms: A Survey of LLMs"
  authors: "Khanh-Tung Tran, Dung Dao, Minh-Duong Nguyen, Quoc-Viet Pham, Barry O’Sullivan, Hoang D. Nguyen"
  year: "2025"
  link: "https://arxiv.org/abs/2501.06322"
  section: "References"
---

# LLM 多智能体综述中文精读（第 3 篇-09）：参考文献索引

> 本页不逐条翻译全部参考文献，而是提炼出对理解“协作机制”最关键的一批文献，作为后续延展阅读入口。

## 本章导读

> [!note] 译者注（非原文）
> 第 3 篇论文的 references 很长，如果全部平铺列出，检索价值反而下降。
> 所以这里改成“主题索引式保留”：按协作机制相关主题整理关键文献，方便后续沿线追读。

## 协作机制核心文献

### 1. 多智能体协作总览 / 奠基综述
- [[raw/paper/multi-agent/01-large-language-model-based-multi-agents-survey-of-progress-and-challenges-2024.pdf]]
- [[raw/paper/multi-agent/02-a-survey-on-llm-based-multi-agent-system-recent-advances-and-new-frontiers-in-application-2024.pdf]]
- Guo et al. (2024) — *Large Language Model Based Multi-agents: A Survey of Progress and Challenges*
- Han et al. (2024) — *LLM Multi-Agent Systems: Challenges and Open Problems*
- Lu et al. (2024) — *Merge, Ensemble, and Cooperate! A Survey on Collaborative Strategies in the Era of Large Language Models*

### 2. 合作 / 角色协作
- Chen et al. (2024c) — *AgentVerse: Facilitating Multi-Agent Collaboration and Exploring Emergent Behaviors*
- Hong et al. (2024) — *MetaGPT: Meta Programming for A Multi-Agent Collaborative Framework*
- Li et al. (2023a) — *CAMEL: Communicative Agents for "Mind" Exploration of Large Language Model Society*
- Wu et al. (2024b) — *AutoGen: Enabling Next-Gen LLM Applications via Multi-Agent Conversation*
- Islam et al. (2024) — *MapCoder: Multi-Agent Code Generation for Competitive Problem Solving*

### 3. 竞争 / 辩论 / 对抗
- Du et al. (2023) — *Improving Factuality and Reasoning in Language Models through Multiagent Debate*
- Chen et al. (2024b) — *LLMArena: Assessing Capabilities of Large Language Models in Dynamic Multi-Agent Environments*
- Liang et al. (2024) — *Encouraging Divergent Thinking in Large Language Models through Multi-Agent Debate*
- Zhao et al. (2024a) — *CompeteAI: Understanding the Competition Dynamics of Large Language Model-based Agents*
- He et al. (2023) — *LEGO: A Multi-agent Collaborative Framework with Role-playing and Iterative Feedback for Causality Explanation Generation*

### 4. 协作策略 / 社会心理 / 心智理论
- Li et al. (2023b) — *Theory of Mind for Multi-Agent Collaboration via Large Language Models*
- Zhang et al. (2024c) — *Exploring Collaboration Mechanisms for LLM Agents: A Social Psychology View*
- Cao et al. (2024) — *Enhancing Human-AI Collaboration Through Logic-Guided Reasoning*
- Xu et al. (2023c) — *Towards reasoning in large language models via multi-agent peer review collaboration*
- Chen et al. (2023) — *Multi-Agent Consensus Seeking via Large Language Models*

### 5. 通信结构 / 组织拓扑 / 动态网络
- Liu et al. (2024d) — *A Dynamic LLM-Powered Agent Network for Task-Oriented Agent Collaboration*
- Zhuge et al. (2024b) — *GPTSwarm: Language Agents as Optimizable Graphs*
- Yin et al. (2023) — *Exchange-of-Thought: Enhancing Large Language Model Capabilities through Cross-Model Communication*
- Ishibashi and Nishimura (2024) — *Self-Organized Agents: A LLM Multi-Agent Framework toward Ultra Large-Scale Code Generation and Optimization*
- Dong et al. (2024a) — *VillagerAgent: A Graph-Based Multi-Agent Framework for Coordinating Complex Task Dependencies in Minecraft*

### 6. 应用与评测
- Zhuge et al. (2024a) — *Agent-as-a-Judge: Evaluate Agents with Agents*
- Chan et al. (2024) — *ChatEval: Towards Better LLM-based Evaluators through Multi-Agent Debate*
- Mitra et al. (2024) — *AgentInstruct / Orca-AgentInstruct*
- Park et al. (2023) — *Generative Agents: Interactive Simulacra of Human Behavior*
- Li et al. (2024a) — *CulturePark: Boosting Cross-cultural Understanding in Large Language Models*

### 7. 安全与风险
- Shayegani et al. (2023) — *Survey of vulnerabilities in large language models revealed by adversarial attacks*
- Zhang et al. (2024e) — *PsySafe: A Comprehensive Framework for Psychological-based Attack, Defense, and Evaluation of Multi-agent System Safety*
- Meinke et al. (2024) — *Frontier Models are Capable of In-context Scheming*
- Deshpande et al. (2023) — *Anthropomorphization of AI: Opportunities and Risks*

## 如何继续往下读

### 如果你关心研究切口
优先顺序建议：
1. [[raw/lessons/Multi-Agent-Survey/03-collaboration-mechanisms/03-collaboration-mechanisms-types]]
2. [[raw/lessons/Multi-Agent-Survey/03-collaboration-mechanisms/05-collaboration-mechanisms-structures]]
3. [[raw/lessons/Multi-Agent-Survey/03-collaboration-mechanisms/08-collaboration-mechanisms-open-problems-and-conclusion]]
4. 然后追上面索引里的 debate / ToM / dynamic topology / benchmark 代表论文。

### 如果你关心工程落地
优先顺序建议：
1. MetaGPT
2. AutoGen
3. CAMEL
4. Magentic-One
5. Swarm / Bee Agent Framework / LangChain Agents

### 如果你关心社会模拟与研究方法
优先顺序建议：
1. Generative Agents
2. CulturePark
3. Agent-as-a-Judge
4. 社会心理 / ToM / consensus 相关论文

## 本章小结

> [!note] 译者注（非原文）
> 第 3 篇真正值得追读的 references，不是“所有文献平均读”，而是按你的研究问题选线：
> - 做拓扑与群智 → 读 dynamic topology / consensus / swarm；
> - 做工程框架 → 读 orchestration / role-based systems；
> - 做评测 → 读 judge / benchmark / arena；
> - 做安全 → 读 adversarial / psychosafe / deception。

## 相关页面

- [[raw/lessons/Multi-Agent-Survey/03-collaboration-mechanisms/00-collaboration-mechanisms-essence]]
- [[raw/lessons/Multi-Agent-Survey/03-collaboration-mechanisms/08-collaboration-mechanisms-open-problems-and-conclusion]]
- [[raw/paper/multi-agent/03-multi-agent-collaboration-mechanisms-a-survey-of-llms-2025.pdf]]
- [[raw/paper/multi-agent/00-reading-manual]]

## 下一步学习

- 上一篇：[[raw/lessons/Multi-Agent-Survey/03-collaboration-mechanisms/08-collaboration-mechanisms-open-problems-and-conclusion|08-open-problems-and-conclusion]]
- 返回总览：[[raw/lessons/Multi-Agent-Survey/03-collaboration-mechanisms/00-collaboration-mechanisms-essence|00-essence]]
- 下一步建议：从 references 中挑 3-5 篇 debate / topology / benchmark 方法论文继续精读
