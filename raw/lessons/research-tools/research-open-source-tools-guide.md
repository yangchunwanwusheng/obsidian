---
type: lesson
tags: [科研工具, 开源工具, 学术写作, 文献管理, 数据分析]
created: 2026-04-18
updated: 2026-04-18
difficulty: beginner
prerequisites: []
topic: 科研开源工具全景指南
status: completed
---

# 科研开源工具全景指南

> 系统整理 25+ 款科研相关开源/免费工具，覆盖学术写作、文献管理、论文阅读、数据分析、图表绘制和自动化研究等全流程。

## 学习目标

- [ ] 了解主流科研开源工具的功能定位和适用场景
- [ ] 能根据自身研究阶段选择合适工具组合
- [ ] 掌握各类工具的核心使用方式

---

## 一、学术写作辅助

### 1. GPT Academic（中科院学术版 GPT）

| 项目 | 内容 |
|------|------|
| **功能定位** | 为 GPT/GLM 等 LLM 提供学术优化的交互界面，一键完成论文润色、翻译、代码解释等 |
| **GitHub** | https://github.com/binary-husky/gpt_academic |
| **费用** | 开源免费（需自备 API Key） |
| **Stars** | 60k+ |

**核心功能**：
- 一键论文润色/语法检查/中英互译
- PDF/LaTeX 论文全文翻译与总结
- Python/C++ 项目代码解析与自译解
- 多 LLM 模型并行问询（GPT-4、ChatGLM、通义千问等）
- 自定义快捷按钮和函数插件
- ArXiv 论文一键翻译摘要+下载 PDF
- 谷歌学术统合助手，自动生成 Related Works

**科研流程中的使用**：
- 论文写作阶段：润色英文表达、中英互译
- 论文阅读阶段：翻译 ArXiv 论文、总结 PDF 全文
- 代码研究阶段：解析开源项目代码

**安装方式**：
```bash
git clone --depth=1 https://github.com/binary-husky/gpt_academic.git
cd gpt_academic
pip install -r requirements.txt
# 配置 config_private.py 中的 API Key
python main.py
```

---

### 2. STORM（斯坦福开源写作工具）

| 项目 | 内容 |
|------|------|
| **功能定位** | LLM 驱动的长文写作助手，自动搜集资料、生成大纲、模拟专家对话，输出维基级结构化长文 |
| **GitHub** | https://github.com/stanford-kg/storm |
| **官网** | https://storm.genie.stanford.edu |
| **费用** | 开源免费（需 API Key） |
| **Stars** | 22.5k+ |

**核心功能**：
- 输入主题即可自动深挖资料，多角度收集参考信息
- 自动生成详细大纲
- 模拟专家问答对话，逐步完善内容
- 附带引用支持，可一键导出 PDF
- 支持多模型协作（GPT-4 / Claude）
- 支持与本地知识库和 Zotero 联动
- 可一键导出 LaTeX 格式

**科研流程中的使用**：
- 文献综述撰写：输入研究主题，自动生成结构化综述
- 论文 Introduction/Related Work 草稿生成
- 研究报告撰写

**注意**：目前主要支持英文输出，暂不原生支持中文。

---

### 3. Writefull

| 项目 | 内容 |
|------|------|
| **功能定位** | 专为学术写作训练的 AI 润色工具，基于 Springer Nature 期刊全文数据库训练 |
| **官网** | https://www.writefull.com |
| **费用** | 基础版免费；高级版付费 |

**核心功能**：
- 基于 AI 的学术语言校正
- 句子补全和改写建议
- 语言搜索（检查短语在学术文献中的使用频率）
- Title Generator（根据摘要生成标题）
- Paraphraser（句子重写）
- 提供 Overleaf 插件和 Word 插件

**科研流程中的使用**：
- 论文写作完成后进行语言润色
- 检查学术表达的地道性
- 生成合适的论文标题

---

### 4. Grammarly

| 项目 | 内容 |
|------|------|
| **功能定位** | 广泛使用的英文语法检查与语言润色工具 |
| **官网** | https://www.grammarly.com |
| **费用** | 基础版免费；Premium 版付费（约 $12/月） |

**核心功能**：
- 实时语法、拼写、标点检查
- 句子结构优化建议
- 语气和风格调整
- 学术写作模式（Premium）
- 查重功能（Premium）
- 支持 Word 插件、Chrome 扩展、独立客户端

**科研流程中的使用**：
- 论文写作全过程的语法检查
- 投稿前的最终润色

---

## 二、论文阅读与研究工具

### 5. ChatPaper

| 项目 | 内容 |
|------|------|
| **功能定位** | 开源论文速读总结工具，基于 AI 加速论文阅读和 ArXiv 论文筛选 |
| **GitHub** | https://github.com/kaushiktrivedi/ChatPaper（原版）；多个衍生版本 |
| **费用** | 开源免费（需 API Key） |

**核心功能**：
- 根据关键词自动在 ArXiv 下载最新论文
- 利用 ChatGPT 将论文总结为固定格式
- 一分钟阅读 AI 总结，快速判断论文价值
- 支持本地 PDF 文档直接处理
- 论文优缺点分析与改进建议
- 审稿回复辅助

**科研流程中的使用**：
- 每日 ArXiv 论文筛选：快速浏览大量论文摘要
- 精读前的快速预览：决定哪些论文值得精读
- 论文阅读后的要点回顾

---

### 6. Elicit

| 项目 | 内容 |
|------|------|
| **功能定位** | AI 研究助手，利用语义搜索帮助研究者快速找到相关文献并提取关键信息 |
| **官网** | https://elicit.com |
| **费用** | 基础版免费；Plus 版付费 |

**核心功能**：
- 语义搜索：无需精确关键词即可找到相关论文
- 自动文献总结：提取论文关键结论和方法
- 数据提取与分析：从论文中抽取统计信息
- 表格式多篇论文对比
- 智能对话功能：基于文献内容回答问题
- 基于 Semantic Scholar 数据库检索

**科研流程中的使用**：
- 文献综述阶段：快速搜索和筛选文献
- 提取多篇论文的核心数据进行对比
- 了解新领域的入门工具

---

### 7. PaperQA2

| 项目 | 内容 |
|------|------|
| **功能定位** | 高精度 RAG 工具，专注于科学文献的问答、总结和矛盾检测 |
| **GitHub** | https://github.com/Future-House/paper-qa |
| **费用** | 开源免费（MIT 许可证） |

**核心功能**：
- 提供带引文支持的精确答案
- 文档元数据感知的智能检索
- Agentic RAG 支持：迭代优化查询和答案
- 自动提取文献元数据（DOI、期刊质量等）
- 全文搜索引擎：支持 PDF 和文本文件
- 矛盾检测：发现论文间的冲突结论
- 默认支持所有 LiteLLM 兼容模型

**安装使用**：
```bash
pip install paper-qa
# 创建项目文件夹，添加 PDF 文件
pqa add paper.pdf
pqa ask "How can carbon nanotubes be manufactured at scale?"
```

**科研流程中的使用**：
- 针对特定研究问题从文献库中检索答案
- 检测多篇论文间是否存在矛盾结论
- 辅助撰写文献综述中的对比分析部分

---

### 8. SciSpace（原 Typeset.io）

| 项目 | 内容 |
|------|------|
| **功能定位** | 一站式 AI 科研辅助平台，提供论文阅读、文献综述、文本润色等功能 |
| **官网** | https://typeset.io （域名 scispace.com 跳转） |
| **费用** | 基础版免费；高级版付费 |

**核心功能**：
- Copilot 辅助阅读论文，可分析文本、图表和公式
- 截取公式/图表后自动解析
- 自动生成文献综述
- 文字润色改写
- AI 率检查
- 批量论文管理和解读
- Chrome 浏览器插件

**科研流程中的使用**：
- 论文精读：边读边问，快速理解复杂公式和图表
- 文献综述：自动整合多篇文献内容
- 写作润色：改善学术表达

---

### 9. ScholarAI

| 项目 | 内容 |
|------|------|
| **功能定位** | ChatGPT 插件/独立工具，专为学术研究人员设计，从海量文献中提取和总结关键信息 |
| **官网** | https://scholarai.io |
| **费用** | 基础功能免费；高级功能付费 |

**核心功能**：
- 高效处理成千上万篇学术文献
- 根据主题快速返回文献综合摘要
- 支持学术论文的语义检索
- 可作为 ChatGPT 插件使用

**科研流程中的使用**：
- 在 ChatGPT 对话中直接检索学术文献
- 快速获取某个研究方向的核心文献摘要

---

## 三、自动化研究工具

### 10. AI-Researcher（港大开源）

| 项目 | 内容 |
|------|------|
| **功能定位** | 开源自动化科研智能体框架，覆盖从文献综述到论文撰写的全流程自动化 |
| **GitHub** | https://github.com/HKUDS/AI-Researcher |
| **费用** | 开源免费（需 API Key） |
| **定位** | Google AI Co-Scientist 的开源替代方案 |

**核心功能**：
- 全流程自动化：文献综述 -> 创意生成 -> 算法设计 -> 验证优化 -> 论文撰写
- 两种模式：(1) 提供研究想法，系统生成实现策略；(2) 提供参考文献，系统自主生成创新方向
- 支持多领域研究（CV、NLP 等）
- 以 Claude-3.5-Sonnet 为核心引擎，兼容 DeepSeek、HuggingFace 等模型
- 自动检索学术数据库（ArXiv、IEEE Xplore 等）和代码平台（GitHub、HuggingFace）

**科研流程中的使用**：
- 快速生成研究方向初步报告
- 辅助实验设计和结果分析
- 自动化论文初稿生成

**注意**：该工具为实验性项目，产出需人类研究者审查和验证。

---

### 11. GPT Researcher

| 项目 | 内容 |
|------|------|
| **功能定位** | 基于 LLM 的自主研究代理，对任意主题进行全面在线研究并生成带引用的研究报告 |
| **GitHub** | https://github.com/assafelovic/gpt-researcher |
| **官网** | https://gptr.dev |
| **费用** | 开源免费（MIT 许可证，需 API Key） |
| **Stars** | 21k+ |

**核心功能**：
- 自动生成 2000+ 字详细研究报告
- 整合 20+ 个网络来源，形成客观结论
- 多格式导出（PDF、Word、Markdown）
- 支持动态网页抓取（JavaScript 渲染）
- 本地文档研究（PDF、Word、Excel）
- 多代理协作系统
- SimpleQA 基准测试准确率达 93%
- 支持 DeepSeek、OpenAI 等多种 LLM 后端

**安装使用**：
```bash
pip install gpt-researcher
# 或克隆仓库
git clone https://github.com/assafelovic/gpt-researcher.git
```

**科研流程中的使用**：
- 快速了解新研究领域的全景
- 自动生成文献综述初稿
- 辅助市场研究和技术调研

---

## 四、文献搜索与发现

### 12. Semantic Scholar API

| 项目 | 内容 |
|------|------|
| **功能定位** | AI 驱动的免费学术搜索引擎，提供开放 API 供开发者程序化访问论文数据 |
| **官网** | https://www.semanticscholar.org |
| **API 文档** | https://api.semanticscholar.org |
| **费用** | 免费（API 无需申请 Key 即可使用，有速率限制） |

**核心功能**：
- 收录 2.18 亿+ 篇论文的庞大数据库
- AI 智能摘要和论文推荐
- 被引用情况分类分析（支持/对比/提及）
- 论文 TLDR 摘要
- 开放 API：论文搜索、引用分析、作者信息
- Semantic Reader 增强阅读器
- 与 Zotero 等工具集成

**API 使用示例**：
```python
import requests
# 搜索论文
response = requests.get(
    "https://api.semanticscholar.org/graph/v1/paper/search",
    params={"query": "transformer attention mechanism", "limit": 5}
)
papers = response.json()
```

**科研流程中的使用**：
- 程序化检索论文数据，构建文献数据库
- 分析论文引用关系和研究趋势
- 辅助构建论文关系图谱

---

### 13. ResearchRabbit

| 项目 | 内容 |
|------|------|
| **功能定位** | 基于文献关系图谱的探索工具，可视化发现和追踪学术文献 |
| **官网** | https://www.researchrabbitapp.com |
| **费用** | 完全免费 |

**核心功能**：
- 可视化文献网络：以图谱展示"引用-被引"关系
- 输入一篇论文即可发现相关文献网络
- 支持按题目、DOI、PMID 搜索
- 个性化推荐系统
- 时间轴展示研究发展脉络
- 支持与 Zotero 同步
- 支持多学科领域
- 团队协作和集合共享

**科研流程中的使用**：
- 进入新领域时快速了解文献全景
- 追踪研究脉络，找到关键核心论文
- 文献综述阶段扩展参考文献列表

---

### 14. Connected Papers

| 项目 | 内容 |
|------|------|
| **功能定位** | 论文关系可视化工具，以图谱方式展示论文间的相似性和引用关系 |
| **官网** | https://www.connectedpapers.com |
| **费用** | 免费使用（有使用次数限制，高级版付费） |

**核心功能**：
- 输入一篇论文，自动生成相关论文网络图谱
- 图谱特征：节点距离 = 相似度，颜色 = 发表时间，大小 = 引用次数
- Prior Works 视图：发现领域内重要先前作品
- Derivative Works 视图：发现最新衍生研究
- 基于 Semantic Scholar 数据库
- 一键生成文献列表

**科研流程中的使用**：
- 快速获取新领域的全景概览
- 补充文献综述中可能遗漏的重要参考文献
- 追踪某篇论文的前世今生

---

## 五、数据分析工具

### 15. PandasAI

| 项目 | 内容 |
|------|------|
| **功能定位** | 为 Pandas 增加生成式 AI 功能的 Python 库，用自然语言与数据对话 |
| **GitHub** | https://github.com/Sinaptik-AI/pandas-ai |
| **文档** | https://docs.getpanda.ai |
| **费用** | 开源免费（MIT 许可证） |
| **Stars** | 19.9k+ |

**核心功能**：
- 自然语言查询数据（无需编写代码）
- 支持多种数据源：CSV、XLSX、PostgreSQL、MySQL、BigQuery 等
- 自动数据可视化（生成图表）
- 数据清洗和缺失值处理
- 特征生成提升数据质量
- 跨数据表关联查询
- Docker 安全沙箱执行

**安装使用**：
```bash
pip install "pandasai>=3.0.0b2"
pip install pandasai-openai
```

```python
import pandasai as pai
df = pai.DataFrame({
    "country": ["China", "USA", "Japan", "Germany"],
    "revenue": [7000, 5000, 4500, 4100]
})
print(df.chat("What is the total revenue of the top 3 markets?"))
```

**科研流程中的使用**：
- 实验数据的快速探索性分析
- 调查问卷数据的自动分析
- 生成论文中的统计图表

---

## 六、图表绘制工具

### 16. Draw.io（diagrams.net）

| 项目 | 内容 |
|------|------|
| **功能定位** | 开源免费的在线/离线图表绘制工具，支持流程图、UML、网络图等 |
| **官网** | https://www.drawio.com （或 https://app.diagrams.net） |
| **GitHub** | https://github.com/jgraph/drawio |
| **费用** | 完全免费开源 |

**核心功能**：
- 丰富的图表类型：流程图、组织结构图、UML、ER 图、网络拓扑图等
- 海量预置组件库和模板
- 实时多人协作
- 支持导入 Visio 和 Gliffy 文件
- 多格式导出：PDF、SVG、PNG、HTML
- 支持本地存储（XML）和云存储（Dropbox、Google Drive 等）
- Chrome 插件、桌面客户端、VS Code 集成
- 支持中文界面

**科研流程中的使用**：
- 绘制论文中的方法框架图
- 绘制系统架构图和实验流程图
- 绘制 UML 类图和序列图

---

### 17. Detikzify

| 项目 | 内容 |
|------|------|
| **功能定位** | 将手绘草图或现有图表转换为 TikZ 代码，直接用于 LaTeX 文档 |
| **GitHub** | https://github.com/potion-source/DetikZify |
| **费用** | 开源免费 |

**核心功能**：
- 将手绘草图自动转换为 TikZ 图形程序
- 将现有科学图表转换为 TikZ 代码
- 训练数据集 DaTikZv2（36 万+ TikZ 图形）
- SketchFig 数据集：手绘草图与科学图表配对
- 生成的代码可直接嵌入 LaTeX 文档

**科研流程中的使用**：
- 论文中的高质量 TikZ 图表生成
- 将草图快速转化为出版级图表
- 替代手动编写复杂 TikZ 代码

---

### 18. img2img-turbo

| 项目 | 内容 |
|------|------|
| **功能定位** | 基于 SD-Turbo 的实时图像到图像转换模型，支持风格迁移、线稿上色等 |
| **GitHub** | https://github.com/GaParmar/img2img-turbo |
| **费用** | 开源免费 |

**核心功能**：
- 单步图像到图像转换（极速推理）
- CycleGAN-Turbo：非配对数据转换（如马变斑马、白天变黑夜）
- Pix2Pix-Turbo：配对数据转换（如线稿变真实图像）
- Gradio 交互式 Web 界面
- 支持自定义训练

**科研流程中的使用**：
- 生成论文中的示意图（草图到高质量图像）
- 图像风格转换实验
- 计算机视觉研究中的基线对比

---

## 七、文献管理工具

### 19. Zotero 核心插件

Zotero 是免费开源的文献管理软件（https://www.zotero.org），以下是其最重要的学术插件：

#### Better BibTeX

| 项目 | 内容 |
|------|------|
| **功能定位** | 为 Zotero 生成规范的 BibTeX Citation Key，实现与 LaTeX 无缝配合 |
| **下载** | https://retorque.re/zotero-better-bibtex |
| **费用** | 免费 |

**核心功能**：
- 自动生成规范的 Citation Key（如 `he2016deep`）
- 自定义 Key 生成规则
- 自动导出并同步 .bib 文件
- 与 LaTeX 编辑器无缝集成

#### Jasminum（茉莉花）

| 项目 | 内容 |
|------|------|
| **功能定位** | 支持 Zotero 抓取中文文献（CNKI、万方、维普等） |
| **GitHub** | https://github.com/l0o0/jasminum |
| **费用** | 免费 |

**核心功能**：
- 支持 CNKI、万方、维普、百度学术等中文数据库
- 支持 CAJ 格式
- 自动为 PDF 添加书签

#### Zotero OCR

| 项目 | 内容 |
|------|------|
| **功能定位** | 为 Zotero 中的扫描版 PDF 添加 OCR 文字识别层 |
| **费用** | 免费 |

#### Zotero Citation Counts

| 项目 | 内容 |
|------|------|
| **功能定位** | 自动获取论文的引用次数并显示在 Zotero 中 |
| **费用** | 免费 |

---

## 八、Obsidian 学术插件

### 20. Obsidian 学术工作流核心插件

#### Zotero Integration

| 项目 | 内容 |
|------|------|
| **功能定位** | 将 Zotero 与 Obsidian 深度整合，一键导入文献笔记和标注 |
| **安装** | Obsidian 社区插件市场搜索 "Zotero Integration" |
| **费用** | 免费 |

**核心功能**：
- 从 Zotero 导入文献元数据和 PDF 标注
- 自定义笔记模板
- 支持在 Obsidian 中直接打开 Zotero PDF
- 图像导入和 OCR 支持

#### Citations

| 项目 | 内容 |
|------|------|
| **功能定位** | 在 Obsidian 中管理学术引用，从文献库创建笔记 |
| **安装** | Obsidian 社区插件市场搜索 "Citations" |
| **费用** | 免费 |

**核心功能**：
- 导入 Zotero 导出的 CSL-JSON 或 BibTeX 文件
- 快捷键快速创建文献笔记
- 自动插入引用信息

#### Dataview

| 项目 | 内容 |
|------|------|
| **功能定位** | 将 Obsidian Vault 变为可查询数据库，用类 SQL 语法检索笔记 |
| **安装** | Obsidian 社区插件市场搜索 "Dataview" |
| **费用** | 免费 |

**核心功能**：
- 查询和展示笔记元数据
- 创建动态文献列表、任务列表
- 支持表格、列表、任务视图

```dataview
TABLE title, year, journal
FROM "research/02-papers"
WHERE contains(tags, "cv")
SORT year DESC
```

#### Templater

| 项目 | 内容 |
|------|------|
| **功能定位** | 高级模板引擎，自动化笔记创建流程 |
| **安装** | Obsidian 社区插件市场搜索 "Templater" |
| **费用** | 免费 |

**核心功能**：
- 支持变量、条件语句、循环
- 自动填充日期、文件名等
- 配合论文笔记模板自动创建结构化笔记
- 支持执行 JavaScript 代码

---

## 九、LaTeX 工具

### 21. latexdiff

| 项目 | 内容 |
|------|------|
| **功能定位** | LaTeX 文档版本对比工具，高亮显示两个 .tex 文件之间的差异 |
| **官网** | CTAN 标准包，TeX Live 和 MiKTeX 自带 |
| **费用** | 免费开源 |

**核心功能**：
- 自动标记新增、删除、修改的文本
- 多种标记风格（传统下划线、颜色标记等）
- 支持公式、图表的变更标记
- 可直接编译输出带标记的 PDF

**使用方式**：
```bash
latexdiff old_version.tex new_version.tex > diff.tex
pdflatex diff.tex
```

**Overleaf 中使用**：在项目中创建 `latexmkrc` 文件配置自动 diff。

**科研流程中的使用**：
- 论文修改版本对比（投稿修改时标注改动）
- 合作者之间查看修改内容
- 审稿回复时标记论文修改部分

---

### 22. Overleaf CLI

| 项目 | 内容 |
|------|------|
| **功能定位** | Overleaf 的命令行工具，支持本地编辑 Overleaf 项目 |
| **GitHub** | https://github.com/overleaf/overleaf-cli （社区版） |
| **费用** | 免费 |

**核心功能**：
- 将 Overleaf 项目克隆到本地编辑
- 支持用本地编辑器（VS Code 等）编写 LaTeX
- 自动同步到 Overleaf 云端

**科研流程中的使用**：
- 偏好本地编辑器但使用 Overleaf 协作的研究者
- 自动化论文编译流程

---

## 十、提示词工程

### 23. Awesome ChatGPT Prompts

| 项目 | 内容 |
|------|------|
| **功能定位** | ChatGPT 提示词模板集合，包含 200+ 个角色扮演提示词 |
| **GitHub** | https://github.com/f/awesome-chatgpt-prompts |
| **官网** | https://prompts.chat |
| **费用** | 完全免费开源 |
| **Stars** | 110k+ |

**核心功能**：
- 212+ 个精心设计的提示词模板
- 涵盖编程、写作、教育、研究等各领域
- 支持将 ChatGPT 调教为不同角色（如学术审查员、Linux 终端等）
- 社区持续贡献新提示词
- CSV 格式方便导入各种工具

**科研流程中的使用**：
- 使用学术相关提示词优化 AI 交互
- 论文写作时使用特定角色提示词获得更好的输出
- 代码调试、数据分析等场景的提示词参考

---

## 十一、工具组合推荐

### 按研究阶段的工具组合

| 研究阶段 | 推荐工具组合 |
|----------|-------------|
| **选题探索** | Semantic Scholar + ResearchRabbit + Connected Papers + GPT Researcher |
| **文献综述** | Elicit + SciSpace + ChatPaper + Zotero + Obsidian |
| **论文写作** | GPT Academic + STORM + Writefull/Grammarly + Draw.io + Detikzify |
| **数据分析** | PandasAI + Python (matplotlib/seaborn) |
| **论文修改** | latexdiff + Grammarly + Writefull |
| **知识管理** | Zotero + Obsidian (Dataview + Templater + Zotero Integration) |
| **自动化研究** | AI-Researcher + GPT Researcher + PaperQA2 |

### 推荐基础工具链（免费）

```
文献管理：Zotero + Better BibTeX + Jasminum
笔记管理：Obsidian + Dataview + Templater + Zotero Integration
论文阅读：SciSpace（免费版）+ ChatPaper
论文写作：GPT Academic + Writefull（免费版）+ Draw.io
文献发现：Semantic Scholar + ResearchRabbit + Connected Papers
版本控制：latexdiff + Git
```

---

## 常见误区

| 误区 | 正确理解 |
|------|---------|
| AI 工具可以完全替代人类做研究 | AI 是加速器，核心创新和判断必须由人类主导 |
| 工具越多越好 | 选择适合自己工作流的 5-8 个核心工具即可 |
| 开源工具需要高技术门槛 | 大部分工具有一键安装包或 Web 界面 |
| AI 生成的论文可以直接投稿 | AI 产出必须经过人工审查、验证和修改 |
| 免费工具功能不够 | 很多免费开源工具（如 Zotero、Draw.io）功能超过付费软件 |

---

## 思考题

> [!hint]- 思考提示
> 考虑每个研究阶段最耗时的环节，以及哪些工具可以并行使用。

> [!success]- 参考答案
> 最高效的方式是：用 ResearchRabbit/Connected Papers 发现文献 -> Zotero 管理文献 -> Obsidian 做笔记 -> GPT Academic/STORM 辅助写作 -> Writefull/Grammarly 润色 -> latexdiff 对比版本。这套工具链全程免费。

---

## 关键要点回顾

1. 科研工具的核心目标是**提升效率**，而非替代人类的思考和创新
2. 免费开源工具已经覆盖科研全流程，不必依赖付费软件
3. **Zotero + Obsidian** 是文献管理和知识管理的黄金组合
4. AI 写作工具产出的内容**必须经过人工审查**
5. 选择工具的标准：免费优先、开源优先、与现有工作流兼容优先

---

## 扩展阅读

- [GPT Academic 官方文档](https://github.com/binary-husky/gpt_academic)
- [STORM 官方论文](https://arxiv.org/abs/2402.14207)
- [GPT Researcher 文档](https://docs.gptr.dev)
- [Semantic Scholar API 文档](https://api.semanticscholar.org)
- [Zotero 插件大全](https://www.zotero.org/support/plugins)

## 下一步学习

- 深入学习 Zotero + Obsidian 工作流配置
- 学习 LaTeX 论文写作基础
- 了解 Python 数据分析基础（配合 PandasAI）
