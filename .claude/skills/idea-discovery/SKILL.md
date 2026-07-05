---
name: idea-discovery
description: 从一个宽泛研究方向到"经验证、可试点"的 idea 全流程管线——融合 arXiv + Semantic Scholar + OpenReview 的调研、头脑风暴、新颖性核验、批判反馈与方法/实验细化。触发词：idea 发现 / 找 idea 全流程 / idea discovery / 从零开始找方向 / 选题管线 / 研究方向探索。
argument-hint: [研究方向]
allowed-tools: [Bash, Read, Write, Edit, Grep, Glob, WebSearch, WebFetch, Skill]
---

# Idea Discovery（idea 发现全流程）

> 把一个宽泛方向 **$ARGUMENTS** 变成排序好的、经新颖性核验的候选 idea + 首选 idea 的细化方案与实验计划。

## 核心理念
AI 是选题加速器不是替代品——核心创新与方向拍板必须由人类主导。管线自动做调研/发散/验证/批判，产出可决策的 idea 报告，用户选定后再细化。

## 管线（分阶段）

```
调研(survey) → 发散(brainstorm) → 新颖性核验 → 批判反馈 → 方法+实验细化
```

- **Phase 0 载入研究简报**：若项目根有 `RESEARCH_BRIEF.md` 则优先读它（问题陈述、约束、已试过什么、非目标），与一行方向合并。
- **Phase 0.5 参考论文（可选）**：给了参考论文时先摘要成 `REF_PAPER_SUMMARY.md`，再以它为上下文发散。
- **Phase 1 文献调研**：arXiv + Semantic Scholar + OpenReview 检索前沿，重点看近 1-2 年顶会与作者的 Future Work。
- **Phase 2 idea 发散**：围绕 Gap 生成候选 idea。
- **Phase 3 新颖性核验**：对每个 idea 查是否已被做过（近义检索），标注去重结果。
- **Phase 4 批判反馈**：从强 reviewer 视角列致命风险。
- **Phase 5 细化**：对 top idea 产出细化提案与实验计划。

### 关键约束（默认，可覆盖）
- 试点单次 ≤ 2 GPU·小时，超出标记"需手动试点"；top idea 试点数 ≤ 3；总预算 ≤ 8 GPU·小时。
- `AUTO_PROCEED=true`：checkpoint 无响应时呈现结果后自动推进最优项。

## D:\note 适配
- 检索强制走 `semantic-scholar` MCP + `arxiv` MCP + `web-search-prime`（OpenReview 用网络搜索兜底），禁止裸 WebFetch 做发现。
- idea 报告 → `D:\note\research/01-topics/`（type: research-topic）；灵感暂存 → `research/_inbox/`。
- 首选 idea 的细化提案与实验计划 → `research/03-experiments/`（衔接本机 `experiment-plan` skill）。
- 原版 Codex MCP 跨模型评审环节可用本机 `code-reviewer`/子代理或直接主模型批判替代。
- 完成后更新 `wiki/index.md` 与 `wiki/log.md`（`research`），按仓库 Git 规则 commit + push。

## Self-evolution
如果 `LESSONS.md` 存在，先读它再执行。每次使用后，若发现调研源缺口 / 新颖性误判 / 用户纠错，写入 LESSONS.md 并通过 `/skill-evolve` 升级。
