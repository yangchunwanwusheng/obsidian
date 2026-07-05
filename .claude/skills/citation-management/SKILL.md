---
name: citation-management
description: 学术引用系统化管理——检索论文、抽取准确元数据、校验引用、生成规范 BibTeX，并做重复清理与 APA/MLA/Chicago/Nature 格式化。触发词：引用管理 / citation / BibTeX / 参考文献整理 / DOI 转 BibTeX / 查重引用 / 文献格式化。
allowed-tools: [Read, Write, Edit, Bash]
---

# Citation Management（引用管理）

> 在研究与写作全程系统化管理引用：检索 → 抽元数据 → 校验 → 规范 BibTeX → 去重 → 多风格格式化。

## 核心理念
引用错误会毁掉论文可信度。每条引用的元数据必须来自权威源（CrossRef / arXiv / Semantic Scholar / DBLP），不得凭记忆手写。可复现、可核验、格式一致是底线。

## 操作流程

1. **论文发现**
   - 论文检索走 `semantic-scholar` MCP + `arxiv` MCP；跨学科补充用 `web-search-prime`（兜底 MiniMax web_search）。
   - 精确短语加引号、按作者/标题限定、按年份过滤，找高被引种子论文。

2. **元数据抽取**
   - 从 DOI / PMID / arXiv ID 抽全字段（作者、标题、期刊、年份、卷期页、DOI）。
   - 交叉多源核对，确保与实际发表版本一致（注意 arXiv 预印本 vs 正式 venue 差异）。

3. **BibTeX 生成与校验**
   - 用 `semantic-scholar` 的 `export_bibtex` 生成条目；统一 cite key 规则（author_year）。
   - 校验：字段完整性、venue 正确性、年份一致性。

4. **去重与清理**
   - 检测重复条目（同 DOI/标题合并），删除未被引用的冗余条目。
   - 统一大小写、缩写、页码格式。

5. **多风格格式化**
   - 按目标 venue 输出 APA / MLA / Chicago / Nature / Vancouver 等风格。

## D:\note 适配
- BibTeX 与引用清单 → `D:\note\research/04-writing/`（论文草稿同目录）或 `research/06-tools/` 下的引用工具策略。
- 与 `research/02-papers/` 的论文笔记 frontmatter（sources 字段）保持一致。
- 工具策略参考 `D:\note\research/06-tools/zotero-obsidian-workflow.md`。
- 密钥只放 `.env`；完成后更新 `wiki/log.md` 并按仓库 Git 规则 commit + push。
- 原版 `scripts/search_google_scholar.py` 等本地脚本用本机 MCP 工具替代，不移植脚本。

## Self-evolution
如果 `LESSONS.md` 存在，先读它再执行。每次使用后，若发现元数据源不准 / 格式化规则漏洞 / 用户纠错，写入 LESSONS.md 并通过 `/skill-evolve` 升级。
