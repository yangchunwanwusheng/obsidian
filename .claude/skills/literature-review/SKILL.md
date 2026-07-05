---
name: literature-review
description: 系统化文献综述流程（PRISMA 风格），跨 arXiv / Semantic Scholar / 网络多源检索、主题化综合、逐条引用核验，输出到 D:\note\research/02-papers 下。触发词：文献综述 / literature review / 系统综述 / related work / meta-analysis / 研究现状调研。
allowed-tools: [Read, Write, Edit, Bash]
---

# Literature Review（文献综述）

> 用严谨学术方法做系统化文献综述：多源检索 → 主题综合 → 引用核验 → 结构化输出。

## 核心理念
文献综述不是论文堆砌，而是"检索策略 → 筛选 → 主题综合 → 研究空白识别"的可复现流程。每条引用都必须被核验，禁止编造。

## 操作流程（PRISMA 风格）

1. **定题与范围（Scoping）**
   - 用 PICO / 关键概念拆解研究问题，列同义词与 Boolean 组合。
   - 设定纳入/排除标准：年份、语言、发表类型、研究设计，全部写清。

2. **系统检索（Search）**
   - 论文检索强制走 `semantic-scholar` MCP + `arxiv` MCP（引用/venue 过滤、BibTeX）；网络补充用 `web-search-prime`（兜底 MiniMax web_search）。
   - 优先 2025 年至今顶会顶刊（NeurIPS/ICML/ICLR/ACL/AAAI）+ 高被引经典。
   - 记录每个库的检索日期、检索式、命中数（PRISMA 流程数）。

3. **筛选与精读**
   - 去重 → 标题/摘要筛 → 全文筛，记录各阶段淘汰数。
   - 每篇入 `research/02-papers/` 用 paper-note 模板，或综述篇入 `research/02-papers/_surveys/`。

4. **主题化综合**
   - 按主题（非按论文）组织：已解决什么、未解决什么、争议点。
   - 生成领域知识图谱 `research/02-papers/<领域>-knowledge-graph.md`。

5. **研究空白（Gap）**
   - 列 3-5 个具体 Gap，标注证据来源，供选题决策。

6. **引用核验**
   - 逐条核对作者/年份/venue/标题，禁止幻觉引用；BibTeX 用 semantic-scholar 导出。

## D:\note 适配
- 综述产物 → `D:\note\research/02-papers/`（综述 `_surveys/`、待读 `_reading-queue/`、图谱同目录）。
- 阅读队列命名 `_reading-queue/YYYY-MM-DD.md`。
- 完成后更新 `wiki/index.md` 与 `wiki/log.md`（操作类型 `research`），并按仓库 Git 规则 commit + push。
- 重检索任务派 `literature-reviewer` 子代理执行，主上下文只接收结论。
- 原版依赖的 scientific-schematics 出图流程可选，用本机 `diagram-drawio-suite` 替代。

## Self-evolution
如果 `LESSONS.md` 存在，先读它再执行。每次使用后，若发现检索/筛选/核验流程缺陷或用户纠错，写入 LESSONS.md 并通过 `/skill-evolve` 升级。
