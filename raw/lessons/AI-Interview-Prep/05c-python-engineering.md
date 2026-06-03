---
type: lesson
tags: [面试, Python, 内存管理, 性能优化, 设计模式, 工程实践]
created: 2026-05-20
updated: 2026-05-20
difficulty: advanced
prerequisites: [Python高级特性, 并发编程]
topic: Python工程实践与性能优化
status: in-progress
series: {name: "AI Interview Prep", part: "5c"}
---

# Python 工程实践与性能优化

> 面向 AI 工程师面试的 Python 内存管理、性能调优、设计模式和工程化实践深度指南。

## 学习目标

- [ ] 理解 Python 引用计数和分代 GC 的完整机制
- [ ] 掌握内存泄漏排查的全套工具链
- [ ] 能够对 AI 服务做系统性的性能优化
- [ ] 在 AI 项目中正确应用 6 种核心设计模式
- [ ] 设计健壮的错误处理与容错机制
- [ ] 组织规范的 AI 项目工程结构

## 一、Python 内存管理深入

### 1.1 引用计数机制

Python 中每个对象都是一个 `PyObject` 结构体，其 `ob_refcnt` 字段记录当前被引用的次数。当引用计数降为 0 时，对象立即被回收。

```python
import sys

# 引用计数增加的典型场景
a = []          # ob_refcnt = 1（创建赋值）
b = a           # +1 → 2（别名赋值）
c = [a]         # +1 → 3（加入容器）
d = {'k': a}    # +1 → 4（作为字典值）
sys.getrefcount(a)  # +1（传参给 getrefcount）→ 临时 5，函数返回后回到 4

# 注意：sys.getrefcount() 自身的调用会增加一次临时引用
# 所以实际引用计数 = getrefcount 返回值 - 1
```

> [!hint]- 面试题：`sys.getrefcount(x)` 返回 3，实际引用计数是多少？为什么？
> 实际引用计数是 2。因为 `getrefcount` 内部需要创建一个临时引用（函数参数绑定），加上调用者持有的引用，所以返回值 = 实际引用计数 + 1。如果你用 `a = x; b = x; sys.getrefcount(x)` 来看，返回值是 4（a, b, 栈上临时变量, getrefcount 参数），实际只有 a 和 b 两个引用。

> [!hint]- 面试题：以下代码是否存在内存泄漏？
> ```python
> class Node:
>     def __init__(self):
>         self.parent = None
>         self.children = []
>
> root = Node()
> child = Node()
> child.parent = root
> root.children.append(child)
> del root
> del child
> ```
> 存在循环引用导致的泄漏。`root → children → child → parent → root` 构成循环引用。虽然 `del` 减少了外部引用，但内部循环引用使两个对象的引用计数都不为 0。Python 的分代 GC 可以处理这种情况——但如果 `__del__` 方法被定义，GC 可能无法回收（详见下文）。

### 1.2 分代垃圾回收

Python 采用三代的分代回收策略。新创建的对象放在第 0 代，经过一次 GC 存活后晋升到下一代。

```python
import gc

# 查看当前阈值
print(gc.get_threshold())  # 默认 (700, 10, 5)
# 700: 第 0 代对象数达到 700 时触发第 0 代回收
# 10:  每 10 次第 0 代回收，触发一次第 1 代回收
# 5:   每 5 次第 1 代回收，触发一次第 2 代回收

# 调整阈值（高性能服务中可能需要调优）
gc.set_threshold(1500, 15, 10)

# 调试循环引用
gc.set_debug(gc.DEBUG_SAVEALL)  # 将不可达对象保存到 gc.garbage
gc.collect()
print(f"不可达对象数: {len(gc.garbage)}")
```

> [!hint]- 面试题：为什么定义了 `__del__` 方法的对象循环引用无法被 GC 回收？
> Python 的 GC 在检测到循环引用时，需要确定回收顺序。但如果对象定义了 `__del__`，GC 无法确定哪个 `__del__` 应该先执行（因为它们互相引用），所以 Python 将这些对象放入 `gc.garbage` 列表而不回收。解决方案：用 `weakref` 打破循环，或者使用上下文管理器（`__enter__`/`__exit__`）替代 `__del__`。

### 1.3 内存池机制（pymalloc）

Python 对小对象（≤512 字节）有专门的内存池，避免频繁调用系统 `malloc`/`free`。三级结构为 Arena → Pool → Block：

```
Arena (256KB)
  └── Pool (4KB)
       └── Block (8/16/24/.../512 字节，按 size class 分配)
```

> [!hint]- 面试题：为什么 Python 进程释放对象后，`top` 显示的内存占用不下降？
> pymalloc 分配的内存按 Pool 管理。一个 Pool 中只要还有任何 Block 在使用，整个 Pool 就不会归还给操作系统。大量临时对象会使 pymalloc 向 OS 申请内存，但释放后这些内存被 pymalloc 持有复用，不会归还。只有超过 512 字节的大对象（直接用系统 malloc）释放后才会归还 OS。对于长期运行的服务，这通常不是问题，但如果需要强制释放，可以考虑使用 `malloc_trim(0)` 或子进程隔离。

### 1.4 内存泄漏排查完整流程

```python
import tracemalloc
import objgraph
import gc

# ===== Step 1: tracemalloc 定位泄漏点 =====
tracemalloc.start()

# ... 运行业务代码 ...

snapshot = tracemalloc.take_snapshot()
top_stats = snapshot.statistics('lineno')
for stat in top_stats[:10]:
    print(stat)

# ===== Step 2: objgraph 分析引用链 =====
# 查找某类对象的数量变化
objgraph.show_most_common_types(limit=10)

# 找到某对象到 GC root 的引用链
x = leaked_objects[0]
objgraph.show_backrefs(x, max_depth=5, filename='ref_chain.png')

# ===== Step 3: 常见泄漏模式 =====
# 模式 1: 全局缓存无限增长
_cache = {}  # 没有大小限制

# 修复：使用 LRU Cache
from functools import lru_cache

@lru_cache(maxsize=1024)
def get_embedding(text: str):
    return compute(text)

# 模式 2: 闭包持有大对象引用
def make_handler(model):
    big_data = model.get_all_weights()  # 整个模型权重
    def handler(query):
        return process(query, big_data)
    return handler  # handler 持有 big_data 引用，永不释放

# 模式 3: 事件监听器未注销
class EventBus:
    def __init__(self):
        self._handlers = []
    def subscribe(self, handler):
        self._handlers.append(handler)  # 强引用，handler 永远不释放
```

> [!hint]- 面试场景：线上 AI 推理服务的内存每小时增长约 100MB，如何排查？
> **回答思路**：
> 1. 先用 `tracemalloc` 在测试环境复现，对比两次快照找到增长最多的分配位置
> 2. 用 `objgraph.show_most_common_types()` 查看哪类对象数量异常增长
> 3. 排查常见模式：全局缓存的 Embedding 结果、请求日志未清理、模型中间结果被闭包捕获
> 4. 典型 AI 服务泄漏点：RAG 的向量缓存没有 TTL、会话历史无限增长、Tensor 未 `.detach().cpu()` 就放入列表
> 5. 修复后用 `memory_profiler` 做回归测试，确保修复有效

## 二、Python 性能优化深入

### 2.1 性能分析工具链

```python
# ===== cProfile: 函数级分析 =====
import cProfile
import pstats

cProfile.run('main()', 'profile.stats')
stats = pstats.Stats('profile.stats')
stats.sort_stats('cumulative').print_stats(20)  # 按累计时间排序

# ===== pyinstrument: 更友好的调用栈分析 =====
# pip install pyinstrument
# 命令行：pyinstrument script.py
# 代码内：
from pyinstrument import Profiler

profiler = Profiler()
profiler.start()
# ... 运行业务代码 ...
profiler.stop()
print(profiler.output_text(unicode=True, color=True))

# ===== line_profiler: 行级热点分析 =====
# pip install line_profiler
# 在函数上加 @profile 装饰器，然后：
# kernprof -l -v script.py

# ===== py-spy: 无侵入采样分析（生产环境安全）=====
# pip install py-spy
# py-spy top --pid <PID>       实时查看
# py-spy record --pid <PID> -o profile.svg   生成火焰图
```

### 2.2 常见性能优化模式

```python
import timeit
from collections import defaultdict, deque

# ----- 优化 1: 列表推导 vs 循环 -----
# Before
result = []
for x in range(10000):
    if x % 2 == 0:
        result.append(x * 2)
# After
result = [x * 2 for x in range(10000) if x % 2 == 0]
# 快约 30-40%，因为列表推导在 C 层执行，避免了 LOAD_FAST/STORE_FAST 的字节码开销

# ----- 优化 2: 生成器节省内存 -----
# Before: 一次加载全部
all_embeddings = [compute_embedding(doc) for doc in documents]  # 10万条 → 可能 OOM
# After: 按需生成
def embedding_generator(docs):
    for doc in docs:
        yield compute_embedding(doc)
# 或者分批处理
BATCH_SIZE = 256
for i in range(0, len(documents), BATCH_SIZE):
    batch = documents[i:i+BATCH_SIZE]
    embeddings = [compute_embedding(d) for d in batch]
    index.add(embeddings)

# ----- 优化 3: 集合查找 O(1) vs 列表 O(n) -----
# Before
stop_words_list = ["the", "a", "an", ...]  # 1000 个
"python" in stop_words_list  # O(n) 线性扫描
# After
stop_words_set = set(stop_words_list)
"python" in stop_words_set  # O(1) 哈希查找

# ----- 优化 4: 字符串拼接 -----
# Before
parts = []
for i in range(1000):
    parts.append(f"item_{i}")
result = ""
for p in parts:
    result += p  # 每次创建新字符串对象，O(n²)
# After
result = "".join(parts)  # O(n)，一次性分配

# ----- 优化 5: __slots__ 减少内存 -----
import sys

class UserDict:
    """普通类，内部用 __dict__ 存储属性"""
    def __init__(self, name, age, email):
        self.name = name
        self.age = age
        self.email = email

class UserSlots:
    """使用 __slots__，固定属性，不用 __dict__"""
    __slots__ = ('name', 'age', 'email')
    def __init__(self, name, age, email):
        self.name = name
        self.age = age
        self.email = email

u1 = UserDict("Alice", 30, "a@b.com")
u2 = UserSlots("Alice", 30, "a@b.com")
print(f"dict: {sys.getsizeof(u1.__dict__)} bytes")
print(f"slots: {sys.getsizeof(u2)} bytes")  # 通常节省 40-50% 内存
# 对于百万级对象（如向量库中的元数据），差异显著

# ----- 优化 6: local 变量 vs global 变量 -----
import dis
def use_global():
    total = 0
    for i in range(1000):
        total += LEN  # LOAD_GLOBAL
def use_local():
    len_val = LEN  # 复制到 local
    total = 0
    for i in range(1000):
        total += len_val  # LOAD_FAST，比 LOAD_GLOBAL 快约 40%
LEN = 100
```

### 2.3 NumPy 向量化替代 Python 循环

```python
import numpy as np

# 场景：计算两个 Embedding 的余弦相似度矩阵
# 10000 个 query，50000 个 document

# Before: Python 循环（极慢）
def cosine_sim_loop(queries, docs):
    results = np.zeros((len(queries), len(docs)))
    for i, q in enumerate(queries):
        for j, d in enumerate(docs):
            dot = sum(a * b for a, b in zip(q, d))
            norm_q = sum(a**2 for a in q) ** 0.5
            norm_d = sum(a**2 for a in d) ** 0.5
            results[i][j] = dot / (norm_q * norm_d)
    return results  # 10000 * 50000 = 5亿次循环，可能需要数小时

# After: NumPy 向量化
def cosine_sim_vectorized(queries, docs):
    # queries: (10000, 768), docs: (50000, 768)
    q_norm = queries / np.linalg.norm(queries, axis=1, keepdims=True)
    d_norm = docs / np.linalg.norm(docs, axis=1, keepdims=True)
    # 分批计算避免 OOM
    BATCH = 1000
    results = np.empty((len(queries), len(docs)), dtype=np.float32)
    for i in range(0, len(queries), BATCH):
        results[i:i+BATCH] = q_norm[i:i+BATCH] @ d_norm.T
    return results  # 几十秒完成
```

> [!hint]- 面试场景：RAG 服务的检索延迟是 500ms，如何优化到 50ms？
> **回答框架（分层优化）**：
> 1. **算法层**：确认是否用 ANN 索引（FAISS HNSW / ScaNN）替代暴力搜索，这是最大的增益点
> 2. **计算层**：向量归一化预处理，避免每次查询重复计算；使用 float16 替代 float32 减少内存带宽
> 3. **批处理**：多个 query 合并成 batch 做矩阵乘法，利用 SIMD 和 GPU 并行
> 4. **缓存层**：热点 query 的结果缓存（LRU/TTL），避免重复检索
> 5. **工程层**：预加载索引到内存（mmap）、连接池复用、异步 I/O
> 6. **量化**：如果精度允许，使用 PQ（Product Quantization）压缩向量，减少内存占用和计算量
> 典型路径：暴力搜索 500ms → FAISS IVF 80ms → HNSW + 预处理 30ms → 加缓存 P99 < 20ms

## 三、设计模式深入（AI 项目实战）

### 3.1 工厂模式 + 注册表模式

```python
from abc import ABC, abstractmethod
from typing import Dict, Type

class LLMProvider(ABC):
    """LLM 提供者抽象基类"""
    @abstractmethod
    async def complete(self, prompt: str, **kwargs) -> str:
        ...

    @abstractmethod
    async def stream(self, prompt: str, **kwargs):
        ...

# 注册表：自动注册所有 Provider
_registry: Dict[str, Type[LLMProvider]] = {}

def register_provider(name: str):
    """装饰器：将 Provider 注册到工厂"""
    def decorator(cls: Type[LLMProvider]):
        _registry[name] = cls
        return cls
    return decorator

@register_provider("openai")
class OpenAIProvider(LLMProvider):
    def __init__(self, model: str = "gpt-4", **kwargs):
        self.model = model
        self.api_key = kwargs.get("api_key")

    async def complete(self, prompt: str, **kwargs) -> str:
        # 实际调用 OpenAI API
        return f"[OpenAI {self.model}] {prompt}"

    async def stream(self, prompt: str, **kwargs):
        yield f"[OpenAI {self.model}] streaming..."

@register_provider("anthropic")
class AnthropicProvider(LLMProvider):
    def __init__(self, model: str = "claude-3", **kwargs):
        self.model = model

    async def complete(self, prompt: str, **kwargs) -> str:
        return f"[Anthropic {self.model}] {prompt}"

    async def stream(self, prompt: str, **kwargs):
        yield f"[Anthropic {self.model}] streaming..."

class LLMFactory:
    """工厂：根据配置创建 LLM 实例"""
    @staticmethod
    def create(provider: str, **kwargs) -> LLMProvider:
        if provider not in _registry:
            available = ", ".join(_registry.keys())
            raise ValueError(f"Unknown provider '{provider}'. Available: {available}")
        return _registry[provider](**kwargs)

    @staticmethod
    def list_providers() -> list[str]:
        return list(_registry.keys())

# 使用
llm = LLMFactory.create("openai", model="gpt-4o")
result = await llm.complete("Hello")
```

### 3.2 策略模式

```python
from abc import ABC, abstractmethod
from dataclasses import dataclass

@dataclass
class Chunk:
    content: str
    metadata: dict

class ChunkStrategy(ABC):
    """分块策略基类"""
    @abstractmethod
    def chunk(self, text: str) -> list[Chunk]:
        ...

class FixedSizeChunker(ChunkStrategy):
    """固定大小分块"""
    def __init__(self, chunk_size: int = 512, overlap: int = 50):
        self.chunk_size = chunk_size
        self.overlap = overlap

    def chunk(self, text: str) -> list[Chunk]:
        chunks = []
        start = 0
        idx = 0
        while start < len(text):
            end = start + self.chunk_size
            chunks.append(Chunk(
                content=text[start:end],
                metadata={"index": idx, "start": start, "end": min(end, len(text))}
            ))
            start += self.chunk_size - self.overlap
            idx += 1
        return chunks

class MarkdownChunker(ChunkStrategy):
    """按 Markdown 标题分块"""
    def chunk(self, text: str) -> list[Chunk]:
        import re
        sections = re.split(r'(^#{1,3}\s+.+$)', text, flags=re.MULTILINE)
        chunks = []
        for i in range(1, len(sections), 2):
            title = sections[i].strip()
            content = sections[i + 1].strip() if i + 1 < len(sections) else ""
            chunks.append(Chunk(content=f"{title}\n{content}", metadata={"heading": title}))
        return chunks

class SemanticChunker(ChunkStrategy):
    """语义分块（基于 Embedding 相似度）"""
    def __init__(self, embedding_fn, similarity_threshold: float = 0.7):
        self.embedding_fn = embedding_fn
        self.threshold = similarity_threshold

    def chunk(self, text: str) -> list[Chunk]:
        sentences = [s.strip() for s in text.split('。') if s.strip()]
        if len(sentences) <= 1:
            return [Chunk(content=text, metadata={})]
        # 简化实现：实际中用 embedding 计算相邻句子的相似度来决定分割点
        chunks = []
        current = [sentences[0]]
        for i in range(1, len(sentences)):
            current.append(sentences[i])
            if len(current) >= 5:  # 简化：每 5 句一块
                chunks.append(Chunk(content='。'.join(current), metadata={"sentences": len(current)}))
                current = []
        if current:
            chunks.append(Chunk(content='。'.join(current), metadata={"sentences": len(current)}))
        return chunks

# 使用：配置驱动切换策略
CHUNKERS = {"fixed": FixedSizeChunker, "markdown": MarkdownChunker}

def create_chunker(strategy: str, **kwargs) -> ChunkStrategy:
    return CHUNKERS[strategy](**kwargs)

chunker = create_chunker("fixed", chunk_size=256, overlap=30)
chunks = chunker.chunk(long_document)
```

### 3.3 观察者模式

```python
from typing import Callable
from dataclasses import dataclass
from enum import Enum
import logging

class EventType(Enum):
    STEP_START = "step_start"
    STEP_END = "step_end"
    STEP_ERROR = "step_error"
    PIPELINE_COMPLETE = "pipeline_complete"

@dataclass
class Event:
    type: EventType
    data: dict
    source: str

class EventEmitter:
    """事件发射器——Agent 执行步骤的通知中心"""
    def __init__(self):
        self._listeners: dict[EventType, list[Callable]] = {}

    def on(self, event_type: EventType, listener: Callable):
        self._listeners.setdefault(event_type, []).append(listener)
        return self  # 支持链式调用

    def emit(self, event: Event):
        for listener in self._listeners.get(event.type, []):
            try:
                listener(event)
            except Exception as e:
                logging.error(f"Listener error: {e}")

    def off(self, event_type: EventType, listener: Callable):
        listeners = self._listeners.get(event_type, [])
        if listener in listeners:
            listeners.remove(listener)

# 具体监听器
class LoggingListener:
    def __call__(self, event: Event):
        logging.info(f"[{event.source}] {event.type.value}: {event.data}")

class MetricsListener:
    def __init__(self):
        self.step_durations = {}
    def __call__(self, event: Event):
        if event.type == EventType.STEP_END:
            step = event.data.get("step")
            duration = event.data.get("duration_ms")
            self.step_durations[step] = self.step_durations.get(step, 0) + duration

# 使用
emitter = EventEmitter()
emitter.on(EventType.STEP_START, LoggingListener())
emitter.on(EventType.STEP_END, MetricsListener())
emitter.emit(Event(EventType.STEP_START, {"step": "retrieve"}, "RAGAgent"))
```

### 3.4 模板方法模式

```python
from abc import ABC, abstractmethod

class RAGPipelineBase(ABC):
    """RAG 管道模板：定义标准流程，子类覆盖具体步骤"""

    def run(self, query: str) -> dict:
        """模板方法：固定流程，不可覆盖"""
        # Step 1: 查询预处理
        processed = self.preprocess(query)

        # Step 2: 检索
        documents = self.retrieve(processed)

        # Step 3: 后处理/重排（可选覆盖）
        documents = self.rerank(processed, documents)

        # Step 4: 生成
        answer = self.generate(processed, documents)

        # Step 5: 后处理（hook）
        answer = self.postprocess(answer)
        return {"query": query, "answer": answer, "docs": documents}

    def preprocess(self, query: str) -> str:
        """默认实现：简单清洗"""
        return query.strip()

    @abstractmethod
    def retrieve(self, query: str) -> list[dict]:
        """必须实现：检索逻辑"""
        ...

    def rerank(self, query: str, docs: list[dict]) -> list[dict]:
        """可选覆盖：默认不重排"""
        return docs

    @abstractmethod
    def generate(self, query: str, docs: list[dict]) -> str:
        """必须实现：生成逻辑"""
        ...

    def postprocess(self, answer: str) -> str:
        """可选覆盖：默认不做后处理"""
        return answer

class DenseRAGPipeline(RAGPipelineBase):
    """稠密检索 RAG"""
    def __init__(self, vector_store, llm):
        self.vector_store = vector_store
        self.llm = llm

    def retrieve(self, query: str) -> list[dict]:
        return self.vector_store.search(query, top_k=5)

    def generate(self, query: str, docs: list[dict]) -> str:
        context = "\n".join(d["content"] for d in docs)
        prompt = f"Context:\n{context}\n\nQuestion: {query}\nAnswer:"
        return self.llm.complete(prompt)

class HybridRAGPipeline(RAGPipelineBase):
    """混合检索 RAG（稠密 + 稀疏 + 重排）"""
    def __init__(self, vector_store, bm25, reranker, llm):
        self.vector_store = vector_store
        self.bm25 = bm25
        self.reranker = reranker
        self.llm = llm

    def retrieve(self, query: str) -> list[dict]:
        dense = self.vector_store.search(query, top_k=10)
        sparse = self.bm25.search(query, top_k=10)
        # 合并去重
        seen = set()
        merged = []
        for d in dense + sparse:
            if d["id"] not in seen:
                merged.append(d)
                seen.add(d["id"])
        return merged

    def rerank(self, query: str, docs: list[dict]) -> list[dict]:
        return self.reranker.rerank(query, docs, top_k=5)

    def generate(self, query: str, docs: list[dict]) -> str:
        context = "\n".join(d["content"] for d in docs)
        prompt = f"Based on:\n{context}\n\nAnswer: {query}"
        return self.llm.complete(prompt)
```

### 3.5 责任链模式

```python
from abc import ABC, abstractmethod
from dataclasses import dataclass

@dataclass
class LLMResponse:
    content: str
    metadata: dict

class ResponseHandler(ABC):
    """后处理 Handler 基类"""
    def __init__(self):
        self._next: ResponseHandler | None = None

    def set_next(self, handler: 'ResponseHandler') -> 'ResponseHandler':
        self._next = handler
        return handler  # 支持链式配置

    def handle(self, response: LLMResponse) -> LLMResponse:
        response = self.process(response)
        if self._next:
            return self._next.handle(response)
        return response

    @abstractmethod
    def process(self, response: LLMResponse) -> LLMResponse:
        ...

class FormatValidator(ResponseHandler):
    """格式校验：确保输出是合法 JSON/Markdown"""
    def process(self, response: LLMResponse) -> LLMResponse:
        content = response.content.strip()
        if content.startswith("```"):
            content = content.split("\n", 1)[1].rsplit("```", 1)[0].strip()
        response.content = content
        return response

class SafetyFilter(ResponseHandler):
    """安全过滤：移除有害内容"""
    BLOCKED_PATTERNS = ["<script>", "javascript:", "data:text/html"]

    def process(self, response: LLMResponse) -> LLMResponse:
        content = response.content
        for pattern in self.BLOCKED_PATTERNS:
            content = content.replace(pattern, "[FILTERED]")
        response.content = content
        return response

class PIIRedactor(ResponseHandler):
    """PII 脱敏：遮盖敏感信息"""
    import re
    EMAIL_RE = re.compile(r'[\w.-]+@[\w.-]+\.\w+')
    PHONE_RE = re.compile(r'1[3-9]\d{9}')

    def process(self, response: LLMResponse) -> LLMResponse:
        content = response.content
        content = self.EMAIL_RE.sub('[EMAIL]', content)
        content = self.PHONE_RE.sub('[PHONE]', content)
        response.content = content
        return response

# 链式配置
validator = FormatValidator()
safety = SafetyFilter()
pii = PIIRedactor()
validator.set_next(safety).set_next(pii)

# 使用
raw = LLMResponse(content="```json\n{\"email\": \"user@test.com\"}\n```", metadata={})
clean = validator.handle(raw)
# clean.content → '{"email": "[EMAIL]"}'
```

### 3.6 单例模式

```python
import threading

# 方式 1: 线程安全的模块级单例（Python 最推荐）
_model_manager = None
_lock = threading.Lock()

def get_model_manager():
    global _model_manager
    if _model_manager is None:
        with _lock:
            if _model_manager is None:  # double-check
                _model_manager = ModelManager()
    return _model_manager

# 方式 2: 元类单例
class SingletonMeta(type):
    _instances: dict[type, object] = {}
    _lock = threading.Lock()

    def __call__(cls, *args, **kwargs):
        if cls not in cls._instances:
            with cls._lock:
                if cls not in cls._instances:
                    cls._instances[cls] = super().__call__(*args, **kwargs)
        return cls._instances[cls]

class ConfigManager(metaclass=SingletonMeta):
    def __init__(self):
        self.config = {"model": "gpt-4", "temperature": 0.7}

# 测试
c1 = ConfigManager()
c2 = ConfigManager()
assert c1 is c2  # True
```

> [!hint]- 面试题：AI 项目中哪些组件适合用单例？哪些不适合？
> **适合单例**：全局配置管理（ConfigManager）、模型加载器（一个模型只加载一次）、日志管理器、数据库连接池、向量索引（FAISS index 加载开销大）。
> **不适合单例**：请求上下文（每个请求独立）、用户会话（并发安全风险）、可变的业务状态对象。单例的核心问题是隐式全局状态，测试困难。推荐做法：用依赖注入替代直接访问单例，保持代码可测试性。

## 四、错误处理与日志

### 4.1 自定义异常体系

```python
class AppError(Exception):
    """应用异常基类"""
    def __init__(self, message: str, code: str = "UNKNOWN", retryable: bool = False):
        self.message = message
        self.code = code
        self.retryable = retryable
        super().__init__(message)

class LLMError(AppError):
    """LLM 调用相关"""
    pass

class RateLimitError(LLMError):
    """限流错误——可重试"""
    def __init__(self, message: str = "Rate limited", retry_after: float = 1.0):
        super().__init__(message, code="RATE_LIMIT", retryable=True)
        self.retry_after = retry_after

class TokenLimitError(LLMError):
    """Token 超限——不可重试"""
    def __init__(self, tokens: int, limit: int):
        super().__init__(
            f"Token count {tokens} exceeds limit {limit}",
            code="TOKEN_LIMIT", retryable=False
        )

class EmbeddingError(AppError):
    """Embedding 计算相关"""
    pass

class RetrievalError(AppError):
    """检索相关"""
    pass
```

### 4.2 断路器 + 降级 LLM 客户端

```python
import time
import logging
import asyncio
from enum import Enum
from typing import Optional, Callable

logger = logging.getLogger(__name__)

class CircuitState(Enum):
    CLOSED = "closed"       # 正常
    OPEN = "open"           # 熔断，拒绝请求
    HALF_OPEN = "half_open" # 探测恢复

class CircuitBreaker:
    """断路器"""
    def __init__(self, failure_threshold: int = 5, recovery_timeout: float = 30.0,
                 half_open_max: int = 1):
        self.failure_threshold = failure_threshold
        self.recovery_timeout = recovery_timeout
        self.half_open_max = half_open_max
        self.state = CircuitState.CLOSED
        self.failure_count = 0
        self.last_failure_time = 0.0
        self.half_open_count = 0

    def can_execute(self) -> bool:
        if self.state == CircuitState.CLOSED:
            return True
        if self.state == CircuitState.OPEN:
            if time.time() - self.last_failure_time > self.recovery_timeout:
                self.state = CircuitState.HALF_OPEN
                self.half_open_count = 0
                return True
            return False
        # HALF_OPEN
        return self.half_open_count < self.half_open_max

    def record_success(self):
        if self.state == CircuitState.HALF_OPEN:
            self.state = CircuitState.CLOSED
        self.failure_count = 0

    def record_failure(self):
        self.failure_count += 1
        self.last_failure_time = time.time()
        if self.state == CircuitState.HALF_OPEN:
            self.state = CircuitState.OPEN
        elif self.failure_count >= self.failure_threshold:
            self.state = CircuitState.OPEN

class ResilientLLMClient:
    """带断路器、重试和降级的 LLM 客户端"""
    def __init__(self, primary, fallback=None, max_retries: int = 3):
        self.primary = primary
        self.fallback = fallback
        self.max_retries = max_retries
        self.circuit = CircuitBreaker(failure_threshold=5, recovery_timeout=30.0)

    async def complete(self, prompt: str, **kwargs) -> str:
        if not self.circuit.can_execute():
            logger.warning("Circuit OPEN, using fallback")
            return await self._fallback_complete(prompt, **kwargs)

        last_error = None
        for attempt in range(self.max_retries):
            try:
                result = await self.primary.complete(prompt, **kwargs)
                self.circuit.record_success()
                return result
            except RateLimitError as e:
                last_error = e
                wait = e.retry_after * (2 ** attempt)
                logger.warning(f"Rate limited, retrying in {wait:.1f}s (attempt {attempt+1})")
                await asyncio.sleep(wait)
            except TokenLimitError:
                raise  # 不可重试，直接抛出
            except LLMError as e:
                last_error = e
                logger.error(f"LLM error (attempt {attempt+1}): {e}")
                await asyncio.sleep(0.5 * (2 ** attempt))

        self.circuit.record_failure()
        logger.warning("All retries failed, using fallback")
        return await self._fallback_complete(prompt, **kwargs)

    async def _fallback_complete(self, prompt: str, **kwargs) -> str:
        if self.fallback:
            return await self.fallback.complete(prompt, **kwargs)
        return "抱歉，服务暂时不可用，请稍后重试。"
```

> [!hint]- 面试场景：LLM API 频繁超时（P99 > 5s），如何设计容错机制？
> **完整方案**：
> 1. **超时控制**：设置合理的 connect_timeout 和 read_timeout（如 3s/10s），避免无限等待
> 2. **重试策略**：指数退避重试（1s, 2s, 4s），仅对幂等请求和可重试错误（超时、限流、500）重试
> 3. **断路器**：连续 5 次失败后熔断 30s，避免对已经不健康的下游持续施压
> 4. **降级方案**：切换到更快的模型（GPT-4o → GPT-4o-mini）、使用本地缓存结果、返回兜底回答
> 5. **监控告警**：断路器状态变更时告警，跟踪 P50/P95/P99 延迟和错误率
> 6. **流量控制**：客户端限速（令牌桶），避免在高峰期加剧下游压力

## 五、Python 工程化实践

### 5.1 AI 项目代码结构

```
my-ai-project/
├── pyproject.toml           # 项目配置（uv 管理）
├── .python-version          # Python 版本锁定
├── .env                     # 环境变量（gitignored）
├── .env.example             # 环境变量模板
├── Dockerfile
├── docker-compose.yml
├── src/
│   ├── __init__.py
│   ├── config.py            # pydantic-settings 配置
│   ├── api/
│   │   ├── __init__.py
│   │   ├── routes.py        # FastAPI 路由
│   │   └── deps.py          # 依赖注入
│   ├── core/
│   │   ├── __init__.py
│   │   ├── llm.py           # LLM 客户端封装
│   │   ├── embeddings.py    # Embedding 计算
│   │   └── vectorstore.py   # 向量库交互
│   ├── pipeline/
│   │   ├── __init__.py
│   │   ├── chunking.py      # 分块策略
│   │   ├── retrieval.py     # 检索逻辑
│   │   └── generation.py    # 生成逻辑
│   └── utils/
│       ├── __init__.py
│       ├── logging.py       # structlog 配置
│       └── retry.py         # 重试/断路器工具
├── tests/
│   ├── unit/
│   ├── integration/
│   └── conftest.py
├── scripts/
│   ├── index_documents.py   # 索引构建脚本
│   └── evaluate.py          # 评估脚本
└── README.md
```

### 5.2 pydantic-settings 配置管理

```python
from pydantic_settings import BaseSettings
from pydantic import Field

class AppConfig(BaseSettings):
    """类型安全的配置——自动读取环境变量"""
    # LLM
    llm_provider: str = Field(default="openai", alias="LLM_PROVIDER")
    llm_model: str = Field(default="gpt-4o", alias="LLM_MODEL")
    llm_api_key: str = Field(alias="LLM_API_KEY")
    llm_temperature: float = Field(default=0.7, ge=0.0, le=2.0)

    # 向量库
    vector_db_url: str = Field(default="http://localhost:6333", alias="VECTOR_DB_URL")
    embedding_dim: int = Field(default=768, ge=1)

    # 服务
    host: str = "0.0.0.0"
    port: int = 8000
    workers: int = 4
    log_level: str = "info"

    model_config = {"env_file": ".env", "env_file_encoding": "utf-8"}

config = AppConfig()  # 自动从 .env / 环境变量加载，类型校验
```

## 六、面试编程题精选

### 题 1：LRU Cache

```python
from collections import OrderedDict

class LRUCache:
    def __init__(self, capacity: int):
        self.cache = OrderedDict()
        self.capacity = capacity

    def get(self, key: str):
        if key not in self.cache:
            return None
        self.cache.move_to_end(key)  # 移到末尾 = 标记为最近使用
        return self.cache[key]

    def put(self, key: str, value):
        if key in self.cache:
            self.cache.move_to_end(key)
        self.cache[key] = value
        if len(self.cache) > self.capacity:
            self.cache.popitem(last=False)  # 弹出最老的（FIFO 头部）

# 使用
cache = LRUCache(3)
cache.put("a", 1)
cache.put("b", 2)
cache.put("c", 3)
cache.get("a")          # 访问 a，b 变为最老
cache.put("d", 4)       # 淘汰 b
assert cache.get("b") is None
```

### 题 2：异步令牌桶限流器

```python
import asyncio
import time

class AsyncTokenBucket:
    def __init__(self, rate: float, capacity: int):
        self.rate = rate           # 每秒生成令牌数
        self.capacity = capacity   # 桶容量
        self.tokens = capacity     # 当前令牌数
        self.last_refill = time.monotonic()
        self._lock = asyncio.Lock()

    async def acquire(self, tokens: int = 1):
        while True:
            async with self._lock:
                self._refill()
                if self.tokens >= tokens:
                    self.tokens -= tokens
                    return
            await asyncio.sleep(0.05)  # 短暂等待后重试

    def _refill(self):
        now = time.monotonic()
        elapsed = now - self.last_refill
        self.tokens = min(self.capacity, self.tokens + elapsed * self.rate)
        self.last_refill = now

# 使用：限制 QPS 为 10
limiter = AsyncTokenBucket(rate=10, capacity=10)
async def handle_request(req):
    await limiter.acquire()
    return await process(req)
```

### 题 3：线程安全连接池

```python
import threading
import queue
from contextlib import contextmanager

class ConnectionPool:
    def __init__(self, factory, max_size: int = 10, timeout: float = 5.0):
        self.factory = factory
        self.max_size = max_size
        self.timeout = timeout
        self._pool: queue.Queue = queue.Queue(maxsize=max_size)
        self._created = 0
        self._lock = threading.Lock()

    @contextmanager
    def connection(self):
        conn = self._get()
        try:
            yield conn
        finally:
            self._pool.put(conn)

    def _get(self):
        # 优先从池中获取
        try:
            return self._pool.get_nowait()
        except queue.Empty:
            pass
        # 池为空，尝试新建
        with self._lock:
            if self._created < self.max_size:
                self._created += 1
                return self.factory()
        # 已达上限，阻塞等待
        return self._pool.get(timeout=self.timeout)
```

### 题 4：依赖注入容器

```python
from typing import Any, Callable, Dict, Type

class DIContainer:
    def __init__(self):
        self._factories: Dict[Type, Callable] = {}
        self._singletons: Dict[Type, Any] = {}

    def register(self, interface: Type, factory: Callable, singleton: bool = True):
        self._factories[interface] = (factory, singleton)

    def resolve(self, interface: Type) -> Any:
        if interface in self._singletons:
            return self._singletons[interface]
        factory, singleton = self._factories[interface]
        instance = factory(self)
        if singleton:
            self._singletons[interface] = instance
        return instance

# 使用
container = DIContainer()
container.register(ConfigManager, lambda c: ConfigManager())
container.register(LLMClient, lambda c: LLMClient(c.resolve(ConfigManager)))
llm = container.resolve(LLMClient)
```

### 题 5：Top-K（堆）

```python
import heapq

def top_k_largest(nums: list[int], k: int) -> list[int]:
    """最小堆维护 Top-K，O(n log k)"""
    if k >= len(nums):
        return sorted(nums, reverse=True)[:k]
    heap = nums[:k]
    heapq.heapify(heap)           # O(k)
    for num in nums[k:]:
        if num > heap[0]:          # 比堆顶大才替换
            heapq.heapreplace(heap, num)
    return sorted(heap, reverse=True)

def top_k_smallest(nums: list[int], k: int) -> list[int]:
    """最大堆维护最小 K 个"""
    heap = [-x for x in nums[:k]]
    heapq.heapify(heap)
    for num in nums[k:]:
        if -num > heap[0]:
            heapq.heapreplace(heap, -num)
    return sorted([-x for x in heap])
```

### 题 6：TTL Cache

```python
import time
from threading import Lock

class TTLCache:
    def __init__(self, ttl: float = 60.0, max_size: int = 1024):
        self.ttl = ttl
        self.max_size = max_size
        self._store: dict = {}
        self._lock = Lock()

    def get(self, key: str):
        with self._lock:
            if key in self._store:
                value, expire_at = self._store[key]
                if time.time() < expire_at:
                    return value
                del self._store[key]  # 过期清理
        return None

    def set(self, key: str, value):
        with self._lock:
            if len(self._store) >= self.max_size:
                self._evict()
            self._store[key] = (value, time.time() + self.ttl)

    def _evict(self):
        """淘汰过期的，如果没有过期则淘汰最早的"""
        now = time.time()
        expired = [k for k, (_, t) in self._store.items() if t <= now]
        if expired:
            for k in expired:
                del self._store[k]
        else:
            oldest = min(self._store, key=lambda k: self._store[k][1])
            del self._store[oldest]
```

### 题 7：EventBus

```python
import asyncio
from typing import Callable, Any
from collections import defaultdict

class EventBus:
    def __init__(self):
        self._subscribers: dict[str, list[Callable]] = defaultdict(list)

    def subscribe(self, event: str, handler: Callable):
        self._subscribers[event].append(handler)

    def unsubscribe(self, event: str, handler: Callable):
        self._subscribers[event].remove(handler)

    async def publish(self, event: str, data: Any = None):
        tasks = []
        for handler in self._subscribers.get(event, []):
            result = handler(data)
            if asyncio.iscoroutine(result):
                tasks.append(result)
        if tasks:
            await asyncio.gather(*tasks, return_exceptions=True)

    def publish_sync(self, event: str, data: Any = None):
        for handler in self._subscribers.get(event, []):
            handler(data)

# 使用
bus = EventBus()
bus.subscribe("query_received", lambda d: print(f"Query: {d}"))
await bus.publish("query_received", "What is RAG?")
```

### 题 8：流式读取大文件并分批处理

```python
def batch_process_large_file(filepath: str, batch_size: int = 1000,
                              process_fn=None):
    """流式读取大文件，按批处理，内存友好"""
    batch = []
    for line in _read_lines(filepath):
        record = parse_line(line)
        batch.append(record)
        if len(batch) >= batch_size:
            yield process_fn(batch) if process_fn else batch
            batch = []
    if batch:
        yield process_fn(batch) if process_fn else batch

def _read_lines(filepath: str):
    """生成器逐行读取，不一次性加载"""
    with open(filepath, 'r', encoding='utf-8') as f:
        for line in f:
            yield line.rstrip('\n')

def parse_line(line: str) -> dict:
    parts = line.split('\t')
    return {"id": parts[0], "text": parts[1]} if len(parts) >= 2 else {}

# 使用：处理 10GB 的文档语料
for batch in batch_process_large_file("corpus.tsv", batch_size=500,
                                       process_fn=compute_embeddings):
    vector_store.add(batch)
    print(f"Indexed {len(batch)} documents")
```

> [!hint]- 面试题：处理 10GB 文件时如何保证内存不爆炸？如果处理到一半程序崩溃怎么办？
> **内存控制**：使用生成器逐行读取（不 `readlines()`），按 batch_size 分批处理，每批处理完释放。核心原则是不同时持有全部数据。
> **断点续传**：记录已处理的行号或偏移量到 checkpoint 文件，重启后 `seek` 到该位置继续。用 `tell()` 获取文件偏移量，定期保存到磁盘。对于更复杂的场景，可以使用 WAL（Write-Ahead Log）模式：每批处理前记录意图，处理完成后标记确认。

## 关键要点回顾

1. **内存管理**：引用计数是基础，分代 GC 处理循环引用，pymalloc 管理小对象池；泄漏排查工具链为 tracemalloc → objgraph → memory_profiler
2. **性能优化**：先 profile 再优化（cProfile/pyinstrument/py-spy），优先算法优化再微观优化，NumPy 向量化是 AI 项目最常用的加速手段
3. **设计模式**：工厂+注册表解耦组件创建，策略模式支持算法切换，模板方法统一流程骨架，责任链处理管道式变换
4. **容错设计**：断路器保护下游，指数退避重试处理瞬态故障，降级方案保可用性
5. **工程化**：pydantic-settings 管理配置，uv 管理依赖，Ruff/mypy/pytest 保证质量

## 扩展阅读

- [Python Memory Management (Real Python)](https://realpython.com/python-memory-management/)
- [py-spy: Sampling profiler](https://github.com/benfred/py-spy)
- [structlog: Structured Logging](https://www.structlog.org/)
- [Pydantic Settings Docs](https://docs.pydantic.dev/latest/concepts/pydantic_settings/)

## 下一步学习

- [[raw/lessons/AI-Interview-Prep/05-python-mastery|Python 高级特性]]
- [[raw/lessons/AI-Interview-Prep/03-rag|RAG 系统设计]]
