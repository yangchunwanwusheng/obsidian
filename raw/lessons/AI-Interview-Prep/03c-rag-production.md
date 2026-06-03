---
type: lesson
tags: [面试, RAG, 生产实践, Graph-RAG, Agentic-RAG, 多模态RAG, 企业RAG]
created: 2026-05-20
updated: 2026-05-20
difficulty: advanced
prerequisites: [RAG基础, 检索优化, 向量数据库]
topic: RAG高级架构与生产实践
status: in-progress
series: {name: "AI Interview Prep", part: "3c"}
---

# RAG 高级架构与生产实践

> 面向 AI 工程师面试的 RAG 进阶：Graph RAG、Agentic RAG、多模态 RAG、企业级设计与性能优化的全景深度解析。

## 一、Graph RAG（知识图谱增强检索）

### 1.1 核心问题：传统 RAG 的结构性盲区

传统 Vector RAG 的检索逻辑是将文档分块后做向量相似度匹配。这种范式在回答局部性问题时表现优秀——"这个函数怎么用？""合同的第三条写了什么？"但面对全局性问题（global query）时会出现系统性失效：

```
传统 RAG 的全局性问题处理链路：

用户问题："这家公司所有产品线的共同趋势是什么？"
  │
  ▼
向量检索 top-k chunks ──→ 每个 chunk 只包含某产品的局部信息
  │
  ▼
LLM 基于局部 chunks 汇总 ──→ 缺乏跨文档的结构化关联
  │
  ▼
答案：以偏概全，无法真正做跨文档推理
```

根本原因：向量相似度衡量的是语义接近度，而非实体间的结构关系。要回答全局性问题，检索系统需要理解实体之间的关联拓扑，这正是知识图谱的天然优势。

> [!hint]- 面试题：传统 RAG 为什么无法回答"公司所有产品的共同趋势"这类问题？
>
> 请从检索粒度、语义相似度的局限性、以及信息聚合方式三个角度分析。

> [!success]- 参考答案
>
> **检索粒度问题**：传统 RAG 将文档切成固定大小的 chunk，每个 chunk 只包含某个产品或某个时间段的局部信息。全局性问题需要跨数百个 chunk 的信息聚合，top-k 检索根本无法覆盖。
>
> **语义相似度局限**：向量检索基于 embedding 余弦相似度，衡量的是"文本内容相近"。但"产品 A 在增长"和"产品 B 也在增长"这两段文本在向量空间中并不一定接近，因为它们讨论的具体产品完全不同。
>
> **信息聚合缺失**：即使检索到了所有相关 chunk，LLM 的上下文窗口也塞不下数百个 chunk。传统 RAG 缺少一种"先总结再聚合"的多层次信息压缩机制。

### 1.2 微软 GraphRAG 完整架构

微软在 2024 年开源的 GraphRAG 是目前最具代表性的 Graph RAG 实现。其核心思想是将非结构化文档转化为结构化的知识图谱，然后利用图上的社区结构进行多层次摘要。

```
GraphRAG 完整处理管线：

┌─────────────┐
│  原始文档集   │
└──────┬──────┘
       │
       ▼
┌──────────────────────────────────────┐
│ Phase 1: 文本分块 + 实体/关系抽取     │
│                                      │
│  对每个 chunk 调用 LLM：              │
│  - Named Entity Recognition (NER)    │
│  - 关系抽取 (Relation Extraction)    │
│  - 实体消歧 (Entity Disambiguation)  │
│  输出：(entity1, relation, entity2)   │
└──────┬───────────────────────────────┘
       │
       ▼
┌──────────────────────────────────────┐
│ Phase 2: 知识图谱构建                 │
│                                      │
│  - 合并所有 chunk 的三元组            │
│  - 实体共指消解（"苹果"/"Apple Inc."）│
│  - 构建无向加权图                     │
│  - 节点 = 实体，边 = 关系             │
└──────┬───────────────────────────────┘
       │
       ▼
┌──────────────────────────────────────┐
│ Phase 3: 社区检测 + 层次摘要          │
│                                      │
│  - Leiden 算法做社区检测              │
│  - 每个社区生成 LLM 摘要              │
│  - 多层级社区（从叶子到根）           │
│  - 形成层次化摘要树                   │
└──────┬───────────────────────────────┘
       │
       ▼
┌──────────────────────────────────────┐
│ Phase 4: 查询时检索                  │
│                                      │
│  Local Query:  检索相关实体的子图     │
│  Global Query: 遍历社区摘要做聚合     │
│  Hybrid Query: 结合两者               │
└──────────────────────────────────────┘
```

**关键设计决策解析**：

**为什么用 Leiden 算法而不是简单的聚类？** Leiden 算法能发现图中的层次化社区结构，这正是 GraphRAG 需要的——不同粒度的社区对应不同抽象层级的摘要。顶层社区摘要覆盖全局趋势，底层社区摘要保留细节。

**实体消歧为什么是最大挑战？** 同一实体在不同文档中可能有不同称谓（"特斯拉"/"Tesla"/"TSLA"）。如果消歧做不好，图谱会出现冗余节点，社区检测和后续推理都会受影响。实际工程中通常用 LLM 做第一轮消歧，再用字符串相似度 + embedding 相似度做二次确认。

### 1.3 Graph RAG vs Vector RAG 对比

| 维度 | Vector RAG | Graph RAG |
|------|-----------|-----------|
| **索引结构** | 向量索引 (HNSW/IVF) | 知识图谱 + 向量索引 |
| **检索逻辑** | 语义相似度 top-k | 子图遍历 + 社区摘要 |
| **擅长问题** | 局部性、事实性问题 | 全局性、跨文档推理 |
| **建索引成本** | 低（embedding 计算） | 高（LLM 抽取实体和关系） |
| **查询延迟** | 低（毫秒级 ANN 搜索） | 中等（图遍历 + LLM 摘要） |
| **更新难度** | 增量更新容易 | 图的增量更新复杂 |
| **存储开销** | 向量存储 | 图存储 + 向量存储 + 摘要 |

**选型原则**：如果 80% 的问题是局部性的事实查询，用 Vector RAG；如果大量问题需要跨文档推理、全局趋势分析、多跳关联，投入 Graph RAG 的成本是值得的。生产中常见做法是两者并存，根据问题类型路由。

### 1.4 用 LangChain + Neo4j 构建 Graph RAG

```python
from langchain_community.graphs import Neo4jGraph
from langchain_community.vectorstores import Neo4jVector
from langchain_openai import OpenAIEmbeddings, ChatOpenAI
from langchain.text_splitter import RecursiveCharacterTextSplitter

# ── Step 1: 连接 Neo4j ──
graph = Neo4jGraph(
    url="bolt://localhost:7687",
    username="neo4j",
    password="your_password"
)

llm = ChatOpenAI(temperature=0, model="gpt-4o")
embeddings = OpenAIEmbeddings()

# ── Step 2: 文档分块 + 实体/关系抽取 ──
text_splitter = RecursiveCharacterTextSplitter(
    chunk_size=1000, chunk_overlap=200
)

from langchain_community.graphs.graph_document import (
    GraphDocument, Node, Relationship
)

def extract_graph_documents(chunk_text: str) -> GraphDocument:
    """用 LLM 从文本中抽取实体和关系"""
    prompt = f"""
    从以下文本中提取所有实体和关系。
    输出格式为 JSON：
    {{
        "nodes": [{{"id": "实体名", "type": "Person|Organization|Product|..."}}] ,
        "relationships": [
            {{"source": "实体A", "target": "实体B", "type": "关系类型"}}
        ]
    }}

    文本：{chunk_text}
    """
    result = llm.invoke(prompt)
    parsed = json.loads(result.content)

    nodes = [
        Node(id=n["id"], type=n["type"])
        for n in parsed["nodes"]
    ]
    relationships = [
        Relationship(
            source=Node(id=r["source"], type=""),
            target=Node(id=r["target"], type=""),
            type=r["type"]
        )
        for r in parsed["relationships"]
    ]
    return GraphDocument(nodes=nodes, relationships=relationships)


# ── Step 3: 构建知识图谱 ──
def build_knowledge_graph(documents):
    """处理文档集，构建完整的知识图谱"""
    all_graph_docs = []
    for doc in documents:
        chunks = text_splitter.split_text(doc.page_content)
        for chunk in chunks:
            graph_doc = extract_graph_documents(chunk)
            all_graph_docs.append(graph_doc)

    # 批量写入 Neo4j
    graph.add_graph_documents(
        all_graph_docs,
        baseEntityLabel=True,  # 为所有实体添加基础标签
        include_source=True    # 保留来源文档信息
    )
    return all_graph_docs

# ── Step 4: 创建混合检索器（向量 + 图） ──
vector_index = Neo4jVector.from_existing_graph(
    embedding=embeddings,
    url="bolt://localhost:7687",
    username="neo4j",
    password="your_password",
    index_name="entity_embeddings",
    node_label="Entity",
    text_node_properties=["id", "type"],
    embedding_node_property="embedding"
)

# ── Step 5: 查询时同时做向量检索和图遍历 ──
def hybrid_query(question: str):
    # 向量检索找相关实体
    vector_results = vector_index.similarity_search(question, k=5)

    # 用 Cypher 做图遍历，获取相关实体的子图
    cypher_query = """
    MATCH (e:Entity)-[r]->(related)
    WHERE e.id IN $entity_ids
    RETURN e.id, type(r) as relation, related.id, related.type
    LIMIT 50
    """
    entity_ids = [doc.metadata["id"] for doc in vector_results]
    graph_result = graph.query(cypher_query, params={"entity_ids": entity_ids})

    # 组装上下文
    context = format_graph_context(graph_result)
    return generate_answer(question, context)
```

> [!hint]- 面试题：Graph RAG 的建索引成本远高于 Vector RAG，在什么场景下这个投入是值得的？如何评估是否需要 Graph RAG？

> [!success]- 参考答案
>
> **值得投入的场景**：
> 1. 用户的查询大量涉及全局性分析（"所有产品的共同趋势"、"整个行业的竞争格局"）
> 2. 文档集内部有大量实体交叉引用（法律文书、企业文档、学术论文网络）
> 3. 需要多跳推理（"A 公司的子公司和 B 公司的合资企业有哪些"）
>
> **评估方法**：先用 100-200 条真实用户 query 做分类——统计局部性查询和全局性查询的比例。如果全局性查询占比超过 20%，或者这些查询恰好是最关键的查询（如高管决策类），则值得投入。可以用 NIAH (Needle In A Haystack) 测试集做初步评估。

---

## 二、Agentic RAG

### 2.1 从被动检索到主动检索

传统 RAG 的检索是"一次性"的：用户提问 → 检索 → 生成。如果检索结果不好，系统没有自我修正的机会。Agentic RAG 的核心思想是让 LLM（作为 Agent）自主决定检索策略、评估检索质量、甚至在必要时纠正和重试。

```
传统 RAG vs Agentic RAG 架构对比：

【传统 RAG】
Query ──→ Retrieve ──→ Generate ──→ Answer
（一条直线，没有反馈环）

【Agentic RAG】
                    ┌──────────────────┐
                    │                  │
                    ▼                  │
Query ──→ Router ──→ Retrieve ──→ Evaluate ──┤
              │           │          │        │
              │           │          ▼        │
              │           │     质量OK？      │
              │           │      │    │       │
              │           │     Yes    No ────┘
              │           │      │  (重新检索)
              │           ▼      │
              │        Generate ◄┘
              │           │
              ▼           ▼
         (不需要检索)  Answer
              │
              ▼
         Direct Answer
```

### 2.2 Router Agent

Router Agent 是 Agentic RAG 的入口，负责分析用户 query 的类型，决定走哪条检索路径。

```python
from enum import Enum
from typing import Literal

class QueryType(Enum):
    FACTUAL = "factual"           # 事实查询 → 向量检索
    ANALYTICAL = "analytical"     # 分析查询 → Graph RAG
    PROCEDURAL = "procedural"     # 流程查询 → 知识库检索
    CASUAL = "casual"             # 闲聊 → 无需检索
    CODE = "code"                 # 代码查询 → 代码搜索

def route_query(query: str) -> dict:
    """Router Agent：分析查询类型并路由"""
    prompt = f"""
    分析以下用户查询，判断其类型和最佳检索策略：

    查询：{query}

    返回 JSON：
    {{
        "query_type": "factual|analytical|procedural|casual|code",
        "needs_retrieval": true/false,
        "retriever": "vector|graph|keyword|hybrid|none",
        "reason": "路由原因"
    }}
    """
    result = llm.invoke(prompt)
    return json.loads(result.content)

# 使用示例
query = "我们公司所有产品线的用户增长趋势如何"
route = route_query(query)
# 输出：{"query_type": "analytical", "needs_retrieval": true,
#        "retriever": "graph", "reason": "需要跨产品线全局分析"}
```

### 2.3 Corrective RAG (CRAG)

CRAG 的核心是引入检索质量评估器。不是所有检索结果都值得送给 LLM，如果检索结果和问题不相关，CRAG 会触发纠正——要么重写查询重新检索，要么直接拒绝回答。

```python
def corrective_rag(query: str, max_retries: int = 2):
    """CRAG：检索-评估-纠正循环"""
    current_query = query

    for attempt in range(max_retries + 1):
        # Step 1: 检索
        docs = vector_store.similarity_search(current_query, k=10)

        # Step 2: 评估检索质量
        quality_score = evaluate_retrieval_quality(query, docs)

        if quality_score >= 0.7:
            # 检索质量合格，直接生成
            return generate_with_context(query, docs)

        elif attempt < max_retries:
            # 质量不足，重写查询后重试
            current_query = rewrite_query(query, docs, quality_score)

        else:
            # 重试用尽，使用 web search 兜底
            web_results = web_search(query)
            return generate_with_context(query, web_results)

def evaluate_retrieval_quality(query: str, docs: list) -> float:
    """评估检索文档与查询的相关性"""
    prompt = f"""
    评估以下检索结果与用户查询的相关性。
    返回 0-1 之间的分数。

    查询：{query}

    检索到的文档摘要：
    {[doc.page_content[:200] for doc in docs]}
    """
    result = llm.invoke(prompt)
    return float(result.content)

def rewrite_query(original: str, failed_docs, score: float) -> str:
    """基于失败原因重写查询"""
    prompt = f"""
    原始查询"{original}"的检索结果质量不佳（分数 {score}）。
    检索到的内容偏向：{[d.page_content[:100] for d in failed_docs[:3]]}

    请重写查询，使其更精确地匹配目标信息。
    """
    return llm.invoke(prompt).content
```

### 2.4 Self-RAG

Self-RAG 更进一步——模型自己决定三个关键问题：是否需要检索、检索结果是否相关、生成内容是否自洽。这三个决策通过特殊的 reflection token 实现。

```
Self-RAG 决策流：

Query ──→ [Need Retrieval?]
              │        │
             Yes       No
              │        │
              ▼        ▼
          Retrieve   Generate
              │     from parametric
              ▼        knowledge
     [Relevant?]
       │      │
      Yes     No
       │      │
       ▼      ▼
    Generate  Discard &
       │      Retry
       ▼
  [Is Supported?]
    │        │
   Yes       No
    │        │
    ▼        ▼
  Output   Regenerate
```

```python
from dataclasses import dataclass

@dataclass
class SelfRAGDecision:
    need_retrieval: bool
    is_relevant: bool | None = None
    is_supported: bool | None = None

def self_rag_generate(query: str):
    """Self-RAG：模型自主决策检索和验证"""
    # Step 1: 判断是否需要检索
    decision = judge_retrieval_need(query)

    if not decision.need_retrieval:
        # 模型认为自己能回答（如常识问题）
        answer = llm.invoke(query).content
        return answer

    # Step 2: 检索
    docs = retriever.invoke(query)

    # Step 3: 判断检索结果相关性
    decision.is_relevant = judge_relevance(query, docs)
    if not decision.is_relevant:
        return "抱歉，我没有找到足够相关的信息来回答这个问题。"

    # Step 4: 生成 + 自验证
    answer = generate_with_citations(query, docs)

    # Step 5: 验证生成内容是否被文档支持
    decision.is_supported = verify_answer_support(answer, docs)
    if not decision.is_supported:
        # 重新生成，增加"必须基于文档"的约束
        answer = generate_strict(query, docs)

    return answer
```

> [!hint]- 面试题：请设计一个系统，让 LLM 自动判断"这个问题要不要检索"。具体来说，什么类型的查询不需要检索？判断的依据是什么？

> [!success]- 参考答案
>
> **不需要检索的查询类型**：
> 1. **常识性问题**："什么是 Python 的 list comprehension？"——模型参数知识已覆盖
> 2. **创造性任务**："帮我写一首关于春天的诗"——不需要外部知识
> 3. **闲聊**："你好"、"今天天气怎么样"——无需知识库
> 4. **简单推理**："1+1 等于几？"——纯逻辑推理
>
> **判断依据（Self-RAG 的实现方式）**：
> - 在训练时加入特殊 token `[Retrieve]` / `[No Retrieve]`，让模型学会输出是否需要检索的标记
> - 工程实现中可以用分类器做快速判断：用一个轻量模型（如 BERT）fine-tune 一个二分类器，输入 query 输出 need_retrieval 的概率
> - 启发式规则辅助：查询长度过短（< 5 个词）、包含特定模式（闲聊意图）、query 的 embedding 与知识库所有 centroid 的相似度都很低，这些情况下可以跳过检索

---

## 三、多模态 RAG

### 3.1 多模态 RAG 的核心挑战

企业中的真实文档很少是纯文本——PDF 中混合了文字、表格、图片、图表。传统 RAG 只能处理文本，丢失了大量的视觉信息。多模态 RAG 要解决的核心问题是：如何统一索引和检索不同模态的信息。

```
多模态 RAG 的三条技术路线：

路线 A：全部转文本
┌──────────┐     OCR/API     ┌──────────┐
│ PDF/图片  │ ──────────────→ │ 纯文本    │ ──→ 传统 RAG
│ 表格/图表 │  (信息有损)     │          │
└──────────┘                 └──────────┘

路线 B：多模态 Embedding
┌──────────┐                 ┌──────────┐
│ PDF/图片  │ ──→ CLIP/VLM ──→│ 多模态    │ ──→ 向量检索
│ 表格/图表 │   embedding     │ 向量空间  │
└──────────┘                 └──────────┘

路线 C：VLM 端到端 (ColPali)
┌──────────┐                 ┌──────────┐
│ 文档页面  │ ──→ VLM 直接 ──→│ 页面级    │ ──→ MaxSim 检索
│ (图片形式) │   编码整页      │ embedding │
└──────────┘                 └──────────┘
```

### 3.2 图像检索：CLIP Embedding

对于文档中的图片（产品图、架构图、截图等），可以用 CLIP 或更强大的 VLM（如 SigLIP）做 image embedding，将图片和文本映射到同一个向量空间。

```python
import torch
from transformers import CLIPModel, CLIPProcessor

model = CLIPModel.from_pretrained("openai/clip-vit-large-patch14")
processor = CLIPProcessor.from_pretrained("openai/clip-vit-large-patch14")

def index_images_with_clip(images: list, metadata: list):
    """用 CLIP 将图片编码为向量并存入向量库"""
    embeddings = []
    for img in images:
        inputs = processor(images=img, return_tensors="pt")
        with torch.no_grad():
            img_emb = model.get_image_features(**inputs)
            img_emb = img_emb / img_emb.norm(dim=-1, keepdim=True)
        embeddings.append(img_emb.squeeze().numpy())

    # 存入多模态向量库
    vector_store.add_embeddings(
        embeddings=embeddings,
        metadatas=metadata,
        ids=[f"img_{i}" for i in range(len(images))]
    )

def multimodal_query(query_text: str, top_k: int = 5):
    """用文本 query 检索图片"""
    inputs = processor(text=query_text, return_tensors="pt")
    with torch.no_grad():
        text_emb = model.get_text_features(**inputs)
        text_emb = text_emb / text_emb.norm(dim=-1, keepdim=True)

    # 在统一的向量空间中检索
    results = vector_store.similarity_search_by_vector(
        text_emb.squeeze().numpy(), k=top_k
    )
    return results
```

### 3.3 表格检索：Table QA 的两种策略

表格是多模态 RAG 中最棘手的部分。两种主流策略各有优劣：

**策略一：LLM 直接读取表格**
- 将表格序列化为 Markdown/HTML 格式，直接塞入 LLM 上下文
- 适合：小型表格（< 100 行）、需要理解表格内容的场景
- 缺点：大表格会耗尽上下文窗口

**策略二：Text-to-SQL**
- 将用户问题翻译为 SQL 查询，在数据库上执行后返回结果
- 适合：结构化的大型表格（数据库中的数据）
- 缺点：需要 schema 定义、复杂查询翻译准确率有限

```python
def table_qa(query: str, table_content: str, strategy: str = "auto"):
    """统一表格问答接口"""

    if strategy == "auto":
        # 自动选择策略
        row_count = table_content.count("\n") - 1
        strategy = "direct" if row_count < 50 else "sql"

    if strategy == "direct":
        # 策略一：LLM 直接读取
        prompt = f"""
        基于以下表格数据回答问题。
        表格：
        {table_content}

        问题：{query}
        """
        return llm.invoke(prompt).content

    elif strategy == "sql":
        # 策略二：Text-to-SQL
        sql = text_to_sql(query, table_schema)
        results = execute_sql(sql)
        return format_results(query, results)
```

### 3.4 ColPali：用 VLM 直接编码文档页面

ColPali（2024 年提出）是目前多模态 RAG 领域最具创新性的方法。其核心思想是跳过 OCR 和版面分析，直接用 VLM 对文档页面图片做 embedding，然后用 MaxSim 相似度做检索。

```
传统管线 vs ColPali：

【传统】                         【ColPali】
PDF → OCR → 文本提取             PDF → 页面截图
  → 版面分析 → 分块                    │
    → Embedding → 索引                 ▼
                                   VLM 编码 → 页面 embedding → 索引

优势：跳过所有中间环节，保留完整的视觉信息
劣势：每页的 embedding 维度很高（如 128 tokens × 128 dim）
      需要更多存储空间和 MaxSim 检索算力
```

> [!hint]- 面试题：一份 200 页的 PDF 技术报告，其中包含大量复杂表格（跨行跨列合并单元格）和流程图。传统 RAG 的 OCR 经常解析出错。你会怎么设计检索方案？

> [!success]- 参考答案
>
> 1. **ColPali 路线**：将每页转为图片，用 ColPali/ColQwen2 做页面级 embedding。优势是完全跳过 OCR，保留视觉信息。检索时先定位到相关页面，再用 VLM 直接读取页面图片生成答案。这是目前对复杂排版文档最鲁棒的方案。
>
> 2. **混合路线（生产推荐）**：
>    - 文本页面：正常 OCR + 分块 + embedding（成本低）
>    - 表格/图表页面：用页面截图 + VLM embedding 做视觉索引
>    - 查询时：文本 query 同时在两个索引中检索，合并结果
>
> 3. **表格特化处理**：如果表格数量可控，可以对每个表格单独提取，用 LLM 做结构化理解后存储为 JSON，再建立结构化查询接口（Text-to-SQL）。

---

## 四、企业级 RAG 系统设计

### 4.1 权限控制

企业环境中，不同用户能看到不同文档是刚性需求。权限控制的三种实现方案各有取舍：

**方案一：Metadata Filter（推荐起步方案）**

在文档 embedding 时打上权限标签，检索时用 metadata filter 做过滤。

```python
# 索引时打标签
doc_metadata = {
    "doc_id": "doc_001",
    "departments": ["engineering", "product"],  # 允许的部门
    "access_level": "confidential",             # 机密等级
    "owner": "team_alpha"
}
vector_store.add_documents([doc], metadatas=[doc_metadata])

# 检索时过滤
results = vector_store.similarity_search(
    query="API 设计规范",
    k=10,
    filter={
        "departments": {"$in": user_departments},
        "access_level": {"$in": accessible_levels}
    }
)
```

**方案二：多租户向量数据库**

每个租户（部门/团队）维护独立的向量空间，物理隔离。

```
方案比较：

Metadata Filter：
  优点：实现简单，单索引管理
  缺点：先检索后过滤可能导致召回不足
  适用：租户数量多、文档交叉访问频繁

多租户隔离：
  优点：强隔离，检索性能不受其他租户影响
  缺点：索引管理复杂，跨租户查询困难
  适用：数据隔离要求严格（如金融、医疗）
```

**方案三：检索前过滤 vs 检索后过滤**

这是一个关键的架构决策。检索前过滤（pre-filter）先缩小搜索范围再找 top-k，不会浪费检索配额但可能错过相关文档；检索后过滤（post-filter）先找全局 top-k 再过滤，召回更全但可能过滤后剩余太少。生产中推荐 pre-filter，并适当放大 k 值。

### 4.2 实时更新与增量索引

文档更新后索引如何同步是企业 RAG 的运维难题。

```
增量索引更新策略：

Document Store (Source of Truth)
       │
       │ 文档变更事件（CDC / Webhook）
       ▼
┌──────────────────┐
│ 变更检测器        │
│ - 内容 hash 对比  │
│ - 版本号追踪      │
│ - 增量/全量判断   │
└──────┬───────────┘
       │
       ▼
┌──────────────────┐
│ 索引更新器        │
│ - 删除旧 chunk    │
│ - 重新分块        │
│ - 重新 embedding  │
│ - 写入新索引      │
└──────┬───────────┘
       │
       ▼
┌──────────────────┐
│ 索引切换器        │
│ - 双写缓冲        │
│ - 原子切换        │
│ - 旧索引延迟删除  │
└──────────────────┘
```

**一致性问题的处理**：索引更新期间，读请求可能看到新旧混合的结果。解决方案是维护两个索引版本（蓝绿部署模式）——新文档写入新索引，更新完成后原子切换读指针。

### 4.3 缓存策略

缓存是企业 RAG 控制成本和延迟的关键手段。三层缓存各有不同的命中条件和失效策略：

```python
import hashlib
from functools import lru_cache

class RAGCacheSystem:
    def __init__(self):
        self.exact_cache = {}      # 精确匹配缓存
        self.semantic_cache = None  # 语义相似缓存
        self.answer_cache = {}     # 答案缓存

    def get_cached_answer(self, query: str) -> str | None:
        # Layer 1: 精确匹配
        cache_key = hashlib.md5(query.encode()).hexdigest()
        if cache_key in self.exact_cache:
            return self.exact_cache[cache_key]

        # Layer 2: 语义相似缓存
        query_emb = embed(query)
        similar = self.semantic_cache.search(query_emb, threshold=0.95)
        if similar:
            return similar.answer

        # Layer 3: 无缓存命中
        return None

    def cache_answer(self, query: str, answer: str):
        cache_key = hashlib.md5(query.encode()).hexdigest()
        self.exact_cache[cache_key] = answer
```

> [!hint]- 面试题：请设计一个支持 10 万用户的 RAG 系统，需要考虑权限控制、实时更新和查询性能。给出你的架构设计。

> [!success]- 参考答案
>
> **架构分层设计**：
>
> 1. **接入层**：API Gateway + 限流 + 认证鉴权（JWT/OAuth2）。用户请求先过鉴权，解析出用户身份和权限信息。
>
> 2. **路由层**：Query Router 根据问题类型分发到不同检索通道。简单事实查询走高速缓存；复杂分析走 Graph RAG。
>
> 3. **缓存层**：三级缓存（精确匹配 → 语义缓存 → 答案缓存）。热数据用 Redis，语义缓存用独立的向量索引。预期缓存命中率 60%+。
>
> 4. **检索层**：
>    - 向量索引：Milvus 集群（支持水平扩展），按部门分区
>    - 权限控制：Pre-filter 模式，metadata 包含 access_tags
>    - Graph 索引：Neo4j 企业版（可选，用于复杂分析查询）
>
> 5. **索引更新层**：CDC 监听文档变更，增量更新 embedding。双索引蓝绿切换保证更新期间的一致性。
>
> 6. **存储层**：文档原始文件存对象存储（S3/MinIO），向量数据存 Milvus，元数据存 PostgreSQL。
>
> **关键 SLA 指标**：P99 延迟 < 2s（缓存命中 < 200ms），可用性 99.9%，索引更新延迟 < 5min。

---

## 五、RAG 系统的性能优化

### 5.1 端到端延迟分析

一个典型的 RAG 请求经过以下阶段，每个阶段都有优化空间：

```
RAG 请求的延迟分解（典型值）：

[用户输入] ──→ [Query 理解] ──→ [Embedding] ──→ [检索] ──→ [重排] ──→ [上下文组装] ──→ [LLM 生成] ──→ [输出]
    0ms          50-100ms       20-50ms     10-50ms   50-200ms    10ms       500-3000ms     流式

优化优先级：LLM 生成 >> 重排 > Embedding > 检索 > Query 理解
```

### 5.2 各阶段优化策略

**索引优化**：
- 批量索引时用并发 embedding 计算（GPU 批处理）
- 大规模索引做分区（按时间/部门/类型），检索时只搜索相关分区
- ANN 算法参数调优：HNSW 的 ef_construction 和 M 参数权衡索引质量与查询速度

**查询优化**：
- 多路召回并行执行（向量 + 关键词 + 知识图谱同时检索）
- 异步 I/O：embedding 计算和检索用 async 并发
- Query 批处理：将多个相似 query 合并为一次检索

**生成优化**：
- 上下文压缩：用 LLM 或小型模型对检索到的文档做摘要后再喂给生成模型
- Prompt 精简：减少 system prompt 中的冗余指令
- 流式生成：首 token 延迟从 2-3 秒降到 200-500ms

**成本优化**：
- 缓存命中率提升：语义缓存 > 精确缓存，预期降低 40-60% 的 LLM 调用
- Token 控制：对检索到的文档做截断，只保留与 query 最相关的部分
- 模型路由：简单查询用 Haiku 级模型，复杂查询用旗舰模型

```python
import asyncio
from concurrent.futures import ThreadPoolExecutor

async def optimized_rag_query(query: str):
    """并行多路召回 + 异步生成"""
    # 并行执行多路检索
    with ThreadPoolExecutor(max_workers=3) as executor:
        vector_future = executor.submit(
            vector_store.similarity_search, query, k=20
        )
        keyword_future = executor.submit(
            keyword_store.search, query, k=20
        )
        graph_future = executor.submit(
            graph_store.traverse, query, depth=2
        )

        vector_docs = vector_future.result()
        keyword_docs = keyword_future.result()
        graph_docs = graph_future.result()

    # 融合排序
    all_docs = deduplicate(vector_docs + keyword_docs + graph_docs)
    reranked = reranker.rerank(query, all_docs, top_k=5)

    # 上下文压缩
    compressed = compress_context(query, reranked)

    # 流式生成
    return stream_generate(query, compressed)
```

> [!hint]- 面试题：线上 RAG 系统的 P99 延迟从 2 秒涨到了 8 秒，你如何排查和优化？

> [!success]- 参考答案
>
> **排查步骤**（按优先级）：
>
> 1. **定位慢阶段**：添加各阶段耗时埋点（Query 解析 / Embedding / 检索 / 重排 / LLM 生成），找到瓶颈在哪
> 2. **常见原因排查**：
>    - LLM 生成变慢？→ 检查 API 限流、prompt 长度是否增长
>    - 检索变慢？→ 检查索引大小是否突增、ANN 参数是否合理
>    - 重排变慢？→ 检索返回的候选数是否过多
> 3. **系统资源排查**：GPU/CPU 利用率、内存是否 OOM、网络延迟
>
> **优化手段**：
> - 如果是 LLM 生成慢：启用流式输出、减少上下文长度、启用语义缓存
> - 如果是检索慢：索引分区减少搜索空间、调低 HNSW ef_search 参数、增加检索节点
> - 如果是重排慢：减少送入重排器的候选数（先粗排再精排）

---

## 六、RAG vs Fine-tuning vs Prompt Engineering 决策框架

### 6.1 三种方法的对比矩阵

| 维度 | RAG | Fine-tuning | Prompt Engineering |
|------|-----|-------------|-------------------|
| **知识更新** | 实时（改数据库即可） | 需要重新训练 | 改 prompt 即可 |
| **知识范围** | 无限（受数据库大小） | 受限于训练数据 | 受限于模型参数 |
| **幻觉控制** | 强（有源文档支撑） | 中（可能产生幻觉） | 弱（依赖模型自律） |
| **初始成本** | 中（搭系统） | 高（训练算力 + 数据） | 低（写 prompt） |
| **运行成本** | 中（检索 + LLM 调用） | 低（无检索开销） | 最低 |
| **延迟** | 中高（检索+生成） | 低（直接推理） | 最低 |
| **可控性** | 高（控制知识来源） | 中（通过训练数据控制） | 低（prompt 限制） |
| **适用场景** | 知识密集型、需要引用 | 特定风格/格式/领域 | 快速原型、简单任务 |

### 6.2 决策流程

```
业务需求分析
      │
      ▼
知识是否频繁变化？
    │         │
   Yes        No
    │         │
    ▼         ▼
  用 RAG    是否需要特定风格/格式？
               │            │
              Yes           No
               │            │
               ▼            ▼
          Fine-tune     Prompt Engineering
          (+ RAG 可选)   (够用就先用这个)

组合策略：
- Fine-tune 风格 + RAG 知识 → 最佳效果
- RAG 做事实查询 + Prompt 做格式控制 → 性价比最高
```

> [!hint]- 面试题：业务方说"我们要让模型了解我们的产品手册，能准确回答客户关于产品的问题"。你推荐 RAG、Fine-tuning 还是 Prompt Engineering？为什么？

> [!success]- 参考答案
>
> **首选 RAG，理由如下**：
>
> 1. **产品手册知识需要频繁更新**：产品迭代后手册会更新，RAG 只需更新数据库，无需重新训练
> 2. **需要可追溯的答案**：客户问产品参数，需要引用手册原文，RAG 天然支持引用溯源
> 3. **幻觉控制是刚需**：产品问答场景中，错误信息比"不知道"更危险。RAG 基于文档生成，幻觉风险更低
>
> **但可以叠加其他方法**：
> - 如果产品问答有特定的回复风格要求（如客服语气），可以在 RAG 基础上 fine-tune 一个风格模型
> - Prompt Engineering 用于控制输出格式（如结构化回答、引用格式）
>
> **不推荐纯 Fine-tuning 的原因**：训练数据难以覆盖所有产品细节，更新成本高，且无法提供答案来源。在知识密集型任务中，fine-tuning 的效果不如 RAG。

### 6.3 业界最佳实践

**实践一：RAG 为基础 + Fine-tuning 做增强**
先用 RAG 解决知识获取问题，再在 RAG 生成的问答对上 fine-tune 一个小模型做"风格适配"。这样既有知识准确性，又有统一的输出风格。

**实践二：分级响应策略**
- Level 1：Prompt Engineering（简单问题，直接回答）
- Level 2：RAG（需要查找文档的问题）
- Level 3：Fine-tuned Model + RAG（需要领域专业性的问题）
- 根据问题复杂度自动路由到对应级别

**实践三：持续评估驱动**
不要静态选择方案——上线后持续收集数据，用 RAGAS 等框架评估各环节效果。如果发现特定类型的问题 RAG 效果差，考虑针对这些场景做 fine-tuning。

---

## 关键要点回顾

1. **Graph RAG 解决全局性问题**：当查询需要跨文档推理、多跳关联时，知识图谱的结构化检索远优于向量相似度。但建索引成本高，需要根据查询分布决定是否投入。
2. **Agentic RAG 引入反馈环**：Router/CRAG/Self-RAG 让系统从"检索一次就完"进化为"自主评估、纠正、重试"，大幅提升检索质量和答案可靠性。
3. **多模态 RAG 是企业刚需**：真实文档混合了文字、表格、图片。ColPali 的端到端视觉编码是最有前景的方向，但工程成本需要权衡。
4. **企业级设计关注非功能性需求**：权限控制、增量更新、缓存策略、性能优化这些"工程问题"才是决定 RAG 系统能否上线的关键。
5. **三种方法不是互斥的**：RAG + Fine-tuning + Prompt Engineering 的组合往往比单一方法效果更好。关键是根据业务场景做分级路由。

## 下一步学习

- [[raw/lessons/AI-Interview-Prep/03-rag|03 RAG 基础]]
- [[raw/lessons/AI-Interview-Prep/03a-rag-indexing|03a RAG 索引优化]]
- [[raw/lessons/AI-Interview-Prep/03b-rag-retrieval|03b RAG 检索优化]]
