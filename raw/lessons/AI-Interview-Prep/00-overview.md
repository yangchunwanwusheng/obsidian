---
type: lesson
tags: [面试, AI, LLM, Agent, RAG, Memory, Python]
created: 2026-05-19
updated: 2026-05-20
difficulty: intermediate
prerequisites: [Python基础, 机器学习基础, 深度学习入门]
topic: AI工程师面试准备
status: in-progress
series: {name: "AI Interview Prep", part: 0}
---

# AI 工程师面试全能准备

> 系统化掌握 LLM 架构、AI Agent、RAG、Memory、Python 五大核心领域，面向真实面试场景深度备战。

## 📋 面试知识图谱

```
                    ┌─────────────────────┐
                    │   AI 工程师面试      │
                    └──────────┬──────────┘
           ┌───────────┬──────┴──────┬───────────┐
           ▼           ▼             ▼           ▼
    ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐
    │  LLM架构  │ │ AI Agent │ │   RAG    │ │  Memory  │
    └─────┬────┘ └────┬─────┘ └────┬─────┘ └────┬─────┘
          │           │            │             │
    ┌─────┴────┐ ┌────┴─────┐ ┌───┴────┐ ┌────┴─────┐
    │Transformer│ │ReAct/Plan│ │Embedding│ │工作记忆   │
    │Attention  │ │Tool Use  │ │向量数据库│ │长期记忆   │
    │训练流程   │ │Multi-Agent│ │检索策略  │ │上下文窗口 │
    │推理优化   │ │框架对比   │ │高级RAG  │ │状态追踪   │
    └──────────┘ └──────────┘ └────────┘ └──────────┘
           │           │            │             │
           └───────────┴──────┬─────┴─────────────┘
                              ▼
                    ┌──────────────────┐
                    │  Python 工程能力  │
                    │ 并发/内存/设计模式│
                    └──────────────────┘
```

## 🎯 高频面试题分布

| 领域 | 占比 | 难度分布 | 典型题型 |
|------|------|---------|---------|
| LLM 架构 | 30% | 中→高 | 原理阐述 + 架构设计 |
| AI Agent | 25% | 中→高 | 系统设计 + 场景题 |
| RAG | 20% | 中 | 方案设计 + 痛点分析 |
| Memory | 10% | 中 | 设计思路 + 技术选型 |
| Python | 15% | 中 | 代码实现 + 性能分析 |

## 📚 学习路线

```
第1站：LLM 架构（基石）
  ├─ Transformer 原理（必考！）
  ├─ 注意力机制变体（MHA/GQA/MQA）
  ├─ 训练三阶段（预训练→SFT→RLHF）
  ├─ 推理优化（KV Cache/量化/蒸馏）
  └─ 主流模型架构对比（GPT/Llama/GLM）
       │
       ▼
第2站：RAG（最实用）
  ├─ Naive RAG → Advanced RAG → Modular RAG
  ├─ Embedding 模型选型
  ├─ 向量数据库对比（Milvus/Chroma/Pinecone）
  ├─ Chunk 策略与检索优化
  └─ RAG vs Fine-tuning 场景选择
       │
       ▼
第3站：AI Agent（最热门）
  ├─ Agent = LLM + Planning + Tool Use + Memory
  ├─ ReAct / Plan-and-Execute / 反思模式
  ├─ Function Calling 机制
  ├─ Multi-Agent 协作框架
  └─ LangChain / LangGraph / AutoGen 对比
       │
       ▼
第4站：Memory（深度加分项）
  ├─ 短期记忆（上下文窗口管理）
  ├─ 长期记忆（向量存储 + 摘要）
  ├─ 对话状态追踪
  └─ 记忆压缩与遗忘策略
       │
       ▼
第5站：Python 工程能力（基本功）
  ├─ 高级特性（装饰器/元类/描述符）
  ├─ 并发编程（asyncio/多线程/多进程）
  ├─ 内存管理（GC/引用计数/内存泄漏排查）
  ├─ 常用设计模式
  └─ 性能优化（profiling/加速技巧）
```

## 🔥 面试应对策略

### 架构类问题（答法模板）

```
1. 定义先行："XX 是一种 ... 的技术/架构"
2. 核心原理："它的核心思想是 ..."
3. 技术细节（展示深度）：深入1-2个关键技术点
4. 实际应用："在项目中，我们用它来 ..."
5. 优缺点分析："优点是 ..., 局限在于 ..."
6. 对比扩展："与之类似的还有 XX，区别在于 ..."
```

### 系统设计类问题（答法模板）

```
1. 需求澄清："确认一下，场景是 ..."
2. 规模估算："假设日活 XX，QPS 约 XX"
3. 高层架构："整体分为这几层 ..."
4. 关键组件："核心组件包括 ..."
5. 技术选型："选择 XX 而不是 YY，因为 ..."
6. 瓶颈与优化："可能的瓶颈在 ..., 可以通过 ... 优化"
```

## 📁 系列笔记索引

| 编号 | 笔记 | 核心内容 | 建议学习时间 |
|------|------|---------|------------|
| 00 | [[raw/lessons/AI-Interview-Prep/00-overview]] | 总览与路线图（本页） | 15min |
| 01 | [[raw/lessons/AI-Interview-Prep/01-llm-architecture]] | LLM 架构深度 | 2-3h |
| 02 | [[raw/lessons/AI-Interview-Prep/02-ai-agent]] | AI Agent 系统 | 2-3h |
| 03 | [[raw/lessons/AI-Interview-Prep/03-rag]] | RAG 检索增强生成 | 2h |
| 04 | [[raw/lessons/AI-Interview-Prep/04-memory]] | AI Memory 记忆系统 | 1.5h |
| 05 | [[raw/lessons/AI-Interview-Prep/05-python-mastery]] | Python 精通 | 2h |
| 06 | [[raw/lessons/AI-Interview-Prep/06-hot-topics]] | 业界热点系统深度分析（概览） | 15min |
| 06a | [[raw/lessons/AI-Interview-Prep/06a-hot-agent-systems]] | Agent 热点系统（Claude Code/Cursor/Manus/OpenHands/MCP） | 2h |
| 06b | [[raw/lessons/AI-Interview-Prep/06b-hot-memory-systems]] | Memory 热点系统（Claude Code记忆/Mem0/ChatGPT Memory/Cursor上下文） | 2h |
| 06c | [[raw/lessons/AI-Interview-Prep/06c-hot-llm-systems]] | LLM 热点架构（DeepSeek-V3 MLA+MoE+MTP/SGLang/推理模型） | 2h |

## 💡 使用建议

1. **第一遍**：快速浏览所有笔记，建立知识框架
2. **第二遍**：精读每篇笔记，理解每个技术细节
3. **第三遍**：对着"面试场景题"口头练习回答
4. **实战**：用笔记中的代码片段在本地跑一遍，加深印象

> [!tip] 面试小贴士
> 面试官往往从一个基础问题开始，逐步深入。准备时要注意每个知识点的**层次感**——先能讲清楚是什么，再能说清楚为什么，最后能讨论怎么做得更好。

## 下一步学习

→ 开始第1站：[[raw/lessons/AI-Interview-Prep/01-llm-architecture|LLM 架构深度]]

---

## 🔥 业界热点系统（面试加分利器）

> 掌握业界最新系统架构，是证明技术视野的最佳方式。面试中提到 Claude Code / DeepSeek / SGLang 等热点系统，面试官会认为你关注最新动态、有实际使用经验、有工程思维。

```
第6站：业界热点系统深度分析
  ├─ Agent 热点：Claude Code 完整架构 / Cursor / Windsurf / Manus / OpenHands / MCP 协议
  ├─ Memory 热点：Claude Code 记忆系统 / Mem0 三层架构 / ChatGPT Memory / Cursor 上下文管理
  └─ LLM 热点：DeepSeek-V3 (MLA+MoE+MTP) / SGLang RadixAttention / 推理模型 (o1/R1)
```

→ 进入第6站：[[raw/lessons/AI-Interview-Prep/06-hot-topics|业界热点系统深度分析]]
