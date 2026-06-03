# Wiki 索引

> 本文件是 Wiki 的内容目录，LLM 每次摄入时更新。

## 统计

| 类型 | 数量 |
|------|------|
| 概念页 | 19 |
| 实体页 | 9 |
| 摘要页 | 0 |
| 对比页 | 5 |
| 概述页 | 3 |
| 学习笔记 | 154 |

---

## 概念（concepts/）

### Transformer 核心组件
- [[wiki/concepts/self-attention]] — 序列内部各位置互相关注的基础注意力机制
- [[wiki/concepts/multi-head-attention]] — 多视角并行关注，拆分为多个低维子空间捕捉不同依赖关系
- [[wiki/concepts/positional-encoding]] — 通过数学函数向词嵌入注入位置信号，解决 Self-Attention 置换等变性问题
- [[wiki/concepts/feed-forward-network]] — Position-wise 前馈网络，对每个 token 独立做非线性特征提取，占 Transformer 约 2/3 参数
- [[wiki/concepts/residual-connection]] — 绕过非线性变换的梯度高速公路，让深层网络可训练
- [[wiki/concepts/layer-normalization]] — 逐样本归一化，稳定深层网络的数值分布

### 编码器-解码器架构
- [[wiki/concepts/encoder-decoder]] — Transformer 核心架构范式：编码器理解输入，解码器生成输出
- [[wiki/concepts/cross-attention]] — 连接编码器与解码器的桥梁，Q 来自解码器、K/V 来自编码器的跨序列注意力
- [[wiki/concepts/masked-attention]] — 因果遮罩机制，屏蔽未来位置确保自回归生成的因果性

### 训练与推理
- [[wiki/concepts/autoregressive-generation]] — 语言模型逐 token 生成的核心范式：每步输出成为下步输入，直到生成结束符
- [[wiki/concepts/beam-search]] — 自回归生成中选择下一个 token 的解码策略对比：贪心、Beam Search、Top-k、Top-p、Temperature
- [[wiki/concepts/kv-cache]] — 缓存 Key-Value 向量避免重复计算，将推理复杂度从 O(T²) 降至 O(T)

### 现代扩展
- [[wiki/concepts/scaling-law]] — 模型性能随参数规模、数据量和计算量的幂律关系增长，大模型涌现全新能力
- [[wiki/concepts/mixture-of-experts]] — 通过稀疏激活机制，用少量计算量驱动庞大参数空间的 MoE 架构

### LangGraph / Agent 工作流
- [[wiki/concepts/state-graph]] — LangGraph 最核心的图构建原语：围绕共享状态组织节点、边与执行流程
- [[wiki/concepts/langgraph-api-map]] — 面向系统学习的 LangGraph API 路线图：从 `StateGraph` 到 `langgraph dev` 的能力地图

### 知识库维护
- [[wiki/concepts/series-navigation-consistency]] — 系列课程的章节顺序、导航链接、路线图一致性检查清单与错误预防

### Wiki 方法论
- [[wiki/concepts/knowledge-compilation]] — 让 LLM 一次性编译原始资料为结构化知识，而非每次重新检索
- [[wiki/concepts/LLM Wiki]] — 用 LLM 自动构建和维护 Markdown 个人知识库的模式

## 实体（entities/）

### 模型/架构
- [[wiki/entities/Transformer]] — 2017 年革命性的注意力驱动序列建模架构
- [[wiki/entities/BERT]] — Google 2018 年提出的 Encoder-Only 双向预训练模型，革新 NLP 理解任务
- [[wiki/entities/GPT]] — OpenAI 的 Decoder-Only 自回归语言模型系列，验证了规模法则
- [[wiki/entities/T5]] — Google 2019 年提出的 Encoder-Decoder 模型，统一所有 NLP 任务为 Text-to-Text 生成

### 人物
- [[wiki/entities/Ashish Vaswani]] — Transformer 论文第一作者，提出"只用 Attention，不用递归"的革命性架构
- [[wiki/entities/Dzmitry Bahdanau]] — 注意力机制先驱，2014 年提出 Bahdanau Attention，打破 Seq2Seq 信息瓶颈
- [[wiki/entities/Jay Alammar]] — 技术教育家，The Illustrated Transformer 等可视化教程作者
- [[wiki/entities/Andrej Karpathy]] — AI 研究员、OpenAI 联合创始人、LLM Wiki 提出者

### 工具
- [[wiki/entities/Obsidian]] — 本地优先的 Markdown 笔记应用，LLM Wiki 的前端界面

## 摘要（summaries/）

（暂无）

## 对比分析（comparisons/）

- [[wiki/comparisons/rnn-vs-transformer|RNN vs Transformer]] — 序列建模两大架构的全面对比：并行性、梯度路径、感受野与适用场景
- [[wiki/comparisons/positional-encoding-comparison|位置编码方案对比]] — 正弦编码 / 可学习编码 / RoPE / ALiBi 四种方案的优劣与选型指南
- [[wiki/comparisons/layernorm-vs-batchnorm|LayerNorm vs BatchNorm]] — 两种归一化方法的核心差异与 Transformer 选择 LayerNorm 的原因
- [[wiki/comparisons/encoder-only-vs-decoder-only-vs-encoder-decoder|Encoder-Only vs Decoder-Only vs Encoder-Decoder]] — 三种 Transformer 架构变体的设计哲学、核心差异与选型指南
- [[wiki/comparisons/dense-vs-moe|Dense vs MoE]] — Dense 模型与混合专家架构在激活方式、计算量和适用场景上的对比

## 综合概述（overviews/）

- [[wiki/overviews/transformer-architecture-overview|Transformer 架构全景概述]] — 从输入到输出的完整数据流、组件关系图与设计哲学
- [[wiki/overviews/transformer-training-inference-overview|Transformer 训练与推理概述]] — 训练流程关键步骤、推理策略、训练 vs 推理核心差异
- [[wiki/overviews/modern-llm-landscape|现代 LLM 格局概述]] — 三条技术路线、效率优化技术、2017-2026 发展脉络与架构选型指南

## 学习笔记（lessons/）

### Transformer 深度学习系列（8 篇）

- [[01-why-transformer|第 1 章：为什么需要 Transformer]] — 序列建模的困局与破局
- [[02-self-attention|第 2 章：Self-Attention]] — 序列自我审视的核心引擎
- [[03-multi-head-attention|第 3 章：Multi-Attention]] — 多视角并行关注
- [[04-positional-encoding|第 4 章：位置编码]] — 让 Attention 知道"在哪里"
- [[05-ffn-residual-layernorm|第 5 章：FFN、残差连接与 Layer Normalization]]
- [[06-encoder-decoder|第 6 章：编码器-解码器架构]] — 从零件到整机
- [[07-training-inference|第 7 章：训练与推理]] — 从理论到实践的桥梁
- [[08-beyond-original|第 8 章：超越原始 Transformer]] — BERT、GPT 与现代变体

### 大模型从零训练系列（10 篇）

- [[raw/lessons/LLM-Training/01-llm-training-overview|第 1 章：大模型训练全景]] — 训练流程总览、Pretrain→SFT→RLHF 全流程、nano-LLM 项目设计
- [[raw/lessons/LLM-Training/02-data-engineering|第 2 章：数据工程]] — 数据来源、清洗管道、质量过滤、去重策略、数据配比
- [[raw/lessons/LLM-Training/03-tokenizer-design|第 3 章：分词器设计]] — BPE/WordPiece/Unigram 算法原理、SentencePiece 实践、中文分词
- [[raw/lessons/LLM-Training/04-modern-llm-architecture|第 4 章：现代 LLM 架构]] — Decoder-Only 演进、SwiGLU、RMSNorm、RoPE、GQA、nano-LLM 架构
- [[raw/lessons/LLM-Training/05-distributed-training|第 5 章：分布式训练]] — 数据并行/张量并行/流水线并行、ZeRO/FSDP/DeepSpeed
- [[raw/lessons/LLM-Training/06-training-optimization|第 6 章：训练优化技术]] — 混合精度训练、Flash Attention、梯度累积、Cosine with Warmup
- [[raw/lessons/LLM-Training/07-pretraining-practice|第 7 章：预训练实战]] — nano-LLM 完整训练代码、训练监控、推理测试
- [[raw/lessons/LLM-Training/08-supervised-finetuning|第 8 章：有监督微调（SFT）]] — 指令数据构建、LoRA/QLoRA 高效微调原理与实现
- [[raw/lessons/LLM-Training/09-rlhf-dpo|第 9 章：RLHF 与 DPO]] — 奖励模型、PPO、DPO 直接偏好优化、对齐实战
- [[raw/lessons/LLM-Training/10-evaluation-deployment|第 10 章：评估与部署]] — Benchmark 评估、模型量化、推理优化、本地部署

### 肉鸽塔防挂机游戏开发系列（15 篇）

- [[raw/lessons/game/01-game-dev-fundamentals|第 1 章：游戏开发基础]] — 知识体系总览、游戏类型分析、核心概念与学习路线图
- [[raw/lessons/game/02-game-engine-selection|第 2 章：游戏引擎选型]] — Cocos Creator / Unity / Godot 对比，微信小游戏首选 Cocos
- [[raw/lessons/game/03-roguelike-core|第 3 章：Roguelike 核心机制]] — 柏林诠释、可控随机性、Run 结构、Meta-progression、协同效应
- [[raw/lessons/game/04-procedural-map-generation|第 4 章：程序化地图生成]] — BSP / 元胞自动机 / 随机游走 / WFC 算法详解
- [[raw/lessons/game/05-tower-defense-core|第 5 章：塔防核心机制]] — A* 寻路、塔/敌人设计、波次系统、经济平衡
- [[raw/lessons/game/06-idle-system-design|第 6 章：挂机系统设计]] — 成长曲线数学模型、离线收益、Prestige 转生、数值膨胀控制
- [[raw/lessons/game/07-numerical-balance|第 7 章：数值平衡设计]] — DPS 计算、经济循环、难度曲线、属性预算、配表设计
- [[raw/lessons/game/08-combat-system|第 8 章：战斗系统设计]] — 有限状态机、碰撞检测、弹道系统、Buff/Debuff、伤害结算
- [[raw/lessons/game/09-cocos-creator-practice|第 9 章：Cocos Creator 实战]] — 项目框架、场景管理、TiledMap 集成、微信小游戏适配
- [[raw/lessons/game/10-ui-interaction-design|第 10 章：UI 与交互设计]] — 拇指热区、信息层级、动效设计、新手引导
- [[raw/lessons/game/11-save-data-management|第 11 章：存档与数据管理]] — 数据结构、加密防篡改、版本迁移、云存档同步
- [[raw/lessons/game/12-performance-optimization|第 12 章：性能优化与调试]] — DrawCall 合批、对象池、GC 优化、内存管理
- [[raw/lessons/game/13-publishing-and-operations|第 13 章：游戏发布与运营]] — 微信小游戏发布、变现策略、数据分析、社区运营
- [[raw/lessons/game/14-full-project-walkthrough|第 14 章：完整项目实战]] — 从 0 到 1 构建肉鸽塔防挂机游戏的完整路线图
- [[raw/lessons/game/15-ai-assisted-game-dev|第 15 章：AI 辅助微信小游戏开发]] — Claude Code + cocos-mcp-server + CLAUDE.md 配置，AI 辅助开发完整工作流

### 微信小游戏行业分析

- [[raw/lessons/game/wechat-minigame-viral-analysis|微信小游戏爆火机制深度分析]] — 羊了个羊/抓大鹅/向僵尸开炮等爆款案例拆解、六大爆火机制、用户画像、商业模式与成瘾心理学
- [[raw/lessons/game/minigame-viral-deep-dive|微信爆款小游戏爆火机制深度调研（v2）]] — 6款爆款深度拆解、两条爆款路径对比、Hook上瘾模型分析、IAA/IAP/混合变现模型、2025-2026行业趋势

### 科研开源工具指南

- [[raw/lessons/research-tools/research-open-source-tools-guide|科研开源工具全景指南]] — 25+ 款科研工具全景梳理：学术写作、论文阅读、文献管理、数据分析、图表绘制、自动化研究（beginner）

### Research-to-TopConf 系列（6 篇）

- [[raw/lessons/Research-to-TopConf/00-essence|00 总览：从零到顶会论文的路线图总览]] — 将“进入新方向 → 读综述 → 读论文 → 生 idea → 筛方向”压缩成一套可执行研究路线图，并把主线聚焦到多智能体通信幻觉与协议重构
- [[raw/lessons/Research-to-TopConf/01-entering-new-field|第 1 章：如何从零进入一个陌生研究方向]] — 建立对象 / 问题 / 机制 / 验证四层问题框架，以及 survey / core papers / benchmark / framework / people 五类入口
- [[raw/lessons/Research-to-TopConf/02-survey-reading-strategy|第 2 章：为什么先读综述，以及如何从综述中提取技术地图与研究空白]] — 把综述阅读变成问题地图、方法地图、评测地图、挑战地图与 future-work 地图的提取过程
- [[raw/lessons/Research-to-TopConf/03-reading-papers-critically|第 3 章：如何批判性阅读论文，而不是只抄摘要]] — 区分 prompt / role / workflow / protocol / benchmark 五层改动，抓住 claim、实验设计与 limitation
- [[raw/lessons/Research-to-TopConf/04-idea-generation|第 4 章：如何从文献、异常与矛盾中生成研究 idea]] — 用异常、矛盾、局限、迁移、组合五类来源，把“看出问题”推进成“形成可验证假设”
- [[raw/lessons/Research-to-TopConf/05-direction-selection|第 5 章：如何筛选研究方向并把 idea 收敛成可执行切口]] — 用创新性、可行性、评测可得性、资源成本、叙事潜力五维框架收敛候选方向，并优先锁定协议层切口

### LangGraph 系统学习系列（10 篇）

> 当前项目实战示例已统一刷新为：`langchain` 必须使用 `1.x`、`langgraph` 按最新稳定版主线学习、DeepSeek（`deepseek-chat`）作为默认模型入口，并补充了 Tool Calling 深度讲解与 DeepSeek 多接法对比。

- [[raw/lessons/LangGraph/01-langgraph-overview|第 1 章：LangGraph 学习地图与研究助手项目总览]] — 为什么学 LangGraph、10 章固定路线图、Research Copilot 项目主线、嵌入式 Python 补课地图
- [[raw/lessons/LangGraph/02-stategraph-basics|第 2 章：StateGraph、节点与状态——让研究助手先跑起来]] — `StateGraph`、节点、边、`START` / `END`、最小可运行 Research Copilot、`TypedDict` 状态设计
- [[raw/lessons/LangGraph/03-state-and-messages|第 3 章：State 与 Messages——让研究助手真正进入对话态]] — `MessagesState`、`add_messages`、消息 reducer、Research Copilot v1 对话态升级
- [[raw/lessons/LangGraph/04-tools-and-react|第 4 章：工具与 ReAct——让研究助手学会调用检索工具]] — `@tool`、`ToolNode`、`tools_condition`、最小 ReAct 风格工具回路
- [[raw/lessons/LangGraph/05-checkpointer-memory|第 5 章：记忆与检查点——让研究助手跨轮记住上下文]] — `MemorySaver`、`SqliteSaver`、`thread_id`、线程级状态延续
- [[raw/lessons/LangGraph/06-human-in-the-loop|第 6 章：人在回路——让研究助手在关键步骤停下来等你审核]] — `interrupt()`、`Command(resume=...)`、审核流与暂停恢复
- [[raw/lessons/LangGraph/07-streaming|第 7 章：流式执行——实时看见研究助手正在如何思考与推进]] — `stream()`、`stream_mode`、`version="v2"`、执行过程可观测性
- [[raw/lessons/LangGraph/08-send-and-parallel|第 8 章：并行与分发——让研究助手同时处理多个检索任务]] — `Send`、动态扇出、聚合规则、最小 map-reduce 风格
- [[raw/lessons/LangGraph/09-subgraphs-and-composition|第 9 章：子图与组合——把研究助手拆成可复用模块]] — 子图接入父图、共享状态、私有状态、模块化 Research Copilot
- [[raw/lessons/LangGraph/10-deployment-and-beyond|第 10 章：部署与扩展——把研究助手整理成本地可运行应用]] — `langgraph dev`、`langgraph.json`、应用结构整理、Functional API 视角

### LangGraph 到 Harness 桥接系列（2 篇）

- [[raw/lessons/LangGraph-to-Harness/01-from-graph-to-runtime|第 1 章：从 Graph 到 Runtime——为什么下一步是 LangChain 1.x 与 Harness]] — 从 LangGraph 课程过渡到 LangChain 1.x 与 Claude Code-like harness 的系统桥接，总结你已掌握的 runtime 骨架，并明确后续双轨路线
- [[raw/lessons/LangGraph-to-Harness/02-from-primitives-to-system-modules|第 2 章：从原语到系统模块——LangGraph 能力如何落到真实 Agent 架构]] — 把 state、node、edge、checkpoint、interrupt、subgraph 等 LangGraph 原语稳定映射到 session、planner、executor、permission gate、review 等真实系统模块

### LangChain 1.x 系统学习（6 篇）

- [[raw/lessons/LangChain/01-why-langchain-1-x|第 1 章：为什么 LangChain 1.x 是下一步]] — 明确后续只以 LangChain 1.x 为主线，理解它在模型、消息、工具、结构化输出与 agent abstraction 上的标准组件层价值
- [[raw/lessons/LangChain/02-langchain-1-x-core-abstractions|第 2 章：LangChain 1.x 的分层结构与核心抽象]] — 建立 model、messages、tools、structured output、agent abstraction 的统一组件地图，并明确它们在 Harness 中的系统落点
- [[raw/lessons/LangChain/03-models-messages-and-provider-boundaries|第 3 章：Models、Messages 与 Provider Boundaries]] — 讲清模型接口、消息对象和 provider 差异边界，帮助后续做干净的 provider adapter 与 conversation state
- [[raw/lessons/LangChain/04-tools-and-structured-output|第 4 章：Tools 与 Structured Output——让结果可调用、可验证、可组合]] — 讲清工具调用协议、ToolMessage 回写、schema 思维、provider-native structured output 与 tool strategy 的系统意义
- [[raw/lessons/LangChain/05-prompt-and-context-organization|第 5 章：Prompt 与上下文组织——什么应放在哪一层]] — 讲清模型真正看见的是哪些消息、runtime context 为什么默认不可见，以及 system / task / history / tool results 该如何分层组织，才能稳定支撑 tool calling 与 structured output
- [[raw/lessons/LangChain/06-memory-state-history-boundaries|第 6 章：Memory / State / History——哪些属于组件层，哪些属于系统层]] — 讲清 messages history、conversation state 与 cross-conversation store 的边界，明确 short-term memory 更接近 state，long-term memory 更接近 store，以及哪些职责属于 LangChain 组件层、哪些仍要交给 harness 决策

### Agent Engineering 与 Claude Code-like Harness（8 篇）

- [[raw/lessons/Agent-Engineering/01-claude-code-like-systems-overview|第 1 章：Claude Code-like 系统总览]] — 从公开能力面拆解 coding agent 的最小模块，明确 MVP / intermediate / advanced 三层工程路线
- [[raw/lessons/Agent-Engineering/02-harness-mvp-architecture|第 2 章：Harness MVP 架构——一个最小 coding agent 系统怎么拆]] — 讲清最小 coding agent 的骨架：交互、session、planner、executor、tools、permission、review、logging、config 为什么缺一不可
- [[raw/lessons/Agent-Engineering/03-planner-executor-reviewer-loop|第 3 章：Planner / Executor / Reviewer——受控执行回路是怎样形成的]] — 讲清任务如何在规划、执行、复查之间形成可纠偏的主回路，并说明 permission gate 在其中的闸门位置
- [[raw/lessons/Agent-Engineering/04-tool-layer-read-search-edit-run|第 4 章：工具层——读文件、搜代码、编辑文件、运行命令为什么构成最小 action surface]] — 把 `read / search / edit / run` 解释为 coding agent 的最小动作面，并说明为什么工具层是系统对外动作的法定出口
- [[raw/lessons/Agent-Engineering/05-permissions-and-confirmation-boundaries|第 5 章：权限与确认边界——为什么高风险动作不能直接放行]] — 讲清 allow / ask / deny 三类决策、确认机制的中断语义，以及权限边界如何决定 agent 的行动权
- [[raw/lessons/Agent-Engineering/06-session-memory-checkpoint-resume|第 6 章：Session、Memory、Checkpoint、Resume——为什么不能把状态全塞进一个记忆盒子]] — 区分会话状态、长期记忆、可恢复快照与恢复流程，避免把所有状态混成一个大容器
- [[raw/lessons/Agent-Engineering/07-task-decomposition-and-multi-step-strategy|第 7 章：任务分解与多步执行策略——复杂任务为什么不能只靠临场乱试]] — 讲清任务形状判断、顺序 / 分层 / 条件分支拆解、局部上下文打包，以及 retry / 局部补救 / replan 怎样在真实 harness 中协同发生
- [[raw/lessons/Agent-Engineering/08-project-understanding-and-repo-exploration|第 8 章：项目理解与仓库探索——为什么 agent 不能一上来就盲改代码]] — 讲清目录结构扫描、模式发现、局部精读三层仓库探索策略，以及探索结果如何被压缩成后续步骤可复用的项目上下文对象

### AI 工程师面试准备系列（24 篇）

#### 总览
- [[raw/lessons/AI-Interview-Prep/00-overview|00 总览：面试知识图谱与学习路线]] — 五大核心领域分布、高频面试题占比、面试应对策略模板

#### LLM 架构（4 篇）
- [[raw/lessons/AI-Interview-Prep/01-llm-architecture|01 LLM 架构深度（概览）]] — Transformer原理、注意力变体、训练三阶段、推理优化、主流模型对比
- [[raw/lessons/AI-Interview-Prep/01a-transformer-deep|01a Transformer 内部机制深入]] — Self-Attention完整数学推导、MHA/MQA/GQA/MLA对比、RoPE推导、SwiGLU/RMSNorm、Pre-Norm vs Post-Norm
- [[raw/lessons/AI-Interview-Prep/01b-training-pipeline|01b LLM 训练管线深入]] — 预训练数据工程(MinHash+LSH)、SFT(LoRA/QLoRA数学推导)、RLHF(Bradley-Terry/PPO)、DPO闭式推导及变体
- [[raw/lessons/AI-Interview-Prep/01c-inference-optim|01c LLM 推理优化与部署]] — KV Cache显存计算、量化(GPTQ/AWQ/SmoothQuant/FP8)、Flash Attention Tiling、Speculative Decoding等价证明、vLLM/TensorRT-LLM对比

#### AI Agent（4 篇）
- [[raw/lessons/AI-Interview-Prep/02-ai-agent|02 AI Agent 系统（概览）]] — ReAct/Plan-and-Execute/反思范式、Function Calling机制、Multi-Agent协作模式、框架对比
- [[raw/lessons/AI-Interview-Prep/02a-agent-paradigms|02a Agent 推理范式深度]] — CoT理论基础(计算/信息论/训练数据)、ReAct完整模板与失败模式、Plan-Execute架构、Reflexion反思、结构化输出、Prompt Injection防御
- [[raw/lessons/AI-Interview-Prep/02b-function-calling|02b Function Calling 深入]] — 完整HTTP请求/响应JSON、多Provider差异(OpenAI/Anthropic/DeepSeek/GLM)、工具设计最佳实践、Structured Output、复杂工具编排代码、安全防护
- [[raw/lessons/AI-Interview-Prep/02c-multi-agent|02c 多智能体协作深度]] — 4种协作模式完整设计、通信协议、LangGraph深入实践(完整ReAct Agent代码)、6大框架架构对比、Token成本控制/调试/可观测性

#### RAG（4 篇）
- [[raw/lessons/AI-Interview-Prep/03-rag|03 RAG 检索增强生成（概览）]] — Naive→Advanced→Modular演进、Embedding选型、向量数据库对比、Chunk策略、混合检索+Rerank、RAGAS评估
- [[raw/lessons/AI-Interview-Prep/03a-rag-indexing|03a RAG 索引与文档处理深入]] — PDF解析工具对比、5种Chunk策略完整实现代码、Embedding训练原理(对比学习/InfoNCE)、BM25公式推导、MTEB模型对比、HNSW/IVF/PQ索引算法
- [[raw/lessons/AI-Interview-Prep/03b-rag-retrieval|03b RAG 检索优化深入]] — HyDE/Step-back/Query Decomposition四种改写策略及代码、RRF融合公式与代码、Cross-Encoder vs Bi-Encoder对比、ColBERT/SPLADE、Self-RAG/CRAG/Graph RAG/Agentic RAG架构
- [[raw/lessons/AI-Interview-Prep/03c-rag-production|03c RAG 高级架构与生产实践]] — GraphRAG完整架构(微软)、Agentic RAG(Self-RAG/CRAG)、多模态RAG(CLIP/ColPali)、企业级设计(权限控制/增量索引/缓存策略)、性能优化

#### Memory（3 篇）
- [[raw/lessons/AI-Interview-Prep/04-memory|04 AI Memory 记忆系统（概览）]] — 短期/长期/工作记忆分类、上下文窗口管理、MemGPT架构、对话状态追踪、记忆治理策略
- [[raw/lessons/AI-Interview-Prep/04a-memory-systems|04a 记忆系统架构深入]] — 三层记忆分类学完整对比、Token预算分配代码、滑动窗口+滚动摘要完整实现、MemGPT/Letta架构详解(核心操作+Prompt设计)、记忆写入/检索策略完整代码、用户画像系统设计
- [[raw/lessons/AI-Interview-Prep/04b-memory-advanced|04b 高级记忆与前沿]] — DST对话状态追踪(形式化定义+LLM+Pydantic完整代码)、记忆压缩与遗忘(四种衰减机制代码)、跨会话记忆生命周期管理、多用户记忆隔离(三层架构+多租户设计)、评估指标体系、Stanford小镇等前沿研究

#### Python（4 篇）
- [[raw/lessons/AI-Interview-Prep/05-python-mastery|05 Python 精通（概览）]] — 装饰器/元类/描述符高级特性、asyncio并发编程、内存管理与GC、设计模式、性能优化
- [[raw/lessons/AI-Interview-Prep/05a-python-advanced|05a Python 高级特性深入]] — 装饰器闭包原理与5种完整模式(缓存/重试/限流/校验/异步)、元类4大应用(单例/注册表/ORM/接口)、描述符完整查找链与4种实现、生成器/协程/yield from、Pydantic v2类型系统
- [[raw/lessons/AI-Interview-Prep/05b-python-concurrency|05b Python 并发编程深入]] — GIL机制(check interval/3.12+per-interpreter/PEP 703)、Threading同步原语、asyncio事件循环内幕、Multiprocessing IPC、5种AI并发模式(异步API网关/流式响应/并行工具调用/批量推理/任务队列)
- [[raw/lessons/AI-Interview-Prep/05c-python-engineering|05c Python 工程实践深入]] — 内存管理(引用计数11场景/分代GC/pymalloc/泄漏排查)、性能分析5工具+8优化模式、6种AI设计模式(Factory/Strategy/Observer/Template/CoR/Singleton)、错误处理(CircuitBreaker)、8道面试编程题完整解答

#### 业界热点系统（4 篇）
- [[raw/lessons/AI-Interview-Prep/06-hot-topics|06 业界热点系统深度分析（概览）]] — 热点系统知识地图、面试考察频率、与前面知识的连接映射、3天学习路线
- [[raw/lessons/AI-Interview-Prep/06a-hot-agent-systems|06a Agent 热点系统]] — Claude Code完整架构(Agent循环/工具系统/多智能体/MCP协议/权限模型)、Cursor vs Windsurf vs Claude Code三方对比、Manus多Agent并行架构、OpenHands事件驱动Agent SDK
- [[raw/lessons/AI-Interview-Prep/06b-hot-memory-systems|06b Memory 热点系统]] — Claude Code三层CLAUDE.md+MEMORY.md记忆架构、Mem0三层数据库(Vector+KV+Graph)与记忆生命周期、ChatGPT Memory自动管理、Cursor/Windsurf上下文策略、编码Agent记忆系统设计题
- [[raw/lessons/AI-Interview-Prep/06c-hot-llm-systems|06c LLM 热点架构]] — DeepSeek-V3 MLA完整数学推导(~66x压缩)+MoE 256专家+MTP、SGLang RadixAttention(Radix Tree前缀复用)+CFSM、推理模型(o1 RL+R1纯RL)

### 总规划（1 篇）

- [[raw/lessons/full-plan|LangGraph → LangChain 1.x → Claude Code-like Harness 学习与写作总规划]] — 三条系列的单一权威路线图，统一规定角色分工、联动方式、写作阶段、里程碑与防漂移规则

### 千万字男频神话灵异小说创作系列（18 篇）

#### 模块一：网文商业基础
- [[raw/lessons/novel-writing/01-web-novel-fundamentals|第 1 章：网文写作全景]] — 行业概况、写作心态、常见误区与收入概览
- [[raw/lessons/novel-writing/02-fanqie-platform-guide|第 2 章：番茄小说平台攻略]] — 签约规则、收入构成、等级体系、推荐算法
- [[raw/lessons/novel-writing/03-golden-three-chapters|第 3 章：黄金三章与读者留存]] — 开篇三章的写作公式、案例分析、灵异类特殊技巧

#### 模块二：世界观设定
- [[raw/lessons/novel-writing/04-worldbuilding-mythology|第 4 章：神话体系搭建]] — 中国神话素材、原创体系构建方法论、四层世界观架构
- [[raw/lessons/novel-writing/05-worldbuilding-supernatural|第 5 章：灵异规则与力量体系]] — 规则设计、等级体系、金手指设计、灵异事件模板
- [[raw/lessons/novel-writing/06-worldbuilding-map-faction|第 6 章：地图架构与势力设计]] — 多层级地图、五大势力类型、势力平衡与冲突

#### 模块三：人物塑造
- [[raw/lessons/novel-writing/07-protagonist-design|第 7 章：主角设计与成长弧光]] — 三种主角类型、三维画像法、弧光节奏、金手指结合
- [[raw/lessons/novel-writing/08-ensemble-cast-design|第 8 章：群像塑造技法]] — 四级角色金字塔、视角切换、配角出彩技法、时间管理
- [[raw/lessons/novel-writing/09-antagonist-npc-design|第 9 章：反派与 NPC 立体化]] — 四级反派体系、好反派设计、NPC 快速立体化、关系网

#### 模块四：大纲与结构
- [[raw/lessons/novel-writing/10-outline-mega-structure|第 10 章：千万字大纲架构]] — 三层大纲体系、五阶段规划、10卷大纲示例
- [[raw/lessons/novel-writing/11-multi-thread-narrative|第 11 章：多线叙事设计]] — 五种线型、四种交织技法、线头管理与收束
- [[raw/lessons/novel-writing/12-pacing-rhythm|第 12 章：节奏控制与卷章划分]] — 三级节奏体系、灵异特殊节奏、卷划分策略

#### 模块五：叙事技法
- [[raw/lessons/novel-writing/13-foreshadowing-techniques|第 13 章：伏笔埋设与回收]] — 六种高级伏笔技法、回收技巧、千万字伏笔管理
- [[raw/lessons/novel-writing/14-climax-emotional-design|第 14 章：燃点与刀子设计]] — 五类燃点、五类刀子、燃刀节奏公式、燃刀一体设计
- [[raw/lessons/novel-writing/15-suspense-hook-design|第 15 章：悬念制造与断章技巧]] — 三层悬念、五种断章法、灵异小说悬念特殊技巧

#### 模块六：参考作品拆解
- [[raw/lessons/novel-writing/16-analysis-horror-house|第 16 章：《我有一座恐怖屋》拆解]] — 恐怖屋经营+探险双线、恐怖氛围营造、伏笔设计
- [[raw/lessons/novel-writing/17-analysis-slash-god|第 17 章：《斩神》拆解]] — 多神话融合世界观、燃刀设计、群像塑造、守护主题
- [[raw/lessons/novel-writing/18-analysis-mysterious-revival|第 18 章：《神秘复苏》拆解]] — 规则怪谈式恐怖、拼图力量体系、伏笔误导

---

## 研究工作区（research/）

> 学术论文研究专用工作流。从选题到投稿全流程管理。

### 目录结构

| 目录 | 用途 |
|------|------|
| [[research/_inbox/]] | 每日灵感和想法暂存 |
| [[research/01-topics/]] | 研究方向管理 |
| [[research/02-papers/]] | 论文阅读笔记 |
| [[research/03-experiments/]] | 实验日志与结果 |
| [[research/04-writing/]] | 论文草稿与版本 |
| [[research/05-submission/]] | 投稿与审稿意见 |

### 模板

| 模板 | 用途 |
|------|------|
| [[_templates/paper-note]] | 论文阅读笔记模板 |
| [[_templates/experiment-log]] | 实验日志模板 |
| [[_templates/research-idea]] | 研究想法记录模板 |
| [[_templates/research-topic]] | 研究方向规划模板 |
| [[_templates/paper-writing]] | 论文写作模板 |

### 当前研究

#### 方向 1: 模型压缩与高效部署

- [[01-topics/model-compression-efficient-deployment]] — 方向总览（架构设计/动态稀疏/量化/蒸馏/Mamba）
- [[01-topics/reading-plan]] — 4周精读计划
- [[_inbox/first-reading-ideas]] — MobileViT 知识蒸馏想法

#### 方向 2: Agent 蜂群动态协作拓扑 ⭐ 执行中

- [[01-topics/agent-swarm-dynamic-topology]] — SwarmTopo 研究方案（群体智能 + LLM Agent 动态拓扑适配）
- [[01-topics/CLAUDE-SwarmTopo]] — Claude Code 项目指导文档
- [[01-topics/implementation-plan]] — 14 周完整实施方案（逐日任务 + Claude Code 提示词 + 验收标准）

#### 方向 3: Agent Memory — LLM 智能体记忆机制 🆕 探索中

- [[research/01-topics/agent-memory]] — Agent Memory 研究方向总览（记忆分类体系、核心论文、创新空间分析）
- [[research/_inbox/agent-memory-ideas]] — Agent Memory 研究灵感（多Agent一致性、图结构记忆、层次化反思等5个想法）
- **综述论文**：
  - [[research/02-papers/_surveys/agent-memory-survey-zhang2024|A Survey on Memory Mechanism of LLM Agents (2024)]]
  - [[research/02-papers/_surveys/lifelong-learning-llm-agent-zheng2026|Lifelong Learning of LLM Agents (TPAMI 2026)]]
- **经典论文**：
  - [[research/02-papers/_foundations/generative-agents-park2023|Generative Agents (Park 2023)]]
  - [[research/02-papers/_foundations/memgpt-packer2023|MemGPT (Packer 2023)]]
  - [[research/02-papers/_foundations/reflexion-shinn2023|Reflexion (Shinn 2023)]]
  - [[research/02-papers/_foundations/memorybank-zhong2023|MemoryBank (Zhong 2023)]]

#### 方向 4: Closed-loop Safe Multi-Agent Reasoning ⭐ 选定

- [[research/01-topics/closed-loop-safe-multi-agent-reasoning]] — 基于 G2CP 与 GUARDIAN 的统一研究设计（预防 / 检测 / 恢复 / 学习闭环）
- 核心基础：[[raw/lessons/G2CP/00-essence|G2CP]]、[[raw/lessons/GUARDIAN/00-essence|GUARDIAN]]、[[raw/lessons/MARCH/00-essence|MARCH]]
- 支撑导读：[[research/02-papers/multi-agent-hallucination-reading-guide|多智能体幻觉文献导读与阅读路线]]
- 当前定位：通用问答 / 推理多智能体；中成本；方法为主 + 轻量闭环评测扩展

#### 多智能体综述阅读包（2026-05-07）

- [[raw/paper/multi-agent/00-reading-manual|多智能体综述论文阅读手册]] — 面向入门与研究选题准备的推荐阅读顺序、筛选理由与阅读重点
- 论文 PDF：
  - [[raw/paper/multi-agent/01-large-language-model-based-multi-agents-survey-of-progress-and-challenges-2024.pdf]]
  - [[raw/paper/multi-agent/02-a-survey-on-llm-based-multi-agent-system-recent-advances-and-new-frontiers-in-application-2024.pdf]]
  - [[raw/paper/multi-agent/03-multi-agent-collaboration-mechanisms-a-survey-of-llms-2025.pdf]]
  - [[raw/paper/multi-agent/04-beyond-self-talk-a-communication-centric-survey-of-llm-based-multi-agent-systems-2025.pdf]]
  - [[raw/paper/multi-agent/05-llms-working-in-harmony-a-survey-on-the-technological-aspects-of-building-effective-llm-based-multi-agent-systems-2025.pdf]]
  - [[raw/paper/multi-agent/06-creativity-in-llm-based-multi-agent-systems-a-survey-2025.pdf]]
- 学习友好版中文精读系列（第 1 篇已完成）：
  - [[raw/lessons/Multi-Agent-Survey/01-progress-and-challenges/00-overview|Multi-Agent Survey 中文精读系列总览（第 1 篇）]]
  - [[raw/lessons/Multi-Agent-Survey/01-progress-and-challenges/01-survey-of-progress-and-challenges-introduction|01 导论：问题背景、研究动机与论文结构]]
  - [[raw/lessons/Multi-Agent-Survey/01-progress-and-challenges/02-survey-of-progress-and-challenges-background|02 背景：单智能体能力与单体/多体差异]]
  - [[raw/lessons/Multi-Agent-Survey/01-progress-and-challenges/03-survey-of-progress-and-challenges-dissecting-llm-ma|03 系统拆解：环境接口、画像设定、通信与能力获取]]
  - [[raw/lessons/Multi-Agent-Survey/01-progress-and-challenges/04-survey-of-progress-and-challenges-applications|04 应用：问题求解与世界模拟]]
  - [[raw/lessons/Multi-Agent-Survey/01-progress-and-challenges/05-survey-of-progress-and-challenges-supporting-tools-and-challenges|05 框架、基准、挑战与结论]]
  - [[raw/lessons/Multi-Agent-Survey/01-progress-and-challenges/06-survey-of-progress-and-challenges-references|06 参考文献全文保留]]
	- 学习友好版中文精读系列（第 2 篇已完成）：
	  - [[raw/lessons/Multi-Agent-Survey/02-recent-advances/10-recent-advances-and-new-frontiers-overview|10 总览：Recent Advances and New Frontiers in Application]]
	  - [[raw/lessons/Multi-Agent-Survey/02-recent-advances/11-recent-advances-and-new-frontiers-introduction|11 摘要、导论与应用框架]]
	  - [[raw/lessons/Multi-Agent-Survey/02-recent-advances/12-recent-advances-and-new-frontiers-core-components|12 核心组件：生成式智能体与环境]]
	  - [[raw/lessons/Multi-Agent-Survey/02-recent-advances/13-recent-advances-and-new-frontiers-solving-complex-tasks|13 复杂任务求解：推理框架、通信优化、资源与评测]]
	  - [[raw/lessons/Multi-Agent-Survey/02-recent-advances/14-recent-advances-and-new-frontiers-simulating-specific-scenarios|14 特定场景模拟：社会、物理、游戏]]
	  - [[raw/lessons/Multi-Agent-Survey/02-recent-advances/15-recent-advances-and-new-frontiers-evaluating-generative-agents|15 评估生成式智能体：评价与训练]]
	  - [[raw/lessons/Multi-Agent-Survey/02-recent-advances/16-recent-advances-and-new-frontiers-challenges-future-and-conclusion|16 挑战、未来方向、结论与局限]]
	  - [[raw/lessons/Multi-Agent-Survey/02-recent-advances/17-recent-advances-and-new-frontiers-references|17 参考文献全文保留]]
- 学习友好版中文精读系列（第 3 篇已完成）：
  - [[raw/lessons/Multi-Agent-Survey/03-collaboration-mechanisms/00-collaboration-mechanisms-essence|00 论文精华总览：Collaboration Mechanisms 结构化入口]]
  - [[raw/lessons/Multi-Agent-Survey/03-collaboration-mechanisms/01-collaboration-mechanisms-introduction-background|01 摘要、导论与背景]]
  - [[raw/lessons/Multi-Agent-Survey/03-collaboration-mechanisms/02-collaboration-mechanisms-concept|02 协作机制概念与形式化定义]]
  - [[raw/lessons/Multi-Agent-Survey/03-collaboration-mechanisms/03-collaboration-mechanisms-types|03 协作类型：合作、竞争与竞合]]
  - [[raw/lessons/Multi-Agent-Survey/03-collaboration-mechanisms/04-collaboration-mechanisms-strategies|04 协作策略：规则、角色与模型驱动]]
  - [[raw/lessons/Multi-Agent-Survey/03-collaboration-mechanisms/05-collaboration-mechanisms-structures|05 通信结构：中心化、去中心化与层级化]]
  - [[raw/lessons/Multi-Agent-Survey/03-collaboration-mechanisms/06-collaboration-mechanisms-coordination-and-lessons|06 协调编排与经验总结]]
  - [[raw/lessons/Multi-Agent-Survey/03-collaboration-mechanisms/07-collaboration-mechanisms-applications|07 应用版图]]
  - [[raw/lessons/Multi-Agent-Survey/03-collaboration-mechanisms/08-collaboration-mechanisms-open-problems-and-conclusion|08 开放问题与结论]]
  - [[raw/lessons/Multi-Agent-Survey/03-collaboration-mechanisms/09-collaboration-mechanisms-references|09 参考文献索引]]
- 学习友好版中文精读系列（第 4 篇已完成）：
  - [[raw/lessons/Multi-Agent-Survey/04-beyond-self-talk/00-beyond-self-talk-essence|00 论文精华总览：Beyond Self-Talk 结构化入口]]
  - [[raw/lessons/Multi-Agent-Survey/04-beyond-self-talk/01-beyond-self-talk-background|01 摘要、导论与背景]]
  - [[raw/lessons/Multi-Agent-Survey/04-beyond-self-talk/02-beyond-self-talk-system-level|02 系统间通信：架构、目标与协议]]
  - [[raw/lessons/Multi-Agent-Survey/04-beyond-self-talk/03-beyond-self-talk-system-internal|03 系统内通信：策略、范式、对象与内容]]
  - [[raw/lessons/Multi-Agent-Survey/04-beyond-self-talk/04-beyond-self-talk-challenges-and-conclusion|04 挑战、未来方向与结论]]
- 学习友好版中文精读系列（第 5 篇已完成）：
  - [[raw/lessons/Multi-Agent-Survey/05-llms-working-in-harmony/00-llms-harmony-essence|00 论文精华总览：LLMs Working in Harmony 结构化入口]]
  - [[raw/lessons/Multi-Agent-Survey/05-llms-working-in-harmony/01-llms-harmony-introduction-and-architecture|01 导论与架构：CMD/CoA/Agent Forest/MoA]]
  - [[raw/lessons/Multi-Agent-Survey/05-llms-working-in-harmony/02-llms-harmony-planning|02 规划框架：AdaPlanner/ChatCoT/KnowAgent/RAP/ToT/ReAct]]
  - [[raw/lessons/Multi-Agent-Survey/05-llms-working-in-harmony/03-llms-harmony-memory|03 记忆框架：VecDB/RAG/ChatDB/MemoryBank/RET-LLM/SCM]]
  - [[raw/lessons/Multi-Agent-Survey/05-llms-working-in-harmony/04-llms-harmony-frameworks-and-conclusion|04 框架工具、方法论、讨论与结论]]
- 学习友好版中文精读系列（第 6 篇已完成）：
  - [[raw/lessons/Multi-Agent-Survey/06-creativity/00-creativity-mas-essence|00 论文精华总览：Creativity in MAS 结构化入口]]
  - [[raw/lessons/Multi-Agent-Survey/06-creativity/01-creativity-mas-introduction-and-workflow|01 导论与工作流：三阶段模型与主动性谱系]]
  - [[raw/lessons/Multi-Agent-Survey/06-creativity/02-creativity-mas-techniques|02 创造力技术：发散探索/迭代精炼/协作合成]]
  - [[raw/lessons/Multi-Agent-Survey/06-creativity/03-creativity-mas-persona|03 Persona 设计：三层粒度与三种生成方式]]
  - [[raw/lessons/Multi-Agent-Survey/06-creativity/04-creativity-mas-evaluation|04 评估方法：客观指标/主观TTCT/交互评估]]
  - [[raw/lessons/Multi-Agent-Survey/06-creativity/05-creativity-mas-challenges-and-conclusion|05 挑战、未来方向与结论]]
- 复用模板：[[_templates/paper-translation-lesson|paper-translation-lesson]] — 面向”全文翻译 + 非原文标注 + 系列导航”的论文学习笔记模板

### DebUnc 论文精读系列（5 篇）

- [[raw/lessons/DebUnc/00-essence|00 精华总结]] — DebUnc 核心贡献、方法概述、实验结果、局限与改进方向
- [[raw/lessons/DebUnc/01-introduction|01 引言]] — LLM 幻觉问题、多智能体辩论及其局限、DebUnc 动机与四大贡献
- [[raw/lessons/DebUnc/02-related-work|02 相关工作]] — 三类不确定性度量方法（token 概率/LLM 生成/采样）与多智能体辩论研究
- [[raw/lessons/DebUnc/03-method|03 方法]] — 不确定性度量数学定义、置信度转换、Confidence in Prompt 与 Attention Scaling 实现
- [[raw/lessons/DebUnc/04-experiments|04 实验与结果]] — Mistral-7B/Llama-3-8B 跨基准评估、Oracle 分析、AUROC 相关性分析

### G2CP 论文精读系列（6 篇）

- [[raw/lessons/G2CP/00-essence|00 精华总结]] — G2CP 核心贡献、协议定义、实验结果（准确率+34%/token-73%/幻觉-91%）、局限与改进
- [[raw/lessons/G2CP/01-introduction|01 引言]] — 多智能体自然语言通信三大问题、四类歧义、G2CP 核心思想与四大贡献
- [[raw/lessons/G2CP/02-related-work|02 相关工作]] — ACL 历史（KQML/FIPA/承诺语义）、多智能体 LLM 系统、知识图谱与智能体系统
- [[raw/lessons/G2CP/03-protocol|03 协议定义]] — 形式化定义、操作语义（TRAVERSE/UPDATE）、三大属性、消息语法、承诺语义
- [[raw/lessons/G2CP/04-architecture|04 多智能体架构]] — 四种智能体角色、协调协议、完整工作示例、运行时引擎、LLM 动态操作选择
- [[raw/lessons/G2CP/05-experiments|05 实验评估]] — 实验设置、总体性能、按类别分析、消融实验、复杂诊断案例、真实工业验证

### InEx 论文精读系列（6 篇）

- [[raw/lessons/InEx/00-essence|00 精华总结]] — InEx 核心贡献、In+Ex 双模块设计、TVER/VE-MHA、跨模态共识、实验结果（4%-27% 提升）
- [[raw/lessons/InEx/01-introduction|01 引言]] — MLLM 幻觉问题、三类方法局限、人类认知范式启发、InEx 框架概述与贡献
- [[raw/lessons/InEx/02-related-work|02 相关工作]] — 幻觉检测与缓解方法分类、多智能体协作方法
- [[raw/lessons/InEx/03-preliminaries|03 预备知识]] — MLLM 架构、熵理论、TVER 定义与推导、VE-MHA 注意力头遮蔽机制
- [[raw/lessons/InEx/04-method|04 方法论]] — Internal Introspection + External Cross-Modal Collaboration 完整实现、跨模态迭代验证流程
- [[raw/lessons/InEx/05-experiments|05 实验与结果]] — 通用基准、幻觉基准、消融实验、鲁棒性分析、统计显著性

### GUARDIAN 论文精读系列（7 篇）

- [[raw/lessons/GUARDIAN/00-essence|00 精华总结]] — GUARDIAN 核心贡献、时序属性图建模、信息瓶颈图抽象、异常节点检测、幻觉/错误传播防御效果与局限
- [[raw/lessons/GUARDIAN/01-introduction|01 引言]] — 多智能体协作中的幻觉放大与错误注入问题、现有方法局限、GUARDIAN 动机与贡献
- [[raw/lessons/GUARDIAN/02-related-work|02 相关工作]] — 多智能体协作框架、图异常检测、信息瓶颈与时序图学习等背景脉络
- [[raw/lessons/GUARDIAN/03-framework|03 时序属性图框架]] — 节点/边/时间步建模、交互图表示、整体防御框架与信息流结构
- [[raw/lessons/GUARDIAN/04-method|04 方法]] — 编码器-解码器架构、重构目标、GIB 损失、异常分数计算与增量训练机制
- [[raw/lessons/GUARDIAN/05-experiments|05 实验]] — 幻觉放大与错误注入双场景评测、检测率/FDR/API 调用/运行时间对比与消融分析
- [[raw/lessons/GUARDIAN/06-conclusion|06 结论]] — 结论、局限、适用边界与后续改进方向

### CortexDebate 论文精读系列（7 篇）

- [[raw/lessons/CortexDebate/00-essence|00 精华总结]] — CortexDebate 核心贡献、稀疏辩论图 + MDM 模块、McKinsey Trust Formula 适配、8 数据集全面领先、输入上下文最大缩减 70.79%
- [[raw/lessons/CortexDebate/01-introduction|01 引言]] — MAD 背景、冗长上下文与过度自信两大问题、脑白质类比、MDM 模块设计动机
- [[raw/lessons/CortexDebate/02-related-work|02 相关工作]] — 顺序辩论 vs 并行辩论分类、全辩论/固定部分辩论/动态稀疏辩论三分法对比
- [[raw/lessons/CortexDebate/03-method|03 稀疏辩论图方法]] — 有向图形式化定义、三阶段框架、MDM 四因素、稀疏化策略、置信度重校准
- [[raw/lessons/CortexDebate/04-trust-formula|04 McKinsey Trust Formula 与 MDM 详解]] — 四因素深度解读、消融实验、I/S 因子效果量化、剪枝策略对比
- [[raw/lessons/CortexDebate/05-experiments|05 实验]] — 8 数据集 × 8 方法主实验、上下文缩减分析、大规模 1000 样本验证
- [[raw/lessons/CortexDebate/06-conclusion|06 结论与局限性]] — 两大局限、Prompt 设计、基线方法详解、整体评价与不足

### MARCH 论文精读系列（6 篇）

- [[raw/lessons/MARCH/00-essence|00 精华总结]] — MARCH 核心贡献、三智能体信息不对称流水线、零容忍奖励、MARL 联合优化、跨基准显著提升与局限
- [[raw/lessons/MARCH/01-introduction|01 引言]] — RAG 幻觉与确认偏误问题、为什么现有 LLM-as-a-judge 失效、MARCH 的核心动机
- [[raw/lessons/MARCH/02-methodology|02 方法论]] — Solver-Proposer-Checker 三角色设计、信息不对称机制、ZTR 奖励与 PPO 联合训练
- [[raw/lessons/MARCH/03-experiments|03 实验]] — RAGTruth/FaithBench/Facts Grounding/多跳 QA 等评测结果、消融实验与泛化验证
- [[raw/lessons/MARCH/04-related-work|04 相关工作]] — RAG 幻觉检测、RLHF/RLVR、自检与验证器设计的研究脉络
- [[raw/lessons/MARCH/05-conclusion|05 结论]] — 结论、局限与未来工作方向

### MUG 论文精读系列（7 篇）

- [[raw/lessons/MUG/00-essence|00 精华总结]] — MUG 核心贡献、反事实卧底博弈机制、关键公式、实验结果（HallusionBench +16.0%）、局限与改进
- [[raw/lessons/MUG/01-introduction|01 引言]] — MLLM 幻觉问题、MAD 的幻觉漏洞、"谁是卧底"社交推理游戏启发、MUG 核心思想与三大创新
- [[raw/lessons/MUG/02-related-work|02 相关工作]] — 多模态推理演进、多智能体系统（MAD/Self-Refine）、反事实推理
- [[raw/lessons/MUG/03-mug-protocol|03 协议设计]] — 博弈系统形式化定义、问题表示、角色分配、游戏模式、反事实图像生成三约束
- [[raw/lessons/MUG/04-counterfactual-test|04 反事实测试机制]] — 卧底检测游戏（推理/投票/淘汰）、四投票因子、终止条件、策略动态、总结游戏
- [[raw/lessons/MUG/05-experiments|05 实验与分析]] — 四基准评估、SOTA 对比、幻觉检测性能、消融实验、游戏进程分析、案例研究与时间开销
- [[raw/lessons/MUG/06-conclusion|06 结论与附录]] — 核心贡献总结、MUG 算法伪代码、Prompt 设计详解

### 伯乐 AI 面试平台深度拆解系列（9 篇）🆕

> 基于 LangGraph v1 + FastAPI + Vue3 的全栈 AI 面试与知识库平台完整拆解

- [[raw/lessons/ai-interview/00-essence|00 精华总览]] — 项目定位、技术栈全景图、核心架构决策、数据流全景、学习路线图
- [[raw/lessons/ai-interview/01-architecture-overview|01 整体架构深度剖析]] — 分层设计、服务拓扑、Docker 编排、中间件栈、配置优先级体系
- [[raw/lessons/ai-interview/02-langgraph-agent-system|02 LangGraph Agent 系统深度拆解]] — BaseAgent 生命周期、BaseContext 可配置上下文、中间件体系、Checkpointer 双后端策略
- [[raw/lessons/ai-interview/03-interview-agent-deep-dive|03 面试 Agent 深度剖析]] — 6 阶段面试流程、TodoList 驱动机制、System Prompt 工程、简历上下文注入
- [[raw/lessons/ai-interview/04-rag-knowledge-system|04 RAG 知识库系统]] — OpenViking 向量引擎、BGE-M3 Embedding、分块策略、混合检索与 Rerank
- [[raw/lessons/ai-interview/05-document-parsing-pipeline|05 文档解析管线]] — MinerU/PaddleX/DeepSeek OCR/RapidOCR/Docling 多解析器对比与容错策略
- [[raw/lessons/ai-interview/06-streaming-chat-and-agent-run|06 流式对话与 Agent Run 系统]] — SSE 流式传输、ARQ 任务队列、Redis Stream 事件推送
- [[raw/lessons/ai-interview/07-voice-interview-system|07 语音面试系统]] — WebSocket 双向通信、豆包 TTS、Fun-ASR 实时语音识别、VAD 断句
- [[raw/lessons/ai-interview/08-frontend-architecture|08 前端架构深度剖析]] — Vue3/Pinia/Ant Design/CSS 变量主题/路由守卫
- [[raw/lessons/ai-interview/09-deployment-and-production|09 部署与生产落地实战]] — 24 个踩坑总结、Checkpointer/SSE/WebSocket/LLM API/安全/监控

#### 基础概念补课（6 篇）🆕
- [[raw/lessons/ai-interview/foundations/01-what-is-rag|01 RAG 是什么]] — 检索增强生成完整原理、50行代码手搓最小RAG、Naive→Advanced→Modular三阶段
- [[raw/lessons/ai-interview/foundations/02-langgraph-agent-basics|02 LangGraph 与 Agent 基础]] — LangChain/LangGraph区别、Agent循环机制、ReAct模式、State/Node/Edge/Checkpointer核心概念、中间件体系
- [[raw/lessons/ai-interview/foundations/03-embedding-and-rerank|03 Embedding 与 Rerank 基础]] — 文字→向量的原理、余弦相似度、Bi-Encoder vs Cross-Encoder、BGE-M3选型理由
- [[raw/lessons/ai-interview/foundations/04-sse-websocket-basics|04 SSE 与 WebSocket 协议]] — HTTP→SSE→WebSocket演进、数据格式、伯乐的双协议选型
- [[raw/lessons/ai-interview/foundations/05-docker-compose-basics|05 Docker Compose 基础]] — Dockerfile/Image/Container概念、compose文件结构、容器通信、开发vs生产
- [[raw/lessons/ai-interview/foundations/06-fastapi-vue-basics|06 FastAPI 与 Vue3 基础]] — 异步路由、Pydantic验证、响应式系统、Pinia、Vue Router
