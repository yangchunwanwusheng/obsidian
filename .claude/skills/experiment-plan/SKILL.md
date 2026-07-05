---
name: experiment-plan
description: 把已细化的研究提案/方法 idea 变成"论断驱动"的实验路线图——实验矩阵、评测协议、运行顺序、算力预算、可支撑论文的验证设计。触发词：实验计划 / experiment plan / 实验矩阵 / 评测协议 / 运行顺序 / 算力预算 / 论文实验设计。
allowed-tools: [Bash, Read, Write, Edit, Grep, Glob, WebSearch, WebFetch]
---

# Experiment Plan（论断驱动的实验路线图）

> 细化并具体化：**$ARGUMENTS**。把提案变成"论断 → 证据 → 运行顺序"的路线图，而非庞大的 benchmark 愿望清单。

## 核心理念
实验计划要支撑四件事：①方法真的解决锚定问题；②主导贡献真实且聚焦；③方法足够优雅、无需额外复杂度；④任何前沿组件（LLM/VLM/Diffusion/RL）是真有用而非装饰。

## 操作流程

- **Phase 0 载入提案上下文**：优先读 `FINAL_PROPOSAL.md` / `REVIEW_SUMMARY.md` / `REFINEMENT_REPORT.md`，抽取问题锚点、主导贡献、reviewer 关切、数据/算力/时间约束、核心前沿原语。缺失则从用户 prompt 推导。
- **Phase 1 冻结论文论断**：写下必须捍卫的论断——
  - **主论断**（机制级主贡献，≤ 2 条）
  - **支撑论断**（可选，只在直接强化主线时保留）
  - **需排除的反论断**（如"增益只来自更多参数/更大搜索空间/前沿组件只是装饰"）
  - **最小可信证据**：什么结果能让强 reviewer 信服。
- **Phase 2 搭实验故事线**（≤ 5 个核心块，删掉不需要的）：①主锚点结果 ②新颖性隔离 ③简洁性检验 ④前沿必要性检验 ⑤失败分析/定性诊断。每块归属：主文 / 附录 / 砍掉。强 baseline 少而精（家族 ≤ 3）。
- **Phase 3 逐块细化**：论断、存在理由、数据/split/任务、对比系统（最强 baseline + 消融 + 变体）、指标（决定性优先）、种子数（默认 3）、算力估计、运行顺序。

## D:\note 适配
- 需要查数据集/baseline/SOTA 时走 `web-search-prime` + `semantic-scholar`/`arxiv` MCP。
- 实验计划产物 → `D:\note\research/03-experiments/`（衔接 `experiment-log` 模板与每日/每周科研日志）。
- 与 `research/01-topics/` 的选题、`research/04-writing/` 的论文论断保持一致。
- 长实验遵守本机规则：>30 分钟用 `run_in_background` + 日志轮询，不交子代理；resume 命令记入 `.claude/state/NOW.md`。
- 完成后更新 `wiki/log.md` 并按仓库 Git 规则 commit + push。

## Self-evolution
如果 `LESSONS.md` 存在，先读它再执行。每次使用后，若发现论断/证据映射缺口 / 预算估计偏差 / 用户纠错，写入 LESSONS.md 并通过 `/skill-evolve` 升级。
