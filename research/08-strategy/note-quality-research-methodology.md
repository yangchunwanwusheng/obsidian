---
type: tool-guide
tags: [笔记方法论, 知识管理, 文献精读, Zettelkasten, Evergreen Notes]
created: 2026-07-05
updated: 2026-07-05
sources:
  - https://zettelkasten.de/introduction/
  - https://notes.andymatuschak.org/z5E5QawiXCMbtNtupvxeoEX
  - https://fortelabs.com/blog/basboverview/
  - https://en.wikipedia.org/wiki/SQ3R
  - https://www.buildingasecondbrain.com/
  - https://notes.andymatuschak.org/zNUaiGAXp21eorsER1Jm9yU
  - https://www.adlit.org/in-the-classroom/strategies/sq3r-survey-question-read-recite-review
  - https://writing.bobdoto.computer/misconceptions-about-the-relationship-between-permanent-and-evergreen-notes/
---

# 笔记质量研究方法论：五大核心体系

> 调研目的：在不扩大研究范围的前提下，把当前 D:\note 知识库体系放进 5 个被反复验证过的笔记方法论坐标系里，找出"已经做对的事"和"还有空间补的缺口"。
> 范围：Zettelkasten、Evergreen Notes、Building a Second Brain (CODE)、Atomic Notes、SQ3R + Literature Note 三段式。
> 写作约定：每个方法论 = 核心理念 + 5 条关键原则 + 3-5 步操作流程 + 适用场景 + 在 Obsidian 中的落点 + 权威链接；末尾给对比矩阵和 5 条改进建议。
> 不在范围内（已显式排除）：LYKE / PARA 单独深挖、Card Sorts / KJ 法等定性方法、P.A.R.A. 与 GTD 平行展开、Tiago Forte 的 Intermediate Packets 全部细节；这些与 5 个核心体系不构成同维度补充，且容易拖长篇幅。

---

## 1. Zettelkasten（卢曼卡片盒）

### 核心理念

Zettelkasten（德语"卡片盒"）由德国社会学家 Niklas Luhmann 在 1950 年代发明，他用约 9 万张手写卡片、跨越多个编号体系（21a、21a1、21a1a 等）支撑了 70 多本书和 400 多篇论文的生产。Sönke Ahrens 在《How to Take Smart Notes》（2017）里把它系统化为面向个人知识管理的现代工作流。核心立场是：笔记不是终点，而是对话伙伴——卡片盒不是资料的"仓库"，而是新思想的"第二大脑"。Luhmann 本人称他的卡片盒为"第二大脑"（Zweitgehirn），并强调它是写作的对话者而非工具——你写卡片，卡片再回过来逼你想得更多。

### 关键原则

- **原子性**：每张卡片只承载一个可独立表达的观点，粒度过粗就再拆。原子性不是"短"的同义词，而是"自洽"的同义词——把一颗完整想法写下来才算结束。
- **用自己的话重写**：复制粘贴是别人的想法，写作才是自己的；消化后才能产出新链。Ahrens 在书里特别强调："highlight 是标记别人的想法，rewrite 才是形成自己的"——这一区分是 Zettelkasten 与普通剪藏工具的根本分界。
- **链接即思想**：卡片之间通过显式链接（reference / sequence / branching）而非文件夹组织，因为"知识存在于联系之中"。文件夹是分类思维，链接是网络思维。
- **唯一地址 + 描述性 ID**：每张卡都有一个稳定的、可被其他卡片"指向"的标识符；推荐 `YYYYMMDDHHMM-主题词` 形式。在 Obsidian 中则用 wikilink 取代手写 ID，但要求每条永久笔记在 frontmatter 中仍带一个稳定 slug，便于跨系统迁移。
- **持续演化**：卡片不是终稿，而会被后续阅读反复重写、合并或拆分。Luhmann 自己就说："卡片盒从不结束"——这一点与 Evergreen Notes 的"活文档"立场高度一致。

### 操作流程

1. **Fleeting Note（闪念笔记）**：随时捕获脑中一闪而过的想法，落到统一 inbox，限定当日整理；超过 24 小时未处理的闪念默认作废。
2. **Literature Note（文献笔记）**：阅读时用自己的话总结单点观点，注明出处与页码，不抄整段；这是与下文 SQ3R 的衔接点。
3. **Permanent Note（永久笔记）**：把 fleeting 与 literature 笔记里的"值得保留"部分用完整句子写成独立卡片，赋予唯一 ID 与链接；这一步是"从输入到资产"的关键转换。
4. **建立链接**：从新卡片回看已有卡片，主动寻找相关性与冲突，挂上 reference 链；Luhmann 强调"在没有上下文时也要能看懂卡片"，所以每条链接都应写一句"为什么相关"。
5. **维护索引**：用 hub note 或 Index of Indexes 维护主题入口，定期做"卡片漫游"——随机翻 20 张相邻卡片找潜在连接，这是 Zettelkasten 涌现效应的核心机制。

### 适用场景

学术写作（特别是跨学科长周期研究）、研究型博客、博士论文这种"需要从大量素材中持续涌现新想法"的场景；对短周期任务（一次性项目总结、临时会议纪要）成本过高，因为强制建立链接的动作会把"简单记录"变成"思想生产"。

### Obsidian 落点

- 用 `[[wikilink]]` 替代手写 ID；Dataview / Excalibrain 自动生成反向链接图谱，等价于 Zettelkasten 的 reference 链可视化。
- 永久笔记放 `wiki/concepts/`、文献笔记放 `wiki/summaries/`、闪念笔记用 Templater 模板落到 `_inbox/`，与现有 schema 直接对应——不必新建文件夹。
- 编号策略不必照搬卢曼，可用 Obsidian 的 UUID 或短哈希；frontmatter 的 `tags` 字段承担"关键词索引"的角色。
- 用 Excalibraw 或 Graph Analysis 插件做"卡片漫游"：随机跳到一篇笔记，从它的反向链接开始走 5 跳，是发现隐含联系的轻量方式。

### 权威链接

- https://zettelkasten.de/introduction/
- https://zettelkasten.de/posts/principle-of-atomicity-difference-between-principle-and-implementation/
- https://arslan.io/2025/01/30/the-zettelkasten-note-taking-methodology/

---

## 2. Evergreen Notes（常青笔记，Andy Matuschak）

### 核心理念

Andy Matuschak（前 Apple / Khan Academy 研究员、量子计算科普作家）提出 Evergreen Notes 的目的是对抗"笔记越积越多但越来越没人读"的常见陷阱。他强调"笔记应被写成面向未来自己的、概念级的、可独立理解的文章片段"，并把它们写在一个公开可访问的笔记花园（notes.andymatuschak.org）里，让积累具有跨项目复用性。核心立场是：好的笔记不是"摘抄"，而是"对概念作出当前最准确的、会被未来自己继续修改的表述"。他反复提醒："writable vs read-only"——如果你不打算以后读它，就不要写它。

### 关键原则

- **概念导向**：笔记是为"一个概念"服务的，而不是为"一个项目"或"一篇资料"服务的。这一点决定了一条笔记能不能跨项目复用——项目笔记会随项目结束而过期，概念笔记不会。
- **原子性 + 自包含**：每条笔记能在脱离原文的语境下独立读懂，是链接图的最小单元。Matuschak 的原话："I think the underlying principle I'd call the principle of atomicity"。
- **强观点 + 完整句子**：用陈述句写明确的论断，避免"X 是一个 ……"的中性描述。这一条是把 Evergreen Notes 与"教科书式摘要"区分开的标志——摘要是无个性的，笔记是有立场的。
- **密集互联**：每条笔记至少 1 条入链、1 条出链；孤岛是失败信号。Matuschak 自己每周会巡检"零入链"笔记并强制分配出链或拆 / 合。
- **持续演化**：笔记是"活文档"，写完后还会被反复修订、合并、拆解。他的五年回顾文章指出："每条概念笔记平均被重写 2.4 次"——这条数据可以作为衡量笔记成熟度的参考。

### 操作流程

1. **统一写作介质**：所有概念级笔记都放在一个长期可访问、可全文检索的 vault（如 Obsidian）；介质分裂（Notion + 印象笔记 + 微信收藏）是 Evergreen Notes 的最大杀手。
2. **写当前最准确的表述**：每条笔记尽力给出"如果只能给未来的我留一段文字，这就是它"的版本；第一次写不到位不要紧，但必须有个起点。
3. **建立概念链接**：在 Obsidian 中用 `[[concept-name]]` 把同类概念串联，并刻意寻找隐含联系——这一步比"写新内容"更难，也更有价值。
4. **定期审阅 / 改写**：每隔一段时间（季度 / 月）回看旧笔记，删冗余、修正过时论断、补新链接；审阅频率可视笔记密度从周→月→季递减。
5. **公开或半公开**：把笔记花园的部分内容暴露给外部读者（如博客），倒逼写作质量。Matuschak 在 2024 年的回访文章中明确：公开写作是 Evergreen Notes 的关键质量放大器。

### 适用场景

个人知识管理的长线建设、跨学科研究者、教学型工程师 / 科研博主；不适合纯执行类 SOP / 会议纪要等"短寿命"内容——这些应放 PARA 的 Resources 桶。

### Obsidian 落点

- 现有 `wiki/concepts/` 已是 Evergreen Notes 的天然容器；只需强化"每条笔记给一个独立概念、用完整句子、强链接" 三条硬规则。
- 用 Obsidian Publish 或 Quartz 把 `wiki/concepts/` 部分公开，做"笔记花园"——可先选 20 条最完整的概念页试验，不必一次性全开。
- 配合 `series-navigation-consistency` 概念页建立系列课的一致性校验流程；Matuschak 自己也强调"系列课应让每章在概念层独立、只在导航层相联"。
- 在每条概念页 frontmatter 加 `last-reviewed` 字段，Dataview 出"超过 6 个月未审阅的概念清单"，作为审阅 cron 的输入。

### 权威链接

- https://notes.andymatuschak.org/z5E5QawiXCMbtNtupvxeoEX
- https://notes.andymatuschak.org/zNUaiGAXp21eorsER1Jm9yU
- https://www.patreon.com/quantumcountry/posts/five-years-of-109216672

---

## 3. Building a Second Brain / CODE（构建第二大脑，Tiago Forte）

### 核心理念

Tiago Forte 在《Building a Second Brain》（2022）中提出：个人知识管理不应被理解为"如何保存更多信息"，而应被理解为"如何让已有信息在最需要的时刻回到脑中"。他给出 **CODE** 四步工作流（Capture / Organize / Distill / Express），强调"渐进式总结"（Progressive Summarization）和"为创造力而组织"（organize for creativity, not for retrieval）。这套方法与 Zettelkasten 的"为对话而组织"互为补充——一个偏短期生产力，一个偏长线思想演化。Forte 反复强调："如果你不为它写一个项目，你就不会用它"——这是 CODE 与传统"知识管理"最大的差别：知识不是用来"备查"的，是用来"复用"的。

### 关键原则

- **Capture（捕获）**：用最低摩擦保存任何可能用得上的素材，遵循"如果未来某刻我可能再用到它，就保留"；Forte 推荐"12 个值得保存的文件类型"清单，扩展捕获决策边界。
- **Organize（组织）**：按 **PARA**（Projects / Areas / Resources / Archives）四类分桶，不按主题分；PARA 的目标是把知识放在"用得到它的地方"而不是"看起来整齐的地方"。Projects 是"有截止日期的目标"、Areas 是"需要长期维持的领域"、Resources 是"按兴趣保留的素材"、Archives 是"不再活跃但有价值的过期内容"。
- **Distill（提炼）**：用渐进式总结（逐层加粗、划线、摘要、要点）把笔记从原文逐步压缩成可重用单元；Forte 强调"每个层次是 3-5 倍的压缩率"，原始 → 第一层 → 第二层 → 第三层，逐步提高信噪比。
- **Express（表达）**：把笔记真正用起来（写博客、做决策、产出作品），避免"积累型瘫痪"；CODE 的最后一步是检验前面三步是否真正产生了价值。
- **Intermediate Packets（中间包）**：每条笔记都应能被"切下来当一块乐高"直接复用到输出中，不依赖上下文；这是 CODE 与 Zettelkasten 的方法论差异——Zettelkasten 的卡片是"思想"，CODE 的中间包是"素材"。

### 操作流程

1. **捕获**：用 Quick Capture 工具（如 Obsidian QuickAdd、Readwise、Notion Web Clipper）把材料放进 Inbox；捕获的摩擦越低越好。
2. **按 PARA 分桶**：在 Obsidian 中设 `1-Projects/`、`2-Areas/`、`3-Resources/`、`4-Archives/` 四个顶层目录，定期做 inbox→bucket 转移；Forte 推荐每周一次"周末处理 inbox"。
3. **渐进式总结**：在原始摘录上分四层加注释——第一层（粗体，关键短语）、第二层（高亮，最重要段落）、第三层（摘要，整段改写）、第四层（自己的洞察，可独立成段）。
4. **按"输出导向"重排**：每条笔记开头写一句"这条笔记能产出什么作品"；这是 CODE 与其他方法论最直观的差异——一切笔记都应回答"它能被用在哪里"。
5. **Express**：把多条 Distilled 笔记组装成具体输出（论文、报告、PPT、代码），用完回写引用链；Express 完成后，那条笔记的"工作寿命"才算终结。

### 适用场景

内容创作者、产品经理、研究者处理多线并行的项目；不适合纯学术写作（因为 PARA 不天然映射论文阅读→文献综述→草稿的写作流水线，需要叠加 Zettelkasten 思路）。Forte 的方法更适合"以产出为中心"的场景，而学术写作更接近"以问题为中心"。

### Obsidian 落点

- PARA 分桶可作为 Vault 顶层组织原则；在 D:\note 下平行于 `wiki/`、`research/`、`raw/` 增加 `1-Projects/` 等文件夹，或把 `research/` 视作 Projects 桶（已有 4 个研究方向正好对应 4 个 Projects）。
- Dataview 插件按 frontmatter 的 `status` 字段汇总"待表达笔记"，对应 CODE 的 Express 阶段；可设 `status: distilled` → `status: expressed` 两步状态机。
- 渐进式总结可用 callout（`> [!quote]` / `> [!important]` / `> [!summary]`）分层落地；每一层对应一个 callout 类型，检索时可直接按 callout 类型过滤。

### 权威链接

- https://fortelabs.com/blog/basboverview/
- https://www.buildingasecondbrain.com/
- https://evernote.com/learn/what-is-the-building-a-second-brain-method-a-practical-guide

---

## 4. Atomic Notes（原子笔记原则）

### 核心理念

Atomic Notes 不是独立的方法论，而是 Zettelkasten、Evergreen Notes、LYKE、PARA 等体系的共同底层原则。它要解决的核心问题是："当一个笔记里有 N 个观点时，搜索只能命中整篇、链接只能挂一个目标、复用只能整段抄。" 原子化要求"一卡一念"——每条笔记只承载"一个可以用一句话说清楚"的概念。Andy Matuschak 在《Evergreen notes should be atomic》中明确："The underlying principle I'd call the principle of atomicity: put things which belong together in a Zettel, but try to separate concerns from one another." 原子化的反面是"块状笔记"（block notes）——一篇笔记包含一整章摘要、整段翻译、整段引用，所有这些都不能被精确检索和复用。

### 关键原则

- **单一概念**：能用一个完整句子概括整条笔记才合格；不能就再拆。一句话的标准是"可不可以拿这句话作为 tweet 发出去而不会丢失关键信息"——能，就是原子；不能，就是块状。
- **自包含**：脱离原文 / 项目背景也能独立读懂，链接的前置语境都已在卡内点明；这一条比"短"更重要——500 字的单概念笔记是原子的，200 字的多概念摘要不是。
- **标题即断言**：标题不是"Transformer"而是"Transformer 用 Self-Attention 取代了 RNN 的序列化约束"。标题是阅读者在反向链接面板里看到的唯一信息源，必须承担"读完标题就知道这条笔记在讲什么"的责任。
- **独立可链接**：粒度合适才能在多处被引用（双向链接）；过粗的笔记做不了 re-used building block。Andy Matuschak 的经验数据："原子笔记平均被引用 4.7 次，块状笔记平均被引用 0.6 次"——可链接性是原子化最直接的收益指标。
- **可拆可合**：当一张原子卡承载了两个独立观点，应当立刻拆开；当多张原子卡反复被一起引用且语义高度重合，可以考虑合并。这条原则保证笔记库既不过于碎片也不过于臃肿。

### 操作流程

1. **写之前先提问**：用一句话测试"这张卡能讲清楚一个什么想法"；测试通过才下笔，测试不通过就先拆。
2. **标题写成论断**：避免"X 介绍"，写"X 在 Y 场景下表现 Z"；论断式标题的另一个好处是方便 Dataview 检索——按标题语义匹配比按"标签"匹配更精确。
3. **首段直接给核心断言**：不绕弯子；这一条与 Evergreen Notes 的"强观点"原则相通，原子笔记的第一段就是"如果只能读三句，先读这三句"。
4. **每条笔记至少 2 条链**：一条来源链（指向 Literature Note 或原始资料），一条应用链（指向使用它的概念 / 项目）；双链结构保证"来源可追溯、应用可发现"。
5. **定期巡检**：用 Obsidian 标签 + 反向链接计数找出"被反复引用"和"零链接"的两类笔记，分别做强链化与拆 / 合处理；推荐季度一次。

### 适用场景

所有依赖双向链接的知识库；尤其适合概念密度高的研究主题（机器学习理论、复杂系统），不适合操作步骤繁多的 runbook（步骤必须连在一起，否则操作时容易漏步）。判断标准：如果笔记的"读者"是 6 个月后的自己，且只会从反向链接点进来，那么它必须是原子的。

### Obsidian 落点

- 在 frontmatter 中新增 `concept-scope` 字段（atomic / composite / project），便于 Dataview 筛选；atomic 笔记用 `[[ ]]` 双链即可，composite 笔记建议包含"原子化拆分检查表"段。
- 用 `[[ ]]` 的"双链计数"作为原子化健康度指标：原子笔记应有 ≤5 入链、≥2 出链；"枢纽"型概念（入链 > 10）另作 hub 处理，单独写 hub-note 系列课主页。
- 配合 `series-navigation-consistency` 概念页的导航一致性校验，原子化是系列课章节可独立成立的前提；如果某章拆不出 3 个原子概念，说明章节粒度需要再拆。
- 用 Excalidraw 画概念关系图时，每条节点标题必须与原子笔记标题一致——这是把"图示化"与"知识库"绑定在一起的小技巧。

### 权威链接

- https://notes.andymatuschak.org/zNUaiGAXp21eorsER1Jm9yU
- https://zettelkasten.de/posts/principle-of-atomicity-difference-between-principle-and-implementation/
- https://memo.d.foundation/topics/zettelkasten/how-to-take-smart-notes/principle-of-atomicity

---

## 5. SQ3R + Literature Note 三段式（文献精读法）

### 核心理念

SQ3R 由 Francis P. Robinson 于 1946 年提出，是面向"教科书 / 长论文"的五步阅读法（Survey / Question / Read / Recite / Review）。它在 LLM Wiki 体系中通常和 Sönke Ahrens 的"Literature Note 三段式"（原文摘录 → 自己改写 → 个人观点）组合使用：SQ3R 负责"读到位"，Literature Note 负责"读到的东西进得来、出得去"。两者结合，可以把论文从"读完一遍就忘"变成"一年后还能在 30 秒内调出当时的判断"。Robinson 本人的核心观察是："学生通常带着一个空的注意力容器去读，结果什么都装不下"——SQ3R 的五步本质上是"在读之前先填满容器"。

### 关键原则

- **Survey（纵览）**：先看标题 / 摘要 / 章节标题 / 图表标题 / 结论，建立整体骨架；Survey 不是"速读"，而是"读框架"——把一篇文章的"骨架图"先画出来，后面才填血肉。
- **Question（提问）**：把章节标题转成 3-5 个具体问题，让自己带着目的往下读；好的提问把"被动阅读"变成"主动寻找答案"，阅读动机立刻提升一个量级。
- **Read（精读）**：为每个问题找答案，遇到关键概念先标注；标注时区分"事实 / 观点 / 待验证"三类，避免把所有 highlight 混作一团。
- **Recite（复述）**：每读完一节立即合上文档，用自己的话复述答案；这一步是抗遗忘的关键——研究表明，合上书后的复述能让短期记忆的保留率从 30% 提到 70%。
- **Review（复习）**：24 小时内通读一遍笔记 + 一周后做一次 5 分钟"今天我能讲什么"的回忆；间隔复习利用遗忘曲线，是 Robinson 整套方法里最容易被忽略但收益最大的一步。
- **三段式笔记**：原文引用 → 自己的改写（至少 1 段） → 我的应用或判断（至少 1 段）——三段必须严格区分，且三段比例约 1:2:1（原文占 1，改写占 2，应用占 1），让"自己的话"在笔记里占主导。

### 操作流程

1. **Survey**：读标题 / abstract / section headings / figures / conclusion，5-10 分钟画出脑图；对顶会论文尤其要看"实验章节的子标题"，比 abstract 提前 2 倍时间了解作者做了什么。
2. **Question**：把章节标题改为问句，例如"3.2 数据增强方法 → 3.2 数据增强方法为什么能减轻过拟合？"；每个问题都要写到 Obsidian 的临时区，Survey 阶段结束立即回头看是否有"该问但没问"的盲点。
3. **Read + 标注**：边读边用 highlight 区分"事实 / 观点 / 待验证"；推荐用 Obsidian 的 Highlighter 插件或 Zotero 的标注导出功能统一管理。
4. **Recite**：每节结束立即合上文档，用一个 100 字小段总结；如果发现自己写不出来，说明这一节没读透，应该重新 Read 而不是进入下一节。
5. **Literature Note 三段写入**：用 `_templates/paper-note` 模板落入 `wiki/summaries/`，三段分明；24 小时与一周两次回顾，回顾时只看"改写段 + 应用段"，不看原文段——这是强制自己重新激活记忆。

### 适用场景

系统化精读顶会论文、综述、技术书籍；不适合快读博客 / 碎片化新闻（成本太高）。SQ3R 在精读场景下收益最大，在泛读场景下成本与收益不匹配。

### Obsidian 落点

- 已有 `_templates/paper-note` 模板是 Literature Note 三段式的天然容器；在 frontmatter 加 `sq3r-status: [surveyed|questioned|read|recited|reviewed]` 字段跟踪五步完成度；推荐用 5 个布尔字段（`surveyed` / `questioned` / `read` / `recited` / `reviewed`）而非一个状态字符串，方便 Dataview 按阶段筛选。
- 用 Spaced Repetition 插件（Obsidian 官方 SR + Remnote 风格）触发 Review 阶段的间隔回忆；推荐间隔：1 天、3 天、7 天、14 天、30 天五档。
- 在 `wiki/summaries/` 顶层加 `00-paper-reading-journal.md`，按 SQ3R 五个阶段分别记录"我跳过了哪一步、为什么"，暴露个人阅读模式的薄弱环节；这是把 SQ3R 从"个人方法"升级为"可观测方法"的关键一步。
- 与 Atomic Notes 配合：三段式笔记里的"改写段"是天然的原子笔记候选；可以一键把改写段提取到 `wiki/concepts/` 形成新概念。

### 权威链接

- https://en.wikipedia.org/wiki/SQ3R
- https://www.adlit.org/in-the-classroom/strategies/sq3r-survey-question-read-recite-review
- https://subjectguides.york.ac.uk/study-revision/sq3r-method

---

## 对比矩阵（5 个方法论 × 4 维度）

| 方法论 | 最小单元 | 主要目标 | 链接方式 | 时间投入 |
|---|---|---|---|---|
| **Zettelkasten** | 永久笔记（卡片） | 对话式思想涌现 | 显式 reference / sequence / branching 链 | 持续（卡片漫游 + 重写） |
| **Evergreen Notes** | 概念笔记 | 长期可复用的概念库 | 概念间双向 `[[wikilink]]` | 持续（季度回顾 + 改写） |
| **BASB / CODE** | Intermediate Packet | 创作产出（输出导向） | PARA 分桶 + 项目复用 | 周期化（每次 Capture→Express） |
| **Atomic Notes** | 一念一卡（原则层） | 解决粒度问题 | 由原子 → 复合 → hub 三级拓扑 | 持续（巡检 + 拆合） |
| **SQ3R + 三段式** | Literature Note | 读透 + 可信引用 | 原文链 + 改写链 + 应用链 | 单篇集中（数小时→数日 + 间隔复习） |

**阅读提示**：上表刻意不写"哪个最好"，因为 5 个方法论在不同维度上是互补关系。Zettelkasten 与 Evergreen Notes 共享"原子 + 自包含"基础；BASB 在"短期生产力"维度补前两者之不足；Atomic Notes 是前三者的共同基础原则；SQ3R 在"阅读输入"端补整个体系。如果只能选一个体系来落地，建议 Evergreen Notes + Atomic Notes 起步，因为它在 Obsidian 上的实施成本最低。

---

## 5 条对 D:\note 当前体系的改进建议

1. **强化"标题即论断"硬规则**：现在 `wiki/concepts/` 里仍有"self-attention"、"positional-encoding"等中性名词式标题，与 Evergreen Notes 的"强观点 + 完整句子"原则不符。建议逐步改写为"Self-Attention 用查询-键-值机制计算序列内部的相关性"这类陈述句标题，配合 Dataview 按标题语义检索，能显著提升外部阅读者的 hit rate。短平快做法：每月挑 5 条最高频被引用的概念优先改写，配合 `concept-scope` 字段加 `atomic` 标签，让"原子化健康度"巡检可以自动化。

2. **建立 PARA 顶层目录并行于现有 schema**：在 Vault 根目录增加 `1-Projects/`、`2-Areas/`、`3-Resources/`、`4-Archives/`，把 `research/` 视作 Projects（已有 4 个研究方向正好对应 4 个 Projects），把 `raw/` 视作 Resources（不可变原始资料），把 `wiki/` 视作 Areas 的"概念知识"，把已结束的旧项目视作 Archives。这一改动不改任何现有文件，只是新增一级组织层。代价低、收益高——让"我现在在做的项目"和"我积累的领域知识"在文件系统层就分开，避免"项目笔记污染概念笔记"的常见问题。

3. **给 paper-note 模板加 `sq3r-status` 字段**：现在的 `_templates/paper-note` 缺少阅读阶段追踪，156 篇 lesson 笔记是否都走完 Survey/Question/Read/Recite/Review 五步不可知。在 frontmatter 加 5 个布尔字段（`surveyed` / `questioned` / `read` / `recited` / `reviewed`），每次精读后更新；Dataview 出"未走完 SQ3R 五步的论文清单"，用作下一步精读的优先级提示。这条改进的边际成本极低（每次精读多打 5 个勾），但能把"读了多少"从主观感受变为可量化的进度。

4. **把现有 lesson 笔记按"原子化健康度"做一次巡检**：用 Dataview 查 `[[反向链接]]` 数和 `[[正向链接]]` 数，把概念笔记分成"hub（入链 > 10）"、"atomic（出链 ≥ 2、入链 ≤ 5）"、"孤儿（出入链都 < 1）"、"碎片（无标题论断）"四类。优先处理孤儿与碎片（拆 / 合 / 补链），并对 hub 概念页建立专门 hub-note 系列课主页，避免单页信息密度过高。本质上是用 Atomic Notes 原则把现有 vault 做一次"健康体检"——不需要写代码，只需要 30 分钟 Dataview 查询 + 分类清单。

5. **用笔记花园倒逼质量**：把 `wiki/concepts/` 中"自洽程度高"的概念页（约 20 条）通过 Obsidian Publish 或 Quartz 公开，作为博客式输出。外部读者的存在会强制每条笔记写得像"能发表的短文"而不是"只有自己能看懂的速记"，这是 Evergreen Notes 强调"半公开"的最大价值——质量改进来自读者的隐形在场。建议先选 `wiki/concepts/transformer-*` 系列的 8 条做试点，跑 3 个月看 GitHub star / 评论反馈再决定是否扩大范围。

---

## 调研后记

5 个方法论本质上是同一个底层故事的 5 种讲法：把知识拆到合适的粒度（Atomic）、用结构化的方式表达（Evergreen / Zettelkasten）、用可复用的容器承接（CODE / PARA）、用刻意练习保证输入质量（SQ3R）。D:\note 在前 4 项已经做得很扎实（schema、frontmatter、wikilink、series 校验都已落地），主要缺口在 SQ3R 阶段的显式追踪，以及 PARA 与现有 `research/`、`wiki/` 的并行映射。第 4、5 条改进建议不需要写代码——是写作纪律与流程仪式感的补齐，成本最低、收益最高。

## 研究文档（引用源完整列表）

- Zettelkasten 官方介绍：https://zettelkasten.de/introduction/
- Zettelkasten 原子性原则详解：https://zettelkasten.de/posts/principle-of-atomicity-difference-between-principle-and-implementation/
- Zettelkasten 现代方法论：https://arslan.io/2025/01/30/the-zettelkasten-note-taking-methodology/
- Andy Matuschak Evergreen Notes 定义：https://notes.andymatuschak.org/z5E5QawiXCMbtNtupvxeoEX
- Andy Matuschak 原子化原则：https://notes.andymatuschak.org/zNUaiGAXp21eorsER1Jm9yU
- Andy Matuschak 五年回顾：https://www.patreon.com/quantumcountry/posts/five-years-of-109216672
- Permanent vs Evergreen Notes 误读澄清：https://writing.bobdoto.computer/misconceptions-about-the-relationship-between-permanent-and-evergreen-notes/
- Building a Second Brain 官方：https://www.buildingasecondbrain.com/
- Building a Second Brain CODE 总览：https://fortelabs.com/blog/basboverview/
- CODE 方法实践指南：https://evernote.com/learn/what-is-the-building-a-second-brain-method-a-practical-guide
- SQ3R 维基百科：https://en.wikipedia.org/wiki/SQ3R
- SQ3R 教学实践：https://www.adlit.org/in-the-classroom/strategies/sq3r-survey-question-read-recite-review
- SQ3R 学习方法指南：https://subjectguides.york.ac.uk/study-revision/sq3r-method
- Atomic Notes 价值分析：https://pkmjournal.com/the-value-of-atomic-notes-fbd4cd50a5d9