---
type: index
tags: [wiki, index, 知识库索引]
created: 2026-07-05
updated: 2026-07-05
---

# Wiki 索引（v2.0）

> **本 Wiki 由 LLM Claudian 维护。**
> 质量标准：`[[research/08-strategy/note-quality-standards|笔记质量 10 条标准]]`
> 自检流程：每次新增/更新笔记后必须跑 quality-control-checker skill（总分 ≥ 7 才合格）

## 📁 目录结构

```
wiki/
├── entities/         # 实体页面（人物、组织、工具等）
├── concepts/         # 概念页面（思想、方法论、理论等）
├── summaries/        # 来源摘要页面（论文、博客、文档）
├── comparisons/      # 对比分析页面（方法 A vs B）
└── overviews/        # 综合概述页面（领域全景图）
```

## 📊 当前内容统计

> 由 Dataview 自动生成（需安装 Dataview 插件）

```dataview
TABLE WITHOUT ID
  length(rows) as "总数",
  length(filter(rows, (r) => r.tags = "#concept")) as "概念",
  length(filter(rows, (r) => r.tags = "#entity")) as "实体",
  length(filter(rows, (r) => r.tags = "#summary")) as "摘要",
  length(filter(rows, (r) => r.tags = "#comparison")) as "对比",
  length(filter(rows, (r) => r.tags = "#overview")) as "概述"
FROM "wiki"
WHERE type = "concept" OR type = "entity" OR type = "summary" OR type = "comparison" OR type = "overview"
GROUP BY "统计"
```

## 🔍 最近更新（自动）

```dataview
TABLE
  type as "类型",
  file.mtime as "更新时间"
FROM "wiki"
SORT file.mtime DESC
LIMIT 20
```

## 📝 添加新页面

1. 在对应子目录创建 `.md` 文件
2. 复制对应模板（见 `_templates/`）
3. 跑 quality-control-checker 自评（≥7 分）
4. 在本文件末尾追加链接
5. 在 `wiki/log.md` 追加操作记录

## ⚠️ 质量要求

所有 wiki/ 下的笔记必须满足 [[research/08-strategy/note-quality-standards|笔记质量 10 条标准]]（每条 1 分）：

- **9-10 分**：顶会水准
- **7-8 分**：付费资料水准（合格线）
- **< 7 分**：必须重写

## 📚 相关文档

- [[research/08-strategy/note-quality-standards|笔记质量 10 条标准]]
- [[research/08-strategy/note-quality-research|笔记质量升级 · 调研汇总]]
- [[research/06-tools/obsidian-advanced-toolkit|Obsidian 高级工具链]]