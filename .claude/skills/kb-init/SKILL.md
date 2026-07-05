---
name: kb-init
description: 在 D:\note 下初始化一个 Research/{project-slug}/ 风格的项目子 KB(wiki/ 研究模板的子集)
---

# kb-init (vault scope)

> 与全局 `kb-init` 同名但作用于 vault 内部,创建项目子 KB,**不**改根 LLM Wiki 结构。

## 输入

- `<project-slug>`(kebab-case,用户必给)

## 步骤

1. 创建 `D:\note\Research/<slug>/` 及其子目录(`00-Hub.md` `01-Plan.md` `02-Index.md` `Sources/` `Knowledge/` `Experiments/` `Results/Reports/` `Writing/` `Daily/` `Maps/` `Archive/` `_system/`)
2. 写入 `00-Hub.md`(项目定位 + 关键链接)
3. 写入 `01-Plan.md`(初始阶段任务)
4. 写入 `_system/registry.md`(空 schema 占位)
5. 在根 `D:\note\wiki\index.md` 追加一行 `Research/<slug>` 入口
6. 在根 `D:\note\wiki\log.md` 追加 `create | project kb <slug>`

## 命名

- slug 用英文 kebab-case
- 内部 wiki 页面用中文标签 + 英文文件名

## 不做

- 不动根 `D:\note\CLAUDE.md`
- 不动根 `D:\note\wiki/` 已有内容,只追加 index.md
- 不自动生成占位笔记;空目录是允许的
