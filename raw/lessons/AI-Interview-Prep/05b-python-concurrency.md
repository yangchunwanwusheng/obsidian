---
type: lesson
tags: [面试, Python, 并发, asyncio, 多线程, 多进程, GIL]
created: 2026-05-20
updated: 2026-05-20
difficulty: advanced
prerequisites: [Python基础, 操作系统基础]
topic: Python并发编程深入
status: in-progress
series: {name: "AI Interview Prep", part: "5b"}
---

# Python 并发编程深入：GIL、多线程、asyncio、多进程

> 面向 AI 工程师面试，深入 Python 并发底层机制。涵盖 GIL 本质、asyncio 事件循环、多进程通信、以及 AI 场景下的并发架构设计。

## 学习目标

- [ ] 理解 GIL 的工作机制，能正确选择多线程/多进程/异步
- [ ] 掌握 asyncio 事件循环原理，能设计异步 AI 管道
- [ ] 掌握进程间通信机制，能实现多进程数据预处理
- [ ] 能设计高并发 LLM 推理服务架构

## 一、GIL（全局解释器锁）深入

### 1.1 GIL 存在的原因

CPython 的内存管理基于引用计数，引用计数的增减不是原子操作。如果没有 GIL，多线程并发修改引用计数可能导致内存泄漏或提前释放。

```python
import sys
a = []
b = a  # refcount = 2
# 线程A: 获取 a，refcount += 1
# 线程B: 获取 a，refcount += 1
# 如果没有锁，两个线程可能读到相同的旧值，最终只加了 1 → 内存泄漏

sys.getrefcount(a)  # 返回引用计数
```

GIL 是一把全局互斥锁，任何线程要执行 Python 字节码都必须先获取 GIL。这意味着在任意时刻，只有一个线程在执行 Python 代码。

### 1.2 GIL 的切换机制

在 Python 3.2+（Antoine Pitrou 的 GIL 重构），GIL 切换基于时间而非字节码指令数：

```
GIL 切换流程：
1. 线程 A 获取 GIL，开始执行
2. 线程 A 设置一个超时（默认 5ms，由 sys.getswitchinterval() 控制）
3. 超时后，线程 A 被强制挂起，释放 GIL
4. 其他等待线程竞争 GIL（通过 condition variable）
5. 获胜的线程获取 GIL，继续执行
```

```python
import sys
print(sys.getswitchinterval())  # 0.005（5 毫秒）
sys.setswitchinterval(0.01)     # 设置为 10 毫秒
```

### 1.3 GIL 对不同操作的影响

```python
import time
import threading

# 场景1：CPU 密集型 — GIL 是瓶颈
def cpu_bound(n):
    total = 0
    for i in range(n):
        total += i * i
    return total

# 单线程
start = time.time()
cpu_bound(10_000_000)
cpu_bound(10_000_000)
print(f"单线程: {time.time() - start:.2f}s")

# 多线程（不会加速，甚至可能更慢）
start = time.time()
t1 = threading.Thread(target=cpu_bound, args=(10_000_000,))
t2 = threading.Thread(target=cpu_bound, args=(10_000_000,))
t1.start(); t2.start()
t1.join(); t2.join()
print(f"多线程: {time.time() - start:.2f}s")  # 通常比单线程慢

# 场景2：I/O 密集型 — GIL 在 I/O 等待时释放，多线程有效
def io_bound(url):
    import requests
    return requests.get(url).status_code

# 多线程可以显著加速 I/O 操作

# 场景3：NumPy/C 扩展 — 可以显式释放 GIL
import numpy as np
# NumPy 的底层 C 实现在执行数组运算时释放 GIL
a = np.random.rand(10000, 10000)
b = np.random.rand(10000, 10000)
# 多线程可以并行执行 np.dot(a, b)
```

### 1.4 Python 3.12+ 的 GIL 改进

Python 3.12 引入了 per-interpreter GIL（PEP 684），允许每个子解释器拥有独立的 GIL：

```python
# Python 3.12+ 使用子解释器实现真正的并行
import interpreters
interp = interpreters.create()
interp.run("print('Hello from sub-interpreter')")
```

Python 3.13 实现了 PEP 703（Free-threaded Python），提供实验性的 no-GIL 模式：

```bash
# 编译 no-GIL 版本
./configure --disable-gil
make

# 或使用 free-threaded 构建标志
python3.13t  # "t" 后缀表示 free-threaded
```

> [!hint]- 面试题：PyTorch 推理服务用多线程还是多进程？
> 考虑 GIL 的影响，PyTorch 推理服务该如何选择并发模型？

> [!success]- 参考答案
> **答案是：取决于推理规模和部署架构。**
>
> 1. **单次推理、高并发请求**：多线程足够。PyTorch 的 C++ backend 在执行 tensor 运算时会释放 GIL，所以 Python 层的多线程可以真正并行。FastAPI + 线程池 或 asyncio + 线程池是常见方案。
>
> 2. **批量推理（Batch Inference）**：单进程 + CUDA 流。GPU 已天然并行，Python 多线程只负责提交任务。用 `torch.cuda.Stream` 管理并发。
>
> 3. **多模型部署**：多进程。每个进程加载一个模型到不同 GPU（或同一 GPU 的不同内存区域），避免 GIL 竞争和 CUDA 上下文切换开销。
>
> 4. **CPU 推理 + ONNX Runtime**：多进程。ONNX Runtime 的 CPU 推理会占用 Python CPU 时间，受 GIL 影响，多进程更可靠。
>
> ```python
> # 典型方案：FastAPI + 多进程
> # uvicorn server:app --workers 4
> # 每个 worker 加载一份模型
>
> from fastapi import FastAPI
> import torch
>
> app = FastAPI()
> model = None  # lazy load
>
> @app.on_event("startup")
> def load_model():
>     global model
>     model = torch.jit.load("model.pt")
>
> @app.post("/predict")
> async def predict(input_data: dict):
>     # PyTorch 推理会释放 GIL，asyncio 可以并发处理
>     loop = asyncio.get_event_loop()
>     result = await loop.run_in_executor(
>         None,  # 默认 ThreadPoolExecutor
>         lambda: model(torch.tensor(input_data["features"]))
>     )
>     return {"prediction": result.tolist()}
> ```

---

## 二、多线程深入

### 2.1 线程同步原语对比

```python
import threading

# Lock：基础互斥锁，不可重入
lock = threading.Lock()
with lock:
    critical_section()

# RLock：可重入锁，同一线程可以多次 acquire
rlock = threading.RLock()
def recursive_func(depth):
    with rlock:  # 同一线程可以多次进入
        if depth > 0:
            recursive_func(depth - 1)

# Semaphore：限流
sem = threading.Semaphore(5)  # 最多 5 个线程同时执行
with sem:
    limited_resource()

# Event：一次性信号通知
event = threading.Event()
# 等待方
event.wait()  # 阻塞直到 set()
# 通知方
event.set()

# Condition：生产者-消费者
cond = threading.Condition()
shared_data = []

def producer():
    with cond:
        shared_data.append("item")
        cond.notify()  # 通知一个等待的消费者

def consumer():
    with cond:
        while not shared_data:
            cond.wait()  # 释放锁并等待通知
        item = shared_data.pop()
```

### 2.2 生产者-消费者完整实现

```python
import threading
import queue
import time

class BoundedProducerConsumer:
    """线程安全的有界生产者-消费者模式"""
    def __init__(self, maxsize=100):
        self.queue = queue.Queue(maxsize=maxsize)
        self._running = True
        self._producers = []
        self._consumers = []

    def add_producer(self, func, count=1):
        for _ in range(count):
            t = threading.Thread(target=self._produce, args=(func,), daemon=True)
            self._producers.append(t)

    def add_consumer(self, func, count=1):
        for _ in range(count):
            t = threading.Thread(target=self._consume, args=(func,), daemon=True)
            self._consumers.append(t)

    def _produce(self, func):
        while self._running:
            item = func()
            if item is None:
                break
            self.queue.put(item)  # 队列满时自动阻塞

    def _consume(self, func):
        while self._running:
            try:
                item = self.queue.get(timeout=1.0)
                func(item)
                self.queue.task_done()
            except queue.Empty:
                continue

    def start(self):
        for t in self._producers + self._consumers:
            t.start()

    def stop(self):
        self._running = False
        for t in self._producers + self._consumers:
            t.join(timeout=5.0)

# 使用：并发 Embedding 生成
def produce_texts():
    for doc in documents:
        yield doc

pc = BoundedProducerConsumer(maxsize=50)
pc.add_producer(lambda: next(produce_texts(), None), count=2)
pc.add_consumer(lambda doc: save_embedding(embed(doc)), count=4)
pc.start()
```

### 2.3 ThreadPoolExecutor 最佳实践

```python
from concurrent.futures import ThreadPoolExecutor, as_completed
import threading

class LLMAPIClient:
    """线程安全的 LLM API 客户端"""
    def __init__(self, api_key: str, max_workers=10, rate_limit=5):
        self.api_key = api_key
        self.session = requests.Session()
        self.session.headers.update({"Authorization": f"Bearer {api_key}"})
        self._lock = threading.Lock()
        self._semaphore = threading.Semaphore(rate_limit)
        self._executor = ThreadPoolExecutor(
            max_workers=max_workers,
            thread_name_prefix="llm_api"
        )

    def _call_with_retry(self, prompt: str, max_retries=3) -> dict:
        for attempt in range(max_retries):
            with self._semaphore:  # 限流
                try:
                    resp = self.session.post(
                        "https://api.openai.com/v1/chat/completions",
                        json={"model": "gpt-4", "messages": [{"role": "user", "content": prompt}]},
                        timeout=30
                    )
                    resp.raise_for_status()
                    return resp.json()
                except requests.RequestException as e:
                    if attempt == max_retries - 1:
                        raise
                    time.sleep(2 ** attempt)

    def batch_call(self, prompts: list[str]) -> list[dict]:
        """并发调用 LLM API"""
        futures = {
            self._executor.submit(self._call_with_retry, p): i
            for i, p in enumerate(prompts)
        }
        results = [None] * len(prompts)
        for future in as_completed(futures):
            idx = futures[future]
            try:
                results[idx] = future.result()
            except Exception as e:
                results[idx] = {"error": str(e)}
        return results

    def shutdown(self):
        self._executor.shutdown(wait=True)
```

### 2.4 线程局部存储

```python
import threading

# 每个线程独立的存储空间
local = threading.local()

def process_request(request_id):
    local.request_id = request_id  # 只在当前线程可见
    local.start_time = time.time()
    # 在当前线程的任意函数中都可以访问 local.request_id
    do_work()

def do_work():
    print(f"处理请求 {local.request_id}")

threads = [threading.Thread(target=process_request, args=(i,)) for i in range(5)]
for t in threads:
    t.start()
```

### 2.5 死锁场景与避免

```python
# 经典死锁：两个线程互相等待对方持有的锁
lock_a = threading.Lock()
lock_b = threading.Lock()

def task1():
    with lock_a:           # 先获取 A
        time.sleep(0.01)
        with lock_b:       # 再获取 B → 如果 B 已被 task2 持有，死锁
            pass

def task2():
    with lock_b:           # 先获取 B
        time.sleep(0.01)
        with lock_a:       # 再获取 A → 如果 A 已被 task1 持有，死锁
            pass

# 避免策略：
# 1. 使用 RLock（但只在同一线程重入时有效）
# 2. 使用超时：lock.acquire(timeout=5)
# 3. 使用层级锁（所有线程按相同顺序获取锁）
# 4. 使用 threading.Condition 代替多锁组合
# 5. 使用 queue.Queue 避免共享状态
```

> [!hint]- 面试题：如何实现一个线程安全的 Embedding 缓存？
> 要求：支持并发读写、LRU 淘汰、TTL 过期。多线程环境下缓存命中率不能下降。

> [!success]- 参考答案
> ```python
> import threading
> import time
> from collections import OrderedDict
> from typing import Optional
>
> class ThreadSafeEmbeddingCache:
>     def __init__(self, maxsize=10000, ttl=3600):
>         self.maxsize = maxsize
>         self.ttl = ttl
>         self._cache: OrderedDict[str, tuple[list, float]] = OrderedDict()
>         self._lock = threading.Lock()
>         self._hits = 0
>         self._misses = 0
>
>     def get(self, key: str) -> Optional[list]:
>         with self._lock:
>             if key not in self._cache:
>                 self._misses += 1
>                 return None
>             value, expire_at = self._cache[key]
>             if time.time() > expire_at:
>                 del self._cache[key]
>                 self._misses += 1
>                 return None
>             self._cache.move_to_end(key)  # LRU 更新
>             self._hits += 1
>             return value
>
>     def put(self, key: str, value: list) -> None:
>         with self._lock:
>             if key in self._cache:
>                 self._cache.move_to_end(key)
>             self._cache[key] = (value, time.time() + self.ttl)
>             while len(self._cache) > self.maxsize:
>                 self._cache.popitem(last=False)  # 淘汰最旧
>
>     def stats(self) -> dict:
>         total = self._hits + self._misses
>         return {
>             "size": len(self._cache),
>             "hits": self._hits,
>             "misses": self._misses,
>             "hit_rate": self._hits / total if total > 0 else 0.0,
>         }
>
> # 使用：多线程 Embedding 服务
> cache = ThreadSafeEmbeddingCache(maxsize=50000, ttl=1800)
>
> def get_embedding(text: str) -> list:
>     cached = cache.get(text)
>     if cached is not None:
>         return cached
>     embedding = call_embedding_api(text)
>     cache.put(text, embedding)
>     return embedding
> ```

---

## 三、asyncio 深入

### 3.1 事件循环工作原理

asyncio 的事件循环是一个单线程的任务调度器，核心流程：

```
事件循环的每次迭代：
1. 检查是否有已完成的 I/O 操作（通过 epoll/kqueue/IOCP）
2. 将已完成的 I/O 对应的 Future 标记为完成
3. 检查是否有到期的定时器
4. 从就绪队列中取出所有就绪的 Task
5. 依次执行每个 Task 直到它遇到 await 挂起
6. 回到步骤 1
```

```python
import asyncio

# 查看事件循环的底层选择器
loop = asyncio.get_event_loop()
print(type(loop._selector))
# Linux: EpollSelector（epoll）
# macOS: KqueueSelector（kqueue）
# Windows: ProactorEventLoop（IOCP）

# 事件循环的内部结构（简化）
class EventLoop:
    def __init__(self):
        self._ready = deque()      # 就绪队列
        self._scheduled = []       # 定时器堆
        self._selector = SelectSelector()  # I/O 多路复用

    def run_once(self):
        # 1. 处理 I/O 就绪事件
        ready_io = self._selector.select(timeout=0)
        for key, events in ready_io:
            callback = key.data
            self._ready.append(callback)

        # 2. 处理到期定时器
        while self._scheduled and self._scheduled[0] <= now:
            handle = heapq.heappop(self._scheduled)
            self._ready.append(handle)

        # 3. 执行就绪任务
        ntodo = len(self._ready)
        for _ in range(ntodo):
            handle = self._ready.popleft()
            handle._run()  # 执行 Task 直到下一个 await
```

### 3.2 核心概念区分

**Coroutine vs Task vs Future**

```python
import asyncio

# Coroutine（协程函数的返回值）：不可直接调度
async def my_coroutine():
    await asyncio.sleep(1)
    return "done"

coro = my_coroutine()  # 这是一个 coroutine 对象，还不是 Task

# Task（任务的封装）：可以被事件循环调度
task = asyncio.create_task(coro)  # 立即包装为 Task 并加入事件循环

# Future（未来的结果）：Task 的基类
# Task = Coroutine 的包装 + Future 的结果容器
assert isinstance(task, asyncio.Future)  # True

# Task 的状态机
print(task.done())  # False
await task
print(task.done())  # True
print(task.result())  # "done"
```

**并发组合器对比**

```python
async def fetch(url):
    await asyncio.sleep(1)
    return f"response from {url}"

urls = ["http://a.com", "http://b.com", "http://c.com"]

# gather：等待全部完成，保持顺序
results = await asyncio.gather(*[fetch(u) for u in urls])
# results = ["response from http://a.com", ...]

# TaskGroup（Python 3.11+）：结构化并发，任一异常取消所有任务
async with asyncio.TaskGroup() as tg:
    tasks = [tg.create_task(fetch(u)) for u in urls]
results = [t.result() for t in tasks]

# as_completed：流式处理，谁先完成谁先返回
for coro in asyncio.as_completed([fetch(u) for u in urls]):
    result = await coro
    print(result)  # 按完成顺序输出

# wait：更灵活的等待策略
done, pending = await asyncio.wait(
    [asyncio.create_task(fetch(u)) for u in urls],
    timeout=2.0,
    return_when=asyncio.FIRST_COMPLETED  # 或 ALL_COMPLETED, FIRST_EXCEPTION
)
for task in done:
    print(task.result())
for task in pending:
    task.cancel()
```

### 3.3 异步上下文管理器和迭代器

```python
import aiohttp
import asyncio

# 异步上下文管理器
class DBConnection:
    async def __aenter__(self):
        self.conn = await create_connection()
        return self.conn

    async def __aexit__(self, exc_type, exc_val, exc_tb):
        await self.conn.close()

async def main():
    async with DBConnection() as conn:
        await conn.execute("SELECT 1")

# 异步迭代器
class AsyncLineReader:
    def __init__(self, filepath):
        self.filepath = filepath

    async def __aiter__(self):
        self._file = await aiofiles.open(self.filepath)
        return self

    async def __anext__(self):
        line = await self._file.readline()
        if not line:
            await self._file.close()
            raise StopAsyncIteration
        return line.rstrip()

async def process_file():
    async for line in AsyncLineReader("data.txt"):
        await process(line)
```

### 3.4 完整示例：异步 RAG 管道

```python
import asyncio
import aiohttp
from typing import AsyncIterator

class AsyncRAGPipeline:
    """异步 RAG 管道：并发 Embedding + 并发检索"""
    def __init__(self, embedding_url: str, retrieval_url: str,
                 llm_url: str, max_concurrent=10):
        self.embedding_url = embedding_url
        self.retrieval_url = retrieval_url
        self.llm_url = llm_url
        self._semaphore = asyncio.Semaphore(max_concurrent)

    async def _post(self, session: aiohttp.ClientSession,
                     url: str, payload: dict) -> dict:
        async with self._semaphore:
            async with session.post(url, json=payload) as resp:
                resp.raise_for_status()
                return await resp.json()

    async def embed_batch(self, session, texts: list[str]) -> list[list]:
        """并发批量 Embedding"""
        tasks = [self._post(session, self.embedding_url, {"text": t})
                 for t in texts]
        results = await asyncio.gather(*tasks)
        return [r["embedding"] for r in results]

    async def retrieve(self, session, query_embedding: list,
                        top_k: int = 5) -> list[dict]:
        """异步检索"""
        return await self._post(session, self.retrieval_url, {
            "query_embedding": query_embedding,
            "top_k": top_k,
        })

    async def generate(self, session, query: str,
                        contexts: list[str]) -> str:
        """异步生成"""
        prompt = f"Context:\n{''.join(contexts)}\n\nQuestion: {query}"
        result = await self._post(session, self.llm_url, {
            "prompt": prompt,
            "max_tokens": 512,
        })
        return result["text"]

    async def query(self, query: str, top_k: int = 5) -> str:
        """完整的 RAG 查询流程"""
        async with aiohttp.ClientSession() as session:
            # Step 1: Embed query
            embeddings = await self.embed_batch(session, [query])
            query_embedding = embeddings[0]

            # Step 2: Retrieve contexts
            retrieval_result = await self.retrieve(
                session, query_embedding, top_k
            )
            contexts = [doc["text"] for doc in retrieval_result["documents"]]

            # Step 3: Generate answer
            answer = await self.generate(session, query, contexts)
            return answer

    async def batch_query(self, queries: list[str]) -> list[str]:
        """并发处理多个查询"""
        async with aiohttp.ClientSession() as session:
            tasks = [self.query(q) for q in queries]
            return await asyncio.gather(*tasks)

# 使用
pipeline = AsyncRAGPipeline(
    embedding_url="http://localhost:8001/embed",
    retrieval_url="http://localhost:8002/retrieve",
    llm_url="http://localhost:8003/generate",
    max_concurrent=20,
)

async def main():
    answers = await pipeline.batch_query([
        "What is transformer?",
        "How does attention work?",
        "Explain positional encoding.",
    ])
    for q, a in zip(queries, answers):
        print(f"Q: {q}\nA: {a}\n")
```

> [!hint]- 面试题：处理 1000 个并发 LLM 请求，如何设计异步架构？
> 要求：支持限流（不超过 API 速率限制）、超时控制、失败重试、结果按请求顺序返回。

> [!success]- 参考答案
> ```python
> import asyncio
> import time
> from typing import Any
>
> class ConcurrentUserver:
>     """高并发 LLM 请求处理器"""
>     def __init__(self, max_concurrent=50, rate_limit=10,
>                  timeout=30, max_retries=3):
>         self._concurrency = asyncio.Semaphore(max_concurrent)
>         self._rate_limiter = asyncio.Semaphore(rate_limit)
>         self._timeout = timeout
>         self._max_retries = max_retries
>         self._request_times: list[float] = []
>         self._lock = asyncio.Lock()
>
>     async def _wait_for_rate_limit(self):
>         """滑动窗口限流"""
>         async with self._lock:
>             now = time.time()
>             self._request_times = [
>                 t for t in self._request_times if now - t < 1.0
>             ]
>             if len(self._request_times) >= 10:
>                 wait = 1.0 - (now - self._request_times[0])
>                 await asyncio.sleep(max(0, wait))
>             self._request_times.append(time.time())
>
>     async def _call_llm(self, session, prompt: str) -> dict:
>         for attempt in range(self._max_retries):
>             try:
>                 await self._wait_for_rate_limit()
>                 async with asyncio.timeout(self._timeout):
>                     async with session.post(
>                         "https://api.openai.com/v1/chat/completions",
>                         json={"model": "gpt-4", "messages": [
>                             {"role": "user", "content": prompt}
>                         ]}
>                     ) as resp:
>                         resp.raise_for_status()
>                         return await resp.json()
>             except (asyncio.TimeoutError, aiohttp.ClientError) as e:
>                 if attempt == self._max_retries - 1:
>                     return {"error": str(e)}
>                 await asyncio.sleep(0.5 * (2 ** attempt))
>
>     async def _process_one(self, session, prompt: str) -> dict:
>         async with self._concurrency:
>             return await self._call_llm(session, prompt)
>
>     async def process_batch(self, prompts: list[str]) -> list[dict]:
>         """按顺序返回结果"""
>         async with aiohttp.ClientSession() as session:
>             # 使用 gather 保证顺序
>             tasks = [self._process_one(session, p) for p in prompts]
>             return await asyncio.gather(*tasks)
>
> # 使用
> server = ConcurrentUserver(max_concurrent=50, rate_limit=10)
> prompts = [f"Explain concept {i}" for i in range(1000)]
> results = await server.process_batch(prompts)
> ```

---

## 四、多进程深入

### 4.1 多进程 vs 多线程决策树

```
任务类型？
├─ I/O 密集型（网络请求、文件读写）
│   └─ asyncio（首选）或 多线程
├─ CPU 密集型（数值计算、数据处理）
│   ├─ 数据量小、需要频繁通信
│   │   └─ multiprocessing + shared_memory
│   ├─ 数据量大、任务独立
│   │   └─ multiprocessing.Pool / ProcessPoolExecutor
│   └─ 需要 GPU 加速
│       └─ 每个进程独占一个 GPU
└─ 混合型
    └─ 多进程（CPU） + asyncio（I/O）组合使用
```

### 4.2 进程间通信（IPC）

```python
import multiprocessing as mp
from multiprocessing import shared_memory
import numpy as np
import struct

# 方式1：Queue（进程安全队列）
def producer(queue: mp.Queue):
    for i in range(100):
        queue.put({"id": i, "data": [1.0, 2.0, 3.0]})
    queue.put(None)  # 哨兵

def consumer(queue: mp.Queue, results: list):
    while True:
        item = queue.get()
        if item is None:
            break
        results.append(process(item))

q = mp.Queue()
p1 = mp.Process(target=producer, args=(q,))
p2 = mp.Process(target=consumer, args=(q, []))

# 方式2：Pipe（双向管道）
parent_conn, child_conn = mp.Pipe()
def worker(conn):
    data = conn.recv()
    result = process(data)
    conn.send(result)

# 方式3：Shared Memory（Python 3.8+）—— 零拷贝
def worker_shared(shm_name, shape):
    # 附加到已有的共享内存
    shm = shared_memory.SharedMemory(name=shm_name)
    arr = np.ndarray(shape, dtype=np.float32, buffer=shm.buf)
    result = arr * 2  # 直接操作共享内存
    shm.close()
    return result

# 创建共享内存
data = np.random.rand(1000, 768).astype(np.float32)
shm = shared_memory.SharedMemory(create=True, size=data.nbytes)
shared_arr = np.ndarray(data.shape, dtype=np.float32, buffer=shm.buf)
np.copyto(shared_arr, data)

# 启动进程
p = mp.Process(target=worker_shared, args=(shm.name, data.shape))
p.start()
p.join()
shm.close()
shm.unlink()  # 清理

# 方式4：Manager（代理对象，适合复杂结构）
manager = mp.Manager()
shared_dict = manager.dict()
shared_list = manager.list()

def worker(d, l):
    d["key"] = "value"
    l.append(1)

p = mp.Process(target=worker, args=(shared_dict, shared_list))
```

### 4.3 多进程数据预处理管道

```python
import multiprocessing as mp
from concurrent.futures import ProcessPoolExecutor
from typing import Iterator
import json

def preprocess_document(doc: dict) -> dict:
    """CPU 密集型：文本预处理"""
    text = doc["text"]
    # 清洗、分词、特征提取
    tokens = tokenize(text)
    features = extract_features(tokens)
    return {"id": doc["id"], "features": features, "tokens": tokens}

class DataPreprocessingPipeline:
    """多进程数据预处理管道"""
    def __init__(self, num_workers=None, chunk_size=100):
        self.num_workers = num_workers or mp.cpu_count()
        self.chunk_size = chunk_size

    def _read_chunks(self, filepath: str) -> Iterator[list[dict]]:
        """流式读取，避免内存溢出"""
        chunk = []
        with open(filepath, 'r') as f:
            for line in f:
                chunk.append(json.loads(line))
                if len(chunk) == self.chunk_size:
                    yield chunk
                    chunk = []
        if chunk:
            yield chunk

    def process_file(self, input_path: str, output_path: str):
        """多进程处理大文件"""
        with ProcessPoolExecutor(max_workers=self.num_workers) as executor:
            with open(output_path, 'w') as out:
                for chunk in self._read_chunks(input_path):
                    # 并行处理每个 chunk
                    results = list(executor.map(preprocess_document, chunk))
                    for result in results:
                        out.write(json.dumps(result) + '\n')

    def process_with_shared_memory(self, data: np.ndarray,
                                    output: np.ndarray):
        """使用共享内存避免序列化开销"""
        shm_in = shared_memory.SharedMemory(create=True, size=data.nbytes)
        shm_out = shared_memory.SharedMemory(create=True, size=output.nbytes)

        arr_in = np.ndarray(data.shape, dtype=data.dtype, buffer=shm_in.buf)
        arr_out = np.ndarray(output.shape, dtype=output.dtype, buffer=shm_out.buf)
        np.copyto(arr_in, data)

        chunk_size = len(data) // self.num_workers
        processes = []
        for i in range(self.num_workers):
            start = i * chunk_size
            end = start + chunk_size if i < self.num_workers - 1 else len(data)
            p = mp.Process(
                target=self._process_chunk,
                args=(shm_in.name, shm_out.name, data.shape, output.shape,
                      start, end)
            )
            processes.append(p)
            p.start()

        for p in processes:
            p.join()

        np.copyto(output, arr_out)
        shm_in.close(); shm_in.unlink()
        shm_out.close(); shm_out.unlink()

# 使用
pipeline = DataPreprocessingPipeline(num_workers=8)
pipeline.process_file("raw_data.jsonl", "processed_data.jsonl")
```

> [!hint]- 面试题：如何加速 CPU 密集型的 Embedding 计算？
> 有 100 万条文本需要生成 Embedding，使用本地模型（无 GPU），如何最大化 CPU 利用率？

> [!success]- 参考答案
> ```python
> import multiprocessing as mp
> from multiprocessing import shared_memory
> import numpy as np
> from typing import Generator
>
> def compute_embeddings_chunk(texts: list[str], model_name: str,
>                               start_idx: int) -> tuple[int, np.ndarray]:
>     """每个进程独立加载模型，处理一批文本"""
>     from sentence_transformers import SentenceTransformer
>     model = SentenceTransformer(model_name)
>     embeddings = model.encode(texts, batch_size=32, show_progress_bar=False)
>     return start_idx, embeddings
>
> def parallel_embed(texts: list[str], model_name: str,
>                     num_workers: int = None) -> np.ndarray:
>     num_workers = num_workers or mp.cpu_count()
>     total = len(texts)
>     chunk_size = (total + num_workers - 1) // num_workers
>
>     # 准备参数
>     tasks = []
>     for i in range(num_workers):
>         start = i * chunk_size
>         end = min(start + chunk_size, total)
>         if start < end:
>             tasks.append((texts[start:end], model_name, start))
>
>     # 使用进程池
>     all_embeddings = np.empty((total, 768), dtype=np.float32)
>
>     with mp.Pool(processes=num_workers) as pool:
>         results = pool.starmap(compute_embeddings_chunk, tasks)
>         for start_idx, embeddings in results:
>             all_embeddings[start_idx:start_idx + len(embeddings)] = embeddings
>
>     return all_embeddings
>
> # 进一步优化：
> # 1. 使用共享内存避免大数据的 pickle 序列化
> # 2. 设置 OMP_NUM_THREADS 避免每个进程创建太多线程
> # 3. 使用 spawn 而非 fork 避免内存复制
> # 4. 对于非常大的数据集，使用生产者-消费者模式流式处理
>
> import os
> os.environ["OMP_NUM_THREADS"] = str(max(1, mp.cpu_count() // num_workers))
>
> if __name__ == "__main__":
>     mp.set_start_method("spawn")  # 避免死锁
>     texts = ["Hello world"] * 1_000_000
>     embeddings = parallel_embed(texts, "all-MiniLM-L6-v2", num_workers=8)
>     print(embeddings.shape)  # (1000000, 384)
> ```

---

## 五、并发模式在 AI 项目中的应用

### 模式1：异步 API 网关

```python
from fastapi import FastAPI
from pydantic import BaseModel
import asyncio
import aiohttp

app = FastAPI()

class ChatRequest(BaseModel):
    messages: list[dict]
    model: str = "gpt-4"
    temperature: float = 0.7

class ChatResponse(BaseModel):
    content: str
    model: str
    usage: dict

# 全局连接池（复用 TCP 连接）
_session: aiohttp.ClientSession | None = None

@app.on_event("startup")
async def startup():
    global _session
    _session = aiohttp.ClientSession(
        timeout=aiohttp.ClientTimeout(total=60),
        connector=aiohttp.TCPConnector(limit=100)  # 连接池上限
    )

@app.on_event("shutdown")
async def shutdown():
    await _session.close()

@app.post("/chat", response_model=ChatResponse)
async def chat(request: ChatRequest):
    async with _session.post(
        "https://api.openai.com/v1/chat/completions",
        json=request.model_dump(),
        headers={"Authorization": f"Bearer {API_KEY}"}
    ) as resp:
        data = await resp.json()
    return ChatResponse(
        content=data["choices"][0]["message"]["content"],
        model=data["model"],
        usage=data["usage"]
    )

# 启动：uvicorn app:app --workers 4 --port 8000
```

### 模式2：流式响应（SSE）

```python
from fastapi import FastAPI
from fastapi.responses import StreamingResponse
import aiohttp
import json

app = FastAPI()

@app.post("/chat/stream")
async def chat_stream(request: ChatRequest):
    async def generate():
        async with _session.post(
            "https://api.openai.com/v1/chat/completions",
            json={**request.model_dump(), "stream": True},
            headers={"Authorization": f"Bearer {API_KEY}"}
        ) as resp:
            async for line in resp.content:
                line = line.decode("utf-8").strip()
                if line.startswith("data: "):
                    data = line[6:]
                    if data == "[DONE]":
                        yield f"data: [DONE]\n\n"
                        break
                    chunk = json.loads(data)
                    content = chunk["choices"][0].get("delta", {}).get("content", "")
                    if content:
                        yield f"data: {json.dumps({'content': content})}\n\n"

    return StreamingResponse(generate(), media_type="text/event-stream")
```

### 模式3：并行工具调用

```python
import asyncio
from typing import Any

class ToolExecutor:
    """Agent 的并行工具执行器"""
    def __init__(self, max_concurrent=5):
        self._semaphore = asyncio.Semaphore(max_concurrent)
        self._tools: dict[str, callable] = {}

    def register(self, name: str, func):
        self._tools[name] = func

    async def execute_one(self, tool_call: dict) -> dict:
        name = tool_call["name"]
        args = tool_call["arguments"]
        async with self._semaphore:
            func = self._tools[name]
            if asyncio.iscoroutinefunction(func):
                result = await func(**args)
            else:
                result = await asyncio.to_thread(func, **args)
        return {"tool": name, "result": result}

    async def execute_parallel(self, tool_calls: list[dict]) -> list[dict]:
        """并行执行多个独立的工具调用"""
        tasks = [self.execute_one(tc) for tc in tool_calls]
        return await asyncio.gather(*tasks)

# 使用
executor = ToolExecutor(max_concurrent=5)
executor.register("web_search", search_web)
executor.register("code_execute", execute_code)
executor.register("calculator", calculate)

results = await executor.execute_parallel([
    {"name": "web_search", "arguments": {"query": "Python asyncio"}},
    {"name": "calculator", "arguments": {"expression": "2**10"}},
    {"name": "code_execute", "arguments": {"code": "print(42)"}},
])
```

### 模式4：批量推理请求的并发处理

```python
import asyncio
import aiohttp
import time
from dataclasses import dataclass

@dataclass
class BatchConfig:
    max_batch_size: int = 32
    max_wait_time: float = 0.1  # 秒
    max_concurrent_batches: int = 4

class DynamicBatcher:
    """动态批处理器：将单个请求自动组批"""
    def __init__(self, process_fn, config: BatchConfig):
        self._process_fn = process_fn
        self._config = config
        self._queue: asyncio.Queue = asyncio.Queue()
        self._semaphore = asyncio.Semaphore(config.max_concurrent_batches)

    async def submit(self, item: Any) -> Any:
        future = asyncio.get_event_loop().create_future()
        await self._queue.put((item, future))
        return await future

    async def run(self):
        while True:
            batch = []
            futures = []
            deadline = time.time() + self._config.max_wait_time

            while len(batch) < self._config.max_batch_size:
                timeout = max(0, deadline - time.time())
                try:
                    item, future = await asyncio.wait_for(
                        self._queue.get(), timeout=timeout
                    )
                    batch.append(item)
                    futures.append(future)
                except asyncio.TimeoutError:
                    break

            if batch:
                async with self._semaphore:
                    try:
                        results = await self._process_fn(batch)
                        for f, r in zip(futures, results):
                            f.set_result(r)
                    except Exception as e:
                        for f in futures:
                            if not f.done():
                                f.set_exception(e)

# 使用
async def batch_inference(items: list[str]) -> list[str]:
    """模拟批量推理"""
    async with aiohttp.ClientSession() as session:
        resp = await session.post(
            "http://localhost:8000/batch_predict",
            json={"inputs": items}
        )
        return (await resp.json())["results"]

batcher = DynamicBatcher(batch_inference, BatchConfig(
    max_batch_size=32, max_wait_time=0.05, max_concurrent_batches=4
))

async def main():
    asyncio.create_task(batcher.run())
    # 100 个并发请求会自动组批
    results = await asyncio.gather(*[
        batcher.submit(f"Input {i}") for i in range(100)
    ])
```

### 模式5：后台任务队列

```python
import asyncio
import json
from pathlib import Path

class SimpleTaskQueue:
    """轻量级异步任务队列（生产环境建议用 Celery/RQ）"""
    def __init__(self, num_workers=4):
        self._queue: asyncio.Queue = asyncio.Queue()
        self._num_workers = num_workers
        self._results: dict[str, Any] = {}
        self._running = False

    async def start(self):
        self._running = True
        workers = [
            asyncio.create_task(self._worker(f"worker-{i}"))
            for i in range(self._num_workers)
        ]
        await asyncio.gather(*workers)

    async def stop(self):
        self._running = False
        for _ in range(self._num_workers):
            await self._queue.put(None)

    async def submit(self, task_id: str, func, *args) -> str:
        await self._queue.put((task_id, func, args))
        return task_id

    async def get_result(self, task_id: str, timeout=300) -> Any:
        start = time.time()
        while time.time() - start < timeout:
            if task_id in self._results:
                result = self._results.pop(task_id)
                if isinstance(result, Exception):
                    raise result
                return result
            await asyncio.sleep(0.1)
        raise TimeoutError(f"Task {task_id} timed out")

    async def _worker(self, name: str):
        while self._running:
            item = await self._queue.get()
            if item is None:
                break
            task_id, func, args = item
            try:
                result = await func(*args) if asyncio.iscoroutinefunction(func) \
                         else func(*args)
                self._results[task_id] = result
            except Exception as e:
                self._results[task_id] = e

# 使用示例
queue = SimpleTaskQueue(num_workers=4)

async def embed_document(doc_id: str):
    # 模拟耗时操作
    await asyncio.sleep(2)
    return {"doc_id": doc_id, "embedding": [0.1] * 768}

async def main():
    worker_task = asyncio.create_task(queue.start())
    task_ids = [
        await queue.submit(f"doc-{i}", embed_document, f"doc-{i}")
        for i in range(20)
    ]
    results = await asyncio.gather(*[
        queue.get_result(tid) for tid in task_ids
    ])
    await queue.stop()
```

> [!hint]- 面试题：设计一个高并发的 LLM 推理服务
> 要求：支持 1000 QPS、流式和非流式两种模式、API 限流、健康检查、优雅关闭。

> [!success]- 参考答案
> ```
> 架构设计：
>
> Client → Nginx/LB → [FastAPI Worker × N] → DynamicBatcher → GPU Inference
>                                      ↓
>                              Redis (限流 + 缓存)
>                                      ↓
>                              Prometheus (监控)
>
> 关键设计点：
> 1. 多进程部署：uvicorn --workers $((CPU_CORES * 2 + 1))
> 2. 每个 Worker 内部使用 asyncio 处理 I/O 并发
> 3. DynamicBatcher 将请求组批，最大化 GPU 利用率
> 4. Redis 实现分布式限流（令牌桶）
> 5. 流式响应使用 SSE，非流式使用标准 JSON
> 6. 健康检查端点：/health（检测模型加载状态 + GPU 内存）
> 7. 优雅关闭：SIGTERM → 停止接收新请求 → 等待当前批次完成 → 退出
> ```
>
> ```python
> # 核心架构代码
> from fastapi import FastAPI, HTTPException
> from fastapi.responses import StreamingResponse
> from contextlib import asynccontextmanager
> import asyncio
> import signal
>
> _shutting_down = False
>
> @asynccontextmanager
> async def lifespan(app: FastAPI):
>     global model, batcher
>     model = load_model()
>     batcher = DynamicBatcher(model.batch_predict, BatchConfig(
>         max_batch_size=32, max_wait_time=0.05, max_concurrent_batches=4
>     ))
>     asyncio.create_task(batcher.run())
>     yield
>     await batcher.stop()
>     del model
>
> app = FastAPI(lifespan=lifespan)
>
> @app.get("/health")
> async def health():
>     if _shutting_down:
>         raise HTTPException(503, "Shutting down")
>     gpu_mem = torch.cuda.memory_allocated() / 1e9
>     return {"status": "ok", "gpu_memory_gb": f"{gpu_mem:.1f}"}
>
> @app.post("/v1/completions")
> async def completions(request: CompletionRequest):
>     if _shutting_down:
>         raise HTTPException(503, "Service shutting down")
>     if not await check_rate_limit(request.api_key):
>         raise HTTPException(429, "Rate limit exceeded")
>
>     if request.stream:
>         return StreamingResponse(
>             stream_generate(request),
>             media_type="text/event-stream"
>         )
>     else:
>         result = await batcher.submit(request.prompt)
>         return {"choices": [{"text": result}]}
>
> async def stream_generate(request):
>     async for token in model.stream_generate(request.prompt):
>         yield f"data: {json.dumps({'token': token})}\n\n"
>     yield "data: [DONE]\n\n"
>
> # 优雅关闭
> def handle_shutdown(signum, frame):
>     global _shutting_down
>     _shutting_down = True
>
> signal.signal(signal.SIGTERM, handle_shutdown)
> ```

## 关键要点回顾

1. **GIL 是 CPython 的全局锁**：CPU 密集型受限制，I/O 密集型自动释放，C 扩展可手动释放
2. **多线程适合 I/O 密集型**：网络请求、文件操作。注意死锁和线程安全
3. **asyncio 是单线程并发**：事件循环 + I/O 多路复用，适合大量 I/O 操作。学习曲线陡但性能优异
4. **多进程适合 CPU 密集型**：Embedding 计算、数据预处理。IPC 开销是关键瓶颈
5. **AI 系统通常组合使用**：多进程（模型推理）+ asyncio（API 网关）+ 批处理（GPU 利用率）

## 扩展阅读

- [asyncio 官方文档](https://docs.python.org/3/library/asyncio.html)
- [PEP 703 — Making the Global Interpreter Lock Optional](https://peps.python.org/pep-0703/)
- [Python Concurrency Patterns](https://python-patterns.guide/)
- FastAPI 官方教程 — Async/Await 章节
- `concurrent.futures` 文档

## 下一步学习

- [[05a-python-advanced|Python 高级特性深入]] — 装饰器、元类、描述符、生成器、类型系统
- 分布式任务队列（Celery/RQ/Dramatiq）
- GPU 并行编程（CUDA/PyTorch 分布式）
