---
type: lesson
tags: [Docker, Docker Compose, 容器化, 基础知识, 部署]
created: 2026-05-25
updated: 2026-05-25
difficulty: beginner
prerequisites: [Linux基础命令]
topic: 开源项目深度拆解
status: completed
series: { name: "ai-interview-伯乐深度拆解-基础篇", part: 5 }
---

# Docker Compose 基础概念

> 伯乐项目完全通过 Docker Compose 管理。为什么需要容器化？Dockerfile 和 docker-compose.yml 分别做什么？多个服务之间如何通信？本文从零讲清。

---

## 一、为什么需要 Docker？

### 没有 Docker 的世界

```
开发者A：macOS, Python 3.12
开发者B：Windows, Python 3.11
服务器：  Ubuntu, Python 3.10

"我的机器上能跑啊！" → 经典问题
```

### 有 Docker 的世界

```
所有环境： Docker 容器
  ├── Python 3.12（固定的）
  ├── PostgreSQL 16（固定的）
  ├── Redis 7（固定的）
  └── 所有依赖（固定的）

"只要能跑 Docker，就能跑这个项目"
```

Docker 把你应用的**运行环境**打包成一个镜像（Image），在任何地方运行都一致。

---

## 二、核心概念三件套

### Dockerfile — 如何构建镜像

```dockerfile
# 伯乐的 API Dockerfile（简化版）

# ① 基础镜像
FROM python:3.12-slim

# ② 安装系统依赖
RUN apt-get update && apt-get install -y curl

# ③ 安装 Python 依赖
COPY pyproject.toml uv.lock /app/
RUN pip install uv && uv sync

# ④ 复制应用代码
COPY src/ /app/src/
COPY server/ /app/server/

# ⑤ 定义启动命令
CMD ["uv", "run", "uvicorn", "server.main:app", "--host", "0.0.0.0", "--port", "5050"]
```

### Image（镜像）— 构建好的"模板"

```
docker build -t bole-api:0.5 .
# 根据 Dockerfile 构建镜像
# 镜像包含：操作系统+Python+依赖+代码
```

### Container（容器）— 运行中的镜像实例

```
docker run bole-api:0.5
# 根据镜像启动一个容器
# 可以同时启动多个容器（都基于同一个镜像）
```

**类比**：Dockerfile = 菜谱，Image = 做好的半成品，Container = 端上桌的菜。

---

## 三、Docker Compose — 管理多个容器

### 为什么需要 Compose？

伯乐项目需要 5+ 个容器同时运行：

```
api (FastAPI)  ─┐
worker (ARQ)   ─┤
web (Vue.js)   ─┼── 它们之间还需要通信！
postgres       ─┤
redis          ─┘
```

一个个手动 `docker run` 太麻烦了。Docker Compose 让你用一个 YAML 文件定义所有服务。

### docker-compose.yml 结构

```yaml
services:                    # 定义所有服务（容器）
  api:
    build: .                 # 用当前目录的 Dockerfile 构建
    ports: ["5050:5050"]    # 端口映射：宿主机:容器
    depends_on: [postgres]   # 依赖 postgres 先启动
    environment:             # 环境变量
      - POSTGRES_URL=postgresql://...
    volumes:                 # 目录挂载
      - ./src:/app/src       # 代码改动实时同步（热重载）

  postgres:
    image: postgres:16       # 用官方镜像，不需要自己构建
    environment:
      - POSTGRES_DB=ai_interview

  redis:
    image: redis:7-alpine
```

### 常用命令

```bash
docker compose up -d          # 后台启动所有服务
docker compose down           # 停止并删除所有容器
docker compose ps             # 查看运行状态
docker compose logs -f api    # 跟踪 api 服务日志
docker compose exec api bash  # 进入 api 容器的 shell
docker compose up -d --build # 重新构建镜像并启动
```

---

## 四、容器间网络通信

Docker Compose 自动创建网络，服务之间通过**服务名**互访：

```yaml
# 在 api 容器内：
# postgres 不是 localhost！而是服务名 "postgres"
POSTGRES_URL=postgresql://postgres:5432/ai_interview
#                            ↑ 直接用服务名，Docker 自动解析为容器 IP
```

---

## 五、数据持久化 — Volume 挂载

容器删除后，里面的数据就没了。需要把数据存到宿主机：

```yaml
services:
  postgres:
    volumes:
      - ./docker/volumes/postgresql:/var/lib/postgresql/data
      #   ↑ 宿主机路径                  ↑ 容器内路径
```

这样即使删除容器，数据库文件还在 `./docker/volumes/postgresql/` 里。

---

## 六、开发 vs 生产

伯乐用两套 compose 文件：

| 特性 | docker-compose.yml（开发） | docker-compose.prod.yml（生产） |
|------|--------------------------|-------------------------------|
| 代码挂载 | ✅ `./src:/app/src` | ❌ 不挂载（镜像内已打包） |
| 热重载 | ✅ `--reload` | ❌ |
| 端口 | 5173（Vite）、5050 | 80（Nginx） |
| 多 worker | ❌ | ✅ `--workers 4` |

---

## 下一步学习

阅读 [[06-fastapi-vue-basics|FastAPI 与 Vue3 基础概念]]，理解前后端框架。
