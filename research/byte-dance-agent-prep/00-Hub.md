---
project: byte-dance-agent-prep
type: hub
status: in-progress
created: 2026-07-05
updated: 2026-07-05
target_role: 字节跳动 · Agent 应用开发
duration: 8 周（2026-07-05 → 2026-08-30）
owner: Claudian（用户委托）
tags: [hub, project, byte-dance, agent, 2026-summer]
---

# 字节跳动 Agent 应用开发 · 暑假冲刺（项目 Hub）

> **目标**：8 周内从 Python 基础跃迁到字节 Agent 应用开发 P6/P7 水平。
> **起点**：Python 基础（变量 / 循环 / 函数 / 类）
> **终点**：能通过字节 Agent 应用开发技术面 + 拿出 3-5 个高质量实战项目

---

## 🎯 项目概览

### 一句话定位

> **2 个月，从零到字节 Agent 应用开发。**

### 核心数字

| 指标 | 目标 | 验收标准 |
|---|---|---|
| **学习周数** | 8 周 | 路线图 100% 覆盖 |
| **深度笔记** | 16-24 篇 | 每篇 ≥ 5000 字，10/10 自评 |
| **实战项目** | 4 个 | 企业 RAG / Multi-Agent / Function Calling / Agent 评估 |
| **面试题库** | 系统设计 30+ / 真题 50+ | 含解题思路 + 代码示例 |
| **简历项目** | 3 个 | 含 GitHub README + 演示视频 |

### 时间窗口

```
2026-07-05（Day 1）→ 2026-08-30（Day 56，剩余 ≥ 5 天给模拟面试）
├── Week 1-2（基础夯实）：Python 进阶 + LLM 基础
├── Week 3-4（RAG + Agent 核心）：检索 / 工具调用 / 单 Agent 框架
├── Week 5-6（Multi-Agent + 工程化）：多 Agent / Memory / 评估 / 可观测
├── Week 7（系统设计 + 部署）：高并发 / 限流 / 容灾 / 部署
└── Week 8（面试冲刺）：真题 / 简历 / 模拟面试
```

---

## 📚 知识资产（Knowledge）

按主题组织的笔记：

- **[[Knowledge/01-python-advanced|01 Python 进阶]]**（Week 1）
- **[[Knowledge/02-llm-fundamentals|02 LLM 基础]]**（Week 1）
- **[[Knowledge/03-prompt-engineering|03 Prompt Engineering]]**（Week 2）
- **[[Knowledge/04-function-calling|04 Function Calling]]**（Week 2）
- **[[Knowledge/05-rag-system|05 RAG 系统]]**（Week 3）
- **[[Knowledge/06-agent-frameworks|06 Agent 框架]]**（Week 4）
- **[[Knowledge/07-multi-agent|07 Multi-Agent 协作]]**（Week 5）
- **[[Knowledge/08-memory-system|08 Memory 系统]]**（Week 5）
- **[[Knowledge/09-agent-evaluation|09 Agent 评估]]**（Week 6）
- **[[Knowledge/10-observability|10 可观测性]]**（Week 6）
- **[[Knowledge/11-system-design|11 系统设计]]**（Week 7）
- **[[Knowledge/12-deployment|12 部署与运维]]**（Week 7）
- **[[Knowledge/13-interview-bank|13 字节面试题库]]**（Week 8）
- **[[Knowledge/14-resume-projects|14 简历项目模板]]**（Week 8）

---

## 🛠️ 实战项目（Experiments → Results）

4 个项目作为简历亮点：

| # | 项目名 | 技术栈 | 周次 | 目标 |
|---|---|---|---|---|
| **P1** | 企业知识库 RAG | LangChain + FAISS + FastAPI + Streamlit | Week 3 | 多文档 / 长文档 / 结构化 |
| **P2** | Multi-Agent 协同系统 | LangGraph + CrewAI + Redis | Week 5 | 产品/工程/测试 角色协作 |
| **P3** | Function Calling 工具集 | OpenAI SDK + 通义千问 + 错误处理 | Week 6 | 网页/DB/计算/文件 工具 |
| **P4** | Agent 评估 + 可观测性平台 | Langfuse + Prometheus + Grafana | Week 6 | 端到端评估 + 仪表盘 |

每个项目结构：
- `Experiments/<project>/README.md` — 项目说明
- `Experiments/<project>/design.md` — 架构设计
- `Results/<project>/report.md` — 完成后复盘

---

## 📊 当前进度

> 由 Dataview 自动生成（详见 02-Index.md）

```dataview
TABLE WITHOUT ID
  length(filter(rows, (r) => r.week = "W1")) as "Week 1 笔记",
  length(filter(rows, (r) => r.week = "W2")) as "Week 2 笔记",
  length(filter(rows, (r) => r.week = "W3")) as "Week 3 笔记",
  length(filter(rows, (r) => r.week = "W4")) as "Week 4 笔记"
FROM "Research/byte-dance-agent-prep/Knowledge"
WHERE type = "lesson"
GROUP BY "进度统计"
```

---

## 🚀 立即开始

1. 阅读 [[01-Plan|01-Plan.md]]（8 周详细路线图）
2. 从 Week 1 第 1 篇笔记开始：**Python 进阶速成**
3. 每天在 [[Daily/2026-07-05|Daily/]] 写当日学习日志
4. 每周日在 `Daily/` 写周报

---

## ⚠️ 质量纪律

- 每篇笔记必须通过 `quality-control-checker` 自评，**≥ 8/10 才合格**
- 每个项目必须有 README + design + 可运行代码 + 演示
- 每周至少完成 2 篇深度笔记
- 不追求覆盖，追求**深度 + 实战可演示**

---

## 🔗 相关文档

- [[01-Plan|01-Plan.md]] — 8 周路线图
- [[02-Index|02-Index.md]] — 笔记索引（自动统计）
- [[_system/registry|_system/registry.md]] — 项目注册表
- [[../../CLAUDE|../../CLAUDE.md]] — 全局规则（v2.0）