---
type: project-hub
project: closed-loop-safe-multi-agent-reasoning
title: Closed-Loop Safe Multi-Agent Reasoning — 项目中枢
status: active
tags: [multi-agent, hallucination, closed-loop-control, communication-protocol, anomaly-detection, reinforcement-learning]
created: 2026-05-17
updated: 2026-05-17
target_conference: NeurIPS / ICLR / ACL
---

# Closed-Loop Safe Multi-Agent Reasoning

> 将多智能体推理中的错误传播建模为一个**部分可观测的动态风险控制问题**，统一处理预防、检测、恢复与学习。

## 核心定位

> **A belief-guided high-level controller that adaptively selects communication constraints and recovery interventions under partial observability.**

## 项目结构

```
research/closed-loop-safe-multi-agent-reasoning/
├── 00-hub.md                         # 本文件 — 项目中枢
├── 01-paper/                         # 论文各章节
│   ├── abstract.md
│   ├── introduction.md
│   ├── related-work.md
│   ├── problem-formulation.md
│   ├── method.md
│   ├── experimental-setup.md
│   ├── results.md
│   ├── conclusion.md
│   └── appendix.md
├── 02-figures/                       # 图表规划
│   └── figure-table-plan.md
├── 03-experiments/                   # 实验执行
│   └── experiment-checklist.md
└── 04-writing/                       # 写作支撑
    └── claim-evidence-sheet.md
```

## 快速导航

### 论文章节
- [[01-paper/abstract|Abstract]]
- [[01-paper/introduction|Introduction]]
- [[01-paper/related-work|Related Work]]
- [[01-paper/problem-formulation|Problem Formulation]]
- [[01-paper/method|Method]]
- [[01-paper/experimental-setup|Experimental Setup]]
- [[01-paper/results|Results and Analysis]]
- [[01-paper/conclusion|Conclusion]]
- [[01-paper/appendix|Appendix]]

### 实验与图表
- [[02-figures/figure-table-plan|Figure & Table Plan]]
- [[03-experiments/experiment-checklist|Experiment Execution Checklist]]

### 论证支撑
- [[04-writing/claim-evidence-sheet|Claim–Evidence Sheet]]

## 关联资源

### 核心参考论文
- [[raw/lessons/G2CP/00-essence|G2CP]] (AAMAS 2026) — 预防：graph-grounded communication
- [[raw/lessons/GUARDIAN/00-essence|GUARDIAN]] (NeurIPS 2025) — 检测：temporal graph anomaly detection
- [[raw/lessons/MARCH/00-essence|MARCH]] (ACL 2026) — 学习：multi-agent RL self-check

### 代码仓库
- `C:\Users\oobbee\Desktop\mah_seasrch\`
- 核心实现：`src/orchestrator.py`, `src/controller/`, `src/risk_observer/`

### 文献地图
- [[research/02-papers/multi-agent-hallucination-reading-guide|Multi-Agent Hallucination Reading Guide]]

## 当前状态

| 维度 | 状态 |
|------|------|
| 研究设计 | ✅ 完成 |
| 论文草稿 | ✅ 所有章节初稿完成 |
| 实验数据 | 🔴 待产出（所有数值为 [FILL]） |
| 图表绘制 | 🔴 待绘制（5 张 PDF 图） |
| 代码实现 | ✅ 核心模块完成，trainer/experiment_runner 待提交 |

## 下一步

1. 运行 `python src/experiment_runner.py` 产出实验数据
2. 按 [[02-figures/figure-table-plan|图表规范]] 绘制 5 张图
3. 将所有 [FILL] 替换为实际数值
4. 投稿前补充 Reproducibility Checklist / Ethics Statement
