---
description: 把 raw/ 中指定文件处理为 wiki 摘要页(LLM Wiki 摄入流程)
argument-hint: <raw 文件相对路径,例如 raw/foo.md>
---

执行 LLM Wiki 的"摄入"流程,目标文件由 `$ARGUMENTS` 给出(默认如果未传则询问用户)。

## 步骤

1. 读取 `D:\note\raw\$ARGUMENTS`(若文件不存在,先 grep `D:\note\raw\` 让用户选)
2. 提取关键要点(≤3 段)
3. 在 `D:\note\wiki\summaries\` 下新建 `<源文件名>.md`,frontmatter 完整:
   - `type: summary`
   - `tags` ≥1
   - `created` / `updated` 当天
   - `sources: ["raw/<文件名>"]`
4. 涉及的实体 → 在 `D:\note\wiki\entities/` 创建/更新
5. 涉及的概念 → 在 `D:\note\wiki\concepts/` 创建/更新
6. 追加一条记录到 `D:\note\wiki\log.md`,格式见根 `CLAUDE.md`
7. 把新页面登记到 `D:\note\wiki\index.md`
8. 在底部"相关页面"中用 `[[wikilink]]` 串到现有相关页

## 规则

- 不要修改 `raw/` 下的文件(`raw/lessons/` 例外)
- 用户已有的 `idea/` `news/` 不要动
- 所有写入前先 Read 现状,避免覆盖用户已有的同名页
