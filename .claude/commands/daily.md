---
description: 在 research/07-journal/daily/ 创建/更新今日科研日志
argument-hint: (无参数)
---

今天日期:由 Claude 在执行时通过 `Get-Date -Format yyyy-MM-dd` 或 `date +%Y-%m-%d` 取

## 步骤

1. 检查 `D:\note\research\07-journal\daily\` 是否有今日文件,有则读取追加,无则按 `daily-research-log` 类型创建
2. 必含三维度:
   - 做了什么
   - 为什么
   - 有什么用
3. 论文材料:今天读了/听了哪几篇论文?关键数据/创新/突破用 `> [!info] 论文材料` 折叠块记录
4. 列出"明日计划" 1–3 条

## 写入

- 仅写入/追加 `D:\note\research\07-journal\daily\<today>.md`
- 不要修改其他文件
- 若用户补了"周报"相关字样,额外汇总 `research/07-journal/weekly/` 下当周所有 daily
