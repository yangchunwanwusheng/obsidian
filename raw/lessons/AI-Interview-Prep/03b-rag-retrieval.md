---
type: lesson
tags: [面试, RAG, 检索优化, Rerank, Query改写, 混合检索]
created: 2026-05-20
updated: 2026-05-20
difficulty: advanced
prerequisites: [RAG基础, Embedding, 向量数据库]
topic: RAG检索优化深入
status: in-progress
series: {name: "AI Interview Prep", part: "3b"}
---

# RAG 检索优化深入

> 面试核心：检索质量决定 RAG 上限。掌握 Query 改写、混合检索、Rerank、高级策略和评估体系，能系统性地提升检索效果。

## 学习目标

- [ ] 能实现 HyDE、Step-back Prompting、Query Decomposition 三种改写策略
- [ ] 能推导 RRF 融合公式并实现混合检索
- [ ] 理解 Cross-Encoder 和 Bi-Encoder 的本质区别
- [ ] 能用 RAGAS 框架评估 RAG 系统质量
- [ ] 理解 Self-RAG、CRAG、Graph RAG 的架构差异
- [ ] 能说出 RAG 的三大固有局限性及应对方案

## 一、Query 改写深入

用户原始 Query 通常不适合直接检索：太短、太模糊、缺少关键词。Query 改写是检索优化的第一步。

### 1.1 Query Expansion（查询扩展）

用 LLM 生成多个语义等价的查询，扩大检索覆盖面：

```python
from openai import OpenAI
client = OpenAI()

def expand_query(original_query: str, n_variants: int = 3) -> list[str]:
    """用 LLM 生成多个等价查询"""
    prompt = f"""请为以下问题生成 {n_variants} 个不同表述的同义问题。
保持核心语义不变，但使用不同的关键词和表达方式。
每行一个，不要编号。

原始问题：{original_query}"""

    response = client.chat.completions.create(
        model="gpt-4o-mini",
        messages=[{"role": "user", "content": prompt}],
        temperature=0.7,  # 适度随机，保证多样性
    )
    variants = response.choices[0].message.content.strip().split('\n')
    return [original_query] + [v.strip() for v in variants if v.strip()]

# 示例
# 原始: "怎么报销差旅费"
# 扩展: ["出差费用怎么申请", "公司差旅报销流程", "travel expense reimbursement policy"]
```

检索时对每个变体分别检索，然后合并结果并去重。**适用场景**：用户查询表达不规范、多语言混合、专业术语和非专业术语交替使用的情况。

### 1.2 HyDE（Hypothetical Document Embedding）

HyDE 的核心思想：先让 LLM 生成一个"假想的答案文档"，然后用这个假想文档的 Embedding 去检索。

```
用户 Query: "什么是注意力机制？"
         ↓
    LLM 生成假想答案:
    "注意力机制是 Transformer 模型的核心组件，
     通过 Q/K/V 矩阵计算注意力权重，
     实现对输入序列不同位置的自适应关注..."
         ↓
    Embedding(假想答案) → 向量检索
         ↓
    命中真实文档（因为假想答案和真实文档在语义空间更接近）
```

```python
def hyde_retrieve(query: str, top_k: int = 5) -> list[dict]:
    """HyDE 检索完整流程"""
    # Step 1: LLM 生成假想文档
    prompt = f"""请写一段详细的回答，回答以下问题。
即使你不确定答案，也请给出你认为最合理的解释。

问题：{query}"""

    response = client.chat.completions.create(
        model="gpt-4o-mini",
        messages=[{"role": "user", "content": prompt}],
    )
    hypothetical_doc = response.choices[0].message.content

    # Step 2: 用假想文档的 Embedding 检索
    hyde_embedding = embedding_model.encode(hypothetical_doc)

    # Step 3: 向量检索
    results = vector_db.search(
        embedding=hyde_embedding,
        top_k=top_k,
    )
    return results
```

**为什么有效**：原始 Query 通常很短（5-15 个字），Embedding 信息量少。假想文档更接近真实文档的长度和表达方式，在向量空间中与真实文档的距离更近。

**风险**：如果 LLM 生成的假想答案方向完全错误，可能检索到不相关文档。适用场景：事实型问题（"X 是什么"），不适用观点型问题。

### 1.3 Step-back Prompting

先问一个更抽象、更广泛的问题获取背景知识，再用原始问题检索具体答案。

```
原始问题: "Python 3.12 的 match-case 语句怎么处理嵌套结构？"
         ↓ Step-back
抽象问题: "Python 的模式匹配机制的设计原理和使用场景是什么？"
         ↓ 检索背景知识
获取: 模式匹配的设计理念、语法规范、与 switch-case 的区别
         ↓ 再用原始问题检索
获取: match-case 嵌套的具体代码示例和注意事项
         ↓ 合并两轮检索结果
完整的上下文 = 背景知识 + 具体答案
```

```python
def stepback_prompting(query: str) -> tuple[str, str]:
    """生成 Step-back 抽象问题"""
    prompt = f"""你是一个知识抽象专家。给定一个具体问题，
请生成一个更广泛、更抽象的背景问题，帮助获取必要的背景知识。

具体问题：{query}
抽象问题："""

    response = client.chat.completions.create(
        model="gpt-4o-mini",
        messages=[{"role": "user", "content": prompt}],
        temperature=0.3,
    )
    stepback_query = response.choices[0].message.content.strip()
    return stepback_query, query  # 返回抽象问题和原始问题
```

### 1.4 Query Decomposition（查询分解）

将复杂的多条件问题拆成多个子问题，分别检索后合并。

```python
def decompose_query(complex_query: str) -> list[str]:
    """将复杂问题拆解为子问题"""
    prompt = f"""将以下复杂问题拆解为 2-4 个独立的子问题。
每个子问题应该可以用单一信息源回答。
每行一个子问题，不要编号。

复杂问题：{complex_query}"""

    response = client.chat.completions.create(
        model="gpt-4o-mini",
        messages=[{"role": "user", "content": prompt}],
    )
    sub_queries = response.choices[0].message.content.strip().split('\n')
    return [q.strip() for q in sub_queries if q.strip()]

# 示例
# 输入: "比较 Transformer 和 LSTM 在长序列建模上的优劣，以及各自的计算复杂度"
# 输出: [
#   "Transformer 如何处理长序列？有什么优势和劣势？",
#   "LSTM 如何处理长序列？有什么优势和劣势？",
#   "Transformer 的计算复杂度是多少？",
#   "LSTM 的计算复杂度是多少？"
# ]
```

> [!hint]- 面试场景：用户问了一个模糊的问题，如何改进检索？
> 思考：先判断是哪种"模糊"——太短？多义？多条件？然后选择对应的改写策略。

> [!success]- 参考答案
> 分三步走：
> 1. **诊断模糊类型**：
>    - 太短/关键词不足 → Query Expansion 或 HyDE
>    - 多义/歧义 → Query Expansion + 并行检索 + Rerank
>    - 多条件复合 → Query Decomposition
>    - 超出知识范围 → Step-back 获取背景知识
> 2. **组合策略**：实际场景中通常组合使用。例如先 Decomposition 拆子问题，每个子问题用 HyDE 增强，最后 RRF 融合结果
> 3. **效果验证**：每种改写策略都应该用标注数据集做 A/B 测试，不要拍脑袋选择

## 二、混合检索深入

### 2.1 稠密检索 vs 稀疏检索的互补性

```
场景 1：用户问"公司的差旅报销标准是什么"
  稀疏检索(BM25)：命中包含"差旅"、"报销"、"标准"关键词的文档 ✅
  稠密检索(Embedding)：可能命中语义相近但措辞不同的文档 ✅

场景 2：用户问"出差花了很多钱怎么弄"
  稀疏检索(BM25)：没有命中任何关键词 ❌ (没有"差旅"、"报销")
  稠密检索(Embedding)：理解语义，命中差旅报销文档 ✅

场景 3：用户问"RAG 中的 BM25 参数 k1 怎么调"
  稀疏检索(BM25)：精确匹配"BM25"、"k1"、"参数" ✅
  稠密检索(Embedding)：可能语义泛化过度，命中无关内容 ❌

结论：两者互补，稀疏擅长精确匹配，稠密擅长语义理解
```

### 2.2 RRF（Reciprocal Rank Fusion）公式与实现

RRF 是最常用的混合检索结果融合算法：

$$\text{RRF}(d) = \sum_{r \in R} \frac{1}{k + \text{rank}_r(d)}$$

公式解读：对每个文档 $d$，它在每个检索器 $r$ 中的排名为 $\text{rank}_r(d)$。该文档的最终得分是它在所有检索器中排名的倒数之和。参数 $k$（通常取 60）用于平滑，防止排名第一的文档权重过大。

直觉理解：一个文档在多个检索器中都排名靠前，那它大概率是相关的。RRF 不依赖原始分数，只依赖排名，因此不受不同检索器分数尺度的影响。

```python
def reciprocal_rank_fusion(
    result_lists: list[list[dict]],
    k: int = 60,
    top_n: int = 10,
) -> list[dict]:
    """RRF 融合多个检索器的结果"""
    rrf_scores = {}  # doc_id -> 累积 RRF 分数

    for results in result_lists:
        for rank, doc in enumerate(results, start=1):
            doc_id = doc["id"]
            if doc_id not in rrf_scores:
                rrf_scores[doc_id] = {"score": 0.0, "doc": doc}
            rrf_scores[doc_id]["score"] += 1.0 / (k + rank)

    # 按 RRF 分数降序排列
    sorted_results = sorted(
        rrf_scores.values(), key=lambda x: x["score"], reverse=True
    )
    return [item["doc"] for item in sorted_results[:top_n]]


# 使用示例：BM25 + 稠密检索混合
bm25_results = bm25_search(query, top_k=20)
dense_results = dense_search(query_embedding, top_k=20)

fused = reciprocal_rank_fusion(
    [bm25_results, dense_results],
    k=60,
    top_n=10,
)
```

### 2.3 其他融合算法对比

| 算法 | 公式 | 特点 |
|------|------|------|
| RRF | $\sum \frac{1}{k + \text{rank}}$ | 不依赖原始分数，对分数尺度不敏感 |
| CombSUM | $\sum \text{norm\_score}_r(d)$ | 需要分数归一化，简单直接 |
| CombMNZ | $\text{CombSUM}(d) \times \text{count}(d)$ | 在多个列表中同时出现的文档获得额外奖励 |
| Weighted Sum | $\sum w_r \cdot \text{score}_r(d)$ | 需要调权重，灵活但需要实验确定 |

实际经验：**RRF 在大多数场景下表现最稳定**，无需调权重。CombMNZ 在文档被多个检索器同时命中的场景有优势。Weighted Sum 适合有充足标注数据做权重搜索的场景。

### 2.4 SPLADE：学习型稀疏检索

SPLADE 将稠密模型的语义理解能力注入稀疏表示，生成**词项级别的稀疏向量**：

```
传统 BM25 稀疏向量（基于精确匹配）:
  "机器学习" → 1.0, "算法" → 0.8, "深度" → 0

SPLADE 稀疏向量（基于语义扩展）:
  "机器学习" → 2.1, "算法" → 1.5, "深度" → 0.9,
  "神经网络" → 1.3, "AI" → 0.8, "模型训练" → 0.7
  ↑ 自动扩展了语义相关但原文未出现的词项
```

SPLADE 通过 MLM（Masked Language Model）头预测每个词项的重要性权重，并用稀疏正则化确保向量稀疏。优势：兼具稠密检索的语义理解能力和稀疏检索的可解释性，且可以直接用现有倒排索引存储。

> [!hint]- 面试场景：如何确定稠密检索和稀疏检索的权重比？
> 思考：有没有理论指导？实际怎么操作？

> [!success]- 参考答案
> 没有通用最优比例，必须根据数据集特点实验确定。实践方法：
> 1. **先跑基线**：分别只用稠密和只用稀疏跑一遍，记录各自动 Baseline 指标
> 2. **网格搜索**：权重比从 0:10 到 10:0，步长 1，用标注数据集评估
> 3. **经验参考**：大多数场景下稠密权重 0.6-0.7、稀疏权重 0.3-0.4 效果好
> 4. **特殊情况**：技术文档（大量专业术语、版本号）→ 稀疏权重提高到 0.5+；自然语言问答 → 稠密权重 0.7+
> 5. **如果用 RRF 做融合**：不需要手动调权重比，RRF 自动基于排名融合

## 三、Rerank 深入

### 3.1 Cross-Encoder vs Bi-Encoder

```
Bi-Encoder（双塔模型，用于初始检索）:
  Query → Encoder → Embedding_q ─┐
                                  ├→ cos(q, d)  ← 计算向量相似度
  Doc   → Encoder → Embedding_d ─┘

  特点：Query 和 Doc 分别编码，可以预计算 Doc Embedding
       检索速度 O(1)（近似最近邻搜索）
       但 Query 和 Doc 之间没有交互，精度有限

Cross-Encoder（交叉编码器，用于精排）:
  [CLS] Query [SEP] Doc [SEP] → Encoder → P(relevance)  ← 直接输出相关性分数

  特点：Query 和 Doc 拼接后一起编码，每个 Transformer 层都有交叉注意力
       精度高（因为能看到 Query-Doc 的细粒度交互）
       但无法预计算，每对 (Query, Doc) 都要过一遍模型，速度慢
```

数学上的区别：
- Bi-Encoder: $\text{sim}(q, d) = \cos(f(q), f(d))$，Query 和 Doc 的表示独立计算
- Cross-Encoder: $P(\text{rel}|q,d) = \sigma(g([q; d]))$，Query 和 Doc 拼接后联合计算

### 3.2 主流 Rerank 模型对比

| 模型 | 架构 | 中文支持 | 延迟 | 特点 |
|------|------|----------|------|------|
| bge-reranker-v2-m3 | Cross-Encoder | 原生 | 中等 | 开源首选，多语言 |
| bge-reranker-large | Cross-Encoder | 原生 | 中等 | 中文效果好 |
| Cohere Rerank | API | 良好 | 快 | API 调用，无需部署 |
| Jina Reranker v2 | Cross-Encoder | 良好 | 快 | 支持长文档 |
| ColBERTv2 | Late Interaction | 良好 | 中等 | 精度和速度的折中 |
| RankGPT | LLM | 良好 | 慢 | 用 GPT 做 permutation-based 重排 |

### 3.3 ColBERT：Late Interaction 折中方案

ColBERT 介于 Bi-Encoder 和 Cross-Encoder 之间：Query 和 Doc 分别编码为 token-level Embedding，然后在 token 级别做最大相似度交互。

$$\text{ColBERT}(q, d) = \sum_{i=1}^{|q|} \max_{j=1}^{|d|} \text{sim}(q_i, d_j)$$

公式解读：对 Query 中的每个 token $q_i$，找到 Doc 中与之最相似的 token $d_j$（取最大相似度），然后对所有 Query token 的最大相似度求和。

```
Query tokens:  [Q1]  [Q2]  [Q3]
                ↓     ↓     ↓
Doc tokens:   [D1]  [D2]  [D3]  [D4]  [D5]

相似度矩阵:
       D1   D2   D3   D4   D5
  Q1  0.8  0.3  0.1  0.2  0.4  → max = 0.8
  Q2  0.2  0.9  0.7  0.1  0.3  → max = 0.9
  Q3  0.1  0.2  0.8  0.6  0.3  → max = 0.8

ColBERT Score = 0.8 + 0.9 + 0.8 = 2.5
```

优势：比 Bi-Encoder 精度高（token 级交互），比 Cross-Encoder 速度快（Doc 表示可以预计算）。

### 3.4 Rerank 对效果的提升数据

基于公开基准测试的典型提升幅度：

| 指标 | 仅稠密检索 | + Rerank | 提升 |
|------|-----------|----------|------|
| NDCG@10 | 0.72 | 0.82 | +14% |
| Recall@100 | 0.85 | 0.91 | +7% |
| MRR | 0.68 | 0.79 | +16% |

**结论**：Rerank 几乎总是值得加的。增加的延迟（通常 50-200ms）换来 10-15% 的精度提升，在大多数场景下都是划算的交换。

### 3.5 集成 Rerank 到 RAG Pipeline

```python
from sentence_transformers import CrossEncoder

# 1. 初始检索（Bi-Encoder 向量检索）
initial_results = vector_db.search(query_embedding, top_k=50)

# 2. Rerank（Cross-Encoder 精排）
reranker = CrossEncoder('BAAI/bge-reranker-v2-m3')

pairs = [(query, doc["content"]) for doc in initial_results]
scores = reranker.predict(pairs)

# 3. 按 Rerank 分数重新排序
scored_results = list(zip(initial_results, scores))
scored_results.sort(key=lambda x: x[1], reverse=True)

# 4. 取 Top-K 送入 LLM
top_k_results = [doc for doc, score in scored_results[:5]]
```

## 四、高级检索策略

### 4.1 Self-RAG：让模型自己决定是否需要检索

```
         用户 Query
             │
        ┌────▼────┐
        │   LLM   │ ← 是否需要检索？
        └────┬────┘
             │
     ┌───────┴───────┐
     │ 不需要         │ 需要
     ▼               ▼
  直接生成      检索相关文档
     │               │
     │         ┌─────▼─────┐
     │         │ 文档相关？ │ ← 检索结果评估
     │         └─────┬─────┘
     │          是   │   否
     │           │   │
     │     ┌─────▼───▼─────┐
     │     │ 用文档生成答案  │
     │     └──────┬────────┘
     │            │
     │      ┌─────▼─────┐
     │      │ 答案有支撑？│ ← 生成结果评估
     │      └─────┬─────┘
     │       是   │   否 → 重新检索/生成
     │        │
     └────────┤
              ▼
          最终答案
```

Self-RAG 的核心：模型在三个关键节点自主决策——是否检索、检索结果是否相关、生成答案是否有支撑。通过插入特殊 token（如 `[Retrieve]`、`[IsRel]`、`[IsSup]`）让模型学会自我评估。

### 4.2 CRAG（Corrective RAG）：检索结果纠正

```
Query → 检索 → 检索评估器（轻量级 LLM/规则）
                     │
          ┌──────────┼──────────┐
          │ 正确      │ 不确定    │ 错误
          ▼          ▼          ▼
      直接使用    补充网络搜索  完全重定向
          │          │          │
          └──────────┴──────────┘
                     │
               纠正后的上下文
                     │
                  LLM 生成
```

CRAG 的关键：不是无条件信任检索结果，而是先评估质量，再决定如何使用。评估器可以是一个轻量级模型，判断检索文档与 Query 的相关度是否超过阈值。

### 4.3 Graph RAG：基于知识图谱的检索

微软的 GraphRAG 解决了传统 RAG 难以处理**全局性问题**（"这篇文档的主要观点是什么？"）的痛点。

```
原始文档 → LLM 提取实体和关系 → 构建知识图谱
                                        │
                              ┌─────────┴─────────┐
                              │                   │
                         局部查询              全局查询
                         (具体问题)           (总结性问题)
                              │                   │
                         子图检索            社区检测+汇总
                         返回相关三元组      返回社区摘要
```

核心流程：
1. **实体抽取**：用 LLM 从文档中抽取实体和关系
2. **社区检测**：用 Leiden 算法对图谱做社区划分
3. **社区摘要**：为每个社区生成 LLM 摘要
4. **查询路由**：具体问题走子图检索，全局问题走社区摘要

**适用场景**：需要回答跨文档、跨章节的全局性问题。不适合简单事实型问答（开销大于收益）。

### 4.4 Agentic RAG：用 Agent 编排检索

```
              ┌─────────────────┐
              │   Router Agent  │ ← 分析 Query 类型
              └────────┬────────┘
                       │
        ┌──────────────┼──────────────┐
        │              │              │
   ┌────▼────┐   ┌────▼────┐   ┌────▼────┐
   │ 工具检索 │   │ 知识库  │   │ 网络搜索│
   │ Agent   │   │ Agent   │   │ Agent   │
   └────┬────┘   └────┬────┘   └────┬────┘
        │              │              │
        └──────────────┼──────────────┘
                       │
                ┌──────▼──────┐
                │ Synthesis   │ ← 汇总多个 Agent 的结果
                │ Agent       │
                └──────┬──────┘
                       │
                  最终答案
```

Agentic RAG 的核心：不再是一条固定的 Pipeline，而是一个**动态编排系统**。Router Agent 根据问题类型决定调用哪些工具/Agent，支持多轮迭代、工具调用、结果验证。

> [!hint]- 面试场景：什么时候该用 Graph RAG，什么时候用传统 RAG？
> 思考：两种方法的成本和收益分别是什么？

> [!success]- 参考答案
> **传统 RAG 适合**：事实型问答、具体条款查询、单文档检索（"第三季度的营收是多少"）
> **Graph RAG 适合**：跨文档关联分析、全局总结、实体关系挖掘（"公司各产品线的竞争格局如何"）
> **决策因素**：
> 1. **问题类型**：局部事实 → 传统 RAG；全局分析 → Graph RAG
> 2. **数据规模**：Graph RAG 的图谱构建成本高（每个文档都要过 LLM 提取实体），适合静态文档库，不适合频繁更新的数据
> 3. **延迟要求**：Graph RAG 的社区检测和摘要生成耗时较长，实时性要求高的场景用传统 RAG
> 4. **实际建议**：先用传统 RAG 跑基线，如果发现全局性问题的回答质量差，再考虑引入 Graph RAG 作为补充

## 五、RAG 评估深入

### 5.1 RAGAS 框架核心指标

RAGAS（Retrieval Augmented Generation Assessment）是目前最主流的 RAG 评估框架，包含以下核心指标：

```
                    RAG 流程
    Query → Retriever → Context → Generator → Answer
      │         │           │          │          │
      │    Context      Context     Answer     Answer
      │    Precision    Relevancy   Faithful-  Relevancy
      │    & Recall                  ness
      │         │           │          │          │
      └─────────┴───────────┴──────────┴──────────┘
                          RAGAS 评估
```

**Faithfulness（忠实度）**：生成答案是否忠于检索到的上下文，没有"幻觉"。

$$\text{Faithfulness} = \frac{|\text{faithful claims}|}{|\text{total claims in answer}|}$$

计算方法：将答案拆分为多个原子声明（claims），用 LLM 逐个判断每个声明是否可以从上下文中推导出来。忠实度 = 可推导的声明数 / 总声明数。

**Context Relevancy（上下文相关性）**：检索到的上下文与问题的相关程度。

$$\text{Context Relevancy} = \frac{|\text{relevant sentences in context}|}{|\text{total sentences in context}|}$$

计算方法：从上下文中提取与问题相关的句子，相关句子占比即为相关性分数。

**Answer Relevancy（答案相关性）**：生成的答案与问题的相关程度。

计算方法：用 LLM 从答案反向生成可能的问题，计算生成问题与原始问题的 Embedding 相似度。相似度越高说明答案越切题。

### 5.2 自动评估 vs 人工评估

| 维度 | 自动评估（RAGAS） | 人工评估 |
|------|------------------|----------|
| 成本 | 低（API 调用费） | 高（标注员时薪） |
| 速度 | 快（分钟级） | 慢（天级） |
| 一致性 | 高（可复现） | 低（标注员差异） |
| 深度 | 浅（指标有限） | 深（可发现细微问题） |
| 适用场景 | 持续监控、A/B 测试 | 上线前质量把关 |

**最佳实践**：日常用 RAGAS 做自动评估，每月做一轮人工评估校准自动评估的准确性。

### 5.3 A/B 测试设计

```python
# RAG 系统 A/B 测试示例
from ragas import evaluate
from ragas.metrics import faithfulness, answer_relevancy, context_relevancy

# 准备测试数据集
test_data = {
    "question": ["Q1", "Q2", ..., "Q100"],
    "answer": [answer_A_1, answer_A_2, ...],    # 系统 A 的答案
    "contexts": [ctx_A_1, ctx_A_2, ...],         # 系统 A 检索的上下文
    "ground_truth": ["GT1", "GT2", ..., "GT100"],
}

# 评估
results_A = evaluate(
    dataset=test_data,
    metrics=[faithfulness, answer_relevancy, context_relevancy],
)

# 同样评估系统 B，然后对比
# 统计显著性检验：对每个指标做配对 t 检验
```

> [!hint]- 面试场景：如何持续监控线上 RAG 系统的质量？
> 思考：线上没有 ground truth 怎么做评估？

> [!success]- 参考答案
> 线上监控体系分三层：
> 1. **在线信号采集**（无需 ground truth）：
>    - 用户反馈（点赞/踩/重新提问）
>    - 用户行为（是否复制答案、是否追问、会话轮数）
>    - 检索阶段指标（检索命中的 top-1 相似度分数分布、Rerank 分数分布）
> 2. **离线定期评估**（需要标注数据）：
>    - 每周用 RAGAS 跑一次完整评估，对比上周基线
>    - 自动标注：用 GPT-4 生成 pseudo ground truth，成本低但有噪声
> 3. **Bad Case 追踪**：
>    - 收集用户不满意的案例（反馈差/追问>3轮）
>    - 每周人工分析 Top-20 Bad Case，分类归因（检索问题/生成问题/知识库缺失）
>    - 基于归因结果针对性优化

## 六、RAG 的局限性

### 6.1 Lost in the Middle 问题

研究表明，LLM 对上下文中间部分的信息利用效率显著低于开头和结尾。

```
上下文位置:  [开头]  ████████ [中间]  ░░░░░░░░ [结尾]  ████████
注意力分布:   高                低                  高
```

实验数据：在 20+ 段上下文的场景下，中间段落的利用率比开头和结尾低 30-50%。

**解决方案**：
- 减少送入 LLM 的上下文数量（Rerank 后只取 Top-3 而非 Top-10）
- 将最相关的文档放在开头和结尾位置
- 用 Long Context 模型（Gemini 1.5 Pro 支持 100 万 token）部分缓解

### 6.2 检索噪声问题

即使检索 Top-K 都是相关的，也包含不相关的段落。噪声越多，生成质量越差。

**缓解策略**：
- 提高检索精度（Rerank + 相似度阈值过滤）
- 上下文压缩（用 LLM 压缩检索到的内容，只保留与问题相关的部分）
- 使用 LLM 的 Context Citations 功能，让模型标注每个事实的来源

### 6.3 多跳推理（Multi-hop Reasoning）

"公司 A 收购的公司的 CEO 之前在哪工作？"需要两步检索：先找公司 A 收购了谁，再找那个公司的 CEO 背景信息。

**解决方案**：
- Iterative Retrieval：先检索第一跳，将结果作为上下文再检索第二跳
- Graph RAG：预先构建实体关系图谱，多跳推理转化为图遍历
- Agent 驱动：让 LLM Agent 自主规划多步检索流程

### 6.4 实时更新索引的一致性问题

文档更新后，向量索引的更新存在延迟。用户可能检索到旧版本的内容。

**解决方案**：
- 文档版本管理：每次更新生成新版本向量，标记旧版本为 deprecated
- 增量索引：只重新 Embedding 变更的文档，不全量重建
- 最终一致性：接受短暂延迟，用 TTL 机制定期清理过期索引

### 6.5 RAG + Fine-tuning 组合策略

RAG 和 Fine-tuning 不是互斥的，可以组合使用：

```
┌──────────────────────────────────────────────────┐
│                 组合策略选择                        │
├──────────────┬───────────────────────────────────┤
│ 场景          │ 推荐策略                            │
├──────────────┼───────────────────────────────────┤
│ 知识频繁更新  │ RAG 为主（实时检索）                  │
│ 特定输出风格  │ Fine-tune 为主（学习风格）             │
│ 专业知识+风格 │ RAG + Fine-tune                     │
│ 复杂推理能力  │ Fine-tune（训练推理模式）+ RAG（提供知识）│
│ 多语言适配    │ Fine-tune（语言能力）+ RAG（领域知识）  │
└──────────────┴───────────────────────────────────┘
```

**推荐路径**：先用 RAG 快速验证效果，如果 RAG 无法解决（输出风格不对、推理能力不足），再考虑 Fine-tuning。Fine-tuning 是最后的手段，成本高且需要持续维护。

## 关键要点回顾

1. **Query 改写**是检索优化的第一步，四种策略（Expansion/HyDE/Step-back/Decomposition）解决不同类型的查询问题
2. **混合检索**（稠密+稀疏）是工业界标配，RRF 是最稳定的融合算法
3. **Rerank 几乎总是值得加的**，10-15% 的精度提升换取 50-200ms 延迟是划算的
4. **Self-RAG / CRAG / Graph RAG** 不是替代传统 RAG，而是针对不同痛点的增强方案
5. **评估体系**是持续优化的基础，线上用信号监控 + 离线用 RAGAS + 定期人工校准
6. **RAG 有固有局限**（Lost in the Middle / 多跳推理 / 实时一致性），需要组合其他技术解决

## 扩展阅读

- [Self-RAG: Learning to Retrieve, Generate, and Critique through Self-Reflection](https://arxiv.org/abs/2310.11511)
- [Corrective RAG (CRAG)](https://arxiv.org/abs/2401.15884)
- [Microsoft GraphRAG](https://microsoft.github.io/graphrag/)
- [RAGAS: Automated Evaluation of RAG Systems](https://arxiv.org/abs/2309.15217)
- [ColBERT: Fast and Accurate Retrieval](https://arxiv.org/abs/2004.12832)
- [Lost in the Middle: How Language Models Use Long Contexts](https://arxiv.org/abs/2307.03172)

## 下一步学习

- [[raw/lessons/AI-Interview-Prep/04-memory|04 - AI Agent 记忆系统深入]]：短期记忆、长期记忆、记忆检索与更新策略
