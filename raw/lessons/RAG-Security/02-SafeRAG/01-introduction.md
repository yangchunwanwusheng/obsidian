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

# 01 — 摘要与引言

> 论文：SafeRAG: Benchmarking Security in Retrieval-Augmented Generation of Large Language Model
> 发表：ACL 2025 | arXiv: 2501.18636
> 作者：Xun Liang, Simin Niu, Zhiyu Li (通讯) 等 11 人（人大 + IAAR + 北航）

---

## 摘要（Abstract）

### 原文翻译

检索增强生成（RAG）的"索引-检索-生成"范式通过将外部知识集成到大语言模型（LLM）中，在解决知识密集型任务方面取得了巨大成功。然而，引入**外部且未经验证的知识**增加了 LLM 的脆弱性——攻击者可以通过操纵知识来执行攻击任务。

本文介绍了一个名为 **SafeRAG** 的基准，旨在评估 RAG 的安全性。

首先，我们将攻击任务分类为四类：**银噪声（Silver Noise）、上下文间冲突（Inter-Context Conflict）、软广告（Soft Ad）和白拒绝服务（White Denial-of-Service）**。

其次，我们**以人工构造为主**为每类任务构建了 RAG 安全评估数据集（SafeRAG 数据集）。

然后，我们利用 SafeRAG 数据集模拟 RAG 可能遇到的各种攻击场景。在 **14 个代表性 RAG 组件**上进行的实验表明，RAG 对所有攻击任务都表现出**显著的脆弱性**——即使是最明显的攻击任务也能轻易绕过现有的检索器、过滤器或高级 LLM，导致 RAG 服务质量下降。

代码开源：https://github.com/IAAR-Shanghai/SafeRAG

> [!note] 译者注（非原文）
> SafeRAG 的贡献不在于提出新的攻击方法（像 PoisonedRAG 那样），而在于**构建评测基准**——这是"怎么测"而非"怎么打"。它系统性地揭示了 RAG 安全的全貌：四种攻击类型 × 四个注入阶段 × 14 个 RAG 组件。

---

## 1. 引言（Introduction）

### 正文翻译

#### RAG 的成功与隐忧

检索增强生成（RAG）[Zhao et al., 2024; Gupta et al., 2024; Fan et al., 2024; Wang et al., 2024] 为扩展 LLM 的知识边界提供了高效解决方案。许多先进 LLM，如 ChatGPT [OpenAI et al., 2024]、Gemini [Team et al., 2024] 和 Perplexity.ai，已在其网络平台中融入了外部检索模块。

然而，在 RAG Pipeline 中，与查询相关的文本依次经过**检索器**、**过滤器**，最后被**生成器**合成为回答——这引入了潜在的安全风险，因为**攻击者可以在 Pipeline 的任何阶段操纵文本**。

> [!tip] 理解提示（非原文）
> 这是 SafeRAG 与 PoisonedRAG 的第一大区别——视角不同。PoisonedRAG 聚焦于"知识库注入"这一个攻击点，SafeRAG 则关注整个 Pipeline 的每个阶段（知识库 → 检索上下文 → 过滤后上下文 → 生成）。这种系统性视角是 SafeRAG 的核心特色。

#### 现有四类攻击及其致命缺陷

当前针对 RAG 的攻击可分四类，但**每类都有被现有安全组件防御的致命缺陷**：

**R-1：噪声攻击** → 简单安全过滤器即可防御。现有噪声集中于"相似主题但不相关"或"相关但不含答案"的上下文——过滤器能轻易识别。

**R-2：冲突攻击** → 自适应检索器已能缓解。现有冲突主要测试 context-memory 冲突（外部文档 vs LLM 内部知识）——LLM 凭参数知识即可判断。

**R-3：毒性攻击** → 高级生成器能检测。生成器在检测偏见、歧视、隐喻和讽刺方面已有强大能力。

**R-4：DoS 攻击** → 过滤器拦截或生成器忽略。显式拒绝信号（如"I don't know"）要么被过滤，要么被混入证据时被忽略。

#### SafeRAG 的四类升级攻击

针对上述四大盲区，SafeRAG 提出了升级版攻击：

| 新型攻击 | 针对的盲区 | 核心策略 |
|---------|----------|---------|
| **Silver Noise（银噪声）** | R-1：过滤器可防噪声 | 部分包含答案——能绕过过滤器，但稀释有用知识 |
| **Inter-Context Conflict（上下文间冲突）** | R-2：检索器可防 context-memory 冲突 | LLM 缺乏参数知识判断的外部冲突，更易被误导 |
| **Soft Ad（软广告）** | R-3：生成器可防毒性 | 伪装成权威陈述的隐式毒性，能绕过内容检测 |
| **White DoS（白拒绝服务）** | R-4：过滤器/生成器可防 DoS | 以"安全警告"为伪装，虚假指控证据不实 |

#### 核心贡献

1. 揭示了**四类能绕过现有 RAG 安全组件的升级攻击任务**
2. 构建了以人工构造为主、LLM 辅助的轻量级 RAG 安全评估数据集
3. 提出了经济高效的攻击特定评测指标体系，与人工判断高度一致
4. 构建了**首个中文 RAG 安全基准 SafeRAG**，分析了 14 个 RAG 组件在四类攻击下的风险
5. 系统评估了噪声、冲突、毒性、DoS 在不同 Pipeline 阶段注入的影响

---

## 下一步阅读

[[02-related-work|第 2 节：相关工作]]——RAG 安全评测数据集全景表、SafeRAG 在攻击构造上的创新对比
