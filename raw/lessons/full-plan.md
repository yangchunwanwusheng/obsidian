---
type: lesson
tags:
  - 课程规划
  - LangChain 1.x
  - LangGraph
  - Agent Engineering
  - Claude Code
  - Harness
  - Python
created: 2026-04-29
updated: 2026-04-29
topic: LangGraph → LangChain 1.x → Claude Code-like Harness 学习与写作总规划
difficulty: intermediate
role: control-document
prerequisites:
  - "[[raw/lessons/LangGraph/10-deployment-and-beyond|第 10 章：部署与扩展——把研究助手整理成本地可运行应用]]"
  - "[[raw/lessons/LangGraph-to-Harness/01-from-graph-to-runtime|第 1 章：从 Graph 到 Runtime——为什么下一步是 LangChain 1.x 与 Harness]]"
sources:
  - raw/lessons/LangGraph-to-Harness/01-from-graph-to-runtime.md
  - raw/lessons/LangGraph-to-Harness/02-from-primitives-to-system-modules.md
  - raw/lessons/LangChain/01-why-langchain-1-x.md
  - raw/lessons/LangChain/02-langchain-1-x-core-abstractions.md
  - raw/lessons/LangChain/03-models-messages-and-provider-boundaries.md
  - raw/lessons/LangChain/04-tools-and-structured-output.md
  - raw/lessons/LangChain/05-prompt-and-context-organization.md
  - raw/lessons/LangChain/06-memory-state-history-boundaries.md
  - raw/lessons/Agent-Engineering/01-claude-code-like-systems-overview.md
  - raw/lessons/Agent-Engineering/02-harness-mvp-architecture.md
  - raw/lessons/Agent-Engineering/03-planner-executor-reviewer-loop.md
  - raw/lessons/Agent-Engineering/04-tool-layer-read-search-edit-run.md
  - raw/lessons/Agent-Engineering/05-permissions-and-confirmation-boundaries.md
  - raw/lessons/Agent-Engineering/06-session-memory-checkpoint-resume.md
  - raw/lessons/Agent-Engineering/07-task-decomposition-and-multi-step-strategy.md
  - raw/lessons/Agent-Engineering/08-project-understanding-and-repo-exploration.md
  - https://docs.langchain.com/oss/python/langchain/overview
  - https://docs.langchain.com/oss/python/langgraph/overview
  - https://github.com/anthropics/claude-code/blob/master/README.md
status: in-progress
---

# LangGraph → LangChain 1.x → Claude Code-like Harness 学习与写作总规划

> 一句话摘要：本文件是后续整套课程与工程路线的单一权威总纲。它不负责讲具体某个 API，而负责定义三条主线分别该写什么、怎么联动、按什么顺序推进，以及怎样防止后续写着写着偏题、重复、失衡。

## 学习目标

完成阅读本规划后，你应该能够：
- [ ] 清楚区分 **LangGraph-to-Harness**、**LangChain 1.x**、**Agent Engineering / Harness** 三条线各自的职责
- [ ] 知道为什么这三条线不能各写各的，而必须按同步点联动
- [ ] 理解后续章节应按什么阶段和什么顺序写
- [ ] 知道每个阶段的目标产出、里程碑与示例 demo 应该是什么
- [ ] 明确后续所有课程都必须坚持 **LangChain 1.x 主线** 和 **Python 正文内联讲解** 这两条硬规则
- [ ] 把本文件当成后续写作与学习的总控制台，而不是一篇普通笔记

## 前置知识

| 知识点 | 要求 | Wiki 参考 |
|--------|------|-----------|
| LangGraph 全系列 | 至少已经建立状态、工具、检查点、流式、子图的整体直觉 | [[raw/lessons/LangGraph/01-langgraph-overview|第 1 章：LangGraph 学习地图与研究助手项目总览]] |
| LangGraph 收官视角 | 知道图系统已经成型，但还不等于完整应用 | [[raw/lessons/LangGraph/10-deployment-and-beyond|第 10 章：部署与扩展——把研究助手整理成本地可运行应用]] |
| 桥接总纲 | 知道下一步要从 Graph 转向 Runtime / Harness 视角 | [[raw/lessons/LangGraph-to-Harness/01-from-graph-to-runtime|第 1 章：从 Graph 到 Runtime——为什么下一步是 LangChain 1.x 与 Harness]] |
| LangChain 1.x 定位 | 知道它是现代 agent engineering 的标准组件层 | [[raw/lessons/LangChain/01-why-langchain-1-x|第 1 章：为什么 LangChain 1.x 是下一步]] |
| Claude Code-like 总览 | 知道 coding agent 是受控执行系统，而不只是聊天机器人 | [[raw/lessons/Agent-Engineering/01-claude-code-like-systems-overview|第 1 章：Claude Code-like 系统总览]] |

---

## 一、为什么必须先写这份 full-plan？

因为后续不是只补几篇笔记，而是要持续推进三条互相咬合的学习线。

如果没有一份统一总纲，最容易出现 5 类问题：

1. **LangChain 线和 Harness 线各讲各的**
2. **桥接系列越写越长，侵占主线**
3. **重复讲概念，但没有统一术语和映射关系**
4. **LangChain 混进旧版本内容，破坏 1.x 主线**
5. **工程系列过早追求“像真实 Claude Code 一样全”，导致失控**

所以这份文件的作用，不是“说明一下计划”，而是：

> **作为后续所有课程、所有新章节、所有系列导航的总控文档。**

### 1.1 它是什么？

它是：
- 三条系列课的统一路线图
- 后续写作顺序与里程碑同步表
- 防漂移规则集合
- 后续自检与维护依据

### 1.2 它不是什么？

它不是：
- 某个具体框架教程正文
- 某一条系列的替代品
- 一个临时灵感收集箱
- 一篇只看一次就结束的计划说明

后面每次新增章节前，都应该先回来检查这份规划。

---

## 二、三条系列分别负责什么？

## 2.1 LangGraph-to-Harness：负责“认知转场”

### 角色
这条线的核心职责不是补 API，而是把你从“我会用 LangGraph 画流程图”推进到“我正在学习设计一个真实运行中的 agent 系统”。

### 它主要回答的问题
- 我从 LangGraph 里到底已经学到了哪些系统原语？
- 为什么下一步不是继续零散堆图能力？
- 为什么要进入 LangChain 1.x 与 Harness 视角？
- framework、runtime、harness 分别处在哪一层？

### 它不负责什么
- 不负责完整讲 LangChain 1.x API
- 不负责完整讲 Harness 具体模块实现
- 不负责重复讲 LangGraph 基础教程

### 定位一句话
> **它是转场层，不是主战场。**

---

## 2.2 LangChain 1.x：负责“标准组件层”

### 角色
这条线负责建立现代 agent engineering 最重要的通用零件认知，而且必须是 **LangChain 1.x** 语境，不允许混入旧版教程主线。

### 它主要回答的问题
- models / messages / tools / structured output 到底是什么？
- 为什么这些抽象值得长期依赖？
- 什么该交给 LangChain 1.x，什么不该交给它？
- 它与 LangGraph 的边界在哪里？
- 这些组件在 Claude Code-like harness 中分别落到哪一层？

### 它不负责什么
- 不负责讲复杂路由控制细节
- 不负责代替 Harness 系列讲系统装配
- 不负责沦为“API 清单课”

### 定位一句话
> **它是标准零件层，不是最终整机。**

---

## 2.3 Agent Engineering / Harness：负责“真实系统装配”

### 角色
这条线是最工程化的一条，负责把前两条线中得到的认知真正装配成 Claude Code-like harness 的系统视图。

### 它主要回答的问题
- 一个 coding agent 最小由哪些模块组成？
- plan / act / review / verify 应该如何组织？
- 工具、权限、状态、恢复、验证怎样变成工程系统？
- 为什么 CLI agent 和普通聊天助手不是一回事？
- 怎样按 MVP → intermediate → advanced 逐步演进，而不是一口气做“大而全”？

### 它不负责什么
- 不负责再讲一遍 LangChain 组件定义
- 不负责只做产品功能介绍
- 不负责陷入“完全复刻真实 Claude Code”的幻觉

### 定位一句话
> **它是系统装配课，不是产品崇拜课。**

---

## 三、三条系列之间如何联动？

## 3.1 总依赖关系

建议你把三条线理解成下面这张三层结构：

```text
已完成 LangGraph 系列
        │
        ▼
LangGraph-to-Harness（桥接层）
        │
        ├──► LangChain 1.x（标准组件层）
        │
        └──► Agent Engineering / Harness（系统装配层）
                     ▲
                     │
        LangChain 1.x 为其提供长期可复用零件认知
```

也就是说：

- 桥接系列负责“定向”
- LangChain 1.x 负责“补齐组件层语言”
- Harness 系列负责“系统装配与工程取舍”

---

## 3.2 联动原则

### 原则 A：桥接系列先行，但不能过长
桥接系列必须先建立整体地图，否则后面会把 LangChain 当孤立框架学，也会把 Harness 当单独项目做。

但它不能写太长，因为它的职责是送你进入主线，而不是自己变成主线。

### 原则 B：LangChain 1.x 与 Harness 要交替推进
如果先把 LangChain 全部讲完，再开 Harness，问题是：
- LangChain 会变成脱离系统落点的 API 笔记
- Harness 会重复定义已经讲过的组件

更好的方式是：
- 先讲一组组件
- 立刻讲这些组件在 Harness 中对应什么位置

### 原则 C：每个重要概念都要有“双落点”
后续每遇到一个关键概念，都应该回答两件事：

1. 在 LangChain 1.x 里，它是什么组件？
2. 在 Harness 里，它最终落到哪个系统模块？

例如：
- tool calling
  - 组件层：模型如何知道工具存在
  - 系统层：工具路由与执行器如何组织
- structured output
  - 组件层：如何得到稳定 schema
  - 系统层：如何让计划、审查、变更摘要可验证

---

## 3.3 同步里程碑表

| 同步里程碑 | 桥接系列负责什么 | LangChain 1.x 负责什么 | Harness 系列负责什么 |
|------|------|------|------|
| M1 世界观搭建 | graph → runtime → harness 三层关系 | 为什么必须是 1.x | Claude Code-like 系统总览 |
| M2 组件意识建立 | 把 LangGraph 原语映射到系统模块 | models / messages / tools | 最小系统模块拆分 |
| M3 执行回路建立 | 从 ReAct 到 runtime loop | tool calling / structured output | planner / executor / reviewer |
| M4 状态与控制建立 | state / checkpoint 的系统意义 | memory / history 的边界 | session / checkpoint / resume |
| M5 工程落地建立 | 路线图与收束 | 从组件到最小 agent | MVP harness 架构 |
| M6 进阶扩展建立 | 进入长期项目建议 | 复杂 agent pattern 与边界 | permission / verification / extensibility |

---

## 四、每个系列应该怎么写？

## 4.1 LangGraph-to-Harness 系列写法

### 推荐总章数
4 到 6 章。

### 建议章节主题
1. 从 Graph 到 Runtime：为什么下一步是 LangChain 1.x 与 Harness
2. 从节点、边、状态到真实 agent 模块映射
3. graph、framework、runtime、harness 四层分工图
4. 从“会画图”到“会设计受控执行系统”
5. 三条系列的路线图与配套阅读方法
6. 收束章：如何进入自己的 coding agent / repo agent / research agent 项目

### 写法重点
- 少堆 API，多做分层图与映射图
- 每章都要告诉读者“下一步该去哪条线继续读”
- 每章都要强调“这章是桥，不是终点”

### 典型输出
- 概念对照表
- 分层架构图
- 路线图
- 系列导航说明

---

## 4.2 LangChain 1.x 系列写法

### 推荐总章数
8 到 10 章。

### 建议章节主题
1. 为什么 LangChain 1.x 是下一步
2. LangChain 1.x 的分层结构与核心抽象
3. Models、Messages 与 Provider Boundaries
4. Tools 与 Structured Output
5. Prompt 与上下文组织：什么应放在哪一层
6. Memory / State / History：什么该交给谁管理
7. LangChain 1.x 与 LangGraph 的边界
8. 从组件到最小 agent：拼出一个可运行原型
9. 为 Harness 做准备：哪些组件后续会直接复用

### 写法重点
- 必须在标题、正文、例子、误区中持续强调：**我们学的是 LangChain 1.x**
- 每章都要讲：
  - 它是什么
  - 它解决什么问题
  - 它不解决什么问题
  - 它在 Harness 中会落到哪一层
- 不要写成“API 词典”

### 典型输出
- 组件说明页
- 最小代码例子
- 结构化输出示例
- 与 Harness 的模块映射表

---

## 4.3 Agent Engineering / Harness 系列写法

### 推荐总章数
10 到 14 章。

### 建议章节主题
1. Claude Code-like 系统总览
2. coding agent 的最小系统模块
3. planner / executor / reviewer 的执行回路
4. 工具层：读文件、搜代码、执行命令、编辑文件
5. 权限与确认：为什么 agent 不能无限行动
6. session、memory、checkpoint、resume
7. 任务分解与多步执行策略
8. 项目理解：仓库扫描、模式发现、局部阅读
9. 验证层：测试、lint、build、总结
10. CLI 交互与用户体验设计
11. 插件 / 扩展点 / provider abstraction
12. 从 MVP 到 intermediate harness
13. 从 intermediate 到 advanced system
14. 复盘：你真正学到的是哪套工程能力

### 写法重点
- 不写成“Claude Code 很强，它有这些功能”
- 要写成“一个 coding agent 系统为什么必须有这些模块”
- 每章都尽量落到下面四类问题之一：
  1. 怎么组织执行
  2. 怎么安全行动
  3. 怎么验证结果
  4. 怎么演进系统

### 典型输出
- 模块职责表
- 执行时序图
- 权限边界清单
- MVP / intermediate / advanced 演进路线图

---

## 五、后续章节应该按什么顺序推进？

这里给出一个可执行的 phase 计划。你后面写作时，尽量按这个顺序推进，而不是想到哪写到哪。

## Phase 0：先建立唯一权威规划

### 目标
先把本文件定为唯一总纲，然后以后每一章都回到这里检查是否偏航。

### 本阶段完成标准
- [x] `[[raw/lessons/full-plan]]` 已创建
- [x] 三条系列角色、边界、同步规则全部明确
- [x] 已定义统一写作标准、里程碑和防漂移规则

### 产出
- 本文件本身

---

## Phase 1：稳住三条线的开篇世界观

### 目标
让三条线都具备稳定入口，但重点先放在“定向”而不是“铺得很广”。

### 推荐顺序
1. 已完成：[[raw/lessons/LangGraph-to-Harness/01-from-graph-to-runtime]]
2. 已完成：[[raw/lessons/LangChain/01-why-langchain-1-x]]
3. 已完成：[[raw/lessons/Agent-Engineering/01-claude-code-like-systems-overview]]
4. 已完成：[[raw/lessons/LangGraph-to-Harness/02-from-primitives-to-system-modules|桥接系列第 2 章]]
5. 已完成：[[raw/lessons/LangChain/02-langchain-1-x-core-abstractions|LangChain 第 2 章]]
6. 已完成：[[raw/lessons/Agent-Engineering/02-harness-mvp-architecture|Harness 第 2 章]]

### 本阶段完成标准
- [x] 读者能够说清三条线分别在什么层
- [x] 读者不再把 LangChain 和 LangGraph 混为一谈
- [x] 读者不再把 Harness 理解成“一个大项目文件夹”

### 里程碑 demo
- 一张三层关系图：LangGraph / LangChain 1.x / Harness

---

## Phase 2：先打牢 LangChain 1.x 的核心组件层

### 目标
让后续 Harness 章节拥有统一语言，不再临时定义组件。

### 推荐顺序
1. 已完成：[[raw/lessons/LangChain/02-langchain-1-x-core-abstractions|LangChain：分层结构与核心抽象]]
2. 已完成：[[raw/lessons/LangChain/03-models-messages-and-provider-boundaries|LangChain：models / messages / provider]]
3. 已完成：[[raw/lessons/LangChain/04-tools-and-structured-output|LangChain：tools / structured output]]
4. 已在桥接线完成：[[raw/lessons/LangGraph-to-Harness/02-from-primitives-to-system-modules|这些组件如何映射到 harness]]
5. 已完成：[[raw/lessons/LangChain/05-prompt-and-context-organization|LangChain：prompt 与上下文组织]]
6. 已完成：[[raw/lessons/LangChain/06-memory-state-history-boundaries|LangChain：memory / state / history 边界]]

### 本阶段完成标准
- [x] 读者能说清 messages / tool / structured output 的系统意义
- [x] 后续 Harness 系列可以直接引用这些术语，不再重复从零定义

### 里程碑 demo
- 一张 tool calling 生命周期图
- 一张 structured output vs 自由文本对比表

---

## Phase 3：推进 Harness MVP 主线

### 目标
把抽象零件变成可解释的最小系统模块。

### 推荐顺序
1. 已完成：[[raw/lessons/Agent-Engineering/02-harness-mvp-architecture|Harness：coding agent 最小系统模块]]
2. 已完成：[[raw/lessons/Agent-Engineering/03-planner-executor-reviewer-loop|Harness：planner / executor / reviewer 回路]]
3. 已完成：[[raw/lessons/Agent-Engineering/04-tool-layer-read-search-edit-run|Harness：工具层]]
4. 已完成：[[raw/lessons/Agent-Engineering/05-permissions-and-confirmation-boundaries|Harness：权限层]]
5. 已完成：[[raw/lessons/Agent-Engineering/06-session-memory-checkpoint-resume|Harness：session / checkpoint / resume]]
6. 下一波待补：verification 层

### 本阶段完成标准
- [x] 读者能说清一个 MVP harness 至少要有哪几层
- [x] 读者能描述受控任务执行回路
- [ ] 读者知道为什么 verification 不是收尾小事，而是系统核心

### 里程碑 demo
- 一张 MVP harness 架构图
- 一张任务执行时序图
- 一张高风险动作确认清单

---

## Phase 4：双线交替深化

### 目标
让 LangChain 与 Harness 不脱节，而是持续互相映射。

### 推荐顺序
1. 已完成：LangChain：prompt 与上下文组织
2. 已完成：Harness：任务分解与多步执行策略
3. 已完成：LangChain：memory / state / history 边界
4. 已完成：Harness：项目理解与代码库探索
5. LangChain：LangChain 1.x 与 LangGraph 的边界
6. Harness：verification 层

### 本阶段完成标准
- [ ] 读者能判断哪些事情适合交给 LangChain，哪些需要自己实现
- [ ] 读者能把状态管理、任务管理、消息历史区分清楚

### 里程碑 demo
- 一张“组件到系统”的映射表
- 一张“何时用 LangChain，何时自己实现”的判断表

---

## Phase 5：收束与长期路线图

### 目标
把三条线再收回成一张统一地图，并为真正项目实践做准备。

### 推荐顺序
1. 桥接系列收束章
2. LangChain 系列收束章
3. Harness 系列收束章
4. 总结页：从 LangGraph 到 LangChain 1.x 到 Harness 的完整认知地图

### 本阶段完成标准
- [ ] 新读者只看总览页也能顺着系列学习
- [ ] 三条线的导航一致
- [ ] 后续项目实践路线明确

### 里程碑 demo
- 一张完整课程地图
- 一份“下一步项目建议”：coding agent / repo assistant / research agent

---

## 六、三条线应该怎样交替写，而不是各写各的？

最不推荐的方式是：

- 先把 LangChain 系列全部写完
- 再把 Harness 系列全部写完
- 最后再补桥接总结

这样会产生 3 个问题：

1. LangChain 会脱离真实系统落点
2. Harness 会重复解释前面讲过的组件
3. 桥接会变成事后总结，而不是前置导航

### 推荐交替节奏

采用下面的循环：

```text
桥接定向
  -> 补一组 LangChain 组件
  -> 立刻落到一组 Harness 模块
  -> 再回补下一组 LangChain 组件
  -> 再推进下一组 Harness 工程问题
```

### 建议的最近 6 篇写作顺序

> [!tip] 下一轮推荐 wave
> 当前第一轮世界观、组件骨架、Prompt / Context 组织与多步任务拆解已经串成一个连续骨架。接下来最推荐的推进方式，是从 AE07 继续向后推进，而不是回头重讲已完成章节。

1. [[raw/lessons/LangChain/07-langchain-vs-langgraph-boundaries|LangChain 第 7 章：LangChain 1.x 与 LangGraph 的边界——什么时候该上图，什么时候停在组件层]]
2. [[raw/lessons/Agent-Engineering/09-verification-layer-tests-lint-build-summary|Harness 第 9 章：验证层——为什么测试、lint、build 与总结不是收尾小事]]
3. [[raw/lessons/LangChain/08-from-components-to-minimal-agent|LangChain 第 8 章：从组件到最小 agent——什么时候开始把零件装起来]]
4. [[raw/lessons/Agent-Engineering/10-cli-interaction-and-user-experience|Harness 第 10 章：CLI 交互与用户体验设计]]
5. [[raw/lessons/Agent-Engineering/11-plugin-extension-provider-abstraction|Harness 第 11 章：插件 / 扩展点 / provider abstraction]]
6. [[raw/lessons/Agent-Engineering/12-from-mvp-to-intermediate-harness|Harness 第 12 章：从 MVP 到 intermediate harness]]

注意：这里给出的是当前最稳的下一波联动顺序，不是不可更改的唯一答案；如果后续优先级变化，应同步回写本文件，而不是在系列里各自偏航。

---

## 七、统一写作标准

## 7.1 所有系列的共同标准

### 标准 A：每章都必须同时覆盖三层
每一章尽量都要覆盖：
1. 直觉层：它像什么，解决什么痛点
2. 技术层：它在系统里的输入输出、结构和边界
3. 工程层：它在 Claude Code-like harness 中最终落到哪里

### 标准 B：每章必须有具体例子
例子优先使用同一批可持续复用的场景：
- coding agent
- repo assistant
- research assistant

不建议每一章都换完全不同的业务例子，否则系统连续性会被打断。

### 标准 C：每章都必须回答“它在系统里有什么用”
不能写只停留在“定义是什么”的笔记。

必须写清：
- 为什么需要它
- 没有它会出什么问题
- 它会在后面的 harness 中落到哪个位置

---

## 7.2 Python 正文内联讲解规则

这是硬规则，后续所有系列都必须遵守。

### 原则
- Python 语法与概念必须在正文里就地讲
- 就算重复，也优先保证当前上下文可读
- 不要求读者先去单独补完 Python 再回来

### 必须反复内联解释的内容
只要本章用到了，就可以重复讲：
- `def`
- 参数与返回值
- `list` / `dict`
- `class`
- `Enum`
- `dataclass`
- `pathlib`
- `import`
- 类型标注
- `try/except`
- `async/await`
- JSON / 序列化

### 推荐解释模板
1. 这段代码要解决什么问题
2. 先看整体结构
3. 再拆关键语法
4. 最后解释它在本章系统位置中的作用

### 防失控规则
- 只讲当前例子必须懂的 Python
- 不要把单章变成独立 Python 语法课
- Python 解释必须始终服务于 agent / harness 主线

### 重复衰减规则
为避免“每一章都把同一语法完整重讲一遍”导致篇幅失控，后续统一采用下面的衰减策略：

- **第 1 次出现**：使用完整 4 步模板（用途 → 整体结构 → 关键语法 → 系统位置）
- **第 2 次出现**：一句话快速回顾 + 指向首次系统解释出现的位置
- **第 3 次及以后**：只在当前上下文确实会卡住读者时，用行内说明或简短注释补充，不再展开成长段

这样做的原则不是“省略教学”，而是：
- 首次要讲透
- 第二次要帮回忆
- 后续只在真正阻塞理解时再补

---

## 7.3 系列特有标准

### 桥接系列特有标准
- 偏概念地图、分层图、术语对照
- 少堆 API
- 每章末尾必须强引导“接下来去哪条线继续学”

### LangChain 1.x 系列特有标准
- 必须显式写出 **1.x**
- 若提到旧资料风险，只能作为误区提醒，不得变成主线
- 每章都要写“它解决什么 / 它不解决什么 / 它在 Harness 中落到哪里”

### Harness 系列特有标准
- 必须转译为工程原则，不做产品宣传
- 每章要尽量对应以下四类问题之一：
  - 怎么组织执行
  - 怎么安全行动
  - 怎么验证结果
  - 怎么演进系统

---

## 八、每个阶段要交付哪些里程碑输出？

## Phase 0 输出
- [x] 一份完整 `full-plan.md`
- [x] 三系列总表
- [x] phase 路线图
- [x] 防漂移规则表

## Phase 1 输出
- [x] 三层世界观图
- [x] 三个系列的稳定开篇
- [x] 统一术语表

## Phase 2 输出
- [x] tool calling 生命周期图
- [x] structured output 示例
- [x] LangChain 核心组件图

## Phase 3 输出
- [x] MVP harness 架构图
- [x] planner -> executor -> reviewer 回路图
- [x] 权限确认节点清单

## Phase 4 输出
- [ ] “何时用 LangChain，何时自己实现”的判断表
- [ ] prompt / context / task decomposition 的跨系列映射页
- [ ] 项目理解、验证层与组件边界的联动总表

## Phase 5 输出
- [ ] 统一课程总地图
- [ ] 后续项目建议清单
- [ ] 系列导航收束页

---

## 九、风险与防漂移规则

## 9.1 风险：LangChain 被旧资料污染

### 表现
- 标题里写 LangChain，正文却混入旧版范式
- 出现过时 API、旧教程思路、历史链式写法主导全章

### 约束规则
- 每次写新章都先确认是否处于 **1.x 文档语境**
- 旧资料最多作为“误区提醒”出现
- 不得把旧版内容写成主线教学

---

## 9.2 风险：Harness 系列变成功能介绍

### 表现
- 一直在列 Claude Code 能做什么
- 却没有提炼成模块、边界、流程和工程原则

### 约束规则
- 每章至少输出一项工程结构：模块表、流程图、边界规则或演进路径
- 不允许停留在产品表象崇拜层

---

## 9.3 风险：桥接系列过长，侵占主线

### 表现
- 桥接系列越来越像总教程
- 重复主系列内容

### 约束规则
- 一旦进入组件定义，转到 LangChain 1.x
- 一旦进入系统装配，转到 Harness
- 桥接系列只负责导航、分层、映射、转场

---

## 9.4 风险：Python 内联教学失控

### 表现
- 一章里花大量篇幅讲语言本身
- 主线概念被语法细节打断

### 约束规则
- 只讲当前例子所需 Python
- 解释必须始终绑在 agent / harness 场景上
- 不延展成完整 Python 教材

---

## 9.5 风险：三系列不同步

### 表现
- Harness 开始使用尚未在 LangChain 线中定义的概念
- 或 LangChain 讲了很深，但工程线没有落点

### 约束规则
- 每完成 2 到 3 章，就回看本文件里的同步里程碑表
- 每个新章节必须检查：它依赖的概念是否已经在前置系列中落地

---

## 9.6 风险：过早追求“完全复刻真实 Claude Code”

### 表现
- 一开始就想覆盖全部表面、全部插件、全部产品体验
- 导致规划失真、实施失控

### 约束规则
- 先按 MVP → intermediate → advanced 演进
- 先构建公开可复现能力面
- 把“完全覆盖真实功能”视为长期渐近目标，而不是短期承诺

---

## 十、如何把本文件当成真正的权威路线图使用？

## 10.1 推荐位置
本文件应固定放在：
- [[raw/lessons/full-plan|raw/lessons/full-plan.md]]

### 为什么放这里？
因为它属于：
- 课程写作与学习路线总控文档

而不是：
- `wiki/concepts/` 中的某个概念页
- `research/` 中的项目实验文档

它应该和各系列 lesson 并列，方便后续持续引用。

---

## 10.2 使用规则

后续每次准备写新章前，都先回来检查以下 4 件事：

1. 它属于哪个 phase？
2. 它服务哪条系列？
3. 它依赖的前置概念是否已经落地？
4. 它写完后应该把读者引向哪条线的下一章？

如果这 4 个问题回答不清，就说明这一章还不该写。

---

## 10.3 推荐维护方式

### 每次新增章节后，更新本文件中的两类信息
1. 当前进度追踪区
2. 最近 6 篇推荐顺序

### 每完成一个 phase 后，做一次小复盘
检查：
- 是否仍然遵守 LangChain 1.x 主线
- 是否 Python 仍然是正文内联讲解
- 是否三条线仍然按里程碑同步
- 是否出现概念重复却没有新增系统价值的章节

---

## 十一、当前进度追踪

## 11.1 已完成
- [x] [[raw/lessons/LangGraph-to-Harness/01-from-graph-to-runtime|桥接系列第 1 章]]
- [x] [[raw/lessons/LangGraph-to-Harness/02-from-primitives-to-system-modules|桥接系列第 2 章]]
- [x] [[raw/lessons/LangChain/01-why-langchain-1-x|LangChain 1.x 系列第 1 章]]
- [x] [[raw/lessons/LangChain/02-langchain-1-x-core-abstractions|LangChain 1.x 系列第 2 章]]
- [x] [[raw/lessons/LangChain/03-models-messages-and-provider-boundaries|LangChain 1.x 系列第 3 章]]
- [x] [[raw/lessons/LangChain/04-tools-and-structured-output|LangChain 1.x 系列第 4 章]]
- [x] [[raw/lessons/LangChain/05-prompt-and-context-organization|LangChain 1.x 系列第 5 章]]
- [x] [[raw/lessons/LangChain/06-memory-state-history-boundaries|LangChain 1.x 系列第 6 章]]
- [x] [[raw/lessons/Agent-Engineering/01-claude-code-like-systems-overview|Harness 系列第 1 章]]
- [x] [[raw/lessons/Agent-Engineering/02-harness-mvp-architecture|Harness 系列第 2 章]]
- [x] [[raw/lessons/Agent-Engineering/03-planner-executor-reviewer-loop|Harness 系列第 3 章]]
- [x] [[raw/lessons/Agent-Engineering/04-tool-layer-read-search-edit-run|Harness 系列第 4 章]]
- [x] [[raw/lessons/Agent-Engineering/05-permissions-and-confirmation-boundaries|Harness 系列第 5 章]]
- [x] [[raw/lessons/Agent-Engineering/06-session-memory-checkpoint-resume|Harness 系列第 6 章]]
- [x] [[raw/lessons/Agent-Engineering/07-task-decomposition-and-multi-step-strategy|Harness 系列第 7 章]]
- [x] [[raw/lessons/Agent-Engineering/08-project-understanding-and-repo-exploration|Harness 系列第 8 章]]
- [x] [[raw/lessons/full-plan|本总规划文件]]

## 11.2 当前最推荐的下一步顺序

> [!info] 当前状态说明
> 当前第一轮桥接、LangChain 核心组件层，以及 Harness MVP 主线已经形成一个完整骨架。下一步不再是回头补旧 wave，而是进入 Phase 4 的交替深化阶段。

当前最自然的后续方向通常有两种：

- **方向 A：严格按三线交替继续推进**
  - LangChain：LangChain 1.x 与 LangGraph 的边界
  - Harness：verification 层
  - LangChain：从组件到最小 agent
  - Harness：CLI 交互与用户体验设计

- **方向 B：优先补强 Harness 主线，再回补组件封边**
  - Harness：verification 层
  - Harness：CLI 交互与用户体验设计
  - Harness：插件 / 扩展点 / provider abstraction
  - LangChain：LangChain 1.x 与 LangGraph 的边界

建议：优先采用**方向 A**。因为 prompt / context 组织、memory / state / history，以及仓库探索四章已经把“模型眼前有什么”“系统内部现在知道什么”“面对陌生仓库时先如何建立现场感”这一整条执行前链路补齐了，下一步最自然的承接，就是回到 LangChain 与 LangGraph 的边界，再继续推进 verification 层。

---

## 十二、计划自检清单

在你后续继续写任何一篇新课之前，先用这份清单自检：

- [ ] 它是否明确属于某一条系列？
- [ ] 它是否服务于当前 phase，而不是乱跳？
- [ ] 它是否仍然坚持 **LangChain 1.x** 主线？
- [ ] 它是否会在正文内联讲清相关 Python？
- [ ] 它是否给出系统落点，而不只是定义？
- [ ] 它是否有具体例子和后续导航？
- [ ] 它是否与另外两条线形成联动，而不是孤立存在？

如果其中有 2 项以上答不上来，就不要急着写正文，先回到本规划修正路线。

---

## 相关页面

- [[raw/lessons/LangGraph-to-Harness/01-from-graph-to-runtime|第 1 章：从 Graph 到 Runtime——为什么下一步是 LangChain 1.x 与 Harness]]
- [[raw/lessons/LangGraph-to-Harness/02-from-primitives-to-system-modules|第 2 章：从原语到系统模块——LangGraph 能力如何落到真实 Agent 架构]]
- [[raw/lessons/LangChain/01-why-langchain-1-x|第 1 章：为什么 LangChain 1.x 是下一步]]
- [[raw/lessons/LangChain/04-tools-and-structured-output|第 4 章：Tools 与 Structured Output——让结果可调用、可验证、可组合]]
- [[raw/lessons/LangChain/05-prompt-and-context-organization|第 5 章：Prompt 与上下文组织——什么应放在哪一层]]
- [[raw/lessons/LangChain/06-memory-state-history-boundaries|第 6 章：Memory / State / History——哪些属于组件层，哪些属于系统层]]
- [[raw/lessons/Agent-Engineering/01-claude-code-like-systems-overview|第 1 章：Claude Code-like 系统总览]]
- [[raw/lessons/Agent-Engineering/06-session-memory-checkpoint-resume|第 6 章：Session、Memory、Checkpoint、Resume——为什么不能把状态全塞进一个记忆盒子]]
- [[raw/lessons/Agent-Engineering/07-task-decomposition-and-multi-step-strategy|第 7 章：任务分解与多步执行策略——复杂任务为什么不能只靠临场乱试]]
- [[raw/lessons/Agent-Engineering/08-project-understanding-and-repo-exploration|第 8 章：项目理解与仓库探索——为什么 agent 不能一上来就盲改代码]]
- [[wiki/concepts/series-navigation-consistency|系列课程导航一致性检查]]

## 下一步学习

按本规划，当前最合适的下一步已经不再是补 prompt / context 或多步任务拆解本身，而是顺着它们进入 **Phase 4：双线交替深化** 的后续章节。建议优先按下面顺序继续：

1. [[raw/lessons/LangChain/07-langchain-vs-langgraph-boundaries|LangChain 第 7 章：LangChain 1.x 与 LangGraph 的边界——什么时候该上图，什么时候停在组件层]]
2. [[raw/lessons/Agent-Engineering/09-verification-layer-tests-lint-build-summary|Harness 第 9 章：验证层——为什么测试、lint、build 与总结不是收尾小事]]
3. [[raw/lessons/LangChain/08-from-components-to-minimal-agent|LangChain 第 8 章：从组件到最小 agent——什么时候开始把零件装起来]]
4. [[raw/lessons/Agent-Engineering/10-cli-interaction-and-user-experience|Harness 第 10 章：CLI 交互与用户体验设计]]

如果你想暂时偏向系统工程主线，也可以先继续：
- [[raw/lessons/Agent-Engineering/09-verification-layer-tests-lint-build-summary|Harness 第 9 章：验证层——为什么测试、lint、build 与总结不是收尾小事]]
- [[raw/lessons/Agent-Engineering/10-cli-interaction-and-user-experience|Harness 第 10 章：CLI 交互与用户体验设计]]
