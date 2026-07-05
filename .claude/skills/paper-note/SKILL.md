---
name: paper-note
description: 把一篇论文(arXiv ID / URL / 本地 PDF)转成 vault 内的双语阅读笔记,写入 `D:\note\raw\lessons\<主题>/` 下
---

# paper-note (vault scope)

> D:\note 专属版本:复用全局 `paper-note` 技能,但落点必须在 `D:\note\raw\lessons\` 下,沿用根 `CLAUDE.md` 的"论文翻译笔记规范"。

## 输入

- arXiv ID / 论文 URL / 本地 PDF 路径
- 必问:这篇论文属于哪个学习主题?若用户不指定,默认放 `D:\note\raw\lessons\_inbox/<论文名>/`

## 必须产出

每个论文子文件夹至少包含:

```
D:\note\raw\lessons\<主题>/<论文子目录>/
├── 00-essence.md         ← 论文精华(强制)
├── 00-overview.md        ← 全文概述
├── 01-introduction.md    ← 引言翻译
├── ...                   ← 其余按原文结构分篇
└── figures/              ← 可选,原图
```

### 00-essence.md 必含字段

- 会议/期刊 + 发表时间(至少到年)
- 核心问题(1–3 句)
- 方法 + 数据集
- 主要结果(相对基线/现有方法的提升)
- 局限与可改进点(若原文未说,标注"推断"并与原文明确区分)

## 翻译规范

- **原文翻译**:`## 正文翻译` 下不加任何 callout
- **译者注**:`> [!note] 译者注(非原文)`
- **理解提示**:`> [!tip] 理解提示(非原文)`
- **易混点**:`> [!warning] 易混点(非原文)`
- 原论文的表格/公式/图描述完整保留

## 关联

- 更新 `D:\note\wiki\index.md` 系列入口
- 把关键实体/概念回写到 `D:\note\wiki\entities/` `D:\note\wiki\concepts/`
- 在 `D:\note\wiki\log.md` 追加一条 `learn | ...`

## 不做

- 不在 `raw/lessons/_inbox/` 长期堆放:超过 3 篇未归主题的提醒用户整理
- 不改 `raw/` 其他子目录
