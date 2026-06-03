---
type: lesson
tags: [面试, Multi-Agent, 多智能体, Agent框架, LangGraph, AutoGen]
created: 2026-05-20
updated: 2026-05-20
difficulty: advanced
prerequisites: [Agent基础, Function-Calling]
topic: 多智能体协作与框架深入
status: in-progress
series: {name: "AI Interview Prep", part: "2c"}
---

# 多智能体协作与框架深入

> 面向 AI 工程师面试的 Multi-Agent 全景指南：从协作模式到框架选型到系统设计。

## 学习目标

- [ ] 能说出四种多智能体协作模式的适用场景和优缺点
- [ ] 能设计多 Agent 间的通信协议和状态同步方案
- [ ] 能用 LangGraph 实现完整的 ReAct Agent
- [ ] 能对比主流 Agent 框架并给出选型建议
- [ ] 能设计可扩展、可调试的多 Agent 系统

---

## 一、多智能体协作模式深入

### 1.1 主从模式（Orchestrator-Worker）

最常用也最实用的模式。一个中央 Agent 负责理解意图、分配任务、汇总结果。

```
┌────────────────────────────────────────────────┐
│                 Orchestrator                    │
│  职责：理解意图 → 分配任务 → 汇总结果            │
│  组件：Router + Dispatcher + Aggregator         │
└────┬──────────┬──────────┬──────────┬──────────┘
     │          │          │          │
┌────▼───┐ ┌───▼────┐ ┌──▼───┐ ┌───▼────┐
│Worker A│ │Worker B│ │Worker C│ │Worker D│
│ 搜索   │ │ 分析   │ │ 生成  │ │ 审核   │
└────────┘ └────────┘ └──────┘ └────────┘
```

**完整设计**：

```python
from dataclasses import dataclass
from typing import Any

@dataclass
class Task:
    id: str
    description: str
    assigned_to: str
    dependencies: list[str] = None
    result: Any = None

class Orchestrator:
    """主从模式编排器"""

    def __init__(self, llm_client):
        self.client = llm_client
        self.workers: dict[str, "Worker"] = {}

    def register_worker(self, name: str, worker: "Worker"):
        self.workers[name] = worker

    async def run(self, user_request: str) -> str:
        # Step 1: Router — 理解意图，拆分任务
        tasks = await self._route(user_request)

        # Step 2: Dispatcher — 分配并执行任务
        results = {}
        for task in tasks:
            worker = self.workers[task.assigned_to]
            task.result = await worker.execute(task.description)
            results[task.id] = task.result

        # Step 3: Aggregator — 汇总结果
        final = await self._aggregate(user_request, results)
        return final

    async def _route(self, request: str) -> list[Task]:
        """用 LLM 分析用户请求，拆分为子任务"""
        routing_prompt = f"""分析用户请求，拆分为子任务，分配给最合适的 Worker。

可用 Workers: {list(self.workers.keys())}

用户请求: {request}

输出 JSON 格式:
{{
    "tasks": [
        {{
            "id": "task_1",
            "description": "具体任务描述",
            "assigned_to": "worker_name",
            "dependencies": []
        }}
    ]
}}"""
        response = self.client.chat.completions.create(
            model="gpt-4o",
            messages=[{"role": "user", "content": routing_prompt}],
            response_format={"type": "json_object"}
        )
        data = json.loads(response.choices[0].message.content)
        return [Task(**t) for t in data["tasks"]]

    async def _aggregate(self, original_request: str, results: dict) -> str:
        """汇总所有 Worker 的结果"""
        agg_prompt = f"""基于以下子任务结果，回答用户的原始问题。

原始问题: {original_request}

子任务结果:
{json.dumps(results, ensure_ascii=False, indent=2)}

请给出完整的回答。"""
        response = self.client.chat.completions.create(
            model="gpt-4o",
            messages=[{"role": "user", "content": agg_prompt}]
        )
        return response.choices[0].message.content
```

### 1.2 辩论模式（Debate）

多个 Agent 对同一问题给出不同视角，由评委 Agent 做最终裁决。

```python
@dataclass
class DebateRound:
    round_number: int
    debater: str
    argument: str
    critique: str  # 对方论点的反驳

class DebateSystem:
    """辩论模式系统"""

    def __init__(self, llm_client, num_rounds: int = 3):
        self.client = llm_client
        self.num_rounds = num_rounds

    async def debate(self, question: str) -> dict:
        positions = ["支持", "反对"]
        history = []

        for round_num in range(self.num_rounds):
            for i, position in enumerate(positions):
                opponent_pos = positions[1 - i]
                prompt = f"""你是一个辩论者，立场是"{position}"。

论题: {question}
对手立场: {opponent_pos}

{"这是第1轮，请陈述你的论点。" if round_num == 0 else f"前几轮论点: {json.dumps(history[-2:], ensure_ascii=False)}。请回应对手的论点并强化你的立场。"}

请简洁有力地陈述（200字以内）。"""

                resp = self.client.chat.completions.create(
                    model="gpt-4o",
                    messages=[{"role": "user", "content": prompt}]
                )
                history.append(DebateRound(
                    round_number=round_num + 1,
                    debater=position,
                    argument=resp.choices[0].message.content,
                    critique=""
                ))

        # 评委裁决
        judge_prompt = f"""你是一个中立的评委。请基于以下辩论内容给出裁决。

论题: {question}

辩论记录:
{json.dumps([{"round": h.round_number, "debater": h.debater, "argument": h.argument} for h in history], ensure_ascii=False, indent=2)}

请给出:
1. 最终结论（支持/反对/中立）
2. 理由（300字以内）
3. 关键论点分析"""

        judge_resp = self.client.chat.completions.create(
            model="gpt-4o",
            messages=[{"role": "user", "content": judge_prompt}]
        )
        return {
            "question": question,
            "debate_history": history,
            "verdict": judge_resp.choices[0].message.content
        }
```

### 1.3 四种模式对比

| 模式 | 适用场景 | 优点 | 缺点 | 实现复杂度 |
|------|----------|------|------|-----------|
| **主从** | 任务可拆分、结果需汇总 | 结构清晰、易扩展 | 单点瓶颈（Orchestrator） | 中 |
| **辩论** | 需要多角度分析、决策 | 减少偏见、提高鲁棒性 | Token 成本高（多轮） | 中高 |
| **流水线** | 数据需要逐步处理 | 每步专注、可调试 | 串行瓶颈、错误传播 | 中 |
| **层级** | 大规模复杂任务 | 可扩展、职责清晰 | 通信开销大、调试难 | 高 |

**流水线模式示意**：

```
Input → [Agent A: 理解] → [Agent B: 检索] → [Agent C: 生成] → [Agent D: 审核] → Output
         纯文本           检索结果+原文       草稿+检索结果      审核意见+草稿
```

**层级模式示意**：

```
                    ┌──────────┐
                    │ Manager  │  ← 战略决策
                    └──┬───┬───┘
                       │   │
              ┌────────▼┐ ┌▼────────┐
              │ Lead A  │ │ Lead B  │  ← 战术规划
              │(搜索域) │ │(分析域) │
              └──┬───┬──┘ └──┬───┬──┘
                 │   │       │   │
              ┌──▼┐┌▼──┐  ┌─▼─┐┌▼──┐
              │W-1││W-2│  │W-3││W-4│  ← 具体执行
              └───┘└───┘  └───┘└───┘
```

> [!hint]- 面试题：用户要做一个"AI 辅助编程"产品，支持需求分析、代码生成、代码审查、测试编写。你会选择哪种多 Agent 协作模式？为什么？

> [!success]- 参考答案
> 推荐**流水线 + 主从混合模式**：
>
> 1. **主 Orchestrator** 负责理解用户需求，判断走哪条流水线
> 2. **流水线设计**：需求分析 → 代码生成 → 代码审查 → 测试编写
> 3. 代码审查阶段可以用**辩论模式**：两个审查 Agent 分别从"安全性"和"可维护性"角度审查
> 4. 为什么不全用主从？因为编程任务天然是流水线——后一步依赖前一步的输出
> 5. 为什么不全用流水线？因为有时用户只需要代码审查，不需要走全流程，Orchestrator 做路由
>
> 关键设计决策：流水线中任何一步失败，都要有反馈回路（不能简单丢弃）。

---

## 二、多智能体通信

### 2.1 通信协议设计

```python
from dataclasses import dataclass, field
from datetime import datetime
from enum import Enum
import uuid

class MessageType(Enum):
    TASK_ASSIGN = "task_assign"       # 分配任务
    TASK_RESULT = "task_result"       # 任务结果
    QUERY = "query"                   # 询问其他 Agent
    RESPONSE = "response"             # 回答询问
    ERROR = "error"                   # 错误报告
    STATUS_UPDATE = "status_update"   # 状态更新
    CONTROL = "control"               # 控制指令（暂停/取消/重试）

@dataclass
class AgentMessage:
    """统一的 Agent 间消息格式"""
    id: str = field(default_factory=lambda: str(uuid.uuid4()))
    sender: str = ""                  # 发送者 Agent ID
    receiver: str = ""                # 接收者 Agent ID（空=广播）
    msg_type: MessageType = MessageType.QUERY
    content: str = ""                 # 主要内容
    metadata: dict = field(default_factory=dict)
    timestamp: str = field(default_factory=lambda: datetime.now().isoformat())
    reply_to: str = ""                # 回复的消息 ID
    error: str = ""                   # 错误信息（如果有）

    def to_dict(self) -> dict:
        return {
            "id": self.id,
            "sender": self.sender,
            "receiver": self.receiver,
            "type": self.msg_type.value,
            "content": self.content,
            "metadata": self.metadata,
            "timestamp": self.timestamp,
            "reply_to": self.reply_to,
            "error": self.error
        }
```

### 2.2 共享状态 vs 消息传递

| 维度 | 共享状态 | 消息传递 |
|------|----------|----------|
| 耦合度 | 高（都依赖同一状态） | 低（只通过消息交互） |
| 一致性 | 需要加锁/事务 | 自然一致（消息有序） |
| 可调试性 | 状态变化难追踪 | 消息日志天然可追溯 |
| 扩展性 | 状态膨胀后难维护 | 新增 Agent 只需订阅消息 |
| 适用场景 | 强一致性需求（如共编辑） | 松耦合任务协作 |

### 2.3 黑板模式（Blackboard Architecture）

```
┌─────────────────────────────────────────────────┐
│                  Blackboard (共享数据区)          │
│  ┌───────────┬───────────┬───────────┐          │
│  │ 问题定义   │ 中间结果   │ 最终答案   │          │
│  │ "分析这段  │ "情感:正面"│           │          │
│  │  文本"    │ "主题:科技"│           │          │
│  └───────────┴───────────┴───────────┘          │
└──────────┬──────────┬──────────┬────────────────┘
           │          │          │
      ┌────▼───┐ ┌───▼────┐ ┌──▼───┐
      │Agent A │ │Agent B │ │Agent C│
      │情感分析│ │主题提取│ │摘要  │
      └────────┘ └────────┘ └──────┘

特点：各 Agent 独立读写黑板，不需要直接通信
```

```python
class Blackboard:
    """黑板模式实现"""

    def __init__(self):
        self.data: dict[str, Any] = {}
        self.history: list[dict] = []  # 操作日志

    def write(self, key: str, value: Any, agent_id: str):
        old = self.data.get(key)
        self.data[key] = value
        self.history.append({
            "action": "write",
            "key": key,
            "old_value": old,
            "new_value": value,
            "agent": agent_id,
            "timestamp": datetime.now().isoformat()
        })

    def read(self, key: str) -> Any:
        return self.data.get(key)

    def read_all(self) -> dict:
        return dict(self.data)

    def subscribe(self, key: str, callback):
        """订阅某个 key 的变化（简化实现）"""
        pass  # 实际生产中用观察者模式
```

### 2.4 通信频率控制

每个 LLM 调用都在花钱和花时间。设计原则：

1. **批量通信**：攒够一批消息再发送，而不是逐条
2. **摘要压缩**：长对话历史在传递给其他 Agent 前做摘要
3. **条件通信**：只在结果有意义时才传递（跳过空结果、错误重试）
4. **本地决策**：能本地判断的不要问 LLM

```python
class CommunicationBudget:
    """通信预算控制器"""

    def __init__(self, max_calls_per_agent: int = 10, max_total_calls: int = 50):
        self.max_per_agent = max_calls_per_agent
        self.max_total = max_total_calls
        self.calls: dict[str, int] = {}  # agent_id → call_count
        self.total_calls = 0

    def can_call(self, agent_id: str) -> bool:
        if self.total_calls >= self.max_total:
            return False
        if self.calls.get(agent_id, 0) >= self.max_per_agent:
            return False
        return True

    def record_call(self, agent_id: str):
        self.calls[agent_id] = self.calls.get(agent_id, 0) + 1
        self.total_calls += 1
```

### 2.5 面试场景：设计 3-Agent 客服系统

> [!hint]- 面试题：用 3 个 Agent 设计一个智能客服系统，要求能处理咨询、退换货、投诉三类问题。请给出 Agent 设计、通信方式、错误处理方案。

> [!success]- 参考答案
> **Agent 设计**：
>
> 1. **Router Agent**：理解用户意图，分类为咨询/退换货/投诉，转发给对应 Agent
> 2. **Service Agent**：处理咨询和退换货（查订单、查库存、生成退货单）
> 3. **Escalation Agent**：处理投诉和复杂问题（有情绪分析能力，能安抚用户）
>
> **通信方式**：主从模式 + 消息传递
> - Router → Service/Escalation：传递用户消息 + 意图分类
> - Service/Escalation → Router：传递处理结果
> - Router → 用户：传递最终回复
>
> **错误处理**：
> - Service Agent 处理失败 → 转给 Escalation Agent
> - Escalation Agent 也处理不了 → 标记为"需人工介入"，保存上下文
> - Router 分类不确定 → 先尝试 Service，如果用户反馈不对则转 Escalation
>
> **关键设计**：
> - 所有 Agent 共享对话历史（黑板模式）
> - 情绪检测前置：如果用户情绪激动，直接转 Escalation
> - 工具权限隔离：Service 只能查询，不能退款；Escalation 可以申请退款

---

## 三、LangGraph 深入实践

### 3.1 StateGraph 核心概念

LangGraph 是 LangChain 团队推出的有状态多步 Agent 框架，核心是基于图的状态机。

```
┌────────────────────────────────────────────────┐
│  StateGraph 核心概念                            │
├────────────────────────────────────────────────┤
│                                                │
│  State ─── 图中流转的共享状态对象               │
│    │                                           │
│  Node ─── 处理状态的函数                        │
│    │                                           │
│  Edge ─── 节点之间的连接（条件/固定）           │
│    │                                           │
│  Checkpointer ─── 状态持久化                   │
│    │                                           │
│  Human-in-the-loop ─── 人工中断和恢复           │
└────────────────────────────────────────────────┘
```

### 3.2 完整代码：用 LangGraph 实现 ReAct Agent

```python
from typing import Annotated, TypedDict
from langgraph.graph import StateGraph, END
from langgraph.graph.message import add_messages
from langgraph.checkpoint.memory import MemorySaver
from langchain_openai import ChatOpenAI
from langchain_core.tools import tool
import json

# ===== Step 1: 定义状态 =====
class AgentState(TypedDict):
    """Agent 的状态定义"""
    messages: Annotated[list, add_messages]  # 对话历史
    iterations: int                           # 当前迭代次数

# ===== Step 2: 定义工具 =====
@tool
def search_web(query: str) -> str:
    """搜索互联网获取最新信息"""
    # 实际实现调用搜索 API
    return f"搜索结果：关于 '{query}' 的最新信息..."

@tool
def calculate(expression: str) -> str:
    """执行数学计算，输入数学表达式"""
    try:
        result = eval(expression)  # 生产环境请用安全的计算库
        return f"计算结果: {result}"
    except Exception as e:
        return f"计算错误: {e}"

tools = [search_web, calculate]

# ===== Step 3: 创建 LLM =====
llm = ChatOpenAI(model="gpt-4o").bind_tools(tools)

# ===== Step 4: 定义节点 =====
def agent_node(state: AgentState) -> dict:
    """Agent 主节点：调用 LLM 决定下一步"""
    response = llm.invoke(state["messages"])
    return {
        "messages": [response],
        "iterations": state.get("iterations", 0) + 1
    }

def should_continue(state: AgentState) -> str:
    """条件边：决定是否继续循环"""
    last_message = state["messages"][-1]
    iterations = state.get("iterations", 0)

    # 超过最大迭代次数
    if iterations >= 10:
        return "end"

    # LLM 请求调用工具 → 继续执行
    if hasattr(last_message, "tool_calls") and last_message.tool_calls:
        return "tools"

    # LLM 直接回复 → 结束
    return "end"

# ===== Step 5: 构建图 =====
from langgraph.prebuilt import ToolNode

def build_agent():
    graph = StateGraph(AgentState)

    # 添加节点
    graph.add_node("agent", agent_node)
    graph.add_node("tools", ToolNode(tools))

    # 设置入口
    graph.set_entry_point("agent")

    # 添加条件边
    graph.add_conditional_edges(
        "agent",
        should_continue,
        {
            "tools": "tools",  # 需要调用工具 → 去 tools 节点
            "end": END         # 不需要 → 结束
        }
    )

    # 工具执行后回到 agent
    graph.add_edge("tools", "agent")

    return graph.compile(checkpointer=MemorySaver())

# ===== Step 6: 运行 =====
agent = build_agent()

result = agent.invoke(
    {"messages": [("user", "2025年最新的RAG技术有哪些？帮我搜索并总结")],
     "iterations": 0},
    config={"configurable": {"thread_id": "demo-1"}}
)

print(result["messages"][-1].content)
```

### 3.3 状态设计模式

```python
# 方式 1: TypedDict（LangGraph 推荐，最轻量）
class StateV1(TypedDict):
    messages: Annotated[list, add_messages]
    context: str

# 方式 2: Pydantic（类型安全，自动验证）
from pydantic import BaseModel

class StateV2(BaseModel):
    messages: list[dict] = []
    context: str = ""
    score: float = 0.0

# 方式 3: dataclass（Python 原生）
from dataclasses import dataclass, field

@dataclass
class StateV3:
    messages: list = field(default_factory=list)
    context: str = ""

# 选择建议：
# - 简单场景 → TypedDict（性能最好）
# - 需要验证 → Pydantic（类型检查）
# - 需要默认值 → dataclass
```

### 3.4 Human-in-the-loop

```python
from langgraph.types import interrupt, Command

def approval_node(state: AgentState) -> dict:
    """需要人工审批的节点"""
    last_message = state["messages"][-1]

    # 中断执行，等待人工输入
    human_decision = interrupt(
        f"Agent 想要执行以下操作，是否同意？\n\n"
        f"{last_message.content}\n\n"
        f"请输入 'yes' 或 'no'："
    )

    if human_decision == "yes":
        return {"messages": [("system", "人工已批准，继续执行")]}
    else:
        return {"messages": [("system", "人工已拒绝，停止执行")]}

# 恢复执行
result = agent.invoke(
    Command(resume="yes"),  # 人工输入 "yes"
    config={"configurable": {"thread_id": "approval-1"}}
)
```

### 3.5 Send 并行：Map-Reduce 模式

```python
from langgraph.types import Send

def map_node(state: AgentState) -> list[Send]:
    """Map 阶段：并行分发子任务"""
    topics = ["技术分析", "市场分析", "竞争对手分析"]
    return [
        Send("researcher", {"topic": t, "original_query": state["query"]})
        for t in topics
    ]

def researcher_node(state: dict) -> dict:
    """单个研究 Agent"""
    # 每个 researcher 独立执行
    result = llm.invoke(f"对以下主题进行{state['topic']}：{state['original_query']}")
    return {"research_result": result.content}

def reduce_node(state: AgentState) -> dict:
    """Reduce 阶段：汇总所有研究结果"""
    all_results = state.get("research_results", [])
    summary = llm.invoke(f"汇总以下研究结果：{all_results}")
    return {"messages": [summary]}
```

---

## 四、框架深度对比

### 4.1 框架架构概览

| 框架 | 核心抽象 | 状态管理 | 执行模型 | 学习曲线 |
|------|----------|----------|----------|---------|
| **LangChain** | Chain/Agent/Tool | Memory | 线性链 | 中 |
| **LangGraph** | StateGraph/Node/Edge | State + Checkpointer | 有状态图 | 中高 |
| **AutoGen** | ConversableAgent/GroupChat | 对话历史 | 多轮对话 | 中 |
| **CrewAI** | Crew/Agent/Task | Task 状态 | 流程驱动 | 低 |
| **Dify** | 可视化节点 | 工作流状态 | DAG | 低 |
| **Semantic Kernel** | Kernel/Plugin/Plan | 上下文变量 | 计划执行 | 中 |

### 4.2 LangGraph vs LangChain 的关系

```
LangChain 生态
├── langchain-core        ← 基础抽象（LLM、Tool、Callback）
├── langchain             ← Chain、Agent、Memory
├── langgraph             ← 有状态多步 Agent（独立但兼容）
├── langsmith             ← 可观测性平台
└── langserve             ← API 部署

关系：LangGraph 不是 LangChain 的替代品，而是互补
- LangChain：线性/简单 Chain 的快速构建
- LangGraph：复杂的有状态 Agent 和多 Agent 系统
- 两者共享 langchain-core 的抽象（Tool、LLM 等）
```

### 4.3 AutoGen 的设计

```python
# AutoGen 核心：ConversableAgent + GroupChat
import autogen

# 创建 Agent
assistant = autogen.AssistantAgent(
    name="assistant",
    llm_config={"model": "gpt-4o"}
)

user_proxy = autogen.UserProxyAgent(
    name="user",
    human_input_mode="TERMINATE",  # 只在终止时需要人工输入
    code_execution_config={"use_docker": True}
)

# GroupChat：多 Agent 讨论模式
group_chat = autogen.GroupChat(
    agents=[assistant, user_proxy, critic_agent],
    messages=[],
    max_round=10
)

manager = autogen.GroupChatManager(
    groupchat=group_chat,
    llm_config={"model": "gpt-4o"}
)

# 启动对话
user_proxy.initiate_chat(
    manager,
    message="请讨论并实现一个快速排序算法"
)
```

### 4.4 选择决策树

```
你的需求是什么？
│
├─ 快速原型 / 简单 Chain
│   └─ LangChain 或 Dify
│
├─ 复杂有状态 Agent
│   ├─ 需要人工介入？
│   │   └─ LangGraph（Human-in-the-loop）
│   ├─ 需要持久化状态？
│   │   └─ LangGraph（Checkpointer）
│   └─ 需要多 Agent 协作？
│       └─ LangGraph 或 AutoGen
│
├─ 多 Agent 讨论 / 对话
│   └─ AutoGen（GroupChat）
│
├─ 团队协作（非技术人员参与）
│   └─ CrewAI 或 Dify
│
├─ .NET / 企业集成
│   └─ Semantic Kernel
│
└─ 不确定 / 需要灵活切换
    └─ LangGraph（最灵活）
```

> [!hint]- 面试题：面试官问"你们项目为什么选 LangGraph 而不是 LangChain"？你如何回答？

> [!success]- 参考答案
> 从三个维度回答：
>
> 1. **需求匹配**：
>    - 我们的 Agent 需要多步骤、有条件分支的执行流程
>    - 需要 Human-in-the-loop（审核关键操作）
>    - 需要状态持久化（用户可以中断后继续）
>    - LangChain 的线性 Chain 做不到这些
>
> 2. **技术优势**：
>    - LangGraph 基于图的状态机模型，天然支持条件分支和循环
>    - 内置 Checkpointer 做状态持久化
>    - 原生支持 Human-in-the-loop（interrupt/resume）
>    - Send 机制支持并行执行（map-reduce）
>
> 3. **权衡**：
>    - 学习曲线比 LangChain 陡一些
>    - 但对于复杂 Agent 场景，LangChain 最终也会陷入 hack，不如直接用图模型
>    - 两者共享 langchain-core，迁移成本低

---

## 五、多智能体系统的工程挑战

### 5.1 Token 成本控制

多 Agent 系统的 Token 消耗是单 Agent 的 3-10 倍。优化策略：

```
┌──────────────────────────────────────────────────────┐
│  Token 成本优化策略金字塔                              │
│                                                      │
│                    ┌──────┐                          │
│                    │ 架构 │ ← 减少不必要的 Agent      │
│                   ─┤优化  │   调用                    │
│                 /  └──────┘                          │
│               /                                       │
│            ┌─┴──────┐                                │
│            │ 模型   │ ← 简单任务用小模型              │
│            │ 分层   │   复杂任务用大模型               │
│           ─┤       │                                 │
│         /  └───────┘                                 │
│       /                                               │
│    ┌─┴────────┐                                      │
│    │ Prompt   │ ← 精简系统提示                       │
│    │ 压缩     │   压缩中间结果                       │
│   ─┤         │                                       │
│  /  └─────────┘                                      │
│/                                                     │
│┌────────────────────────────────┐                    │
│  缓存（相同子问题不重复调用 LLM）│                    │
└────────────────────────────────┘                    │
└──────────────────────────────────────────────────────┘
```

```python
class TieredModelRouter:
    """模型分层路由器"""

    TIERS = {
        "simple": {"model": "gpt-4o-mini", "cost_per_1k": 0.00015},
        "medium": {"model": "gpt-4o", "cost_per_1k": 0.0025},
        "complex": {"model": "gpt-4o", "cost_per_1k": 0.0025},
    }

    def route(self, task_type: str, complexity: str) -> str:
        if task_type in ["format", "extract", "classify"]:
            return self.TIERS["simple"]["model"]
        elif task_type in ["summarize", "translate", "write"]:
            return self.TIERS["medium"]["model"]
        else:  # reason, plan, analyze
            return self.TIERS["complex"]["model"]

    def estimate_cost(self, input_tokens: int, output_tokens: int, tier: str) -> float:
        rate = self.TIERS[tier]["cost_per_1k"]
        return (input_tokens + output_tokens) * rate / 1000
```

### 5.2 延迟优化

| 策略 | 效果 | 实现复杂度 |
|------|------|-----------|
| 并行调用独立 Agent | 延迟降低 50-70% | 低 |
| 流式输出（Streaming） | 用户感知延迟降低 | 低 |
| 结果缓存 | 重复查询零延迟 | 中 |
| 预计算/预加载 | 提前准备数据 | 中 |
| 模型量化/蒸馏 | 推理速度提升 2-3x | 高 |

### 5.3 调试与可观测性

```python
import logging
from datetime import datetime

class AgentTracer:
    """Agent 调用链追踪器"""

    def __init__(self):
        self.traces: list[dict] = []
        self.logger = logging.getLogger("agent_trace")

    def trace_call(self, agent_id: str, input_data: str,
                   output_data: str, latency_ms: float,
                   token_usage: dict = None):
        trace = {
            "timestamp": datetime.now().isoformat(),
            "agent_id": agent_id,
            "input_preview": input_data[:200],
            "output_preview": output_data[:200],
            "latency_ms": latency_ms,
            "token_usage": token_usage,
        }
        self.traces.append(trace)
        self.logger.info(
            f"[{agent_id}] latency={latency_ms:.0f}ms "
            f"tokens={token_usage}"
        )

    def get_trace_chain(self) -> str:
        """生成可读的调用链"""
        lines = ["=== Agent 调用链 ==="]
        for i, t in enumerate(self.traces):
            lines.append(
                f"{i+1}. [{t['agent_id']}] "
                f"→ {t['latency_ms']:.0f}ms "
                f"→ tokens: {t.get('token_usage', {})}"
            )
        return "\n".join(lines)
```

### 5.4 面试场景：设计可扩展的多 Agent 系统

> [!hint]- 面试题：设计一个可处理 1000+ 并发用户的多 Agent 客服系统，要求：低延迟（<3s）、低成本、可扩展。

> [!success]- 参考答案
> **架构设计**：
>
> 1. **入口层**：API Gateway + 负载均衡
>    - WebSocket 长连接，支持流式输出
>    - 请求队列，削峰填谷
>
> 2. **路由层**：轻量分类模型（非 LLM）
>    - 用 BERT 级别的小模型做意图分类
>    - 90% 的简单咨询直接走规则引擎，不调 LLM
>    - 只有 10% 的复杂问题才进入 Agent 系统
>
> 3. **Agent 层**：分层设计
>    - L1（轻量）：FAQ Agent，用小模型 + 向量检索
>    - L2（中等）：业务 Agent，用中等模型 + Function Calling
>    - L3（重度）：专家 Agent，用强模型 + 多工具
>
> 4. **优化策略**：
>    - 结果缓存：相同/相似问题的答案缓存 5 分钟
>    - 模型分层：L1 用 gpt-4o-mini，L2 用 gpt-4o，L3 用 gpt-4o
>    - 并行处理：用户历史 + 上下文检索并行进行
>    - 预加载：用户进入页面时就预加载 FAQ 和上下文
>
> 5. **成本估算**（1000 并发用户）：
>    - 90% 命中缓存/规则：几乎零成本
>    - 10% 需要 LLM：约 100 次/分钟 × $0.01/次 = $1/分钟
>    - 月成本约 $43K，可通过缓存优化进一步降低

---

## 六、前沿研究方向

### 6.1 Agent 之间的信任机制

多 Agent 系统中，一个 Agent 的输出是另一个 Agent 的输入。如何建立信任？

- **验证 Agent**：专门的 Agent 负责验证其他 Agent 的输出
- **置信度评分**：每个 Agent 输出附带置信度，低置信度的结果需要额外验证
- **交叉验证**：关键决策由多个独立 Agent 同时做出，取多数一致的结果
- **声誉系统**：记录每个 Agent 的历史表现，低质量 Agent 被降权

### 6.2 动态拓扑

传统多 Agent 系统的拓扑是固定的。前沿研究探索根据任务动态组织 Agent：

- **能力注册表**：每个 Agent 声明自己的能力，Router 根据任务需求动态选择
- **自适应分组**：根据任务复杂度自动调整参与的 Agent 数量
- **自组织网络**：Agent 之间根据效果自动建立/断开连接

### 6.3 涌现行为

当多个 Agent 协作时，可能出现单个 Agent 不会表现出的行为：

- **群体智慧**：多个一般能力的 Agent 协作可能超越单个强能力 Agent
- **群体偏差**：Agent 之间可能互相强化错误（echo chamber）
- **不可预测的策略**：Agent 可能找到人类没有预想到的解决方案路径

> [!hint]- 思考题：在多 Agent 系统中，如何防止 Agent 之间的"回音室效应"——即错误信息在 Agent 间被反复强化？

> [!success]- 参考答案
> 几个策略：
>
> 1. **引入"魔鬼代言人" Agent**：专门负责质疑和反驳主流观点
> 2. **独立验证**：关键结论由独立 Agent 验证，而不是让 Agent 互相引用
> 3. **置信度衰减**：Agent 引用其他 Agent 的结论时，降低该结论的置信度
> 4. **信息溯源**：标注每个信息的原始来源，区分"一手信息"和"二手引用"
> 5. **定期"重启"**：不累积 Agent 间的对话历史，每轮都从原始问题开始

### 6.4 多 Agent 系统的安全性

- **Agent 身份伪造**：恶意 Agent 伪装成可信 Agent 发送错误信息
- **资源耗尽攻击**：恶意 Agent 触发大量无意义的工具调用，消耗资源
- **信息泄露**：Agent 间的通信可能泄露用户隐私
- **级联失败**：一个 Agent 的错误导致整条链路崩溃

---

## 关键要点回顾

1. **四种协作模式**：主从（最实用）、辩论（多角度）、流水线（数据处理）、层级（大规模任务）
2. **通信设计**：统一消息格式、状态同步策略、频率控制——通信设计是多 Agent 系统成败的关键
3. **LangGraph 是当前最灵活的框架**：基于图的状态机、支持 Human-in-the-loop、状态持久化、并行执行
4. **框架选型看需求**：简单用 LangChain/Dify，复杂用 LangGraph，讨论用 AutoGen，团队用 CrewAI
5. **工程挑战核心是成本**：Token 消耗、延迟、一致性、可观测性——需要分层设计和缓存策略
6. **安全和信任是前沿方向**：Agent 间的信任验证、防止回音室、动态拓扑自组织

## 扩展阅读

- LangGraph 官方文档（2024-2025 版）
- AutoGen: "Enabling Next-Gen LLM Applications via Multi-Agent Conversation"
- CrewAI: "Framework for Orchestrating Role-Playing Autonomous AI Agents"
- "The Rise and Potential of Large Language Model Based Agents" (arXiv 2023)
- "Generative Agents: Interactive Simulacra of Human Behavior" (Stanford 2023)

## 下一步学习

- [[raw/lessons/AI-Interview-Prep/02b-function-calling|Function Calling 与工具调用深入]] — 多 Agent 系统的底层基础设施
