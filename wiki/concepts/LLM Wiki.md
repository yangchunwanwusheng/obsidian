---
type: concept
tags: [LLM, 知识管理, Obsidian, Karpathy]
created: 2026-04-08
updated: 2026-04-08
sources: [raw/Karpathy-Obsidian-LLM-Wiki.md]
---

# LLM Wiki

> 一种用 LLM 自动构建和维护个人知识库的模式，基于纯 Markdown 文件，无需向量数据库。

## 概述

LLM Wiki 是 Andrej Karpathy 于 2026 年 4 月提出的一种知识管理模式。核心思想是：不用传统的 RAG 系统（向量数据库 + 检索），而是让 LLM 直接读写一个 Markdown 文件目录，逐步构建和维护一个结构化的、相互链接的个人 Wiki。

## 核心要点

- **三层架构**：raw/（不可变原始资料）→ wiki/（LLM 维护的知识库）→ CLAUDE.md（Schema 配置）
- **三大操作**：摄入（Ingest）、查询（Query）、维护（Lint）
- **纯 Markdown**：所有知识以 .md 文件存储，便携、未来兼容、LLM 原生可读
- **Obsidian 作为前端**：图谱视图、反向链接、插件生态

## 架构说明

```
raw/          ← 原始资料（不可变，LLM 只读）
wiki/         ← LLM 生成的结构化知识（LLM 读写）
CLAUDE.md     ← Schema（定义 Wiki 结构和规则）
```

### 关键文件

- **index.md**：内容索引，按类别列出所有页面及摘要
- **log.md**：操作日志，按时间记录所有摄入、查询、维护操作

### 页面类型

| 类型 | 说明 |
|------|------|
| concept | 概念页面——思想、方法论、理论 |
| entity | 实体页面——人物、组织、工具 |
| summary | 来源摘要——对 raw/ 中文件的总结 |
| comparison | 对比分析——多来源的交叉对比 |
| overview | 综合概述——某主题的全景总结 |

## 与传统 RAG 的区别

| 维度 | RAG | LLM Wiki |
|------|-----|----------|
| 知识状态 | 无状态，每次重新检索 | 有状态，知识已编译 |
| 基础设施 | 向量数据库、嵌入模型 | Markdown 文件目录 |
| 交叉引用 | 运行时发现 | 已预先建立 |
| 矛盾检测 | 无 | 已标记和处理 |
| 维护成本 | 高（需要重新索引） | 近零（LLM 自动维护） |

## 相关概念

- [[wiki/concepts/knowledge-compilation]]
- [[wiki/concepts/RAG]]
- [[wiki/concepts/Schema]]

## 相关实体

- [[wiki/entities/Andrej Karpathy]]
- [[wiki/entities/Obsidian]]
