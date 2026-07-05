# wiki/ — 知识库目录

> 由 LLM Claudian 维护的"可复用知识资产库"。
> 任何 wiki/ 下的笔记都必须符合 [[research/08-strategy/note-quality-standards|笔记质量 10 条标准]]。

## 子目录说明

- **entities/**：实体页面（人物、组织、工具等）。例：[[entities/Transformer]]
- **concepts/**：概念页面（思想、方法论、理论等）。例：[[concepts/attention-sink]]
- **summaries/**：来源摘要页面（论文、博客、文档等的精华摘要）
- **comparisons/**：对比分析页面（方法 A vs 方法 B）
- **overviews/**：综合概述页面（领域全景图）

## 添加新页面流程

1. 决定页面类型（entity / concept / summary / comparison / overview）
2. 在对应子目录创建 `.md`，frontmatter 必填 `type / tags / created / updated`
3. 正文必须满足 10 条标准
4. 跑 quality-control-checker 自评（≥7 分）
5. 在 `wiki/index.md` 追加链接
6. 在 `wiki/log.md` 追加操作记录
7. commit + push