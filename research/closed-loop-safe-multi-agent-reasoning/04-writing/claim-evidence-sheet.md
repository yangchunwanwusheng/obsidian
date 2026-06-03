---
type: planning
topic: claim-evidence-sheet
paper: Guardian-Controller
status: draft
---

# Claim–Evidence Sheet

## Claim 1: 自适应闭环优于静态协议或静态防御

| Claim | Direct Evidence | Supporting Evidence | Invalidating Evidence | Target Experiment | Target Metric | Likely Reviewer Attack |
|-------|----------------|--------------------|-----------------------|------------------|---------------|----------------------|
| 自适应闭环优于静态协议或静态防御 | Ours vs MA-FreeText / MA-SemiStruct / G2CP-Static / GUARDIAN-Heuristic 主结果 | 不同污染场景下的稳定提升；frontier 曲线 | 仅在单一场景有效；成本显著更高 | 主实验 (Table 1, Figure 3) | Accuracy, Propagation Rate, Cost Frontier | "只是拼装，不是统一方法" |

## Claim 2: belief-guided 控制优于直接 anomaly-to-action

| Claim | Direct Evidence | Supporting Evidence | Invalidating Evidence | Target Experiment | Target Metric | Likely Reviewer Attack |
|-------|----------------|--------------------|-----------------------|------------------|---------------|----------------------|
| belief-guided 控制优于直接 anomaly-to-action | Ours vs no-belief ablation | case study；时序稳定性分析 | 两者几乎无差异 | belief 消融 (Table 3) | Recovery Success, Over-intervention Rate | "belief 没必要" |

## Claim 3: 学习策略优于 hand-crafted heuristics

| Claim | Direct Evidence | Supporting Evidence | Invalidating Evidence | Target Experiment | Target Metric | Likely Reviewer Attack |
|-------|----------------|--------------------|-----------------------|------------------|---------------|----------------------|
| 学习策略优于 heuristics | Ours vs ClosedLoop-Heuristic | 干预时机更优；误杀更少 | heuristic 已基本达到同样效果 | policy 对比实验 (Table 2) | Unsafe Pass, Over-intervention, Accuracy–Cost trade-off | "RL 只是自动调阈值" |

## Claim 4: graph-grounded 高约束模式在高风险阶段是必要的

| Claim | Direct Evidence | Supporting Evidence | Invalidating Evidence | Target Experiment | Target Metric | Likely Reviewer Attack |
|-------|----------------|--------------------|-----------------------|------------------|---------------|----------------------|
| graph-grounded 模式在高风险阶段必要 | Ours vs no-graph-mode / MA-SemiStruct | 高污染场景传播率显著下降 | 与简单结构化通信无显著差异 | protocol 消融 (Table 3) | Propagation Rate, Token Efficiency | "只是 prompt engineering" |

## Claim 5: 恢复动作集合优于仅检测不恢复

| Claim | Direct Evidence | Supporting Evidence | Invalidating Evidence | Target Experiment | Target Metric | Likely Reviewer Attack |
|-------|----------------|--------------------|-----------------------|------------------|---------------|----------------------|
| 恢复动作优于仅检测 | Ours vs detect-only baseline | 不同 failure mode 的动作分工统计 | 恢复动作几乎不被用到或收益不明显 | recovery 消融 (Table 3) | Recovery Success Rate, Final Accuracy | "检测就够了，恢复没贡献" |

## Claim 6: 轻量闭环评测协议能覆盖真实 failure family

| Claim | Direct Evidence | Supporting Evidence | Invalidating Evidence | Target Experiment | Target Metric | Likely Reviewer Attack |
|-------|----------------|--------------------|-----------------------|------------------|---------------|----------------------|
| 闭环评测覆盖真实 failure | 自然失败轨迹与三类污染注入的 failure mapping | 多任务一致性；失败案例分析 | 收益仅存在于人工注入，真实失败无效 | 自然失败补充实验 (Appendix) | Failure Coverage, Qualitative Alignment | "评测太人造，外推不足" |

## Reviewer Attack Map

| Attack | Defense Strategy | Key Evidence |
|--------|-----------------|--------------|
| "模块拼装" | 坚持主叙事：研究对象是统一闭环控制问题；给出统一映射 | Figure 1 + POMDP formulation |
| "RL 只是自动调阈值" | 设置 ClosedLoop-Heuristic baseline；多指标联合报告 | Table 1 (Ours vs ClosedLoop-Heur gap) |
| "污染注入太人造" | 三类场景对应真实 failure family；补充自然失败分析 | Appendix natural failure mapping |
| "belief 没必要" | no-belief ablation；case study 展示时序区分能力 | Table 3 + Figure 5 |
| "只是 prompt engineering" | 设置 MA-SemiStruct baseline；分离 token efficiency 和 ambiguity reduction | Table 3 (no-graph-mode ablation) |
| "恢复动作归因不清" | 控制动作空间；动作级消融；场景-动作匹配表 | Table 3 + Appendix action breakdown |

## 当前最危险的信号

1. **如果 Ours 仅比 ClosedLoop-Heuristic 略好** → "学习不必要"
2. **如果 no-belief 与完整方法几乎无差异** → "belief 不必要"
3. **如果所有收益只出现在强注入场景** → "外推不足"
4. **如果 GRAPH-GROUNDED 与 SEMI-STRUCTURED 无显著差异** → "G2CP 不是关键"
