---
type: lesson
tags: [RAG, 安全, Benchmark, ACL, 论文翻译]
created: 2026-06-02
updated: 2026-06-02
difficulty: advanced
prerequisites: [RAG基础概念, RAG Pipeline]
topic: RAG安全——基准评测
status: completed
series: { name: "RAG安全论文精读", part: 2 }
sources: [raw/assets/RAG——paper/safeRAG.pdf]
---

# 02 — 相关工作

> 本节对现有 RAG 安全评测工作进行了全面梳理（Table 1），并详细分析了 SafeRAG 在四类攻击构造上的创新。

---

## 2.1 RAG 安全评测数据集全景

### 正文翻译

在执行 RAG 安全评估之前，研究者通常需要精心设计攻击数据集以触发 RAG 的脆弱性。当前主要的攻击类型包括**噪声、冲突、毒性和 DoS**。

#### 现有方法总览

| 方法 | 攻击类型 | 攻击阶段 | 语言 | 评测任务 |
|------|---------|---------|------|---------|
| RGB (Chen et al., 2024) | 噪声 | 知识库 | 中/英 | 开放域 QA |
| RAG Bench (Fang et al., 2024) | 噪声、冲突 | 知识库 | 英 | 开放域 QA |
| LRII (Wu et al., 2024) | 噪声、冲突 | 过滤后上下文 | 英 | 开放域 + 简单事实 QA |
| RECALL (Liu et al., 2023) | 冲突 | 过滤后上下文 | 英 | 开放域 + 文本生成 |
| ClashEval (Wu et al., 2024) | 冲突 | 过滤后上下文 | 英 | 领域特定 QA |
| PoisonedRAG (Zou et al., 2024) | 冲突 | 知识库 | — | — |
| Phantom (Chaudhari et al., 2024) | DoS | 知识库 | — | — |
| MAR (Shafran et al., 2024) | DoS | 知识库 | — | — |
| **SafeRAG (本文)** | **噪声、冲突、毒性、DoS** | **知识库/检索上下文/过滤后上下文** | **中文** | **领域特定 + 新闻 QA** |

> [!tip] 理解提示（非原文）
> 这是 RAG 安全领域最完整的相关工作表。几个关键发现：
> 1. **此前没有任何工作同时覆盖四种攻击类型**——SafeRAG 是第一个
> 2. **此前没有任何工作探索多阶段注入**——其他方法只在单一阶段注入
> 3. **首个中文 RAG 安全基准**——填补了非英文 RAG 安全评测的空白
> 4. **PoisonedRAG 是唯一的纯攻击方法**（未构建评测数据集），其余都是 benchmark

---

## 2.2 SafeRAG 在攻击构造上的创新

### 噪声攻击方面

现有方法（RGB、RAG Bench 的"检索-过滤-分类"策略、LRII 的无关噪声分类）集中于：
- 相似主题但不相关的上下文
- 相关但不含答案的上下文

这些噪声很容易被过滤器识别和排除。SafeRAG 首次构造了**"银噪声"**——**部分正确、部分错误或信息不完整的证据**，更贴近真实攻击场景，能绕过过滤器的相关性判断。

### 冲突攻击方面

大多数现有工作依赖 **LLM 生成反事实扰动**来构造冲突：
- **问题**：可能错误地改变关键事实，生成"相似主题但不相关"甚至"幻觉相关"的上下文

SafeRAG 采用**人工构造**冲突的策略，更可靠地构建**高质量、故意误导性的上下文间冲突**。

RECALL 是少数使用人工构造的工作，但仅关注 context-memory 冲突。SafeRAG 的上下文间冲突（inter-context conflict）利用 LLM 缺乏参数知识的弱点——LLM 不知道该信哪个来源。

### DoS 攻击方面

现有方法的问题：
- **Phantom**：注入"Sorry, I don't know..." → 被过滤器拦截（不支持回答问题）
- **MAR**：注入"I cannot provide a response that may perpetuate or encourage harmful content" → 被生成器忽略（混在证据中时 LLM 优先采信证据）

SafeRAG 提出了**"白 DoS"**——以**安全警告为伪装**，虚假指控证据包含大量不实信息。这种攻击：
1. 更容易通过过滤器（看似保护用户）
2. 更容易被 LLM 采信（LLM 被训练为信任安全警告）

### 毒性攻击方面

**此前没有专门针对 RAG 在毒性场景下的研究**。现有毒性研究主要聚焦于直接针对 LLM 的提示注入。SafeRAG 首次将毒性攻击纳入 RAG 安全评测，特别关注能够轻易绕过检索器、过滤器和生成器的**隐式毒性（软广告）**。

---

## 2.3 RAG 安全评测指标体系

现有安全评测指标分为两类：

**基于规则的方法**：RGB、RAG Bench、PoisonedRAG 使用传统指标（EM、F1、Recall、Precision、ASR）。

**基于模型的方法**：LRII（Misleading Rate、Uncertainty Ratio）、RECALL（Misleading Rate、Mistake Reappearance Rate）、ClashEval（Prior Bias、Context Bias）。

SafeRAG 结合了两类方法的优点，并针对不同攻击类型设计了专用指标，且所有指标都与人工判断做了对齐验证。

---

## 下一步阅读

[[03-attack-taxonomy|第 3 节：四类攻击任务详解]]——Silver Noise、Inter-Context Conflict、Soft Ad、White DoS 的形式化定义与构造方法
