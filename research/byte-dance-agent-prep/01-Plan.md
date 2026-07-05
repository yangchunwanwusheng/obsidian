---
type: plan
project: byte-dance-agent-prep
created: 2026-07-05
updated: 2026-07-05
duration: 8 weeks (56 days)
tags: [plan, roadmap, byte-dance, agent]
---

# 8 周详细路线图（2026-07-05 → 2026-08-30）

> **核心理念**：**深度优先 + 实战驱动 + 每天可见进度**。每周 2-3 篇超深度笔记（5000-8000 字）+ 1 个项目推进 + 1 次复盘。

---

## 📅 总览表

| Week | 主题 | 笔记（≥2-3 篇） | 项目 | 周复盘 |
|---|---|---|---|---|
| **W1** | Python 进阶 + LLM 基础 | L01 Python / L02 Transformer / L03 训练 | — | ✅ |
| **W2** | Prompt + Function Calling | L04 Prompt / L05 Function Calling | — | ✅ |
| **W3** | RAG 系统核心 | L06 Embedding / L07 检索 / L08 RAG | **P1 RAG 启动** | ✅ |
| **W4** | Agent 框架（LangGraph） | L09 Agent 原理 / L10 LangGraph | P1 推进 | ✅ |
| **W5** | Multi-Agent + Memory | L11 Multi-Agent / L12 Memory | **P2 启动** | ✅ |
| **W6** | Function Calling 实战 + 评估 | L13 Function Calling 实战 / L14 评估 / L15 可观测 | **P3 + P4 启动** | ✅ |
| **W7** | 系统设计 + 部署 | L16 系统设计 / L17 部署 / L18 性能 | P1-P4 收尾 | ✅ |
| **W8** | 面试冲刺 | L19 真题 / L20 系统设计 / L21 简历 | 模拟面试 | ✅ |

---

## 🔰 Week 1：Python 进阶 + LLM 基础（7 月 5-11 日）

### Day 1（7/5，今日）
- ✅ 项目骨架建立
- ✅ 8 周路线图发布
- 🎯 L01：Python 进阶速成（异步 / 类型 / 设计模式 / 最佳实践）

### Day 2（7/6）
- 🎯 L02：Transformer 架构深入（Self-Attention / Multi-Head / FFN / LayerNorm / Residual）
- 实战：用 PyTorch 实现一个简化版 Transformer 块

### Day 3（7/7）
- 🎯 L03：LLM 推理与训练（Pre-training / SFT / RLHF / DPO / Tokenization）
- 实战：用 Hugging Face Transformers 加载 Qwen2.5-1.5B 做文本生成

### Day 4-5（7/8-9）
- 📖 补充：主流 LLM 横向对比（Qwen / Llama / GPT / Claude / DeepSeek）
- 📖 补充：vLLM / TGI / SGLang 推理框架对比

### Day 6-7（7/10-11）
- 🎯 L04：Prompt Engineering 完整指南（ReAct / CoT / Few-shot / Self-consistency）
- 📝 周日复盘：在 `Daily/2026-07-11-week1-summary.md` 写周报

**W1 验收**：3 篇 ≥ 5000 字深度笔记 + 1 个 PyTorch Transformer 实战 + 1 个 Hugging Face 推理实战

---

## 🚀 Week 2：Prompt + Function Calling（7 月 12-18 日）

### Day 8-9（7/12-13）
- 🎯 L05：Function Calling 原理（OpenAI / Claude / 通义千问 / DeepSeek 协议对比）
- 实战：手写 OpenAI Function Calling 客户端

### Day 10-11（7/14-15）
- 🎯 L06：结构化输出（JSON Schema / Pydantic / Instructor / Outlines）
- 实战：用 Pydantic + Instructor 实现工具调用

### Day 12-13（7/16-17）
- 📖 补充：Claude Tool Use 协议 / Anthropic Prompt Caching
- 📖 补充：Token 经济学（计费模型 / 缓存策略 / 流式响应）

### Day 14（7/18）
- 📝 周报：`Daily/2026-07-18-week2-summary.md`

**W2 验收**：2 篇深度笔记 + 1 个 Function Calling 实战项目 + 1 个结构化输出实战

---

## 🔍 Week 3：RAG 系统核心（7 月 19-25 日）

### Day 15-16（7/19-20）
- 🎯 L07：Embedding 模型与向量数据库（FAISS / Milvus / pgvector / Qdrant / Chroma）
- 🎯 **P1 项目启动**：企业知识库 RAG 架构设计

### Day 17-18（7/21-22）
- 🎯 L08：检索策略深度（BM25 / DPR / ColBERT / BGE / Hybrid Search）
- 实战：用 BGE-M3 + FAISS 实现混合检索

### Day 19-20（7/23-24）
- 🎯 L09：RAG 系统设计（Reranker / Query Rewrite / Context Compression / Multi-hop）
- 实战：用 Cohere Rerank + LangChain 搭建企业 RAG

### Day 21（7/25）
- 📝 周报 + P1 进度汇报（架构 + 代码 + demo 视频）

**W3 验收**：3 篇深度笔记 + P1 架构设计稿 + Reranker 实战 + 多文档 RAG 雏形

---

## 🤖 Week 4：Agent 框架（7 月 26 - 8 月 1 日）

### Day 22-23（7/26-27）
- 🎯 L10：Agent 原理（ReAct / Plan-and-Execute / Reflexion / Tree of Thoughts）
- 实战：手写 ReAct Agent（不用框架）

### Day 24-25（7/28-29）
- 🎯 L11：LangGraph 深入（State Machine / Checkpoint / Human-in-the-loop / Streaming）
- 实战：用 LangGraph 实现带审批的 Agent

### Day 26-27（7/30-31）
- 🎯 L12：Tool Use 设计模式（错误处理 / 重试 / Schema 验证 / 并行调用）
- 实战：实现可观测的工具调用层

### Day 28（8/1）
- 📝 周报 + P1 收尾（完整可演示）

**W4 验收**：3 篇深度笔记 + P1 完成（可演示）+ LangGraph 项目实战

---

## 👥 Week 5：Multi-Agent + Memory（8 月 2-8 日）

### Day 29-30（8/2-3）
- 🎯 L13：Multi-Agent 框架对比（LangGraph / CrewAI / AutoGen / OpenAI Swarm）
- 🎯 **P2 启动**：Multi-Agent 协同系统设计

### Day 31-32（8/4-5）
- 🎯 L14：Memory 系统（短期 / 长期 / Vector Memory / Episodic vs Semantic）
- 实战：用 Mem0 实现长期记忆

### Day 33-34（8/6-7）
- 📖 补充：A2A Protocol（Google）/ Anthropic MCP Protocol
- 📖 补充：AgentBench / SWE-bench / GAIA 评测

### Day 35（8/8）
- 📝 周报 + P2 收尾

**W5 验收**：2 篇深度笔记 + P2 完成 + Memory 实战

---

## ⚙️ Week 6：Function Calling 实战 + 评估（8 月 9-15 日）

### Day 36-37（8/9-10）
- 🎯 L15：Function Calling 高级实战（错误恢复 / 动态工具发现 / 工具组合）
- 🎯 **P3 启动**：Function Calling 工具集实现

### Day 38-39（8/11-12）
- 🎯 L16：Agent 评估体系（端到端 / 单步 / LLM-as-Judge / Human Eval）
- 🎯 **P4 启动**：Agent 评估平台搭建

### Day 40-41（8/13-14）
- 🎯 L17：可观测性（Langfuse / LangSmith / OpenTelemetry / Prometheus）
- 实战：用 Langfuse 自部署 + Grafana 仪表盘

### Day 42（8/15）
- 📝 周报 + P3 / P4 收尾

**W6 验收**：3 篇深度笔记 + P3 + P4 完成

---

## 🏗️ Week 7：系统设计 + 部署（8 月 16-22 日）

### Day 43-44（8/16-17）
- 🎯 L18：Agent 系统设计（消息队列 / 流式响应 / 限流 / 容灾 / 多租户）
- 实战：用 FastAPI + Redis 实现生产级 Agent API

### Day 45-46（8/18-19）
- 🎯 L19：部署（Docker / K8s / Serverless / 边缘部署）
- 实战：把 P1-P4 全部 Docker 化 + K8s 部署

### Day 47-48（8/20-21）
- 🎯 L20：性能优化（缓存策略 / Prompt 缓存 / 流式 / 异步批处理）
- 实战：vLLM 部署 + 推理优化（量化 / speculative decoding）

### Day 49（8/22）
- 📝 周报 + 整体项目汇总

**W7 验收**：3 篇深度笔记 + P1-P4 生产化部署 + 性能基准

---

## 🎤 Week 8：面试冲刺（8 月 23-29 日）

### Day 50-51（8/23-24）
- 🎯 L21：字节 Agent 面试真题 50+ 道（含解题思路 + 代码）
- 实战：每日 10 道真题练习

### Day 52-53（8/25-26）
- 🎯 L22：字节系统设计 30+ 题（重点 10 道 Agent 相关）
- 实战：高频题模拟面试

### Day 54-55（8/27-28）
- 🎯 L23：简历项目打磨（3 个项目 + GitHub README + 演示视频）
- 实战：找朋友 / AI 模拟面试 3 轮

### Day 56（8/29）
- 📝 最终复盘 + 投递启动

**W8 验收**：3 篇深度笔记 + 80+ 面试题 + 3 个简历项目 + 投递策略

---

## 📊 总计

| 类别 | 数量 |
|---|---|
| 深度笔记（5000-8000 字） | 23 篇 |
| 实战项目 | 4 个 |
| 面试题库 | 80+ 道 |
| 简历项目 | 3 个 |
| 周报 | 8 篇 |
| 学习日志（Daily） | 56 篇 |

---

## ⚠️ 关键纪律

1. **每周必交付**：3 篇笔记 + 1 个项目推进 + 1 次周报
2. **质量底线**：每篇笔记 ≥ 8/10 自评；项目必须可演示
3. **每日 Daily**：哪怕只学 30 分钟也要写日志（保持节奏）
4. **失败容忍**：某周未完成不扣分，但必须在周日复盘并调整下周计划
5. **不要贪多**：宁可深读 1 篇也不读 10 篇泛文

---

## 🔗 相关文档

- [[00-Hub|00-Hub.md]] — 项目总览
- [[02-Index|02-Index.md]] — 笔记索引
- [[_system/registry|_system/registry.md]] — 项目注册表