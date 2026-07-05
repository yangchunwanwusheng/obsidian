---
type: index
project: byte-dance-agent-prep
created: 2026-07-05
updated: 2026-07-05
tags: [index, byte-dance, agent]
---

# 笔记索引（自动统计）

> 由 Dataview 自动生成。每篇笔记创建后会自动出现在此索引中。

## 📚 全部笔记（按时间倒序）

```dataview
TABLE
  week as "周次",
  topic as "主题",
  file.mtime as "更新时间",
  rating as "自评分"
FROM "Research/byte-dance-agent-prep/Knowledge"
WHERE type = "lesson"
SORT file.mtime DESC
```

## 📊 按主题统计

```dataview
TABLE WITHOUT ID
  topic as "主题",
  length(rows) as "笔记数",
  length(filter(rows, (r) => r.rating = "10/10")) as "10/10 篇数"
FROM "Research/byte-dance-agent-prep/Knowledge"
WHERE type = "lesson"
GROUP BY topic
SORT length(rows) DESC
```

## 🎯 按周次进度

```dataview
TABLE WITHOUT ID
  week as "周次",
  length(rows) as "已完成笔记",
  length(filter(rows, (r) => r.rating = "10/10")) as "满分笔记"
FROM "Research/byte-dance-agent-prep/Knowledge"
WHERE type = "lesson"
GROUP BY week
SORT week ASC
```

## 🏆 满分笔记榜（10/10）

```dataview
LIST
FROM "Research/byte-dance-agent-prep/Knowledge"
WHERE type = "lesson" AND rating = "10/10"
SORT file.mtime DESC
```

---

## 📖 笔记访问

- Week 1：
  - [[Knowledge/01-python-advanced|L01 Python 进阶速成]]
  - [[Knowledge/02-transformer-architecture|L02 Transformer 架构深入]]
  - [[Knowledge/03-llm-training-inference|L03 LLM 训练与推理]]
- Week 2：
  - [[Knowledge/04-prompt-engineering|L04 Prompt Engineering]]
  - [[Knowledge/05-function-calling|L05 Function Calling]]

（后续笔记创建后会自动出现在此列表）