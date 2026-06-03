---
type: concept
tags:
  - LangGraph
  - Graph API
  - StateGraph
  - Agent
  - 工作流
created: 2026-04-28
updated: 2026-04-28
sources:
  - raw/lessons/LangGraph/01-langgraph-overview.md
  - raw/lessons/LangGraph/02-stategraph-basics.md
  - https://docs.langchain.com/oss/python/langgraph/graph-api
---

# StateGraph

> 一句话摘要：`StateGraph` 是 LangGraph 中最核心的图构建原语。它围绕“共享状态如何在多个节点之间流动”来组织工作流，是理解节点、边、路由、子图、并行和记忆能力的总入口。

## 概述

如果把 LangGraph 看成一个用来组织 agent 系统的框架，那么 `StateGraph` 就是最重要的骨架。它要求你先定义一份共享状态，再定义若干节点如何读取和更新这份状态，最后用边把这些步骤连成一个可执行流程。

`StateGraph` 的重要性，不只是“能画图”，而是它把复杂任务从一团黑盒逻辑拆成了更清晰的结构：状态是什么、每一步负责什么、下一步去哪里、哪些步骤可以组合或并行，都能在图中显式表达出来。

## 要点

- `StateGraph` 以 **共享状态** 为中心，而不是以单次模型调用为中心
- 节点通常只返回 **局部 state 更新**，而不是重写全部状态
- `START` / `END` 让流程边界显式化，更适合调试和扩展
- 后续的 `MessagesState`、工具调用、记忆、并行、子图，本质上都建立在 `StateGraph` 的状态流动心智之上
- 对初学者来说，先学 `StateGraph` 往往比先学高层 helper 更能建立稳定的系统直觉

## 详细内容

### 1. 它解决的核心问题是什么？

很多初学者最先遇到的问题不是“模型不会回答”，而是：

- 多步任务应该怎么拆？
- 每一步做完后要把什么信息留给下一步？
- 哪些字段应该长期保存？
- 为什么系统会走到这一步？

`StateGraph` 的答案是：

1. 先定义共享状态 schema
2. 再定义节点如何更新状态
3. 再定义步骤之间的流向

这样你得到的不是“一个神秘 agent”，而是一个可解释的工作流系统。

### 2. 最小心智模型

你可以把 `StateGraph` 想成：

- **state** = 共享工作台
- **node** = 在工作台上完成一项明确工作的成员
- **edge** = 下一步交接路线

最小形态通常像这样：

```text
START -> node_1 -> node_2 -> node_3 -> END
```

而每个节点只负责补充自己那一小步应该产出的状态。

### 3. 为什么 state-first 很重要？

LangGraph 的很多设计质量，其实都来自 state schema 的质量。

如果状态定义清楚：

- 节点职责更容易切分
- 路由条件更容易设计
- 子图输入输出更容易对齐
- 流式调试更容易解释

如果状态定义混乱：

- 节点会职责重叠
- 后续新增工具、记忆、审核会越来越乱
- 父图和子图很容易对不齐字段

### 4. 它和高层 helper 的关系是什么？

像 `create_react_agent` 这类高层 helper 很方便，但它们并没有绕过 `StateGraph` 背后的核心问题：

- 状态如何组织
- 工具如何接入
- 消息如何累积
- 流程如何路由

所以从系统学习角度看，`StateGraph` 更像底层骨架，而高层 helper 是在这个骨架之上帮你省去部分样板工作。

### 5. 它如何延伸到后续能力？

在 LangGraph 课程主线里，很多能力都可以看成 `StateGraph` 的自然扩展：

- `MessagesState`：把普通状态升级成对话态状态
- `ToolNode`：把工具执行变成标准节点
- checkpointer：让状态具备线程级持续性
- `interrupt()`：让图在关键位置暂停并恢复
- `Send`：让状态驱动多个并行任务实例
- 子图：把局部流程继续封装成更大的图节点

## 相关页面

- [[raw/lessons/LangGraph/01-langgraph-overview|第 1 章：LangGraph 学习地图与研究助手项目总览]]
- [[raw/lessons/LangGraph/02-stategraph-basics|第 2 章：StateGraph、节点与状态——让研究助手先跑起来]]
- [[raw/lessons/LangGraph/03-state-and-messages|第 3 章：State 与 Messages——让研究助手真正进入对话态]]
- [[wiki/concepts/langgraph-api-map|LangGraph API 学习地图]]
- [[wiki/concepts/knowledge-compilation|知识编译]]
