---
type: lesson
tags:
  - LangGraph
  - Agent
  - Python
  - 工作流
  - 学习路线图
created: 2026-04-28
updated: 2026-04-28
topic: LangGraph 学习地图与研究助手项目总览
difficulty: beginner
prerequisites:
  - 基础 Python 语法（函数、dict、list）
  - 对 LLM、Agent、工作流有粗略印象即可
sources:
  - https://docs.langchain.com/oss/python/langgraph/overview
  - https://docs.langchain.com/oss/python/langgraph/quickstart
  - https://docs.langchain.com/oss/python/langgraph/graph-api
  - https://docs.langchain.com/oss/python/langgraph/streaming
  - https://api-docs.deepseek.com/zh-cn/
  - research/01-topics/agent-memory.md
status: completed
series:
  name: LangGraph 系统学习
  part: 1
---

# LangGraph 学习地图与研究助手项目总览

> 一句话摘要：本章不是“泛泛介绍 LangGraph”，而是整套课的开学总纲。你将一次性看清 10 章固定路线、Research Copilot 项目主线、环境准备、项目目录约定、嵌入式 Python 补课地图，以及每一章最终会交付什么。

## 学习目标

完成本章后，你应该能够：
- [ ] 说清楚 LangGraph 解决的是“复杂任务如何组织执行”，而不只是“多调几次模型”
- [ ] 记住本系列固定的 10 章顺序，并理解为什么不建议跳学
- [ ] 搭好 LangGraph 学习环境，并完成一次最小安装验证
- [ ] 看懂 Research Copilot 项目会如何从 v0 长到 v8
- [ ] 知道每一章会补哪些 Python 知识，以及这些补课会落到什么代码上
- [ ] 明白本系列采用现代 LangGraph API 主线，而不是旧式 `AgentExecutor` / while-loop agent 教学路线

## 前置知识

| 知识点 | 掌握程度要求 | 参考页面 |
|------|-------------|---------|
| Python 函数、`dict`、`list` | 基本会用即可 | — |
| 大语言模型（LLM） | 知道“给提示词，模型回文本”即可 | — |
| Agent 的基本印象 | 知道它可能调用模型、工具、记忆 | [[research/01-topics/agent-memory|Agent Memory 研究方向]] |
| 科研工作流 | 只需对“搜资料→整理→输出”有直觉 | [[raw/lessons/research-tools/research-open-source-tools-guide|科研开源工具全景指南]] |

---

## 直观理解

### 类比一：LangGraph 像“实验室里的流程白板”

真正麻烦的 agent 系统，难点往往不在“模型够不够聪明”，而在：

- 这一步该做什么？
- 做完后该交给谁？
- 哪些信息要一直保留？
- 什么时候必须停下来等人审核？
- 哪些步骤可以并行？

LangGraph 的价值，就是把这些问题显式画成图：

- **节点（node）** = 一个明确步骤
- **边（edge）** = 步骤之间的流向
- **状态（state）** = 全流程共享工作台
- **检查点（checkpointer）** = 流程进度可保存、可恢复
- **中断（interrupt）** = 人在回路的暂停点

> [!tip] 为什么这比“巨型 prompt + while 循环”更稳？
> 因为白板上的流程图一旦画清楚：
> - 你知道系统为什么走到这里
> - 你知道哪一步出了错
> - 你知道该替换哪个节点而不是推翻全部
> - 你知道如何加记忆、审批、并行，而不是继续往黑盒里塞逻辑

### 类比二：Research Copilot 像“会分工的研究助理团队”

如果只写普通脚本，你很容易得到一个“边想边说、想到哪做到哪”的单线程助理。

而 LangGraph 更像一个可组织的研究助理团队：

- 共享笔记本 = state
- 明确分工 = nodes
- 工作流程 = edges
- 会后纪要 = checkpoints
- 老师审批 = human-in-the-loop
- 多路检索 = `Send` 并行分发

这就是为什么本系列选择 **Research Copilot（研究助手）** 作为贯穿项目：它几乎天然覆盖 LangGraph 的全部关键能力。

---

## 一、为什么现在学 LangGraph？

### 1.1 从“会调模型”到“会设计工作流”

很多人入门 LLM 时，最先形成的直觉是：

> “只要 prompt 写得足够好，agent 就会自己完成复杂任务。”

但一旦任务变成：

- 多步推理
- 工具调用
- 跨轮上下文记忆
- 人工审批
- 流式反馈
- 并行检索
- 失败后恢复

你会发现真正难的，不再是单次调用模型，而是：

> **如何把一个复杂任务组织成稳定、可解释、可扩展的系统。**

LangGraph 恰好把“系统组织能力”变成一等公民。

### 1.2 LangGraph 不主要提高模型智商，而是提高系统组织能力

LangGraph 不是在训练模型参数，也不是神秘地让模型突然更聪明。

它做的是：

- 规定什么时候思考
- 规定什么时候用工具
- 规定什么时候停下来
- 规定哪些信息应该积累
- 规定下一步应该去哪

换句话说：

> **LangGraph 更像是 agent 系统的流程操作系统。**

### 1.3 本系列为什么不用旧式主线开讲

本系列明确采用现代 LangGraph 学习路线，优先围绕下面这些能力展开：

- `StateGraph`
- `START` / `END`
- 自定义状态 / `MessagesState`
- `add_messages`
- `ToolNode`
- `tools_condition`
- `create_react_agent`
- `MemorySaver` / `SqliteSaver`
- `interrupt()` + `Command(resume=...)`
- `stream()`
- `Send`
- `langgraph dev`

> [!warning] 旧教程避坑说明
> 本系列**不会**把下面这些模式当成入门主线：
> - `AgentExecutor`
> - 旧式 LangChain agent loop
> - 手搓一个 `while True` 的工具循环，再把它叫成“agent 框架”
> - 把旧 memory 组件当成现代 LangGraph 记忆主角
>
> 这些内容可以知道，但不应该作为 2026 年系统学习 LangGraph 的第一条主路。

---

## 二、先把环境搭好：这一章结束前必须完成的准备

### 2.1 推荐环境

- Python：`3.11+`
- 包管理：`uv`
- 主要依赖：`langgraph`、`langchain`、`langchain-deepseek`
- 本地开发 CLI：`langgraph-cli[inmem]`

### 2.2 最小安装步骤

> [!example]- 用 `uv` 初始化一个练习项目
> ```bash
> uv init research-copilot
> cd research-copilot
> uv add "langchain>=1,<2" langgraph langchain-deepseek typing-extensions "langgraph-cli[inmem]"
> uv lock
> ```

如果你会在后续章节跟着项目实战调用 DeepSeek，再准备 `.env`：

```bash
DEEPSEEK_API_KEY=你的密钥
```

> [!tip] 为什么这里优先用 `langchain-deepseek`
> DeepSeek 官方文档同时提供了 OpenAI 兼容接口和 Anthropic 兼容接口，但对 LangGraph 初学者来说，课程里优先使用 `langchain-deepseek` + `init_chat_model(..., model_provider="deepseek")` 更直接。
> 这样第 4 章的 `bind_tools()`、后续可能扩展到的流式输出，也更容易保持和官方能力对齐。

### 2.2.5 项目实战的版本策略（以 2026-04 为基准）

这次口径需要明确统一：

> **`langchain` 必须使用 1.x；`langgraph` 统一按最新稳定版主线学习；项目默认模型接入统一走 DeepSeek。**

这意味着本系列不再把 `langchain 0.x` 或旧的 `langgraph 0.x` 版本带当成推荐主线。更稳的做法是：

1. `langchain` 明确锁在 `1.x`
2. `langgraph`、`langchain-deepseek`、`langgraph-cli[inmem]` 直接安装最新稳定版
3. 首次安装完成后用 `uv lock` 固化解析结果，而不是在正文里继续写死过时的小版本带

| 依赖 | 当前推荐策略 | 为什么这样选 |
| --- | --- | --- |
| Python | `3.11.x` 优先 | 生态最稳，课程里最少遇到兼容性边角问题 |
| `langchain` | `>=1,<2` | 这是本项目的硬约束；课程示例统一按 LangChain 1.x 写法组织 |
| `langgraph` | 最新稳定版 | 本系列围绕当前稳定 API 主线编写，不再沿用旧的 `0.x` 版本带 |
| `langchain-deepseek` | 最新稳定版 | 作为 DeepSeek 的默认 LangChain 适配层，优先与当前 LangChain 1.x 对齐 |
| `langgraph-cli[inmem]` | 最新稳定版 | 第 10 章会用到 `langgraph dev`，CLI 应与当前 LangGraph 代际保持一致 |
| `typing-extensions` | 最新稳定版 | `TypedDict`、`Annotated` 等类型工具会频繁出现 |

如果你想同时满足“跟教程一致”和“不要落回旧版本”，建议直接这样装：

> [!example]- 推荐安装方式（LangChain 1.x + 最新稳定版）
> ```bash
> uv add "langchain>=1,<2" langgraph langchain-deepseek typing-extensions "langgraph-cli[inmem]"
> uv lock
> ```

### 2.2.6 这些依赖各自扮演什么角色？

很多初学者会把“装了几个包”理解成纯体力活，但如果你不知道谁负责什么，后面一旦报错就很难定位。

- `langgraph`
  - **作用**：提供图工作流骨架
  - **你会直接用到的能力**：`StateGraph`、`START`、`END`、`Send`、`interrupt()`、`MemorySaver`
  - **输入/输出视角**：它不负责生成答案，而是负责组织“状态如何流经多个节点”

- `langchain`
  - **作用**：提供统一的模型抽象、消息对象、工具抽象
  - **你会直接用到的能力**：`init_chat_model()`、`HumanMessage`、`SystemMessage`、`@tool`
  - **输入/输出视角**：它负责把“模型调用”封装成统一接口，让不同模型都能用相似方式接入

- `langchain-deepseek`
  - **作用**：把 DeepSeek 模型能力接到 LangChain 的统一接口上
  - **你会直接用到的能力**：让 `init_chat_model(..., model_provider="deepseek")` 真的可用
  - **输入/输出视角**：它相当于“适配层”，把 LangChain 的抽象翻译成 DeepSeek 的实际 API 请求

- `langgraph-cli[inmem]`
  - **作用**：提供本地开发与调试体验
  - **你会直接用到的能力**：`langgraph dev`
  - **输入/输出视角**：它不改变图逻辑本身，而是负责把图应用跑起来、暴露本地调试入口

### 2.2.7 五类最常见的版本与环境坑

> [!important] 当前课程的版本底线
> - `langchain`：必须是 `1.x`
> - `langgraph`：使用最新稳定版
> - `langchain-deepseek`：使用与当前 LangChain 主线兼容的最新稳定版
> - 遇到环境异常时，先排查是否误装了 `langchain 0.x`

> [!warning] 先记住一个总原则
> 如果你遇到的错误看起来“明明代码没错却 import 失败”，第一反应不要急着改代码，先检查**依赖版本和安装是否对齐**。

#### 坑 1：`langgraph` 已升级，但 `langchain` 还停在 0.x

**现象**：
- 某些 import 能成功，某些 import 失败
- 教程里的代码在你电脑上报“找不到某个对象”
- `bind_tools()`、消息对象、streaming 接口行为和教程不一致

**原因**：
`langgraph` 与 `langchain` 虽然分包，但很多抽象层是配套演进的。你如果把 `langgraph` 装成当前稳定版，却把 `langchain` 留在 `0.x`，最容易出现“每个包单看都装好了，但放在一起就不对”。

**处理方式**：
- 先确认 `langchain` 是否明确处于 `1.x`
- 再确认 `langgraph`、`langchain-deepseek`、`langgraph-cli[inmem]` 是否都来自当前稳定代
- 不要让教程主线一边讲 1.x，一边在环境里残留 0.x

#### 坑 2：装了 `langchain`，但漏装 `langchain-deepseek`

**现象**：
- `init_chat_model()` 能 import
- 但一旦写 `model_provider="deepseek"` 就报 provider 相关错误

**原因**：
`init_chat_model()` 是统一入口，但真正的 DeepSeek 适配逻辑不在 `langchain` 主包里，而在 `langchain-deepseek`。

**处理方式**：
- 检查是否真的执行过 `uv add langchain-deepseek`
- 不要以为“装了 langchain 就等于所有模型都能接”

#### 坑 3：忘了装 `langgraph-cli[inmem]`

**现象**：
- 前面章节的 Python 脚本都能跑
- 到第 10 章执行 `uv run langgraph dev` 时突然失败

**原因**：
CLI 是单独的依赖，不会因为你能 `import StateGraph` 就自动存在。

**处理方式**：
- 一开始就把 `langgraph-cli[inmem]` 装上
- 不要等到第 10 章再补

#### 坑 4：Windows 下命令能装包，但运行脚本时环境不一致

**现象**：
- `uv add` 成功
- 但 `python xxx.py` 报找不到包

**原因**：
你可能直接用了系统 Python，而不是通过 `uv run` 进入当前项目环境。

**处理方式**：
- 本系列统一优先使用 `uv run python xxx.py`
- 不要混用系统 Python 和项目环境

#### 坑 5：`.env` 文件看起来没问题，实际读不到 API Key

**现象**：
- 明明写了 `DEEPSEEK_API_KEY`
- 实际运行时报缺少密钥

**原因**：
可能是：
- `.env` 文件没有被正确加载
- 文件编码异常
- 变量名前后多了空格
- 你把 key 写到了别的目录下的 `.env`

**处理方式**：
- 先确认 `.env` 文件就在项目根目录
- 再确认变量名完全一致：`DEEPSEEK_API_KEY`
- 如果仍不生效，先在终端临时打印环境变量排查路径问题

### 2.3 最小安装验证

在正式进入第 2 章之前，先完成一次最小验证：

> [!example]- 验证 LangGraph 已正确安装
> ```bash
> uv run python -c "from langgraph.graph import StateGraph, START, END; print('LangGraph OK')"
> ```

如果终端输出：

```text
LangGraph OK
```

说明你至少已经具备继续学习第 2 章的环境基础。

如果你还想进一步确认“第 4 章的模型接入也没问题”，可以再做一个更贴近项目实战的检查：

> [!example]- 验证 DeepSeek 适配层已正确安装
> ```bash
> uv run python -c "from langchain.chat_models import init_chat_model; m = init_chat_model('deepseek-chat', model_provider='deepseek'); print(type(m).__name__)"
> ```

如果这一步也能正常输出模型对象类型，通常说明：
- `langchain` 已安装，且处于 1.x 主线
- `langchain-deepseek` 已安装
- `model_provider="deepseek"` 这条主线在你的环境里是成立的

> [!tip] 为什么这里默认写 `deepseek-chat`？
> 因为在当前能查到的 LangChain 官方集成文档里，DeepSeek 的示例默认使用 `deepseek-chat`。后续如果你已经确认具体项目模型名，例如 `deepseek-v4-pro`，可以再替换成自己的生产模型。

### 2.4 第 2 章前的“开学自检”

在进入实作前，你最好确认下面 4 件事：

- [ ] 我知道如何用 `uv run python xxx.py` 运行脚本
- [ ] 我知道 `StateGraph`、`START`、`END` 这三个名字是后面反复出现的主角
- [ ] 我知道后面每章都在扩展同一个 Research Copilot，而不是每章重开新玩具项目
- [ ] 我知道这套课会边学 LangGraph 边补 Python，而不是先上完整 Python 预科

---

## 三、贯穿全系列的项目：Research Copilot

### 3.1 为什么选“研究助手”而不是玩具 Demo

本系列不会每章换一个互不相干的小例子，而是坚持一条主线：

**Research Copilot（研究助手）**。

原因很简单：

1. 它贴合你当前 vault 的研究工作区
2. 它天然包含检索、整理、记忆、审批、并行、部署等能力
3. 它适合逐章递进，而不是一上来就做成黑盒 agent

### 3.2 Research Copilot 会怎样长大

| 章节     | 项目版本 | 你会交付什么                       |
| ------ | ---- | ---------------------------- |
| 第 2 章  | v0   | 一个最小 3 节点 `StateGraph` 研究助手  |
| 第 3 章  | v1   | 一个能处理消息状态的对话版研究助手            |
| 第 4 章  | v2   | 一个能调用检索工具的研究助手               |
| 第 5 章  | v3   | 一个有跨轮记忆、支持 `thread_id` 的研究助手 |
| 第 6 章  | v4   | 一个支持人工审核与恢复执行的研究助手           |
| 第 7 章  | v5   | 一个能实时流式展示执行过程的研究助手           |
| 第 8 章  | v6   | 一个能并行检索多个来源的研究助手             |
| 第 9 章  | v7   | 一个拆成多个子图、可复用模块化的研究助手         |
| 第 10 章 | v8   | 一个整理成应用结构、可本地开发的研究助手         |

### 3.3 最终项目轮廓

```text
用户提出研究问题
        │
        ▼
明确任务 / 规划检索
        │
        ├──► 并行查论文 / 博客 / 现有知识库
        │
        ▼
整理证据与草稿
        │
        ├──► 需要人工审核？ ──► interrupt() 暂停
        │
        ▼
生成结构化研究回答
        │
        ▼
保存线程状态与上下文
```

你可以把它理解成一个“会查、会记、会暂停、会分工”的研究助理。

---

## 四、项目目录约定：从一开始就把骨架定清楚

为了避免到第 10 章才发现文件结构混乱，本系列从第 1 章就固定项目目录约定。

### 4.1 推荐目录结构

```text
research-copilot/
├── .env
├── pyproject.toml
├── langgraph.json
├── src/
│   └── research_copilot/
│       ├── __init__.py
│       ├── state.py          # 状态定义
│       ├── tools.py          # 工具定义
│       ├── nodes.py          # 节点函数
│       ├── graphs/
│       │   ├── basic.py      # 第 2 章最小图
│       │   ├── chat.py       # 第 3 章消息态图
│       │   ├── react.py      # 第 4 章工具调用图
│       │   └── app.py        # 第 10 章整理后的应用入口
│       └── prompts.py        # 可选：系统提示词集中管理
└── notebooks/               # 可选：实验性验证脚本
```

### 4.2 为什么要这么早讲目录结构

因为很多新手的第一轮混乱，不是出在 API，而是出在“代码随便放”。

如果你越学越把所有内容都塞进一个 `main.py`，会很快遇到这些问题：

- state 定义到处散落
- 工具函数和业务节点纠缠在一起
- 第 5 章加记忆时，不知道配置该放哪里
- 第 9 章讲子图时，文件结构完全无从拆分

所以本系列主张：

> **目录先有边界，后面章节才长得稳。**

---

## 五、固定的 10 章路线图

> [!important] 本系列先固化章节顺序，再逐章展开
> 这不是“写到哪算哪”的草稿式课程，而是固定为 10 章的系统课程。后续导航、`series.part`、项目成长表都以此为准。

### 5.1 项目时间线 / 路线图

```text
第 1 章 [本章] ─── LangGraph 学习地图 + 研究助手项目总览
                    │
第 2 章 ────────── StateGraph 基础：节点、边、START/END
                    │
第 3 章 ────────── State 与 Messages：MessagesState / add_messages
                    │
第 4 章 ────────── 工具与 ReAct：ToolNode / tools_condition / create_react_agent
                    │
第 5 章 ────────── 记忆与检查点：MemorySaver / SqliteSaver
                    │
第 6 章 ────────── 人在回路：interrupt() / Command(resume=...)
                    │
第 7 章 ────────── 流式执行：stream() / stream_mode
                    │
第 8 章 ────────── 并行与分发：Send / map-reduce 风格
                    │
第 9 章 ────────── 子图与组合：模块化 Research Copilot
                    │
第 10 章 ───────── 部署与扩展：langgraph dev / 函数式 API
```

### 5.2 学习路线总表

| 章节 | 文件名 | LangGraph 主轴 | Python 补课 | 章节交付物 | 难度 |
|------|------|---------------|-------------|-----------|------|
| 第 1 章 | `01-langgraph-overview.md` | 路线图、环境、项目骨架 | 环境与运行习惯 | 学习地图 + 安装验证 | ★☆☆ |
| 第 2 章 | `02-stategraph-basics.md` | `StateGraph`、`START`、`END` | `TypedDict`、类型注解 | Research Copilot v0 | ★☆☆ |
| 第 3 章 | `03-state-and-messages.md` | `MessagesState`、`add_messages` | `Annotated`、消息列表 | Research Copilot v1 | ★★☆ |
| 第 4 章 | `04-tools-and-react.md` | `ToolNode`、`tools_condition`、`create_react_agent` | 装饰器、函数签名、docstring | Research Copilot v2 | ★★☆ |
| 第 5 章 | `05-checkpointer-memory.md` | `MemorySaver`、`SqliteSaver`、`thread_id` | `with`、SQLite 直觉 | Research Copilot v3 | ★★☆ |
| 第 6 章 | `06-human-in-the-loop.md` | `interrupt()`、`Command` | 联合类型、分支返回 | Research Copilot v4 | ★★★ |
| 第 7 章 | `07-streaming.md` | `stream()`、`stream_mode` | 迭代器 / 生成器直觉 | Research Copilot v5 | ★★☆ |
| 第 8 章 | `08-send-and-parallel.md` | `Send` | 列表推导式、任务拆分 | Research Copilot v6 | ★★★ |
| 第 9 章 | `09-subgraphs-and-composition.md` | 子图、组合式架构 | 模块化、文件组织 | Research Copilot v7 | ★★★ |
| 第 10 章 | `10-deployment-and-beyond.md` | `langgraph dev`、函数式 API | 包结构、`__init__.py` | Research Copilot v8 | ★★★ |

---

## 六、嵌入式 Python 补课地图：不是“先补一整门”，而是按需落地

用户这次的目标很明确：**LangGraph 要学，Python 也要真正跟上。**

本系列不再单独开一门脱离上下文的 Python 预科，而采用“嵌入式补课”：

- 只补当前马上要用到的知识
- 每次补课都直接落到当章代码
- 让“语法”立刻转化成“项目能力”

### 6.1 Python 补课地图（含落地代码）

| 章节 | Python 补课主题 | 这章会把它写成什么代码 |
|------|----------------|------------------------|
| 第 2 章 | `TypedDict`、函数返回部分状态 | 写出最小 `ResearchState` 与 3 个节点函数 |
| 第 3 章 | `Annotated`、消息列表结构 | 写出带 `messages` reducer 的状态定义 |
| 第 4 章 | 装饰器、参数说明、docstring | 写出 `@tool` 搜索工具并接入图 |
| 第 5 章 | `with`、SQLite 连接、配置字典 | 写出 `SqliteSaver` 版本的持久化图 |
| 第 6 章 | 联合类型、条件分支返回 | 写出带 `Command(goto=...)` 的审核流 |
| 第 7 章 | 迭代器 / 生成器直觉 | 写出 `for chunk in graph.stream(...)` 迭代代码 |
| 第 8 章 | 列表推导式、聚合列表 | 写出 `Send(...)` 并行扇出逻辑 |
| 第 9 章 | 模块拆分、导入组织 | 把单文件图拆成 `state.py / nodes.py / graphs/` |
| 第 10 章 | 包结构、配置文件 | 写出 `langgraph.json` 与应用入口 |

> [!info] 这套补课的真正目标
> 不是把你训练成“Python 语法考试选手”，而是让你具备一种更实用的能力：
>
> **看到 LangGraph 代码时，不会被 Python 语法卡住。**

---

## 七、章节依赖说明：哪些章节不建议跳过？

### 7.1 强依赖链

这条链几乎不能跳：

```text
第 2 章 → 第 3 章 → 第 4 章 → 第 5 章
```

原因：

- 第 2 章建立 state / node / edge 的基础心智模型
- 第 3 章把“普通状态字典”升级为“消息驱动状态”
- 第 4 章才开始工具调用与 agent 回路
- 第 5 章再加入记忆与检查点

### 7.2 中后段依赖

- 第 6 章依赖第 5 章，因为 `interrupt()` 通常与 checkpointer 一起理解更自然
- 第 7 章依赖第 4-5 章，因为流式展示的内容往往来自真正的图执行
- 第 8 章依赖第 3-4 章，因为并行任务要基于已有状态与工具思维
- 第 9 章依赖第 4-8 章，因为模块化拆分前，必须先知道系统有哪些稳定能力
- 第 10 章依赖全系列，因为它是应用级整理，而不是新玩具功能

### 7.3 可跳但不建议跳的原因

理论上，你可以直接看 `create_react_agent` 或 `langgraph dev`。

但那样你会很容易掉进这类误区：

- 只会“抄一个能跑的 agent helper”
- 不知道状态为什么这样设计
- 一加记忆或审批就开始混乱
- 看到图结构问题时，只会继续堆 prompt

所以本系列坚持：

> **先学图思维，再学高层便利工具。**

---

## 八、这一章结束后，你应该已经具备什么？

如果本章真的起到了“开学总纲”的作用，那么你现在应该至少具备以下成果：

### 8.1 认知层成果

- 你知道 LangGraph 主要解决什么问题
- 你知道本系列的 10 章主路线不会乱跳
- 你知道 Research Copilot 才是整套课的骨架
- 你知道 Python 补课会在每章真正落地，而不是纸上承诺

### 8.2 工程层成果

- 你完成了 `uv` 初始化与安装
- 你完成了最小 import 验证
- 你知道接下来会采用什么目录结构
- 你知道第 2 章要交付的是一个最小可运行的 Research Copilot v0

### 8.3 自测题

你可以先不看答案，试着口头回答：

1. 为什么本系列不把 `create_react_agent` 放在第 2 章开头？
2. 为什么 Research Copilot 比“算术计算器 agent”更适合做完整系列主线？
3. 为什么 Python 补课必须嵌入每章，而不是只在第 1 章给一张表？
4. 为什么第 2 章之前要先完成环境和目录约定？

如果这 4 个问题你都能说清楚，说明你已经不是“只是看了个概览”，而是真的准备好开学了。

---

## 常见误区

| 误区 | 正确理解 |
|------|----------|
| “LangGraph 就是再一个 agent 库。” | 不够准确。它更像一个围绕共享状态组织复杂工作流的执行框架。 |
| “只要模型够强，就不需要显式流程。” | 错。任务越复杂，越需要把状态、节点、边、暂停点和恢复策略说清楚。 |
| “先学高层 helper 最快。” | 短期快，长期黑盒。先学图思维，后学 helper，更稳。 |
| “Python 不够好，就没法学 LangGraph。” | 不必等“全会了再学”。关键是把最小 Python 能力嵌进真实项目里。 |
| “环境准备不重要，先抄代码再说。” | 环境与目录没定清楚，后面每一章的项目连续性都会断掉。 |

---

## 思考题

> [!note] 如何使用
> 先自己回答，再展开提示与参考答案。你会更容易分清自己卡在“概念”还是“术语”。

### 基础理解

**Q1. 为什么本系列坚持先学 `StateGraph`，再讲 `create_react_agent`？**

> [!hint]- 💡 提示
> 想想“理解系统结构”和“调用高层 helper”之间的差别。

> [!success]- ✅ 参考答案
> 因为 `create_react_agent` 是高层便利工具，它能让你很快跑通一个 agent，但容易掩盖状态、节点、边和路由的底层结构。系统学习时，更重要的是先掌握图思维：流程如何流动、状态如何累积、工具调用为什么要有明确节点。等理解了这些，再看高层 helper，才知道它到底帮你自动做了什么。

**Q2. 为什么第 1 章要把 10 章顺序、项目目录、交付物一次性讲清楚？**

> [!hint]- 💡 提示
> 想想系列课程最怕什么：中途改名、导航错乱、项目断层。

> [!success]- ✅ 参考答案
> 因为系统课程最怕“每章各写各的”，最后路线图、文件名、项目成长线和导航互相打架。第 1 章提前固定 10 章顺序、目录结构和交付物，等于把全系列的契约先写清楚。这样第 2-10 章都能围绕同一骨架稳定递进。

### 深入思考

**Q3. “LangGraph 主要提升系统组织能力，而不是模型智商” 这句话具体是什么意思？**

> [!hint]- 💡 提示
> 节点、边、检查点、中断、并行分发，这些是在改模型参数，还是在改流程结构？

> [!success]- ✅ 参考答案
> 这句话的意思是：LangGraph 不改变模型本身的参数能力，而是改善模型周围的执行框架。它通过图结构定义“先做什么、后做什么、哪些信息共享、何时暂停、如何恢复、哪些步骤并行”，从而把原本混乱的 agent 脚本变成一个更稳定、可解释、可维护的系统。

**Q4. 为什么本系列要用“嵌入式 Python 补课”而不是单开一个 Python 系列？**

> [!hint]- 💡 提示
> 想想学习动机、记忆效果和立即落地这三个维度。

> [!success]- ✅ 参考答案
> 因为嵌入式补课能让每个 Python 知识点立刻服务当前任务。例如第 2 章学 `TypedDict` 就马上用于状态定义，第 4 章学装饰器就马上用于 `@tool`，第 7 章学迭代器就马上用于 `stream()`。这样知识不是孤立记忆，而是被项目主线即时消化，学习效率更高。

---

## 关键要点回顾

- ✅ LangGraph 更擅长解决“复杂任务如何组织执行”，而不是单纯“如何多调几次模型”
- ✅ 本系列固定为 10 章，先固化路线，再逐章扩展内容
- ✅ 贯穿项目统一使用 Research Copilot，让每章都推动同一个系统成长
- ✅ 第 1 章必须承担“开学总纲”职责：环境、项目目录、学习路线、交付物、Python 补课地图都要定清楚
- ✅ Python 复习采用嵌入式补课：每章补当前必需、并立即落到代码
- ✅ 本系列主线是现代 LangGraph API，而不是旧式 `AgentExecutor` / while-loop agent 路线

---

## 扩展阅读

- 📄 [LangGraph Overview](https://docs.langchain.com/oss/python/langgraph/overview) — 官方总览，适合建立“状态化工作流”第一印象
- 📄 [Quickstart](https://docs.langchain.com/oss/python/langgraph/quickstart) — 对照图 API 与函数式 API 的官方入门页
- 📄 [Graph API](https://docs.langchain.com/oss/python/langgraph/graph-api) — 后续第 2-9 章最重要的底层参考之一
- 📄 [Streaming](https://docs.langchain.com/oss/python/langgraph/streaming) — 提前预习第 7 章将解决的问题
- 📝 [[research/01-topics/agent-memory|Agent Memory 研究方向]] — 从研究助手视角理解为什么后面要学记忆与检查点

## 相关页面

- [[raw/lessons/LangGraph/02-stategraph-basics|第 2 章：StateGraph、节点与状态——让研究助手先跑起来]]
- [[wiki/concepts/state-graph|StateGraph]]
- [[wiki/concepts/langgraph-api-map|LangGraph API 学习地图]]
- [[research/01-topics/agent-memory|Agent Memory 研究方向总览]]
- [[wiki/concepts/series-navigation-consistency|系列课程导航一致性检查]]

## 下一步学习

下一章我们不再停留在“地图”层面，而是正式写出第一个最小可运行的 Research Copilot v0：

- 定义 `ResearchState`
- 理解 node / edge / `START` / `END`
- 跑通你的第一张 LangGraph 图

继续阅读：[[raw/lessons/LangGraph/02-stategraph-basics|第 2 章：StateGraph、节点与状态——让研究助手先跑起来]]
