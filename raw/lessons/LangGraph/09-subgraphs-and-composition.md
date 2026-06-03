---
type: lesson
tags:
  - LangGraph
  - Subgraphs
  - Composition
  - Python
  - Agent
created: 2026-04-28
updated: 2026-04-28
topic: 子图与组合——把研究助手拆成可复用模块
difficulty: advanced
prerequisites:
  - "[[raw/lessons/LangGraph/08-send-and-parallel|第 8 章：并行与分发——让研究助手同时处理多个检索任务]]"
sources:
  - https://docs.langchain.com/oss/python/langgraph/use-subgraphs
  - https://docs.langchain.com/oss/python/langgraph/add-memory
status: completed
series:
  name: LangGraph 系统学习
  part: 9
---

# 子图与组合——把研究助手拆成可复用模块

> 一句话摘要：当 Research Copilot 发展到并行检索、多步汇总以后，继续把所有节点堆在一个大图里会越来越难维护。本章会把它升级为模块化的 v7：你将掌握子图、父图组合、共享状态与私有状态的基本模式，并把研究助手拆成更可复用的结构。

## 学习目标

完成本课后，你应该能够：
- [ ] 理解为什么图越来越大后，需要用子图拆分职责
- [ ] 会把编译好的子图作为节点接入父图
- [ ] 理解共享状态与私有状态在子图中的区别
- [ ] 能把 Research Copilot 拆成搜索、整理、汇总等模块化子系统
- [ ] 知道模块化组织和单文件堆砌在可维护性上的差异
- [ ] 会调试子图流式输出和父子状态衔接问题

## 前置知识

| 知识点 | 要求 | Wiki 参考 |
|--------|------|-----------|
| 并行图 | 理解扇出和汇总 | [[raw/lessons/LangGraph/08-send-and-parallel|第 8 章：并行与分发——让研究助手同时处理多个检索任务]] |
| 流式观察 | 知道 `subgraphs=True` 会更有价值 | [[raw/lessons/LangGraph/07-streaming|第 7 章：流式执行——实时看见研究助手正在如何思考与推进]] |
| Graph API | 能读懂多节点图 | [[raw/lessons/LangGraph/02-stategraph-basics|第 2 章：StateGraph、节点与状态——让研究助手先跑起来]] |
| Python 模块组织 | 知道不同函数可以拆到不同文件 | — |

---

## 直观理解

### 类比一：子图像“研究助理小组”

到第 8 章为止，你的 Research Copilot 已经会：

- 对话
- 调工具
- 保持线程
- 支持审核
- 流式观察
- 并行检索

如果你继续把所有东西堆在一张总图里，最后会很像：

- 一个巨大的会议室里所有人同时说话
- 职责边界越来越模糊
- 改一处逻辑时很容易牵一发而动全身

子图的思路就是：

> 把一个大团队拆成多个职责清晰的小组。

例如：

- 检索子图
- 汇总子图
- 审核子图

### 类比二：父图像总导演，子图像专业分组

父图负责：

- 总体流程编排
- 决定什么时候进入哪个模块

子图负责：

- 在自己的局部范围内完成一套更细致的流程

这和软件工程里的“模块化”“子系统”“组件复用”其实是同一个思想。

---

## 一、为什么第 9 章必须讲模块化，而不是继续往前堆功能？

因为从现在开始，问题已经不是“还能不能多加一个节点”，而是：

> **系统还能不能被人类稳定理解、修改和扩展。**

如果你把所有东西继续放在一个 `agent.py` 里，常见后果会是：

- 状态字段越来越多
- 节点函数越来越杂
- 工具逻辑、路由逻辑、汇总逻辑混在一起
- 后面第 10 章想整理成应用结构时非常痛苦

所以第 9 章的核心不是新奇 API，而是：

> **让增长中的图系统重新获得结构。**

---

## 二、Python 补课：模块化、文件组织与“边界”意识

> [!info] Python 补课：为什么这章会自然引出模块化？
> 因为图一旦变大，你就必须开始认真区分：
> - 状态定义放哪里
> - 工具放哪里
> - 节点函数放哪里
> - 哪些节点属于哪个子系统

### 2.1 从“一个文件能跑”到“多个文件可维护”

前几章很多代码都可以写在一个文件里，这是合理的。

但从本章开始，你要开始培养一个更工程化的直觉：

- 能跑 ≠ 容易长期维护
- 模块边界清晰，才容易继续进化

### 2.2 本章的 Python 重点不是导入语法，而是结构设计

你要学会问：

- 哪几个节点属于同一件事？
- 这些节点能不能被抽成一个子图？
- 这个子图和父图共享哪些字段？
- 哪些字段只应该留在子图内部？

---

## 三、最小可运行示例：把编译好的子图当成父图节点

> [!example]- 共享状态的最小子图示例（点击展开）
>
> ```python
> from typing_extensions import TypedDict
> from langgraph.graph import StateGraph, START
>
>
> class State(TypedDict):
>     foo: str
>
>
> def subgraph_node_1(state: State):
>     return {"foo": "hi! " + state["foo"]}
>
>
> subgraph_builder = StateGraph(State)
> subgraph_builder.add_node("subgraph_node_1", subgraph_node_1)
> subgraph_builder.add_edge(START, "subgraph_node_1")
> subgraph = subgraph_builder.compile()
>
>
> builder = StateGraph(State)
> builder.add_node("node_1", subgraph)
> builder.add_edge(START, "node_1")
> graph = builder.compile()
> ```

### 3.1 这里最关键的一句是什么？

```python
builder.add_node("node_1", subgraph)
```

它表示：

- 父图里的一个节点，不一定非得是普通函数
- 它也可以是一个已经编译好的子图

这就是组合的核心。

### 3.2 为什么这很重要？

因为它让你可以：

- 先独立设计和测试一个局部流程
- 再把它作为模块嵌入更大的系统

这比从一开始就在总图里塞几十个节点清晰得多。

---

## 四、共享状态 vs 私有状态：子图最容易搞混的地方

### 4.1 共享状态是什么意思？

如果父图和子图都有同名字段，例如：

```python
foo: str
```

那么子图就可以直接读取和更新这部分共享状态。

### 4.2 私有状态是什么意思？

有些字段只对子图内部有意义，例如：

- 某个中间 `bar`
- 子图内部临时摘要
- 子图自己的局部标记

这类字段不一定需要暴露给父图。

### 4.3 一个更完整的官方风格示例心智

- 共享字段：父图和子图都理解
- 私有字段：只在子图内部使用

你可以把它想成：

- 共享字段 = 小组向总导演汇报时会使用的标准表单字段
- 私有字段 = 小组内部自己的草稿纸

---

## 五、Research Copilot v7：拆成可复用的几个模块

### 5.1 本章项目升级目标

Research Copilot v7 的重点不是加新业务能力，而是：

- 把已有能力拆成结构清晰的模块
- 让“搜索、整理、汇总、审核”不再纠缠在一个大文件里

### 5.2 一个合理的模块化拆分思路

可以先粗拆成：

1. **检索子图**
   - 拆问题
   - 查资料
   - 收集结果

2. **汇总子图**
   - 读取检索结果
   - 组织结构化总结

3. **审核子图**
   - 暴露草稿
   - 等待人工确认
   - 返回最终版本

父图负责串联这几个子图。

---

## 六、Research Copilot v7：一个更贴近课程主线的子图组合示例

> [!example]- 父图 + 检索子图（点击展开）
>
> ```python
> from typing_extensions import TypedDict
> from langgraph.graph import StateGraph, START, END
>
>
> class RetrievalState(TypedDict):
>     topic: str
>     notes: str
>
>
> def collect_definition(state: RetrievalState):
>     return {"notes": f"先解释 {state['topic']} 的核心定义。"}
>
>
> def collect_papers(state: RetrievalState):
>     return {"notes": state["notes"] + " 再找代表论文。"}
>
>
> retrieval_builder = StateGraph(RetrievalState)
> retrieval_builder.add_node("collect_definition", collect_definition)
> retrieval_builder.add_node("collect_papers", collect_papers)
> retrieval_builder.add_edge(START, "collect_definition")
> retrieval_builder.add_edge("collect_definition", "collect_papers")
> retrieval_subgraph = retrieval_builder.compile()
>
>
> class ParentState(TypedDict):
>     topic: str
>     notes: str
>     final_answer: str
>
>
> def synthesize(state: ParentState):
>     return {"final_answer": f"Research Copilot 总结：{state['notes']}"}
>
>
> parent_builder = StateGraph(ParentState)
> parent_builder.add_node("retrieval", retrieval_subgraph)
> parent_builder.add_node("synthesize", synthesize)
> parent_builder.add_edge(START, "retrieval")
> parent_builder.add_edge("retrieval", "synthesize")
> parent_builder.add_edge("synthesize", END)
> parent_graph = parent_builder.compile()
> ```

### 6.1 这个例子最重要的教学价值是什么？

不是它有多复杂，而是它让你第一次真正看到：

- “检索”可以是一个完整局部系统
- 父图不用关心检索内部每一步细节
- 父图只关心子图对共享状态交付了什么

这就是模块化思维的开始。

---

## 七、如果父图和子图状态不一样怎么办？

这也是非常真实的问题。

### 7.1 两种常见方式

#### 方式 A：共享一部分状态键

最简单，也最适合初学者。

例如父图和子图都用：

- `topic`
- `notes`

这样子图能直接读写共享字段。

#### 方式 B：进入子图前后做状态转换

如果父图和子图字段完全不同，你可以在某个普通节点中：

1. 手动把父图状态转换成子图输入
2. 调用子图 `invoke()`
3. 再把子图输出转换回父图状态

这种方式更灵活，但也更复杂。

> [!tip] 本系列建议
> 第一次学子图时，优先采用“共享关键状态字段”的方式，心智负担更小。

---

## 八、为什么子图和 streaming 是绝配？

子图一多，如果你只看最终结果，会很难知道：

- 当前到底在父图还是子图里
- 哪个子图先更新了状态
- 子图内部哪一步出了问题

官方流式接口支持：

```python
subgraphs=True
```

这意味着你可以在流式输出中看到：

- 父图级更新
- 子图级更新
- 命名空间 `ns` 帮你区分这些更新来自哪里

这对调试子图特别重要。

---

## 九、状态 / 流程 walkthrough：父图 + 子图是怎么协作的？

### 9.1 父图视角

父图只知道：

```text
START -> retrieval_subgraph -> synthesize -> END
```

### 9.2 子图视角

检索子图内部可能是：

```text
START -> collect_definition -> collect_papers -> END
```

### 9.3 两层图叠起来的真实心智模型

```text
父图：
START
  │
  ▼
retrieval_subgraph
  ├── collect_definition
  └── collect_papers
  │
  ▼
synthesize
  │
  ▼
END
```

这就是为什么“子图”不是“单个超级节点”，而是：

> **一个被嵌入到更大流程中的局部流程系统。**

---

## 十、常见报错与调试

### 10.1 父图和子图状态键对不上

如果父图期待：

- `notes`

子图却返回的是：

- `draft_notes`

那父图后续节点就会读不到预期字段。

所以设计子图时一定要先想清楚：

- 共享哪些字段
- 对外承诺交付哪些字段

### 10.2 子图职责定义不清

很多人第一次做模块化时，会把子图切得很奇怪：

- 一半检索、一半审核、一半汇总

这会让边界更糊，而不是更清楚。

更好的原则是：

- 每个子图围绕一个清晰职责组织

### 10.3 子图太小，不值得拆

不是每个两行逻辑都要单独变子图。

如果拆分不能带来：

- 复用性
- 清晰边界
- 独立理解

那可能只是人为增加复杂度。

### 10.4 调试建议：先单独跑子图，再接父图

这几乎是本章最实用的建议：

1. 子图先独立验证
2. 再把它编译后接入父图
3. 接入后用 `stream(..., subgraphs=True, version="v2")` 观察整合过程

---

## 十一、动手练习

### 练习 1：把审核流程也拆成子图

要求：

- 不要把 `human_review` 直接放父图里
- 单独做一个审核子图
- 父图只负责在合适位置调用它

### 练习 2：为检索子图增加一个私有字段

要求：

- 子图里增加一个只对子图内部有意义的字段
- 父图不直接依赖它
- 观察共享字段和私有字段的区别

### 练习 3：流式观察父图 + 子图

要求：

- 运行父图
- 开启 `subgraphs=True`
- 记录你看到的父子图更新层次

---

## 常见误区

| 误区 | 正确理解 |
|------|----------|
| “子图只是把代码拆成多个文件。” | 不够。子图是流程级模块化，不只是文件物理拆分。 |
| “父图应该知道子图内部每一步。” | 不一定。好的模块化会让父图更多关注子图对外提供的结果，而不是内部细节。 |
| “所有节点都值得拆成子图。” | 不对。只有当一组节点形成清晰职责模块时，拆成子图才有价值。 |
| “子图状态最好和父图完全独立。” | 不一定。初学时共享关键状态字段通常更简单；复杂场景再考虑状态转换。 |
| “子图接进父图后就不必单独测试了。” | 仍然建议先单独验证子图，再做组合。 |

---

## 思考题

### 基础理解

**Q1. 为什么图越来越大后，需要引入子图而不是继续往主图里堆节点？**

> [!hint]- 💡 提示
> 想想可维护性、边界和后续扩展。

> [!success]- ✅ 参考答案
> 因为随着节点、状态字段和路由逻辑不断增加，把所有内容继续堆在一张主图里会让职责边界越来越模糊，后续修改和扩展都更困难。子图能把局部流程封装成更清晰的模块，让系统重新获得结构。

**Q2. 把编译好的子图当成父图节点，这件事为什么重要？**

> [!hint]- 💡 提示
> 它是不是意味着“局部流程也可以像组件一样复用”？

> [!success]- ✅ 参考答案
> 因为这意味着一个完整的小流程也可以像普通节点一样被嵌入到更大的图中。你可以先独立设计、测试和复用局部图，再把它接入父图，从而让系统具备真正的组合能力。

### 深入思考

**Q3. 为什么说模块化设计的关键不在“拆文件”，而在“划边界”？**

> [!hint]- 💡 提示
> 如果边界不清楚，拆成多少文件也可能一样乱。

> [!success]- ✅ 参考答案
> 因为真正决定模块化质量的不是代码放在哪个文件，而是职责是否清晰、状态接口是否明确、父图与子图是否只通过必要字段交互。如果边界没划清楚，只是物理拆分文件，复杂度并不会真正降低。

**Q4. 为什么子图特别适合配合 streaming 调试？**

> [!hint]- 💡 提示
> 父图和子图更新混在一起时，只看最终结果够吗？

> [!success]- ✅ 参考答案
> 因为子图引入了流程层级结构：更新可能来自父图，也可能来自某个子图内部节点。如果只看最终结果，很难知道问题出在哪一层。配合 `subgraphs=True` 的流式输出，可以更清楚地看到父图和子图各自的推进过程，从而更高效地调试组合式系统。

---

## 关键要点回顾

- ✅ 子图让大图重新获得模块边界，而不是继续无限堆节点
- ✅ 编译好的子图可以作为父图中的一个节点接入
- ✅ 共享状态字段是初学者最容易掌握的父子图协作方式
- ✅ 私有状态适合留在子图内部，不必全暴露给父图
- ✅ Research Copilot v7 的升级重点是“可复用、可维护、可组合”的结构能力

---

## 扩展阅读

- 📄 [Use Subgraphs](https://docs.langchain.com/oss/python/langgraph/use-subgraphs) — 本章最核心的官方参考
- 📄 [Add Memory](https://docs.langchain.com/oss/python/langgraph/add-memory) — 继续理解父图与子图的 checkpoint 关系

## 相关页面

- [[raw/lessons/LangGraph/08-send-and-parallel|第 8 章：并行与分发——让研究助手同时处理多个检索任务]]
- [[raw/lessons/LangGraph/10-deployment-and-beyond|第 10 章：部署与扩展——把研究助手整理成本地可运行应用]]
- [[wiki/concepts/langgraph-api-map|LangGraph API 学习地图]]

## 下一步学习

下一章我们会进入收官阶段：

- `langgraph dev` 在本地开发里扮演什么角色
- 项目目录如何整理成应用结构
- 为什么还要补一个函数式 API 视角

继续阅读：[[raw/lessons/LangGraph/10-deployment-and-beyond|第 10 章：部署与扩展——把研究助手整理成本地可运行应用]]
