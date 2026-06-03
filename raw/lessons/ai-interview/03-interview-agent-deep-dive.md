---
type: lesson
tags: [模拟面试, Agent设计, System Prompt, TodoList, 中间件]
created: 2026-05-25
updated: 2026-05-25
difficulty: advanced
prerequisites: [LangGraph Agent, Prompt Engineering, 面试流程理解]
topic: 开源项目深度拆解
status: completed
series: { name: "ai-interview-伯乐深度拆解", part: 3 }
---

# 面试 Agent 深度剖析

> InterviewAgent 是伯乐的核心 Agent，展示了一个生产级 LangGraph Agent 的完整实现。本节拆解其 6 阶段面试流程、TodoList 驱动、System Prompt 工程和上下文注入。

---

## 一、6 阶段面试流程设计

### 1.1 流程全景

```
┌──────────────────────────────────────────────────────────────────┐
│  阶段1: 开场 & 自我介绍                                          │
│  "你好，我是今天的面试官。请先做一个简单的自我介绍。"              │
│  目标：破冰，获取候选人背景概述                                   │
├──────────────────────────────────────────────────────────────────┤
│  阶段2: 项目经历追问                                             │
│  "你在XX项目中提到用了微服务架构，能具体说说服务拆分依据吗？"      │
│  目标：基于简历深度追问，验证项目经验真实性                        │
├──────────────────────────────────────────────────────────────────┤
│  阶段3: 技术知识提问                                             │
│  "请解释一下 MySQL 的索引下推优化是什么？"                        │
│  目标：考察技术基础是否扎实（结合岗位方向随机抽题）                │
├──────────────────────────────────────────────────────────────────┤
│  阶段4: 代码考核                                                 │
│  "请实现一个 LRU Cache，支持 O(1) 的 get 和 put 操作。"           │
│  目标：考察编码能力和问题解决能力（接入在线判题系统）              │
├──────────────────────────────────────────────────────────────────┤
│  阶段5: 匹配度评估                                               │
│  "你觉得你的哪些经验最契合这个岗位？有什么短板需要补？"            │
│  目标：双向匹配，识别风险点                                       │
├──────────────────────────────────────────────────────────────────┤
│  阶段6: 总结与评分                                               │
│  输出结构化评分卡（技术基础/项目经验/沟通表达/岗位匹配）            │
│  目标：给出量化评价和改进建议                                     │
└──────────────────────────────────────────────────────────────────┘
```

### 1.2 TodoListMiddleware 驱动机制

这是面试流程**可控性**的关键所在。

```python
INTERVIEW_TODO_PROMPT = """## write_todos

你正在进行一场模拟面试。你必须始终维护固定的 6 个 todo...

使用规则：
- 首轮正式发问前先初始化 6 条任务：第 1 条为 in_progress，其余为 pending。
- 开场问题发出后：第 1 条改为 completed，第 2 条改为 in_progress。
- ...
- 代码考核阶段，除非用户明确请求提示，否则不要主动点评代码或给出解法。
- 每轮回答最多调用一次 write_todos。
- 除这 6 条固定任务外，不要创建任何额外 todo。
"""
```

**为什么 TodoList 如此重要？**

在 LLM Agent 中，没有 TodoList 的情况下，Agent 的行为依赖于 system prompt 中的自然语言描述，这导致：
1. **流程不可控**：Agent 可能跳过某些阶段或重复提问
2. **状态不可观测**：无法知道当前面试进行到哪一步
3. **调试困难**：不知道是 prompt 问题还是模型问题

TodoList 机制将"流程控制"从"自然语言"提升到"结构化的工具调用"，Agent 必须显式更新任务状态，这既约束了行为，也提供了可观测性。

**TodoList 的状态转换示例**：

```
[初始化]
1. 发起开场并请候选人自我介绍 → in_progress
2. 追问项目经历与技术细节 → pending
3. 相关技术知识提问 → pending
4. 代码考核 → pending
5. 评估岗位匹配度与风险点 → pending
6. 输出总结与评分卡 → pending

[发出开场问题后]
1. 发起开场并请候选人自我介绍 → completed
2. 追问项目经历与技术细节 → in_progress
...
```

---

## 二、System Prompt 工程深度分析

### 2.1 完整 System Prompt 拆解

```python
INTERVIEW_SYSTEM_PROMPT = """你是一名专业、克制、友好的中文技术面试官...

你的行为规则：
1. 始终以面试官身份发言，不要代替候选人作答。
2. 面试节奏遵循固定 6 个阶段...
3. 如果系统已经注入选中简历上下文，优先直接使用该上下文...
4. 问题要口语化、简洁、有连续性...
5. 在第 4 阶段，发出每一道技术题前都调用 pick_random_technical_question...
6. 当第 4 阶段结束...必须调用 start_code_assessment...
7. 启动代码考核前，你要先根据候选人在前 4 阶段的表现判断编程题难度...
   - 明显吃力、基础薄弱 → easy
   - 整体合格但不算突出 → medium
   - 扎实、细节到位 → hard
   - 拿不准，默认使用 medium
8. 代码考核阶段不要主动点评代码...
9. 代码考核完成后，继续完成第 6、7 阶段。
10. 输出最终总结时，给出明确的岗位匹配判断...
"""
```

### 2.2 Prompt 工程要点

| 技巧 | 在 prompt 中的体现 | 为什么有效 |
|------|-------------------|-----------|
| **角色锚定** | "专业、克制、友好的中文技术面试官" | 三个限定词精确定义行为边界 |
| **强制约束** | "不要代替候选人作答" | 负面指令比正面指令更有效 |
| **结构化指令** | 1-10 条行为规则 | 编号比段落更容易被 LLM 遵循 |
| **决策规则** | "拿不准，默认使用 medium" | 为歧义场景提供默认行为，减少随机性 |
| **工具使用时机** | "发出每一道技术题前都调用..." | 精确描述何时调用工具 |
| **输出格式约束** | "评分卡请继续保持现有 interview_scorecard 代码块格式" | 确保输出可解析 |

### 2.3 简历上下文注入

```python
@classmethod
def build_runtime_system_prompt(cls, ...):
    # 1. 格式化基础 prompt
    rendered_prompt = template.format(
        target_position=normalized_position,
        interview_round=normalized_round,
    )
    # 2. 拼接简历上下文块
    return rendered_prompt + build_selected_resume_prompt_block(
        selected_resume_filename=...,
        selected_resume_summary=...,
        selected_resume_structured=...,
        selected_resume_markdown_excerpt=...,
    )
```

注入的简历上下文包含四个层次：
- **结构化字段**：姓名、学历、工作年限（用于快速匹配）
- **摘要**：LLM 生成的简历摘要（用于全局了解）
- **正文摘录**：原文关键段落（用于细节追问）
- **文件名**：供 read_file 工具使用的路径

---

## 三、核心面试工具分析

### 3.1 query_kb — RAG 知识检索

```python
# 用途：从知识库检索技术知识，辅助提问
# 限制：通过 ToolCallLimitMiddleware 限制为每场面试最多1次
# 场景：面试官需要参考某个技术点的标准答案时
```

### 3.2 pick_random_technical_question — 随机抽题

```python
# 关键参数：
excluded_questions: list[str]  # 已问过的问题列表，避免重复
# 策略：通过传入 excluded_questions 确保每道题只问一次
```

**抽题流程**：
```
Agent 准备发题
  → 调用 pick_random_technical_question(excluded_questions=["已问题1", "已问题2"])
  → 后端从题库中排除已问题目，随机抽取
  → 返回题目内容
  → Agent 将题目发给候选人
  → 将此题加入 excluded_questions 列表
```

### 3.3 start_code_assessment — 代码考核启动

```python
# 参数：
difficulty_level: str  # "easy" | "medium" | "hard"
# 行为：
# 1. 根据难度随机抽取一道编程题
# 2. 为当前 thread 创建代码考核会话
# 3. 返回 workbench_path（前端工作台路径）
# 4. Agent 告知用户前往工作台答题
```

---

## 四、VideoContextMiddleware — 视频面试增强

```python
# 中间件注入的后台上下文格式
<internal_interview_observation>
情绪状态：紧张（置信度 0.78）
注意力水平：中等（连续注视屏幕）
姿态：身体前倾，手部动作较多
</internal_interview_observation>
```

**使用规则（来自 System Prompt）**：
- 不向面试者提及"视频分析""摄像头""表情识别"
- 观察到紧张 → 调整语气温和，给予鼓励
- 注意力下降 → 换更有趣的话题
- 自信流畅 → 适当增加问题深度

> **设计分析**：视频分析作为"后台上下文"而非"前台工具"，Agent 可以将非语言信号融入对话策略，但不会让候选人感到被监视。这是隐私与体验的平衡。

---

## 五、从零手搓面试 Agent 的关键要点

### 5.1 流程可控性

```
❌ 纯 Prompt 控制 → 行为随机、不可预测
✅ TodoList + Prompt 双约束 → 可预测、可观测
```

### 5.2 上下文注入时机

```
❌ 在中间件中注入 → 但可能被后续中间件覆盖
✅ 在 Context.from_file() 中就 build 好 system_prompt
```

### 5.3 工具安全边界

```
❌ 给 Agent 完整的文件系统访问 → 安全风险
✅ 只给 read_file + 白名单路径 → 最小权限
```

### 5.4 代码考核阶段隔离

```
Agent 发题 → 用户去工作台答题 → OJ 判题 → 结果回传
              ↑ Agent 不做代码审查       ↑ 客观评分
```

---

## 下一步学习

阅读 [[04-rag-knowledge-system|RAG 知识库系统]]，深入理解文档解析→分块→Embedding→Rerank→检索的完整链路。
