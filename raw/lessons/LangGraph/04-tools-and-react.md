---
type: lesson
tags:
  - LangGraph
  - ToolNode
  - ReAct
  - Python
  - Agent
created: 2026-04-28
updated: 2026-04-28
topic: 工具与 ReAct——让研究助手学会调用检索工具
difficulty: intermediate
prerequisites:
  - "[[raw/lessons/LangGraph/03-state-and-messages|第 3 章：State 与 Messages——让研究助手真正进入对话态]]"
sources:
  - https://docs.langchain.com/oss/python/langgraph/quickstart
  - https://docs.langchain.com/oss/python/langgraph/workflows-agents
  - https://docs.langchain.com/oss/python/langgraph/graph-api
  - https://api-docs.deepseek.com/zh-cn/
  - https://api-docs.deepseek.com/zh-cn/guides/function_calling/
status: completed
series:
  name: LangGraph 系统学习
  part: 4
---

# 工具与 ReAct——让研究助手学会调用检索工具

> 一句话摘要：前两章我们已经让 Research Copilot 拥有了图结构和消息状态，但它仍然只是在“空想”。本章会让它真正学会调用工具：你将掌握 `@tool`、`ToolNode`、`tools_condition`，并理解 `create_react_agent` 与底层图写法之间的关系。

## 学习目标

完成本课后，你应该能够：
- [ ] 理解为什么 agent 只有消息而没有工具时，能力依然很有限
- [ ] 会用 `@tool` 定义一个最小检索工具
- [ ] 能用 `ToolNode` + `tools_condition` 搭建最小 ReAct 风格图
- [ ] 知道 `create_react_agent` 是高层 helper，而不是替代图思维的黑盒神谕
- [ ] 能把 Research Copilot 升级为能查资料的 v2
- [ ] 遇到工具调用问题时，知道先检查工具签名、docstring、消息回路和条件边

## 前置知识

| 知识点 | 要求 | Wiki 参考 |
|--------|------|-----------|
| `MessagesState` | 理解消息列表与 reducer | [[raw/lessons/LangGraph/03-state-and-messages|第 3 章：State 与 Messages——让研究助手真正进入对话态]] |
| `StateGraph` | 能读懂最小图结构 | [[raw/lessons/LangGraph/02-stategraph-basics|第 2 章：StateGraph、节点与状态——让研究助手先跑起来]] |
| Python 函数 | 知道参数和返回值 | — |
| 基本 agent 直觉 | 知道模型可能决定“要不要调工具” | — |

---

## 直观理解

### 类比一：工具是“研究助理的外部能力”

如果一个研究助理只能看着聊天记录回答问题，但不能：

- 查资料
- 读知识库
- 调接口
- 调用搜索函数

那它再会聊天，也只是“会说话但不会查证”的助手。

工具让 agent 从“只会组织语言”升级为“能接触外部世界”。

### 类比二：ReAct 像“先想，再做，再回看结果”

ReAct 的核心节奏可以非常直白地理解为：

1. 先想：要不要用工具？
2. 如果要，就做：调用工具
3. 看结果：把工具结果带回上下文
4. 再想：现在能回答了吗？

LangGraph 的优势在于，它不是把这套过程藏在黑盒里，而是可以把它明确画成图。

---

## 一、为什么消息状态之后，下一步一定是工具？

第 3 章的 Research Copilot v1 已经可以多轮对话，但它仍然有一个根本局限：

> 它只能基于已有上下文生成回答，不能主动获取新信息。

这意味着它做不了真正的研究助手，因为研究任务天然需要：

- 检索资料
- 调知识库
- 看结构化数据
- 读取已有笔记

所以从课程递进上看，工具调用几乎是消息状态之后最自然的一步。

### 1.2 为什么说 Tool Calling 是 agent 的分水岭？

很多人第一次接触 agent 时，会误以为：

> “只要模型更聪明、prompt 更长，它就会自然变成 agent。”

这其实只说对了一半。

真正的分水岭不在于模型会不会“多想几步”，而在于它是否具备：

1. **调用外部能力的接口**
2. **把推理结果转成可执行动作的机制**
3. **在动作结果回来后继续下一轮判断的回路**

也就是说，Tool Calling 把系统从“只能说”推进到“既能判断，又能行动”。

| 维度 | 只有对话能力的系统 | 具备 Tool Calling 的系统 |
| --- | --- | --- |
| 信息来源 | 主要依赖模型参数和当前上下文 | 可以额外读取数据库、搜索结果、文件、API 响应 |
| 行为类型 | 生成文本 | 生成文本 + 触发动作 |
| 任务边界 | 更像解释器、总结器 | 更像可执行工作流协调器 |
| 错误修正 | 只能在文本层面自我修饰 | 可以根据工具结果继续修正计划 |
| 多步任务能力 | 容易停在“建议你去做” | 更容易进入“我先替你做一部分” |

从技术角度看，Tool Calling 至少引入了三个关键变化：

#### A. 它打破了模型知识的封闭性

没有工具时，模型只能依赖：
- 训练时见过的知识
- 当前对话里提供的材料

有工具后，模型可以把“我不知道最新数据”变成：
- 去查
- 去读
- 去拿结果
- 再继续回答

这就是 agent 能从“像顾问”迈向“像助手”的关键一步。

#### B. 它把“文本输出”升级成“动作决策”

普通聊天模型输出的终点通常是一段话。

而 Tool Calling 让模型输出可以变成另一种东西：
- 要调用哪个工具
- 工具参数是什么
- 调用之后要不要继续

也就是说，模型不再只是生成一句自然语言，而是在生成**下一步动作指令**。

#### C. 它把单轮问答升级成可回环的 ReAct 结构

ReAct 的核心不是口号里的“Reason + Act”四个字，而是：

```text
想一下 → 决定是否行动 → 拿到外部结果 → 再判断下一步
```

如果没有 Tool Calling，这个回路就很容易退化成：

```text
想一下 → 直接输出一段话 → 结束
```

所以你可以把这一章理解成：

> **从“会聊天的图”走向“会协调动作的图”。**

这就是为什么它是本系列真正的分水岭。

---

## 二、Python 补课：装饰器、函数签名与 docstring 为什么在这章突然重要了？

> [!info] Python 补课：`@tool` 本质上是装饰器
> 装饰器可以理解成：
> - 你先写一个普通函数
> - 再用 `@tool` 给它加上“这是可被 agent 调用工具”的语义包装

### 2.1 最小工具长什么样？

```python
from langchain.tools import tool

@tool
def search_paper_topics(query: str) -> str:
    """根据研究问题返回建议检索方向。"""
    return f"建议优先检索：{query} 的定义、代表性论文、应用场景。"
```

这里至少有 3 个要点：

1. 函数参数要清楚
2. 返回值类型尽量明确
3. docstring 不能乱写，因为模型和工具生态会参考它理解工具用途

### 2.2 为什么工具函数签名要尽量简单？

因为 agent 要决定是否调用工具、如何填参数。

如果你写成极度混乱的工具接口：

- 参数意义模糊
- docstring 不说人话
- 输入输出结构乱

模型即使能力强，也会更容易调用错。

### 2.3 这章的 Python 补课核心不是“装饰器语法”，而是“接口设计”

你在这章真正要学会的是：

> **工具函数不只是能跑，还要让模型容易理解和调用。**

---

## 三、Research Copilot v2：从对话助手升级为“可检索助手”

### 3.1 本章项目目标

本章的 Research Copilot v2 要获得一个新能力：

- 当用户提研究问题时，先考虑是否需要调用检索工具
- 如果需要，就执行工具
- 再基于工具结果组织回答

### 3.2 图结构会怎么变化？

从前一章的简单线性流：

```text
START -> 读消息 -> 写回复 -> END
```

升级成带回路的最小 ReAct 风格图：

```text
START -> llm_call
            │
            ├── 有工具调用 ──► tool_node ──► llm_call
            │
            └── 没有工具调用 ──► END
```

这就是本章最重要的结构升级。

---

## 四、最小可运行示例：`ToolNode` + `tools_condition`

> [!example]- 最小可运行 ReAct 风格图（点击展开）
>
> ```python
> from langchain.tools import tool
> from langchain.chat_models import init_chat_model
> from langchain.messages import SystemMessage, HumanMessage
> from langgraph.graph import StateGraph, START, END, MessagesState
> from langgraph.prebuilt import ToolNode, tools_condition
>
>
> @tool
> def search_paper_topics(query: str) -> str:
>     """根据研究问题返回建议检索方向。"""
>     return f"建议优先检索：{query} 的定义、代表性论文、应用场景。"
>
>
> model = init_chat_model(
>     "deepseek-chat",
>     model_provider="deepseek",
>     temperature=0,
> )
> llm_with_tools = model.bind_tools([search_paper_topics])
>
>
> def llm_call(state: MessagesState):
>     return {
>         "messages": [
>             llm_with_tools.invoke(
>                 [
>                     SystemMessage(content="你是一个研究助手。必要时调用工具帮助用户规划检索方向。")
>                 ]
>                 + state["messages"]
>             )
>         ]
>     }
>
>
> builder = StateGraph(MessagesState)
> builder.add_node("llm_call", llm_call)
> builder.add_node("tool_node", ToolNode([search_paper_topics]))
>
> builder.add_edge(START, "llm_call")
> builder.add_conditional_edges(
>     "llm_call",
>     tools_condition,
>     {
>         "tools": "tool_node",
>         END: END,
>     },
> )
> builder.add_edge("tool_node", "llm_call")
>
> graph = builder.compile()
>
> result = graph.invoke(
>     {
>         "messages": [
>             HumanMessage(content="我想快速了解 LangGraph memory 的研究起点，应该查哪些方向？")
>         ]
>     }
> )
>
> print(result["messages"][-1].content)
> ```

### 4.1 这段图为什么是“最小 ReAct 风格”？

因为它已经具备了 ReAct 最关键的循环：

- LLM 先决定要不要用工具
- 如果要，就进入 `tool_node`
- 工具结果回到消息上下文
- 再回到 LLM 判断是否继续或直接回答

### 4.2 `ToolNode` 到底帮你做了什么？

`ToolNode` 可以理解成：

> **一个标准化的工具执行节点。**

它会根据上一条消息里的工具调用信息，执行对应工具，并把结果以合适形式写回消息流。

这比你手搓“解析 tool_calls -> 执行函数 -> 手动组装消息”的流程更稳。

### 4.3 `tools_condition` 在判断什么？

`tools_condition` 的作用是：

- 看 LLM 刚输出的消息里有没有工具调用
- 如果有，走到工具节点
- 如果没有，流程结束

也就是说，它把“是否进入工具执行分支”这件事标准化了。

> [!info] 本课程为什么把项目默认模型主线定为 DeepSeek
> 根据 DeepSeek 官方文档，官方文档列出的部分模型（如 `deepseek-v4-pro`）支持 Function Calling，也支持流式输出；而 LangChain 当前能查到的集成示例默认使用 `deepseek-chat` 进行 `init_chat_model(..., model_provider="deepseek")` 初始化。
> 这意味着本章讲的 `bind_tools()`、`ToolNode`、`tools_condition` 这条主线，在 DeepSeek 项目实战里继续沿用是成立的。
> 如果你后面要把示例模型名从 `deepseek-chat` 切到 `deepseek-v4-pro` 或团队内的其他 DeepSeek 型号，第一件事仍然是确认该模型是否支持工具调用，而不是只看它“能不能聊天”。

### 4.4 `init_chat_model(...)` 在这段图里扮演什么角色？

**角色**：
它是这张图里的**模型实例构造器**。你可以把它理解成“给这条工作流接上一颗真正会推理、会决定是否调工具的大脑”。

在本章示例里：

```python
model = init_chat_model(
    "deepseek-chat",
    model_provider="deepseek",
    temperature=0,
)
```

这段代码做的事不是“马上去请求模型”，而是：
- 选择模型名字：`deepseek-chat`
- 选择 provider：`deepseek`
- 配置调用参数：例如 `temperature=0`
- 返回一个后续可反复 `invoke()` 的 chat model 对象

**输入**：
- 模型名（字符串）
- provider 名
- 其他推理参数（如 `temperature`、`max_tokens` 等）

**输出**：
- 一个 LangChain 风格的 chat model 实例
- 这个实例后面可以继续 `bind_tools()`、`invoke()`、`stream()`

**为什么这一步重要？**
因为它决定了后面整张图到底连接的是哪类模型能力。你换模型，不只是换名字，还可能改变：
- 是否支持工具调用
- 是否支持流式输出
- 是否支持结构化输出
- 参数名和行为边界是否一致

> [!tip] 对初学者最重要的理解
> `init_chat_model()` 的输出不是“答案”，而是一个**可被图反复调用的模型对象**。

### 4.5 `bind_tools([...])` 到底绑定了什么？

**角色**：
它的职责不是执行工具，而是把“有哪些工具可用、每个工具的名字/参数/用途是什么”告诉模型。

```python
llm_with_tools = model.bind_tools([search_paper_topics])
```

这一步之后，模型才知道：
- 有一个叫 `search_paper_topics` 的工具
- 它接收一个 `query: str`
- 它的用途是“根据研究问题返回建议检索方向”

**输入**：
- 一个或多个工具对象（通常来自 `@tool` 包装后的函数）

**输出**：
- 一个“带工具描述”的新模型对象
- 这个对象在 `invoke()` 时，模型才可能返回 tool calls

**最关键的工程直觉**：
`bind_tools()` 是“注册工具语义”，不是“执行工具函数”。

也就是说：
- `@tool`：把普通函数包装成工具
- `bind_tools()`：把这些工具描述暴露给模型
- `ToolNode`：在图里真正执行工具

三者缺一不可。

**常见误区**：
很多人会以为“我已经写了 `@tool`，模型自然就会调用它”。

其实不会。

如果你没有 `bind_tools()`，模型根本不知道当前会话里有哪些工具可选。

### 4.6 `ToolNode([...])` 的输入输出到底是什么？

**角色**：
它是图里的**标准工具执行器**。

```python
builder.add_node("tool_node", ToolNode([search_paper_topics]))
```

这行代码的意思不是“把函数直接当节点塞进去”，而是：

> 用 LangGraph 提供的标准执行器，把工具调用那一段通用流程封装成节点。

如果你手写这一段，你通常要自己做：
1. 读取上一条 AIMessage
2. 找出里面的 tool call 信息
3. 找到同名工具函数
4. 解析参数
5. 执行函数
6. 把结果重新组装成消息
7. 再把它放回消息状态

`ToolNode` 就是在替你做这整套标准化工作。

**输入**：
- 当前 `MessagesState`
- 更具体地说，它主要关心最后一条 AIMessage 里有没有 tool calls

**输出**：
- 工具执行后的结果消息
- 这些结果会被写回 `messages`，供下一轮 `llm_call` 继续读取

**输入输出数据流可以粗略理解成**：

```text
AIMessage(我要调哪个工具 + 参数)
        ↓
     ToolNode
        ↓
ToolMessage(工具执行结果)
```

### 4.7 `tools_condition` 为什么能决定流程去向？

**角色**：
它是一个**条件路由判断器**，负责告诉图：

- 如果模型刚刚请求了工具 → 去 `tool_node`
- 如果模型已经直接给出答案 → 去 `END`

```python
builder.add_conditional_edges(
    "llm_call",
    tools_condition,
    {
        "tools": "tool_node",
        END: END,
    },
)
```

这段代码里最容易忽略的一点是：

- `llm_call` 负责产生新的 AIMessage
- `tools_condition` 读取这条 AIMessage
- 然后根据其中是否存在 tool calls 来返回路由结果

**输入**：
- 当前状态（重点读取 `messages` 的最后一条）

**输出**：
- 表示路由选择的结果
- 在本章语境里，你可以把它理解成返回“去 tools 分支”或“结束”

**为什么这一步是回路成立的关键？**
因为没有它，图就不会知道：
- 什么时候应该执行工具
- 什么时候已经可以停止

也就是说，`tools_condition` 不是在“执行工具”，而是在做**流程控制判断**。

> [!tip] 一句话总结四个关键对象的分工
> - `init_chat_model()`：创建模型对象
> - `bind_tools()`：把工具能力告诉模型
> - `ToolNode(...)`：真正执行工具
> - `tools_condition`：决定图是去执行工具，还是直接结束

---

## 五、Research Copilot v2：加入研究助手语境的版本

前面的例子已经能说明结构，但我们可以稍微把业务语义写得更贴近研究助手。

> [!example]- 更贴近 Research Copilot 的版本（点击展开）
>
> ```python
> from langchain.tools import tool
> from langchain.chat_models import init_chat_model
> from langchain.messages import SystemMessage, HumanMessage
> from langgraph.graph import StateGraph, START, END, MessagesState
> from langgraph.prebuilt import ToolNode, tools_condition
>
>
> @tool
> def search_concepts(query: str) -> str:
>     """给出某研究问题值得优先检索的概念维度。"""
>     return (
>         f"围绕 {query}，建议优先检索：\n"
>         "1. 核心定义\n"
>         "2. 常见架构\n"
>         "3. 代表性论文\n"
>         "4. 应用场景\n"
>         "5. 与相关概念的区别"
>     )
>
>
> model = init_chat_model(
>     "deepseek-chat",
>     model_provider="deepseek",
>     temperature=0,
> )
> llm_with_tools = model.bind_tools([search_concepts])
>
>
> def llm_call(state: MessagesState):
>     return {
>         "messages": [
>             llm_with_tools.invoke(
>                 [
>                     SystemMessage(
>                         content="你是 Research Copilot。遇到需要规划检索路径的问题时，优先调用检索规划工具。"
>                     )
>                 ]
>                 + state["messages"]
>             )
>         ]
>     }
>
>
> builder = StateGraph(MessagesState)
> builder.add_node("llm_call", llm_call)
> builder.add_node("tool_node", ToolNode([search_concepts]))
> builder.add_edge(START, "llm_call")
> builder.add_conditional_edges(
>     "llm_call",
>     tools_condition,
>     {"tools": "tool_node", END: END},
> )
> builder.add_edge("tool_node", "llm_call")
> graph = builder.compile()
> ```

### 5.1 为什么本章仍然尽量控制工具数量？

因为本章核心不是“工具越多越强”，而是先把这条主线讲透：

- 工具如何定义
- 工具节点如何接入
- 条件路由如何回环

工具一多，初学者很容易把注意力都花在业务细节上，而忽略图结构本身。

---

## 六、`create_react_agent` 应该怎么理解？

### 6.1 它是什么？

`create_react_agent` 可以把它理解成：

> **一个高层 helper，用来快速搭建 ReAct 风格 agent。**

也就是说，它帮你把“模型 + 工具 + 基本回路”封装成了一个更方便的入口。

### 6.2 为什么这章不一上来就只讲它？

因为本系列的目标不是“会调用一个封装好的 helper”，而是：

- 理解回路结构
- 理解工具节点
- 理解条件边
- 知道系统为什么会回到 LLM 再思考一次

只有你先理解了底层图结构，再看 `create_react_agent`，你才知道它究竟帮你省去了什么。

### 6.3 更合适的心智模型

你可以把两者关系记成：

- `StateGraph + ToolNode + tools_condition` = 你亲手搭骨架
- `create_react_agent` = 在常见场景下帮你更快组装骨架

### 6.4 什么时候更适合直接用 `create_react_agent`？

当你：

- 已经理解图思维
- 场景比较标准
- 不需要很多自定义路由
- 想先快速起一个常规 agent

这时直接用 helper 很合理。

但如果你：

- 需要插入中间审核
- 需要复杂状态字段
- 需要多阶段路由
- 需要和子图组合

那你仍然应该更偏向底层 Graph API。

---

## 七、状态 / 流程 walkthrough：这一章图是怎么跑的？

### 7.1 初始状态

```python
{
    "messages": [HumanMessage(content="我想快速了解 LangGraph memory 的研究起点，应该查哪些方向？")]
}
```

### 7.2 第一次进入 `llm_call`

LLM 会基于系统提示词和用户消息做判断：

- 如果它认为需要工具，就会输出带 tool call 的 AI 消息
- 如果它认为直接回答即可，就可能直接结束

### 7.3 `tools_condition` 分支判断

- 有 tool call → 进入 `tool_node`
- 无 tool call → 直接 `END`

### 7.4 `tool_node` 执行工具

工具执行结果会被写回消息上下文。

于是消息列表从：

```text
[HumanMessage]
```

变成近似：

```text
[HumanMessage, AIMessage(带工具调用), ToolMessage(工具结果)]
```

### 7.5 回到 `llm_call`

这时 LLM 再看消息上下文，就已经能看到工具结果，于是更容易给出结构化回答。

### 7.6 用 ASCII 图看回路

```text
用户消息
   │
   ▼
llm_call
   │
   ├── 无工具调用 ─────────────► END
   │
   └── 有工具调用
           │
           ▼
        tool_node
           │
           ▼
        llm_call
           │
           ▼
          END
```

---

## 八、常见报错与调试

### 8.1 工具参数设计太模糊

例如：

```python
@tool
def search(x: str) -> str:
    """搜索"""
```

虽然能跑，但对模型不友好：

- `x` 是什么？
- 搜什么？
- 返回什么？

更好的是写清楚：

- 参数名语义明确
- docstring 说明工具用途

### 8.2 忘了 `bind_tools()`

如果你定义了工具，但没把工具绑定给模型：

```python
llm_with_tools = model.bind_tools([...])
```

那模型就算理论上想调工具，也没有实际工具上下文可用。

### 8.3 条件边没回工具节点

如果你写错条件映射，例如：

- `"tools"` 映射节点名拼错
- `END` 分支没写对

那图就会在关键分支上失效。

### 8.4 工具节点执行完没回 LLM

下面这条边很关键：

```python
builder.add_edge("tool_node", "llm_call")
```

如果没有这条边，系统就没法“看到工具结果后再继续思考”。

### 8.5 调试建议：先看最后一条消息是不是 tool call

当你怀疑 agent 没有进入工具分支时，优先检查：

- LLM 输出的最后一条消息是否包含 `tool_calls`
- 工具节点是否真正执行
- 回来之后消息列表是否多了工具结果

---

## 九、动手练习

### 练习 1：新增第二个工具

新增一个工具：

```python
search_papers(query: str) -> str
```

要求：

- 它返回“代表性论文检索建议”
- 与 `search_concepts` 并存
- 观察模型会如何选择工具

### 练习 2：把工具结果整理进回答结构

要求：

- 不要只返回一段泛泛回答
- 让最终 AI 回复明确分成：
  - 核心概念
  - 检索关键词
  - 建议阅读方向

你会练到：

- 工具不是终点
- 工具结果需要被组织成对用户真正有价值的输出

### 练习 3：尝试改写成 `create_react_agent` 版本并做对比

要求：

- 先保留本章底层图版本
- 再查资料尝试用 `create_react_agent` 起一个类似 agent
- 自己总结两者差异：
  - 哪个更快
  - 哪个更透明
  - 哪个更容易插入自定义逻辑

---

## 常见误区

| 误区 | 正确理解 |
|------|----------|
| “只要有工具，agent 就一定更聪明。” | 不对。工具只是外部能力入口，关键仍是如何设计工具接口和回路结构。 |
| “`ToolNode` 是可有可无的便利玩具。” | 不是。它是非常实用的标准工具执行节点，能减少很多手搓消息处理的混乱。 |
| “`tools_condition` 只是语法糖，不必理解。” | 不建议这样想。它代表的是一个非常核心的问题：是否应该进入工具执行分支。 |
| “学会 `create_react_agent` 就等于学会了 LangGraph agent。” | 不够。它让你更快起步，但不能替代对图结构和状态流的理解。 |
| “工具定义随便写写就行。” | 工具接口设计质量会直接影响模型是否能正确调用。 |

---

## 思考题

### 基础理解

**Q1. 为什么消息状态之后，下一步最自然的能力升级是工具调用？**

> [!hint]- 💡 提示
> 研究助手如果不能查资料，会缺什么？

> [!success]- ✅ 参考答案
> 因为只有消息状态，助手依然只能基于已有上下文组织语言，无法主动获取外部信息。研究任务天然需要检索、查阅和调用外部资源，所以工具调用是从“会对话”升级到“能完成任务”的关键一步。

**Q2. `ToolNode` 的核心作用是什么？**

> [!hint]- 💡 提示
> 想想它是不是在标准化工具执行这一步。

> [!success]- ✅ 参考答案
> `ToolNode` 的核心作用是把工具执行节点标准化。它会根据消息里的工具调用信息执行相应工具，并把结果按合适格式带回消息上下文。这样你不必每次都手动解析 tool calls、执行函数、组装工具结果消息。

### 深入思考

**Q3. 为什么本系列不直接从 `create_react_agent` 开始？**

> [!hint]- 💡 提示
> 快速搭建和真正理解系统结构是不是一回事？

> [!success]- ✅ 参考答案
> 因为 `create_react_agent` 虽然方便，但容易把工具回路、条件分支和状态演进都藏起来。系统学习更重要的是先理解底层图结构：模型什么时候决定用工具、工具结果如何回到消息、为什么要再回到 LLM 思考。理解这些后，再用 helper 才不会黑盒化。

**Q4. 为什么说工具函数的接口设计本身也是 agent 设计的一部分？**

> [!hint]- 💡 提示
> 参数名、docstring、返回内容会影响谁？

> [!success]- ✅ 参考答案
> 因为模型需要根据工具接口来决定是否调用、如何填参数、如何理解返回值。接口越清晰，模型越容易正确使用工具；接口越模糊，模型越容易误调或根本不用。所以工具定义不只是 Python 编码问题，也是 agent 可用性设计问题。

---

## 关键要点回顾

- ✅ 工具让 Research Copilot 从“只会说”升级为“能查、能做”
- ✅ `@tool`、函数签名和 docstring 决定了工具是否容易被模型正确使用
- ✅ `ToolNode` 是标准工具执行节点，能显著降低手搓工具回路的复杂度
- ✅ `tools_condition` 负责判断图是否要进入工具分支
- ✅ `create_react_agent` 很有用，但应建立在你已经理解底层图思维的基础上
- ✅ Research Copilot v2 已经具备最小 ReAct 风格的能力骨架

---

## 扩展阅读

- 📄 [LangGraph Quickstart](https://docs.langchain.com/oss/python/langgraph/quickstart) — 对照 Graph API 版本的最小 agent 示例
- 📄 [Workflows + Agents](https://docs.langchain.com/oss/python/langgraph/workflows-agents) — 查看工具回路的更多图式写法
- 📄 [Graph API](https://docs.langchain.com/oss/python/langgraph/graph-api) — 回查消息状态与条件边设计

## 相关页面

- [[raw/lessons/LangGraph/03-state-and-messages|第 3 章：State 与 Messages——让研究助手真正进入对话态]]
- [[raw/lessons/LangGraph/05-checkpointer-memory|第 5 章：记忆与检查点——让研究助手跨轮记住上下文]]
- [[wiki/concepts/langgraph-api-map|LangGraph API 学习地图]]

## 下一步学习

下一章我们会解决一个关键问题：

- 为什么“消息历史”还不等于“真正的跨轮记忆”
- `MemorySaver` 和 `SqliteSaver` 在做什么
- `thread_id` 为什么是后续对话系统的重要配置

继续阅读：[[raw/lessons/LangGraph/05-checkpointer-memory|第 5 章：记忆与检查点——让研究助手跨轮记住上下文]]
