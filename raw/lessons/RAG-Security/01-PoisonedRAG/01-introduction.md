---
type: lesson
tags: [RAG, 安全, 对抗攻击, 知识污染, USENIX-Security, 论文翻译]
created: 2026-06-02
updated: 2026-06-02
difficulty: advanced
prerequisites: [RAG基础概念, Embedding, 检索器原理]
topic: RAG安全——知识污染攻击
status: completed
series: { name: "RAG安全论文精读", part: 1 }
sources: [raw/assets/RAG——paper/PoisonedRAG： Knowledge Corruption Attacks to Retrieval-Augmented Generation of Large Language Models.pdf]
---

# 01 — 摘要与引言

> 论文：PoisonedRAG: Knowledge Corruption Attacks to Retrieval-Augmented Generation of Large Language Models
> 发表：USENIX Security 2025 | arXiv: 2402.07867
> 作者：Wei Zou, Runpeng Geng (Penn State), Binghui Wang (IIT), Jinyuan Jia (Penn State)

---

## 摘要（Abstract）

### 原文翻译

大语言模型（LLM）因其卓越的生成能力取得了显著成功。尽管取得了成功，它们仍存在固有的局限性，如缺乏最新知识和幻觉问题。检索增强生成（RAG）是缓解这些局限性的最先进技术。RAG 的核心理念是将 LLM 的答案生成建立于从知识数据库中检索到的外部知识之上。现有研究主要集中在提高 RAG 的准确性或效率上，其安全性在很大程度上尚未被探索。我们旨在通过本工作弥补这一空白。

我们发现 RAG 系统中的知识数据库引入了一个**新的、实用的攻击面**。基于此攻击面，我们提出了 **PoisonedRAG**——首个针对 RAG 的知识污染攻击。在此攻击中，攻击者可以向 RAG 系统的知识数据库注入少量恶意文本，诱导 LLM 对攻击者选择的目标问题生成攻击者选择的目标答案。

我们将知识污染攻击形式化为一个优化问题，其解是一组恶意文本。根据攻击者对 RAG 系统的背景知识（黑盒和白盒设定），我们分别提出了两种求解该优化问题的方案。实验结果表明，PoisonedRAG 在向包含数百万条文本的知识数据库中对每个目标问题注入**五条**恶意文本时，可实现 **90% 的攻击成功率**。我们还评估了若干防御方法，结果表明它们不足以抵御 PoisonedRAG，凸显了对新型防御机制的迫切需求。

代码开源：https://github.com/sleeepeer/PoisonedRAG

---

## 1. 引言（Introduction）

### 正文翻译

#### LLM 的局限性

大语言模型如 GPT-3.5 [1]、GPT-4 [2] 和 PaLM 2 [3] 因其卓越的生成能力而被广泛部署于现实世界。尽管如此，它们也存在固有的局限性。例如：
- **缺乏最新知识**：因为它们是在过去的数据上预训练的（如 GPT-4 预训练数据的截止日期是 2023 年 4 月 [2]）
- **幻觉行为** [4]：生成不准确的答案
- **特定领域的知识缺口**：例如医疗领域

这些局限性对在医疗 [5, 6]、金融 [7]、法律 [8, 9] 和科研 [10, 11, 12] 等许多现实应用中部署 LLM 构成了严重挑战。

#### RAG 简介

检索增强生成（RAG）[13, 14, 15, 16] 是缓解这些局限性的最先进技术，它通过从知识数据库中检索到的外部知识来增强 LLM。如图 1 所示，RAG 有三个组件：

1. **知识数据库**：包含从各种来源收集的大量文本，如 Wikipedia [17]、金融文档 [7]、新闻文章 [18]、COVID-19 出版物 [19] 等
2. **检索器**：用于从知识数据库中检索与问题最相关的一组文本
3. **LLM**：借助系统提示，将检索到的文本作为上下文来生成答案

RAG 使 LLM 能够以即插即用的方式利用外部知识，可以减少幻觉并增强 LLM 的领域专业知识。

由于这些优势，我们已看到各种开发的工具（ChatGPT Retrieval Plugin [20]、LlamaIndex [21]、ChatRTX [22]、LangChain [23]）和现实应用（WikiChat [24]、Bing Search [25]、Clinfo.AI [26]、Google Search with AI Overviews [27]、Perplexity AI [28]、LLM Agent [29, 30]）。

> [!tip] 理解提示（非原文）
> 引言的开篇结构非常经典——"问题→现有方案→现有方案的局限→本文贡献"。作者先列举 LLM 的三大固有缺陷（知识截止、幻觉、领域知识缺口），引出 RAG 作为解决方案，再指出 RAG 的安全性研究空白，为 PoisonedRAG 的提出做好铺垫。

#### 现有研究的不足

现有研究 [31-36] 主要集中在提高 RAG 的准确性和效率上。例如，一些研究 [32, 33, 36] 设计了新的检索器以检索更相关的知识，另一些研究 [31, 34, 35] 提出了各种提高知识检索效率的技术。然而，**RAG 的安全性在很大程度上未被探索**。为弥补这一空白，我们提出了 **PoisonedRAG**——首个针对 RAG 的知识污染攻击。

#### 知识数据库作为新的攻击面

在本工作中，我们发现 RAG 系统的知识数据库引入了一个新的、实用的攻击面。具体而言，攻击者可以向 RAG 的知识数据库注入恶意文本，以诱导 LLM 生成攻击者期望的答案。攻击向量包括：

- 当知识数据库从 Wikipedia 收集时，攻击者可通过**恶意编辑 Wikipedia 页面** [37] 注入恶意文本
- 当知识数据库从互联网收集时，攻击者可**发布假新闻或托管恶意网站**
- **内部人员**可向企业私有知识数据库注入恶意文本

#### 威胁模型

在 PoisonedRAG 中，攻击者首先选择一组**目标问题**（target questions），并为每个目标问题选择一个任意的**目标答案**（target answer）。攻击者的目标是向知识数据库注入恶意文本，使 LLM 为每个目标问题生成对应的目标答案。

攻击示例：
- **错误信息攻击**：目标问题 "Who is the CEO of OpenAI?" → 目标答案 "Tim Cook"
- **商业偏见攻击**：推荐特定品牌而非其他品牌
- **金融虚假信息**：虚假声称某公司面临破产

> [!warning] 易混点（非原文）
> 这里的"目标答案"不是 ground truth（正确答案），而是攻击者想要 LLM 输出的**任意答案**。攻击成功意味着 LLM 输出了攻击者指定的错误答案，而不是正确答案。

攻击者的能力边界：
- **不能**访问知识数据库中的文本
- **不能**访问/查询 RAG 中的 LLM
- **可能知道也可能不知道**检索器参数

据此分为两种设定：
- **白盒设定**：攻击者可访问检索器参数（如 RAG 使用公开可用的检索器）
- **黑盒设定**：攻击者既不能访问参数也不能查询检索器

攻击者可以向知识数据库注入**少量**恶意文本。

#### PoisonedRAG 概览

我们将恶意文本的构造形式化为一个优化问题。然而，直接求解该优化问题极具挑战性。因此，我们采用启发式解决方案，推导出恶意文本能导致有效攻击的**两个条件**：

- **检索条件（Retrieval Condition）**：恶意文本能被检索器检索到用于目标问题
- **生成条件（Generation Condition）**：当恶意文本作为上下文时，能误导 LLM 对目标问题生成目标答案

> [!note] 译者注（非原文）
> 这两个条件是整篇论文最核心的理论贡献。它们分别对应 RAG 的两个阶段：检索阶段（恶意文本必须被"找到"）和生成阶段（恶意文本必须能"说服"LLM）。单独满足任一条件都无法构成有效攻击——这是 PoisonedRAG 区别于传统提示注入攻击的关键。

我们随后在黑盒和白盒设定下设计了同时满足这两个条件的攻击方法。核心思路是将恶意文本**分解为两个子文本**，分别服务于两个条件。拼接后，它们同时满足这两个条件。

#### PoisonedRAG 的评估

我们在多个数据集（Natural Questions (NQ) [38]、HotpotQA [39]、MS-MARCO [40]）、8 种 LLM（如 GPT-4 [2]、LLaMA-2 [41]）和三个真实应用（高级 RAG 方案、Wikipedia 聊天机器人、LLM Agent）上对 PoisonedRAG 进行了系统性评估。

使用**攻击成功率（Attack Success Rate, ASR）**作为评估指标，衡量目标答案被成功生成的目标问题比例。主要观察：

1. PoisonedRAG 能以**极少量的恶意文本**实现高 ASR。例如，在 NQ 数据集上，黑盒设定下仅注入 5 条恶意文本（知识库含 2,681,468 条干净文本），ASR 即达 **97%**
2. PoisonedRAG **远超**现有基线方法 [42, 43]。在 NQ 上，PoisonedRAG 黑盒 ASR 为 97%，而 5 种基线的 ASR 均低于 70%
3. 消融实验表明 PoisonedRAG 对不同超参数具有鲁棒性

#### 防御探索

我们探索了若干防御方法，包括释义（paraphrasing）[44] 和基于困惑度的检测（perplexity-based detection）[44, 45, 46]。结果表明这些防御**不足以抵御** PoisonedRAG，凸显了对新型防御机制的迫切需求。

#### 主要贡献

1. 提出了 **PoisonedRAG**——首个利用 RAG 知识数据库这一新攻击面的知识污染攻击
2. 推导了针对 RAG 系统有效攻击的**两个必要条件**，并设计了同时满足这两个条件的攻击方法
3. 在多个知识数据库、检索器、RAG 方案和 LLM 上进行了广泛评估，并与 5 种基线方法进行了比较
4. 探索了若干针对 PoisonedRAG 的防御方法

---

## 关键术语对照

| 英文 | 中文 | 说明 |
|------|------|------|
| Retrieval-Augmented Generation (RAG) | 检索增强生成 | LLM + 外部知识库检索 |
| Knowledge Database | 知识数据库 | 存储检索文本的向量/文本库 |
| Retriever | 检索器 | 从知识库中检索相关文本的组件 |
| Target Question | 目标问题 | 攻击者选定的、希望 LLM 被误导的问题 |
| Target Answer | 目标答案 | 攻击者希望 LLM 输出的（错误）答案 |
| Attack Success Rate (ASR) | 攻击成功率 | 目标答案被成功生成的比例 |
| White-box / Black-box | 白盒 / 黑盒 | 攻击者是否知道检索器参数 |
| Retrieval Condition | 检索条件 | 恶意文本必须能被检索到 |
| Generation Condition | 生成条件 | 恶意文本必须能误导 LLM 生成目标答案 |

---

## 下一步阅读

[[02-background|第 2 节：背景与相关工作]]——RAG 系统的形式化定义、现有 LLM 攻击方法的分类与局限
