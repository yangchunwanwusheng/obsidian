---
type: lesson
tags: [RAG, 基础知识, 检索增强生成, 向量数据库]
created: 2026-05-25
updated: 2026-05-25
difficulty: beginner
prerequisites: [Python基础]
topic: 开源项目深度拆解
status: completed
series: { name: "ai-interview-伯乐深度拆解-基础篇", part: 1 }
sources: []
---

# RAG 是什么？—— 从零理解检索增强生成

> 为什么大模型需要"外挂"？RAG 到底解决了什么问题？Embedding 又是怎么工作的？本文用最通俗的语言讲清 RAG 的完整原理。

---

## 一、没有 RAG 的大模型有什么问题？

假设你问 ChatGPT："伯乐项目中用的是哪个 Embedding 模型？"

**没有 RAG 的模型会这样回答**：

> "抱歉，我的训练数据截止到 2024 年，无法获取伯乐项目的具体信息。不过常见的 Embedding 模型有..."

问题很明显：
1. **知识截止** —— 模型不知道训练数据之后发生的事情
2. **幻觉** —— 模型可能会编造一个看似合理但完全错误的答案
3. **私有知识** —— 模型不知道你公司内部的文档、代码、规范

RAG 就是解决这三个问题的方案。

---

## 二、RAG 的核心思想 —— 给模型一本"参考书"

**RAG = Retrieval-Augmented Generation（检索增强生成）**

通俗理解：
```
你问："伯乐用的什么 Embedding？"

传统 LLM：         RAG：
大脑里搜 → 没找到 → 瞎编    先去知识库找相关文档 → 找到配置说明
                                     ↓
                              带着文档内容问 LLM：
                              "根据这段配置，回答用户问题"
                                     ↓
                              LLM："根据配置，伯乐使用 BGE-M3，1024 维"
```

**说白了就是：先检索相关资料，再把资料和问题一起喂给 LLM，让它基于事实回答。**

---

## 三、RAG 的完整工作流程

```
┌──────────────────────────────────────────────────────────────────┐
│                        离线阶段（建库）                            │
│                                                                  │
│  文档 PDF/Word/Markdown                                           │
│       │                                                          │
│       ▼                                                          │
│  ┌─────────┐    ┌─────────┐    ┌──────────┐    ┌──────────────┐ │
│  │ 文档解析 │ → │  分块   │ → │ Embedding │ → │ 向量数据库存储 │ │
│  │         │    │ (Chunk) │    │ (向量化)  │    │              │ │
│  └─────────┘    └─────────┘    └──────────┘    └──────────────┘ │
│                                                                  │
├──────────────────────────────────────────────────────────────────┤
│                        在线阶段（检索+生成）                        │
│                                                                  │
│  用户提问："伯乐用的什么 Embedding？"                               │
│       │                                                          │
│       ▼                                                          │
│  ┌──────────┐    ┌──────────┐    ┌──────────┐    ┌─────────────┐ │
│  │ 提问向量化│ → │ 相似度搜索│ → │ 取 Top-K │ → │ 拼接 Prompt │ │
│  │(Embedding)│   │(向量数据库)│   │ 相关文档  │    │             │ │
│  └──────────┘    └──────────┘    └──────────┘    └──────┬──────┘ │
│                                                         │        │
│                                                         ▼        │
│                                              ┌─────────────────┐ │
│                                              │   LLM 生成回答   │ │
│                                              │ (基于检索结果)    │ │
│                                              └─────────────────┘ │
└──────────────────────────────────────────────────────────────────┘
```

### 每一步具体在做什么？

#### 步骤 1：文档解析

把 PDF/Word 等格式的文件转成纯文本。过程中要：
- 识别标题、段落、表格
- OCR 识别图片中的文字
- 保持文档结构（知道哪些文字属于同一个段落）

#### 步骤 2：分块（Chunking）

把长文档切成小块。为什么？

```
整本《Java 面试指南》可能有 50 万字。
如果一次全部塞给 LLM → 上下文窗口装不下，且检索精度极差。

切成 500 字一块 → 每块是一个独立的知识单元
用户问"HashMap 原理" → 只检索到相关的那几块 → 精准高效
```

#### 步骤 3：Embedding（向量化）—— 这是最核心的概念

**Embedding 就是把文字变成一串数字（向量）。**

```
"HashMap 的工作原理" → [0.023, -0.451, 0.782, ..., -0.134]  (1024个数字)
"ArrayList 的工作原理" → [0.019, -0.447, 0.775, ..., -0.128]  (1024个数字)
"今天天气真好"       → [-0.831, 0.212, -0.091, ..., 0.667]  (1024个数字)
```

**关键特性**：语义相近的文字，向量也相近。

```
cos_similarity("HashMap", "ArrayList") = 0.95  ← 都是 Java 集合类，向量很接近
cos_similarity("HashMap", "天气")      = 0.12  ← 完全不相关，向量很远
```

这就是 Embedding 模型（如 BGE-M3）的神奇之处——它把语义相似度转化成了数学上的向量距离。

#### 步骤 4：向量数据库存储

把切好的文本块和对应的向量一起存起来，方便快速搜索。

#### 步骤 5：检索（Retrieval）

用户提问 → 同样做 Embedding → 在向量数据库中找最相似的 Top-K 个文档块。

```
用户问："HashMap 怎么解决哈希冲突？"

向量检索结果（按相似度排序）：
1. [0.95] "HashMap 使用链表法解决哈希冲突..."  ← 最相关
2. [0.87] "哈希冲突是指不同 key 映射到同一个桶..."
3. [0.72] "Java 8 中 HashMap 引入了红黑树优化..."
4. [0.45] "ConcurrentHashMap 的线程安全实现..."
5. [0.31] "ArrayList 的扩容机制..."              ← 不太相关了
```

#### 步骤 6：增强生成（Augmented Generation）

把检索到的文档和用户问题拼接成一个 Prompt：

```
System: 你是一个技术助手。请基于以下参考资料回答问题。
如果参考资料中没有相关信息，请说"我目前没有这方面的资料"。

参考资料：
---
[文档1] HashMap 使用链表法解决哈希冲突，当链表长度超过 8 时转为红黑树...
[文档2] 哈希冲突是指不同 key 映射到同一个桶位置的情况...
---

用户问题：HashMap 怎么解决哈希冲突？
```

LLM 基于这些资料回答，基本不会胡说八道。

---

## 四、RAG 的关键组件对照（以伯乐为例）

| RAG 组件    | 伯乐使用的技术                         | 作用                 |
| --------- | ------------------------------- | ------------------ |
| 文档解析      | MinerU / PaddleX / DeepSeek OCR | PDF → 纯文本 Markdown |
| 分块策略      | General / QA / Books / Laws     | 按场景最优方式切分文本        |
| Embedding | BGE-M3 @ SiliconFlow            | 文字 → 1024 维向量      |
| 向量数据库     | OpenViking                      | 存储向量 + 快速检索        |
| Rerank    | Cross-Encoder 模型                | 对检索结果二次精排          |
| LLM 生成    | DeepSeek / Qwen / GLM           | 基于检索结果生成回答         |

---

## 五、RAG 的三个进化阶段

### Naive RAG（朴素 RAG）
```
文档 → 分块 → Embedding → 检索 → 拼接 Prompt → LLM 生成
```
问题：检索不精准，可能漏掉重要信息或引入无关信息。

### Advanced RAG（进阶 RAG）
```
文档 → 分块 → Embedding → 检索 → **Rerank 重排序** → **混合检索** → LLM 生成
```
改进：加入 Rerank 提高精度，加入关键词检索（BM25）弥补纯向量检索的不足。

### Modular RAG（模块化 RAG）—— 伯乐使用的
```
文档 → 分块 → Embedding → 检索 → Rerank → **元数据过滤** → **知识图谱** → LLM 生成
                                  ↑                    ↑
                            按来源/难度过滤          关联知识发现
```
改进：支持按元数据过滤（如只查"中级"难度的 Java 题目），引入知识图谱发现关联知识。

---

## 六、常见误区

| 误解 | 真相 |
|------|------|
| "RAG 就是向量搜索" | RAG = 检索 + 生成，向量搜索只是检索的一种方式 |
| "Embedding 维度越高越好" | 1024 维 BGE-M3 在很多任务上不输 4096 维模型 |
| "分块越大越好" | 太大精度低，太小语义不完整。500 token 是常用平衡点 |
| "RAG 可以完全消除幻觉" | RAG 大幅减少但无法完全消除，LLM 仍可能误解资料 |
| "RAG 不需要微调模型" | 这是 RAG 的优势（不需要微调），但结合微调（RAFT）效果更好 |

---

## 七、手搓一个最小 RAG 系统（50 行代码）

```python
import numpy as np
from openai import OpenAI

# 1. 准备知识库文档
documents = [
    "HashMap 使用链表法解决哈希冲突，Java 8 引入红黑树优化",
    "ArrayList 底层是动态数组，默认容量 10，扩容 1.5 倍",
    "ConcurrentHashMap 使用分段锁保证线程安全",
    "LinkedList 是双向链表，适合频繁插入删除",
]

# 2. 创建 Embedding 客户端
client = OpenAI(
    api_key="your-api-key",
    base_url="https://api.siliconflow.cn/v1"
)

# 3. 向量化所有文档
doc_embeddings = []
for doc in documents:
    resp = client.embeddings.create(
        model="Pro/BAAI/bge-m3",
        input=doc,
    )
    doc_embeddings.append(resp.data[0].embedding)

# 4. 向量化用户问题
question = "HashMap 怎么处理冲突？"
q_resp = client.embeddings.create(
    model="Pro/BAAI/bge-m3",
    input=question,
)
q_embedding = q_resp.data[0].embedding

# 5. 计算相似度（余弦相似度）
def cosine_similarity(a, b):
    return np.dot(a, b) / (np.linalg.norm(a) * np.linalg.norm(b))

scores = [cosine_similarity(q_embedding, d) for d in doc_embeddings]
top_k_idx = np.argsort(scores)[-2:][::-1]  # Top-2

# 6. 拼接 Prompt 并生成回答
context = "\n".join([documents[i] for i in top_k_idx])
prompt = f"""基于以下参考资料回答问题：
---
{context}
---
问题：{question}
回答："""

response = client.chat.completions.create(
    model="deepseek-ai/DeepSeek-V3",
    messages=[{"role": "user", "content": prompt}],
)
print(response.choices[0].message.content)
```

这个 50 行的代码就是一个完整的 RAG 系统！伯乐项目本质上就是这个流程的**工程化版本**。

---

## 下一步学习

阅读 [[02-langgraph-agent-basics|LangGraph 与 Agent 基础概念]]，理解什么是 Agent、LangGraph 如何编排工具调用循环。
