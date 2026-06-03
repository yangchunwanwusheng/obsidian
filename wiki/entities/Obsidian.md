---
type: entity
tags: [工具, 知识管理, Markdown]
created: 2026-04-08
updated: 2026-04-08
sources: [raw/Karpathy-Obsidian-LLM-Wiki.md]
---

# Obsidian

> 本地优先的 Markdown 笔记应用，是 LLM Wiki 工作流的前端界面。

## 基本信息

| 属性 | 值 |
|------|-----|
| 类型 | 工具 |
| 领域 | 个人知识管理、笔记 |
| 官网 | https://obsidian.md |
| 文件格式 | 纯 Markdown |

## 简介

Obsidian 是一款本地优先的 Markdown 笔记应用。所有笔记都以纯 Markdown 文件存储在本地磁盘上，用户完全拥有自己的数据。在 Karpathy 的 LLM Wiki 工作流中，Obsidian 作为人类阅读和浏览 Wiki 的前端界面。

## 关键特性

- **本地优先**：所有文件存储在本地，纯 Markdown，无锁定
- **图谱视图**：可视化笔记间的连接关系，展示知识结构
- **反向链接**：`[[wikilink]]` 语法创建笔记间的双向连接
- **插件生态**：Dataview、Marp、Excalidraw 等丰富插件
- **Web Clipper**：浏览器扩展，快速将网页转为 Markdown

## 在 LLM Wiki 中的角色

> "Obsidian 是 IDE，LLM 是程序员，Wiki 是代码库。"

- 人类在 Obsidian 中浏览 Wiki、检查更新
- 图谱视图用于查看 Wiki 形状（枢纽页、孤立页）
- LLM 在另一侧操作文件，人类实时看到 Obsidian 中的变化
- Web Clipper 用于快速将网页导入 raw/ 目录

## 推荐设置

1. 附件文件夹路径设为 `raw/assets/`（本地化图片）
2. 绑定"下载附件"快捷键（如 Ctrl+Shift+D）
3. 安装 Dataview 插件（查询 frontmatter）
4. 可选安装 Marp 插件（从 Wiki 内容生成演示文稿）

## 相关概念

- [[wiki/concepts/LLM Wiki]]
- [[wiki/concepts/knowledge-compilation]]

## 相关实体

- [[wiki/entities/Andrej Karpathy]]
