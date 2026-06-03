---
type: experiment-log
date: {{date}}
experiment_id: EXP-YYYY-NNN
title: 实验标题
status: planned | running | completed | failed
hypothesis: 验证的假设
tags: [实验标签]
created: {{date}}
updated: {{date}}
---

## 实验目的

> 这次实验要验证什么？对应的假设是什么？

## 实验设置

| 配置项 | 值 |
|--------|-----|
| 代码版本 | commit hash 或版本号 |
| 数据集 |  |
| 模型 |  |
| Batch Size |  |
| Learning Rate |  |
| Epochs |  |
| GPU |  |
| 运行时间 |  |

### 超参数对比（如果有）

| 参数 | 基线 | 本次实验 |
|------|------|----------|
|      |      |          |

## 结果

### 主要指标

| 方法 | Metric 1 | Metric 2 | Metric 3 |
|------|----------|----------|----------|
| 基线 |          |          |          |
| 本次 |          |          |          |

### 可视化
> 插入图表或截图

### 详细数据
```json
（原始结果数据）
```

## 分析

### 结果解读
> 这个结果说明了什么？

###是否符合预期？
- [ ] 是，符合假设
- [ ] 否，需要分析原因

### 意外发现
> 有没有意料之外的结果？

## 下一步计划

- [ ] 基于这个结果，调整方案
- [ ] 扩大实验规模
- [ ] 进行消融实验
- [ ] 对比更多基线

## 反思与总结

> 这次实验学到了什么？犯了什么错误？

## 代码记录

```bash
# 关键命令或代码片段
```

## 相关实验

- [[03-experiments/EXP-YYYY-NNN-标题]]（前序实验）
- [[03-experiments/EXP-YYYY-NNN-标题]]（后续实验）
