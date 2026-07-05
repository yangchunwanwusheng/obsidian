---
name: peer-review
description: 系统化审稿工具箱——评估方法学、统计、实验设计、可复现性、伦理、图表完整性与报告规范，分节给出建设性意见，适用于会议/期刊稿件与基金评审。触发词：审稿 / peer review / 稿件评审 / review 意见 / 方法学评估 / 帮我评审这篇论文。
allowed-tools: [Read, Write, Edit, Bash]
---

# Peer Review（同行评审）

> 系统化评审科学稿件：从初判到分节精评，给出严谨且建设性的意见。

## 核心理念
好的评审是可操作、有依据、分级的——区分致命缺陷（major）与可修问题（minor），指出问题同时给改进路径。对照目标 venue 的标准校准严格度。

## 操作流程

### Stage 1：初判
- 中心问题/假设是什么？主要发现与结论？
- 工作是否科学可靠、有显著性、契合目标 venue？有无立即否决的致命缺陷？
- 输出 2-3 句稿件本质摘要 + 初步印象。

### Stage 2：分节精评
- **Abstract/Title**：是否准确反映内容、清晰、可被广义读者理解。
- **Introduction**：背景是否充分且最新、动机是否清楚、新颖性是否明确、相关工作是否引全。
- **Methods**：可复现性（他人能否重现）、严谨性、细节、伦理、统计合理性；核查样本量/功效、随机化/盲法、软件版本、多重比较校正。
- **Results**：呈现是否清晰、图表是否规范、统计报告是否完整（效应量/置信区间/p 值）、是否过度解读、是否含负结果。
- **Discussion/Conclusion**：结论是否被证据支撑、局限是否诚实、与已有工作对比是否公允。

### Stage 3：意见分类与产出
- 每条意见标 **Major / Minor**，给具体位置与改进建议。
- 检查报告规范合规性（CONSORT / STROBE / PRISMA 等适用时）。
- 汇总为结构化审稿意见（Summary → Strengths → Major → Minor → Questions）。

## D:\note 适配
- 审稿记录 → `D:\note\research/05-submission/_reviews/YYYY-MM-DD-venue.md`（type: review-record）。
- 校准标准可参考 `research/05-submission/_journals/journal-registry.md` 中目标 venue 的要求。
- 被评稿件若在库内，读 `research/04-writing/` 下草稿。
- 完成后更新 `wiki/log.md` 并按仓库 Git 规则 commit + push。
- 注：本 skill 是"评审他人稿件/自审"；真实收到审稿意见后的回复走本机 `review-response` skill / `/rebuttal`。

## Self-evolution
如果 `LESSONS.md` 存在，先读它再执行。每次使用后，若发现评审维度缺失 / venue 校准偏差 / 用户纠错，写入 LESSONS.md 并通过 `/skill-evolve` 升级。
