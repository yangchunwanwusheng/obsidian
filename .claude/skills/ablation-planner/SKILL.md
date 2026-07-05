---
name: ablation-planner
description: 系统化设计消融实验，回答 reviewer 必问的问题——隔离每个新颖组件的贡献、测超参敏感性、对比自然的替代设计选择，给出优先级与运行顺序。触发词：消融实验 / ablation / 消融设计 / ablation study / 组件贡献分析 / reviewer 会问的实验。
argument-hint: [方法描述或论断]
allowed-tools: [Bash, Read, Grep, Glob, Write, Edit]
---

# Ablation Planner（消融实验设计）

> 系统化设计消融，回答 reviewer 一定会问的问题。从 reviewer 视角设计，再校验可行性并落地。

## 上下文：$ARGUMENTS

## 何时使用
- 主结果已通过 `result-to-claim`（claim_supported = yes 或 partial）。
- 用户明确要求消融规划，或 review-loop 指出缺消融。

## 操作流程

### Step 1 准备上下文
读项目文件建立全貌：方法描述与组件（`research_contract.md` 或项目 CLAUDE.md）、当前实验结果（EXPERIMENT_LOG / TRACKER / W&B）、已确认与拟主张的论断、可用算力。

### Step 2 从 reviewer 视角设计消融
对给定方法与结果，设计消融以：①隔离每个新颖组件的贡献；②回答 reviewer 一定会问的问题；③测关键超参敏感性；④对比自然的替代设计。
每个消融指定：`name`（改什么，如"移除模块 X""用 Z 替换 Y"）、`what_it_tests`、`expected_if_component_matters`（若组件重要预测什么）、`priority`（1 必跑 ~ 5 可选）。
另给：覆盖度评估、不必要的消融（看似有用实则无信息，跳过）、建议运行顺序（早期信息最大化）、算力估计（GPU·小时）。

### Step 3 结构化消融计划
归类成三张表 + 评估：
- **组件消融**（最高优先）：# / Name / 测什么 / 若重要预测 / 优先级。
- **超参敏感性**：# / 参数 / 待测值 / 测什么 / 优先级。
- **设计选择对比**：# / Name / 测什么 / 优先级。
- **覆盖度评估** + **不必要消融清单**（明确跳过）。

## D:\note 适配
- 消融计划产物 → `D:\note\research/03-experiments/`（衔接 experiment-log 模板与本机 `experiment-plan` skill）。
- 主结果与论断来源对齐 `research/04-writing/` 与 experiment 日志。
- 原版 Codex 主导设计 + CC 复核可用本机主模型（reviewer 视角）+ `code-reviewer`/子代理复核可行性替代。
- 长消融跑批遵守本机长任务规则（`run_in_background` + 日志轮询，resume 记入 `.claude/state/NOW.md`）。
- 完成后更新 `wiki/log.md` 并按仓库 Git 规则 commit + push。

## Self-evolution
如果 `LESSONS.md` 存在，先读它再执行。每次使用后，若发现漏掉的 reviewer 问题 / 优先级误判 / 用户纠错，写入 LESSONS.md 并通过 `/skill-evolve` 升级。
