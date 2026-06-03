---
type: planning
topic: figure-table-plan
paper: Guardian-Controller
status: pending
---

# Figure & Table Plan

## 设计原则

- **主文只放高信息密度图表**，不堆表格
- **图负责模式感知与 trade-off 展示**
- **表负责精确数值、benchmark 主结果、消融细节**
- **每个主图都应能独立回答一个 reviewer 质疑**

## 正文主图（5 张，全部待绘制）

### Figure 1. Framework Overview
- **承载 claim**: 本文不是模块拼装，而是统一的 closed-loop control framework
- **形式**: 四层系统架构图（Task Collaboration → Communication Control → Risk Observation → Belief-Guided Control），含反馈回路箭头
- **格式**: PDF, `\linewidth`
- **详细规范**: 见 `paper/sections/method.tex` 中的 [FIGURE SPEC]

### Figure 2. Temporal Contamination & Intervention Timeline
- **承载 claim**: 系统处理的是动态风险传播，而不是单点纠错
- **形式**: 多轮交互时间轴 + 传播路径图
- **格式**: PDF, `\linewidth`

### Figure 3. Accuracy–Safety–Cost Frontier
- **承载 claim**: Ours 改善的是 trade-off，而非仅准确率
- **形式**: 散点/气泡图，x=cost, y=accuracy, 颜色=unsafe pass rate
- **格式**: PDF, `\linewidth`, 3 子图 (FEVER/HotpotQA/MATH)
- **详细规范**: 见 `paper/sections/results.tex` 中的 [FIGURE SPEC]

### Figure 4. Action Usage Distribution
- **承载 claim**: learned controller 学到了结构化干预模式
- **形式**: 分组柱状图，左=通信模式分布，右=恢复动作频率
- **格式**: PDF, `\linewidth`

### Figure 5. Case Study
- **承载 claim**: belief-guided 闭环干预在具体轨迹上可解释
- **形式**: 四 panel 案例图：(a) 基线失败, (b) 风险信号时间线, (c) 控制器干预, (d) 最终输出对比
- **格式**: PDF, `\linewidth`

## 正文主表（3 张，全部待填充数据）

### Table 1. Main Benchmark Results
- **承载 claim**: 完整方法在主 benchmark 上整体最优
- **行**: 7 methods × 3 tasks = Accuracy + Tokens
- **数据需求**: `[FILL]` × 42 cells

### Table 2. Safety & Cost Metrics
- **承载 claim**: Ours 在安全指标上整体最优
- **行**: 6 methods × 5 safety metrics
- **数据需求**: `[FILL]` × 30 cells

### Table 3. Ablation Study
- **承载 claim**: 各组件各有必要
- **行**: 1 full + 5 ablated + 5 delta rows
- **数据需求**: `[FILL]` × 30 cells

## 附录图表（4 项，待填充）

### Figure A1. Belief vs Anomaly Over Time
### Figure A2. Per-Task Frontier Plots
### Table A1. Per-Action Success Breakdown
### Table A2. Natural Failure Distribution

## Chart–Claim Binding

| Artifact | Primary Claim |
|----------|--------------|
| Figure 1 | 统一闭环框架，非拼装 |
| Figure 2 | 错误传播是动态风险问题 |
| Figure 3 | Ours 优化 accuracy–safety–cost trade-off |
| Figure 4 | Controller 学到结构化干预 |
| Figure 5 | 方法在具体轨迹上可解释 |
| Table 1 | 主 benchmark 整体有效 |
| Table 2 | 安全—成本联合改善 |
| Table 3 | 各组件各有必要 |

## 最小可发表核心包

1. **Table 1** 主结果表
2. **Figure 3** frontier 图
3. **Table 3** 消融表
4. **Figure 5** 案例研究图
