---
type: lesson
tags: [面试, AI-Agent, ReAct, Function-Calling, Multi-Agent, LangChain]
created: 2026-05-19
updated: 2026-05-19
difficulty: advanced
prerequisites: [LLM架构基础, Python编程]
topic: AI Agent系统
status: in-progress
series: {name: "AI Interview Prep", part: 2}
---

# AI Agent 系统——从 ReAct 到多智能体协作

> Agent 是 2024-2026 最热方向。面试必考：理解 Agent = LLM + 规划 + 工具 + 记忆。

## 学习目标

- [ ] 能完整阐述 Agent 的核心架构与各组件作用
- [ ] 理解 ReAct / Plan-and-Execute / 反思模式等经典范式
- [ ] 掌握 Function Calling 的实现原理
- [ ] 能设计 Multi-Agent 协作系统
- [ ] 能对比 LangChain / LangGraph / AutoGen / CrewAI 等框架

## 一、什么是 AI Agent？

### 1.1 直觉理解

> 普通的 LLM 像一个"只会纸上谈兵的书生"——知识丰富但无法行动。
> Agent 给 LLM 加上了"手脚"和"记性"——能感知环境、使用工具、规划行动、记忆经验。

### 1.2 Agent 核心公式

```
Agent = LLM（大脑） + Planning（规划） + Tool Use（工具调用） + Memory（记忆）
```

```
┌──────────────────────────────────────────────┐
│                  AI Agent                     │
│                                              │
│   ┌─────────┐    ┌──────────┐               │
│   │  用户    │───▶│  LLM     │               │
│   │  输入    │    │  (大脑)   │               │
│   └─────────┘    └────┬─────┘               │
│                       │                      │
│              ┌────────┼────────┐             │
│              ▼        ▼        ▼             │
│        ┌─────────┐┌───────┐┌───────┐        │
│        │Planning ││Tool   ││Memory │        │
│        │(规划)   ││Use    ││(记忆) │        │
│        └────┬────┘└───┬───┘└───┬───┘        │
│             │         │        │             │
│             ▼         ▼        ▼             │
│        ┌──────────────────────────┐          │
│        │       外部环境/工具       │          │
│        │  搜索/代码/数据库/API    │          │
│        └──────────────────────────┘          │
└──────────────────────────────────────────────┘
```

### 1.3 Agent vs 普通 LLM 对比

| 维度 | 普通 LLM | AI Agent |
|------|---------|----------|
| 输出 | 纯文本 | 行动（调用API、执行代码） |
| 状态 | 无状态 | 有状态（记忆历史） |
| 推理 | 一次生成 | 观察-思考-行动循环 |
| 工具 | 无 | 可调用外部工具 |
| 自主性 | 被动回答 | 主动规划、纠错 |

## 二、经典 Agent 范式

### 2.1 ReAct（Reasoning + Acting）

**面试必问：解释 ReAct 范式**

ReAct 是目前最主流的 Agent 模式，核心是"思考-行动-观察"循环。

```
用户: "北京今天适合爬山吗？"

思考1: 我需要查询北京今天的天气情况
行动1: call_weather_api(city="北京")
观察1: {"temp": "28°C", "weather": "晴", "humidity": "45%", "aqi": 65}

思考2: 天气晴朗、温度适宜、空气质量良好。
       但我还需要检查是否有高温预警或紫外线指数
行动2: call_weather_alert(city="北京")
观察2: {"alert": null, "uv_index": 6}

思考3: 紫外线指数6属于"强"级别，建议防晒。
       综合来看天气适合爬山，但需要注意防晒和补水。
行动3: 返回最终回答

最终回答: 北京今天天气晴朗，气温28°C，空气质量良好，
         适合户外活动。不过紫外线较强（指数6），建议
         做好防晒措施，多带水。
```

**ReAct 的 Prompt 模板**：

```
你可以使用以下工具：
{tool_descriptions}

请严格按照以下格式回答：

Question: 用户的输入问题
Thought: 你的思考和推理
Action: 要使用的工具名称
Action Input: 工具的输入参数
Observation: 工具返回的结果（由系统填入）
...（Thought/Action/Observation 可重复多次）
Thought: 我现在知道最终答案了
Final Answer: 最终回答
```

> [!hint]- 面试场景：ReAct 的常见问题是什么？怎么解决？
> 1. **死循环**：Agent 反复调用同一工具 → 设最大步数限制
> 2. **工具选择错误**：选了不相关的工具 → 优化工具描述、few-shot 示例
> 3. **幻觉**：编造不存在的工具调用 → 严格校验 Action 格式
> 4. **效率低**：过多中间步骤 → 结合 Plan-and-Execute 减少交互轮次

### 2.2 Plan-and-Execute（先规划后执行）

```
用户: "帮我分析竞品 XXX 并生成报告"

Planning Phase（规划阶段）:
  步骤1: 搜索竞品 XXX 的基本信息
  步骤2: 查找其核心功能和定价
  步骤3: 对比我们产品的差异
  步骤4: 生成竞品分析报告

Execution Phase（逐步骤执行）:
  执行步骤1 → 得到结果 → 检查是否需要调整计划
  执行步骤2 → 得到结果 → 继续下一步
  执行步骤3 → 得到结果 → 继续下一步
  执行步骤4 → 输出最终报告
```

**适用场景**：复杂、多步骤任务，需要全局规划而非逐步试探。

### 2.3 反思模式（Reflection）

```
┌────────────────────────────────────────────┐
│                反思循环                      │
│                                            │
│  生成初始回答 ──▶ 自我评估/反思 ──▶ 改进    │
│       ▲                                │    │
│       └────────────────────────────────┘    │
│           （重复直到满意或达到上限）          │
└────────────────────────────────────────────┘
```

**Reflexion 论文核心**：Agent 执行任务后，自我反思失败原因，将反思结果存入记忆，下次执行时参考。

## 三、Function Calling（函数调用）

### 3.1 核心机制

Function Calling 是 LLM 调用外部工具的标准方式。

```python
# 工具定义（传给 LLM 的 tools 参数）
tools = [
    {
        "type": "function",
        "function": {
            "name": "get_weather",
            "description": "获取指定城市的天气信息",
            "parameters": {
                "type": "object",
                "properties": {
                    "city": {
                        "type": "string",
                        "description": "城市名称"
                    },
                    "unit": {
                        "type": "string",
                        "enum": ["celsius", "fahrenheit"],
                        "description": "温度单位"
                    }
                },
                "required": ["city"]
            }
        }
    }
]
```

### 3.2 Function Calling 完整流程

```
1. 用户: "北京天气怎么样？"

2. LLM 分析意图 → 决定调用 get_weather
   返回: {
       "name": "get_weather",
       "arguments": {"city": "北京", "unit": "celsius"}
   }

3. 应用层解析函数名和参数 → 执行实际函数
   result = get_weather(city="北京", unit="celsius")
   # result = {"temp": 28, "weather": "晴"}

4. 将工具结果附加到对话历史 → 再次调用 LLM
   messages.append({"role": "tool", "content": result})

5. LLM 结合工具结果生成最终回答
   "北京今天天气晴朗，气温28°C。"
```

> [!warning] 易混点：Function Calling 不是 LLM 直接执行代码
> LLM **不直接执行**任何函数！它只是生成结构化的 JSON 指令（函数名 + 参数），
> 由**应用层代码**解析并执行。LLM 是"大脑"下达指令，"手脚"（你的代码）来执行。

### 3.3 Function Calling 面试要点

> [!success]- 场景：如何让 LLM 正确选择工具？
> 1. **工具描述要精确**：description 写清楚什么时候用、什么时候不用
> 2. **参数描述要清晰**：每个参数的 type、description、enum 都很重要
> 3. **Few-shot 示例**：在 system prompt 中给出几个工具使用的例子
> 4. **限制工具数量**：同时提供的工具不要超过 10-20 个（LLM 容易混淆）
> 5. **tool_choice 策略**：
>    - `"auto"`：LLM 自己决定是否调用
>    - `"required"`：强制调用某个工具
>    - `"none"`：不允许调用工具

## 四、Multi-Agent（多智能体协作）

### 4.1 常见协作模式

```
模式1：主从模式（Orchestrator-Worker）
┌──────────┐
│ 主Agent  │ ← 分配任务、汇总结果
└──┬───┬───┘
   │   │
   ▼   ▼
┌─────┐┌─────┐
│子Agent││子Agent│ ← 各自执行专业任务
│(搜索)││(代码)│
└─────┘└─────┘

模式2：辩论模式（Debate）
┌─────┐     ┌─────┐
│Agent│◄───▶│Agent│ ← 相互质疑、纠正
│  A  │     │  B  │
└─────┘     └─────┘

模式3：流水线模式（Pipeline）
┌─────┐   ┌─────┐   ┌─────┐
│Agent│──▶│Agent│──▶│Agent│
│(规划)│   │(执行)│   │(审查)│
└─────┘   └─────┘   └─────┘

模式4：对等协作（Peer）
┌─────┐     ┌─────┐
│Agent│◄───▶│Agent│
│  A  │     │  B  │
└──┬──┘     └──┬──┘
   │           │
   ▼           ▼
┌──────────────────┐
│  共享状态/黑板    │
└──────────────────┘
```

### 4.2 多 Agent 面试设计题

> [!hint]- 面试场景：设计一个"智能客服系统"，要求能处理退款、技术支持、投诉
> **架构设计**：
>
> ```
> 用户消息 → Router Agent（意图分类）
>                │
>        ┌───────┼───────┐
>        ▼       ▼       ▼
>     退款Agent 技术Agent 投诉Agent
>        │       │       │
>        ▼       ▼       ▼
>     退款工具  知识库   升级人工
>        │       │       │
>        └───────┼───────┘
>                ▼
>         汇总Agent → 生成回复
> ```
>
> **关键设计点**：
> 1. Router Agent 用 function calling 做意图分类
> 2. 每个 Agent 有独立的 system prompt 和工具集
> 3. 共享对话历史（通过共享 memory）
> 4. 设兜底策略：Agent 无法处理时升级人工
> 5. 加安全限制：退款金额超阈值需二次确认

## 五、框架对比

| 框架 | 定位 | 核心概念 | 适用场景 | 学习曲线 |
|------|------|---------|---------|---------|
| **LangChain** | Agent 开发框架 | Chain/Agent/Tool | 通用 Agent 开发 | 中 |
| **LangGraph** | 状态图 Agent | State Graph/Node/Edge | 复杂工作流、有环Agent | 中高 |
| **AutoGen** | 多 Agent 协作 | Conversation/Agent | 多 Agent 对话协作 | 低 |
| **CrewAI** | 角色扮演多 Agent | Crew/Agent/Task | 团队协作场景 | 低 |
| **Dify** | 低代码 Agent 平台 | 可视化工作流 | 快速搭建 Agent 应用 | 低 |
| **Semantic Kernel** | 企业级 Agent | Plugin/Planner | 微软生态集成 | 中 |

> [!tip] 面试建议
> 不需要精通所有框架，但要能说出：
> 1. 至少一个框架的核心设计思想
> 2. 框架之间的关键差异
> 3. 什么场景选什么框架

### 5.1 LangGraph 核心概念

LangGraph 是当前最推荐的 Agent 框架，用状态图建模 Agent 工作流：

```python
from langgraph.graph import StateGraph, END

# 1. 定义状态
class AgentState(TypedDict):
    messages: list
    next_action: str

# 2. 定义节点
def router(state):
    """分析用户意图，决定下一步"""
    ...

def search_tool(state):
    """执行搜索"""
    ...

def generate_answer(state):
    """生成最终回答"""
    ...

# 3. 构建图
graph = StateGraph(AgentState)
graph.add_node("router", router)
graph.add_node("search", search_tool)
graph.add_node("answer", generate_answer)

graph.add_edge("router", "search")  # 路由到搜索
graph.add_edge("search", "answer")  # 搜索后回答
graph.add_edge("answer", END)       # 结束

# 4. 条件边（根据状态决定下一步）
graph.add_conditional_edges(
    "router",
    lambda state: state["next_action"],
    {"search": "search", "answer": "answer"}
)
```

## 六、Agent 工程实践要点

### 6.1 Agent 可靠性设计

```
可靠性挑战           解决方案
─────────────       ──────────────
输出格式不稳定      → Structured Output (JSON Schema)
工具选择错误        → 精细的 tool description + few-shot
推理错误/幻觉       → Reflection + 自我验证
长任务中断          → 检查点机制 + 断点续跑
死循环              → 最大步数限制 + 退出条件
上下文溢出          → 记忆压缩 + 摘要
```

### 6.2 Agent 评估

| 维度 | 评估方法 | 指标 |
|------|---------|------|
| 任务完成率 | 人工评估 / LLM-as-Judge | 正确完成比例 |
| 工具使用准确率 | 对比 ground truth | 工具选择正确率 |
| 效率 | 统计步数 | 平均完成步数 |
| 健壮性 | 注入对抗性输入 | 异常处理成功率 |
| 延迟 | 端到端计时 | P50/P95 延迟 |

## 七、面试高频问题

> [!success]- Q1：Agent 和 Chain 的区别是什么？
> **Chain**：固定流程，A→B→C 依次执行，路径确定
> **Agent**：动态决策，根据观察结果决定下一步，路径不确定
> Agent 的核心是**自主决策能力**，而不是执行预设步骤。

> [!success]- Q2：如何防止 Agent 陷入死循环？
> 1. 设最大迭代次数（如 10 轮）
> 2. 检测重复操作（连续 3 次相同 Action → 强制退出）
> 3. 加超时机制
> 4. 在 prompt 中明确告诉 Agent "如果你已经知道答案，直接返回 Final Answer"
> 5. 用 LangGraph 的状态图约束 Agent 的行为空间

> [!success]- Q3：设计一个能写代码的 Agent，需要考虑什么？
> 1. **安全沙箱**：代码执行必须在隔离环境中（Docker/沙箱）
> 2. **错误处理**：代码执行失败 → Agent 看到报错 → 自动修复 → 重试
> 3. **上下文管理**：大项目需要管理文件间的依赖关系
> 4. **测试驱动**：Agent 先写测试，再写实现，用测试验证
> 5. **人工确认**：关键操作（如删除文件、部署）需人工确认

## 关键要点回顾

1. **Agent = LLM + Planning + Tool Use + Memory**，四要素缺一不可
2. **ReAct** 是最经典的 Agent 范式：Thought → Action → Observation 循环
3. **Function Calling** 是工具调用的标准接口——LLM 只生成 JSON，不执行代码
4. **Multi-Agent** 适合复杂任务，常见模式：主从/流水线/辩论
5. **可靠性**是 Agent 工程化的核心挑战：格式约束、死循环防护、错误恢复

## 深度扩展阅读

本篇为基础概览，以下三篇深度扩展笔记覆盖了面试所需的全部细节：

| 扩展笔记 | 核心内容 | 建议时间 |
|----------|---------|---------|
| [[raw/lessons/AI-Interview-Prep/02a-agent-paradigms\|02a Agent 推理范式深度]] | CoT 理论基础（计算/信息论/训练数据三角度）、ReAct 完整 Prompt 模板与失败模式、Plan-and-Execute 架构、Reflexion 自我反思、结构化输出/人在回路/可观测性、高级 Prompt Engineering | 2h |
| [[raw/lessons/AI-Interview-Prep/02b-function-calling\|02b Function Calling 深入]] | 完整 HTTP 请求/响应 JSON、多 Provider 差异、工具设计最佳实践、Structured Output 与 constrained generation、复杂工具编排代码、安全防护（注入攻击/沙箱/权限） | 1.5h |
| [[raw/lessons/AI-Interview-Prep/02c-multi-agent\|02c 多智能体协作深度]] | 4 种协作模式完整设计、多智能体通信协议、LangGraph 深入实践（完整 ReAct Agent 代码）、6 大框架架构对比、Token 成本控制/调试/可观测性 | 2h |

## 导航

← [[raw/lessons/AI-Interview-Prep/01-llm-architecture|第1站：LLM 架构]]
→ [[raw/lessons/AI-Interview-Prep/03-rag|第3站：RAG 检索增强生成]]
→ [[raw/lessons/AI-Interview-Prep/04-memory|第4站：AI Memory 记忆系统]]
