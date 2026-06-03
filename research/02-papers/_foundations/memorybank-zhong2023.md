---
type: paper-note
title: "MemoryBank: Enhancing Large Language Models with Long-Term Memory"
authors: [Wanjun Zhong, Lianghao Xia, Jiapeng Wang, Yusong Qiu, Chi Zhang, Chao Zhang, Lei Huang, Qun Liu, Min Yang]
venue: arXiv 2023
year: 2023
paper_link: https://arxiv.org/abs/2305.10250
code_link: https://github.com/zhongwanjun/MemoryBank-SiliconFriend
status: read
tags: [agent-memory, 长期记忆, 遗忘曲线, 对话Agent]
created: 2026-04-18
updated: 2026-04-18
---

## 核心贡献

> MemoryBank 提出了一种基于**艾宾浩斯遗忘曲线（Ebbinghaus Forgetting Curve）**的长期记忆机制，使 LLM 能够像人类一样选择性遗忘和强化记忆，从而在长期交互中持续进化和个性化。

## 研究背景与动机

- **解决的问题**：LLM 在多轮对话中无法保留历史信息，每次对话都是"重新开始"
- **现有方法的不足**：
  - 简单的上下文窗口（Context Window）无法存储长期信息
  - 简单的向量检索（Vector Retrieval）没有遗忘机制，记忆无限膨胀
  - 缺乏模拟人类记忆的选择性保留和遗忘能力
- **为什么重要**：个性化 AI 伴侣、心理咨询助手等场景需要 Agent 能记住用户偏好和历史交互

## 方法概述

### 核心思想

借鉴认知心理学中的人类记忆机制，特别是**艾宾浩斯遗忘曲线**，设计一种可以**遗忘不重要记忆、强化重要记忆**的 Agent 记忆系统。

### 关键技术

#### 1. 记忆存储（Memory Storage）

```
记忆条目结构：
{
  content: "自然语言描述的记忆内容",
  timestamp: "创建时间",
  access_count: "访问次数",
  forgetting_strength: "遗忘强度值（基于艾宾浩斯曲线计算）"
}
```

#### 2. 艾宾浩斯遗忘曲线（Ebbinghaus Forgetting Curve）

**核心公式**：

$$R = e^{-t/S}$$

其中：
- $R$（Retention，保留率）：记忆的保留程度，0~1 之间
- $t$（Time，时间）：自上次访问以来经过的时间
- $S$（Stability，稳定性）：记忆的稳定程度，与记忆的重要性相关
- $e$：自然常数（约 2.718）

**通俗解释**：
- 刚记住的信息，保留率很高（$R$ 接近 1）
- 随时间推移，如果不复习，保留率指数衰减
- 越重要的信息（$S$ 越大），衰减越慢
- **每次被重新访问/检索到的记忆，其稳定性 $S$ 会增加**（复习效应）

**遗忘决策**：
```
计算遗忘强度值
  ↓
与随机阈值比较
  ↓
遗忘强度值 > 阈值 → 删除该记忆（遗忘）
遗忘强度值 ≤ 阈值 → 保留该记忆（强化）
```

#### 3. 记忆检索（Memory Retrieval）

- 当用户发起查询时，遍历历史对话记录
- 计算当前查询与历史记忆的语义相似度（Semantic Similarity）
- 使用 Embedding 模型将记忆和查询向量化
- 检索最相关的记忆并注入到 LLM 的上下文中

#### 4. 记忆更新（Memory Update）

- **新记忆写入**：新的交互信息被提取并写入记忆库
- **记忆强化**：被检索到的记忆，其稳定性 $S$ 增加
- **记忆遗忘**：根据遗忘曲线计算，不重要的记忆被自动清理
- **记忆摘要**：定期对历史记忆进行摘要压缩

### 方法流程

```
用户输入
  ↓
1. 记忆检索：从记忆库中检索与当前查询相关的历史记忆
  ↓
2. 上下文构建：将检索到的记忆 + 当前查询组装为 LLM 输入
  ↓
3. LLM 生成：LLM 基于增强的上下文生成回复
  ↓
4. 记忆更新：从交互中提取新记忆，更新记忆库
  ↓
5. 遗忘决策：根据艾宾浩斯曲线计算遗忘，清理不重要记忆
  ↓
输出回复
```

## 实验设置

| 项目 | 内容 |
|------|------|
| 数据集 | MultiSession Chat（多会话对话）|
| 基线方法 | 无记忆 LLM、简单检索增强 |
| 评估指标 | 对话一致性、个性化程度、用户满意度 |

### 主要结果

- MemoryBank 在长期对话一致性上显著优于无记忆基线
- 艾宾浩斯遗忘机制使记忆库大小保持在合理范围
- 支持多种 LLM 后端（ChatGPT、ChatGLM 等），兼容性强

## 优点

- ✅ **模拟人类遗忘**：首次将艾宾浩斯遗忘曲线引入 Agent 记忆管理，使记忆行为更自然
- ✅ **记忆自动管理**：不需要手动清理记忆，系统自动决定保留和遗忘
- ✅ **兼容性强**：适用于封闭源（Closed-source）和开放源（Open-source）LLM
- ✅ **实现简洁**：核心算法简单（遗忘曲线 + 向量检索），易于复现

## 局限性 / 未来工作

- ❌ **遗忘曲线固定**：使用预设的艾宾浩斯曲线参数，无法根据任务类型自适应调整
- ❌ **线性检索效率低**：遍历所有历史记录计算相似度，记忆量大时效率下降
- ❌ **缺乏反思机制**：只有简单的记忆存储和检索，没有对记忆的高层次抽象
- ❌ **单一记忆类型**：只考虑了情景记忆（具体交互），缺乏语义记忆（知识）和程序性记忆（技能）
- 💡 **改进方向**：
  - 自适应遗忘曲线参数（根据用户反馈调整遗忘速率）
  - 分层记忆结构（原始记忆 → 摘要 → 反思洞察）
  - 结合知识图谱增强语义记忆

## 与我的研究关联

- MemoryBank 的遗忘机制是 Agent Memory 设计的重要参考
- 其**固定遗忘曲线**的局限性可以作为我研究的切入点——设计自适应记忆管理策略
- 与我的"想法 4：记忆重要性自适应评估"直接相关
- 可以考虑将遗忘曲线与图结构记忆结合

## 待思考的问题

> 1. 艾宾浩斯遗忘曲线是否真的适合 AI Agent？人类的遗忘是被动的能力限制，Agent 的遗忘应该是主动的策略选择
> 2. 如何量化"记忆的重要性"？当前的 LLM 打分方法是否可靠？
> 3. 如果记忆被错误地遗忘，如何恢复？

## 相关论文

- [[research/02-papers/_foundations/generative-agents-park2023|Generative Agents (Park 2023)]] — 同样涉及记忆重要性评估和检索
- [[research/02-papers/_foundations/memgpt-packer2023|MemGPT (Packer 2023)]] — 不同的记忆管理策略（OS 式 vs 遗忘曲线式）
- [RET-LLM](https://arxiv.org/abs/2305.14322) — 通用读写记忆机制
- [[research/02-papers/_surveys/agent-memory-survey-zhang2024|A Survey on Memory Mechanism of LLM Agents]] — 综述中对 MemoryBank 的分析
