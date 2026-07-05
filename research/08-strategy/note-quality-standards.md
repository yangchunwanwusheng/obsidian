---
type: strategy-note
tags: [笔记质量标准, 评分卡, 自检清单, 科研笔记, 顶级标准]
created: 2026-07-05
updated: 2026-07-05
sources:
  - https://notes.andymatuschak.org/z5E5QawiXCMbtNtupvxeoEX
  - https://notes.andymatuschak.org/zNUaiGAXp21eorsER1Jm9yU
  - https://zettelkasten.de/introduction/
  - https://fortelabs.com/blog/basboverview/
  - https://lilianweng.github.io/posts/2023-06-23-agent/
  - https://jalammar.github.io/illustrated-transformer/
  - D:\note\research\08-strategy\note-quality-research-methodology.md
  - D:\note\research\08-strategy\note-quality-research-blogs.md
  - D:\note\research\06-tools\obsidian-advanced-toolkit.md
  - D:\note\research\08-strategy\note-quality-research-skills.md
---

# 笔记质量 10 条标准 + 评分卡（v1.0）

> **版本**：v1.0（2026-07-05 立）
> **范围**：D:\note vault 内所有由 LLM 生成的笔记（学习笔记、论文笔记、调研笔记、Wiki 页面）
> **目标**：通过 10 条可验证标准 + 自检流程，把笔记质量稳定推到 **吊打付费资料 / 顶会论文精读水准**（评分 8-10/10）

---

## 摘要

调研 Lilian Weng、Jay Alammar、Chris Olah、Karpathy、Distill.pub 等顶级范例，结合 Zettelkasten / Evergreen Notes / CODE / Atomic Notes / SQ3R 五大方法论，并消化 kepano/obsidian-skills、K-Dense-AI/claude-scientific-writer、ARIS 等 GitHub 顶级 skill 实践后，提炼出 **10 条可量化的笔记质量标准**：

1. **结构化与导航**（TOC + 公式编号 + References）
2. **可视化强制**（自绘图 + 对照表，禁纯文字铺陈）
3. **公式三件套**（直觉 + 公式 + 符号 + 数值 + 总结）
4. **引用密度**（≥20 条起步，全可点击）
5. **强观点 + 完整句子**（陈述句，无"是一个"句式）
6. **密集互联**（≥3 入链 + ≥3 出链，孤岛即失败）
7. **Case Studies 回扣**（概念 → 真实论文/项目案例）
8. **Last updated 时间戳**（frontmatter.updated 每次必更）
9. **可验证性**（论断必有源，禁"业内人士说"）
10. **可复用性**（Atomic Note 独立成页，便于跨项目）

每条标准配 **自检命令**（可由 LLM 在生成后强制跑一遍）+ **不合格的反例**。附 **10 分制评分卡** 和 **生成后自检流程**（如何在 D:\note 全流程嵌入）。

---

## 1. 结构化与导航（Structure & Navigation）

**标准描述**

每篇 ≥3000 字的笔记必须满足：

- **TOC**：前 200 字内给出目录（手写或 `[[wiki/index|TOC]]` 嵌入），用三级标题
- **H2/H3 分级**：H2 = 大节（≥3 节），H3 = 小节（按需）
- **公式编号**：公式必须有 `(Eq. 1)` 形式编号，并在正文里 `见 Eq. 1` 式回引
- **References 段**：文末 References 段独立成节，全可点击
- **Last updated**：frontmatter.updated 反映最新修改日期

**自检命令**

```markdown
[自检·结构]
- TOC 是否在前 200 字内？[是/否]
- H2 节数 ≥3？[是/否]
- 公式数量 = 编号数量？[是/否]
- References 节存在且全部可点击？[是/否]
- frontmatter.updated = 今日？[是/否]
```

**不合格反例**

- 直接进入正文，无 TOC
- 公式一堆但没编号，后续笔记无法引用
- 引用源只在 frontmatter `sources` 字段，正文里看不到
- 修改后没更新 `updated`

**对标范例**

Lilian Weng 的 [LLM Powered Autonomous Agents](https://lilianweng.github.io/posts/2023-06-23-agent/) 是标杆：前置 TOC + 6 大 H2 节 + 70+ References 全部可点击。

---

## 2. 可视化强制（Visualization Required）

**标准描述**

每篇核心概念笔记必须包含：

- **≥1 张自绘示意图**（Excalidraw / Mermaid / ASCII 流程图）
- **≥1 个对照表**（Markdown 表格，对比方法 / 对比 benchmark / 对比前人工作）
- **关键论点图示**：每节"核心论点"必须有图（不是装饰，是论据）
- **配色规范**：统一色板（蓝 #2563eb 主色 / 灰 #64748b 次色 / 绿 #16a34a 强调），不用花哨渐变

**自检命令**

```markdown
[自检·可视化]
- 自绘图/流程图/示意图数量 ≥1？[是/否]
- 对照表数量 ≥1？[是/否]
- 每节核心论点是否配图？[是/否]
- 配色是否在统一色板内？[是/否]
```

**不合格反例**

- 一篇 5000 字的笔记 0 张图，全靠文字
- 用 bullet list 列举方法对比（应该用表格）
- 插图全是表情符号 / emoji 而非真图
- 配色花哨（彩虹色块 / 渐变背景）

**对标范例**

Jay Alammar 的 [The Illustrated Transformer](https://jalammar.github.io/illustrated-transformer/) 是标杆：每一步配一张自绘图，整篇 30+ 张。

---

## 3. 公式三件套（Formula Triplet）

**标准描述**

每个数学公式必须按以下五件套组织，缺一不可：

```
1. 直觉动机（为什么需要这个公式？）
2. 公式本身（$$...$$）
3. 逐符号拆解（每个变量/运算符是什么意思？）
4. 数值示例（代入具体数字验证）
5. 直觉总结（这个公式在"说"什么？）
```

**自检命令**

```markdown
[自检·公式]
- 每个公式前有直觉动机？[是/否]
- 每个公式后有符号拆解？[是/否]
- 每个公式后有数值示例？[是/否]
- 每个公式后有直觉总结？[是/否]
```

**不合格反例**

- 直接堆叠公式 `$$a^2 + b^2 = c^2$$`，无任何解释
- 公式后只有一句话"显然可得"
- 公式抄原文但没给具体数值

**对标范例**

Lilian Weng 的公式编排是标杆：每个公式前讲动机，公式后讲直觉，并配合 ASCII 示意图。

---

## 4. 引用密度（Reference Density）

**标准描述**

| 笔记字数 | References 数量下限 |
|---|---|
| < 1000 字 | ≥5 |
| 1000-3000 字 | ≥10 |
| 3000-5000 字 | ≥20 |
| ≥5000 字 | ≥30 |

**强制规则**

- 全部引用源 **必须可点击**（`[label](url)` 或 `[[wikilink]]`）
- 引用源 ≥50% 应该是 **学术论文 / 官方文档**，禁全网志链接
- 引用源标注日期（论文标注发表年份，文档标注访问日期）

**自检命令**

```markdown
[自检·引用]
- References 总数 ≥字数阈值下限？[是/否]
- 全部可点击？[是/否]
- ≥50% 是论文/官方文档？[是/否]
- 每条引用标注日期？[是/否]
```

**不合格反例**

- 整篇笔记 0 条 References
- 引用全靠"业内传闻" / "据我所知"
- 引用只有博客，没有论文支撑

**对标范例**

Lilian Weng 的 [Prompt Engineering](https://lilianweng.github.io/posts/2023-03-15-prompt-engineering/) 是标杆：单篇 80+ References，80% 是学术论文。

---

## 5. 强观点 + 完整句子（Strong Opinions & Complete Sentences）

**标准描述**

- **陈述句优先**：用"X 是 Y 因为 Z"形式，避免"X 是一个 Y"形式
- **断言先行**：每节第一句话给结论，再展开论证
- **避免对冲**：少用"可能 / 也许 / 大概 / 或许"，多用"在 X 条件下 Y 成立"
- **学术表达**：避免"我觉得 / 我认为 / 非常重要"，用"研究表明 / 实验显示 / 数据表明"
- **不夸大**：避免"突破性 / 革命性 / 颠覆性"，用"显著优于 / 在 X 条件下提升 Y%"

**自检命令**

```markdown
[自检·风格]
- 全文陈述句占比 ≥70%？[是/否]
- 每节首句是断言？[是/否]
- "可能/也许/大概/或许" 出现次数 ≤5？[是/否]
- 无"我觉得/我认为/非常重要"？[是/否]
- 无"突破性/革命性/颠覆性"？[是/否]
```

**不合格反例**

- "Transformer 是一个很重要的模型"（应改为"Transformer 通过自注意力机制解决了 RNN 的长程依赖问题"）
- "我觉得这个方法可能有用"（应改为"该方法在 X 数据集上取得 Y% 提升，表明 Z 假设成立"）
- "这是一个突破性进展"（应改为"在严格控制变量下，该方法相对基线提升 Y%，且在 N 个 benchmark 中 K 个达到 SOTA"）

**对标范例**

Chris Olah 的 Distill 文章是标杆：每段首句下断言，技术性强但不失可读性。

---

## 6. 密集互联（Dense Linking）

**标准描述**

- **入链 ≥3**：每条笔记至少被 3 篇其他笔记引用
- **出链 ≥3**：每条笔记至少主动引用 3 个相关概念
- **孤岛即失败**：Dataview 查询 `inlinks.length = 0` 的笔记必须处理（合并 / 删除 / 加链）
- **概念自包含**：每条笔记能被独立读懂，链接出去的笔记也应自包含

**自检命令**

```markdown
[自检·互联]
- inlinks 数量 ≥3？[是/否]
- outlinks 数量 ≥3？[是/否]
- 与其他笔记存在显式双向链接（不是单方面提及）？[是/否]
```

**不合格反例**

- 笔记 A 大量引用 B、C、D，但 B、C、D 都不引用回 A（单向引用）
- 一篇笔记只引用了"参见其他资料"，没具体链接

**对标范例**

Andy Matuschak 的 notes.andymatuschak.org 是标杆：每条笔记平均 5+ 入链 + 5+ 出链。

**D:\note 落点**

- 用 Dataview 查询孤立页：见 `obsidian-advanced-toolkit.md` 示例 B
- 用 Obsidian Graph View 检查"叶子节点"
- 每月底跑一次 `kb-lint` 自动检测

---

## 7. Case Studies 回扣（Real-World Case Studies）

**标准描述**

每篇概念类笔记必须包含：

- **≥2 个 Case Studies**：真实论文 / 项目 / 实验中的应用例子
- **结构**：背景 → 用到本文概念 → 结果 → 启示
- **回扣正文**：每个 Case 必须显式引用正文里的某个概念（`如 Case X 中，§3 的 Y 方法被应用于 ...`）

**自检命令**

```markdown
[自检·案例]
- Case Studies 数量 ≥2？[是/否]
- 每个 Case 有背景/应用/结果/启示？[是/否]
- 每个 Case 显式回扣正文某节？[是/否]
```

**不合格反例**

- 讲完概念就结束，没有真实应用例子
- Case 只是名字列出，没讲怎么用上本文概念
- Case 全是 ChatGPT / "某个研究" 等空泛引用

**对标范例**

Lilian Weng 的"Component 3: Tool Use"末尾用 **Scientific Discovery Agents + Generative Agents Simulation** 两个 Case Studies 把抽象概念拉回具体例子。

---

## 8. Last updated 时间戳（Evolution Trace）

**标准描述**

- **frontmatter.updated**：每次修改笔记必须更新此字段（精确到日期 YYYY-MM-DD）
- **大段修改日志**：对内容结构 / 论断做了大改时，在正文末尾加 `## 更新日志` 段记录
- **可追溯**：读者能看到"这条笔记的演化轨迹"

**自检命令**

```markdown
[自检·时间]
- frontmatter.updated = 今日？[是/否]
- 重大修改有 ## 更新日志？[是/否]
- 更新日志记录具体改了什么？[是/否]
```

**不合格反例**

- 改了内容但忘了更新 frontmatter.updated
- 修改前内容已不可信但无任何标记

**对标范例**

Lilian Weng 的博客每篇末尾都有 "Last updated on ..." 标注。

**D:\note 落点**

- 用 Templater 在每次保存时自动更新 `updated` 字段（见 `obsidian-advanced-toolkit.md` 配方 4）
- 配套规则写入 D:\note\CLAUDE.md

---

## 9. 可验证性（Verifiability）

**标准描述**

- **每条论断有源**：所有重要论断都能追到原始论文 / 官方文档 / 数据集 / 代码仓库
- **无来源论断禁词**："业内人士说 / 大家都知道 / 显然 / 显然可得"等
- **数据可复现**：引用的实验结果必须给出 论文 / 数据集 / 复现仓库链接
- **承认局限**：方法 / 数据的局限性必须明确说明，不藏拙

**自检命令**

```markdown
[自检·可验证]
- 每条论断有引用？[是/否]
- 无"业内人士/大家都知道/显然"？[是/否]
- 引用的实验结果有复现链接？[是/否]
- 局限性独立成节？[是/否]
```

**不合格反例**

- "研究表明 XXX" 但没说哪篇研究
- "业界普遍认为 YYY" 无来源
- 只给 benchmark 数字不给数据集 / 模型大小 / 超参

**对标范例**

Distill.pub 每篇都有论文级参考文献 + 复现 notebook 链接。

---

## 10. 可复用性（Reusability / Atomic Notes）

**标准描述**

- **Atomic Note 原则**：每个核心概念独立成页，跨项目可复用
- **不与项目笔记混写**：概念笔记不夹杂项目进度（项目进度放 Daily/Plan）
- **强 slug 命名**：文件名稳定，便于 `[[wikilink]]` 引用（避免"2024-01-15-Transformer-笔记"这类带日期的命名）
- **Frontmatter 完整**：`type / tags / created / updated / sources / prerequisites` 至少 5 个字段

**自检命令**

```markdown
[自检·可复用]
- 文件名稳定（不带日期）？[是/否]
- 不夹杂项目进度？[是/否]
- frontmatter 字段 ≥5？[是/否]
- 在 wiki/index.md 可被检索到？[是/否]
```

**不合格反例**

- 概念笔记掺杂了"上周组会汇报"、"本周实验进展"等短期信息
- 文件名带日期导致每次笔记都要重新链接

**对标范例**

Zettelkasten / Evergreen Notes 的核心就是这条：每条笔记都是"乐高块"，能跨项目复用。

---

## 评分卡（10 分制）

每条标准 1 分。生成后强制自评。

| # | 维度 | 评分（0/1） | 证据（具体行号/段落） |
|---|---|---|---|
| 1 | 结构化与导航 |  |  |
| 2 | 可视化强制 |  |  |
| 3 | 公式三件套 |  |  |
| 4 | 引用密度 |  |  |
| 5 | 强观点 + 完整句子 |  |  |
| 6 | 密集互联 |  |  |
| 7 | Case Studies 回扣 |  |  |
| 8 | Last updated 时间戳 |  |  |
| 9 | 可验证性 |  |  |
| 10 | 可复用性 |  |  |
| **总分** |  | /10 | |

**评级标准**

| 总分 | 评级 | 对标 |
|---|---|---|
| 9-10 | 顶会水准 | Lilian Weng / Distill.pub / 顶会论文 |
| 7-8 | 付费资料水准 | 知名研究员博客 / 顶级技术书 |
| 5-6 | 普通博客水准 | Medium 优秀技术博客 |
| 3-4 | 教科书水准 | 普通教材章节 |
| 0-2 | 不合格 | 待重写 |

**自评动作**

- 总分 < 7：必须修改后再次自评
- 单条 0：必须重写该维度后再次自评
- 连续 3 篇 < 7：触发 `/skill-evolve`，升级对应 skill

---

## 生成后自检流程（强制）

**触发时机**：任何 LLM 生成的笔记落盘后立即执行。

**流程**

```
1. LLM 完成笔记生成
2. LLM 强制执行"10 条自检命令"（见每条末尾的命令块）
3. LLM 填写评分卡，得出总分
4. 若总分 < 7：
   - LLM 自主修改 → 重跑自检 → 提交
5. 若总分 ≥ 7：
   - 在笔记末尾追加 `## 质量自评` 段，记录总分与各维度得分
   - commit + push
```

**集成方式**

1. 新建 `D:\note\.claude\skills\quality-control-checker\SKILL.md`（下一步执行）
2. 在 D:\note\CLAUDE.md 注入"自检为强制环节"规则
3. 升级 paper-note / note-research / daily-paper-generator 三大 skill，把评分卡嵌入输出末尾

---

## 与现有 skill 体系的衔接

| 现有 skill | 升级方向 | 对应标准 |
|---|---|---|
| paper-note | 输出末尾强制附评分卡 | 1-10 全 |
| note-research | 调研笔记必须满足引用密度 + 强观点 | 4, 5, 9 |
| daily-paper-generator | Top 1 论文精读走 10 条全检 | 1-10 全 |
| obsidian-project-kb-core | 笔记入库前自检 | 6, 8, 10 |
| 新建 quality-control-checker | 通用自检 skill | 全部 |

---

## 引用源

- **方法论基础**：Zettelkasten (Luhmann) / Evergreen Notes (Matuschak) / CODE (Forte) / Atomic Notes / SQ3R
- **顶级范例**：Lilian Weng / Jay Alammar / Chris Olah / Karpathy / Chollet / Sebastian Raschka / Distill.pub / 3Blue1Brown / Papers with Code / ar5iv
- **Obsidian 工具**：kepano/obsidian-skills + K-Dense-AI/claude-scientific-writer + ARIS + Master-cai/Research-Paper-Writing-Skills
- **本仓配套调研**：
  - [[research/08-strategy/note-quality-research-methodology|方法论调研]]
  - [[research/08-strategy/note-quality-research-blogs|博客范例调研]]
  - [[research/06-tools/obsidian-advanced-toolkit|Obsidian 工具链]]
  - [[research/08-strategy/note-quality-research-skills|Skill 生态调研]]