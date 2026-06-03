---
type: lesson
tags: [面试, Python, 并发, 内存管理, 设计模式, 性能优化]
created: 2026-05-19
updated: 2026-05-19
difficulty: intermediate
prerequisites: [Python基础, 数据结构基础]
topic: Python精通面试
status: in-progress
series: {name: "AI Interview Prep", part: 5}
---

# Python 精通——面试必备的进阶知识

> AI 工程岗的 Python 要求不低。核心考察：高级特性理解深度、并发编程能力、内存管理意识、工程实践水平。

## 学习目标

- [ ] 掌握装饰器、元类、描述符等高级特性
- [ ] 理解 asyncio/多线程/多进程的适用场景
- [ ] 能分析 Python 内存管理和 GC 机制
- [ ] 掌握常用的设计模式及其 Python 实现
- [ ] 能进行性能分析和优化

## 一、高级特性——面试必考

### 1.1 装饰器（Decorator）

**面试必问：解释装饰器的原理，手写一个带参数的装饰器**

```python
# 基础装饰器
def timer(func):
    """计算函数执行时间"""
    import time
    def wrapper(*args, **kwargs):
        start = time.time()
        result = func(*args, **kwargs)
        elapsed = time.time() - start
        print(f"{func.__name__} 耗时 {elapsed:.4f}s")
        return result
    return wrapper

@timer  # 等价于 my_func = timer(my_func)
def my_func():
    time.sleep(1)

# 带参数的装饰器（三层嵌套）
def retry(max_attempts=3, delay=1.0):
    """失败重试装饰器"""
    def decorator(func):
        def wrapper(*args, **kwargs):
            for attempt in range(max_attempts):
                try:
                    return func(*args, **kwargs)
                except Exception as e:
                    if attempt == max_attempts - 1:
                        raise
                    time.sleep(delay)
        return wrapper
    return decorator

@retry(max_attempts=5, delay=2.0)
def call_api():
    """调用API，失败自动重试"""
    ...

# 用 functools.wraps 保留元信息
from functools import wraps

def my_decorator(func):
    @wraps(func)  # 保留 func 的 __name__, __doc__ 等
    def wrapper(*args, **kwargs):
        return func(*args, **kwargs)
    return wrapper
```

> [!hint]- 面试场景：装饰器在 AI 项目中的实际应用？
> 1. **API 限流**：`@rate_limit(calls=100, period=60)`
> 2. **缓存**：`@lru_cache(maxsize=128)` — 缓存 Embedding 计算结果
> 3. **重试**：`@retry(max_attempts=3)` — LLM API 调用失败重试
> 4. **日志**：`@log_execution` — 记录函数调用和参数
> 5. **超时**：`@timeout(seconds=30)` — 防止工具调用卡死

### 1.2 上下文管理器（Context Manager）

```python
# 方式1：类实现 __enter__ / __exit__
class DatabaseConnection:
    def __enter__(self):
        self.conn = create_connection()
        return self.conn

    def __exit__(self, exc_type, exc_val, exc_tb):
        self.conn.close()
        # 返回 True 表示异常已处理
        return False

# 方式2：contextmanager 装饰器
from contextlib import contextmanager

@contextmanager
def temp_directory():
    """临时目录，用完自动删除"""
    import tempfile, shutil
    dirpath = tempfile.mkdtemp()
    try:
        yield dirpath
    finally:
        shutil.rmtree(dirpath)

# 使用
with temp_directory() as tmpdir:
    # 在临时目录中处理文件
    process_files(tmpdir)
# 目录已自动删除
```

### 1.3 元类（Metaclass）

```python
# 面试重点：理解 type 是所有类的类

# 普通类创建
class MyClass:
    pass
# 等价于：
MyClass = type('MyClass', (), {})

# 元类：控制类的创建过程
class SingletonMeta(type):
    """单例元类——确保只有一个实例"""
    _instances = {}

    def __call__(cls, *args, **kwargs):
        if cls not in cls._instances:
            cls._instances[cls] = super().__call__(*args, **kwargs)
        return cls._instances[cls]

class LLMService(metaclass=SingletonMeta):
    """全局只有一个 LLM 服务实例"""
    def __init__(self):
        self.model = load_model()  # 只加载一次

# AI项目中的实际应用
class RegistryMeta(type):
    """注册表元类——自动注册子类"""
    _registry = {}

    def __new__(mcs, name, bases, namespace):
        cls = super().__new__(mcs, name, bases, namespace)
        if name != 'BaseTool':  # 跳过基类
            mcs._registry[name] = cls
        return cls

class BaseTool(metaclass=RegistryMeta):
    """所有工具的基类，子类自动注册"""
    pass

class SearchTool(BaseTool): pass
class CodeTool(BaseTool): pass

print(RegistryMeta._registry)
# {'SearchTool': <class SearchTool>, 'CodeTool': <class CodeTool>}
```

### 1.4 描述符（Descriptor）

```python
class ValidatedField:
    """验证描述符——自动校验字段值"""
    def __init__(self, name, field_type, required=True):
        self.name = name
        self.field_type = field_type
        self.required = required

    def __set_name__(self, owner, name):
        self.attr_name = f'_{name}'

    def __get__(self, obj, objtype=None):
        if obj is None:
            return self
        return getattr(obj, self.attr_name, None)

    def __set__(self, obj, value):
        if self.required and value is None:
            raise ValueError(f"{self.name} is required")
        if not isinstance(value, self.field_type):
            raise TypeError(
                f"{self.name} must be {self.field_type.__name__}, "
                f"got {type(value).__name__}"
            )
        setattr(obj, self.attr_name, value)

class ToolConfig:
    name = ValidatedField("name", str)
    timeout = ValidatedField("timeout", int, required=False)

    def __init__(self, name, timeout=None):
        self.name = name
        self.timeout = timeout

# Python 的 property、classmethod、staticmethod 本质上都是描述符！
```

## 二、并发编程——高频考点

### 2.1 三种并发模型对比

| 模型 | 机制 | 适用场景 | GIL 影响 |
|------|------|---------|---------|
| **多线程** | threading | I/O 密集型 | 受 GIL 限制 |
| **多进程** | multiprocessing | CPU 密集型 | 不受 GIL 限制 |
| **asyncio** | 事件循环 | I/O 密集型（网络） | 单线程无 GIL 问题 |

> [!warning] 易混点：GIL（全局解释器锁）
> GIL 使得同一时刻只有一个线程执行 Python 字节码。
> - **CPU 密集型**：多线程无加速（GIL 串行化），必须用多进程
> - **I/O 密集型**：多线程/asyncio 有效（等待 I/O 时释放 GIL）
> - AI 推理通常用 C++ 后端（PyTorch），不受 GIL 限制

### 2.2 asyncio 异步编程

```python
import asyncio
import aiohttp

async def call_llm_api(session, prompt):
    """异步调用 LLM API"""
    async with session.post(
        "https://api.openai.com/v1/chat/completions",
        json={"model": "gpt-4", "messages": [{"role": "user", "content": prompt}]},
        headers={"Authorization": f"Bearer {API_KEY}"}
    ) as response:
        return await response.json()

async def batch_process(prompts):
    """并发处理多个 prompt"""
    async with aiohttp.ClientSession() as session:
        tasks = [call_llm_api(session, p) for p in prompts]
        # 并发执行所有任务
        results = await asyncio.gather(*tasks)
        return results

# asyncio.gather vs TaskGroup (Python 3.11+)
async def batch_process_v2(prompts):
    async with aiohttp.ClientSession() as session:
        async with asyncio.TaskGroup() as tg:
            tasks = [tg.create_task(call_llm_api(session, p)) for p in prompts]
        return [task.result() for task in tasks]
```

> [!hint]- 面试场景：AI Agent 中如何并发调用多个工具？
> ```python
> async def agent_execute(tools_to_call):
>     """Agent 并发调用多个工具"""
>     async with asyncio.TaskGroup() as tg:
>         tasks = {
>             name: tg.create_task(tool.execute(**params))
>             for name, (tool, params) in tools_to_call.items()
>         }
>     return {name: task.result() for name, task in tasks.items()}
>
> # ReAct Agent 中使用
> # 如果 Thought 阶段确定需要同时调用搜索和数据库
> results = await agent_execute({
>     "search": (search_tool, {"query": "天气"}),
>     "database": (db_tool, {"sql": "SELECT ..."})
> })
> ```

### 2.3 线程安全

```python
from threading import Lock
from concurrent.futures import ThreadPoolExecutor

class ThreadSafeCache:
    """线程安全的缓存"""
    def __init__(self):
        self._cache = {}
        self._lock = Lock()

    def get(self, key):
        with self._lock:
            return self._cache.get(key)

    def set(self, key, value):
        with self._lock:
            self._cache[key] = value

# 线程池
def parallel_embedding(texts, model):
    """并行计算 Embedding"""
    def embed_batch(batch):
        return model.encode(batch)

    batches = [texts[i:i+32] for i in range(0, len(texts), 32)]
    with ThreadPoolExecutor(max_workers=8) as executor:
        results = list(executor.map(embed_batch, batches))
    return [item for batch in results for item in batch]
```

## 三、内存管理

### 3.1 Python 内存管理机制

```
引用计数（主要机制）
    每个对象维护一个引用计数器
    引用 +1：赋值、传参、加入容器
    引用 -1：变量离开作用域、del、容器移除
    计数 = 0 → 立即释放

    ┌──────────────────────────────┐
    │ 引用计数无法处理循环引用       │
    │                               │
    │ a → b → a  （互相引用）       │
    │ 即使外部不再引用 a 和 b       │
    │ 它们的计数都不为 0            │
    │ → 内存泄漏！                  │
    └──────────────┬───────────────┘
                   │ 解决
                   ▼
分代垃圾回收（辅助机制）
    第0代：新创建的对象（频繁扫描）
    第1代：存活了一次 GC 的对象
    第2代：存活了多次 GC 的对象（很少扫描）

    触发条件：第0代对象数量达到阈值
```

### 3.2 常见内存泄漏场景

```python
# 场景1：全局缓存无限增长
class BadCache:
    _cache = {}  # 只加不删 → OOM

    @classmethod
    def get(cls, key):
        if key not in cls._cache:
            cls._cache[key] = expensive_compute(key)
        return cls._cache[key]

# 修复：用 LRU 缓存
from functools import lru_cache

@lru_cache(maxsize=1024)  # 自动淘汰最少使用的
def get_cached(key):
    return expensive_compute(key)

# 场景2：循环引用
class Node:
    def __init__(self):
        self.parent = None
        self.children = []

    def add_child(self, child):
        child.parent = self  # 循环引用！
        self.children.append(child)

# 修复：用 weakref
import weakref

class Node:
    def __init__(self):
        self._parent_ref = None
        self.children = []

    @property
    def parent(self):
        return self._parent_ref() if self._parent_ref else None

    @parent.setter
    def parent(self, node):
        self._parent_ref = weakref.ref(node)  # 弱引用，不增加计数

# 场景3：大模型推理时的显存泄漏
# PyTorch 中忘记 .detach() 或 .item()
loss = model(inputs)  # 计算图保留
# 修复：
loss = model(inputs).item()  # 只取数值，释放计算图
```

### 3.3 内存分析工具

```python
# 1. 内存分析
import tracemalloc

tracemalloc.start()
# ... 运行代码 ...
snapshot = tracemalloc.take_snapshot()
top_stats = snapshot.statistics('lineno')
for stat in top_stats[:10]:
    print(stat)

# 2. 对象引用追踪
import gc
gc.collect()
for obj in gc.get_objects():
    if isinstance(obj, MyLeakingClass):
        print(f"泄漏对象: {gc.get_referrers(obj)}")

# 3. 线下分析
# pip install memory-profiler
from memory_profiler import profile

@profile
def my_function():
    ...
```

## 四、设计模式——Python 风格

### 4.1 AI 项目常用设计模式

| 模式 | 用途 | AI 项目场景 |
|------|------|-----------|
| **工厂模式** | 创建对象 | 根据配置创建不同的 LLM 实例 |
| **策略模式** | 算法切换 | 切换不同的检索/分块策略 |
| **观察者模式** | 事件通知 | Agent 执行步骤的通知 |
| **单例模式** | 全局唯一 | LLM 服务/配置管理 |
| **装饰器模式** | 功能增强 | 给工具加缓存/重试/日志 |

### 4.2 工厂模式

```python
from abc import ABC, abstractmethod
from typing import Dict, Type

class LLMProvider(ABC):
    @abstractmethod
    def generate(self, prompt: str) -> str: ...

class OpenAIProvider(LLMProvider):
    def generate(self, prompt: str) -> str:
        return call_openai(prompt)

class AnthropicProvider(LLMProvider):
    def generate(self, prompt: str) -> str:
        return call_anthropic(prompt)

class LLMFactory:
    _providers: Dict[str, Type[LLMProvider]] = {}

    @classmethod
    def register(cls, name: str, provider: Type[LLMProvider]):
        cls._providers[name] = provider

    @classmethod
    def create(cls, name: str, **kwargs) -> LLMProvider:
        if name not in cls._providers:
            raise ValueError(f"Unknown provider: {name}")
        return cls._providers[name](**kwargs)

# 注册
LLMFactory.register("openai", OpenAIProvider)
LLMFactory.register("anthropic", AnthropicProvider)

# 使用
llm = LLMFactory.create("openai", model="gpt-4")
result = llm.generate("Hello")
```

### 4.3 策略模式

```python
from abc import ABC, abstractmethod
from typing import List

class ChunkStrategy(ABC):
    @abstractmethod
    def chunk(self, text: str) -> List[str]: ...

class FixedSizeChunker(ChunkStrategy):
    def __init__(self, chunk_size=512, overlap=50):
        self.chunk_size = chunk_size
        self.overlap = overlap

    def chunk(self, text: str) -> List[str]:
        tokens = tokenize(text)
        chunks = []
        for i in range(0, len(tokens), self.chunk_size - self.overlap):
            chunks.append(detokenize(tokens[i:i + self.chunk_size]))
        return chunks

class SemanticChunker(ChunkStrategy):
    def chunk(self, text: str) -> List[str]:
        # 基于语义相似度切分
        ...

# 使用
class RAGPipeline:
    def __init__(self, chunk_strategy: ChunkStrategy):
        self.chunker = chunk_strategy

    def process(self, document):
        chunks = self.chunker.chunk(document)
        ...

# 灵活切换
pipeline = RAGPipeline(FixedSizeChunker(chunk_size=256))
pipeline = RAGPipeline(SemanticChunker())  # 一行切换策略
```

## 五、性能优化

### 5.1 Profiling（性能分析）

```python
# 1. cProfile —— 找热点
import cProfile

cProfile.run('my_rag_pipeline(query)', sort='cumulative')

# 2. timeit —— 精确计时
import timeit
timeit.timeit('embedding_model.encode(["hello"])', number=100, globals=globals())

# 3. line_profiler —— 逐行分析
# pip install line_profiler
from line_profiler import profile

@profile
def process_documents(docs):
    for doc in docs:    # ← 这行花了 80% 时间？
        chunks = chunk(doc)
        embeddings = encode(chunks)
        store(embeddings)
```

### 5.2 常见优化技巧

```python
# 1. 用生成器替代列表（节省内存）
# 差
def load_all_docs(paths):
    return [open(p).read() for p in paths]  # 全部加载到内存

# 好
def load_docs_streaming(paths):
    for p in paths:
        yield open(p).read()  # 按需加载

# 2. 字典/集合查找 vs 列表查找
# O(n)
if item in my_list: ...

# O(1)
if item in my_set: ...

# 3. 字符串拼接
# 差（每次创建新字符串）
result = ""
for chunk in chunks:
    result += chunk

# 好
result = "".join(chunks)

# 4. NumPy 批量操作替代循环
# 差
results = [cosine_similarity(a, b) for a, b in zip(A, B)]

# 好
results = np.sum(A * B, axis=1) / (np.linalg.norm(A, axis=1) * np.linalg.norm(B, axis=1))
```

### 5.3 Python 类型系统

```python
# 现代Python用类型注解提高代码质量和可维护性
from typing import Optional, Union, TypedDict, Protocol

# TypedDict
class ToolCall(TypedDict):
    name: str
    arguments: dict[str, Any]
    id: str

# Protocol（结构化类型）
class EmbeddingModel(Protocol):
    def encode(self, texts: list[str]) -> list[list[float]]: ...
    def encode_query(self, text: str) -> list[float]: ...

# 使用泛型
from typing import TypeVar, Generic

T = TypeVar('T')

class Cache(Generic[T]):
    def __init__(self, maxsize: int = 128):
        self._cache: dict[str, T] = {}
        self._maxsize = maxsize

    def get(self, key: str) -> Optional[T]:
        return self._cache.get(key)

# 类型注解在AI项目中的价值：
# 1. 更好的IDE补全和错误检测
# 2. 代码即文档
# 3. 团队协作更清晰
```

## 六、面试高频编程题

> [!success]- Q1：实现一个 LRU 缓存
> ```python
> from collections import OrderedDict
>
> class LRUCache:
>     def __init__(self, capacity: int):
>         self.cache = OrderedDict()
>         self.capacity = capacity
>
>     def get(self, key: str):
>         if key not in self.cache:
>             return None
>         self.cache.move_to_end(key)  # 移到末尾（最近使用）
>         return self.cache[key]
>
>     def put(self, key: str, value):
>         if key in self.cache:
>             self.cache.move_to_end(key)
>         self.cache[key] = value
>         if len(self.cache) > self.capacity:
>             self.cache.popitem(last=False)  # 淘汰最久未使用
> ```

> [!success]- Q2：实现一个异步限流器
> ```python
> import asyncio
> import time
>
> class AsyncRateLimiter:
>     def __init__(self, max_calls: int, period: float):
>         self.max_calls = max_calls
>         self.period = period
>         self.calls: list[float] = []
>         self._lock = asyncio.Lock()
>
>     async def acquire(self):
>         async with self._lock:
>             now = time.time()
>             # 清理过期记录
>             self.calls = [t for t in self.calls if now - t < self.period]
>
>             if len(self.calls) >= self.max_calls:
>                 # 等待最早的调用过期
>                 wait_time = self.period - (now - self.calls[0])
>                 await asyncio.sleep(wait_time)
>
>             self.calls.append(time.time())
>
> # 使用：限制 LLM API 调用频率
> limiter = AsyncRateLimiter(max_calls=10, period=60)
>
> async def call_api(prompt):
>     await limiter.acquire()
>     return await llm.generate(prompt)
> ```

> [!success]- Q3：Python 的 GIL 是什么？对 AI 项目有什么影响？
> **GIL（Global Interpreter Lock）**：
> - CPython 的全局互斥锁，确保同一时刻只有一个线程执行 Python 字节码
> - CPU 密集型：多线程无法利用多核（受 GIL 限制）
> - I/O 密集型：多线程/asyncio 有效（I/O 等待时释放 GIL）
> - **AI 项目的影响**：
>   - PyTorch 的张量计算在 C++ 层，会释放 GIL → 多线程有效
>   - 数据预处理（Python 代码）受 GIL 限制 → 用多进程
>   - API 调用（I/O）不受 GIL 限制 → 用 asyncio
>   - Python 3.13 开始实验性 no-GIL 模式（PEP 703）

## 关键要点回顾

1. **装饰器**是 AI 项目最常用的高级特性（缓存/重试/限流/日志）
2. **asyncio** 是 AI 服务端的核心——并发 API 调用、工具调用
3. **内存管理**：引用计数 + 分代 GC，注意循环引用和缓存膨胀
4. **设计模式**：工厂（创建LLM实例）、策略（切换分块/检索策略）最实用
5. **性能优化**：先 profile 找瓶颈，再优化——不要过早优化

## 深度扩展阅读

本篇为基础概览，以下三篇深度扩展笔记覆盖了面试所需的全部细节：

| 扩展笔记 | 核心内容 | 建议时间 |
|----------|---------|---------|
| [[raw/lessons/AI-Interview-Prep/05a-python-advanced\|05a Python 高级特性深入]] | 装饰器源码级（闭包原理+5 个完整模式）、元类 4 个实际应用（单例/注册表/ORM/接口检查）、描述符完整查找链+ORM 模拟、生成器底层（帧对象/yield from）、Pydantic v2 实战 | 2h |
| [[raw/lessons/AI-Interview-Prep/05b-python-concurrency\|05b Python 并发编程深入]] | GIL 机制（check interval/3.12+ per-interpreter/3.13 no-GIL）、线程安全 LLM 客户端完整代码、asyncio 事件循环原理+异步 RAG 管道、多进程 IPC、5 种 AI 并发模式完整架构 | 2h |
| [[raw/lessons/AI-Interview-Prep/05c-python-engineering\|05c Python 工程实践与性能优化]] | 内存管理深入（pymalloc 三级结构）、性能优化 8 个模式 before/after 代码、6 种设计模式 AI 项目实战代码、断路器模式、8 道面试编程题完整解答 | 2h |

## 导航

← [[raw/lessons/AI-Interview-Prep/04-memory|第4站：AI Memory 记忆系统]]
→ 回到 [[raw/lessons/AI-Interview-Prep/00-overview|总览页]] 制定复习计划
