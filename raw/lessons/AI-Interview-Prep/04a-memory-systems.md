---
type: lesson
tags: [面试, Memory, 记忆系统, 短期记忆, 长期记忆, MemGPT, 对话管理]
created: 2026-05-20
updated: 2026-05-20
difficulty: advanced
prerequisites: [LLM基础, Agent基础, RAG基础]
topic: AI记忆系统架构深入
status: in-progress
series: {name: "AI Interview Prep", part: "4a"}
---

# AI 记忆系统架构深入

> 深入理解 AI Agent 记忆系统的分层架构、工程实现与面试高频考点。从短期对话管理到长期记忆检索，掌握可落地的记忆系统设计方案。

## 学习目标

- [ ] 理解记忆系统的三层分类学及其工程对应
- [ ] 掌握对话历史的 Token 预算管理与摘要压缩实现
- [ ] 深入分析 MemGPT/Letta 的虚拟内存架构
- [ ] 实现一个可运行的长期记忆管理器
- [ ] 设计用户画像系统并理解隐私合规要求

## 一、记忆系统分类学深入

### 人类记忆模型的工程映射

认知心理学中的 Atkinson-Shiffrin 模型将人类记忆分为感觉记忆、短期记忆和长期记忆三个阶段。AI 系统的记忆设计可以精确类比这一模型：

```
人类记忆模型                    AI 记忆系统
─────────────                  ────────────────
感觉记忆                        感觉记忆（输入缓冲区）
│  < 1秒，高容量                 │  原始多模态输入的暂存
│  视觉/听觉/触觉                │  图像像素、音频波形、文本 token
↓                               ↓
短期记忆 → 工作记忆              工作记忆（上下文窗口）
│  7±2 信息块                    │  当前对话上下文 + 任务状态
│  15-30秒                       │  受模型窗口大小限制
↓                               ↓
长期记忆                        长期记忆（持久化存储）
   ├── 情景记忆                     ├── 对话历史摘要
   │   "上周三我和张三吃了火锅"       │   "用户在 session_42 问过退货政策"
   ├── 语义记忆                     ├── 用户画像 / 知识库
   │   "巴黎是法国首都"              │   "用户偏好 Python，资深后端"
   └── 程序记忆                     └── 工具使用模式 / SOP
       "骑自行车的技能"                 "该用户总用 curl 测试 API"
```

### AI 记忆三层架构详解

| 维度 | 感觉记忆 | 工作记忆（短期） | 长期记忆 |
|------|---------|----------------|---------|
| **存储介质** | 内存缓冲区（numpy/tensor） | LLM Context Window | 向量数据库/KV Store/知识图谱 |
| **容量** | 受输入模态限制 | 4K-2M tokens | 无理论上限 |
| **生命周期** | 单次请求内 | 单次会话内 | 跨会话持久化 |
| **读写频率** | 写一次、读一次 | 每轮读写 | 低频写入、检索式读取 |
| **典型技术** | Whisper/ViT 预处理 | Token 计数 + 摘要压缩 | Embedding + ANN 检索 |
| **失败模式** | 信号丢失/噪声 | 上下文溢出/关键信息被截断 | 检索不准/记忆过时 |

```
┌──────────────────────────────────────────────────────┐
│                    AI Agent 记忆架构                    │
│                                                        │
│  ┌──────────┐   ┌──────────────────┐   ┌───────────┐ │
│  │ 感觉记忆  │   │    工作记忆        │   │  长期记忆  │ │
│  │          │   │                   │   │           │ │
│  │ 图片像素  │──▶│ System Prompt     │   │ 用户画像   │ │
│  │ 音频波形  │   │ + 检索结果        │◀──│ 对话摘要   │ │
│  │ 文本输入  │   │ + 对话历史        │   │ 知识库     │ │
│  │          │   │ + 输出空间        │   │ 工具日志   │ │
│  └──────────┘   └──────────────────┘   └───────────┘ │
│       │                  │                    ▲        │
│       │          Token Budget Mgr     Memory Manager  │
│       │          Summary Compressor   Retrieval Engine│
│       └────────────────────────────────────────────── │
└──────────────────────────────────────────────────────┘
```

> [!hint]- 面试题：为什么 AI 记忆系统需要分层？用一个统一的向量数据库存所有东西不行吗？
> 思考方向：不同类型信息的访问模式、延迟要求、精度要求。

> [!success]- 参考答案
> 分层的核心原因是**不同类型记忆的访问模式差异巨大**：
>
> 1. **延迟要求不同**：工作记忆每轮对话都必须读取，要求毫秒级；长期记忆只在需要时检索，可以容忍百毫秒延迟。
> 2. **精度要求不同**：当前对话的精确 token 必须一字不差地保留（"把那个改成红色"需要知道"那个"是什么）；长期记忆用向量相似度检索，允许模糊匹配。
> 3. **存储成本不同**：LLM Context Window 里的每个 token 都要付出推理成本（$0.01-0.1/1K tokens）；向量存储成本极低（$0.0001/1K vectors）。
> 4. **访问模式不同**：工作记忆是**顺序扫描**（模型注意力机制），长期记忆是**随机检索**（ANN 查询）。
>
> 统一存储会导致：短期记忆检索变慢（向量匹配不如精确保留），长期记忆成本过高（塞进 Context Window 太贵）。

## 二、短期记忆：对话历史管理深入

### Token 预算分配策略

每个模型调用时，可用 token 是有限的。一个工程化的预算分配方案：

```
总 Token 预算（以 GPT-4o 为例，128K）
│
├── System Prompt:        固定 ~500 tokens
├── Memory Injection:     动态 ~1000 tokens（用户画像 + 摘要）
├── RAG Results:          动态 ~2000 tokens（检索到的文档片段）
├── Tool Definitions:     固定 ~500 tokens
├── Conversation History:  动态，占剩余空间的大部分
├── Few-shot Examples:    可选 ~300 tokens
└── Output Space:         预留 ~2000 tokens（生成回复的空间）
```

```python
import tiktoken
from dataclasses import dataclass, field
from typing import List, Optional

@dataclass
class TokenBudget:
    """Token 预算管理器——精确控制每部分占用的 token 数。"""
    model: str = "gpt-4o"
    total_limit: int = 128_000

    # 固定开销
    system_prompt_tokens: int = 500
    tool_def_tokens: int = 500
    output_reserve_tokens: int = 2_000

    # 动态开销上限
    max_memory_tokens: int = 1_000
    max_rag_tokens: int = 2_000
    max_fewshot_tokens: int = 300

    def __post_init__(self):
        self._encoder = tiktoken.encoding_for_model(self.model)

    def count_tokens(self, text: str) -> int:
        return len(self._encoder.encode(text))

    def available_for_history(self, memory_tokens: int = 0,
                               rag_tokens: int = 0,
                               fewshot_tokens: int = 0) -> int:
        """计算当前可用于对话历史的 token 数。"""
        used = (self.system_prompt_tokens
                + self.tool_def_tokens
                + self.output_reserve_tokens
                + min(memory_tokens, self.max_memory_tokens)
                + min(rag_tokens, self.max_rag_tokens)
                + min(fewshot_tokens, self.max_fewshot_tokens))
        return max(0, self.total_limit - used)

# 主流模型上下文窗口对比
MODEL_CONTEXT_WINDOWS = {
    "gpt-4o":              128_000,
    "gpt-4o-mini":         128_000,
    "gpt-4-turbo":         128_000,
    "gpt-3.5-turbo":       16_385,
    "claude-3.5-sonnet":   200_000,
    "claude-3-opus":       200_000,
    "gemini-1.5-pro":      1_000_000,
    "gemini-1.5-flash":    1_000_000,
    "deepseek-v3":         128_000,
    "qwen-72b":            131_072,
}
```

### 摘要压缩的完整实现

当对话历史超过 token 预算时，需要对历史进行压缩。**滑动窗口 + 滚动摘要**是工程中最常用的方案：

```
对话历史处理流程：

[Msg1] [Msg2] [Msg3] [Msg4] [Msg5] [Msg6] [Msg7] [Msg8]
  │                                          │
  └────── 摘要压缩 ──────┐                    │
                         ▼                    ▼
                    [Summary]          [Msg5] [Msg6] [Msg7] [Msg8]
                    │                  │
                    └── 保留最近 N 轮 ──┘
                         │
                         ▼
              最终送入 LLM 的上下文
```

```python
from datetime import datetime
from typing import List, Dict, Tuple
import json

class ConversationMemoryManager:
    """对话历史管理器：滑动窗口 + 滚动摘要。

    策略：
    - 保留最近 N 轮完整对话（滑动窗口）
    - 超出窗口的历史压缩为摘要
    - 摘要逐层合并，避免信息指数级丢失
    """

    def __init__(self, model: str = "gpt-4o",
                 keep_recent_turns: int = 6,
                 summary_max_tokens: int = 800):
        self.model = model
        self.keep_recent_turns = keep_recent_turns
        self.summary_max_tokens = summary_max_tokens
        self.messages: List[Dict] = []
        self.summary: Optional[str] = None
        self.budget = TokenBudget(model=model)

    def add_message(self, role: str, content: str):
        self.messages.append({
            "role": role,
            "content": content,
            "timestamp": datetime.now().isoformat()
        })

    def _count_history_tokens(self) -> int:
        total = 0
        for msg in self.messages:
            total += self.budget.count_tokens(msg["content"])
            total += 4  # role 标记开销
        return total

    def _extract_key_info(self, old_summary: str,
                          messages_to_compress: List[Dict]) -> str:
        """调用 LLM 生成摘要，保留关键信息。

        Prompt 设计要点：
        1. 明确告知摘要用途——供后续对话参考
        2. 列出必须保留的信息类型
        3. 给出字数限制
        """
        conversation_text = "\n".join(
            f"{m['role']}: {m['content']}" for m in messages_to_compress
        )
        prompt = f"""请将以下对话历史压缩为简洁摘要。

已有摘要：
{old_summary or '（无）'}

新增对话：
{conversation_text}

要求：
1. 保留所有用户提到的具体信息（姓名、日期、数字、偏好）
2. 保留所有已做出的决策和结论
3. 保留未解决的问题或待办事项
4. 省略寒暄、重复确认等冗余内容
5. 不超过 {self.summary_max_tokens} tokens"""

        # 实际工程中这里调用 LLM
        # response = client.chat.completions.create(
        #     model=self.model, messages=[{"role": "user", "content": prompt}]
        # )
        # return response.choices[0].message.content
        return prompt  # placeholder

    def should_compress(self, available_tokens: int) -> bool:
        """判断是否需要压缩。"""
        return self._count_history_tokens() > available_tokens

    def compress(self, available_tokens: int):
        """执行压缩：保留最近 N 轮，其余压缩为摘要。"""
        if len(self.messages) <= self.keep_recent_turns * 2:
            return  # 消息太少，不压缩

        # 分割：旧消息 vs 最近消息
        split_idx = len(self.messages) - self.keep_recent_turns * 2
        old_messages = self.messages[:split_idx]
        self.messages = self.messages[split_idx:]

        # 滚动摘要：旧摘要 + 旧消息 → 新摘要
        self.summary = self._extract_key_info(self.summary, old_messages)

    def get_context(self, rag_context: str = "",
                     memory_context: str = "") -> List[Dict]:
        """构建最终送入 LLM 的上下文。"""
        available = self.budget.available_for_history(
            memory_tokens=self.budget.count_tokens(memory_context) if memory_context else 0,
            rag_tokens=self.budget.count_tokens(rag_context) if rag_context else 0
        )

        # 检查并压缩
        if self.should_compress(available):
            self.compress(available)

        context = []
        if self.summary:
            context.append({
                "role": "system",
                "content": f"[对话历史摘要]\n{self.summary}"
            })
        context.extend(self.messages)
        return context
```

### 回指消解（Coreference Resolution）

多轮对话中，用户常用"它"、"这个"、"上面说的那个"指代前文内容。这是短期记忆的核心挑战之一。

> [!hint]- 面试题：对话已经进行了 200 轮，用户突然问"那个问题后来怎么解决的？"——你的系统怎么处理？
> 思考方向：回指消解 + 摘要中的信息保留 + 检索增强。

> [!success]- 参考答案
> 需要多层策略配合：
>
> 1. **摘要层保留**：滚动摘要时，对"未解决的问题"做显式标记，确保摘要中包含每个未解决问题的状态。
>
> 2. **回指消解**：用一个轻量级 LLM 调用将代词替换为具体指代对象：
>    ```
>    原句："那个问题后来怎么解决的？"
>    消解后："[第 47 轮] 用户报告的 API 超时问题后来怎么解决的？"
>    ```
>    消解时需要访问最近的对话历史 + 摘要中的"未解决问题"列表。
>
> 3. **长期记忆检索**：如果回指消解后仍然模糊，用消解后的文本作为 query，从长期记忆中检索最相关的对话片段。
>
> 4. **置信度评估**：如果系统对回指消解结果不确定，主动向用户确认："您是指 API 超时的问题吗？"
>
> 工程实现上，最关键的是**摘要质量**——摘要必须保留"待解决问题"的结构化列表，而不是笼统地说"讨论了 API 问题"。

## 三、长期记忆深入

### MemGPT / Letta 架构完整分析

MemGPT（Memory-GPT，现更名为 Letta）将操作系统的虚拟内存概念引入 LLM Agent，是记忆系统研究中最具影响力的工作之一。

```
MemGPT 虚拟内存类比：

┌─────────────────────────────────────────────┐
│              Operating System                │
│  ┌─────────────┐     ┌──────────────────┐   │
│  │  Main Memory │     │   Hard Disk      │   │
│  │  (RAM)       │     │   (SSD)          │   │
│  │              │     │                  │   │
│  │ Active pages │◀───▶│ Swapped pages    │   │
│  │ of process   │     │ and files        │   │
│  └─────────────┘     └──────────────────┘   │
│       Page In/Out via MMU                    │
└─────────────────────────────────────────────┘

┌─────────────────────────────────────────────┐
│              MemGPT Architecture             │
│  ┌─────────────┐     ┌──────────────────┐   │
│  │ Core Memory  │     │ Archival Memory  │   │
│  │ (In-Context) │     │ (Vector DB)      │   │
│  │              │     │                  │   │
│  │ User profile │◀───▶│ Full conversation│   │
│  │ Key facts    │     │ Documents        │   │
│  │ Current task │     │ External data    │   │
│  └─────────────┘     └──────────────────┘   │
│       Memory operations via function calls   │
└─────────────────────────────────────────────┘
```

**MemGPT 的四个核心操作**：

| 操作 | 作用 | 类比 |
|------|------|------|
| `core_memory_append(key, value)` | 向 Context 中的记忆块追加信息 | 在 RAM 中写入新数据 |
| `core_memory_replace(old, new)` | 替换 Context 中的已有记忆 | 更新 RAM 中的数据 |
| `archival_memory_insert(content)` | 将信息存入向量数据库 | 数据写入硬盘 |
| `archival_memory_search(query)` | 从向量数据库检索信息 | 从硬盘读取到 RAM |

**MemGPT 的 Prompt 设计精髓**：让 LLM 自己决定何时读写记忆，通过 function calling 实现：

```python
# MemGPT 核心 Prompt（简化版）
MEMGPT_SYSTEM_PROMPT = """You are MemGPT, a memory-augmented AI assistant.

## Your Memory System
You have two types of memory:

### Core Memory (limited, always visible):
- Stored inside your context window
- Use for: user name, preferences, current task state, key facts
- Operations: core_memory_append, core_memory_replace
- WARNING: Space is limited. Only store CRITICAL information here.

### Archival Memory (unlimited, search-based):
- Stored in an external vector database
- Use for: conversation history, documents, detailed notes
- Operations: archival_memory_insert, archival_memory_search
- Access via search - you must query to retrieve information.

## Memory Management Rules
1. When you learn new user info (name, preferences, background):
   → core_memory_append to save it
2. When user info changes (new job, updated preferences):
   → core_memory_replace to update it
3. When conversation contains important details to remember:
   → archival_memory_insert to archive it
4. When you need past information you don't have in context:
   → archival_memory_search to find it
5. Periodically review core_memory and remove outdated items
"""
```

> [!hint]- 面试题：MemGPT 让 LLM 自己管理记忆，这有什么风险？如何缓解？
> 思考方向：LLM 不可靠的"判断力"、记忆冲突、token 浪费。

> [!success]- 参考答案
> MemGPT 的"LLM 自管理"模式存在以下风险：
>
> 1. **写入噪声**：LLM 可能将不重要的信息写入 core_memory，浪费宝贵的 Context 空间。缓解：设置 core_memory 容量上限 + 定期清理策略。
>
> 2. **记忆冲突**：LLM 可能重复写入或写入矛盾信息（用户说"我喜欢红色"后写入，之后用户说"其实我更喜欢蓝色"但 LLM 忘记更新）。缓解：core_memory_replace 必须语义匹配旧值，而非盲目追加。
>
> 3. **读取遗漏**：LLM 可能忘记在需要时调用 archival_memory_search。缓解：在 System Prompt 中加入条件触发规则："当用户提到过去的内容时，必须先搜索 archival_memory"。
>
> 4. **额外 token 开销**：每次记忆操作都消耗 token。缓解：批量操作 + 异步写入（用户回复后后台处理）。
>
> 5. **安全性**：LLM 可能将敏感信息写入不安全的存储。缓解：写入前做 PII 检测和过滤。

### 长期记忆存储方案对比

| 方案 | 适用场景 | 优势 | 劣势 |
|------|---------|------|------|
| 向量数据库（Pinecone/Milvus/Chroma） | 语义检索、模糊匹配 | 灵活、支持相似度搜索 | 无法精确查询、缺乏结构 |
| 知识图谱（Neo4j） | 实体关系、推理链 | 结构化、支持图查询 | 构建成本高、维护复杂 |
| KV 存储（Redis/Postgres） | 精确查找、用户画像 | 速度快、确定性查询 | 无法模糊匹配 |
| 混合方案（推荐） | 全场景 | 各取所长 | 系统复杂度高 |

**推荐混合方案**：

```
用户画像 ──────▶ Redis / Postgres JSON（精确读写，O(1)）
对话摘要 ──────▶ 向量数据库（语义检索，ANN）
实体关系 ──────▶ 知识图谱（结构化推理）
文档知识 ──────▶ 向量数据库 + 元数据过滤
```

### 记忆写入策略：什么信息值得记忆

不是所有对话内容都值得写入长期记忆。需要一套信息筛选机制：

```python
from enum import Enum
from typing import List, Dict, Optional
import json

class MemoryType(Enum):
    USER_PROFILE = "user_profile"       # 用户属性：姓名、职业、偏好
    EPISODIC = "episodic"               # 事件记录："上周讨论了X方案"
    DECISION = "decision"               # 决策结论："决定使用方案B"
    PREFERENCE = "preference"           # 用户偏好："喜欢简洁的代码风格"
    FACT = "fact"                       # 事实信息："项目截止日期是3月15日"
    TASK_STATE = "task_state"           # 任务状态："代码审查进行到第3步"

class MemoryManager:
    """长期记忆管理器——负责写入、检索、更新的完整生命周期。"""

    MEMORY_IMPORTANCE_PROMPT = """评估以下信息是否值得长期记忆。

信息内容：{content}

评分标准（1-10分）：
- 10分：用户的核心属性（姓名、职业、关键偏好）
- 8-9分：重要决策、截止日期、明确表达的需求
- 5-7分：有参考价值的讨论结论
- 3-4分：临时性、可能很快变化的信息
- 1-2分：寒暄、通用知识、冗余信息

只返回一个数字。"""

    def __init__(self, importance_threshold: float = 5.0,
                 time_decay_rate: float = 0.95):
        self.importance_threshold = importance_threshold
        self.time_decay_rate = time_decay_rate
        self.memories: Dict[str, Dict] = {}  # memory_id -> memory_data
        self.user_profile: Dict = {}          # 快速查找的画像缓存

    async def should_memorize(self, content: str) -> Tuple[bool, float]:
        """判断信息是否值得记忆。返回 (是否记忆, 重要性分数)。"""
        # 工程实现：调用轻量模型或规则引擎
        # 这里用关键词启发式规则作为演示
        high_value_patterns = [
            "我叫", "我是", "我的名字",           # 自我介绍
            "我喜欢", "我偏好", "我不喜欢",       # 偏好表达
            "截止日期", "deadline", "到期",       # 时间约束
            "决定", "选择", "方案",               # 决策信息
            "注意", "记住", "别忘了",             # 显式记忆请求
        ]
        for pattern in high_value_patterns:
            if pattern in content:
                return True, 8.0
        return False, 3.0

    async def write_memory(self, content: str, memory_type: MemoryType,
                           user_id: str, metadata: Dict = None):
        """写入一条长期记忆。"""
        should, score = await self.should_memorize(content)
        if not should or score < self.importance_threshold:
            return None

        memory_id = f"{user_id}_{memory_type.value}_{len(self.memories)}"
        memory = {
            "id": memory_id,
            "user_id": user_id,
            "type": memory_type.value,
            "content": content,
            "importance": score,
            "created_at": datetime.now().isoformat(),
            "access_count": 0,
            "last_accessed": None,
            "metadata": metadata or {},
        }
        self.memories[memory_id] = memory

        # 用户画像实时更新
        if memory_type in (MemoryType.USER_PROFILE, MemoryType.PREFERENCE):
            self.user_profile[user_id] = {
                **self.user_profile.get(user_id, {}),
                "content": content,
                "updated_at": datetime.now().isoformat()
            }

        return memory_id

    async def retrieve(self, query: str, user_id: str,
                       top_k: int = 5) -> List[Dict]:
        """检索相关记忆，综合相关性和重要性排序。

        排序公式：score = relevance * importance * time_decay
        """
        candidates = [
            m for m in self.memories.values()
            if m["user_id"] == user_id
        ]

        scored = []
        for m in candidates:
            # 相关性：实际工程中用 embedding 余弦相似度
            relevance = self._compute_relevance(query, m["content"])
            # 时间衰减
            days_old = self._days_since(m["created_at"])
            time_factor = self.time_decay_rate ** days_old
            # 访问频率加权（被频繁引用的记忆更重要）
            access_factor = 1.0 + 0.1 * min(m["access_count"], 10)

            final_score = (
                relevance
                * (m["importance"] / 10.0)
                * time_factor
                * access_factor
            )
            scored.append((final_score, m))

        scored.sort(key=lambda x: x[0], reverse=True)

        # 更新访问计数
        results = []
        for score, m in scored[:top_k]:
            m["access_count"] += 1
            m["last_accessed"] = datetime.now().isoformat()
            results.append(m)

        return results

    def _compute_relevance(self, query: str, content: str) -> float:
        """简化的相关性计算（生产环境使用 embedding）。"""
        query_words = set(query.lower().split())
        content_words = set(content.lower().split())
        overlap = query_words & content_words
        if not query_words:
            return 0.0
        return len(overlap) / len(query_words)

    def _days_since(self, iso_date: str) -> int:
        dt = datetime.fromisoformat(iso_date)
        return (datetime.now() - dt).days

    def format_for_injection(self, memories: List[Dict],
                              max_tokens: int = 1000) -> str:
        """将检索到的记忆格式化为注入 System Prompt 的文本。"""
        sections = {
            "user_profile": [],
            "preference": [],
            "episodic": [],
            "decision": [],
            "fact": [],
        }
        for m in memories:
            sections.get(m["type"], sections["fact"]).append(m["content"])

        output_parts = ["[长期记忆]"]
        for mtype, items in sections.items():
            if items:
                output_parts.append(f"  {mtype}:")
                for item in items:
                    output_parts.append(f"    - {item}")

        return "\n".join(output_parts)
```

> [!hint]- 面试题：用户说"记住我喜欢深色主题"，你会怎么处理？如果下次对话用户说"还是浅色吧"呢？
> 思考方向：记忆写入 → 记忆更新 → 冲突处理。

> [!success]- 参考答案
> 完整流程：
>
> 1. **首次写入**：检测到"我喜欢深色主题" → 识别为偏好类记忆 → 写入 memory_type=PREFERENCE，content="用户偏好深色主题（UI/代码编辑器）"
>
> 2. **冲突检测**：当用户说"还是浅色吧"时：
>    - 检索现有记忆，发现已有 PREFERENCE "偏好深色主题"
>    - 语义匹配：新信息"浅色"与旧信息"深色"冲突
>    - 执行 core_memory_replace：将"偏好深色主题"替换为"偏好浅色主题"
>    - 保留历史记录：archival_memory_insert("用户从深色主题切换为浅色主题，原因未知")
>
> 3. **工程细节**：
>    - 偏好类记忆用 KV 存储，key="theme_preference"，方便精确查找和更新
>    - 冲突检测可以在写入前用一个轻量分类器判断新旧信息是否矛盾
>    - 审计日志：所有记忆变更记录到 changelog，支持回滚

## 四、用户画像（User Profile）系统

### 数据结构设计

```python
from pydantic import BaseModel, Field
from typing import Dict, List, Optional
from datetime import datetime

class UserProfile(BaseModel):
    """用户画像数据结构。"""
    user_id: str
    created_at: datetime = Field(default_factory=datetime.now)
    updated_at: datetime = Field(default_factory=datetime.now)

    # 基本信息（用户主动提供或从对话中推断）
    name: Optional[str] = None
    role: Optional[str] = None               # 职业/角色
    expertise_level: Optional[str] = None     # "beginner" / "intermediate" / "expert"
    language_preference: str = "zh-CN"        # 语言偏好

    # 偏好系统
    preferences: Dict[str, str] = Field(default_factory=dict)
    # 例：{"theme": "dark", "code_style": "verbose", "output_format": "markdown"}

    # 交互模式
    interaction_stats: Dict = Field(default_factory=dict)
    # 例：{"total_sessions": 42, "avg_turns_per_session": 8.5, "preferred_time": "evening"}

    # 兴趣/专长标签
    interests: List[str] = Field(default_factory=list)
    # 例：["Python", "LLM", "系统设计"]

    # 沟通风格偏好
    communication_style: Optional[str] = None
    # "concise" / "detailed" / "academic" / "casual"

    # 最近上下文（用于跨会话衔接）
    last_session_summary: Optional[str] = None
    active_projects: List[str] = Field(default_factory=list)
```

### 从对话中自动提取用户信息

```python
EXTRACTION_PROMPT = """从以下对话片段中提取用户信息。

对话内容：
{conversation}

请提取以下类型的信息（如果存在）：
- name: 用户姓名
- role: 职业/角色
- expertise_level: 技术水平（beginner/intermediate/expert）
- preferences: 偏好（如代码风格、输出格式、语言等）
- interests: 兴趣/专长领域

以 JSON 格式输出，只包含能从对话中明确推断的字段。
如果某个字段无法确定，不要输出该字段。
输出格式：
{{"field": "value", ...}}"""
```

### 画像在生成中的利用

画像注入 System Prompt 的最佳位置：

```
System Prompt 结构：
┌──────────────────────────────────┐
│ 1. 角色定义（你是XX助手）          │  ← 固定
├──────────────────────────────────┤
│ 2. 用户画像                      │  ← 从长期记忆注入
│    "用户名：张三                  │
│     职业：高级后端工程师           │
│     偏好：简洁回复、Python"       │
├──────────────────────────────────┤
│ 3. 当前任务/上下文                │  ← 从工作记忆注入
├──────────────────────────────────┤
│ 4. RAG 检索结果                   │  ← 按需注入
├──────────────────────────────────┤
│ 5. 对话历史                      │  ← 短期记忆
└──────────────────────────────────┘
```

> [!hint]- 面试题：如何设计一个合规的"记住用户偏好"系统？如果用户要求删除所有数据怎么办？
> 思考方向：GDPR 的"被遗忘权"、数据最小化原则、显式同意。

> [!success]- 参考答案
> 合规的用户画像系统必须满足：
>
> 1. **数据最小化**：只收集提供服务所必需的信息。不要存储用户的 IP、设备指纹等与核心功能无关的数据。
>
> 2. **显式同意**：首次使用记忆功能时，告知用户"我将记住您的偏好以提供更好的体验"，并提供 opt-out 选项。
>
> 3. **被遗忘权（GDPR Article 17）**：
>    ```python
>    async def delete_all_user_data(self, user_id: str):
>        """删除用户所有数据，包括：
>        - 用户画像（KV Store）
>        - 对话历史（向量数据库中 user_id 的所有记录）
>        - 审计日志中的个人信息
>        """
>        await self.kv_store.delete(f"profile:{user_id}")
>        await self.vector_db.delete(filter={"user_id": user_id})
>        await self.audit_log.anonymize(user_id)
>    ```
>
> 4. **数据可携带权**：支持导出用户全部数据为 JSON/CSV 格式。
>
> 5. **用途限制**：用户画像只用于改善服务，不得用于广告投放或出售给第三方。
>
> 6. **定期审查**：超过 90 天未访问的记忆自动降级或删除（Retention Policy）。

## 关键要点回顾

1. **记忆分层是工程必然**：不同类型信息有不同的访问模式和成本结构，分层存储是系统设计的最优解。
2. **Token 预算管理是短期记忆的核心**：精确的预算分配 + 滚动摘要压缩是工程落地的关键。
3. **MemGPT 的核心创新是让 LLM 自管理记忆**，但需要额外的质量保障机制（冲突检测、容量限制）。
4. **长期记忆的检索质量决定系统上限**：相关性 + 重要性 + 时间衰减的综合排序公式是通用方案。
5. **用户画像系统必须设计合规能力**：数据最小化、被遗忘权、显式同意是底线要求。

## 扩展阅读

- MemGPT Original Paper: "MemGPT: Towards LLMs as Operating Systems" (2023)
- Letta Framework: https://github.com/letta-ai/letta
- "Generative Agents: Interactive Simulacra of Human Behavior" (Stanford, 2023)
- "Advancing Conversational AI with Large Language Models: A Survey on Memory" (2024)

## 下一步学习

- [[raw/lessons/AI-Interview-Prep/04b-memory-advanced|高级记忆主题：DST、压缩遗忘、跨会话、多用户]]
