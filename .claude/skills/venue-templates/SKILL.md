---
name: venue-templates
description: CCF-A 类会议与主流期刊的 LaTeX 模板、格式要求与投稿指南目录（Nature/Science/PLOS/IEEE/ACM 期刊，NeurIPS/ICML/ICLR/CVPR/ACL/AAAI/KDD 会议，海报与基金）。触发词：会议模板 / venue template / 投稿格式 / 页数要求 / LaTeX 模板 / camera-ready 格式 / 目标会议要求。
allowed-tools: [Read, Write, Edit, Bash]
---

# Venue Templates（投稿模板目录）

> 为目标 venue 提供 LaTeX 模板、格式规范与投稿指南，确保稿件合规。

## 核心理念
每个 venue 有独特的模板、页数限制、匿名规则与引用风格。投稿前先锁定 venue 规范，避免因格式被 desk reject。本 skill 是"目录 + 规范"，与本机 `latex-conference-template-organizer`（整理模板 zip）、`latex-document-suite`（编译/排版）互补。

## 覆盖范围

- **ML/AI 会议**：NeurIPS、ICML、ICLR、CVPR、AAAI、ACL、EMNLP、KDD（CCF-A 优先）。
- **期刊**：Nature Portfolio、Science 家族、PLOS、Cell Press、IEEE Transactions、ACM Transactions、Springer/Elsevier/Wiley。
- **其他**：研究海报、基金申请（NSF/NIH/DOE/DARPA）。

## 操作流程

1. **识别目标 venue**：确认会议/期刊 + 当年 CFP 版本。
2. **拉取规范**：模板不确定时用 `web-search-prime` 查官方 CFP/作者指南，核对页数、模板版本、匿名要求、引用风格、补充材料规则。
3. **匹配模板**：定位对应 LaTeX 模板（如 KDD/ACM 匿名模式预设、NeurIPS style file）。
4. **定制填充**：填作者信息（注意双盲时匿名化）、标题、致谢占位。
5. **合规检查**：页数、字体、边距、引用风格、图表清晰度、camera-ready 差异。

## D:\note 适配
- 模板与规范笔记 → `D:\note\research/05-submission/_journals/[venue-name].md`（type: journal-info）。
- venue 选择参考 `research/05-submission/_journals/journal-registry.md`（含截稿时间、CCF 等级、方向匹配度）。
- 实际模板 zip 整理 → 交本机 `latex-conference-template-organizer`；编译排版 → `latex-document-suite`。
- 完成后更新 `wiki/log.md` 并按仓库 Git 规则 commit + push。

## Self-evolution
如果 `LESSONS.md` 存在，先读它再执行。每次使用后，若发现 venue 规范过时 / 模板缺失 / 用户纠错，写入 LESSONS.md 并通过 `/skill-evolve` 升级。
