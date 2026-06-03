---
type: concept
tags:
  - LangGraph
  - API地图
  - Graph API
  - Agent
  - 学习路线
created: 2026-04-28
updated: 2026-04-28
sources:
  - raw/lessons/LangGraph/01-langgraph-overview.md
  - raw/lessons/LangGraph/03-state-and-messages.md
  - raw/lessons/LangGraph/04-tools-and-react.md
  - raw/lessons/LangGraph/05-checkpointer-memory.md
  - raw/lessons/LangGraph/06-human-in-the-loop.md
  - raw/lessons/LangGraph/07-streaming.md
  - raw/lessons/LangGraph/08-send-and-parallel.md
  - raw/lessons/LangGraph/09-subgraphs-and-composition.md
  - raw/lessons/LangGraph/10-deployment-and-beyond.md
---

# LangGraph API 学习地图

> 一句话摘要：这是一张面向系统学习的 LangGraph API 路线图。它不追求罗列全部接口，而是把最常用、最值得先掌握的 API 按“从基础骨架到应用结构”的顺序组织起来，帮助你把零散知识点重新串成完整心智模型。

## 概述

LangGraph 的 API 很容易让初学者产生两种极端：

- 要么只记住几个高层 helper，却不知道底层结构
- 要么陷入文档细节，学了很多名字，却不知道先后顺序

这张学习地图的目的，就是回答一个更实用的问题：

> **如果我要真正做出一个可维护的图式 agent，哪些 API 应该先学，彼此是什么关系？**

本页按课程主线整理，优先突出那些能直接支撑 Research Copilot 项目成长的接口。

## 要点

- 第一优先级是 `StateGraph`、`START`、`END`：先建立图骨架直觉
- 第二优先级是消息状态：`MessagesState`、`add_messages`
- 第三优先级是工具回路：`@tool`、`ToolNode`、`tools_condition`
- 第四优先级是运行时能力：checkpointer、`thread_id`、`interrupt()`、`Command`
- 第五优先级是可观测性与扩展能力：`stream()`、`Send`、子图
- 最后再看应用结构与补充视角：`langgraph dev`、`langgraph.json`、Functional API

## 详细内容

### 1. 基础骨架层：先学会“图是怎么长出来的”

#### `StateGraph`
整个系统的底层骨架。围绕共享状态组织节点与边。

#### `START` / `END`
显式标出流程边界，让图不是一团隐式逻辑。

这一层回答的问题是：

- 流程从哪里开始？
- 中间有哪些明确步骤？
- 最后在哪里结束？

对应学习入口：[[wiki/concepts/state-graph|StateGraph]]、[[raw/lessons/LangGraph/02-stategraph-basics|第 2 章]]。

### 2. 对话状态层：让图进入消息驱动模式

#### `MessagesState`
LangGraph 提供的消息态状态模板，适合对话型 agent。

#### `add_messages`
消息 reducer，用于定义消息如何稳定累积，而不是简单列表拼接。

这一层回答的问题是：

- 图怎样保存多轮消息？
- 新消息如何追加进已有上下文？

对应学习入口：[[raw/lessons/LangGraph/03-state-and-messages|第 3 章]]。

### 3. 工具调用层：让系统从“会说”升级为“能做”

#### `@tool`
把普通 Python 函数声明成模型可调用工具。

#### `ToolNode`
标准工具执行节点，负责把工具调用真正跑起来。

#### `tools_condition`
用于判断当前流程是否要进入工具分支。

#### `create_react_agent`
高层 helper，适合在理解底层后再用来加速搭建。

这一层回答的问题是：

- 模型何时决定调工具？
- 工具调用如何进入图流程？
- 为什么不要把高层 helper 误当作全部心智模型？

对应学习入口：[[raw/lessons/LangGraph/04-tools-and-react|第 4 章]]。

### 4. 记忆与线程层：让状态跨轮延续

#### `MemorySaver`
最小 checkpointer，适合理解状态保存与恢复机制。

#### `SqliteSaver`
更接近本地持久化场景的 checkpointer。

#### `thread_id`
线程级状态归属标识，是会话持续性的关键。

这一层回答的问题是：

- 为什么“有 messages”还不等于“有跨轮记忆”？
- 同一条研究线怎样稳定接着往下走？

对应学习入口：[[raw/lessons/LangGraph/05-checkpointer-memory|第 5 章]]。

### 5. 人在回路层：让系统学会设计性暂停

#### `interrupt()`
在节点中创建暂停点，把关键信息暴露给外部。

#### `Command(resume=...)`
恢复暂停流程，并把恢复值送回中断点。

#### `Command(goto=...)`
控制恢复后流程走向哪个分支。

这一层回答的问题是：

- 哪些高风险步骤不该让 agent 自己跑到底？
- 暂停后如何恢复同一线程？

对应学习入口：[[raw/lessons/LangGraph/06-human-in-the-loop|第 6 章]]。

### 6. 可观测性层：让图执行过程变得可见

#### `graph.stream()`
边执行边输出中间过程。

#### `stream_mode`
常见模式包括：
- `updates`
- `values`
- `messages`
- `custom`

#### `version="v2"`
现代流式格式中更统一的输出结构。

这一层回答的问题是：

- 为什么复杂图不能只看最终结果？
- 如何看到状态更新、消息输出和业务进度？

对应学习入口：[[raw/lessons/LangGraph/07-streaming|第 7 章]]。

### 7. 并行层：让一个问题拆成多个任务实例

#### `Send`
用于动态扇出多个任务实例，是并行 map-reduce 风格的关键接口。

#### `Annotated[..., operator.add]`
常见聚合规则写法，帮助多个并行任务把结果合并回主状态。

这一层回答的问题是：

- 一个研究问题如何拆成多个并行子任务？
- 多个结果如何安全汇总？

对应学习入口：[[raw/lessons/LangGraph/08-send-and-parallel|第 8 章]]。

### 8. 组合层：让增长中的图重新获得结构

#### 编译子图作为节点接入父图
这是子图组合最核心的能力。

#### `subgraphs=True`
在流式观察中查看父图与子图的更新层次。

这一层回答的问题是：

- 图越来越大后，怎么避免继续堆成一个巨文件？
- 父图和子图怎样通过共享状态协作？

对应学习入口：[[raw/lessons/LangGraph/09-subgraphs-and-composition|第 9 章]]。

### 9. 应用结构层：让图成为真正的本地应用骨架

#### `langgraph dev`
启动本地开发服务器，进入更接近应用的调试方式。

#### `langgraph.json`
定义依赖、graph 入口、环境变量等运行配置。

#### Functional API
作为补充视角，适合把某些工作流包装成更像函数入口的调用体验。

这一层回答的问题是：

- 图写好了之后，如何把它整理成应用？
- 为什么最终要统一入口与目录结构？

对应学习入口：[[raw/lessons/LangGraph/10-deployment-and-beyond|第 10 章]]。

## 一个可执行的学习顺序

```text
StateGraph
  ↓
MessagesState / add_messages
  ↓
@tool / ToolNode / tools_condition
  ↓
MemorySaver / SqliteSaver / thread_id
  ↓
interrupt() / Command
  ↓
stream() / stream_mode / version="v2"
  ↓
Send / 聚合规则
  ↓
subgraphs / 父子图组合
  ↓
langgraph dev / langgraph.json / Functional API
```

## 相关页面

- [[wiki/concepts/state-graph|StateGraph]]
- [[raw/lessons/LangGraph/01-langgraph-overview|第 1 章：LangGraph 学习地图与研究助手项目总览]]
- [[raw/lessons/LangGraph/04-tools-and-react|第 4 章：工具与 ReAct——让研究助手学会调用检索工具]]
- [[raw/lessons/LangGraph/05-checkpointer-memory|第 5 章：记忆与检查点——让研究助手跨轮记住上下文]]
- [[raw/lessons/LangGraph/10-deployment-and-beyond|第 10 章：部署与扩展——把研究助手整理成本地可运行应用]]
