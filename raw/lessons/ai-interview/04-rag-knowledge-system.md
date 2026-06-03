---
type: lesson
tags: [RAG, 知识库, OpenViking, Embedding, Rerank, 分块策略, 文档解析]
created: 2026-05-25
updated: 2026-05-25
difficulty: advanced
prerequisites: [向量检索基础, Embedding概念, LangChain文档处理]
topic: 开源项目深度拆解
status: completed
series: { name: "ai-interview-伯乐深度拆解", part: 4 }
---

# RAG 知识库系统深度拆解

> 伯乐的 RAG 系统以 OpenViking 为核心，实现了从文档上传到智能检索的完整链路。本节拆解文件生命周期管理、分块策略、Embedding/Rerank 模型集成和检索优化。

---

## 一、整体架构

```
用户上传文件
    │
    ▼
┌──────────────────────────────────────────────────────────┐
│                 知识库文件生命周期                         │
│                                                          │
│  UPLOADED → PARSING → PARSED → INDEXING → INDEXED       │
│    │          │         │         │          │           │
│    │    文档解析中   解析完成   索引构建中   就绪可检索    │
│    │    (异步)               (异步)                      │
└──────────────────────────────────────────────────────────┘
    │                    │                    │
    ▼                    ▼                    ▼
┌──────────┐    ┌──────────────┐    ┌──────────────────┐
│ 文档解析  │    │   分块策略    │    │   向量化 & 索引   │
│          │    │              │    │                  │
│ MinerU   │    │ General分块  │    │ BGE-M3 Embedding │
│ PaddleX  │    │ QA分块       │    │ 1024维向量       │
│ DeepSeek │    │ Book分块     │    │ 存入 OpenViking  │
│ RapidOCR │    │ Laws分块     │    │                  │
│ docling  │    │              │    │                  │
└──────────┘    └──────────────┘    └──────────────────┘
                                            │
                                            ▼
                                    ┌──────────────┐
                                    │   检索阶段    │
                                    │              │
                                    │ 混合检索      │
                                    │ Rerank 重排序 │
                                    │ Top-K 返回    │
                                    └──────────────┘
```

---

## 二、知识库抽象层

### 2.1 KnowledgeBase 基类

`src/knowledge/base.py`（约 1200 行）是知识库系统的核心抽象：

```python
class KnowledgeBase(ABC):
    """知识库抽象基类"""

    # ── 文件生命周期管理 ──
    @abstractmethod
    async def upload_file(self, kb_id, file) -> str: ...
    @abstractmethod
    async def parse_file(self, file_id) -> None: ...       # UPLOADED → PARSING → PARSED
    @abstractmethod
    async def index_file(self, file_id) -> None: ...       # PARSED → INDEXING → INDEXED
    @abstractmethod
    async def delete_file(self, file_id) -> None: ...

    # ── 检索接口 ──
    @abstractmethod
    async def search(self, kb_id, query, top_k) -> list[Document]: ...
    @abstractmethod
    async def hybrid_search(self, kb_id, query, top_k) -> list[Document]: ...

    # ── 评估接口 ──
    @abstractmethod
    async def evaluate_retrieval(self, kb_id, test_queries) -> dict: ...
```

### 2.2 文件状态机

```
UPLOADED ──→ PARSING ──→ PARSED ──→ INDEXING ──→ INDEXED
   │            │           │            │            │
   │       解析失败时    索引失败时    可重新索引   最终状态
   │       可重试        可重试
   ▼            ▼           ▼            ▼
 ERROR       PARSE_ERROR  INDEX_ERROR   (不变)
```

**状态管理的关键**：每个状态转换都是**幂等的**—重复调用不会产生副作用。比如对已 INDEXED 的文件再次调用 `index_file()`，应该先清理旧索引，再重新构建。

### 2.3 知识库工厂

```python
# src/knowledge/factory.py

async def create_knowledge_base(backend: str = None) -> KnowledgeBase:
    backend = backend or os.getenv("RAG_BACKEND", "openviking")
    if backend == "openviking":
        return OpenVikingKnowledgeBase()
    # 可扩展其他后端...
```

**环境变量驱动**：通过 `RAG_BACKEND=openviking` 切换实现，支持后续接入其他向量数据库（如 Milvus、Weaviate、Qdrant 等）。

---

## 三、OpenViking — 核心向量检索引擎

### 3.1 什么是 OpenViking？

OpenViking 是一个开源的 RAG 后端，提供了：
- 向量存储与索引
- 混合检索（关键词 + 向量）
- 元数据过滤
- 内置评估能力

### 3.2 关键配置

```env
RAG_BACKEND=openviking
OPENVIKING_WORKSPACE=./saves/openviking
OPENVIKING_EMBEDDING_PROVIDER=openai        # 兼容 OpenAI API 的 Provider
OPENVIKING_EMBEDDING_API_BASE=https://api.siliconflow.cn/v1
OPENVIKING_EMBEDDING_MODEL=Pro/BAAI/bge-m3  # BGE-M3 模型
OPENVIKING_EMBEDDING_DIMENSION=1024          # 向量维度
```

### 3.3 为什么选 BGE-M3？

| 特性 | BGE-M3 | 其他 Embedding 模型 |
|------|--------|-------------------|
| 多语言支持 | 中英文均优 | 很多模型偏英文 |
| 维度 | 1024 | 768-4096 |
| 稠密+稀疏检索 | 同时支持 | 多数只支持稠密 |
| MTEB 中文榜 | 前列 | - |
| API 可用性 | SiliconFlow 免费 | - |

### 3.4 OpenViking 服务层

`src/services/openviking_service.py`（约 1200 行）封装了 OpenViking 的所有操作：

```python
# 知识库管理
async def create_knowledge_base(name, description, chunk_strategy) -> dict
async def list_knowledge_bases() -> list[dict]
async def delete_knowledge_base(kb_id) -> bool

# 文件操作
async def upload_files(kb_id, files) -> list[dict]
async def parse_and_index_file(kb_id, file_id) -> dict

# 检索
async def search_knowledge(kb_id, query, top_k=5) -> list[dict]
async def evaluate_retrieval(kb_id, test_queries) -> dict
```

---

## 四、分块策略 (Chunking)

### 4.1 策略概述

`src/knowledge/chunking/` 目录实现了多种分块策略：

| 策略 | 适用场景 | 分块逻辑 |
|------|---------|---------|
| **General** | 通用文档 | 固定大小 + 重叠，按段落边界分割 |
| **QA** | 问答对 | 按 Q: / A: 标记分割，保持问答配对 |
| **Book** | 书籍/长文 | 按章节标题分割，保持层级结构 |
| **Laws** | 法律/规范 | 按条款编号分割，保持法条完整性 |

### 4.2 General 分块（最常用）

```python
# 核心参数
chunk_size = 500          # 每块约500个 token
chunk_overlap = 50        # 相邻块重叠50个 token
separators = ["\n\n", "\n", "。", ".", " "]  # 优先在段落边界分割
```

**为什么需要重叠？**

```
块1: "...Transformer 模型的核心是自注意力机制。自注意力机制通过"
块2: "自注意力机制通过计算查询(Q)、键(K)、值(V)之间的点积..."
                         ↑ 重叠部分保证关键概念不被截断
```

### 4.3 QA 分块（面试场景的关键）

```python
# 问答格式
Q: 请解释 MySQL 的索引下推优化是什么？
A: 索引下推（Index Condition Pushdown, ICP）是 MySQL 5.6 引入的优化...

# 分块策略：保持 Q+A 不拆分
# 将整个 QA 对作为一个 chunk
```

> **为什么面试场景需要 QA 分块？** 面试题库通常是 QA 格式。如果按 General 策略分块，一个完整的 QA 对可能被拆到两个 chunk 中，导致检索时只返回半个答案。

---

## 五、Embedding 与 Rerank 模型集成

### 5.1 Embedding 模型封装

```python
# src/models/embed.py (~300行)

class EmbeddingModel:
    def __init__(self, provider, api_base, model_name, dimension):
        self.client = OpenAI(base_url=api_base)  # 兼容 OpenAI API
        self.model = model_name
        self.dimension = dimension

    async def embed_documents(self, texts: list[str]) -> list[list[float]]:
        """批量向量化"""
        response = await self.client.embeddings.create(
            model=self.model,
            input=texts,
            dimensions=self.dimension,  # 指定输出维度
        )
        return [d.embedding for d in response.data]

    async def embed_query(self, query: str) -> list[float]:
        """查询向量化（单条）"""
        ...
```

**设计分析**：统一的 Embedding 接口使得切换模型只需改配置，不需要改代码。

### 5.2 Rerank 模型封装

```python
# src/models/rerank.py (~180行)

class RerankModel:
    async def rerank(self, query: str, documents: list[str], top_n: int) -> list[dict]:
        """对检索结果重排序"""
        response = await self.client.rerank(
            model=self.model,
            query=query,
            documents=documents,
            top_n=top_n,
        )
        return sorted_results
```

**为什么要 Rerank？**

```
Embedding 检索 (粗排)          Rerank (精排)
┌──────────────────┐      ┌──────────────────┐
│ 召回 Top-50      │ ───→ │ 重排序 Top-5     │
│ 速度快，精度一般  │      │ 速度慢，精度高    │
│ BGE-M3 (1024维)  │      │ Cross-Encoder    │
└──────────────────┘      └──────────────────┘

两阶段检索：
1. Embedding 从全量中快速召回候选（粗排）
2. Rerank 对候选精细打分（精排）
3. 返回 Top-K 最相关结果
```

---

## 六、检索策略优化

### 6.1 混合检索

```python
# 融合两种检索信号
score_final = α × score_dense + (1-α) × score_sparse

# score_dense: BGE-M3 稠密向量相似度
# score_sparse: BM25 关键词匹配分数
# α: 融合权重（通常 0.7-0.8）
```

**为什么需要混合检索？** 纯向量检索可能漏掉精确的关键词匹配（如 API 名称、错误码），BM25 适合精确匹配，向量适合语义匹配，两者互补。

### 6.2 元数据过滤

```python
# 检索时可按元数据过滤
results = await kb.search(
    kb_id="java_backend",
    query="微服务架构设计",
    top_k=5,
    filters={
        "source": "JavaGuide",          # 来源过滤
        "difficulty": ["中级", "高级"],  # 难度过滤
    }
)
```

---

## 七、知识库评估体系

`src/services/evaluation_service.py`（约 900 行）实现了自动化的检索质量评估：

```python
# 评估维度
evaluation_metrics = {
    "hit_rate": "Top-K 中是否包含正确答案",
    "mrr": "正确答案的平均倒数排名",
    "ndcg": "考虑排序位置的归一化折损累积增益",
}

# 评估流程
1. 上传测试数据集（问题+标准答案+相关文档）
2. 自动执行检索
3. 计算各指标
4. 生成评估报告
```

> **实际价值**：不同的分块策略、Embedding 模型、检索参数组合会产生不同的检索效果。评估系统可以量化对比，帮助选择最优配置。

---

## 八、从零手搓 RAG 系统的关键步骤

### 最小可行 RAG 系统需要：

1. **文档解析器**（至少支持 PDF + Word）
2. **分块器**（至少支持 General 策略）
3. **Embedding 模型**（如 BGE-M3 via SiliconFlow API）
4. **向量数据库**（如 ChromaDB / OpenViking / Milvus Lite）
5. **检索接口**（search + hybrid_search）
6. **Rerank 模型**（可选，但推荐）

### 面试场景的 RAG 优化建议：

1. **QA 分块优先**：面试题库用 QA 策略分块，保证问答完整性
2. **元数据丰富**：记录题目来源、难度、岗位方向，支持精准过滤
3. **去重机制**：`excluded_questions` 参数避免重复抽题
4. **缓存热点**：高频面试题的结果缓存，减少 API 调用

---

## 下一步学习

阅读 [[05-document-parsing-pipeline|文档解析管线]]，深入理解 MinerU、PaddleX、DeepSeek OCR 和 RapidOCR 的集成与选择策略。
