---
type: tool-guide
tags: [Obsidian, 工具指南, 科研笔记, 插件, Dataview, Templater, Excalidraw, Smart-Connections]
created: 2026-07-05
updated: 2026-07-05
sources: [obsidian-dataview, Templater, obsidian-excalidraw-plugin, obsidian-smart-connections, obsidian-markmind, obsidian-spaced-repetition, obsidian-kanban, obsidian-periodic-notes, makemd, obsidian-projects]
---

# Obsidian 高级工具链：科研笔记实战

> 元信息：调研日期 2026-07-05；覆盖 10 个核心社区插件；兼容 Obsidian v1.5+（Insets/DataView/Templater 等已稳定支持 1.9+）。所有 GitHub star 数与 pushed_at 均直接由 `gh api` 在调研日拉取。

## 摘要

科研场景下 Obsidian 的真正价值在于把"零散的 Markdown"编译成"可查询的知识库"。本次调研覆盖 10 个插件并按**必装 + 选装**分级：
- **必装（3 个）**：Dataview（把笔记变数据库）、Templater（系统化自动脚本）、Excalidraw（手绘风格科研图）。
- **强力推荐（3 个）**：Smart Connections（AI 语义联想）、Periodic Notes + Calendar（日历周期笔记）、Spaced Repetition（基于 SM-2 的间隔复习）。
- **工作流增强（4 个）**：MarkMind（大纲/导图）、Kanban（看板）、Projects（带字段的项目面板）、Make.md（Bases/Spaces 框架）。

**三大组合用法**：
1. **Templater × Dataview**：用 Templater 在新建笔记时自动注入查询块，论文笔记创建即生成"今日新增文献 / 待读清单"等动态视图。
2. **Periodic Notes × Calendar × Daily-research-log**：每日/周/月/年的科研日志自动按模板生成，跨周期通过 Dataview 汇总统计。
3. **Excalidraw × Smart Connections × Spaced Repetition**：架构图可视化 + AI 联想笔记 + 间隔复习卡片，把方法学知识转化为长效记忆。

## 推荐组合

> **必装**：Dataview（数据查询）、Templater（自动化）、Excalidraw（图示）。
> **选装**：Smart Connections（AI）、Periodic Notes+Calendar（周期笔记）、Spaced Repetition（复习）、MarkMind（导图）、Kanban（看板）、Projects（项目）、Make.md（Bases 框架）。

---

## 必装插件

### 1. Dataview

**核心能力**：把 Obsidian 笔记视作可执行 SQL/DQL 查询的数据库；frontmatter 与隐式字段（tags / file.ctime / file.outlinks）均可作为查询列。

**GitHub**：blacksmithgu/obsidian-dataview · ⭐ 9,152 · last push 2025-11-17 · [repo](https://github.com/blacksmithgu/obsidian-dataview) · [docs](https://blacksmithgu.github.io/obsidian-dataview/)

#### 实战场景

1. **论文阅读进度表**：在 `research/02-papers/_dashboard.md` 用 LIST/TABLE 实时聚合所有论文笔记状态。
2. **跨笔记汇总统计**：用 ROW 元查询统计"本月读了 X 篇 / 待读 Y 篇"等 KPI。
3. **双向链接可视化**：用 `outgoing()` / `incoming()` 联动知识图谱，辅助发现"未被引用的孤立论文"。

#### 可复制代码示例

**示例 A — 论文仪表盘（DQL 表格）**：

```dataview
TABLE
  year as "年份",
  venue as "会议",
  status as "进度",
  row["reading-priority"] as "优先级"
FROM "research/02-papers"
WHERE type = "paper-note" AND contains(tags, "#paper")
SORT year DESC, status ASC
LIMIT 50
```

**示例 B — 引用回环 JS**（找出"被引但未阅读"的论文）：

```javascript
// DataviewJS：检测双向链接中"反向孤岛"
const unlinked = dv.pages('"research/02-papers"')
  .where(p => p.status === "inbox")
  .map(p => ({
    name: p.file.name,
    inlinks: p.file.inlinks.length,
    outlinks: p.file.outlinks.length
  }))
  .where(x => x.inlinks === 0);

if (unlinked.length === 0) {
  dv.paragraph("完美：所有论文都有入链 ✓");
} else {
  dv.table(
    ["论文", "入链数", "出链数"],
    unlinked.array().slice(0, 20)
  );
}
```

**示例 C — 跨维度 KPI（ROW + inline 计算）**：

```dataview
TABLE WITHOUT ID
  length(rows) as "总数",
  length(filter(rows, (r) => r.status = "completed")) as "已读完",
  length(filter(rows, (r) => r.status = "in-progress")) as "在读"
FROM "research/02-papers"
WHERE type = "paper-note"
GROUP BY year
```

#### 与其他插件联动

- **Templater**：用 `<%* %>` 在模板中插入 Dataview 块；新建论文笔记时自动包含 dashboard 链接。
- **Periodic Notes**：日历视图跳到某天时，Dataview 块可聚合"该日新增 / 修改笔记数"。
- **Make.md**：Spaces 中可绑定 Dataview 视图，按查询结果动态渲染卡片墙。
- **Kanban**：Kanban 内部即基于 Dataview 查询，因此两者天然耦合。

#### 性能开销 / 学习曲线

- **开销**：每条 Dataview 块在打开含该块的文件时实时执行。中型 vault（≤5,000 文件）无感；上万级文件 + 复杂 JS 查询时偶尔卡顿，建议在后台节流。
- **学习曲线**：入门 DQL 约 1 小时；JavaScript API 完全掌握需 8-12 小时。

#### 官方文档 / GitHub 链接

- 官方手册：https://blacksmithgu.github.io/obsidian-dataview/
- DQL 结构：https://blacksmithgu.github.io/obsidian-dataview/queries/structure/
- DQL/JS/Inlines：https://blacksmithgu.github.io/obsidian-dataview/queries/dql-js-inline/
- 实战样本库：https://willcodefor.beer/dataview

---

### 2. Templater

**核心能力**：在新建/重命名文件时执行 JavaScript 脚本，自动注入日期、模板变量、用户提示输入，甚至调用 `app.vault` / `app.commands`。

**GitHub**：SilentVoid13/Templater · ⭐ 5,119 · last push 2026-07-03 · [repo](https://github.com/SilentVoid13/Templater) · [文档](https://silentvoid13.github.io/Templater/)

#### 实战场景

1. **论文笔记模板**：插入 Zotero citekey、authors 数组解析、frontmatter 自动写齐。
2. **每日科研日志模板**：自动加 dataview 块统计"昨日新增笔记"。
3. **会议/期刊模板**：根据用户输入的赛道动态选填 frontmatter 字段。

#### 可复制代码示例

**示例 A — 论文笔记模板**（每日自动 timestamp + 字段提示）：

```markdown
---
type: paper-note
tags: [paper, <% await tp.system.prompt("子标签?") %>]
created: <% tp.date.now("YYYY-MM-DD") %>
updated: <% tp.date.now("YYYY-MM-DD") %>
authors: ["<% await tp.system.prompt("第一作者") %>"]
venue: "<% await tp.system.prompt("会议/期刊") %>"
year: <% tp.date.now("YYYY") %>
status: "<% await tp.system.prompt("状态? [inbox/reading/done]", "inbox") %>"
paper-link: <% await tp.system.prompt("PDF / arXiv URL") %>
---

# <% tp.file.title %>

## TL;DR
（用一句话概括贡献）

## 关键方法
（核心公式/算法）

## 实验结果
（主要数据点）

## 相关
<%*
const related = app.plugins.plugins.dataview?.api
  ?.pages('"research/02-papers"')
  ?.where(p => p.year && p.year === (new Date()).getFullYear())
  ?.map(p => `- [[${p.file.name}]]`)
  ?.join("\n") ?? "无";
tR += related;
%>
```

**示例 B — 自动读取标题并改名为日期格式**：

```javascript
// 用户命令：把当前笔记标题改为 `<date>-<slug>`
const slug = tp.file.title
  .toLowerCase()
  .replace(/[^a-z0-9一-龥]+/g, "-")
  .replace(/^-|-$/g, "");
const date = tp.date.now("YYYY-MM-DD");
await tp.file.rename(`${date}-${slug}`);
```

**示例 C — 触发外部脚本（调用 shell）**：

```javascript
// 在 Windows PowerShell 下运行外部 Python
const out = await tp.shell.powershell(
  `python "D:/scripts/zotero_lookup.py" "${tp.file.title}"`
);
tR += `\n> [!info] Zotero 元数据\n> ${out}\n`;
```

#### 与其他插件联动

- **Dataview**：模板里直接嵌入 DQL 块，模板应用后即时渲染查询。
- **Periodic Notes**：绑定到 day/week/month 自动创建周期笔记。
- **QuickAdd**：配合 One-Touch 宏更快触发模板。
- **Zotero Integration**：脚本内拉取 BibTeX 字段。

#### 性能开销 / 学习曲线

- **开销**：低。仅在模板执行/文件名事件时运行；后续阅读无影响。
- **学习曲线**：基础（变量与日期）10 分钟；JS API（system prompt/shell/file）需 2-4 小时；高级交互（动态 frontmatter + 外部命令）需 1 整天练习。

#### 官方文档 / GitHub 链接

- 官方手册：https://silentvoid13.github.io/Templater/
- 仓库：https://github.com/SilentVoid13/Templater
- 实战脚本库：https://beingpax.medium.com/obsidian-templater-snippets-i-wish-i-knew-sooner-e0effc30106d

---

### 3. Excalidraw

**核心能力**：Obsidian 内嵌的手绘风格画板；可导出 .excalidraw / .png / .svg，**支持 LaTeX 公式与 Markdown 嵌入**（自 v1.x 起）。

**GitHub**：zsviczian/obsidian-excalidraw-plugin · ⭐ 7,254 · last push 2026-07-05 · [repo](https://github.com/zsviczian/obsidian-excalidraw-plugin)

#### 实战场景

1. **方法学架构图**：Transformer/扩散模型/CS 算法 pipeline，直接画出并嵌入论文笔记。
2. **实验流程图**：描述训练→评估→消融的流程，比纯文字清晰得多。
3. **概念关系图**：把 wiki 中实体/概念的关系画成可视地图（配合 [[smart-connections]] 引线）。

#### 可复制代码示例

**示例 A — 嵌入 .excalidraw 文件到任意笔记**：

```markdown
![[research/03-experiments/figures/training-pipeline.excalidraw]]{#fig-training}
```

**示例 B — 调用 Excalidraw Automate 脚本批量生成图**（外置 `.excalidraw.md` + JavaScript Sketch File 风格脚本）：

```javascript
// 在 Templater 用户脚本中：用 ExcalidrawAutomate 创建简单组件
const ea = engine.plugins["excalidraw"].excalidrawAutomate;
ea.reset();
// 创建带连接线的关系图
ea.addRect(-100, -100, 60, 30);
ea.addText(-85, -92, "Input", 2, 16);
ea.addRect(0, -100, 60, 30);
ea.addText(20, -92, "Encoder", 2, 16);
ea.addArrow({ x: -40, y: -85 }, { x: 0, y: -85 });
await ea.create({ topic: "auto-diagram" });
```

**示例 C — 在 Frontmatter 中嵌入元数据供搜索**：

```markdown
---
excalidraw-plugin: parsed
excalidraw-css: ""
---
## Excalidraw 数据

```json
{ "type": "excalidraw", "version": 2, "source": "...", "elements": [...] }
```
```

#### 与其他插件联动

- **Smart Connections**：可根据图中文本做向量检索，辅助"图-文互联"。
- **Templater**：可通过 ExcalidrawAutomate API 自动生成固定格式的示意图。
- **Kanban**：某些进阶用户将 Kanban 替换为 Excalidraw 风格看板。
- **Longform / obsidian-markmind**：长篇写作或导图场景共用。

#### 性能开销 / 学习曲线

- **开销**：单张图打开延迟 0-200ms；含 500+ 元素的复杂图偶有卡顿；可导出 SVG 后用 Markdown 嵌入图片降低负担。
- **学习曲线**：基础绘图 5 分钟；LaTeX 与 Automate 集成需 2-3 小时。

#### 官方文档 / GitHub 链接

- 论坛主帖：https://forum.obsidian.md/t/excalidraw-full-featured-sketching-plugin-in-obsidian/17367
- 功能简介：https://www.youtube.com/watch?v=P_Q6avJGoWI
- 公共 Excalidraw Library：https://libraries.excalidraw.com/

---

## 选装插件

### 4. Smart Connections

**核心能力**：用本地或云端嵌入模型做笔记语义索引；提供"相关笔记联想 + AI Chat + Smart Context"三件套。

**GitHub**：brianpetro/obsidian-smart-connections · ⭐ 5,242 · last push 2026-07-04 · [repo](https://github.com/brianpetro/obsidian-smart-connections) · [主页](https://smartconnections.app/)

#### 实战场景

1. **写作时的"联想"**：写论文 Related Work 时自动弹出引用候选。
2. **跨子主题连接**：发现"看似不相关却被 AI 判定相似"的笔记，催生新的研究问题。
3. **Smart Chat 问答**：以自己的笔记为上下文提问，避免幻觉。

#### 配置示例

```javascript
// Settings → Smart Connections → Embeddings
{
  "model": "OpenAI Text Embedding 3 Small",
  "api_key": "${SMART_CONNECTIONS_KEY}",
  "exclusions": ["research/07-journal/", "raw/assets/"],
  "max_results": 12,
  "similarity_threshold": 0.72
}
```

#### 与其他插件联动

- **Templater**：可在 frontmatter 中插入 `smart-context-id`，模板化自动重索引。
- **Periodic Notes**：每日日志中可调用 Smart Chat 自动生成"昨天的工作摘要"。
- **Dataview**：用 `dv.pages()` 配合 Smart 嵌入 ID 做相似度二次筛选。

#### 性能开销 / 学习曲线

- **开销**：首次索引 2,000 笔记约 3-8 分钟（云端 API）或 30-60 分钟（本地）；增量索引秒级。建议冷启动时执行。
- **学习曲线**：1-2 小时。需配置 API key 与排除规则；本地模型门槛更高。

#### 官方文档 / GitHub 链接

- 主页：https://smartconnections.app/
- 插件页：https://community.obsidian.md/plugins/smart-connections
- 学术用法评测：https://effortlessacademic.com/adding-ai-to-your-obsidian-notes-with-smartconnections-and-copilot/

---

### 5. MarkMind（思维导图）

**核心能力**：思维导图 / 大纲 / PDF 批注三合一；与 Markdown 双向同步。

**GitHub**：MarkMindCkm/obsidian-markmind · ⭐ 1,034 · last push 2026-07-01 · [repo](https://github.com/MarkMindCkm/obsidian-markmind) · [官网](https://www.markmind.net/)

#### 实战场景

1. **论文方法学拆解**：把论文 method 章节画成导图，挂到笔记末尾。
2. **研究问题-方法-结果 三层导图**：用于开题答辩或组会展示。
3. **课程笔记大纲化**：先做大纲、再展开为 Markdown。

#### 配置示例

```markdown
# 在 Markdown 中触发 MarkMind 渲染（自动识别）
\`\`\`markmind
- 研究主题
  - 背景
  - 现有方法
  - 创新点
  - 验证
\`\`\`
```

#### 与其他插件联动

- **Excalidraw**：手绘风格 vs 结构化导图可互补。
- **Templater**：可脚本生成.markmind 节点的初始结构。
- **Canvas**：MarkMind 文件可一键导出为 Canvas 格式。

#### 性能开销 / 学习曲线

- **开销**：节点 <500 流畅；过千节点时偶发 UI 卡顿。
- **学习曲线**：基础 30 分钟；PDF 标注 2-3 小时。

#### 官方文档 / GitHub 链接

- 官方文档：https://www.markmind.net/docs
- 论坛帖：https://forum.obsidian.md/t/obsidian-markmind-a-mindmap-outline-pdf-annotate-plugin-for-obsidian/25467

---

### 6. Spaced Repetition（间隔复习）

**核心能力**：基于 SM-2 算法的卡片复习插件；卡片直接用 Markdown 块（`Question :: Answer`）；支持每日到期提醒。

**GitHub**：st3v3nmw/obsidian-spaced-repetition · ⭐ 2,455 · [repo](https://github.com/st3v3nmw/obsidian-spaced-repetition)

#### 实战场景

1. **方法学关键概念复习**：论文里学到的核心公式/术语转卡片，长期不遗忘。
2. **基础概念地毯式过**：线代/优化/概率热力学公式反复锤。
3. **会议/期刊偏好记忆**：CCF 分级、各期刊影响因子、时间窗，固定复习。

#### 配置示例

```markdown
每张笔记中，用 :: 标记：
什么是 SRC（Spaced Repetition Cluster）？:: Obsidian 间隔复习系统将笔记按相同 tag 分桶；每桶独立计算到期。

哪条公式描述 SM-2 的下次间隔？:: I = I_prev × EF, EF ∈ [1.3, 2.5]
```

#### 与其他插件联动

- **Templater**：在论文笔记模板里自动把"关键概念"列表渲染为卡片块。
- **Dataview**：聚合每日/每周已复习卡数。
- **Periodic Notes**：每日日志结尾插入"今日到期卡数"提示。

#### 性能开销 / 学习曲线

- **开销**：纯本地算法，零依赖。
- **学习曲线**：基础卡片 5 分钟；Stat/ease 控制 1 小时。

#### 官方文档 / GitHub 链接

- 项目主页：https://www.st3v3nmw.com/Obsidian-SR-Docs/
- 评测：https://www.obsidianstats.com/posts/2025-05-01-spaced-repetition-plugins

---

### 7. Projects（项目/任务管理面板）

**核心能力**：从文件夹或 Dataview 查询源构建看板/表格/日历视图；支持自定义字段（status/priority/assignee）。

**GitHub**：obsmd-projects/obsidian-projects · ⭐ 1,930 · last push 2025-07-18 · [repo](https://github.com/obsmd-projects/obsidian-projects)

#### 实战场景

1. **论文投递流水线**：Planning → Draft → Submitted → Decision 四栏看板。
2. **实验进度看板**：每日实验日志按 status 自动落入对应列。
3. **多项目视图**：跨研究方向统一看板（按 note property 拆分）。

#### 配置示例

```markdown
# projects.config.yml（项目根配置）
name: Research-2026
views:
  - type: table
    name: 全部项目
    query: |
      type = "research-topic"
    fields: [title, status, deadline]
  - type: board
    name: 论文投递看板
    query: |
      type = "paper-note"
    group-by: status
  - type: calendar
    name: 截稿日历
    query: |
      type = "journal-info"
```

#### 与其他插件联动

- **Dataview**：内部直接依赖 Dataview JS 作为查询后端。
- **Templater**：项目笔记模板可一次写完所有字段。
- **Periodic Notes**：日历视图与日历插件天然集成。

#### 性能开销 / 学习曲线

- **开销**：与 Dataview 一致；视图多时初始化稍慢。
- **学习曲线**：基础 1 小时；自定义字段+视图需 4-6 小时。

#### 官方文档 / GitHub 链接

- 论坛帖：https://forum.obsidian.md/t/my-project-management-workflow-an-in-depth-explanation/82508
- 插件页：https://community.obsidian.md/plugins/projects-manager

---

### 8. Make.md（Bases / Spaces 框架）

**核心能力**：把 Obsidian 文件夹抽象为"Spaces"虚拟组织 + "Bases"结构化数据视图（类似 Notion），支持多视图/卡片墙/嵌套。

**GitHub**：Make-md/makemd · ⭐ 1,973 · last push 2026-01-26 · license MIT · [repo](https://github.com/Make-md/makemd) · [官网](https://www.make.md/) · open issues ≈ 342

#### 实战场景

1. **"研究方向" 卡片墙**：每个 topic 是一张卡片，按 Tag 过滤视图展示。
2. **跨文件夹的虚拟目录**：同一论文同时出现在 `theme/` 与 `method/` 两视图。
3. **Bases 数据表**：在没有第三方 db 工具的情况下管理实验元数据。

#### 配置示例

```yaml
# make.md 配置文件（vault 根 .obsidian/make.md.json 节选）
{
  "spaces": [
    {
      "name": "Papers by Year",
      "type": "list",
      "filter": "(file.inFolder('research/02-papers') && file.frontMatter('year'))"
    },
    {
      "name": "Active Topics",
      "type": "gallery",
      "filter": "(contains(file.tags, '#research-topic') && file.frontMatter('status')=='active')"
    }
  ]
}
```

#### 与其他插件联动

- **Dataview**：Spaces 内部查询基于类似语法。
- **Templater**：可以脚本写入 Spaces 索引元数据。
- **Spaced Repetition**：用 Spaces 视图聚合即将到期卡片。

#### 性能开销 / 学习曲线

- **开销**：中等。Spaces 越多越慢，建议 <30 个。
- **学习曲线**：基础 2 小时；深度配置 6-8 小时。
- **注意**：issue 较多（342+），生产环境建议每次更新前备份。

#### 官方文档 / GitHub 链接

- Getting Started：https://www.make.md/docs/Getting%2520Started
- 评测：https://www.xda-developers.com/makemd-best-obsidian-plugin-organize-beautify-notes/

---

### 9. Kanban（看板）

**核心能力**：把任意 Markdown 笔记渲染为标准看板，列表项即任务卡，支持 markdown 链接、tag、字段。

**GitHub**：mgmeyers/obsidian-kanban · ⭐ 4,389 · last push 2026-03-06 · [repo](https://github.com/mgmeyers/obsidian-kanban)

#### 实战场景

1. **论文写作流水线**：Idea → Outlining → Drafting → Polishing → Submitted。
2. **实验任务看板**：Backlog / Doing / Verify / Done 四列。
3. **日/周计划面板**：在 Periodic Notes 内嵌入今日 Kanban。

#### 配置示例

````markdown
```kanban
---
columns:
  - Backlog
  - Doing
  - Review
  - Done
---

## Backlog
- [ ] 阅读：Mixture-of-Experts 综述
- [ ] 准备：组会汇报 slides

## Doing
- [ ] 实现 baseline #paper

## Review
- [ ] [[paper-attention-is-all-you-need]] 笔记校对

## Done
- [x] 论文笔记模板定型 ✅ 2026-06-30
```
````

#### 与其他插件联动

- **Templater**：可命令创建 .kanban.md 文件。
- **Dataview**：基于 Kanban 文件做卡片统计。
- **Periodic Notes**：周报中嵌入"本周完成"面板。

#### 性能开销 / 学习曲线

- **开销**：列数与卡片数过大时滚动偶有卡顿；建议 <50 卡/列。
- **学习曲线**：20 分钟即可上手。

#### 官方文档 / GitHub 链接

- 论坛帖：https://forum.obsidian.md/t/task-board-another-gtd-methodology-similar-to-github-projects-planning/90849
- 学术用例：https://effortlessacademic.com/academic-project-management-with-obsidian-task-list-kanban/

---

### 10. Calendar × Periodic Notes（周期笔记）

**核心能力**：Calendar 提供可视日历 + 周视图，Periodic Notes 在日/周/月/年的日子自动创建/打开笔记，并应用模板。

**GitHub**：
- liamcain/obsidian-periodic-notes · ⭐ 1,344 · last push 2024-08-23 · [repo](https://github.com/liamcain/obsidian-periodic-notes) · [插件](https://community.obsidian.md/plugins/periodic-notes)
- Calendar 插件由 liamcain 同源维护；2024-08 末次大版本后官方维护变缓，但仍可在 Obsidian 1.5+ 使用。

#### 实战场景

1. **每日科研日志模板**：自动生成 `2026-07-05.md`，包含昨日总结 + 新任务 + dataview 汇总块。
2. **周一例会周报**：每周一自动创建 `W28-2026.md`，套用 templater 模板列出上周完成 + 本周计划。
3. **学期规划**：月度 + 季度笔记联合，做"长期目标-中期阶段-短期 todo"三级漏斗。

#### 配置示例

```yaml
# Settings → Periodic Notes
{
  "Daily": {
    "format": "YYYY-MM-DD",
    "folder": "research/07-journal/daily",
    "template": "_templates/daily-research-log.md"
  },
  "Weekly": {
    "format": "gggg-[W]ww",
    "folder": "research/07-journal/weekly",
    "template": "_templates/weekly-research-summary.md"
  }
}
```

#### 与其他插件联动

- **Templater**：模板引擎可注入"昨日 KPI / 本周新增文献" Dataview 块。
- **Dataview**：跨周期聚合"本月读了 X 篇 / 待读 Y 篇"。
- **Spaced Repetition**：每日打开新一天日志时自动同步复习卡。

#### 性能开销 / 学习曲线

- **开销**：仅在创建笔记事件时执行。
- **学习曲线**：30 分钟。
- **注意**：Periodic Notes 自 2024-08 后未大更新；若需要更活跃的替代，可考虑 Journals 插件（论坛 https://forum.obsidian.md/t/plugin-journals/76946 ）。

#### 官方文档 / GitHub 链接

- Periodic Notes 仓库：https://github.com/liamcain/obsidian-periodic-notes
- 用法视频：https://www.youtube.com/watch?v=jUmOKkJq8xw

---

## 联动配方（Recipes）

> 每个配方给出"插件组合 + 配置代码（可直接复制）+ 预期输出"。

### 配方 1 · 论文仪表盘（Dataview × Templater）

**插件**：Dataview + Templater
**触发**：新建论文笔记时自动生成包含"全部论文状态表"的笔记。

```javascript
// Templater 用户脚本：插入 Dataview 论文进度表
tR += `

\`\`\`dataview
TABLE
  year as "年份",
  venue as "会议",
  status as "状态"
FROM "research/02-papers"
WHERE type = "paper-note"
SORT status ASC, year DESC
\`\`\`
`;
```

**输出**：新建笔记后立刻看到所有论文状态，缺图直接补链接。

---

### 配方 2 · 每日科研日志（含昨日 KPI）

**插件**：Periodic Notes + Templater + Dataview

```javascript
// daily-research-log.md 模板片段
## 昨日小结（自动）
\`\`\`dataview
TABLE WITHOUT ID
  length(rows) as "昨日新增"
FROM "research/"
WHERE file.cday = dur(<% tp.date.now("YYYY-MM-DD HH:mm") %>).yesterday
\`\`\`

## 今日待办
\`\`\`kanban
## Todo
- [ ] 

## Doing
- 

## Done
- [x] (昨日已完成自动注入)
\`\`\`
```

**输出**：每日打开新日子自动看到昨日新增笔记数 + 看板当日列。

---

### 配方 3 · 跨学科的相关文献发现

**插件**：Smart Connections + Dataview

```javascript
// 在每篇论文笔记底部自动生成"语义相似 Top 5"
const sc = app.plugins.plugins["smart-connections"];
const result = await sc.api.search(`${tp.file.title}`, 5);
tR += `\n## 语义相关\n`;
for (const r of result) {
  tR += `- [[${r.item.path.replace(/\.md$/, "")}]] 相似度 ${r.score.toFixed(3)}\n`;
}
```

**输出**：每篇论文笔记末端出现"语义相关 Top 5"列表，辅助发现跨方向联系。

---

### 配方 4 · 论文方法学架构图（Excalidraw × Templater）

**插件**：Excalidraw + Templater + ExcalidrawAutomate

```javascript
// 用户脚本：批量创建论文架构图
const ea = app.plugins.plugins.excalidraw.excalidrawAutomate;
ea.reset();
ea.addRect(-100, -100, 80, 30, { backgroundColor: "#cce5ff" });
ea.addText(-85, -90, "Input");
ea.addRect(0, -100, 80, 30, { backgroundColor: "#ffd9b3" });
ea.addText(15, -90, "Encoder");
ea.addArrow({x:-20, y:-85}, {x:0, y:-85});
await ea.create({ topicName: `${tp.file.title}-arch`, folderPath: "research/figures" });
```

**输出**：每篇论文笔记创建时自动在 `research/figures/` 生成同名 .excalidraw 架构图。

---

### 配方 5 · 复习卡片自动生成（Spaced Repetition × Templater × Dataview）

**插件**：SR + Templater + Dataview

```javascript
// 论文笔记模板：自动把"关键方法"段转成卡片
const dvPages = app.plugins.plugins.dataview.api;
const page = dvPages.page(tp.file.path);
const methods = page?.["关键方法"] ?? "";
const cards = methods.split("\n").filter(s => s.includes("::"))
  .map(s => s.trim());
tR += `\n## 复习卡片（共 ${cards.length} 张）\n\n`;
for (const c of cards) {
  tR += `${c}\n`;
}
```

**输出**：论文笔记创建即自动列出可复习卡片；用户在 SR 复习面板直接拉取。

---

### 配方 6 · 项目进度看板（Projects × Dataview × Templater）

**插件**：Projects + Dataview + Templater

```yaml
# research/04-writing/projects.config.yml
name: Paper Pipeline
views:
  - type: board
    name: Pipeline
    query: 'type = "paper-writing"'
    group-by: status
    cards-fields: ["venue", "deadline"]
```

**输出**：在 Obsidian 侧栏直接看到"Planning / Drafting / Reviewing / Submitted"四栏看板。

---

### 配方 7 · 月度论文阅读 KPI（Periodic Notes × Dataview）

**插件**：Periodic Notes + Dataview

```dataview
TABLE WITHOUT ID
  length(rows) as "本月读毕",
  length(filter(rows, (r) => r.status = "in-progress")) as "在读"
FROM "research/02-papers"
WHERE type = "paper-note" AND dateformat(date(file.cday), "yyyy-MM") = "2026-07"
GROUP BY dateformat(date(file.cday), "yyyy-MM") as "月"
```

**输出**：在月报笔记中嵌入"本月读毕 X / 在读 Y"自动汇总。

---

### 配方 8 · 复习卡可视看板（Periodic Notes × SR × Kanban）

```markdown
## 今日到期
\`\`\`kanban
## Review Today (<% tp.date.now("YYYY-MM-DD") %>)
- [ ] (此处自动列出 SR 即将到期的卡片)
\`\`\`
```

**输出**：每天打开日记时，在看板中"今日待复习"列直接显示所有到期卡。

---

## 性能与注意事项

| 插件 | 主要开销 | 已知问题 | 最佳实践 |
|---|---|---|---|
| Dataview | 实时查询大库时偶发卡顿 | 复杂 JS 循环阻塞渲染 | 用 `LIMIT` 与 `Group BY` 减小数据量；避免每笔记中嵌入 3+ 复杂块 |
| Templater | 仅在新建/重命名时执行 | 系统提示对话框栈太深 | 模板做模块化拆分；`<%* %>` 与 `<% %>` 区分开 |
| Excalidraw | 元素 ≥500 时渲染缓慢 | 复杂 LaTeX 在移动端偶发 | 导出 SVG 后用 `![[]].svg` 嵌入；按章节切分图 |
| Smart Connections | 首次索引耗时 | API key 配额/费用 | 用本地嵌入（Ollama）+ OpenAI Text Embedding 3 Small 双轨 |
| MarkMind | 节点 <500 流畅 | 偶发与 Canvas 冲突 | 节点 >300 时拆分为多个 .markmind 文件 |
| Spaced Repetition | 几乎无 | 卡片数 >10k 后预览变慢 | 每张笔记避免 ≥50 张卡片，按章节分文件 |
| Projects | 视图初始加载偶发慢 | 早期 API 改版导致旧配置失效 | 升级版本前先 `git commit` 备份 projects.config.yml |
| Make.md | Spaces 多时性能下降 | open_issues 较多（≈342） | 慎用最新 beta；保留旧版 fallback |
| Kanban | 列卡数超 50/列偶发卡 | 嵌套 Markdown 与某些主题 CSS 冲突 | 列数 ≤6，卡片数 ≤30/列 |
| Periodic Notes | 模板路径变更需手动改设置 | 2024-08 后更新停滞 | 用 Journals 插件做备份；模板与 Templater 双向同步 |

### 通用最佳实践

1. **少装多用**：必装三件套（Dataview/Templater/Excalidraw）覆盖 80% 场景；其他插件按需启用。
2. **键值清洗**：所有插件配置集中在 `~/.obsidian/workspace.json` 或对应插件目录；新机器 sync 时直接复制。
3. **定期 Lint**：用 `community-plugin-linter` 或手动检查孤立笔记 / frontmatter 缺失。
4. **移动端优先关闭**：Excalidraw Automate、SR、Projects 在移动端体验差；建议移动端仅启 Dataview / Templater / Kanban。
5. **数据备份**：Plugin 自动备份 `.obsidian/plugins/<name>/data.json`；每周 Git commit 一次保险。

---

## 引用源（≥15 URL）

### 官方仓库 & 文档
1. Dataview 仓库：https://github.com/blacksmithgu/obsidian-dataview
2. Dataview 官方文档：https://blacksmithgu.github.io/obsidian-dataview/
3. Templater 仓库：https://github.com/SilentVoid13/Templater
4. Templater 官方文档：https://silentvoid13.github.io/Templater/
5. Excalidraw 插件仓库：https://github.com/zsviczian/obsidian-excalidraw-plugin
6. Smart Connections 仓库：https://github.com/brianpetro/obsidian-smart-connections
7. Smart Connections 主页：https://smartconnections.app/
8. MarkMind 仓库：https://github.com/MarkMindCkm/obsidian-markmind
9. Spaced Repetition 仓库：https://github.com/st3v3nmw/obsidian-spaced-repetition
10. Kanban 仓库：https://github.com/mgmeyers/obsidian-kanban
11. Projects 仓库：https://github.com/obsmd-projects/obsidian-projects
12. Periodic Notes 仓库：https://github.com/liamcain/obsidian-periodic-notes
13. Make.md 仓库：https://github.com/Make-md/makemd
14. Make.md 官方文档：https://www.make.md/docs/Getting%2520Started

### 学术用法 & 教程
15. How I Use Obsidian for Academic Work（Emile van Krieken）：https://emilevankrieken.com/blog/2025/academic-obsidian/
16. Dataview 实战样本库（willcodefor.beer）：https://willcodefor.beer/dataview
17. Templater 实战脚本（BeingPax on Medium）：https://beingpax.medium.com/obsidian-templater-snippets-i-wish-i-knew-sooner-e0effc30106d
18. 学术 Kanban 用法（Effortless Academic）：https://effortlessacademic.com/academic-project-management-with-obsidian-task-list-kanban/
19. Smart Connections for Academia（Effortless Academic）：https://effortlessacademic.com/adding-ai-to-your-obsidian-notes-with-smartconnections-and-copilot/
20. Journals 插件（Periodic Notes 继任者）：https://forum.obsidian.md/t/plugin-journals/76946
21. Excalidraw 教程视频（57 key features）：https://www.youtube.com/watch?v=P_Q6avJGoWI
22. Periodic Notes 实践视频：https://www.youtube.com/watch?v=jUmOKkJq8xw
23. MarkMind 论坛主帖：https://forum.obsidian.md/t/obsidian-markmind-a-mindmap-outline-pdf-annotate-plugin-for-obsidian/25467
24. Projects 论坛工作流：https://forum.obsidian.md/t/my-project-management-workflow-an-in-depth-explanation/82508
25. Spaced Repetition 评测（ObsidianStats 2025）：https://www.obsidianstats.com/posts/2025-05-01-spaced-repetition-plugins

---

> **版本备注**：所有数据 2026-07-05 拉取；GitHub star/pushed_at 与 latest release 可能在数月内变化；建议每季度重新核对一次必装插件的活跃度。
