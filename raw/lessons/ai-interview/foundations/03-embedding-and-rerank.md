---
type: lesson
tags: [Embedding, Rerank, 向量化, 相似度, 基础知识]
created: 2026-05-25
updated: 2026-05-25
difficulty: beginner
prerequisites: [高中数学向量概念]
topic: 开源项目深度拆解
status: completed
series: { name: "ai-interview-伯乐深度拆解-基础篇", part: 3 }
---

# Embedding 与 Rerank 基础概念

> "文字变成数字"到底是怎么变的？为什么这些数字能代表语义？Rerank 又是在什么阶段介入的？本文用最直观的方式讲清楚。

---

## 一、Embedding：把文字变成数字

### 1.1 为什么需要 Embedding？

计算机只认识数字，不认识文字。要比较两段文字的相似度，必须先转成数字。

```
"苹果" 和 "香蕉" 有多像？  →  人脑可以判断（都是水果）
                             计算机需要数字才能计算

"苹果" → [0.8, -0.3, 0.5, ...]  ← 一个高维空间中的点
"香蕉" → [0.7, -0.2, 0.4, ...]  ← 离"苹果"很近
"汽车" → [-0.6, 0.8, -0.1, ...] ← 离"苹果"很远
```

### 1.2 从一句话直观理解

把 Embedding 想象成一个**智能地图**：

```
                    技术话题
                       ↑
          "微服务架构"  │  "RESTful API"
                       │
                       │
    ←─────────────────┼─────────────────→  生活话题
                       │
          "找工作"     │
          "面试技巧"   │
                       │
                       ↓
                    人文话题
```

在这个"语义空间"中：
- 距离近 = 语义相似（"微服务"和"API"挨着）
- 距离远 = 语义不同（"微服务"和"美食"隔着远）

### 1.3 Embedding 模型怎么做到的？

以 BGE-M3 为例，训练过程（简化版）：

```
第 1 步：准备训练数据
  - 正例对（语义近的句子）：
    "HashMap 的工作原理" ↔ "HashMap 底层实现机制"
  - 负例对（语义远的句子）：
    "HashMap 的工作原理" ↔ "今天天气不错"

第 2 步：对比学习
  模型收到一对句子 → 输出两个向量
  如果正例 → 让两个向量靠近（增大相似度）
  如果负例 → 让两个向量远离（减小相似度）

第 3 步：反复训练
  经过数十亿对数据训练后
  模型学会了：什么样的文字应该产生怎样的向量
```

### 1.4 为什么是 1024 维？

```
维度太少 (128)：
  "HashMap" 和 "TreeMap" 可能挤在一起，分不开
  
维度适中 (1024)：
  有足够的空间区分不同概念，同时计算效率合理
  
维度太多 (4096)：
  区分能力更强，但存储和计算成本翻倍
```

BGE-M3 的 1024 维是经过大量实验得出的**性价比最优**选择。

### 1.5 余弦相似度 —— 如何计算两个向量的距离

```python
import numpy as np

def cosine_similarity(vec_a, vec_b):
    """
    余弦相似度：值在 [-1, 1] 之间
    1 = 完全同向（语义相同）
    0 = 正交（无关）
    -1 = 完全反向（语义相反）
    """
    dot_product = np.dot(vec_a, vec_b)
    norm_a = np.linalg.norm(vec_a)
    norm_b = np.linalg.norm(vec_b)
    return dot_product / (norm_a * norm_b)

# 例子
hashmap_vec = [0.5, 0.3, -0.1, ...]
treemap_vec = [0.45, 0.32, -0.08, ...]
weather_vec = [-0.6, 0.8, 0.2, ...]

print(cosine_similarity(hashmap_vec, treemap_vec))  # 0.93 — 很像
print(cosine_similarity(hashmap_vec, weather_vec))  # 0.05 — 不相关
```

---

## 二、Rerank：在初筛结果上做精排

### 2.1 为什么需要 Rerank？

```
Embedding 检索（粗排）的问题：
1. 速度快（向量索引），但可能漏掉或误召
2. Embedding 模型是"压缩"过的（1024维概括全文义），细节可能丢失

Rerank（精排）的优势：
1. 把问题和文档全文一起喂给模型，做逐字精读
2. 准确率远超 Embedding 的粗排
3. 代价：速度慢（所以只对 Top-N 做 Rerank）

两阶段策略：
粗排 (Embedding) → Top-50 → 精排 (Rerank) → Top-5 → 喂给 LLM
   快但粗                   慢但精
```

### 2.2 对比：Bi-Encoder vs Cross-Encoder

```
Bi-Encoder（Embedding 模型，如 BGE-M3）：
┌─────────┐     ┌─────────┐
│  "苹果"  │ →  │ "苹果向量" │    独立的！可以预先计算并缓存
└─────────┘     └─────────┘
┌─────────┐     ┌─────────┐
│  "水果"  │ →  │ "水果向量" │
└─────────┘     └─────────┘
→ 比较时只需算两个向量的余弦相似度，很快

Cross-Encoder（Rerank 模型）：
┌────────────┐     ┌──────────┐
│ "苹果"     │ ──→ │          │ → 相似度 0.95
│ + "水果"   │     │ 联合编码  │      (极高)
└────────────┘     └──────────┘
→ 把问题和文档拼接在一起，全注意力计算，很慢但很准
```

### 2.3 实际效果对比

```
用户问题："HashMap 怎么解决哈希冲突？"

Embedding 粗排结果 (Top-5):
1. [0.87] "HashMap 使用链表法..."                   ← 对
2. [0.82] "哈希冲突的几种解决方案..."                 ← 对
3. [0.78] "Java 集合框架概述..."                     ← 不太对
4. [0.75] "ConcurrentHashMap 的线程安全..."          ← 偏了
5. [0.72] "HashMap 源码分析..."                     ← 对

Rerank 精排后 (Top-3):
1. [0.96] "HashMap 使用链表法..."                   ← 原来第1，提分了
2. [0.91] "哈希冲突的几种解决方案..."                 ← 原来第2
3. [0.88] "HashMap 源码分析..."                     ← 原来第5，大幅提分！
4. [0.23] "Java 集合框架概述..."                     ← 原来第3，降分了
5. [0.15] "ConcurrentHashMap 的线程安全..."          ← 原来第4，证明了它不相关
```

Rerank 把真正相关但被粗排低估的第 5 名提到了第 3 名。

---

## 三、常见的 Embedding 模型对比

| 模型 | 维度 | 中文效果 | API 获取 | 推荐场景 |
|------|------|---------|---------|---------|
| BGE-M3 | 1024 | ⭐⭐⭐⭐⭐ | SiliconFlow 免费 | 中文场景首选 |
| text-embedding-3-large | 3072 | ⭐⭐⭐ | OpenAI 付费 | 英文场景 |
| m3e-base | 768 | ⭐⭐⭐⭐ | 本地部署 | 离线/隐私敏感 |
| bge-large-zh | 1024 | ⭐⭐⭐⭐⭐ | 本地部署 | 学术/研究 |

伯乐选择 BGE-M3 的三个理由：
1. **中英文双优** — 面试题库中英文混合
2. **1024 维** — 性价比最佳
3. **SiliconFlow 免费 API** — 降低使用门槛

---

## 四、手写一个 Embedding 相似度计算

```python
import numpy as np
from openai import OpenAI

client = OpenAI(
    api_key="your-key",
    base_url="https://api.siliconflow.cn/v1"
)

def embed(text: str) -> list[float]:
    """把文字转成向量"""
    resp = client.embeddings.create(
        model="Pro/BAAI/bge-m3",
        input=text,
    )
    return resp.data[0].embedding

# 语义相似度测试
sentences = [
    "Java 中的 HashMap 是如何工作的？",
    "请解释 HashMap 的底层实现原理",
    "今天中午吃什么好呢？",
    "ArrayList 和 LinkedList 有什么区别？",
]

target = "HashMap 的 put 方法做了什么？"
target_vec = embed(target)

print(f"目标问题：「{target}」\n")
for s in sentences:
    s_vec = embed(s)
    sim = np.dot(target_vec, s_vec) / (
        np.linalg.norm(target_vec) * np.linalg.norm(s_vec)
    )
    print(f"  [{sim:.3f}] 「{s}」")

# 输出示例：
# [0.912] 「Java 中的 HashMap 是如何工作的？」
# [0.885] 「请解释 HashMap 的底层实现原理」
# [0.051] 「今天中午吃什么好呢？」       ← 明显不相关
# [0.423] 「ArrayList 和 LinkedList 有什么区别？」← 都是集合，有一定相似度
```

---

## 下一步学习

阅读 [[04-sse-websocket-basics|SSE 与 WebSocket 协议基础]]，理解实时通信的两种方式。
