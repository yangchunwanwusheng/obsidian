---
type: tool-guide
tags: [科研工具, 工具策略]
created: 2026-04-18
updated: 2026-04-18
---

# 科研工具使用策略

> 一站式科研工具选择与使用指南——按研究阶段匹配最佳工具，避免工具泛滥，聚焦核心效率。

## 工具矩阵

下面是按研究阶段 × 功能分类的工具选择矩阵，帮助你在每个阶段快速定位需要的工具。

| 阶段 | 文献发现 | 论文阅读 | 笔记管理 | 代码实验 | 论文写作 | 格式排版 |
|------|----------|----------|----------|----------|----------|----------|
| **选题** | Semantic Scholar, ResearchRabbit, Connected Papers | ChatPaper, Elicit | Obsidian | - | - | - |
| **文献综述** | ScholarAI, Papers with Code | PaperQA2, Elicit | Obsidian + Zotero | - | STORM | - |
| **方法设计** | - | - | Obsidian | Claude Code, PyTorch | - | draw.io |
| **实验** | - | - | Obsidian 日志 | Claude Code, PandasAI | - | matplotlib |
| **写作** | - | - | Obsidian | - | GPT Academic, Writefull | LaTeX / WPS |
| **投稿** | - | - | - | - | GPT Academic | latexdiff |

> [!tip] 使用原则
> 不要试图一次性学会所有工具。先掌握 Tier 1 的 3 个核心工具，然后按研究阶段需要逐步引入 Tier 2 工具。Tier 3 仅在特定场景下使用。

---

## Tier 1：日常必用（3 个）

这是贯穿整个研究生涯的三个核心工具，每天都会用到。

### 1. Obsidian — 知识库中枢

**功能定位**：个人知识管理系统（PKM），管理你的所有笔记、想法、文献笔记和知识链接。

**使用场景**：
- 日常想法记录（`research/_inbox/`）
- 文献阅读笔记（`research/02-papers/`）
- 实验日志（`research/03-experiments/`）
- 论文草稿（`research/04-writing/`）
- 与 Wiki 知识库双向链接

**核心操作**：
- `[[双链]]`：建立笔记之间的关联，形成知识网络
- `#标签`：快速分类和检索
- Callout 语法（`> [!note]`）：折叠长内容，保持页面整洁
- Dataview 插件：自动聚合和查询笔记（如列出所有未完成的论文笔记）
- Templates 插件：使用 `_templates/` 下的模板快速创建标准化笔记
- Zotero Integration 插件：从 Zotero 导入文献元数据和标注到 Obsidian

**注意事项**：
- 不要在 Obsidian 中存储 PDF 全文，只存笔记和元数据（PDF 由 Zotero 管理）
- 定期使用 `wiki/log.md` 记录操作，保持知识库可追溯
- 文件命名使用英文，内容使用中文

### 2. Zotero — 文献管理

**功能定位**：学术论文的收集、管理、标注和引用工具，与 Obsidian 联动形成完整的文献工作流。

**使用场景**：
- 浏览器一键抓取论文元数据（标题、作者、年份、期刊）
- 管理论文 PDF 全文和阅读标注
- 自动生成 BibTeX 引用格式
- 与 Obsidian 联动，将标注同步到笔记

**核心操作**：
- **安装 Zotero Connector 浏览器插件**：在 Google Scholar、arXiv、Semantic Scholar 等页面一键保存论文
- **Better BibTeX 插件**：自动生成稳定的 citation key，配合 LaTeX 使用
- **Zotero PDF Reader**：内置 PDF 阅读器，直接在 PDF 上高亮和批注
- **Zotfile 插件**（可选）：自动重命名和移动 PDF 文件到指定文件夹
- **Zotero Integration 插件**（Obsidian 端）：将 Zotero 标注导入 Obsidian 笔记

**分类策略**：
```
我的文库/
├── 00-Inbox/          ← 新导入的论文暂存
├── 01-研究方向/        ← 按研究方向分文件夹
│   ├── 目标检测/
│   ├── 语义分割/
│   └── 异常检测/
├── 02-已读/           ← 精读完成的论文
├── 03-待读/           ← 计划阅读的论文
└── 04-参考文献/       ← 写作时引用的论文
```

**注意事项**：
- 设置 Zotero 同步（免费 300MB，可配合 WebDAV 扩展）
- 导入论文后立即打标签（如 `#must-read`、`#survey`、`#baseline`）
- citation key 命名规则建议：`作者年份关键词`（如 `he2017mask`）

### 3. Claude Code — 代码/写作/自动化

**功能定位**：AI 编程助手和科研自动化引擎，覆盖代码编写、调试、数据分析和写作辅助。

**使用场景**：
- 快速复现论文代码（从论文描述到可运行代码）
- 数据预处理和实验脚本编写
- 实验结果分析和可视化
- 论文段落润色和翻译
- LaTeX 排版辅助
- 自动化工具链搭建（如批量下载论文、生成报告）

**核心操作**：
- **代码生成**：描述需求，生成 PyTorch 模型定义、训练循环、数据加载器
- **Debug**：粘贴报错信息，获取修复建议
- **代码解释**：理解开源论文代码的实现逻辑
- **数据分析**：使用 PandasAI 模式，自然语言查询实验数据
- **写作辅助**：段落润色、学术翻译、逻辑检查

**注意事项**：
- 涉及核心创新点的设计时，不要完全依赖 AI——你自己才是创新的主人
- 生成代码后务必理解每一行，不要"复制粘贴就跑"
- 使用 CLAUDE.md 配置项目上下文，让 AI 更好地理解你的研究背景
- 保护敏感数据——不要将未发表的实验数据或创新思路直接发送给公共 API

---

## Tier 2：按需使用（7 个）

### 4. Semantic Scholar API — 论文搜索

**功能定位**：由 Allen AI 研究所开发的免费学术论文搜索引擎，API 支持程序化访问。

**使用场景**：
- 按关键词、作者、主题搜索相关论文
- 查看论文引用关系和影响力指标
- 批量获取论文元数据（标题、摘要、引用数）

**核心操作**：
- **基础搜索**：`https://api.semanticscholar.org/graph/v1/paper/search?query=xxx`
- **获取引用**：`/paper/{paper_id}/citations` — 找到引用某篇论文的所有后续工作
- **获取参考文献**：`/paper/{paper_id}/references` — 找到某篇论文引用的所有前置工作
- **批量查询**：一次最多查询 500 篇论文的元数据

**推荐字段**：`title,abstract,year,citationCount,referenceCount,authors,externalIds`

**注意事项**：
- API 免费但有限流（每秒 1 请求，可申请提升）
- 适合在选题阶段做大规模文献扫描
- 配合 Python 脚本可以自动化搜索和筛选

### 5. ChatPaper — 快速论文摘要

**功能定位**：基于 LLM 的论文速读工具，自动生成论文的三段式摘要（动机/方法/结论）。

**使用场景**：
- 快速判断一篇论文是否值得精读
- 每周扫描 arXiv 新论文时批量生成摘要
- 辅助文献综述的初稿编写

**核心操作**：
- 输入 arXiv 链接或上传 PDF，自动生成结构化摘要
- 支持批量处理多篇论文
- 生成中英文双语摘要

**注意事项**：
- ChatPaper 生成的摘要不能替代精读，只能作为"是否值得精读"的判断依据
- 对于关键基线论文，仍然需要手动精读

### 6. Elicit — 论文分析

**功能定位**：AI 驱动的论文分析平台，可以从大量论文中提取特定信息并汇总。

**使用场景**：
- 文献综述时需要快速了解"某个问题的主流解决方案有哪些"
- 比较多篇论文的实验设置、数据集、指标
- 找到特定子领域的高引用论文

**核心操作**：
- 输入研究问题（如 "What are the main approaches for anomaly detection in industrial images?"）
- Elicit 返回相关论文列表，附带关键信息提取
- 支持自定义提取字段（如"使用了什么数据集"、"SOTA 指标是多少"）

**注意事项**：
- 免费版有每月查询次数限制
- 提取的信息需要人工核实，可能存在幻觉
- 适合作为文献综述的起点，不是终点

### 7. GPT Academic — 论文润色翻译

**功能定位**：为学术写作优化的 AI 工具，支持论文润色、翻译、语法检查。

**使用场景**：
- 将中文初稿翻译为学术英文
- 润色已有的英文段落（提升表达准确性和流畅度）
- 检查语法和拼写错误

**核心操作**：
- **一键润色**：输入英文段落，输出学术化改写
- **中英互译**：保持学术术语的准确性
- **查找语法错误**：逐句标注语法问题
- **自定义 Prompt**：可以配置特定领域的术语表

**注意事项**：
- 翻译后务必人工检查专业术语的准确性
- 不要将整篇论文一次性提交——分段落处理效果更好
- 保留原文和修改版本的对比，方便回退

### 8. STORM — 文献综述草稿

**功能定位**：斯坦福大学开发的自动文献综述生成工具，可以从一个主题出发自动搜索、阅读并撰写综述文章。

**使用场景**：
- 进入新领域时快速获取综述性概述
- 文献综述章节的初稿生成
- 了解某个子领域的全景

**核心操作**：
- 输入研究主题（如 "Vision Transformer for Image Classification"）
- STORM 自动搜索相关论文，生成结构化综述
- 支持在线预览和编辑

**注意事项**：
- 生成的综述只能作为参考框架，不能直接使用
- 需要人工补充最新的论文和自己的分析
- 适合用于构建综述大纲，而非替代文献综述

### 9. PandasAI — 数据分析

**功能定位**：将自然语言查询转化为 Pandas 操作的 AI 工具，适合非程序员快速分析实验数据。

**使用场景**：
- 快速查询实验结果（如"哪个模型在 COCO 上的 mAP 最高？"）
- 生成统计摘要和对比表
- 探索性数据分析（EDA）

**核心操作**：
```python
import pandasai as pai
df = pai.read_csv("experiment_results.csv")
df.chat("按数据集分组，显示每个模型的平均 mAP 和标准差")
df.chat("画出各模型训练 loss 的变化趋势")
```

**注意事项**：
- 复杂的数据分析仍需手动编写 Pandas 代码
- 确保数据格式正确（CSV 列名清晰、无缺失值干扰）
- 适合快速探索，不适合精确的统计检验

### 10. draw.io — 图表绘制

**功能定位**：免费的在线图表绘制工具，适合绘制方法框架图、系统架构图、流程图。

**使用场景**：
- 论文中的 Method Framework 图（方法整体架构）
- 实验流程图
- 系统架构图
- 对比示意图

**核心操作**：
- 使用模板快速开始（选择"流程图"或"网络架构"模板）
- 导出为 PDF 或 SVG（矢量格式，适合论文）
- 支持自定义形状、颜色、箭头样式
- 与 Google Drive / OneDrive 同步

**注意事项**：
- 论文图表需要高分辨率导出（建议 300dpi 以上或矢量格式）
- 配色方案保持简洁（最多 4-5 种颜色）
- 图表中的文字大小要适中（最终印刷后仍然清晰）

---

## Tier 3：特殊场景（10 个）

### 11. PaperQA2

AI 驱动的论文问答系统，可以上传多篇 PDF 并针对内容提问。适合精读某篇论文后进行深度问答，或跨论文对比特定细节。

### 12. ScholarAI

ChatGPT 插件，可以在对话中直接搜索和阅读论文。适合与 ChatGPT 对话式地探索某个研究问题，快速获取论文摘要和关键信息。

### 13. GPT Researcher

自主研究代理，可以自动搜索、阅读、汇总多个来源的信息。适合对某个全新领域做初步调研，生成领域概况报告。

### 14. ResearchRabbit

可视化的论文关系发现工具。输入一篇种子论文，自动展示其引用网络和相关论文。特别适合在选题阶段发现相关论文——找到一个关键论文后，通过引用网络快速扩展到整个领域。

### 15. Connected Papers

类似 ResearchRabbit 的论文图谱工具，以图谱方式展示论文间的引用关系。特别适合理解一个领域的演进脉络——从奠基论文到最新进展的完整路径。

### 16. Writefull

专门为学术写作优化的语言工具，提供实时语言反馈。可以集成到 Overleaf 中使用，实时检查语法、用词和句式。适合在 LaTeX 写作过程中持续获得语言优化建议。

### 17. AI-Researcher

开源的自动化研究工具，可以端到端地完成文献搜索、实验设计和论文撰写。目前仍在快速发展中，适合关注前沿但暂不建议依赖。

### 18. img2img-turbo

图像到图像的快速转换工具，可以用于生成论文中的示意图或可视化结果。适合需要快速将手绘草图转换为精美插图的场景。

### 19. detikzify

将自然语言描述转换为 TikZ/LaTeX 图表代码。适合需要在 LaTeX 中直接绘制专业图表（如神经网络结构图、数据流图）的场景。

### 20. Awesome ChatGPT Prompts

ChatGPT 提示词集合，包含多个学术研究相关的提示词模板。适合学习如何更有效地使用 AI 进行学术写作、文献分析和研究规划。

---

## 按研究阶段的工具组合推荐

### 阶段一：选题（2-4 周）

**核心目标**：确定研究方向，找到有价值的选题。

**工具组合**：
```
Semantic Scholar（搜索论文）
    ↓ 发现相关论文
ResearchRabbit / Connected Papers（扩展论文网络）
    ↓ 理解领域全景
ChatPaper（快速阅读候选论文）
    ↓ 筛选出精读列表
Obsidian（记录选题笔记到 research/01-topics/）
    ↓ 形成选题方案
Claude Code（辅助可行性分析）
```

**工作流**：
1. 在 Semantic Scholar 搜索 2-3 个候选方向的关键词，每个方向收集 10-20 篇论文
2. 用 ResearchRabbit 找到每个方向的核心论文和引用网络
3. 用 ChatPaper 批量生成摘要，筛选出 5-8 篇精读论文
4. 在 Obsidian 中为每个方向创建笔记（`research/01-topics/方向名.md`）
5. 与导师讨论，确定最终选题

### 阶段二：文献综述（3-4 周）

**核心目标**：全面了解选定方向的已有工作，找到研究空白。

**工具组合**：
```
Semantic Scholar API（批量搜索 30+ 篇论文）
    ↓
Zotero（管理所有论文 PDF 和元数据）
    ↓
Elicit / PaperQA2（提取论文关键信息）
    ↓
Obsidian + Zotero Integration（记录阅读笔记）
    ↓
STORM（生成综述初稿框架）
    ↓
Obsidian（人工补充和完善综述）
```

**工作流**：
1. 用 Semantic Scholar API 批量搜索 30+ 篇相关论文
2. 导入 Zotero，按子主题分类（`00-Inbox → 01-研究方向/具体子主题`）
3. 精读核心论文，在 Zotero 中标注，同步到 Obsidian（`research/02-papers/`）
4. 用 STORM 生成综述框架，人工补充和修正
5. 构建论文知识图谱（`research/02-papers/领域名-knowledge-graph.md`）

### 阶段三：方法设计（4-6 周）

**核心目标**：提出创新方法，设计实验方案。

**工具组合**：
```
Obsidian（记录方法设计思路）
    ↓
draw.io（绘制方法框架图）
    ↓
Claude Code（辅助代码实现和 Debug）
    ↓
PyTorch（模型实现和初步验证）
```

**工作流**：
1. 在 Obsidian 中记录方法核心思想和设计动机
2. 用 draw.io 绘制方法框架图（先草图，后精细版）
3. 用 Claude Code 辅助实现核心模块（如自定义 Attention、Loss 函数）
4. 在小数据集上验证想法可行性
5. 迭代修改方法设计

### 阶段四：实验验证（6-10 周）

**核心目标**：设计实验、运行实验、分析结果。

**工具组合**：
```
Claude Code（编写实验脚本、Debug）
    ↓
PyTorch（训练和评估）
    ↓
PandasAI（快速分析实验结果）
    ↓
matplotlib / seaborn（生成实验图表）
    ↓
Obsidian 日志（记录每次实验到 research/03-experiments/）
```

**工作流**：
1. 每次实验前在 Obsidian 日志记录目的和设置
2. 用 Claude Code 编写/修改实验脚本
3. 在 Kaggle / Colab 上运行实验
4. 用 PandasAI 快速分析结果
5. 用 matplotlib 生成标准图表（折线图、对比柱状图、混淆矩阵等）
6. 在 Obsidian 日志中记录结果分析和下一步计划

### 阶段五：论文写作（4-6 周）

**核心目标**：完成论文初稿，反复修改至投稿质量。

**工具组合**：
```
Obsidian（初稿撰写，段落级）
    ↓
GPT Academic / Writefull（润色和翻译）
    ↓
LaTeX / Overleaf（正式排版）
    ↓
Claude Code（辅助 LaTeX 语法和排版问题）
    ↓
draw.io（最终版框架图）
```

**写作顺序**：
1. Method → Experiment → Introduction → Related Work → Abstract → Conclusion
2. 先用 Obsidian 或 LaTeX 写中文初稿（表达核心思想）
3. 用 GPT Academic 翻译为学术英文
4. 在 Overleaf 中排版，用 Writefull 实时检查语言
5. 多轮修改（至少 3 轮）

### 阶段六：投稿修改（2-4 周+）

**核心目标**：投稿、回复审稿意见、修改论文。

**工具组合**：
```
LaTeX（最终排版）
    ↓
latexdiff（标记修改内容）
    ↓
GPT Academic（辅助回复审稿意见）
    ↓
Obsidian（记录审稿意见和修改计划）
```

**工作流**：
1. 检查目标会议/期刊的格式要求
2. 在 Overleaf 中完成最终排版
3. 投稿前自查：格式、引用、图表清晰度、拼写
4. 收到审稿意见后，在 Obsidian 中整理每条意见
5. 用 GPT Academic 辅助起草回复
6. 用 latexdiff 标记所有修改
7. 提交修改稿

---

## 免费基础工具链

以下是全流程免费方案，适合预算有限的学生研究者。

| 功能 | 免费工具 | 替代付费工具 |
|------|----------|-------------|
| 文献搜索 | Semantic Scholar (免费) | Scopus, Web of Science (付费) |
| 文献管理 | Zotero (免费，300MB 同步) | Mendeley, EndNote |
| PDF 阅读 | Zotero PDF Reader (免费) | PDF Expert, GoodNotes |
| 笔记管理 | Obsidian (个人免费) | Notion, Roam Research |
| 论文速读 | ChatPaper (免费) | Elicit Pro (付费) |
| 数据分析 | Python + PandasAI (免费) | SPSS, MATLAB (付费) |
| 代码编写 | VS Code + Claude Code | PyCharm Pro |
| 图表绘制 | matplotlib + draw.io (免费) | Origin, GraphPad |
| 论文排版 | Overleaf (免费版) | LaTeX 本地安装 |
| 语言润色 | GPT Academic (开源免费) | Grammarly Premium |
| GPU 计算 | Kaggle T4 (每周30h) | AWS, GCP (付费) |
| 版本管理 | Git + GitHub (免费) | 无替代 |
| 云同步 | OneDrive/Google Drive (免费额度) | 无替代 |

> [!success] 最小免费工具集
> 如果资源极其有限，只需要这 5 个工具就能完成完整的科研流程：
> 1. **Obsidian** — 笔记和知识管理
> 2. **Zotero** — 文献管理
> 3. **Semantic Scholar** — 论文搜索
> 4. **Overleaf** — LaTeX 排版
> 5. **Kaggle** — GPU 计算资源

---

## 相关页面

- [[research/06-tools/zotero-obsidian-workflow|Zotero+Obsidian联动]]
- [[research/06-tools/codex-research-workflow|Codex自动化流程]]
- [[wiki/concepts/research-toolchain|科研工具链]]
