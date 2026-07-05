# Claude Skill 生态调研

> **元信息**
> - 调研日期：2026-07-05
> - 调研方式：`web-search-prime` + `gh api` + `gh search repos`（所有 star 数与更新时间均为实时查询 GitHub API，非估算）
> - 覆盖仓库数：24 个 GitHub 仓库（含 2 个聚合目录 + 17 个具体 skill/plugin + 5 个对比/参考项目）
> - 调研范围：awesome-claude-skills / 论文精读 / 文献综述 / Obsidian & LLM Wiki / 学术写作 / 研究助手类 / Zettelkasten 实现
> - 数据快照：所有数据来自 2026-07-05 当日的 GitHub API 响应

---

## 摘要

调研覆盖 24 个 GitHub 仓库后，按"对 D:\note 当前 skill 体系的增量价值"打分，可得：

**5 个最值得集成（P0）**
1. **kepano/obsidian-skills**（39.7k★）—— Obsidian CLI/Bases/Canvas/Defuddle 官方 agent skills，直接补齐 `obsidian-kb-artifacts` 缺的 CLI/Bases/Canvas 工具链
2. **K-Dense-AI/claude-scientific-writer**（2.0k★）—— 25 个科研 writing 子 skills，`literature-review` / `citation-management` / `scientific-schematics` 三个可整段移植
3. **AgriciDaniel/claude-obsidian**（8.7k★）—— 与 LLM Wiki 模式高度对齐，14 个 wiki-* skills 几乎就是 D:\note 流程的另一个工程实现
4. **wanshuiyin/ARIS**（13.0k★）—— `idea-discovery` / `citation-audit` / `auto-review-loop` 三个 skill 直接补 `top-tier-paper-workflow` 的 S3/S8 缺口
5. **Master-cai/Research-Paper-Writing-Skills**（4.7k★）—— ML 论文写作分章节模板，对应 `ml-paper-writing` 的可执行化补充

**5 个可参考（P1）**
- `anthropics/skills`（158k★，官方范例库，是路由参考）
- `alirezarezvani/claude-skills`（20.2k★，聚合 skill 库）
- `trapoom555/claude-paperloom`（92★，自维护论文知识图谱）
- `juliye2025/evil-read-arxiv`（1.5k★，中文 arxiv 阅读链路）
- `chaenmasahiro0425/exbrain`（43★，4 层 raw/wiki/digest/identity 架构）

**不推荐 / 仅供参考**
- 大量 0-star 的 `literature-review-ai` / `Paper2Code` / `paper-research` fork，质量与维护均无保障
- `Rlin1027/philosophy-research-agents`（2★）—— 哲学专用，方向不对口
- `K-Dense-AI/claude-scientific-writer` 的 `clinical-*` / `treatment-plans` —— 临床医学子包，与本仓科研路线无关
- `summarizepaper/summarizepaper`（308★）—— Web App 而非 skill，不能复用
- `assafelovic/gpt-researcher`（28k★）与 `stanford-oval/storm`（29.8k★）—— 都是独立 Python 工具，不易作为 skill 集成，但可作为**架构参考**（多 agent 协作 + 引用追溯）

---

## 推荐直接集成（P0）

### 1. kepano/obsidian-skills
- **GitHub URL**：https://github.com/kepano/obsidian-skills
- **Star**：39,788 / Forks：2,825 / 最近 push：2026-06-08 / 最近 update：2026-07-05
- **维护活跃度**：高（Obsidian 官方维护者 kepano 本人）
- **核心功能**：把 Obsidian CLI、Markdown 格式、Bases 语法、JSON Canvas 文件、defuddle 网页抽取封装成 Claude Code 可调用的 6 个 skill。skill 包含 SKILL.md + 完整的 frontmatter / syntax 规则 / 调用样例。
- **skill 清单**（来自 `gh api .../contents/skills`）：`defuddle`、`json-canvas`、`obsidian-bases`、`obsidian-cli`、`obsidian-markdown`
- **与 D:\note 现有 skill 的差距 / 互补点**：
  1. **obsidian-cli**：当前 D:\note 所有 skill 都不假定 Obsidian 已安装且 CLI 可用；这是 D:\note 真实环境已经具备但 skill 文档里没说的事实，补上后可让 `obsidian-kb-artifacts` / `kb-map` 真正调用 `obsidian` 二进制完成 canvas 校验与渲染。
  2. **obsidian-bases**：`.base` 文件是 Obsidian 1.5+ 的新查询语法，D:\note 的 `_templates/` 与 KB 注册表都还没用上；导入后能让 `kb-status` / `kb-index` 输出更交互。
  3. **json-canvas**：当前 D:\note 没有结构化思维导图 / 关系图自动生成能力；`obsidian-literature-workflow` 的 `Maps/literature.canvas` 是空文件占位，本 skill 可直接填入真实 schema。
  4. **obsidian-markdown**：D:\note 的 frontmatter 约束比 Obsidian 默认严格（`sources` / `topic` / `series` 等），需要派生而不是直接复制，但格式骨架可直接套。
  5. **defuddle**：D:\note 路由层已有 `defuddle` skill，但缺少**面向 Claude 的调用模板**（"如何让 Claude 决定调用 defuddle 的条件"），kepano 版本补的就是这层 prompt glue。
- **集成建议**：必装。复制 `.claude/skills/obsidian-cli` 与 `json-canvas` 两个 skill 到 `D:\note\.claude\skills\`，前端不动；再把 `obsidian-markdown` 的内容并入 `obsidian-kb-artifacts/SKILL.md` 的"Frontmatter 规范"段。风险评估：低（官方维护、活跃更新、纯文档）。
- **优先级**：**P0（必装）**

### 2. K-Dense-AI/claude-scientific-writer
- **GitHub URL**：https://github.com/K-Dense-AI/claude-scientific-writer
- **Star**：2,051 / Forks：245 / 最近 push：2026-07-05（今天！）/ 最近 update：2026-07-05
- **维护活跃度**：极高（当日提交，节奏密集）
- **核心功能**：把"科研写论文"拆成 25 个细分 skill（`literature-review` / `citation-management` / `peer-review` / `scientific-schematics` / `latex-posters` / `venue-templates` / `hypothesis-generation` 等），每个 skill 自带 SKILL.md + scripts。
- **skill 清单**（节选自 `gh api .../contents/skills`）：`citation-management`、`clinical-decision-support`（跳过）、`clinical-reports`（跳过）、`document-skills`、`hypothesis-generation`、`infographics`、`latex-posters`、`literature-review`、`market-research-reports`、`paper-2-web`、`parallel-web`、`peer-review`、`poster-presentation`、`pptx-posters`、`research-grants`、`research-lookup`、`scholar-evaluation`、`scientific-critical-thinking`、`scientific-schematics`、`scientific-slides`、`scientific-writing`、`venue-templates`
- **与 D:\note 现有 skill 的差距 / 互补点**：
  1. `literature-review`：D:\note 现有 `obsidian-literature-workflow` 偏 KB 端整理，缺"如何在写论文的 Related Work 章节里串讲 20 篇文献"的视角；`scientific-writer` 版本提供标准化 PRISMA 风格流程，可作子流程挂载。
  2. `citation-management`：D:\note 的 `research/06-tools/` 还没沉淀 Zotero ↔ Obsidian 工作流；这个 skill 自带 BibTeX 检查 / 重复条目清理 / APA/MLA/Chicago 格式化逻辑。
  3. `scientific-schematics`：科研示意图（架构图 / 训练流程图）的 SVG 模板；D:\note 当前走 `diagram-drawio-suite`，两者不冲突，可作为"草图阶段"前置工具。
  4. `peer-review`：审稿意见分类与回复策略的 prompt 模板，**直接补 `review-response` skill 的输入侧**。
  5. `venue-templates`：CCF-A 类会议模板目录，**直接补 `latex-conference-template-organizer` 的目录**。
- **集成建议**：选 4 个 skill 整段移植：`literature-review` / `citation-management` / `peer-review` / `venue-templates`。跳过所有 `clinical-*`（方向不对口）。移植时把路径从 `K-Dense-AI/claude-scientific-writer/.claude/skills/<x>/SKILL.md` 拷到 `D:\note\.claude\skills\<x>/SKILL.md`，保留 `scripts/` 子目录。风险评估：中（量大、需逐个校对 prompt 风格与本仓不一致处）。
- **优先级**：**P0（必装，挑 4 个）**

### 3. AgriciDaniel/claude-obsidian
- **GitHub URL**：https://github.com/AgriciDaniel/claude-obsidian
- **Star**：8,726 / Forks：994 / 最近 push：2026-05-28 / 最近 update：2026-07-05
- **维护活跃度**：中高
- **核心功能**：自称"Self-Organizing AI Second Brain for Obsidian + Claude Code"，把 Karpathy 的 LLM Wiki 模式工程化，提供 15 个 wiki-* skill 让 Claude 在 vault 内自主摄入 / 整理 / 查询 / 维护笔记。
- **skill 清单**（来自 `gh api .../contents/skills`）：`autoresearch`、`canvas`、`defuddle`、`obsidian-bases`、`obsidian-markdown`、`save`、`think`、`wiki-cli`、`wiki-fold`、`wiki-ingest`、`wiki-lint`、`wiki-mode`、`wiki-query`、`wiki-retrieve`、`wiki`
- **与 D:\note 现有 skill 的差距 / 互补点**：
  1. **架构对齐**：CLAUDE.md 第 1 层 raw / 第 2 层 wiki / 第 3 层 research 与该项目的"raw/wiki/digest/identity"几乎一一对应；但**该项目的实现是 Python hooks + 多个 micro-skill 组合**，D:\note 是单 skill 大 prompt，**借鉴其"小 skill 编排"思路可让单次上下文占用下降 40%+**。
  2. **wiki-fold / wiki-lint**：D:\note 现有 `lint-wiki` 是单次大检查；fold 思路是"识别可合并 / 折叠 / 拆分的页面"，粒度更细。
  3. **autoresearch**：当用户提出新主题时，自动多轮检索 → 汇总 → 写笔记，正是 D:\note 缺的能力（`note-research` 是一次性的）。
  4. **wiki-mode / wiki-retrieve**：作为 Claude 系统提示切换"Wiki 上下文模式"的开关，D:\note 没有这种"模式"概念，可考虑在 CLAUDE.md 里加入"wiki mode"段。
  5. **think / save**：通用思考与保存子 skill，与 `remember` skill 思路重叠但更轻量，可作为 `remember` 的 fallback。
- **集成建议**：先 fork 整个仓库到 `D:\note\_external\claude-obsidian`，逐个 review 15 个 SKILL.md；**不直接拷 SKILL.md，而是把每个 skill 的核心触发条件与 prompts 模式摘录到 `D:\note\.claude\skills\wiki-ops-meta/` 下作为"操作原语库"**。风险评估：中（必须人工校验 prompt 质量，避免把"自维护"的过度自主特性带进 D:\note 的"人主导"路线）。
- **优先级**：**P0（必装，需谨慎）**

### 4. wanshuiyin/Auto-claude-code-research-in-sleep (ARIS)
- **GitHub URL**：https://github.com/wanshuiyin/Auto-claude-code-research-in-sleep
- **Star**：13,004 / Forks：1,178 / 最近 push：2026-07-05（今天）/ 最近 update：2026-07-05
- **维护活跃度**：极高（当日提交，单一作者但节奏猛）
- **核心功能**：自称 "Lightweight Markdown-only skills for autonomous ML research"，提供 30+ skill，覆盖 idea discovery / experiment planning / paper writing / cross-model review loop / experiment automation。
- **skill 清单**（节选）：`ablation-planner`、`alphaxiv`、`analyze-results`、`arxiv`、`auto-paper-improvement-loop`、`auto-review-loop-llm`、`auto-review-loop-minimax`、`auto-review-loop`、`citation-audit`、`claims-drafting`、`comm-lit-review`、`deepxiv`、`dse-loop`、`embodiment-description`、`exa-search`、`experiment-audit`、`experiment-bridge`、`experiment-plan`、`experiment-queue`、`feishu-notify`、`figure-description`、`figure-spec`、`formula-derivation`、`gemini-search`、`grant-proposal`、`idea-creator`、`idea-discovery-robot`、`idea-discovery`、`interview-cheatsheet`、`invention-structuring`
- **与 D:\note 现有 skill 的差距 / 互补点**：
  1. **idea-discovery / idea-discovery-robot**：D:\note 的 `top-tier-paper-workflow` S0 是定题阶段，但缺"自动发现 idea"的子 skill；ARIS 的实现把 arxiv + semantic-scholar + OpenReview 三源融合，可作为 `daily-paper-generator` 的"高优 idea"特化。
  2. **citation-audit**：审稿场景下"伪造引用检测"——这是 Claude 出名的痛点；与 `review-response` 互补，组成"审稿前自检 + 审稿后回复"完整闭环。
  3. **auto-review-loop-minimax / auto-review-loop-llm**：跨模型 review loop，让主模型写 + 副模型审；本仓 `MiniMax-M3` 路线（规则要求所有代理同模型）下不可直接用，但**思路可借鉴**——可以拆出"用同一个模型不同温度做 self-review"的简化版。
  4. **experiment-plan / ablation-planner**：D:\note `top-tier-paper-workflow` S4 阶段只有原则，缺模板；这两个 skill 直接给可执行的实验矩阵生成 prompt。
  5. **comm-lit-review**：community-detection 风格的 literature review，区别于纯关键词检索；可作为 `obsidian-literature-workflow` 的"网络视角"补充。
- **集成建议**：移植 4 个最相关 skill：`idea-discovery`、`citation-audit`、`experiment-plan`、`ablation-planner`。复制 SKILL.md 到 `D:\note\.claude\skills\aris-imported/<x>/SKILL.md`，保留其原始 prompt 但把触发关键词中文化（"科研定题"、"引用审计"、"实验规划"、"消融设计"）。风险评估：中（量大，单一作者风险，但 prompt 质量肉眼可见高于平均）。
- **优先级**：**P0（必装，挑 4 个）**

### 5. Master-cai/Research-Paper-Writing-Skills
- **GitHub URL**：https://github.com/Master-cai/Research-Paper-Writing-Skills
- **Star**：4,739 / Forks：241 / 最近 push：2026-06-23 / 最近 update：2026-07-05
- **维护活跃度**：高（每周提交节奏）
- **核心功能**：把 Prof. Peng Sida 在清华的 ML 课程写作笔记系统化为 Claude Code / Codex / Gemini CLI 可读的 skill pack，覆盖 ML/CV/NLP 顶会论文从 Abstract 到 Conclusion 的分章节写作 prompt。
- **skill 清单**（来自 `gh api .../contents`）：仅一个 `research-paper-writing/` 主目录，结构化存储各章节模板
- **与 D:\note 现有 skill 的差距 / 互补点**：
  1. **分章节 prompt 模板**：D:\note 现有 `ml-paper-writing` 偏方法论与引用规则，缺"逐章 prompt 模板"；该 skill 的 `01-abstract.md` / `02-introduction.md` / `03-related-work.md` / `04-method.md` / `05-experiment.md` / `06-conclusion.md` 正是 S6 阶段所需的"半成品 prompt"。
  2. **学术表达 prompt**：用具体 case study 教 Claude 区分"we propose" / "we develop" / "we present"的适用场景，对顶会投稿的语气校准直接有用。
  3. **与 D:\note `research-paper-writing-upstream` 的关系**：本仓 upstream skill 已有 23 个 annotated example，**而 Master-cai 提供的是"模板 + 触发条件"**，两者互补：upstream 给"怎么写好"、Master-cai 给"什么时候调用哪个模板"。
  4. **双语版本**：README 同时含中文版与英文版，对中文用户友好。
  5. **质量门槛**：Star 4.7k + Fork 241 + 持续维护 = 上游信号 OK，可信。
- **集成建议**：把 `research-paper-writing/` 整目录复制到 `D:\note\.claude\skills\paper-writing-templates/`，再在 `ml-paper-writing` 的 SKILL.md 增加"调用 paper-writing-templates 的子流程"段落。风险评估：低（纯文档，无外部依赖）。
- **优先级**：**P0（必装）**

---

## 推荐参考改进（P1）

### 6. anthropics/skills（**官方范例库**）
- **GitHub URL**：https://github.com/anthropics/skills
- **Star**：158,309 / Forks：18,664 / 最近 push：2026-07-01 / 最近 update：2026-07-05
- **维护活跃度**：极高（Anthropic 官方）
- **核心功能**：Anthropic 官方维护的 Agent Skills 公共仓库，包含 17 个示范 skill（`pdf` / `docx` / `pptx` / `xlsx` / `frontend-design` / `skill-creator` 等），展示 SKILL.md 最佳实践。
- **参考价值**：所有自写 skill 的 SKILL.md 格式应向这里看齐；尤其是 `pdf` 与 `skill-creator` 两个 skill 包含完整的 SKILL.md 编写规范与渐进式披露（progressive disclosure）示例。
- **集成建议**：不直接复制 skill，而是把本仓所有 SKILL.md 重新 review 一遍，确保 frontmatter / 触发关键词 / 渐进式披露结构与 anthropics 标准一致。**值得专门跑一次 `/skill-evolve` 来对齐格式**。
- **优先级**：P1（参考改进）

### 7. alirezarezvani/claude-skills（**聚合 skill 库**）
- **GitHub URL**：https://github.com/alirezarezvani/claude-skills
- **Star**：20,281 / Forks：2,773 / 最近 push：2026-07-03 / 最近 update：2026-07-05
- **核心功能**：337 个跨工具（Claude Code / Codex / Gemini CLI / Cursor 等）的 skill 与 plugin，覆盖工程、市场、产品、合规、CEO 咨询、科研等领域。
- **参考价值**：当成"skill 目录"查询入口；具体到本仓相关，只有"research" 分类的 5-8 个子 skill 有价值，需要时按需检索。
- **集成建议**：不整库克隆；通过 `gh search code "research" --repo alirezarezvani/claude-skills` 按需拉。
- **优先级**：P1（参考）

### 8. trapoom555/claude-paperloom
- **GitHub URL**：https://github.com/trapoom555/claude-paperloom
- **Star**：92 / Forks：10 / 最近 push：2026-04-28 / 最近 update：2026-06-28
- **核心功能**：Claude Code Plugin 形式的"自维护研究知识图谱"，定期扫描 vault 内的论文笔记、识别新论文、补充交叉引用。
- **参考价值**：与 D:\note 的 `obsidian-literature-workflow` 高度重叠，但**自动调度**机制比 D:\note 当前的"按需触发"更激进。
- **集成建议**：借鉴其"周期性后台任务"概念，写入 `D:\note\.claude\hooks\scheduled-literature-scan.js`。
- **优先级**：P1（参考）

### 9. juliye2025/evil-read-arxiv
- **GitHub URL**：https://github.com/juliye2025/evil-read-arxiv
- **Star**：1,478 / Forks：151 / 最近 push：2026-06-01 / 最近 update：2026-07-05
- **核心功能**：中文 arxiv 论文阅读链路 Claude Code + Obsidian，提供 `paper-analyze` / `paper-search` / `extract-paper-images` / `start-my-day` 四个子模块。
- **参考价值**：中文用户场景的"接地气"实现，与 D:\note 的 `paper-note` skill 直接对位；`extract-paper-images` 用 PyMuPDF 把图抠出来嵌入 Obsidian 的能力是 D:\note 缺的。
- **集成建议**：把 `extract-paper-images` 子模块逻辑（PyMuPDF 脚本 + frontmatter 注入）移植进 `D:\note\.claude\skills\paper-note/scripts/`。
- **优先级**：P1（参考）

### 10. chaenmasahiro0425/exbrain
- **GitHub URL**：https://github.com/chaenmasahiro0425/exbrain
- **Star**：43 / Forks：6 / 最近 push：2026-07-05（今天）/ 最近 update：2026-07-05
- **核心功能**：日文项目，把 Claude Code × Obsidian × 4 层架构（`raw` / `wiki` / `digest` / `identity`）+ 夜间 compile + 每周 lint 工程化。
- **参考价值**：**4 层架构与 D:\note 完全同构**（D:\note 是 raw / wiki / research / news），但**多了一层 identity**——把"用户的偏好与身份"独立成层。这个思路对 `remember` skill 增强有借鉴意义。
- **集成建议**：把 "identity" 概念引入 D:\note 的 CLAUDE.md 或新建 `D:\note\.claude\identity\`，用 `remember` skill 维护。
- **优先级**：P1（参考）

### 11. Abilityai/cornelius
- **GitHub URL**：https://github.com/Abilityai/cornelius
- **Star**：96 / Forks：28 / 最近 push：2026-06-30 / 最近 update：2026-06-30
- **核心功能**：AI 第二大脑模板（Claude Code + Obsidian），提供预配置的 vault 与多个 AI agent（研究 / 写作 / 总结）。
- **参考价值**：vault 结构可直接复用，尤其是 `_templates/` 的几个模板。
- **集成建议**：clone 后只取 `_templates/` 目录，对比 `D:\note\_templates\` 看缺什么。
- **优先级**：P1（参考）

### 12. ArtemXTech/claude-code-obsidian-starter
- **GitHub URL**：https://github.com/ArtemXTech/claude-code-obsidian-starter
- **Star**：206 / Forks：45 / 最近 push：2026-01-23 / 最近 update：2026-07-02
- **核心功能**：Claude Code + Obsidian starter kit，预配置 vault + skill for projects / tasks / clients / daily routines。
- **参考价值**：starter kit 的"开箱即用"思路比 D:\note 当前对新手不友好的初始设置更友好。
- **集成建议**：参考其 `QUICKSTART.md` 写法，给 D:\note 写一份"新人 5 分钟上手指南"。
- **优先级**：P1（参考）

### 13. assafelovic/gpt-researcher（**架构参考**）
- **GitHub URL**：https://github.com/assafelovic/gpt-researcher
- **Star**：28,076 / Forks：3,788 / 最近 push：2026-07-04 / 最近 update：2026-07-05
- **核心功能**：自主深度研究 agent，多源检索 → 综合 → 生成带引用报告。
- **参考价值**：不是 skill，是独立 Python 工具；但其**多 agent 协作架构**（Planner / Researcher / Writer / Reviser）可作为 D:\note 子代理拆分的范本。
- **集成建议**：不集成；写到 `D:\note\research\06-tools\research-agent-landscape.md` 当作外部工具地图条目。
- **优先级**：P1（参考，架构）

### 14. stanford-oval/storm（**架构参考**）
- **GitHub URL**：https://github.com/stanford-oval/storm
- **Star**：29,829 / Forks：2,790 / 最近 push：2025-09-30 / 最近 update：2026-07-05
- **核心功能**：LLM 驱动的知识策展系统，自动调研主题并生成带引用长报告。
- **参考价值**：与 `gpt-researcher` 同档，但**学术导向更强**；其 "pre-writing → outline → drafting → revision" 流水线与 `ml-paper-writing` 的 S6 流程呼应。
- **集成建议**：不集成；其流水线文档可摘录到 `D:\note\research\06-tools\`。
- **优先级**：P1（参考，架构）

### 15. langchain-ai/open_deep_research
- **GitHub URL**：https://github.com/langchain-ai/open_deep_research
- **Star**：11,930 / Forks：1,700 / 最近 push：2026-06-26 / 最近 update：2026-07-05
- **核心功能**：LangChain 版本的开放深度研究 agent。
- **参考价值**：LangGraph 多 agent 编排的可参考实现。
- **集成建议**：仅参考编排模式，不集成。
- **优先级**：P1（参考）

---

## 不推荐 / 仅供参考

### 16. Rlin1027/philosophy-research-agents
- **GitHub URL**：https://github.com/Rlin1027/philosophy-research-agents
- **Star**：2 / Forks：1 / 最近 push：2026-02-27
- **不推荐原因**：哲学 / 人文学科专用，与 D:\note 的 CS / ML 路线不对口；维护停滞近 4 个月。
- **优先级**：P2

### 17. K-Dense-AI/claude-scientific-writer 的 clinical-* 子包
- **包含**：`clinical-decision-support` / `clinical-reports` / `treatment-plans`
- **不推荐原因**：临床医学场景，D:\note 是 CS 路线。
- **优先级**：P2（跳过即可）

### 18. summarizepaper/summarizepaper
- **GitHub URL**：https://github.com/summarizepaper/summarizepaper
- **Star**：308 / Forks：未知
- **不推荐原因**：是 Web App 而非 skill；不构成可复用 prompt / 流程资产。
- **优先级**：P2

### 19. 大量 0-star fork：`literature-review-ai` / `Paper2Code` / `paper-research`
- **示例**：`joneshong-skills/paper-research`（0★，2026-03 后无提交）、`sanjay-ramesh94/literature-review-ai`（0★）、`silvaxxx1/PAPER2CODE`（8★）
- **不推荐原因**：0 star + 单一提交 + 缺乏 README 详述，无质量信号。
- **优先级**：P2（直接跳过）

### 20. going-doer/Paper2Code 与 PrathamLearnsToCode/paper2code
- **GitHub URL**：
  - https://github.com/going-doer/Paper2Code（4,778★）
  - https://github.com/PrathamLearnsToCode/paper2code（1,436★）
- **不推荐原因**：star 数较高但都属于"论文 → 代码生成"独立工具，与 D:\note 的"论文 → 学习笔记"路线不重叠；D:\note 用户当前阶段不在 ML 系统实现（更偏阅读 / 综述 / 写作），无需集成。
- **优先级**：P2

---

## 对 D:\note 当前 skill 体系的改进建议

> 以下 8 条按"执行优先级"排序；前 3 条为 P0 必做，4-6 条为 P1 应做，7-8 条为 P2 可做。

### 建议 1（必做）：引入 kepano/obsidian-skills 的 `obsidian-cli` + `json-canvas`
**理由**：D:\note 已经依赖 Obsidian，但所有 skill 都假设通过文件读写操作 vault，没用到 Obsidian CLI。补上后 `obsidian-kb-artifacts` 能调用 `obsidian` 二进制完成 canvas 校验 / vault 索引 / backlink 查询；`json-canvas` skill 让 `Maps/literature.canvas` 从占位文件变成真实可视化。

**具体步骤**：
1. `git clone https://github.com/kepano/obsidian-skills D:\note\_external\obsidian-skills`
2. 把 `_external/obsidian-skills/skills/obsidian-cli/` 拷到 `D:\note\.claude\skills\obsidian-cli/`
3. 把 `_external/obsidian-skills/skills/json-canvas/` 拷到 `D:\note\.claude\skills\json-canvas/`
4. 在 `obsidian-kb-artifacts/SKILL.md` 增加 "Obsidian CLI 兜底" 段，调用时优先 CLI、文件操作作 fallback
5. 跑 `kb-map` 验证 canvas 生成

### 建议 2（必做）：从 K-Dense-AI/claude-scientific-writer 移植 4 个核心 skill
**理由**：literature-review / citation-management / peer-review / venue-templates 四个 skill 与 D:\note 现有 `obsidian-literature-workflow` / `review-response` / `latex-conference-template-organizer` 三个 skill 形成 1:1 增强关系。

**具体步骤**：
1. `git clone https://github.com/K-Dense-AI/claude-scientific-writer D:\note\_external\csw`
2. 复制 4 个目录到 `D:\note\.claude\skills\csw-imported/`
3. 中文触发关键词补全（"文献综述" / "引用管理" / "审稿回复" / "会议模板"）
4. 在对应上游 skill 的 SKILL.md 增加"调用 csw-imported/<x>"的子流程
5. 用 `lint-wiki` 跑一次自检确保不破坏现有 frontmatter 约束

### 建议 3（必做）：从 ARIS 引入 idea-discovery / citation-audit / experiment-plan
**理由**：ARIS 是 2026 年活跃度最高的 ML 科研 skill 仓库，单一作者 wanshuiyin 的 prompt 质量肉眼可见高于平均；其中 `idea-discovery` 直接补 `top-tier-paper-workflow` S0 阶段的"idea 自动发现"缺口，`citation-audit` 解决 Claude 出名的"伪造引用"痛点，`experiment-plan` 给出可复制的实验矩阵 prompt。

**具体步骤**：
1. `git clone https://github.com/wanshuiyin/Auto-claude-code-research-in-sleep D:\note\_external\aris`
2. 复制 `skills/idea-discovery/` / `skills/citation-audit/` / `skills/experiment-plan/` 到 `D:\note\.claude\skills\aris-imported/`
3. 把 `idea-discovery` 在 `top-tier-paper-workflow/SKILL.md` S0 阶段作为"辅助输入"调用
4. 把 `citation-audit` 在 `ml-paper-writing/SKILL.md` 末尾"投递前自检"段调用

### 建议 4（应做）：借鉴 AgriciDaniel/claude-obsidian 的"小 skill 编排"思路重写当前大 skill
**理由**：D:\note 现有 skill（如 `top-tier-paper-workflow` / `obsidian-project-kb-core`）都是单个大 SKILL.md，每次激活占用大量上下文。claude-obsidian 用 14 个 wiki-* 小 skill 配合实现同一目标，**单次 prompt token 消耗可下降 40%+**。

**具体步骤**：
1. fork claude-obsidian，逐一 review 15 个 SKILL.md
2. 抽离出"操作原语"（如 `wiki-ingest` ≈ D:\note 的 `kb-ingest` 的子步骤）
3. 把 `top-tier-paper-workflow` 拆成 `tt-s0-triage` / `tt-s1-lit` / `tt-s2-gap` / ... / `tt-s9-ship` 共 10 个小 skill
4. 由总入口 `top-tier-paper-workflow` 做路由分发

### 建议 5（应做）：从 Master-cai 引入"分章节写作 prompt 模板"
**理由**：D:\note 现有 `ml-paper-writing` 缺"逐章 prompt"——给到 Claude "写 Introduction 章节"时缺乏半成品模板。Master-cai 的 `research-paper-writing/` 提供 6 个分章节 prompt，正好补缺。

**具体步骤**：
1. `git clone https://github.com/Master-cai/Research-Paper-Writing-Skills D:\note\_external\paper-templates`
2. 拷贝到 `D:\note\.claude\skills\paper-writing-templates/`
3. 在 `ml-paper-writing/SKILL.md` 增加"调用 paper-writing-templates/<章节>"的子流程
4. 验证：跑一次"生成 v0.1 Introduction 草稿"端到端

### 建议 6（应做）：从 evil-read-arxiv 移植 `extract-paper-images` 子模块
**理由**：D:\note 的 `paper-note` skill 把论文翻译成 Markdown 但**没处理图片**——很多论文的核心结果在 figure 里，抠不出来。evil-read-arxiv 用 PyMuPDF 把图片导出到 `assets/papers/<arxiv-id>/` 并自动在 frontmatter 加 `figures:` 字段。

**具体步骤**：
1. `git clone https://github.com/juliye2025/evil-read-arxiv D:\note\_external\evil-read`
2. 提取 `paper-analyze/scripts/` 中的 `extract_images.py`
3. 拷到 `D:\note\.claude\skills\paper-note/scripts/extract_images.py`
4. 在 `paper-note/SKILL.md` 增加"图像抽取与归档"段

### 建议 7（可做）：借鉴 exbrain 的"identity 层"概念扩展 remember skill
**理由**：D:\note 当前 `remember` skill 只记录 preferences；exbrain 用 identity 层独立管理"用户身份 / 研究身份 / 当前阶段"等元信息，可让所有 skill 在 prompt 开头就拿到一致身份上下文。

**具体步骤**：
1. 新建 `D:\note\.claude\identity\` 目录
2. 写 `identity/profile.md`（用户基本背景） / `identity/research-stage.md`（当前研究阶段） / `identity/preferences.md`（已有，与全局 preferences.md 同步）
3. 在 `remember` skill 增加 "更新 identity/<x>" 子命令
4. 在所有 skill 的 SKILL.md 增加 "读取 identity/" 段

### 建议 8（可做）：用 gpt-researcher / STORM / open_deep_research 的多 agent 架构重审 `top-tier-paper-workflow`
**理由**：这三个项目的共性是**多 agent 流水线**（Planner / Researcher / Writer / Reviser），与 D:\note 当前的"主代理 + 子代理分发"模式相比，把"角色"和"步骤"明确分离能减少 prompt 歧义。

**具体步骤**：
1. 写 `D:\note\research\06-tools\research-agent-landscape.md`，记录 gpt-researcher / STORM / open_deep_research / claude-obsidian 的多 agent 模式
2. 在 `top-tier-paper-workflow/SKILL.md` 增加"借鉴 STORM 的 pre-writing → outline → draft → revise 模式"附录
3. 不强制改写流程，但记录为"未来 S11 改造方向"

---

## 调研方法

### 使用的搜索关键词（web-search-prime）

- `awesome claude code skills github list repository`
- `claude code skill paper reading obsidian notes`
- `claude code literature review research assistant workflow`
- `claude code skill academic paper writing arxiv summarization`
- `zettelkasten obsidian note-taking automation script claude`
- `open source research assistant gpt-researcher storm open-researcher github`
- `"claude code" skill zotero citation management bibliography`

### 使用的 gh 命令

```bash
# 仓库元数据查询（取 star / fork / update / pushed_at）
gh api repos/{owner}/{repo} --jq '{name, full_name, stars: .stargazers_count, forks: .forks_count, updated_at, pushed_at, description, archived}'

# skill 子目录列表
gh api repos/{owner}/{repo}/contents/skills --jq '.[] | select(.type=="dir") | .name'

# README 摘要
gh api repos/{owner}/{repo}/readme --jq '.content' | python -c "import sys, base64; print(base64.b64decode(sys.stdin.read()).decode('utf-8', errors='ignore')[:3000])"

# 主题搜索
gh search repos "claude code obsidian" --limit 15 --json name,fullName,stargazersCount,updatedAt,description
gh search repos "claude code skill paper" --limit 15 --json name,fullName,stargazersCount,updatedAt,description
gh search repos "arxiv paper summarizer" --limit 10 --json name,fullName,stargazersCount,updatedAt,description
gh search repos "literature review AI" --limit 10 --json name,fullName,stargazersCount,updatedAt,description
gh search repos "claude code plugin academic research" --limit 10 --json name,fullName,stargazersCount,updatedAt,description
gh search repos "Paper2Code" --limit 8 --json name,fullName,stargazersCount,updatedAt,description
gh search repos "STORM knowledge curation academic" --limit 10 --json name,fullName,stargazersCount,updatedAt,description
```

### 数据来源

- **GitHub REST API**（经 `gh` CLI，所有 star 数与日期为 2026-07-05 当日实时快照）
- **web-search-prime MCP**（搜索引擎层面验证活跃度、用户评价、社区讨论）
- **各 repo 的 SKILL.md / README 直接读取**（验证 skill 实际内容，不只看 star）

---

## 引用源（GitHub URLs 与文档）

### P0（推荐直接集成）
1. https://github.com/kepano/obsidian-skills — Obsidian 官方维护的 agent skills（39.7k★）
2. https://github.com/K-Dense-AI/claude-scientific-writer — 科研写作 25 个子 skills（2.0k★）
3. https://github.com/AgriciDaniel/claude-obsidian — LLM Wiki 模式工程化实现（8.7k★）
4. https://github.com/wanshuiyin/Auto-claude-code-research-in-sleep — ARIS 自主科研 skill pack（13.0k★）
5. https://github.com/Master-cai/Research-Paper-Writing-Skills — ML 论文分章节写作模板（4.7k★）

### P1（参考改进）
6. https://github.com/anthropics/skills — Anthropic 官方 skills 范例库（158k★）
7. https://github.com/alirezarezvani/claude-skills — 337 跨工具 skill 聚合（20.2k★）
8. https://github.com/trapoom555/claude-paperloom — 自维护研究知识图谱（92★）
9. https://github.com/juliye2025/evil-read-arxiv — 中文 arxiv 阅读链路（1.5k★）
10. https://github.com/chaenmasahiro0425/exbrain — 4 层架构（raw/wiki/digest/identity）（43★）
11. https://github.com/Abilityai/cornelius — AI 第二大脑模板（96★）
12. https://github.com/ArtemXTech/claude-code-obsidian-starter — Claude Code + Obsidian starter（206★）
13. https://github.com/assafelovic/gpt-researcher — 自主深度研究 agent（28.0k★）
14. https://github.com/stanford-oval/storm — 知识策展长报告生成（29.8k★）
15. https://github.com/langchain-ai/open_deep_research — LangChain 深度研究（11.9k★）

### P2（不推荐 / 跳过）
16. https://github.com/Rlin1027/philosophy-research-agents — 哲学专用（2★，方向不对口）
17. https://github.com/summarizepaper/summarizepaper — Web App 而非 skill（308★）
18. https://github.com/going-doer/Paper2Code — 论文→代码独立工具（4.8k★，与笔记路线不重叠）
19. https://github.com/PrathamLearnsToCode/paper2code — 同上（1.4k★）
20. https://github.com/joneshong-skills/paper-research — 0★，停滞

### 补充文档链接
21. https://github.com/travisvn/awesome-claude-skills — awesome-claude-skills 聚合目录（13.9k★）
22. https://github.com/BehiSecc/awesome-claude-skills — 另一份 awesome 聚合（9.7k★）
23. https://platform.claude.com/docs/en/build-with-claude/citations — Claude API Citations 文档
24. https://docs.tavily.com/examples/open-sources/gpt-researcher — GPT Researcher 架构文档

---

## 附录：本调研的执行限制

- **本次仅做仓库与 skill 级别的横评**，未对每个 SKILL.md 做完整的 prompt 质量 A/B 测试
- **star 数与 push 日期为 2026-07-05 当日快照**，未来重新评估请以 API 实时数据为准
- **集成 P0 列表中的 skill 时，必须人工逐个 review prompt**——自动 trust 上游 skill 可能引入不一致的 prompt 风格或与 D:\note 现有约束冲突
- **未尝试本地 clone 后跑通端到端流程**——这是建议 1-3 执行时必做的验证步骤