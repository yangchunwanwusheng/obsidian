---
name: citation-audit
description: 零上下文核验论文里每条引用是否真实、归属正确、且用在被引论文真正支持的语境——抓幻觉作者、错误年份、伪造 venue、版本错配、错语境引用。触发词：审查引用 / 引用核对 / citation audit / verify references / 伪造引用检测 / 投稿前引用核查。
argument-hint: "[论文目录或 bib 文件] [--uncited]"
allowed-tools: [Bash, Read, Grep, Glob, Edit, Write, WebSearch, WebFetch]
---

# Citation Audit（引用审计 / 伪造引用检测）

> 对论文里每个 `\cite{...}` 做三层独立核验，专抓审稿场景下危险的"看似真实"引用问题。

## 核心理念
危险的引用不是明显造假（那好抓），而是：**错语境**（真论文但不支持所引论点）、作者幻觉、标题漂移（arXiv v1 vs v3）、venue 混淆（预印本 vs 正式会议）、年份错配、幻影 DOI、自引年份偏移。投稿前必须核清。

## 三层核验
1. **存在性**——被引论文真的存在于所声称的 arXiv ID / DOI / venue。
2. **元数据正确性**——作者、年份、venue、标题与权威源（DBLP、arXiv、ACL Anthology、Nature、OpenReview）一致。
3. **语境恰当性**——被引论文确实支持它在稿件中被用来支持的论点。

## 操作流程

1. **定位**：找 `references.bib`（或 paper.bib）与所有含 `\cite{}` 的 `*.tex`（通常 `sec/`）。
2. **抽取 (cite-key, 语境) 对**：每次 `\cite{}` 记录 key、文件行号、周围整句；建 `cited_keys` 与 `bib_keys` 两集合。
3. **逐条核验**：对每个被引条目做真实网络/DBLP/arXiv 查询（禁止凭记忆），三层逐一判定。
4. **产出报告**：`CITATION_AUDIT.md`（逐条 verdict：OK / FIX / REPLACE / REMOVE + 理由）+ 机读 `CITATION_AUDIT.json`。
5. **--uncited（可选）**：算 `bib_keys \ cited_keys` 报告未被引用的冗余条目（仅检测不核验）。
6. **--soft-only（可选）**：禁止改动 .bib，改为对 .tex 提出逐句改写建议。

> 🔒 不要把本 skill 放进循环/定时任务——它是判定性的，只在 bibliography 变化时重跑一次即可。

## D:\note 适配
- 网络核验走 `web-search-prime`（兜底 MiniMax web_search）+ `semantic-scholar` MCP + `arxiv` MCP，禁止裸 WebFetch 做发现。
- 审计对象论文 → `D:\note\research/04-writing/`；报告写在该论文目录下，并抄要点到 `research/05-submission/`。
- 原版 Codex 跨模型 fresh-thread 评审可用本机独立子代理（一次一条、互不串上下文）替代。
- 完成后更新 `wiki/log.md` 并按仓库 Git 规则 commit + push。

## Self-evolution
如果 `LESSONS.md` 存在，先读它再执行。每次使用后，若发现漏判的引用陷阱 / 权威源缺口 / 用户纠错，写入 LESSONS.md 并通过 `/skill-evolve` 升级。
