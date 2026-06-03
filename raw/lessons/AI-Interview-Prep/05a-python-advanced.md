---
type: lesson
tags: [面试, Python, 装饰器, 元类, 描述符, 生成器, 类型系统]
created: 2026-05-20
updated: 2026-05-20
difficulty: advanced
prerequisites: [Python基础, 面向对象]
topic: Python高级特性深入
status: in-progress
series: {name: "AI Interview Prep", part: "5a"}
---

# Python 高级特性深入：装饰器、元类、描述符、生成器、类型系统

> 面向 AI 工程师面试，深入 Python 底层机制。掌握这些知识点足以应对高级 Python 工程师和 AI 基础设施岗位的技术面试。

## 学习目标

- [ ] 理解装饰器的闭包原理，能手写带参数装饰器
- [ ] 掌握元类的类创建流程，能设计插件注册系统
- [ ] 理解描述符协议和属性查找链
- [ ] 掌握生成器底层机制，能用协程处理流式数据
- [ ] 掌握 Python 类型系统，能用 Pydantic 定义数据校验

## 一、装饰器深入到源码级

### 1.1 装饰器的本质

`@decorator` 语法糖等价于 `func = decorator(func)`。理解这一行代码是一切的起点：

```python
# 这两种写法完全等价
@decorator
def func():
    pass

def func():
    pass
func = decorator(func)
```

### 1.2 闭包（Closure）的完整原理

闭包是装饰器的基石。当内部函数引用了外部函数的变量，且外部函数已经返回时，这些变量被"闭包"捕获，生命周期得以延续。

```python
def outer(x):
    def inner(y):
        return x + y  # x 是自由变量（free variable）
    return inner

add5 = outer(5)
print(add5(3))  # 8

# 检查闭包内部结构
print(add5.__closure__)        # (<cell at 0x...: int object at 0x...>,)
print(add5.__closure__[0].cell_contents)  # 5
```

`__closure__` 是一个 cell 对象元组，每个 cell 对象通过 `cell_contents` 属性持有被捕获的自由变量。Python 解释器在编译时通过 `LOAD_DEREF` / `STORE_DEREF` 字节码指令访问这些 cell 对象。

```python
import dis
dis.dis(add5)
# 关键字节码：
# LOAD_DEREF    0 (x)     ← 从闭包 cell 中加载 x
# LOAD_FAST     0 (y)     ← 从局部变量加载 y
# BINARY_ADD
# RETURN_VALUE
```

### 1.3 functools.wraps 做了什么

每次写装饰器都必须用 `@functools.wraps(func)`，否则被装饰函数的元信息会丢失。它实际执行了以下属性拷贝：

```python
# functools.wraps 的简化源码
WRAPPER_ASSIGNMENTS = ('__module__', '__name__', '__qualname__',
                       '__annotations__', '__doc__')
WRAPPER_UPDATES = ('__dict__',)

def wraps(wrapped):
    def decorator(wrapper):
        # 拷贝属性：__name__, __doc__, __module__ 等
        for attr in WRAPPER_ASSIGNMENTS:
            try:
                value = getattr(wrapped, attr)
                setattr(wrapper, attr, value)
            except AttributeError:
                pass
        # 更新 __dict__
        for attr in WRAPPER_UPDATES:
            getattr(wrapper, attr).update(getattr(wrapped, attr, {}))
        # 设置 __wrapped__ 指向原始函数
        wrapper.__wrapped__ = wrapped
        return wrapper
    return decorator
```

### 1.4 带参数装饰器的三种实现

**方式一：嵌套函数（最常用）**

```python
def retry(max_retries=3, delay=1.0):
    def decorator(func):
        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            for i in range(max_retries):
                try:
                    return func(*args, **kwargs)
                except Exception as e:
                    if i == max_retries - 1:
                        raise
                    time.sleep(delay)
        return wrapper
    return decorator

@retry(max_retries=5, delay=2.0)
def call_api(): ...
```

**方式二：类实现**

```python
class Retry:
    def __init__(self, max_retries=3, delay=1.0):
        self.max_retries = max_retries
        self.delay = delay

    def __call__(self, func):
        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            for i in range(self.max_retries):
                try:
                    return func(*args, **kwargs)
                except Exception:
                    if i == self.max_retries - 1:
                        raise
                    time.sleep(self.delay)
        return wrapper

@Retry(max_retries=5)
def call_api(): ...
```

**方式三：functools.partial**

```python
def _retry(func=None, *, max_retries=3, delay=1.0):
    if func is None:
        return functools.partial(_retry, max_retries=max_retries, delay=delay)
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        for i in range(max_retries):
            try:
                return func(*args, **kwargs)
            except Exception:
                if i == max_retries - 1:
                    raise
                time.sleep(delay)
    return wrapper

# 两种用法都支持
@_retry
def f(): ...

@_retry(max_retries=5)
def g(): ...
```

### 1.5 多装饰器叠加的执行顺序

```python
@decorator_a
@decorator_b
def func():
    pass

# 等价于：
func = decorator_a(decorator_b(func))
```

装饰顺序从下到上（先装饰 b 再装饰 a），执行时从外到内（先进入 a 的 wrapper，再进入 b 的 wrapper）。

### 1.6 常见装饰器模式

**模式1：带 TTL 的 LRU Cache**

```python
import time
import functools

def ttl_lru_cache(maxsize=128, ttl=60):
    """带过期时间的 LRU 缓存装饰器"""
    cache = {}  # key -> (value, expire_time)
    order = []  # 访问顺序列表

    def decorator(func):
        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            key = (args, tuple(sorted(kwargs.items())))
            now = time.time()

            # 命中缓存且未过期
            if key in cache and cache[key][1] > now:
                order.remove(key)
                order.append(key)
                return cache[key][0]

            # 缓存未命中或已过期
            result = func(*args, **kwargs)
            cache[key] = (result, now + ttl)
            order.append(key)

            # 淘汰旧缓存
            while len(cache) > maxsize:
                oldest = order.pop(0)
                cache.pop(oldest, None)

            return result

        wrapper.cache_clear = lambda: (cache.clear(), order.clear())
        return wrapper
    return decorator

@ttl_lru_cache(maxsize=256, ttl=300)
def embed_text(text: str) -> list:
    """带缓存的文本 Embedding"""
    return call_embedding_api(text)
```

**模式2：指数退避重试**

```python
import functools
import time
import random

def retry_with_backoff(max_retries=3, base_delay=1.0,
                       max_delay=60.0, exponential_base=2,
                       jitter=True, exceptions=(Exception,)):
    def decorator(func):
        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            last_exception = None
            for attempt in range(max_retries + 1):
                try:
                    return func(*args, **kwargs)
                except exceptions as e:
                    last_exception = e
                    if attempt == max_retries:
                        raise
                    delay = min(base_delay * (exponential_base ** attempt), max_delay)
                    if jitter:
                        delay *= random.uniform(0.5, 1.5)
                    time.sleep(delay)
            raise last_exception
        return wrapper
    return decorator
```

**模式3：令牌桶限流**

```python
import time
import threading
import functools

def rate_limit(rate: float, capacity: int = None):
    """令牌桶算法限流装饰器

    Args:
        rate: 每秒生成的令牌数
        capacity: 桶容量（默认等于 rate）
    """
    capacity = capacity or int(rate)
    tokens = capacity
    last_refill = time.time()
    lock = threading.Lock()

    def decorator(func):
        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            nonlocal tokens, last_refill
            with lock:
                now = time.time()
                elapsed = now - last_refill
                tokens = min(capacity, tokens + elapsed * rate)
                last_refill = now

                if tokens < 1:
                    wait_time = (1 - tokens) / rate
                    time.sleep(wait_time)
                    tokens = 0
                else:
                    tokens -= 1

            return func(*args, **kwargs)
        return wrapper
    return decorator

@rate_limit(rate=10, capacity=20)
def call_llm_api(prompt: str) -> str:
    return requests.post(...)
```

**模式4：参数验证**

```python
import functools
import inspect

def validate(**validators):
    """参数验证装饰器

    用法: @validate(count=lambda x: x > 0, name=lambda x: len(x) > 0)
    """
    def decorator(func):
        sig = inspect.signature(func)
        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            bound = sig.bind(*args, **kwargs)
            bound.apply_defaults()
            for param_name, validator_fn in validators.items():
                if param_name in bound.arguments:
                    value = bound.arguments[param_name]
                    if not validator_fn(value):
                        raise ValueError(
                            f"参数 '{param_name}' 验证失败: {value!r}"
                        )
            return func(*args, **kwargs)
        return wrapper
    return decorator
```

**模式5：异步装饰器**

```python
import functools
import asyncio

def async_retry(max_retries=3, delay=1.0, exceptions=(Exception,)):
    """异步重试装饰器"""
    def decorator(func):
        @functools.wraps(func)
        async def wrapper(*args, **kwargs):
            last_exception = None
            for attempt in range(max_retries + 1):
                try:
                    return await func(*args, **kwargs)
                except exceptions as e:
                    last_exception = e
                    if attempt == max_retries:
                        raise
                    await asyncio.sleep(delay * (2 ** attempt))
            raise last_exception
        return wrapper
    return decorator

@async_retry(max_retries=3, delay=1.0)
async def fetch_embedding(text: str) -> list:
    async with aiohttp.ClientSession() as session:
        async with session.post(url, json={"text": text}) as resp:
            return await resp.json()
```

> [!hint]- 面试题：手写一个支持超时和重试的 API 调用装饰器
> 要求：支持设置超时时间（秒）、最大重试次数、重试间隔（指数退避）、可指定的异常类型。同时支持同步和异步函数。

> [!success]- 参考答案
> ```python
> import functools
> import time
> import asyncio
> import signal
> import inspect
>
> def api_call(timeout=30, max_retries=3, base_delay=1.0,
>              exceptions=(TimeoutError, ConnectionError)):
>     def decorator(func):
>         is_async = inspect.iscoroutinefunction(func)
>
>         if is_async:
>             @functools.wraps(func)
>             async def wrapper(*args, **kwargs):
>                 last_exc = None
>                 for attempt in range(max_retries + 1):
>                     try:
>                         return await asyncio.wait_for(
>                             func(*args, **kwargs), timeout=timeout
>                         )
>                     except (asyncio.TimeoutError, *exceptions) as e:
>                         last_exc = e
>                         if attempt == max_retries:
>                             raise
>                         await asyncio.sleep(base_delay * (2 ** attempt))
>                 raise last_exc
>         else:
>             @functools.wraps(func)
>             def wrapper(*args, **kwargs):
>                 last_exc = None
>                 for attempt in range(max_retries + 1):
>                     try:
>                         return _run_with_timeout(
>                             func, args, kwargs, timeout
>                         )
>                     except (*exceptions, TimeoutError) as e:
>                         last_exc = e
>                         if attempt == max_retries:
>                             raise
>                         time.sleep(base_delay * (2 ** attempt))
>                 raise last_exc
>
>         def _run_with_timeout(fn, args, kwargs, secs):
>             # Unix 上可用 signal 实现；Windows 可用线程池 + wait
>             import concurrent.futures
>             with concurrent.futures.ThreadPoolExecutor(1) as pool:
>                 future = pool.submit(fn, *args, **kwargs)
>                 return future.result(timeout=secs)
>
>         return wrapper
>     return decorator
>
> # 使用示例
> @api_call(timeout=10, max_retries=3)
> def call_gpt(prompt: str) -> str:
>     return requests.post("https://api.openai.com/v1/chat/completions",
>                          json={"prompt": prompt}).json()
>
> @api_call(timeout=30, max_retries=5)
> async def call_gpt_async(prompt: str) -> str:
>     async with aiohttp.ClientSession() as s:
>         async with s.post(url, json={"prompt": prompt}) as r:
>             return await r.json()
> ```

---

## 二、元类（Metaclass）深入

### 2.1 Python 类创建的完整流程

当我们写 `class Foo:` 时，Python 解释器执行以下步骤：

1. 解释器收集类体中的所有属性，构建 `dict`
2. 查找元类：先看 `metaclass=` 关键字参数，再看基类的 `type`
3. 调用 `metaclass(name, bases, namespace)` 创建类对象
4. 这最终触发 `type.__call__`，它依次调用 `__new__` 和 `__init__`

```python
# 用 type 手动创建一个类
MyClass = type(
    'MyClass',                    # 类名
    (BaseClass,),                 # 基类元组
    {'x': 1, 'method': lambda self: self.x}  # 属性字典
)

# 等价于：
class MyClass(BaseClass):
    x = 1
    def method(self):
        return self.x
```

`type.__call__` 的伪代码：

```python
class type:
    def __call__(cls, *args, **kwargs):
        # 1. 调用 __new__ 创建实例
        obj = cls.__new__(cls, *args, **kwargs)
        # 2. 如果 __new__ 返回的是 cls 的实例，调用 __init__
        if isinstance(obj, cls):
            cls.__init__(obj, *args, **kwargs)
        return obj
```

### 2.2 元类的实际应用

**应用1：单例模式**

```python
class SingletonMeta(type):
    """线程安全的单例元类"""
    _instances = {}
    _lock = threading.Lock()

    def __call__(cls, *args, **kwargs):
        if cls not in cls._instances:
            with cls._lock:
                # 双重检查
                if cls not in cls._instances:
                    cls._instances[cls] = super().__call__(*args, **kwargs)
        return cls._instances[cls]

class LLMClient(metaclass=SingletonMeta):
    def __init__(self, api_key: str):
        self.api_key = api_key
        self.session = requests.Session()

# 无论创建多少次，都是同一个实例
client1 = LLMClient("key-123")
client2 = LLMClient("key-456")
assert client1 is client2  # True
```

**应用2：插件注册表元类**

```python
class PluginRegistryMeta(type):
    """自动注册子类的元类"""
    _registry = {}

    def __new__(mcs, name, bases, namespace):
        cls = super().__new__(mcs, name, bases, namespace)
        # 不注册基类自身
        if bases and any(hasattr(b, '_is_base') for b in bases):
            PluginRegistryMeta._registry[name.lower()] = cls
        return cls

    @classmethod
    def get_plugin(mcs, name: str):
        if name not in mcs._registry:
            raise KeyError(f"未注册的插件: {name}")
        return mcs._registry[name]

    @classmethod
    def list_plugins(mcs):
        return dict(mcs._registry)

class Tool(metaclass=PluginRegistryMeta):
    _is_base = True
    def execute(self, **kwargs):
        raise NotImplementedError

class SearchTool(Tool):
    def execute(self, **kwargs):
        return search_web(kwargs["query"])

class CalculatorTool(Tool):
    def execute(self, **kwargs):
        return eval(kwargs["expression"])

# 子类定义时自动注册
print(PluginRegistryMeta.list_plugins())
# {'searchtool': <class 'SearchTool'>, 'calculatortool': <class 'CalculatorTool'>}

tool = PluginRegistryMeta.get_plugin("searchtool")
tool().execute(query="Python metaclass")
```

**应用3：ORM 字段自动收集**

```python
class Field:
    def __init__(self, field_type, required=False, default=None):
        self.field_type = field_type
        self.required = required
        self.default = default
        self.name = None  # 由 __set_name__ 设置

    def __set_name__(self, owner, name):
        self.name = name

    def __get__(self, obj, objtype=None):
        if obj is None:
            return self
        return obj.__dict__.get(self.name, self.default)

    def __set__(self, obj, value):
        if self.required and value is None:
            raise ValueError(f"{self.name} is required")
        obj.__dict__[self.name] = value

class ModelMeta(type):
    def __new__(mcs, name, bases, namespace):
        fields = {}
        for key, value in namespace.items():
            if isinstance(value, Field):
                fields[key] = value
        namespace['_fields'] = fields
        cls = super().__new__(mcs, name, bases, namespace)
        return cls

class Document(metaclass=ModelMeta):
    title = Field(str, required=True)
    content = Field(str)
    embedding = Field(list)

    def __init__(self, **kwargs):
        for name, field in self._fields.items():
            value = kwargs.get(name, field.default)
            setattr(self, name, value)

doc = Document(title="Hello", content="World")
print(Document._fields)  # {'title': <Field>, 'content': <Field>, 'embedding': <Field>}
```

**应用4：接口强制元类**

```python
class InterfaceMeta(type):
    """检查子类是否实现了所有抽象方法"""
    def __new__(mcs, name, bases, namespace):
        cls = super().__new__(mcs, name, bases, namespace)
        if not bases:
            return cls
        # 收集所有标记为 abstract 的方法
        abstracts = set()
        for base in cls.__mro__[1:]:
            for attr_name, attr in vars(base).items():
                if getattr(attr, '_is_abstract', False):
                    abstracts.add(attr_name)
        # 检查子类是否实现了
        missing = abstracts - set(namespace.keys())
        if missing:
            raise TypeError(
                f"类 {name} 未实现抽象方法: {', '.join(missing)}"
            )
        return cls

def abstract(method):
    method._is_abstract = True
    return method

class AgentInterface(metaclass=InterfaceMeta):
    @abstract
    def plan(self, task): ...
    @abstract
    def execute(self, plan): ...

class MyAgent(AgentInterface):
    def plan(self, task):
        return [task]
    def execute(self, plan):
        return "done"

# class BadAgent(AgentInterface): pass  # TypeError: 未实现抽象方法: plan, execute
```

### 2.3 `__init_subclass__` vs 元类

Python 3.6 引入的 `__init_subclass__` 是元类的轻量替代方案，适合大多数场景：

```python
class PluginBase:
    _registry = {}

    def __init_subclass__(cls, register_name=None, **kwargs):
        super().__init_subclass__(**kwargs)
        if register_name:
            PluginBase._registry[register_name] = cls

class MyTool(PluginBase, register_name="search"):
    pass

# 对比：元类更适合需要控制 __new__ 阶段的场景（如修改类体、拦截创建）
# __init_subclass__ 只能在类创建完成后做初始化
```

> [!hint]- 面试题：设计一个自动注册的 Agent 工具系统
> 要求：(1) 工具类定义时自动注册到全局注册表；(2) 支持按名称查找工具；(3) 支持工具的热替换；(4) 工具必须实现 `execute` 方法，否则注册时抛出异常。

> [!success]- 参考答案
> ```python
> import threading
>
> class ToolRegistry:
>     """全局工具注册表（线程安全）"""
>     _instance = None
>     _lock = threading.Lock()
>
>     def __new__(cls):
>         if cls._instance is None:
>             with cls._lock:
>                 if cls._instance is None:
>                     obj = super().__new__(cls)
>                     obj._tools = {}
>                     obj._tools_lock = threading.Lock()
>                     cls._instance = obj
>         return cls._instance
>
>     def register(self, name: str, tool_cls):
>         if not hasattr(tool_cls, 'execute') or \
>            not callable(getattr(tool_cls, 'execute')):
>             raise TypeError(f"工具 '{name}' 必须实现 execute 方法")
>         with self._tools_lock:
>             self._tools[name] = tool_cls
>
>     def get(self, name: str):
>         with self._tools_lock:
>             if name not in self._tools:
>                 raise KeyError(f"工具 '{name}' 未注册")
>             return self._tools[name]
>
>     def replace(self, name: str, new_tool_cls):
>         self.register(name, new_tool_cls)  # 覆盖注册即热替换
>
>     def list_all(self):
>         with self._tools_lock:
>             return dict(self._tools)
>
> registry = ToolRegistry()
>
> class AutoToolMeta(type):
>     def __new__(mcs, name, bases, namespace):
>         cls = super().__new__(mcs, name, bases, namespace)
>         tool_name = namespace.get('tool_name') or name.lower()
>         if bases and any(isinstance(b, AutoToolMeta) for b in bases):
>             if not namespace.get('_skip_register'):
>                 registry.register(tool_name, cls)
>         return cls
>
> class BaseTool(metaclass=AutoToolMeta):
>     tool_name = None
>     _skip_register = True
>     def execute(self, **kwargs):
>         raise NotImplementedError
>
> class WebSearchTool(BaseTool):
>     tool_name = "web_search"
>     def execute(self, **kwargs):
>         return f"搜索结果: {kwargs['query']}"
>
> class CodeExecutorTool(BaseTool):
>     tool_name = "code_executor"
>     def execute(self, **kwargs):
>         return exec_tool(kwargs["code"])
>
> # 使用
> search = registry.get("web_search")()
> result = search.execute(query="Python metaclass")
>
> # 热替换
> class WebSearchToolV2(BaseTool):
>     tool_name = "web_search"
>     def execute(self, **kwargs):
>         return f"V2搜索: {kwargs['query']}"
> ```

---

## 三、描述符（Descriptor）深入

### 3.1 描述符协议

描述符是实现以下方法之一的对象：

```python
class Descriptor:
    def __get__(self, obj, objtype=None):
        """访问属性时调用。obj 为 None 表示类级别访问"""
    def __set__(self, obj, value):
        """设置属性时调用"""
    def __delete__(self, obj):
        """删除属性时调用"""
    def __set_name__(self, owner, name):
        """Python 3.6+，所在类创建时自动调用，获取属性名"""
```

**数据描述符**：同时定义 `__get__` 和 `__set__`（或 `__delete__`）
**非数据描述符**：只定义 `__get__`

### 3.2 属性查找优先级链

这是 Python 属性访问的核心规则，面试必考：

```
obj.attr 的查找顺序：
1. type(obj).__mro__ 中所有类的 __getattribute__
2. type(obj) 的数据描述符的 __get__
3. obj.__dict__ 中的实例属性
4. type(obj) 的非数据描述符的 __get__
5. type(obj) 的类属性
6. type(obj).__getattr__（如果定义了）
7. AttributeError → 触发 __getattr__
```

```python
# 验证优先级
class DataDescriptor:
    def __get__(self, obj, objtype=None):
        return "data descriptor"
    def __set__(self, obj, value):
        obj.__dict__["_data"] = value

class NonDataDescriptor:
    def __get__(self, obj, objtype=None):
        return "non-data descriptor"

class MyClass:
    data_desc = DataDescriptor()
    non_data_desc = NonDataDescriptor()

obj = MyClass()
obj.__dict__["data_desc"] = "instance attr"
obj.__dict__["non_data_desc"] = "instance attr"

print(obj.data_desc)      # "data descriptor"（数据描述符优先）
print(obj.non_data_desc)  # "instance attr"（实例属性优先于非数据描述符）
```

### 3.3 描述符实现实例

**类型验证描述符**

```python
class TypedField:
    def __init__(self, expected_type, required=False, default=None):
        self.expected_type = expected_type
        self.required = required
        self.default = default

    def __set_name__(self, owner, name):
        self.name = name
        self.storage_name = f"_{name}"

    def __get__(self, obj, objtype=None):
        if obj is None:
            return self
        return getattr(obj, self.storage_name, self.default)

    def __set__(self, obj, value):
        if value is None:
            if self.required:
                raise ValueError(f"'{self.name}' is required")
            setattr(obj, self.storage_name, None)
            return
        if not isinstance(value, self.expected_type):
            raise TypeError(
                f"'{self.name}' 期望 {self.expected_type.__name__}，"
                f"得到 {type(value).__name__}"
            )
        setattr(obj, self.storage_name, value)

class PromptTemplate:
    template = TypedField(str, required=True)
    max_tokens = TypedField(int, default=512)
    temperature = TypedField(float, default=0.7)

    def __init__(self, template, **kwargs):
        self.template = template
        for k, v in kwargs.items():
            setattr(self, k, v)

# tmpl = PromptTemplate(123)  # TypeError: 'template' 期望 str
tmpl = PromptTemplate("Hello {name}", max_tokens=1024)
```

**懒加载描述符**

```python
class LazyField:
    """首次访问时才计算，之后缓存结果"""
    def __init__(self, factory):
        self.factory = factory

    def __set_name__(self, owner, name):
        self.name = name
        self.storage = f"_lazy_{name}"

    def __get__(self, obj, objtype=None):
        if obj is None:
            return self
        if not hasattr(obj, self.storage):
            setattr(obj, self.storage, self.factory(obj))
        return getattr(obj, self.storage)

class Document:
    text = LazyField(lambda self: self._load_text())
    embedding = LazyField(lambda self: self._compute_embedding())

    def __init__(self, path):
        self.path = path

    def _load_text(self):
        with open(self.path) as f:
            return f.read()

    def _compute_embedding(self):
        return embed(self.text)
```

**ORM Field 描述符（模拟 Django）**

```python
class Column:
    """模拟数据库列"""
    def __init__(self, col_type, primary_key=False, nullable=True, default=None):
        self.col_type = col_type
        self.primary_key = primary_key
        self.nullable = nullable
        self.default = default

    def __set_name__(self, owner, name):
        self.name = name
        self.storage = f"_col_{name}"

    def __get__(self, obj, objtype=None):
        if obj is None:
            return self
        return getattr(obj, self.storage, self.default)

    def __set__(self, obj, value):
        if value is None and not self.nullable and not self.primary_key:
            raise ValueError(f"Column '{self.name}' cannot be null")
        setattr(obj, self.storage, value)

class TableMeta(type):
    def __new__(mcs, name, bases, namespace):
        columns = {}
        for key, val in namespace.items():
            if isinstance(val, Column):
                columns[key] = val
        namespace['_columns'] = columns
        namespace['_table_name'] = namespace.get('table_name', name.lower())
        return super().__new__(mcs, name, bases, namespace)

class Model(metaclass=TableMeta):
    def save(self):
        cols = {k: getattr(self, k) for k in self._columns if hasattr(self, f"_col_{k}")}
        placeholders = ", ".join(["?"] * len(cols))
        col_names = ", ".join(cols.keys())
        sql = f"INSERT INTO {self._table_name} ({col_names}) VALUES ({placeholders})"
        return sql, list(cols.values())

class User(Model):
    table_name = "users"
    id = Column(int, primary_key=True, nullable=False)
    name = Column(str, nullable=False)
    email = Column(str)

u = User()
u.id = 1
u.name = "Alice"
sql, params = u.save()
# INSERT INTO users (id, name) VALUES (?, ?)  [1, 'Alice']
```

> [!hint]- 面试题：property、classmethod、staticmethod 是怎么实现的？
> 用描述符原理解释这三个内置装饰器的底层机制。

> [!success]- 参考答案
> ```python
> # property 本质是一个数据描述符
> class Property:
>     def __init__(self, fget=None, fset=None, fdel=None, doc=None):
>         self.fget = fget
>         self.fset = fset
>         self.fdel = fdel
>         self.__doc__ = doc
>
>     def __get__(self, obj, objtype=None):
>         if obj is None:
>             return self
>         if self.fget is None:
>             raise AttributeError("unreadable attribute")
>         return self.fget(obj)
>
>     def __set__(self, obj, value):
>         if self.fset is None:
>             raise AttributeError("can't set attribute")
>         self.fset(obj, value)
>
>     def __delete__(self, obj):
>         if self.fdel is None:
>             raise AttributeError("can't delete attribute")
>         self.fdel(obj)
>
>     def setter(self, fset):
>         self.fset = fset
>         return self
>
> # classmethod 本质是一个非数据描述符
> class ClassMethod:
>     def __init__(self, func):
>         self.func = func
>
>     def __get__(self, obj, cls=None):
>         if cls is None:
>             cls = type(obj)
>         return self.func.__get__(cls, type(cls))  # 绑定到类
>
> # staticmethod 本质也是一个非数据描述符
> class StaticMethod:
>     def __init__(self, func):
>         self.func = func
>
>     def __get__(self, obj, objtype=None):
>         return self.func  # 直接返回原函数，不做任何绑定
> ```
>
> `property` 是数据描述符（有 `__set__`），所以即使实例 `__dict__` 中有同名属性也会被描述符拦截。`classmethod` 和 `staticmethod` 是非数据描述符（只有 `__get__`），但它们通常不会与实例属性同名，所以优先级问题不常出现。

---

## 四、生成器与协程深入

### 4.1 生成器的底层实现

生成器函数在编译时就被标记为生成器（`CO_GENERATOR` 标志位）。调用生成器函数不会执行函数体，而是返回一个生成器对象。生成器对象内部持有帧对象（frame），帧保存了执行状态（局部变量、指令指针、栈）。

```python
def gen():
    x = 1
    yield x
    y = x + 2
    yield y

g = gen()
# g.gi_frame 保存了帧对象
# g.gi_frame.f_lasti 保存了上次执行的字节码偏移
# g.gi_frame.f_locals 保存了局部变量

print(next(g))  # 1  — 执行到第一个 yield 挂起
print(g.gi_frame.f_locals)  # {'x': 1}
print(next(g))  # 3  — 从挂起点恢复，执行到第二个 yield
print(g.gi_frame.f_locals)  # {'x': 1, 'y': 3}
```

### 4.2 send()/throw()/close() 机制

```python
def echo_generator():
    """演示 send/throw/close 的完整机制"""
    print("生成器启动")
    while True:
        try:
            value = yield "ready"
            print(f"收到: {value}")
        except ValueError as e:
            yield f"捕获异常: {e}"
        except GeneratorExit:
            print("生成器被关闭")
            return  # 必须用 return 或 StopIteration 结束

g = echo_generator()
print(next(g))         # "生成器启动" -> "ready"（必须先 next 启动）
print(g.send("hello")) # "收到: hello" -> "ready"
print(g.throw(ValueError, "测试异常"))  # "捕获异常: 测试异常"
g.close()              # "生成器被关闭"
```

### 4.3 yield from 委托机制

`yield from` 建立了一条双向通道，将调用者和子生成器直接连接：

```python
def accumulator():
    total = 0
    while True:
        value = yield total
        if value is None:
            break
        total += value
    return total  # return 值成为 yield from 表达式的值

def delegator():
    result = yield from accumulator()  # 委托给子生成器
    print(f"子生成器返回: {result}")

d = delegator()
next(d)           # 启动
d.send(10)        # 10
d.send(20)        # 30
d.send(5)         # 35
try:
    d.send(None)  # 触发 StopIteration
except StopIteration:
    pass
# 输出: 子生成器返回: 35
```

### 4.4 实际应用

**流式数据处理管道**

```python
def read_lines(filepath, chunk_size=8192):
    """分块读取文件的生成器"""
    with open(filepath, 'r', encoding='utf-8') as f:
        buffer = ""
        while True:
            chunk = f.read(chunk_size)
            if not chunk:
                if buffer:
                    yield buffer
                break
            buffer += chunk
            while '\n' in buffer:
                line, buffer = buffer.split('\n', 1)
                yield line

def parse_json_lines(lines):
    """解析 JSON 行的生成器"""
    import json
    for line in lines:
        try:
            yield json.loads(line)
        except json.JSONDecodeError:
            continue

def filter_valid(records):
    """过滤有效记录"""
    for record in records:
        if record.get('status') == 'ok':
            yield record

def batch(records, size=100):
    """分批生成器"""
    batch_list = []
    for record in records:
        batch_list.append(record)
        if len(batch_list) == size:
            yield batch_list
            batch_list = []
    if batch_list:
        yield batch_list

# 管道串联
pipeline = batch(filter_valid(parse_json_lines(read_lines("data.jsonl"))))
for b in pipeline:
    process_batch(b)
```

**无限序列生成器**

```python
import itertools

def fibonacci():
    a, b = 0, 1
    while True:
        yield a
        a, b = b, a + b

def sliding_window(iterable, window_size):
    """滑动窗口生成器"""
    from collections import deque
    window = deque(maxlen=window_size)
    for item in iterable:
        window.append(item)
        if len(window) == window_size:
            yield tuple(window)

# 使用
for window in sliding_window(fibonacci(), 5):
    if window[-1] > 1000:
        break
    print(window)
```

> [!hint]- 面试题：用生成器实现一个分批读取大文件的函数
> 要求：(1) 文件可能超过内存大小；(2) 支持指定批次大小；(3) 支持跳过头部行数；(4) 支持指定编码。

> [!success]- 参考答案
> ```python
> def batch_read(filepath, batch_size=1000, skip_lines=0,
>                 encoding='utf-8', delimiter='\n'):
>     """分批读取大文件
>
>     Args:
>         filepath: 文件路径
>         batch_size: 每批行数
>         skip_lines: 跳过前 N 行
>         encoding: 文件编码
>         delimiter: 行分隔符
>
>     Yields:
>         每批是一个 list[str]
>     """
>     batch = []
>     line_count = 0
>
>     with open(filepath, 'r', encoding=encoding) as f:
>         for line in f:
>             line_count += 1
>             if line_count <= skip_lines:
>                 continue
>
>             batch.append(line.rstrip(delimiter))
>             if len(batch) == batch_size:
>                 yield batch
>                 batch = []
>
>     if batch:
>         yield batch
>
> # 使用：逐批处理 10GB 的日志文件
> for i, lines in enumerate(batch_read("huge.log", batch_size=5000)):
>     results = process(lines)
>     save_to_db(results)
>     if i % 100 == 0:
>         print(f"已处理 {(i+1) * 5000} 行")
> ```

---

## 五、Python 类型系统深入

### 5.1 Type Hints 的运行时行为

类型注解存储在 `__annotations__` 属性中，Python 运行时不强制检查：

```python
def greet(name: str, times: int = 1) -> str:
    return (f"Hello, {name}! ") * times

print(greet.__annotations__)
# {'name': <class 'str'>, 'times': <class 'int'>, 'return': <class 'str'>}
```

### 5.2 typing 模块高级用法

**泛型（TypeVar + Generic）**

```python
from typing import TypeVar, Generic, List

T = TypeVar('T')

class Stack(Generic[T]):
    def __init__(self) -> None:
        self._items: List[T] = []

    def push(self, item: T) -> None:
        self._items.append(item)

    def pop(self) -> T:
        return self._items.pop()

stack: Stack[int] = Stack()
stack.push(42)
# stack.push("hello")  # mypy 会报错
```

**Protocol（结构化子类型）**

```python
from typing import Protocol, runtime_checkable

@runtime_checkable
class Embeddable(Protocol):
    """任何有 embed 方法的对象都可以被处理"""
    def embed(self, text: str) -> list[float]: ...

class OpenAIEmbedder:
    def embed(self, text: str) -> list[float]:
        return [0.1, 0.2, 0.3]

class LocalEmbedder:
    def embed(self, text: str) -> list[float]:
        return [0.4, 0.5, 0.6]

def process(obj: Embeddable) -> None:
    print(obj.embed("hello"))

process(OpenAIEmbedder())  # OK — 鸭子类型，不需要继承
process(LocalEmbedder())   # OK
```

**TypedDict 和 Literal**

```python
from typing import TypedDict, Literal, Annotated, Final

class LLMConfig(TypedDict):
    model: str
    temperature: float
    max_tokens: int

# Literal 用于限定具体值
Role = Literal["system", "user", "assistant"]

# Annotated 可以附加元数据
Temperature = Annotated[float, "介于 0.0 和 2.0 之间"]

# Final 防止重新赋值
DEFAULT_MODEL: Final = "gpt-4"
```

### 5.3 Pydantic 核心：运行时类型检查

Pydantic v2 的核心原理：

1. 在模型类创建时，通过元类收集所有字段的类型注解
2. 为每个字段生成验证器和序列化器（Rust 实现的 pydantic-core）
3. 实例化时对输入数据执行验证流水线：类型转换 → 约束检查 → 自定义验证器

```python
from pydantic import BaseModel, Field, field_validator, model_validator
from typing import Optional, Literal
from enum import Enum

class MessageRole(str, Enum):
    SYSTEM = "system"
    USER = "user"
    ASSISTANT = "assistant"

class Message(BaseModel):
    role: MessageRole
    content: str
    name: Optional[str] = None

    @field_validator('content')
    @classmethod
    def content_not_empty(cls, v: str) -> str:
        if not v.strip():
            raise ValueError('content 不能为空')
        return v

class AgentInput(BaseModel):
    """Agent 的输入 Schema"""
    messages: list[Message] = Field(..., min_length=1)
    model: str = Field(default="gpt-4", pattern=r"^(gpt-4|claude|gemini)")
    temperature: float = Field(default=0.7, ge=0.0, le=2.0)
    max_tokens: int = Field(default=2048, ge=1, le=128000)
    tools: Optional[list[dict]] = None

    @model_validator(mode='after')
    def validate_messages(self):
        if self.messages[0].role != MessageRole.SYSTEM:
            raise ValueError('第一条消息必须是 system 角色')
        return self

class ToolCall(BaseModel):
    name: str
    arguments: dict

class AgentOutput(BaseModel):
    """Agent 的输出 Schema"""
    content: Optional[str] = None
    tool_calls: Optional[list[ToolCall]] = None
    finish_reason: Literal["stop", "tool_calls", "length"]

    @model_validator(mode='after')
    def validate_output(self):
        if self.finish_reason == "tool_calls" and not self.tool_calls:
            raise ValueError('tool_calls 结束原因必须提供 tool_calls')
        return self

# 使用
inp = AgentInput(
    messages=[
        Message(role="system", content="You are a helpful assistant."),
        Message(role="user", content="What is Python?"),
    ],
    model="gpt-4",
    temperature=0.5,
)
```

> [!hint]- 面试题：如何保证 LLM 输出符合预期的 JSON Schema？
> LLM 的输出不确定，如何确保返回值可以被解析为结构化数据？

> [!success]- 参考答案
> ```python
> import json
> from pydantic import BaseModel, ValidationError
> from typing import Optional
>
> class StructuredOutput(BaseModel):
>     answer: str
>     confidence: float
>     sources: list[str]
>
> def get_structured_llm_output(prompt: str, schema: type[BaseModel],
>                                max_retries=3) -> BaseModel:
>     """从 LLM 获取结构化输出，带重试和修复"""
>     schema_str = json.dumps(schema.model_json_schema(), indent=2)
>
>     system_msg = (
>         f"你必须输出严格符合以下 JSON Schema 的 JSON：\n"
>         f"```json\n{schema_str}\n```\n"
>         f"不要输出任何 JSON 之外的内容。"
>     )
>
>     for attempt in range(max_retries):
>         try:
>             raw = call_llm(system=system_msg, user=prompt)
>             # 尝试直接解析
>             data = json.loads(raw)
>             return schema.model_validate(data)
>         except json.JSONDecodeError:
>             # 尝试修复：提取 JSON 块
>             import re
>             match = re.search(r'\{.*\}', raw, re.DOTALL)
>             if match:
>                 data = json.loads(match.group())
>                 return schema.model_validate(data)
>         except ValidationError as e:
>             if attempt < max_retries - 1:
>                 # 将错误信息反馈给 LLM 修复
>                 system_msg += f"\n\n上次输出有误：{e}\n请修正。"
>             else:
>                 raise
>
>     raise ValueError("无法获取有效结构化输出")
>
> # 方案二：使用 OpenAI 的 function calling / structured output
> # 这是更推荐的方式，由 API 层面保证
> from openai import OpenAI
> client = OpenAI()
>
> response = client.beta.chat.completions.parse(
>     model="gpt-4o",
>     messages=[{"role": "user", "content": prompt}],
>     response_format=StructuredOutput,  # 直接传 Pydantic 模型
> )
> result = response.choices[0].message.parsed  # 类型安全的结构化输出
> ```

## 关键要点回顾

1. **装饰器 = 闭包 + 语法糖**：理解 `__closure__` 和自由变量是掌握装饰器的基础
2. **元类控制类的创建**：`__new__` 创建类对象，`__init__` 初始化它；插件注册是元类最实用的场景
3. **描述符控制属性访问**：数据描述符 > 实例属性 > 非数据描述符；property/classmethod/staticmethod 本质都是描述符
4. **生成器 = 可暂停的函数**：帧对象的挂起/恢复机制；`yield from` 建立双向委托通道
5. **Pydantic = 运行时类型安全**：在 AI 系统中保证数据质量的关键工具

## 扩展阅读

- [Python Data Model 官方文档](https://docs.python.org/3/reference/datamodel.html)
- [PEP 484 — Type Hints](https://peps.python.org/pep-0484/)
- [PEP 544 — Protocols](https://peps.python.org/pep-0544/)
- [Pydantic v2 文档](https://docs.pydantic.dev/latest/)
- Fluent Python, 2nd Edition — Luciano Ramalho（第22-24章）

## 下一步学习

- [[05b-python-concurrency|Python 并发编程深入]] — GIL、asyncio、多进程
- Python 内存管理与垃圾回收机制
- C 扩展开发基础
