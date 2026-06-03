---
type: lesson
tags: [面试, RAG, Embedding, 向量数据库, Chunk策略, 文档解析]
created: 2026-05-20
updated: 2026-05-20
difficulty: advanced
prerequisites: [LLM基础, 信息检索基础]
topic: RAG索引与文档处理深入
status: in-progress
series: {name: "AI Interview Prep", part: "3a"}
---

# RAG 索引与文档处理深入

> 面试核心：理解从原始文档到可检索向量的全链路，掌握每个环节的工程细节和选型依据。

## 学习目标

- [ ] 能说出 5 种 PDF 解析工具的优缺点和适用场景
- [ ] 能实现递归字符分块和语义分块
- [ ] 理解 BM25 和稠密检索的数学基础
- [ ] 能为中文企业知识库选择合适的 Embedding 模型
- [ ] 理解 HNSW / IVF / PQ 三种向量索引算法的原理
- [ ] 能设计 Chunk 策略的 A/B 实验方案

## 一、文档解析深入

### 1.1 PDF 解析的核心挑战

PDF 本质上是一个**页面描述语言**，它记录的是"在哪里画什么"，而非"内容的逻辑结构"。这带来几个根本性难题：

```
┌─────────────────────────────────────────┐
│           PDF 页面渲染结果               │
│                                         │
│  ┌──────────┐  ┌───────────────────┐    │
│  │  标题文字  │  │   正文段落文字     │    │
│  └──────────┘  └───────────────────┘    │
│                                         │
│  ┌─────────────────────────────────┐    │
│  │   表格（视觉上是网格）            │    │
│  │  │ A │ B │ C │                   │    │
│  │  │ 1 │ 2 │ 3 │                   │    │
│  └─────────────────────────────────┘    │
│                                         │
│  [图片]    公式: E = mc²               │
│                                         │
│  但 PDF 内部存储的是：                   │
│  (x:72, y:720, "标题文字", Font:Bold)   │
│  (x:72, y:680, "正文", Font:Regular)    │
│  (x:72, y:100, 绘制线段...)             │
│  → 没有表格、段落、标题等语义结构        │
└─────────────────────────────────────────┘
```

具体挑战包括：**表格**——单元格合并、嵌套表头、跨页表格；**图片**——需要 OCR 或 VLM 理解图中内容；**公式**——MathJax 渲染后的公式无法直接提取 LaTeX 源码；**多栏布局**——双栏论文的阅读顺序如何还原。

### 1.2 解析工具对比

| 工具 | 原理 | 表格支持 | OCR | 布局分析 | 适用场景 |
|------|------|----------|-----|----------|----------|
| PyPDF2 | 纯文本提取 | 无 | 无 | 无 | 简单文本 PDF |
| pdfplumber | 基于 pdfminer.six | 中等 | 无 | 无 | 含表格的简单 PDF |
| PyMuPDF (fitz) | 渲染+提取 | 弱 | 无 | 无 | 快速批量处理 |
| Unstructured | 多引擎+模型 | 强 | 有 | 有 | 通用文档处理 |
| LlamaParse | LLM 辅助解析 | 强 | 有 | 强 | 复杂技术文档 |

```python
# pdfplumber 提取表格示例
import pdfplumber

with pdfplumber.open("report.pdf") as pdf:
    for page in pdf.pages:
        tables = page.extract_tables()
        for table in tables:
            # table 是二维列表，每行是一个列表
            for row in table:
                print(row)

        # 同时提取页面文本
        text = page.extract_text()
```

> [!hint]- 面试场景：如何处理包含复杂表格和图片的技术文档？
> 思考：应该选择什么工具？处理流程是什么？

> [!success]- 参考答案
> 推荐分层处理策略：
> 1. **第一层：布局分析**——用 Unstructured 或 LlamaParse 识别页面区域（标题/段落/表格/图片/页眉页脚）
> 2. **第二层：分区域处理**——表格区域用专用表格解析器（如 Camelot/Tabula）；图片区域用 VLM（如 GPT-4o-mini）生成描述；公式区域用 Nougat 或 Pix2Text 转 LaTeX
> 3. **第三层：结构重组**——按逻辑阅读顺序合并各区域内容，保留结构标记（Markdown 标题层级）
> 4. **关键点**：不要试图用一个工具处理所有内容，而是多工具协同，每个区域用最擅长的方法

### 1.3 多模态文档处理

对于复杂文档（论文、财报、技术手册），**用 VLM 直接"看"页面**正在成为主流方案。核心思路是将每页渲染为图片，然后交给视觉语言模型提取结构化内容。

```
PDF 页面 → 渲染为图片 → VLM (GPT-4o / Qwen-VL)
                              ↓
                    结构化 Markdown 输出
                    （保留表格、公式、层级）
```

优势：不依赖 PDF 内部结构，直接理解视觉布局。劣势：成本高、速度慢，适合高质量要求的离线处理。

## 二、Chunk 策略深入

Chunk（分块）策略直接决定检索质量。太大的块会引入噪声，太小的块会丢失上下文。

### 2.1 固定大小分块与参数调优

参数 `chunk_size`（块大小）和 `overlap`（重叠）的经验值：

| 场景 | chunk_size | overlap | 说明 |
|------|-----------|---------|------|
| FAQ 类文档 | 200-400 | 50 | 短问答，信息密度高 |
| 技术文档 | 500-1000 | 100-200 | 需要完整上下文 |
| 长篇报告 | 1000-2000 | 200-400 | 主题跨度大 |
| 法律合同 | 800-1200 | 150 | 条款需完整保留 |

### 2.2 递归字符分块实现

递归字符分块（Recursive Character Text Splitting）是 LangChain 的默认策略，按分隔符优先级逐级切分：

```python
from langchain.text_splitter import RecursiveCharacterTextSplitter

splitter = RecursiveCharacterTextSplitter(
    separators=["\n\n", "\n", "。", "！", "？", "；", " ", ""],
    chunk_size=500,
    chunk_overlap=100,
    length_function=len,
)

chunks = splitter.split_text(document_text)
```

工作原理：先用 `"\n\n"`（段落）切分，如果块仍然超过 `chunk_size`，则用下一个分隔符继续切分，以此类推。这样可以尽量保持段落完整性。

### 2.3 语义分块

语义分块的核心思想：通过 Embedding 相似度变化点来检测"主题切换"，在切换处断开。

```python
import numpy as np
from sentence_transformers import SentenceTransformer

model = SentenceTransformer('BAAI/bge-small-zh-v1.5')

def semantic_chunk(text: str, threshold_percentile: int = 75) -> list[str]:
    """基于语义相似度变化的分块算法"""
    # Step 1: 按句子切分
    sentences = [s.strip() for s in text.split('。') if s.strip()]

    # Step 2: 计算相邻句子的 Embedding
    embeddings = model.encode(sentences, normalize_embeddings=True)

    # Step 3: 计算相邻句子的余弦相似度
    similarities = [
        np.dot(embeddings[i], embeddings[i+1])
        for i in range(len(embeddings) - 1)
    ]

    # Step 4: 找到相似度低谷（主题切换点）
    threshold = np.percentile(similarities, threshold_percentile)
    # 低于阈值的点即为断点——相邻句子语义差异较大

    chunks = []
    start = 0
    for i, sim in enumerate(similarities):
        if sim < threshold:
            chunks.append('。'.join(sentences[start:i+1]) + '。')
            start = i + 1
    # 添加最后一块
    if start < len(sentences):
        chunks.append('。'.join(sentences[start:]) + '。')

    return chunks
```

> [!hint]- 语义分块的计算开销如何优化？
> 思考：每个句子都要做 Embedding，文档量大时怎么处理？

> [!success]- 参考答案
> 1. **批量编码**：`model.encode()` 支持批量处理，一次传入所有句子比逐句编码快 10-50 倍
> 2. **滑动窗口近似**：不逐句切分，而是按固定窗口（如 3 句）切分后再检测变化点，减少 Embedding 次数
> 3. **离线预处理**：语义分块只在索引构建时执行一次，不影响在线查询性能
> 4. **混合策略**：先用递归字符分块做粗切，再用语义分块在粗块内做细调

### 2.4 句子窗口检索（Sentence Window Retrieval）

核心思想：**小块检索，大窗口返回上下文**。存储时每个句子单独 Embedding，检索命中后返回前后各 K 个句子作为上下文。

```
存储：[S1] [S2] [S3] [S4] [S5] [S6] [S7] [S8]
           ← 检索命中 S4 →
返回：[S2] [S3] [S4] [S5] [S6]  ← 窗口大小=5
```

```python
# 句子窗口检索的实现思路
from dataclasses import dataclass

@dataclass
class SentenceWith Context:
    sentence: str
    prev_context: str    # 前文
    next_context: str    # 后文
    index: int

def build_sentence_windows(
    sentences: list[str], window_size: int = 3
) -> list[SentenceWithContext]:
    """为每个句子构建上下文窗口"""
    results = []
    full_text = ' '.join(sentences)
    for i, sent in enumerate(sentences):
        start = max(0, i - window_size)
        end = min(len(sentences), i + window_size + 1)
        results.append(SentenceWithContext(
            sentence=sent,
            prev_context=' '.join(sentences[start:i]),
            next_context=' '.join(sentences[i+1:end]),
            index=i,
        ))
    return results
```

优势：检索粒度细（单句级），上下文丰富。适合需要精确定位的场景（如法规条款、技术规范）。

### 2.5 Chunk 策略 A/B 实验设计

评估 Chunk 策略不能靠直觉，必须做实验。以下是标准的实验框架：

```
实验变量：chunk_strategy ∈ {fixed_500, fixed_1000, recursive, semantic}
固定变量：embedding_model, vector_db, retriever_top_k, llm_model
评估指标：
  - 检索准确率 (Recall@K)
  - 答案准确率 (Answer Correctness, GPT-4 评估)
  - 答案相关性 (Answer Relevancy)
  - 上下文精确率 (Context Precision)
数据集：标注 100-200 个 (问题, 相关文档段落, 正确答案) 三元组
统计检验：配对 t 检验，显著性水平 p < 0.05
```

> [!hint]- 面试场景：用户反馈 RAG 系统经常检索到无关内容，你如何排查和优化？
> 思考：从哪个环节开始排查？

> [!success]- 参考答案
> 排查顺序：
> 1. **先看原始问题**：收集 Bad Case，分析检索失败的共性问题（问题太短？歧义？专业术语未覆盖？）
> 2. **检查 Chunk 质量**：随机抽样 50 个 chunk，人工评估信息完整性（是否一个 chunk 包含完整语义单元）
> 3. **检查 Embedding 质量**：用已知相关的 query-doc 对测试相似度分数分布，确认语义空间是否合理
> 4. **检查检索参数**：top_k 是否太小？相似度阈值是否合理？
> 5. **针对性优化**：如果 chunk 质量差 → 调整 chunk 策略；如果 embedding 差 → 换模型或微调；如果检索参数差 → 调 top_k + 加 rerank

## 三、Embedding 模型深入

### 3.1 对比学习训练原理

Embedding 模型的核心训练范式是**对比学习**：拉近相关文本对（正例），推远不相关文本对（负例）。

```
训练数据格式：
  (anchor, positive, negative_1, negative_2, ..., negative_N)

例：
  anchor:    "什么是机器学习？"
  positive:  "机器学习是人工智能的一个分支，通过数据训练模型..."
  negative1: "今天天气很好"
  negative2: "Python 是一种编程语言"
```

训练目标函数 **InfoNCE Loss**：

$$\mathcal{L} = -\log \frac{\exp(\text{sim}(a, p^+) / \tau)}{\sum_{i=1}^{N} \exp(\text{sim}(a, p_i) / \tau)}$$

这个公式的自然语言解读：分子是锚点与正例的相似度（经过温度缩放 $\tau$ 后取指数），分母是锚点与所有候选（包括正例和 N-1 个负例）的相似度之和。优化目标是让正例的相似度在所有候选中占比尽可能大，即让模型"从一堆文本中挑出最相关的那个"。

### 3.2 BM25 的完整数学基础

BM25 是稀疏检索的经典算法，基于词频统计和文档长度归一化：

$$\text{BM25}(Q, D) = \sum_{i=1}^{n} \text{IDF}(q_i) \cdot \frac{f(q_i, D) \cdot (k_1 + 1)}{f(q_i, D) + k_1 \cdot (1 - b + b \cdot \frac{|D|}{\text{avgdl}})}$$

逐项解读：
- $Q = (q_1, q_2, ..., q_n)$ 是查询中的 n 个词项
- $f(q_i, D)$ 是词项 $q_i$ 在文档 $D$ 中的词频
- $|D|$ 是文档长度，$\text{avgdl}$ 是平均文档长度
- $k_1$ 控制词频饱和速度（通常取 1.2-2.0），词频越高增益递减
- $b$ 控制文档长度归一化强度（通常取 0.75），$b=0$ 表示不考虑文档长度
- $\text{IDF}(q_i) = \log\frac{N - n(q_i) + 0.5}{n(q_i) + 0.5}$，$N$ 是总文档数，$n(q_i)$ 是包含该词项的文档数

BM25 的核心思想：一个词项对文档相关性的贡献由三个因素决定——**稀有程度**（IDF 高）、**出现频率**（词频高）、**文档长度**（短文档中出现的权重更高）。

### 3.3 主要 Embedding 模型对比

| 模型 | 维度 | 中文支持 | MTEB 排名 | 特点 |
|------|------|----------|-----------|------|
| text-embedding-3-small | 1536 | 良好 | 前列 | API 调用，成本高 |
| text-embedding-3-large | 3072 | 良好 | 前列 | 高维度，精度最高 |
| bge-large-zh-v1.5 | 1024 | 原生 | 中文场景第一 | 中文首选开源模型 |
| bge-m3 | 1024 | 多语言 | 多语言前列 | 支持稠密+稀疏+ColBERT 三合一 |
| gte-large-zh | 1024 | 原生 | 中文前列 | 阿里达摩院出品 |
| e5-large-v2 | 1024 | 良好 | 前列 | 微软出品，需加前缀指令 |
| m3e-large | 1024 | 原生 | 中文良好 | 中文社区广泛使用 |

> [!hint]- 面试场景：中文企业知识库选什么 Embedding 模型？
> 思考：考虑因素有哪些？成本、精度、延迟、数据安全？

> [!success]- 参考答案
> 选型决策树：
> 1. **数据安全要求高**（金融/医疗）→ 必须本地部署 → 开源模型
> 2. **纯中文场景** → bge-large-zh-v1.5 或 gte-large-zh（MTEB 中文榜单前列）
> 3. **中英混合** → bge-m3（多语言支持强）
> 4. **追求最高精度且预算充足** → text-embedding-3-large（API 调用）
> 5. **需要同时支持稀疏检索** → bge-m3（内置稀疏表示生成）
> 6. **最终建议**：先用 bge-large-zh-v1.5 跑基线，如果效果不够再考虑微调或换 API 模型。关键不在模型大小，而在**领域适配**——用你的数据微调小模型可能比直接用大模型效果更好

### 3.4 Matryoshka Representation Learning (MRL)

MRL 的核心创新：训练时同时优化多个维度的表示，使得同一向量可以截断到不同维度使用。

```
原始向量：[0.3, -0.1, 0.8, 0.2, -0.5, 0.7, ...]  (1024维)
截断到 256 维：[0.3, -0.1, 0.8, 0.2, ...]          (精度损失小)
截断到 64 维：[0.3, -0.1, 0.8, 0.2]                 (精度损失可接受)
```

训练时的损失函数是多个维度损失的加权和：

$$\mathcal{L}_{MRL} = \sum_{d \in \{64, 128, 256, 512, 1024\}} \alpha_d \cdot \mathcal{L}(\mathbf{z}_{1:d})$$

实际意义：可以用 64 维向量做粗筛（速度快、内存省），然后用 1024 维向量做精排（精度高），实现**多阶段检索**。

### 3.5 Fine-tune Embedding 模型

微调 Embedding 的关键步骤：

```python
from sentence_transformers import SentenceTransformer, InputExample, losses
from torch.utils.data import DataLoader

# 1. 加载预训练模型
model = SentenceTransformer('BAAI/bge-large-zh-v1.5')

# 2. 准备领域数据（正例对）
train_examples = [
    InputExample(texts=["如何重置密码？", "在设置页面点击'忘记密码'，通过邮件验证后重置"], label=1.0),
    InputExample(texts=["报销流程是什么？", "差旅报销需先在OA系统提交申请，附发票扫描件"], label=1.0),
    # ... 更多领域 QA 对
]

# 3. 使用 MultipleNegativesRankingLoss（无需显式负例）
train_dataloader = DataLoader(train_examples, shuffle=True, batch_size=32)
train_loss = losses.MultipleNegoriesRankingLoss(model)

# 4. 训练
model.fit(
    train_objectives=[(train_dataloader, train_loss)],
    epochs=3,
    warmup_steps=100,
)
```

`MultipleNegativesRankingLoss` 的巧妙之处：batch 内其他样本自动成为负例，不需要手工标注负例对。batch_size 越大，负例越多，训练效果越好。

## 四、向量数据库深入

### 4.1 向量索引算法原理

**HNSW（Hierarchical Navigable Small World）**

HNSW 构建多层图结构，底层包含所有节点，每上一层节点数量指数衰减。搜索时从顶层开始逐层下降，每层做贪心搜索。

```
Layer 2:    N1 ──────────── N5          （少量节点，长距离跳跃）
             │               │
Layer 1:    N1──N3──N5──N7──N9          （中等节点，中距离）
             │  │  │  │  │  │
Layer 0:    N1─N2─N3─N4─N5─N6─N7─N8─N9  （全部节点，短距离精确）
```

查询过程：从 Layer 2 的入口节点开始贪心搜索 → 找到局部最优后进入 Layer 1 → 继续贪心搜索 → 最终在 Layer 0 精确定位。时间复杂度 $O(\log N)$。

**IVF（Inverted File Index）**

将向量空间划分为 K 个 Voronoi 区域（聚类中心），每个区域维护一个倒排列表。查询时只搜索最近的 nprobe 个区域。

```
         * C1
        /|\
       / | \
      *  *  *      ← 属于 C1 的向量
     /   |   \
    *    *    *
         |
    ─────┼─────     ← 区域边界
         |
      *  C2  *      ← 属于 C2 的向量
       \ | /
        \|/
         *
```

参数 `nprobe` 控制精度和速度的平衡：nprobe 越大越精确但越慢。

**PQ（Product Quantization）**

将高维向量切成 M 个子空间，每个子空间独立量化为有限个码字。存储时只存码字索引，内存占用从 $D \times 4$ 字节降到 $M \times 1$ 字节。

```
原始向量（1024维 float32 = 4096 字节）
[d1, d2, ..., d256 | d257, ..., d512 | d513, ..., d768 | d769, ..., d1024]
    子空间 1           子空间 2           子空间 3           子空间 4
    ↓ 量化             ↓ 量化             ↓ 量化             ↓ 量化
    码字 #42           码字 #17           码字 #89           码字 #3
存储: [42, 17, 89, 3] = 4 字节 (压缩 1000 倍)
```

### 4.2 三种算法对比

| 指标 | HNSW | IVF | PQ (通常与 IVF 组合) |
|------|------|-----|---------------------|
| 查询复杂度 | $O(\log N)$ | $O(N \cdot \text{nprobe}/K)$ | $O(N \cdot \text{nprobe}/K)$ |
| 精度 | 高（>95%） | 中等（依赖 nprobe） | 中低（有量化损失） |
| 内存占用 | 高（存图结构） | 中等 | 低（压缩存储） |
| 构建速度 | 慢 | 快 | 快 |
| 适用规模 | <1000 万 | 百万~亿级 | 亿级以上 |

### 4.3 Milvus 架构解析

```
                    ┌─────────────┐
                    │   Client    │
                    └──────┬──────┘
                           │
                    ┌──────▼──────┐
                    │    Proxy    │  ← 请求接入、SQL 解析、执行计划
                    └──────┬──────┘
                           │
          ┌────────────────┼────────────────┐
          │                │                │
   ┌──────▼──────┐  ┌──────▼──────┐  ┌──────▼──────┐
   │  QueryNode  │  │  DataNode   │  │  IndexNode  │
   │  (检索执行)  │  │  (数据写入)  │  │  (索引构建)  │
   └──────┬──────┘  └──────┬──────┘  └──────┬──────┘
          │                │                │
   ┌──────▼──────┐  ┌──────▼──────┐  ┌──────▼──────┐
   │QueryCoord   │  │DataCoord    │  │IndexCoord   │
   │(查询协调)    │  │(数据协调)    │  │(索引协调)    │
   └─────────────┘  └─────────────┘  └─────────────┘
          │                │                │
          └────────────────┼────────────────┘
                           │
                    ┌──────▼──────┐
                    │ Object Store│  ← MinIO / S3
                    │ (持久化存储)  │
                    └─────────────┘
```

核心设计：**存算分离**——QueryNode 负责查询（可水平扩展），DataNode 负责写入，IndexNode 负责后台索引构建。三者独立伸缩，适合云原生部署。

### 4.4 pgvector 索引选择

pgvector 是 PostgreSQL 的向量扩展，适合已有 PG 基础设施的团队：

```sql
-- 创建向量列
ALTER TABLE documents ADD COLUMN embedding vector(1024);

-- IVFFlat 索引（适合 > 10 万行）
CREATE INDEX ON documents USING ivfflat (embedding vector_cosine_ops) WITH (lists = 100);

-- HNSW 索引（适合 < 100 万行，精度更高）
CREATE INDEX ON documents USING hnsw (embedding vector_cosine_ops) WITH (m = 16, ef_construction = 64);

-- 混合查询：向量相似度 + 元数据过滤
SELECT id, content, 1 - (embedding <=> $1) AS similarity
FROM documents
WHERE metadata->>'department' = 'engineering'
ORDER BY embedding <=> $1
LIMIT 10;
```

`lists` 参数（IVFFlat）建议设为 $\sqrt{N}$，其中 $N$ 是行数。`m` 参数（HNSW）控制图的连通度，通常取 16-64。

> [!hint]- 面试场景：100 万文档的向量数据库如何选型和调优？
> 思考：选 Milvus 还是 pgvector？索引参数怎么设？

> [!success]- 参考答案
> **选型**：100 万文档属于中等规模，两个都能胜任，取决于团队背景：
> - 已有 PG 基础设施且团队熟悉 SQL → pgvector（运维简单，学习成本低）
> - 需要独立扩展检索服务、预计数据量会快速增长 → Milvus（存算分离，云原生）
>
> **调优要点**：
> 1. **索引选择**：100 万行用 HNSW（精度优先），参数 m=32, ef_construction=128
> 2. **ef_search**（查询时参数）：从 64 开始，逐步增大到精度满足要求为止
> 3. **量化**：如果内存紧张，考虑 IVF+PQ 组合索引，牺牲 5-10% 精度换 10 倍内存节省
> 4. **预热**：首次查询慢是正常的，用脚本预热缓存
> 5. **监控指标**：查询延迟 P99、召回率（定期用标注数据验证）、内存使用率

### 4.5 过滤检索（Metadata Filtering）

实际场景中，几乎总是需要**先过滤再检索**（或同时进行）：

```python
# Milvus 中的过滤检索示例
from pymilvus import Collection

results = collection.search(
    data=[query_embedding],
    anns_field="embedding",
    param={"metric_type": "COSINE", "params": {"ef": 128}},
    limit=10,
    expr='department == "tech" AND year >= 2024',  # 元数据过滤
    output_fields=["content", "title", "department"]
)
```

注意：过滤应该在索引层面完成，而不是检索后再过滤（否则可能返回空结果，因为 top-K 命中的可能都不满足过滤条件）。Milvus 和 pgvector 都支持索引内过滤。

## 关键要点回顾

1. **文档解析**：PDF 不是结构化文档，需要布局分析 + 多工具协同处理不同区域
2. **Chunk 策略**：没有万能策略，必须根据文档类型和检索场景做 A/B 实验验证
3. **Embedding 选型**：中文场景首选 bge-large-zh-v1.5，有领域数据则微调效果更好
4. **MRL**：一次训练多维度可用，适合粗筛+精排的多阶段检索架构
5. **向量索引**：100 万级用 HNSW，千万级以上考虑 IVF+PQ；选型看团队基础设施
6. **过滤检索**：必须在索引层面做过滤，不能检索后过滤

## 扩展阅读

- [LlamaParse 官方文档](https://docs.llamaindex.ai/en/latest/module_guides/loading/node_parsers/)
- [MTEB Embedding 排行榜](https://huggingface.co/spaces/mteb/leaderboard)
- [HNSW 原始论文](https://arxiv.org/abs/1603.09320)
- [Matryoshka Representation Learning](https://arxiv.org/abs/2205.13147)
- [BGE 模型系列](https://huggingface.co/BAAI)

## 下一步学习

- [[raw/lessons/AI-Interview-Prep/03b-rag-retrieval|03b - RAG 检索优化深入]]：检索策略、Rerank、评估体系
