---
type: log
tags: [wiki, log, 操作日志]
created: 2026-07-05
updated: 2026-07-05
---

# Wiki 操作日志

> 按时间倒序记录所有 wiki/ 下的新增 / 更新 / 删除 / 归档操作。

---

## [2026-07-05] cleanup | Wiki 骨架重建（v2.0）

- **触发**：笔记质量升级（Phase 6）
- **操作**：
  - 清空 wiki/concepts/, wiki/entities/, wiki/summaries/, wiki/comparisons/, wiki/overviews/ 内部旧内容
  - 删除旧 wiki/index.md (50KB) 和 wiki/log.md (96KB)
  - 删除 wiki/entities.md（与 wiki/entities/ 重复）
  - 重建骨架目录 + 新的 index.md 和 log.md（含 Dataview 自动统计块）
- **原因**：旧 wiki 内容生成于 v1.0 质量标准之前，不符合 10 条标准；按用户授权全清
- **下一步**：用 quality-control-checker skill 重新生成符合 v2.0 标准的概念/实体/对比页