# Karpathy 的 Obsidian 工作流：用 LLM Wiki 打造本地知识库

> **核心来源**：Andrej Karpathy 于 2026 年 4 月发布的 llm-wiki Gist（5000+ Star）

## 核心理念

### 从 RAG 到"知识编译"

大多数人使用 LLM 处理文档的方式是 RAG（检索增强生成）：上传文件 → 检索相关片段 → 生成回答。问题是每次提问都要从头重新发现知识，没有积累。

Karpathy 的想法：让 LLM 逐步构建并维护一个持久的 Wiki——一个结构化的、相互链接的 Markdown 文件集合。

### 人类与 LLM 的分工

- 人类：策划信息来源、引导分析方向、提出好问题、思考意义
- LLM：总结、交叉引用、归档、维护一致性

> "Obsidian 是 IDE，LLM 是程序员，Wiki 是代码库。"

### 三层架构

1. raw/ — 原始资料（不可变）
2. wiki/ — LLM 生成的结构化知识库（可变）
3. CLAUDE.md — Schema 配置（约定和规则）

### 三大操作

1. 摄入（Ingest）：添加新资料 → LLM 读取、提取、整合到 Wiki
2. 查询（Query）：向 Wiki 提问 → LLM 搜索、综合回答
3. 维护（Lint）：定期健康检查 → 发现矛盾、孤立页面、缺失引用

### 关键文件

- index.md：内容索引，按类别列出所有页面
- log.md：操作日志，按时间记录所有操作

### Obsidian 的角色

- 本地优先的 Markdown 编辑器
- 图谱视图展示知识连接
- 反向链接支持交叉引用
- Web Clipper 快速导入网页

### 核心洞察

- 不用向量数据库——纯 Markdown + LLM 直接读取
- 知识编译优于检索——编译一次，持续更新
- 复利效应——每次操作都让 Wiki 更丰富
- 维护成本接近零——LLM 不厌烦、不忘更新

来源：https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f
