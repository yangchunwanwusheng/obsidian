# Daily/ — 学习日志目录

> 每天 1 篇日志，记录学了什么、卡在哪里、明天做什么。

## 命名约定

- 日报：`YYYY-MM-DD.md`（如 `2026-07-05.md`）
- 周报：`YYYY-MM-DD-week-N-summary.md`（如 `2026-07-11-week-1-summary.md`）

## 日报模板

每篇日报必须包含：

```markdown
---
type: daily-log
date: YYYY-MM-DD
week: W1
duration: <小时>
mood: <emoji + 1 句话>
---

# YYYY-MM-DD 学习日志

## 📚 今日所学
- [ ] L01 Python 进阶 §3（异步）— ✅ 完成
- [ ] L02 Transformer 架构 §2 — 🔄 进行中（卡在 Multi-Head QKV 拆分）

## 💡 关键洞察
- asyncio.gather 比顺序 await 快 4 倍（实测）
- LayerNorm 的 γ 参数初始化为 1 是关键

## 🐛 卡住的问题
- LangChain LCEL 的 RunnablePassthrough 用途不清 → 明天查文档

## 📅 明日计划
- [ ] L02 完成 Multi-Head 拆分部分
- [ ] 跑通 PyTorch Transformer 实战

## 🎯 今日 1 个行动
跑通 PyTorch 简化版 Transformer 块（明天交付）
```

## 周报模板

每周日写周报，必须包含：

```markdown
---
type: weekly-summary
week: W1
date_range: 2026-07-05 → 2026-07-11
---

# Week 1 复盘

## 🎯 完成情况
- 笔记：3/3 ✅（L01 / L02 / L03）
- 项目：0/1（W3 启动）
- 自评分：3 篇 10/10

## 📊 数据
- 学习时长：35 小时
- 实战 demo：2 个
- 卡住次数：4 次（全部解决）

## 💡 关键收获
- ...

## 🐛 主要卡点
- ...

## 📅 W2 调整
- ...

## 🎯 W2 目标
- 完成 L04 / L05 / L06
- P1 项目架构稿
```