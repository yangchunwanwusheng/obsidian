# Knowledge/ — 知识笔记目录

> 所有深度笔记按编号 L01-L23 存放。每篇笔记必须满足 [[../../../../research/08-strategy/note-quality-standards|笔记质量 10 条标准]]（≥ 8/10 合格）。

## 命名约定

- 文件名格式：`NN-topic-name.md`（如 `01-python-advanced.md`）
- N 是两位数编号（01-23）
- topic 是英文短描述
- 文件名稳定（不带日期），便于 wikilink

## 笔记结构（v2.0 标准）

每篇笔记必须包含：

```markdown
---
type: lesson
topic: <主题>
week: <W1-W8>
rating: <自评分，如 10/10>
created: YYYY-MM-DD
updated: YYYY-MM-DD
prerequisites: [相关概念 wikilink]
tags: [主题标签]
---

# LNN 笔记标题

> 一句话摘要

## 目录
## 1. <核心概念 1>
## 2. <核心概念 2>
...
## N. 质量自评（10 分制评分卡）

## References（≥ 20 条，全部可点击）
```

## 当前笔记列表

见 [[../_system/registry|../_system/registry.md]] Knowledge 表。

## 添加新笔记流程

1. 决定 L 编号（看 registry 下一个空位）
2. 复制 `_templates/lesson.md` 模板
3. 写笔记内容（满足 10 条标准）
4. 跑 quality-control-checker 自评
5. 在 `_system/registry.md` 追加 / 更新状态
6. commit + push