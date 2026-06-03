---
type: lesson
tags: [面试, 热点, 业界分析, Claude-Code, DeepSeek, Mem0, Cursor, Manus]
created: 2026-05-20
updated: 2026-05-20
difficulty: advanced
prerequisites: [LLM架构, AI-Agent, RAG, Memory系统]
topic: 业界热点与面试知识连接
status: in-progress
series: {name: "AI Interview Prep", part: 6}
---

# 业界热点系统深度分析——面试加分利器

> 面试不仅考理论，更考"你关注什么"。掌握业界最新系统架构，是证明技术视野的最佳方式。

## 学习目标

- [ ] 能深入分析 Claude Code / Cursor / Manus 等 Agent 系统的架构设计
- [ ] 能对比 Claude Code 记忆 / Mem0 / ChatGPT Memory 的记忆系统差异
- [ ] 理解 DeepSeek-V3 的 MLA + MoE + MTP 三大创新及数学原理
- [ ] 掌握 SGLang 的 RadixAttention 和推理框架选型逻辑
- [ ] 能在面试中自信讨论"如果让你设计 X 系统"的开放题

## 为什么热点系统是面试加分项？

```
面试考察维度：

┌──────────────────────────────────────────────┐
│            AI 工程师面试考察金字塔             │
│                                              │
│                   ▲                          │
│                  /  \                         │
│                 / 热点 \    ← 你关注什么？     │
│                / 系统分析 \   技术视野广度      │
│               /────────── \                    │
│              /   系统设计   \  ← 你能设计什么？ │
│             /    能力       │   工程落地能力    │
│            /──────────────── \                  │
│           /    基础理论知识    \ ← 你知道什么？ │
│          /                     │  基础扎实度    │
│         /─────────────────────── \              │
└──────────────────────────────────────────────┘

面试官的思维链：
"你了解 DeepSeek-V3 的 MLA 吗？"
  → 了解 → "说说和 GQA 的区别" → 考察深度
  → 不了解 → "说说你知道的最新架构" → 考察视野
  → 都不了解 → 只能考基础理论
```

**关键信号**：面试中提到 Claude Code / DeepSeek / SGLang，面试官会认为你：
1. 关注业界最新动态（> 80% 候选人做不到）
2. 有实际使用经验（不只是纸上谈兵）
3. 有工程思维（能分析"为什么这样设计"）

## 热点系统知识地图

```
                        ┌──────────────────────┐
                        │    AI 面试热点系统    │
                        └──────────┬───────────┘
           ┌───────────────────────┼───────────────────────┐
           ▼                       ▼                       ▼
    ┌──────────────┐       ┌──────────────┐       ┌──────────────┐
    │  Agent 系统   │       │ Memory 系统   │       │  LLM 架构    │
    │  热点分析     │       │  热点分析     │       │  热点分析     │
    ├──────────────┤       ├──────────────┤       ├──────────────┤
    │ Claude Code  │       │ Claude Code  │       │ DeepSeek-V3  │
    │ Cursor       │       │ 记忆系统      │       │ MLA 注意力   │
    │ Windsurf     │       │ Mem0         │       │ MoE 架构     │
    │ Manus        │       │ ChatGPT      │       │ SGLang       │
    │ OpenHands    │       │ Cursor 上下文 │       │ 推理模型      │
    │ MCP 协议     │       │ 设计题       │       │ 训练效率      │
    └──────┬───────┘       └──────┬───────┘       └──────┬───────┘
           │                      │                      │
           ▼                      ▼                      ▼
    [[06a-hot-agent-       [[06b-hot-memory-       [[06c-hot-llm-
     systems]]              systems]]               systems]]
```

## 各系统面试考察频率

| 系统 | 考察频率 | 典型问题 | 连接的知识点 |
|------|---------|---------|------------|
| **Claude Code** | ⭐⭐⭐⭐⭐ | Agent 循环 / 记忆系统 / 工具设计 | ReAct, Function Calling, Memory |
| **DeepSeek-V3** | ⭐⭐⭐⭐ | MLA 原理 / MoE 设计 / 训练效率 | Attention 变体, MoE, 量化 |
| **Cursor** | ⭐⭐⭐⭐ | 上下文管理 / Agent 编辑策略 | RAG, Context Window |
| **Mem0** | ⭐⭐⭐ | 三层记忆架构 / 与自建对比 | Memory, 向量数据库 |
| **SGLang** | ⭐⭐⭐ | RadixAttention / 推理框架选型 | KV Cache, PagedAttention |
| **Manus** | ⭐⭐⭐ | 多 Agent 协作 / 任务分解 | Multi-Agent, Planning |
| **ChatGPT Memory** | ⭐⭐ | 记忆管理 / 自动提取 | Memory, 用户画像 |
| **OpenHands** | ⭐⭐ | 开源 Agent 架构 / 沙箱设计 | Agent 安全性 |

## 学习路线

```
推荐学习顺序（根据面试紧急度）：

Day 1: Claude Code 架构 + 记忆系统（两个文件都涉及 Claude Code）
        ↓
Day 2: DeepSeek-V3 架构 + SGLang
        ↓
Day 3: Cursor + Manus + OpenHands + Mem0

每学完一个系统，练习：
1. 用自己的话复述架构（3分钟电梯演讲）
2. 分析"为什么这样设计"（权衡分析）
3. 设计题：如果让你做一个类似的系统，你会怎么做？
```

## 与前面知识的连接

本系列笔记的核心知识点与热点系统的映射关系：

| 知识点（前面的笔记） | 业界系统（本篇） | 面试连接题 |
|---------------------|-----------------|-----------|
| ReAct 范式 | Claude Code Agent 循环 | "Claude Code 的 Agent 循环和 ReAct 有什么异同？" |
| Function Calling | Cursor 工具系统 | "Cursor 的工具调用和标准 FC 有什么区别？" |
| Multi-Agent 协作 | Manus 多 Agent 并行 | "Manus 的并行 Agent 和传统主从模式有什么区别？" |
| Memory 分类体系 | Claude Code 记忆 / Mem0 | "对比 Claude Code 和 Mem0 的记忆架构" |
| KV Cache 管理 | SGLang RadixAttention | "RadixAttention 解决了什么 KV Cache 问题？" |
| Attention 变体 | DeepSeek MLA | "MLA 和 GQA/MQA 的本质区别是什么？" |
| MoE 架构 | DeepSeek MoE | "为什么 DeepSeek 用 256 个细粒度专家？" |
| 量化 | DeepSeek FP8 训练 | "FP8 训练的精度损失如何控制？" |

## 深度扩展阅读

| 扩展笔记 | 核心内容 | 建议时间 |
|----------|---------|---------|
| [[raw/lessons/AI-Interview-Prep/06a-hot-agent-systems\|06a Agent 热点系统]] | Claude Code 完整架构分析（Agent循环/工具系统/多智能体/MCP协议）、Cursor vs Windsurf vs Claude Code 三方对比、Manus 多Agent架构、OpenHands 开源Agent SDK | 2h |
| [[raw/lessons/AI-Interview-Prep/06b-hot-memory-systems\|06b Memory 热点系统]] | Claude Code 三层 CLAUDE.md + MEMORY.md 记忆系统、Mem0 三层数据库架构、ChatGPT Memory 自动管理、Cursor 上下文管理、记忆系统设计题 | 2h |
| [[raw/lessons/AI-Interview-Prep/06c-hot-llm-systems\|06c LLM 热点架构]] | DeepSeek-V3 MLA 数学推导 + MoE 256专家 + MTP、SGLang RadixAttention + CFSM、推理模型（o1/R1）架构、训练效率创新 | 2h |

## 导航

← [[raw/lessons/AI-Interview-Prep/05-python-mastery|第5站：Python 精通]]
→ [[raw/lessons/AI-Interview-Prep/06a-hot-agent-systems|06a Agent 热点系统]]
→ [[raw/lessons/AI-Interview-Prep/06b-hot-memory-systems|06b Memory 热点系统]]
→ [[raw/lessons/AI-Interview-Prep/06c-hot-llm-systems|06c LLM 热点架构]]
