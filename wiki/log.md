# Wiki 操作日志

> 本文件按时间顺序记录所有 Wiki 操作。格式：`## [日期] 操作类型 | 简要描述`

## [2026-06-02] learn | RAG 安全论文精读系列（2 篇论文，13 篇逐节翻译笔记）🆕

**PoisonedRAG（USENIX Security 2025）— 7 篇逐节翻译**：
- [[raw/lessons/RAG-Security/01-PoisonedRAG/01-introduction|01-摘要与引言]]：RAG局限性、攻击面发现、威胁模型、贡献概览
- [[raw/lessons/RAG-Security/01-PoisonedRAG/02-background|02-背景与相关工作]]：RAG形式化定义、提示注入/越狱/数据投毒的局限分析
- [[raw/lessons/RAG-Security/01-PoisonedRAG/03-problem-formulation|03-问题形式化]]：威胁模型三维刻画、知识污染攻击的约束优化形式化
- [[raw/lessons/RAG-Security/01-PoisonedRAG/04-attack-design|04-攻击设计（核心方法论）]]：两个必要条件推导、文本分解策略、黑盒/白盒方案
- [[raw/lessons/RAG-Security/01-PoisonedRAG/05-experiments|05-实验评估]]：3数据集×8LLM×3检索器、5基线对比、消融、Wikipedia/Agent实测
- [[raw/lessons/RAG-Security/01-PoisonedRAG/06-defense|06-防御方法评估]]：释义/困惑度/重复过滤/知识扩展——均不足以防御
- [[raw/lessons/RAG-Security/01-PoisonedRAG/07-discussion-conclusion|07-讨论与结论]]：任务泛化、非目标副作用<1%、失败分析、未来方向

**SafeRAG（ACL 2025）— 6 篇逐节翻译**：
- [[raw/lessons/RAG-Security/02-SafeRAG/01-introduction|01-摘要与引言]]：RAG安全四大盲区、四类升级攻击的动机
- [[raw/lessons/RAG-Security/02-SafeRAG/02-related-work|02-相关工作]]：RAG安全评测数据集全景表、SafeRAG在四类攻击构造上的创新
- [[raw/lessons/RAG-Security/02-SafeRAG/03-attack-taxonomy|03-四类攻击任务详解]]：银噪声/上下文间冲突/软广告/白DoS的形式化定义
- [[raw/lessons/RAG-Security/02-SafeRAG/04-benchmark-framework|04-基准框架]]：数据集构建流程、评测Pipeline、14组件矩阵、攻击特定指标
- [[raw/lessons/RAG-Security/02-SafeRAG/05-experiments|05-实验结果]]：噪声/冲突/毒性/DoS系统性分析、反直觉发现：更强模型≠更安全
- [[raw/lessons/RAG-Security/02-SafeRAG/06-conclusion|06-结论]]：局限分析、与PoisonedRAG深度对比、两者互补关系

- 更新了 [[wiki/index.md]]：学习笔记从 141 篇扩展为 154 篇

## [2026-05-25] learn | 伯乐 AI 面试平台 — 基础概念补课系列（6 篇）🆕

- 创建了 [[raw/lessons/ai-interview/foundations/01-what-is-rag|01 RAG 是什么]]：从零讲清 RAG 完整原理、50行最小代码、Naive→Advanced→Modular 三阶段进化
- 创建了 [[raw/lessons/ai-interview/foundations/02-langgraph-agent-basics|02 LangGraph 与 Agent 基础]]：LangChain vs LangGraph 区别、Agent 循环原理、ReAct 模式、State/Node/Edge/Checkpointer、中间件体系
- 创建了 [[raw/lessons/ai-interview/foundations/03-embedding-and-rerank|03 Embedding 与 Rerank 基础]]：文字→向量原理、余弦相似度、对比学习训练、Bi-Encoder vs Cross-Encoder、BGE-M3 选型理由
- 创建了 [[raw/lessons/ai-interview/foundations/04-sse-websocket-basics|04 SSE 与 WebSocket 协议]]：HTTP 局限、SSE 格式与用法、WebSocket 握手升级、伯乐对话用 SSE 语音用 WS 的选型逻辑
- 创建了 [[raw/lessons/ai-interview/foundations/05-docker-compose-basics|05 Docker Compose 基础]]：Dockerfile/Image/Container 概念、compose 文件结构、容器间通信、Volume 持久化、开发 vs 生产分离
- 创建了 [[raw/lessons/ai-interview/foundations/06-fastapi-vue-basics|06 FastAPI 与 Vue3 基础]]：路由/依赖注入/流式响应/Pydantic 验证、响应式系统/Pinia/Vue Router/Vite
- 更新了 [[wiki/index.md]]：学习笔记从 135 篇扩展为 141 篇

## [2026-05-25] learn | 伯乐 AI 面试平台深度拆解系列（9 篇）

- 创建了 [[raw/lessons/ai-interview/00-essence|00 精华总览]]：项目定位、技术栈全景图、核心架构决策、数据流全景、学习路线图
- 创建了 [[raw/lessons/ai-interview/01-architecture-overview|01 整体架构深度剖析]]：分层设计、服务拓扑、Docker 编排、中间件执行顺序、异常处理三层体系
- 创建了 [[raw/lessons/ai-interview/02-langgraph-agent-system|02 LangGraph Agent 系统深度拆解]]：BaseAgent 生命周期管理、BaseContext 配置优先级体系、11 层中间件栈逐层分析、Checkpointer 双后端策略、ContextVar 请求上下文
- 创建了 [[raw/lessons/ai-interview/03-interview-agent-deep-dive|03 面试 Agent 深度剖析]]：6 阶段面试流程、TodoListMiddleware 驱动机制、System Prompt 工程 10 条规则、VideoContextMiddleware 视频分析增强
- 创建了 [[raw/lessons/ai-interview/04-rag-knowledge-system|04 RAG 知识库系统]]：OpenViking 向量引擎、BGE-M3 1024 维 Embedding、General/QA/Book/Laws 分块策略、混合检索与 Rerank、知识库评估体系
- 创建了 [[raw/lessons/ai-interview/05-document-parsing-pipeline|05 文档解析管线]]：MinerU/PaddleX/DeepSeek OCR/RapidOCR/Docling 5 种解析器对比、Guard 质量检查、多解析器容错链
- 创建了 [[raw/lessons/ai-interview/06-streaming-chat-and-agent-run|06 流式对话与 Agent Run 系统]]：SSE 流式传输完整代码拆解、ARQ+Redis 异步任务队列、Run 生命周期与取消机制
- 创建了 [[raw/lessons/ai-interview/07-voice-interview-system|07 语音面试系统]]：WebSocket 双向通信、豆包 TTS + Fun-ASR 三阶段流水线、VAD 3 秒断句阈值、AudioContext 浏览器策略
- 创建了 [[raw/lessons/ai-interview/08-frontend-architecture|08 前端架构深度剖析]]：Vue3/Pinia/Ant Design/CSS 变量主题/路由守卫/AgentChat 组件
- 创建了 [[raw/lessons/ai-interview/09-deployment-and-production|09 部署与生产落地实战]]：24 个踩坑总结（Checkpointer/SSE 缓冲/WebSocket 保活/LLM 限流/长对话摘要等）
- 更新了 [[wiki/index.md]]：学习笔记从 126 篇扩展为 135 篇，新增伯乐系列 9 篇入口

## [2026-05-20] learn | 新增业界热点系统深度分析（第6站，4 篇）

- 创建了 [[raw/lessons/AI-Interview-Prep/06-hot-topics|06 业界热点系统深度分析（概览）]]：热点系统知识地图、面试考察频率表（8大系统）、与前面知识的连接映射、3天学习路线
- 创建了 [[raw/lessons/AI-Interview-Prep/06a-hot-agent-systems|06a Agent 热点系统]]：Claude Code完整架构（Agent循环/工具系统Python dict/多智能体/3级权限/MCP协议）、Cursor vs Windsurf vs Claude Code三方对比表、Manus（Planning/Execution/Memory + Wide Research 100并行Agent）、OpenHands事件驱动架构
- 创建了 [[raw/lessons/AI-Interview-Prep/06b-hot-memory-systems|06b Memory 热点系统]]：Claude Code三层CLAUDE.md+MEMORY.md架构、Mem0三层数据库(Vector+KV+Graph)与记忆生命周期、ChatGPT Memory自动管理、Cursor/Windsurf上下文策略对比、编码Agent记忆系统设计题
- 创建了 [[raw/lessons/AI-Interview-Prep/06c-hot-llm-systems|06c LLM 热点架构]]：DeepSeek-V3 MLA完整数学推导（低秩KV压缩~66x）+MoE 256细粒度专家+MTP、SGLang RadixAttention（Radix Tree前缀复用61%节省）+CFSM、推理模型(o1 RL+R1纯RL/Aha Moment)
- 更新了 [[raw/lessons/AI-Interview-Prep/00-overview]]：新增第6站导航和热点系统索引
- 更新了 [[wiki/index.md]]：AI 面试准备系列从 20 篇扩展为 24 篇

## [2026-05-20] learn | AI 面试准备系列扩展为3-4倍深度（14 篇新增）

- 创建了 [[raw/lessons/AI-Interview-Prep/01a-transformer-deep|01a Transformer 内部机制深入]]：Self-Attention完整数学推导、MHA/MQA/GQA/MLA对比、RoPE旋转矩阵推导、SwiGLU/RMSNorm、Pre-Norm vs Post-Norm、Embedding与采样策略
- 创建了 [[raw/lessons/AI-Interview-Prep/01b-training-pipeline|01b LLM 训练管线深入]]：预训练交叉熵推导与MinHash+LSH去重、SFT(Self-Instruct/Evol-Instruct/LoRA/QLoRA数学推导)、RLHF(Bradley-Terry/PPO四步流程)、DPO闭式推导及变体(IPO/ORPO/KTO)、数据工程
- 创建了 [[raw/lessons/AI-Interview-Prep/02a-agent-paradigms|02a Agent 推理范式深度]]：CoT三角度理论、ReAct完整模板与失败模式、Plan-Execute架构、Reflexion反思、结构化输出、人在回路、可观测性
- 创建了 [[raw/lessons/AI-Interview-Prep/02b-function-calling|02b Function Calling 深入]]：完整HTTP请求/响应JSON、多Provider差异、工具设计最佳实践、Structured Output与constrained generation、复杂工具编排、安全防护(注入/沙箱/权限)
- 创建了 [[raw/lessons/AI-Interview-Prep/02c-multi-agent|02c 多智能体协作深度]]：4种协作模式完整设计、通信协议、LangGraph深入(完整ReAct Agent代码)、6大框架架构对比、Token成本控制与调试
- 创建了 [[raw/lessons/AI-Interview-Prep/03a-rag-indexing|03a RAG 索引与文档处理深入]]：PDF解析5工具对比、5种Chunk策略完整代码、Embedding训练原理(对比学习/InfoNCE)、BM25完整公式、MTEB模型对比、HNSW/IVF/PQ索引算法
- 创建了 [[raw/lessons/AI-Interview-Prep/03b-rag-retrieval|03b RAG 检索优化深入]]：HyDE/Step-back/Query Decomposition策略与代码、RRF融合公式、Cross-Encoder vs Bi-Encoder、ColBERT/SPLADE、Self-RAG/CRAG/Graph RAG/Agentic RAG架构
- 创建了 [[raw/lessons/AI-Interview-Prep/03c-rag-production|03c RAG 高级架构与生产实践]]：GraphRAG(微软)完整架构、Agentic RAG、多模态RAG(CLIP/ColPali)、企业级设计(权限/增量索引/缓存)、性能优化与决策框架
- 创建了 [[raw/lessons/AI-Interview-Prep/04a-memory-systems|04a 记忆系统架构深入]]：三层记忆分类学、Token预算分配、滑动窗口+滚动摘要实现、MemGPT/Letta架构(4核心操作+Prompt)、记忆写入/检索完整代码、用户画像系统
- 创建了 [[raw/lessons/AI-Interview-Prep/04b-memory-advanced|04b 高级记忆与前沿]]：DST形式化定义+LLM+Pydantic完整代码、四种记忆衰减机制、跨会话生命周期管理、多用户三层隔离架构、评估指标体系、Stanford小镇等前沿
- 创建了 [[raw/lessons/AI-Interview-Prep/05a-python-advanced|05a Python 高级特性深入]]：装饰器闭包原理+5种模式(缓存/重试/限流/校验/异步)、元类4大应用、描述符完整查找链+4种实现、生成器/协程、Pydantic v2
- 创建了 [[raw/lessons/AI-Interview-Prep/05b-python-concurrency|05b Python 并发编程深入]]：GIL机制与演进、Threading同步原语+线程安全LLM客户端、asyncio事件循环内幕+异步RAG管线、Multiprocessing IPC、5种AI并发模式
- 创建了 [[raw/lessons/AI-Interview-Prep/05c-python-engineering|05c Python 工程实践深入]]：内存管理全栈(引用计数/分代GC/pymalloc/泄漏排查)、性能分析5工具+8优化模式、6种AI设计模式、CircuitBreaker、8道面试编程题完整解答
- 更新了 5 篇概览文件(01-05)：每篇新增"深度扩展阅读"导航表，链接到对应子文件
- 更新了 [[wiki/index.md]]：AI 面试准备系列从 8 篇扩展为 20 篇完整入口

## [2026-05-20] learn | 创建 AI 面试进阶学习笔记（2 篇）

- 创建了 [[raw/lessons/AI-Interview-Prep/01c-inference-optim|01c LLM 推理优化与部署]]：KV Cache 显存计算与 PagedAttention、量化数学原理(GPTQ/AWQ/SmoothQuant)、Flash Attention Tiling 策略、Speculative Decoding 数学证明、推理框架选型、服务化部署
- 创建了 [[raw/lessons/AI-Interview-Prep/02a-agent-paradigms|02a Agent 推理范式深度]]：CoT 变体对比、ReAct 完整模板与失败模式、Plan-and-Execute 架构、Reflexion 反思机制、Agent 可靠性工程、高级 Prompt Engineering
- 更新了 [[wiki/index.md]]：新增 2 篇面试进阶笔记入口

## [2026-05-19] learn | 创建 AI 工程师面试准备系列学习笔记（6 篇）

- 创建了 [[raw/lessons/AI-Interview-Prep/00-overview|面试总览页]]：知识图谱、学习路线、面试应对策略模板
- 创建了 [[raw/lessons/AI-Interview-Prep/01-llm-architecture|LLM 架构深度]]：Transformer原理、MHA/GQA/MLA注意力变体、训练三阶段、KV Cache/量化/Flash Attention推理优化、主流模型对比
- 创建了 [[raw/lessons/AI-Interview-Prep/02-ai-agent|AI Agent 系统]]：ReAct/Plan-and-Execute/反思范式、Function Calling机制、Multi-Agent协作模式、LangGraph/AutoGen/CrewAI框架对比
- 创建了 [[raw/lessons/AI-Interview-Prep/03-rag|RAG 检索增强生成]]：Naive→Advanced→Modular演进、Embedding模型选型、向量数据库对比、Chunk策略、混合检索+Rerank、RAGAS评估
- 创建了 [[raw/lessons/AI-Interview-Prep/04-memory|AI Memory 记忆系统]]：短期/长期/工作记忆分类、上下文窗口管理、MemGPT架构、对话状态追踪、记忆治理策略
- 创建了 [[raw/lessons/AI-Interview-Prep/05-python-mastery|Python 精通]]：装饰器/元类/描述符高级特性、asyncio并发编程、内存管理与GC、工厂/策略设计模式、LRU缓存/异步限流器等面试编程题
- 更新了 [[wiki/index.md]]：新增 AI 工程师面试准备系列入口

## [2026-05-17] research | 新建 Closed-loop Safe Multi-Agent Reasoning 研究设计草案

- 创建了 [[research/01-topics/closed-loop-safe-multi-agent-reasoning]]，汇总并结构化整理了基于 [[raw/lessons/G2CP/00-essence|G2CP]] 与 [[raw/lessons/GUARDIAN/00-essence|GUARDIAN]] 的统一研究设计
- 在草案中明确了论文主叙事：将多智能体推理中的错误传播建模为“部分可观测的动态风险控制问题”
- 形成了完整的前三部分设计：问题定义、四层闭环系统架构、状态/动作/奖励/学习目标
- 继续补写了“最小实验包与评测协议”，包括任务集、污染场景、baseline、指标、消融与 claim-evidence 绑定
- 更新了 [[wiki/index.md]]：新增“方向 4: Closed-loop Safe Multi-Agent Reasoning”入口

## [2026-05-17] research | 完善 Closed-loop Safe Multi-Agent Reasoning 的 baseline ladder 与协议边界

- 在 [[research/01-topics/closed-loop-safe-multi-agent-reasoning]] 中新增了 baseline ladder 与 prompt / protocol 具体定义
- 明确了 6 类关键对照：Single-Agent、MA-FreeText、MA-SemiStruct、G2CP-Static、GUARDIAN-Heuristic、ClosedLoop-Heuristic、Ours
- 补充了公平性边界：same data access / same tool access / matched budget / matched evaluation protocol
- 明确了不同 baseline 之间分别回答什么问题，避免后续实验被质疑 scaffold 不公平或 prompt 不公平

## [2026-05-17] research | 补写 Closed-loop Safe Multi-Agent Reasoning 的最小控制器实现方案

- 在 [[research/01-topics/closed-loop-safe-multi-agent-reasoning]] 中新增了“Belief Encoder 与 Policy Network 的最小实现方案”一节
- 明确建议采用中成本、可解释的最小实现：GRU-based belief encoder + hierarchical policy heads + offline policy learning
- 写清了 controller 的输入特征、belief 编码方式、分层动作头、训练路线与不建议采用的重实现路线
- 使研究文档从概念/实验设计进一步推进到可执行的系统实现层

## [2026-05-17] research | 补写 Closed-loop Safe Multi-Agent Reasoning 的论文图表清单

- 在 [[research/01-topics/closed-loop-safe-multi-agent-reasoning]] 中新增了“论文图表清单（Figure / Table Plan）”
- 预先规划了 5 个正文主图、3 个正文主表和多项附录图表，并把每个图表与其承载的核心 claim 绑定
- 明确了图与表的分工：图负责模式感知与 trade-off 展示，表负责精确 benchmark 数值与消融细节
- 记录了最应优先保证产出的最小可发表核心图表包，以及常见设计陷阱，方便后续实验反向对齐论文展示

## [2026-05-17] research | 补写 Closed-loop Safe Multi-Agent Reasoning 的摘要与引言叙事版本

- 在 [[research/01-topics/closed-loop-safe-multi-agent-reasoning]] 中新增了 reviewer-facing 的摘要 / 引言叙事版本
- 明确了应避免的叙事错误（模块拼装式开头、贡献点模块列表化、只写 hallucination 而不写传播性风险）
- 补入了标题候选、摘要 5 句式结构、摘要草案、引言 opening move、四段主线与 contribution phrasing
- 使研究文档从方法与实验设计进一步推进到论文写作 framing 层，便于后续直接扩展为 abstract 和 introduction

## [2026-05-16] learn | 新增 GUARDIAN 论文精读系列（7 篇）

- 创建了 [[raw/lessons/GUARDIAN/00-essence|00-essence]] 精华总结，提炼论文核心贡献（时序属性图建模 + 信息瓶颈图抽象 + 无监督异常检测）、NeurIPS 2025 元信息、双场景实验结果（幻觉放大 / 错误注入传播）与局限
- 创建了 [[raw/lessons/GUARDIAN/01-introduction|01-introduction]] 引言翻译，涵盖多智能体幻觉放大与错误注入问题、现有防御方法局限、GUARDIAN 的核心思想与贡献
- 创建了 [[raw/lessons/GUARDIAN/02-related-work|02-related-work]] 相关工作翻译，涵盖多智能体协作框架、时序图异常检测、信息瓶颈方法与相关防御研究
- 创建了 [[raw/lessons/GUARDIAN/03-framework|03-framework]] 框架翻译，包含时序属性图定义、节点/边/时间步建模、交互图表示与整体防御流程
- 创建了 [[raw/lessons/GUARDIAN/04-method|04-method]] 方法翻译，包含编码器-解码器架构、属性/结构重构目标、GIB 损失、异常分数与增量训练机制
- 创建了 [[raw/lessons/GUARDIAN/05-experiments|05-experiments]] 实验翻译，包含 MMLU/MATH/FEVER/Biographies 基准、与 LLM Debate/DyLAN/SelfCheckGPT 等方法对比、检测率/FDR/API 调用/耗时分析
- 创建了 [[raw/lessons/GUARDIAN/06-conclusion|06-conclusion]] 结论翻译，包含论文结论、适用边界、局限与未来改进方向
- 更新了 [[wiki/index.md]]：补入 "GUARDIAN 论文精读系列（7 篇）" 入口

## [2026-05-16] learn | 新增 MARCH 论文精读系列（6 篇）

- 创建了 [[raw/lessons/MARCH/00-essence|00-essence]] 精华总结，提炼论文核心贡献（三智能体信息不对称 + 零容忍奖励 + MARL 联合优化）、arXiv 2026 元信息、跨基准结果与局限
- 创建了 [[raw/lessons/MARCH/01-introduction|01-introduction]] 引言翻译，涵盖 RAG 场景中的上下文冲突型幻觉、确认偏误、现有验证方案的信息泄漏问题与 MARCH 动机
- 创建了 [[raw/lessons/MARCH/02-methodology|02-methodology]] 方法论翻译，包含 Solver / Proposer / Checker 三角色设计、信息不对称机制、ZTR 奖励与 PPO 训练目标
- 创建了 [[raw/lessons/MARCH/03-experiments|03-experiments]] 实验翻译，包含 RAGTruth / FaithBench / Facts Grounding / HotpotQA / MuSiQue / 2WikiMultiHopQA 评测、与闭源/开源模型对比、消融实验与泛化验证
- 创建了 [[raw/lessons/MARCH/04-related-work|04-related-work]] 相关工作翻译，涵盖 RAG 幻觉检测、RLHF/RLVR、自检与文档锚定验证器脉络
- 创建了 [[raw/lessons/MARCH/05-conclusion|05-conclusion]] 结论翻译，包含论文结论、方法局限与未来工作方向
- 更新了 [[wiki/index.md]]：补入 "MARCH 论文精读系列（6 篇）" 入口

## [2026-05-16] learn | 新增 MUG 论文精读系列（7 篇）

- 创建了 [[raw/lessons/MUG/00-essence|00-essence]] 精华总结，提炼论文核心贡献（反事实图像编辑 + 卧底博弈 + 主动探测幻觉智能体）、AAAI 2025 元信息、跨基准实验结果与局限
- 创建了 [[raw/lessons/MUG/01-introduction|01-introduction]] 引言翻译，涵盖多模态幻觉问题、MAD 的理性假设缺陷、"谁是卧底" 启发与 MUG 三大创新
- 创建了 [[raw/lessons/MUG/02-related-work|02-related-work]] 相关工作翻译，涵盖多模态推理、多智能体辩论、自反思方法与反事实推理脉络
- 创建了 [[raw/lessons/MUG/03-mug-protocol|03-mug-protocol]] 协议设计翻译，包含形式化问题定义、角色分配、游戏模式与反事实图像生成约束
- 创建了 [[raw/lessons/MUG/04-counterfactual-test|04-counterfactual-test]] 反事实测试机制翻译，包含卧底检测游戏、投票机制、终止条件、策略动态与总结游戏
- 创建了 [[raw/lessons/MUG/05-experiments|05-experiments]] 实验翻译，包含 MMMU / MMStar / HallusionBench / POPE 结果、消融实验、游戏轮次分析与时间开销
- 创建了 [[raw/lessons/MUG/06-conclusion|06-conclusion]] 结论翻译，包含核心结论、算法伪代码、Prompt 设计与局限性总结
- 更新了 [[wiki/index.md]]：补入 / 校正 MUG 系列记录，并同步学习笔记统计

## [2026-05-16] learn | 新增 CortexDebate 论文精读系列（7 篇）

- 创建了 [[raw/lessons/CortexDebate/00-essence|00-essence]] 精华总结，提炼论文核心贡献（稀疏辩论图 + MDM 模块）、McKinsey Trust Formula 四因素适配、8 数据集实验结果（平均 69.41%，输入上下文最大缩减 70.79%）、7 项局限与改进方向
- 创建了 [[raw/lessons/CortexDebate/01-introduction|01-introduction]] 引言翻译，涵盖 MAD 背景、冗长上下文与过度自信导致不平等辩论两大问题、脑白质类比、MDM 模块设计动机与三大贡献
- 创建了 [[raw/lessons/CortexDebate/02-related-work|02-related-work]] 相关工作翻译，涵盖顺序辩论 vs 并行辩论分类、7 种现有方法对比表、CortexDebate 的三分法定位（动态稀疏辩论）
- 创建了 [[raw/lessons/CortexDebate/03-method|03-method]] 方法论翻译，包含有向辩论图形式化定义、三阶段框架（初始答案/多轮辩论/最终答案）、MDM 四因素公式、稀疏化策略（AAT 平均阈值）、置信度重校准公式
- 创建了 [[raw/lessons/CortexDebate/04-trust-formula|04-trust-formula]] McKinsey Trust Formula 深度解读，包含四因素（Credibility/Reliability/Intimacy/Self-Orientation）详细数学定义、消融实验（7 种配置）、I/S 因子量化效果（CVR/DVC 从 33.96% 提升到 64.92%）、边剪枝策略对比、置信度重校准鲁棒性分析、文本相似度计算方法对比
- 创建了 [[raw/lessons/CortexDebate/05-experiments|05-experiments]] 实验翻译，包含 8 数据集描述表、3 类基线方法、骨干模型表（5 个 7B-9B 开源模型）、100 样本主实验结果表、输入上下文长度缩减分析、各轮次分数与共识比例变化表、智能体个体性能表、1000 样本大规模实验验证
- 创建了 [[raw/lessons/CortexDebate/06-conclusion|06-conclusion]] 结论与局限性翻译，包含论文结论、两大局限（效率/成本 + 受限于底层 LLM 能力）、初始答案生成与答案再生成 Prompt 设计、7 种基线方法详细介绍、致谢、整体评价（4 亮点 + 4 不足）
- 更新了 [[wiki/index.md]]：学习笔记统计 151 → 158，新增"CortexDebate 论文精读系列（7 篇）"入口

## [2026-05-16] learn | 新增 G2CP 论文精读系列（6 篇）

- 创建了 [[raw/lessons/G2CP/00-essence|00-essence]] 精华总结，提炼论文核心贡献（图操作替代自然语言通信）、协议定义、实验结果（准确率+34%/token-73%/幻觉-91%/级联错误-100%）、局限与改进
- 创建了 [[raw/lessons/G2CP/01-introduction|01-introduction]] 引言翻译，涵盖语义漂移/幻觉传播/计算浪费三大问题、四类歧义、G2CP 核心思想与四大贡献
- 创建了 [[raw/lessons/G2CP/02-related-work|02-related-work]] 相关工作翻译，涵盖 ACL 历史（KQML/FIPA/承诺语义）、语义异质性、多智能体 LLM 系统、知识图谱与智能体系统
- 创建了 [[raw/lessons/G2CP/03-protocol|03-protocol]] 协议定义翻译，包含形式化定义（知识图/消息元组/图操作）、操作语义（TRAVERSE/UPDATE 递归定义）、三大属性（确定性/可审计性/完备性）、消息语法、与 FIPA-ACL 关系、七种承诺语义
- 创建了 [[raw/lessons/G2CP/04-architecture|04-architecture]] 多智能体架构翻译，包含四种智能体角色与边类型专业化、协调协议三阶段、HC-3 完整工作示例（189 token vs FTMA 1400+）、G2CP 运行时引擎（Algorithm 1）、LLM 动态操作选择（Algorithm 2）
- 创建了 [[raw/lessons/G2CP/05-experiments|05-experiments]] 实验评估翻译，包含实验设置、总体性能对比（表4）、按类别分析、消融实验（表5）、复杂诊断案例研究、真实工业验证（21 案例）、可扩展性分析
- 更新了 [[wiki/index.md]]：学习笔记统计 133 → 139，新增"G2CP 论文精读系列（6 篇）"入口

## [2026-05-16] learn | 新增 DebUnc 论文精读系列（5 篇）

- 创建了 [[raw/lessons/DebUnc/00-essence|00-essence]] 精华总结，提炼论文核心贡献、三大不确定性度量、两种通信方法、Oracle 分析结果与局限
- 创建了 [[raw/lessons/DebUnc/01-introduction|01-introduction]] 引言翻译，涵盖 LLM 幻觉风险、多智能体辩论的提出与局限、DebUnc 的动机与四大贡献
- 创建了 [[raw/lessons/DebUnc/02-related-work|02-related-work]] 相关工作翻译，涵盖三类不确定性度量方法（token 概率/LLM 生成/采样）、多智能体辩论研究、ReConcile 与 DebUnc 的区别
- 创建了 [[raw/lessons/DebUnc/03-method|03-method]] 方法翻译，包含 Mean Token Entropy/TokenSAR/Oracle 的数学定义、置信度转换公式、Confidence in Prompt 和 Attention Scaling 的详细实现
- 创建了 [[raw/lessons/DebUnc/04-experiments|04-experiments]] 实验翻译，包含 Mistral-7B/Llama-3-8B 完整结果表格、Oracle 分析、AUROC 相关性分析与关键发现
- 更新了 [[wiki/index.md]]：学习笔记统计 128 → 133，新增 "DebUnc 论文精读系列（5 篇）" 入口

---

## [2026-05-08] learn | 新增 [[raw/lessons/Research-to-TopConf/00-essence|Research-to-TopConf]] 前 6 篇研究路线图基础课

- 新建了系列总览 [[raw/lessons/Research-to-TopConf/00-essence|00-essence]]，把“进入新领域 → 综述阅读 → 论文精读 → idea 生成 → 方向筛选”压缩成一条面向顶会论文推进的研究路线图，并将主线明确锚定到“通过协议重构缓解多智能体通信幻觉”
- 新建了 5 篇基础 lesson，形成第一批完整前半段闭环：
  - [[raw/lessons/Research-to-TopConf/01-entering-new-field|01-entering-new-field]] — 从零进入陌生方向的方法论、关键词树、人物/会议/benchmark/framework 五类入口
  - [[raw/lessons/Research-to-TopConf/02-survey-reading-strategy|02-survey-reading-strategy]] — 为什么先读综述，以及如何从综述中提取问题地图、方法地图、评测地图与研究空白
  - [[raw/lessons/Research-to-TopConf/03-reading-papers-critically|03-reading-papers-critically]] — 如何批判性阅读论文，区分 prompt / role / workflow / protocol / benchmark 五层改动
  - [[raw/lessons/Research-to-TopConf/04-idea-generation|04-idea-generation]] — 如何从异常、矛盾、局限、迁移、组合中生成 research idea
  - [[raw/lessons/Research-to-TopConf/05-direction-selection|05-direction-selection]] — 如何用创新性、可行性、评测可得性、资源成本、叙事潜力五维框架筛选方向并收敛到可执行切口
- 统一采用 `lesson` frontmatter，并为 00-05 六篇配置了连续的 `series.part`（0→5）、前置知识、来源、相关页面与“下一步学习”导航
- 校验了系列导航一致性：00→01→02→03→04→05 连续成立；第 05 篇末尾采用阶段小结而非硬链到未创建的第 06 篇，避免断链
- 校验了正文体量：00-05 六篇正文字符数分别约为 9596 / 10162 / 10770 / 9068 / 8686 / 10826，均满足首批厚笔记要求
- 更新了 [[wiki/index.md]]：学习笔记统计 106 → 112，并新增 “Research-to-TopConf 系列（6 篇）” 入口

## [2026-05-08] learn | 完成第 3 篇多智能体综述“协作机制”的学习友好版中文精读系列

- 新建了 [[raw/lessons/Multi-Agent-Survey/03-collaboration-mechanisms/00-collaboration-mechanisms-essence|第 3 篇-00 论文精华总览]]，用于提炼论文的核心贡献、关键机制、重要结论与结构化阅读入口
- 基于 [[raw/paper/multi-agent/03-multi-agent-collaboration-mechanisms-a-survey-of-llms-2025.pdf]]，完成了第 3 篇综述论文的分篇全文中文翻译笔记：
  - [[raw/lessons/Multi-Agent-Survey/03-collaboration-mechanisms/01-collaboration-mechanisms-introduction-background|01 摘要、导论与背景]]
  - [[raw/lessons/Multi-Agent-Survey/03-collaboration-mechanisms/02-collaboration-mechanisms-concept|02 协作机制概念与形式化定义]]
  - [[raw/lessons/Multi-Agent-Survey/03-collaboration-mechanisms/03-collaboration-mechanisms-types|03 协作类型：合作、竞争与竞合]]
  - [[raw/lessons/Multi-Agent-Survey/03-collaboration-mechanisms/04-collaboration-mechanisms-strategies|04 协作策略：规则、角色与模型驱动]]
  - [[raw/lessons/Multi-Agent-Survey/03-collaboration-mechanisms/05-collaboration-mechanisms-structures|05 通信结构：中心化、去中心化与层级化]]
  - [[raw/lessons/Multi-Agent-Survey/03-collaboration-mechanisms/06-collaboration-mechanisms-coordination-and-lessons|06 协调编排与经验总结]]
  - [[raw/lessons/Multi-Agent-Survey/03-collaboration-mechanisms/07-collaboration-mechanisms-applications|07 应用版图]]
  - [[raw/lessons/Multi-Agent-Survey/03-collaboration-mechanisms/08-collaboration-mechanisms-open-problems-and-conclusion|08 开放问题与结论]]
  - [[raw/lessons/Multi-Agent-Survey/03-collaboration-mechanisms/09-collaboration-mechanisms-references|09 参考文献索引]]
- 按原论文结构完整覆盖了摘要、导论、背景、协作机制形式化定义、协作类型、协作策略、通信结构、协调编排、应用、开放问题、结论与参考文献索引
- 在分篇笔记中统一加入了术语对照、思考题、译者注，并将参考文献页改写为“主题索引式保留”而非机械平铺
- 导航链 00→01→02→03→04→05→06→07→08→09 已完成串联，形成从论文精华到分章精读的完整学习路线
- 在 [[wiki/index.md]] 中补入“第 3 篇已完成”入口，学习笔记统计 96 → 106
- 已更新 [[CLAUDE.md]] 的论文翻译笔记规范，正式加入每篇论文必须包含 `00-essence.md` 精华汇总笔记的要求

---

## [2026-05-07] learn | 完成第 2 篇多智能体综述的学习友好版全文中文精读系列

- 新建了 [[raw/lessons/Multi-Agent-Survey/02-recent-advances/10-recent-advances-and-new-frontiers-overview|10 总览]]，整理第 2 篇定位、与第 1 篇的差异、术语增量与分篇导航
- 基于 [[raw/paper/multi-agent/02-a-survey-on-llm-based-multi-agent-system-recent-advances-and-new-frontiers-in-application-2024.pdf]]，完成了第 2 篇综述论文的分篇全文中文翻译笔记：
  - [[raw/lessons/Multi-Agent-Survey/02-recent-advances/11-recent-advances-and-new-frontiers-introduction|11 摘要、导论与应用框架]]
  - [[raw/lessons/Multi-Agent-Survey/02-recent-advances/12-recent-advances-and-new-frontiers-core-components|12 核心组件：生成式智能体与环境]]
  - [[raw/lessons/Multi-Agent-Survey/02-recent-advances/13-recent-advances-and-new-frontiers-solving-complex-tasks|13 复杂任务求解：推理框架、通信优化、资源与评测]]
  - [[raw/lessons/Multi-Agent-Survey/02-recent-advances/14-recent-advances-and-new-frontiers-simulating-specific-scenarios|14 特定场景模拟：社会、物理、游戏]]
  - [[raw/lessons/Multi-Agent-Survey/02-recent-advances/15-recent-advances-and-new-frontiers-evaluating-generative-agents|15 评估生成式智能体：评价与训练]]
  - [[raw/lessons/Multi-Agent-Survey/02-recent-advances/16-recent-advances-and-new-frontiers-challenges-future-and-conclusion|16 挑战、未来方向、结论与局限]]
  - [[raw/lessons/Multi-Agent-Survey/02-recent-advances/17-recent-advances-and-new-frontiers-references|17 参考文献全文保留]]
- 按原论文结构完整覆盖了摘要、导论、核心组件、复杂任务求解、特定场景模拟、评估生成式智能体、挑战与未来方向、结论以及参考文献
- 在分篇笔记中补入了统一的术语对照、表格重构（表 1-3）、思考题与译者注
- 严格使用"译者注（非原文）/ 理解提示（非原文）/ 易混点（非原文）"区分原文与补充内容
- 导航链 10→11→12→13→14→15→16→17 已通过一致性校验
- 在 [[wiki/index.md]] 中补入"第 2 篇已完成"入口，学习笔记统计 88 → 96

---

## [2026-04-29] learn | 新增 [[raw/lessons/Agent-Engineering/08-project-understanding-and-repo-exploration|Harness 第 8 章：项目理解与仓库探索——为什么 agent 不能一上来就盲改代码]]

- 新建了 [[raw/lessons/Agent-Engineering/08-project-understanding-and-repo-exploration|第 8 章：项目理解与仓库探索——为什么 agent 不能一上来就盲改代码]]，把项目理解正式拆成目录结构扫描、模式发现、局部精读三层递进策略
- 在正文中补入了“为什么不能一上来就全仓库乱读”“探索结果如何压缩成项目上下文对象”“探索不是准备动作，而是主回路中的工程化阶段”三组核心判断
- 更新了 [[raw/lessons/Agent-Engineering/07-task-decomposition-and-multi-step-strategy]] 的“下一步学习”，从旧的占位回退文案改为正式跳转到第 8 章
- 更新了 [[raw/lessons/full-plan|full-plan]]：将 Harness 第 8 章加入已完成列表，并把当前推荐顺序推进到 LangChain 07 → Harness 09 的下一波
- 更新了 [[wiki/index.md]]：学习笔记统计 80 → 81；Agent Engineering 与 Claude Code-like Harness 系列 7 篇 → 8 篇

## [2026-04-29] learn | 新增 [[raw/lessons/LangChain/06-memory-state-history-boundaries|LangChain 第 6 章：Memory / State / History——哪些属于组件层，哪些属于系统层]]

- 新建了 [[raw/lessons/LangChain/06-memory-state-history-boundaries|第 6 章：Memory / State / History——哪些属于组件层，哪些属于系统层]]，把 messages history、conversation-scoped state 与 cross-conversation store 三层边界正式拆开
- 在正文中补入了 `AgentState`、`ToolRuntime`、`runtime.state`、`runtime.store` 的最小示例，说明 short-term memory 更接近 state，long-term memory 更接近 store
- 将本章与 [[raw/lessons/Agent-Engineering/06-session-memory-checkpoint-resume]] 形成分工：前者讲 LangChain 组件层的标准表达方式，后者讲 harness 系统层的状态骨架
- 更新了 [[raw/lessons/LangChain/05-prompt-and-context-organization]] 的“下一步学习”，从旧的占位回退文案改为正式跳转到第 6 章
- 更新了 [[raw/lessons/full-plan|full-plan]]：将 LangChain 第 6 章加入已完成列表，并把当前推荐顺序推进到 Harness 08 → LangChain 07 的下一波
- 更新了 [[wiki/index.md]]：学习笔记统计 79 → 80；LangChain 1.x 系列 5 篇 → 6 篇

## [2026-04-29] learn | 新增 [[raw/lessons/Agent-Engineering/07-task-decomposition-and-multi-step-strategy|Harness 第 7 章：任务分解与多步执行策略——复杂任务为什么不能只靠临场乱试]]

- 新建了 [[raw/lessons/Agent-Engineering/07-task-decomposition-and-multi-step-strategy|第 7 章：任务分解与多步执行策略——复杂任务为什么不能只靠临场乱试]]，把复杂任务为什么必须先判断任务形状、再做顺序 / 分层 / 条件分支拆解讲清楚
- 在正文中补入了“任务分解不等于 todo 列表”“局部上下文打包”“retry / 局部补救 / replan”三组关键方法，并把它们落到 planner、executor、checkpoint、reviewer 的协同上
- 更新了 [[raw/lessons/Agent-Engineering/06-session-memory-checkpoint-resume]] 的“下一步学习”，从临时回退到 full-plan 的占位文案改为正式跳转到第 7 章
- 更新了 [[raw/lessons/full-plan|full-plan]]：将 Harness 第 7 章加入已完成列表，并刷新当前推荐顺序到 LangChain 06 → Harness 08 的后续 wave
- 更新了 [[wiki/index.md]]：学习笔记统计 78 → 79；Agent Engineering 与 Claude Code-like Harness 系列 6 篇 → 7 篇

## [2026-04-29] learn | 新增 [[raw/lessons/LangChain/05-prompt-and-context-organization|LangChain 第 5 章：Prompt 与上下文组织——什么应放在哪一层]]

- 新建了 [[raw/lessons/LangChain/05-prompt-and-context-organization|第 5 章：Prompt 与上下文组织——什么应放在哪一层]]，把 prompt engineering 从“写一句更好的 system prompt”提升为“组织模型可见信息结构”的系统问题
- 在正文中区分了 messages、runtime context、tool results、history 的边界，明确说明 runtime context 默认不会自动进入模型 prompt
- 使用 `ChatPromptTemplate` 与 `MessagesPlaceholder` 的最小示例，讲清 system / task / history / tool results 该如何分层组织
- 补入了截断、摘要、选择性注入三种上下文控制策略，并将其映射到 harness 的 planner、executor、reviewer 三个落点
- 更新了 [[raw/lessons/LangChain/04-tools-and-structured-output]] 的“下一步学习”，改成组件层继续深入与回到 Harness 主线两条可选路线
- 更新了 [[raw/lessons/full-plan|full-plan]]：将 LangChain 第 5 章加入已完成列表，并刷新当前推荐的下一步顺序
- 更新了 [[wiki/index.md]]：学习笔记统计 77 → 78；LangChain 1.x 系列 4 篇 → 5 篇

## [2026-04-29] update | 梳理并刷新 [[raw/lessons/full-plan|full-plan]] 与三条系列的当前真实进度

- 完整复核了 [[raw/lessons/LangGraph-to-Harness/01-from-graph-to-runtime]]、[[raw/lessons/LangGraph-to-Harness/02-from-primitives-to-system-modules]]、[[raw/lessons/LangChain/01-why-langchain-1-x]] 到 [[raw/lessons/LangChain/04-tools-and-structured-output]]、[[raw/lessons/Agent-Engineering/01-claude-code-like-systems-overview]] 到 [[raw/lessons/Agent-Engineering/06-session-memory-checkpoint-resume]] 的现有内容与系列分工
- 更新了 [[raw/lessons/full-plan|full-plan]] 的 `sources`，把三条系列目前已经落地的章节全部纳入来源列表，避免总规划与真实课程脱节
- 刷新了 [[raw/lessons/full-plan|full-plan]] 中 Phase 0-4 的完成状态、推荐顺序与阶段输出勾选情况，使其与当前磁盘上的已完成章节一致
- 将 [[raw/lessons/full-plan|full-plan]] 的“最近 6 篇写作顺序”“当前最推荐的下一步顺序”“相关页面”“下一步学习”统一改写为面向下一波 Phase 4 深化，而不再停留在已完成的旧 wave
- 更新了 [[wiki/index.md]]：将 Agent Engineering 与 Claude Code-like Harness 系列从 3 篇修正为 6 篇，并补入第 4-6 章索引
- 核对了 Harness 第 4-6 章的 `series.part` 与“下一步学习”导航链：04 → 05 → 06 → 07 连续成立

## [2026-04-29] learn | 完成 LangGraph → LangChain 1.x → Claude Code-like Harness 当前 wave 的 6 篇核心推进

- 新建了 [[raw/lessons/LangGraph-to-Harness/02-from-primitives-to-system-modules|第 2 章：从原语到系统模块——LangGraph 能力如何落到真实 Agent 架构]]，将 LangGraph 原语稳定映射到 session、planner、executor、permission gate、review 等真实系统模块
- 新建了 [[raw/lessons/LangChain/02-langchain-1-x-core-abstractions|第 2 章：LangChain 1.x 的分层结构与核心抽象]]，系统梳理 model、messages、tools、structured output、agent abstraction 五类核心抽象，并明确它们在 Harness 中的落点
- 新建了 [[raw/lessons/Agent-Engineering/02-harness-mvp-architecture|第 2 章：Harness MVP 架构——一个最小 coding agent 系统怎么拆]]，把 coding agent 的最小骨架稳定拆成交互、session、planner、executor、tools、permission、review、logging、config 等层
- 新建了 [[raw/lessons/LangChain/03-models-messages-and-provider-boundaries|第 3 章：Models、Messages 与 Provider Boundaries]]，明确模型接口层、消息协议层与 provider 差异边界之间的分工
- 新建了 [[raw/lessons/Agent-Engineering/03-planner-executor-reviewer-loop|第 3 章：Planner / Executor / Reviewer——受控执行回路是怎样形成的]]，将最小受控执行闭环正式抽出为主线架构主题
- 新建了 [[raw/lessons/LangChain/04-tools-and-structured-output|第 4 章：Tools 与 Structured Output——让结果可调用、可验证、可组合]]，把工具调用协议与结构化输出对象化思维正式接到 Harness 回路上
- 同步更新了 [[raw/lessons/full-plan|full-plan]]：将当前 wave 的 6 篇从占位路径升级为已完成里程碑，并把“当前最推荐的下一步”改为下一阶段方向选择
- 修正了 [[raw/lessons/LangChain/04-tools-and-structured-output]] 中“下一步学习”区块的一处排版错误，避免导航噪音
- 更新了 [[wiki/index.md]]：学习笔记统计 68 → 74；桥接系列 1 → 2，LangChain 系列 1 → 4，Harness 系列 1 → 3

## [2026-04-29] update | 修订 [[raw/lessons/full-plan|full-plan]] 的导航与规则细节

- 将 full-plan frontmatter 补充 `role: control-document`，区分其“总控规划文档”角色，避免与普通 lesson 正文完全混同
- 曾将规划中的 6 个后续章节路径临时改为纯路径占位，以避免在文件尚未创建前制造系列断链；后续在路线图刷新时重新恢复为明确的未来 wikilink，作为下一波写作导航入口
- 在 Python 正文内联讲解规则中新增“重复衰减规则”：首次讲透、第二次回顾、第三次及以后按需简述，平衡可读性与篇幅控制
- 将 full-plan 末尾的“下一步学习”区块从旧的占位路径改写为明确的下一阶段推荐阅读顺序，用于指向 Phase 4 的未来写作目标

## [2026-04-29] create | 新增全局总规划 [[raw/lessons/full-plan|full-plan]]

- 创建了 [[raw/lessons/full-plan|LangGraph → LangChain 1.x → Claude Code-like Harness 学习与写作总规划]]，作为三条系列课的单一权威路线图
- 在规划中明确了三条线各自职责：桥接系列负责认知转场，LangChain 1.x 负责标准组件层，Agent Engineering 负责系统装配层
- 在规划中定义了三线联动方式：同步里程碑、phase 写作顺序、最近 6 篇推荐顺序与系列导航规则
- 将“只讲 LangChain 1.x”“Python 语法正文内联讲解”“按 MVP → intermediate → advanced 演进”三条规则固化为后续硬约束
- 更新了 [[wiki/index.md]]：学习笔记统计 67 → 68，并新增“总规划”入口

## [2026-04-29] learn | 新增 LangGraph → LangChain 1.x → Claude Code-like Harness 三篇桥接总纲

- 新建了 [[raw/lessons/LangGraph-to-Harness/01-from-graph-to-runtime|第 1 章：从 Graph 到 Runtime——为什么下一步是 LangChain 1.x 与 Harness]]，将现有 LangGraph 学习主线桥接到 agent runtime / harness 视角
- 新建了 [[raw/lessons/LangChain/01-why-langchain-1-x|第 1 章：为什么 LangChain 1.x 是下一步]]，明确后续课程必须以 **LangChain 1.x** 为唯一主线，不混入旧版历史包袱
- 新建了 [[raw/lessons/Agent-Engineering/01-claude-code-like-systems-overview|第 1 章：Claude Code-like 系统总览]]，从公开资料拆解 coding agent 的交互层、session 层、planning 层、tool 层、permission 层、review / verify 层与 logging / trace 层
- 三篇新笔记统一遵循“Python 语法正文内嵌补课、允许重复讲解”的写法，降低来回切换语法上下文的成本
- 更新了 [[wiki/index.md]]：学习笔记统计 64 → 67，并新增 3 个新系列入口（LangGraph 到 Harness 桥接、LangChain 1.x 系统学习、Agent Engineering 与 Claude Code-like Harness）

## [2026-04-28] update | LangGraph 系列切换到 LangChain 1.x 与最新稳定版主线

- 审查并刷新了 [[raw/lessons/LangGraph/01-langgraph-overview]]、[[raw/lessons/LangGraph/04-tools-and-react]]、[[raw/lessons/LangGraph/10-deployment-and-beyond]] 的版本与模型口径
- 将第 1 章中的依赖建议从旧的 `0.x` 版本带改为：`langchain` 固定使用 `1.x`，`langgraph`、`langchain-deepseek`、`langgraph-cli[inmem]` 采用最新稳定版，并补充 `uv lock` 的建议
- 将课程默认 DeepSeek 示例模型名统一调整为更贴近当前 LangChain 集成文档的 `deepseek-chat`，同时保留切换到 `deepseek-v4-pro` 等具体官方型号的说明
- 修正了 [[raw/lessons/LangGraph/01-langgraph-overview]] 中的检查点能力表述，使 `MemorySaver` 与第 5 章、第 10 章及全系列主线保持一致
- 在第 4 章与第 10 章中收口 DeepSeek 选型口径：教学主线默认采用 `langchain-deepseek`，OpenAI / Anthropic 兼容入口仅作为迁移方案说明
- 更新了 [[wiki/index.md]] 的 LangGraph 系列索引说明，使其明确反映 `langchain 1.x`、最新稳定版 `langgraph` 与 DeepSeek 主线

## [2026-04-28] update | LangGraph 系列深度增强：版本建议、Tool Calling 与 DeepSeek 接法对比

- 在 [[raw/lessons/LangGraph/01-langgraph-overview]] 中新增项目实战依赖版本建议、依赖角色拆解与常见环境坑说明
- 在 [[raw/lessons/LangGraph/01-langgraph-overview]] 中补充了更贴近第 4 章实战的 DeepSeek 适配验证命令
- 在 [[raw/lessons/LangGraph/04-tools-and-react]] 中新增“Tool Calling 为什么是 agent 的分水岭”，从能力边界、动作决策、ReAct 回路三个层面加深理解
- 在 [[raw/lessons/LangGraph/04-tools-and-react]] 中补充了 `init_chat_model()`、`bind_tools()`、`ToolNode(...)`、`tools_condition` 的角色、输入、输出与数据流解释
- 在 [[raw/lessons/LangGraph/10-deployment-and-beyond]] 中新增 DeepSeek 三种接法对比：`langchain-deepseek`、OpenAI 兼容 client、Anthropic 兼容 client
- 在 [[raw/lessons/LangGraph/10-deployment-and-beyond]] 中补充了不同接法下的教学推荐与工程选型直觉
- 更新了 [[wiki/index.md]] 的 LangGraph 系列索引说明

## [2026-04-28] update | LangGraph 系列接入 DeepSeek V4 Flash 并修正文档细节

> 注：本条记录的是同日一次中间状态，后续已在上方“LangGraph 系列切换到 LangChain 1.x 与最新稳定版主线”中统一收口；当前课程默认示例模型名以 `deepseek-chat` 为准。

- 审查了 [[raw/lessons/LangGraph/01-langgraph-overview]] 到 [[raw/lessons/LangGraph/10-deployment-and-beyond]] 的整体质量，重点核对深度、广度、可理解性与系列一致性
- 修复了 [[raw/lessons/LangGraph/02-stategraph-basics]] 的前置知识表格渲染问题
- 将 [[raw/lessons/LangGraph/04-tools-and-react]] 中的模型示例统一改为 `deepseek-v4-flash`，并显式写出 `model_provider="deepseek"`
- 在 [[raw/lessons/LangGraph/04-tools-and-react]] 中补充了为什么本章工具调用流程可以继续沿用到 DeepSeek 项目实战的说明
- 将 [[raw/lessons/LangGraph/09-subgraphs-and-composition]] 的 `StateGraph` 导入路径统一为 `from langgraph.graph import ...`
- 将 [[raw/lessons/LangGraph/10-deployment-and-beyond]] 中的 `InMemorySaver` 统一为 [[raw/lessons/LangGraph/05-checkpointer-memory]] 主线使用的 `MemorySaver`
- 将第 1 章与第 10 章中的环境准备、依赖示例与 `langgraph.json` 示例统一调整为 `langchain-deepseek` + `DEEPSEEK_API_KEY`
- 补充了 DeepSeek 官方文档来源到相关章节，并在 [[wiki/index.md]] 的 LangGraph 系列索引处注明项目实战已按 `deepseek-v4-flash` 校对

## [2026-04-28] learn | LangGraph 系统学习系列（完整 10 章收官）

- 将学习系列「LangGraph 系统学习」从首批 2 篇扩展为完整 10 章，路径：`raw/lessons/LangGraph/`
- 重写并加厚了：
  - [[raw/lessons/LangGraph/01-langgraph-overview|第 1 章：LangGraph 学习地图与研究助手项目总览]]
  - [[raw/lessons/LangGraph/02-stategraph-basics|第 2 章：StateGraph、节点与状态——让研究助手先跑起来]]
- 新建并补齐了：
  - [[raw/lessons/LangGraph/03-state-and-messages|第 3 章：State 与 Messages——让研究助手真正进入对话态]]
  - [[raw/lessons/LangGraph/04-tools-and-react|第 4 章：工具与 ReAct——让研究助手学会调用检索工具]]
  - [[raw/lessons/LangGraph/05-checkpointer-memory|第 5 章：记忆与检查点——让研究助手跨轮记住上下文]]
  - [[raw/lessons/LangGraph/06-human-in-the-loop|第 6 章：人在回路——让研究助手在关键步骤停下来等你审核]]
  - [[raw/lessons/LangGraph/07-streaming|第 7 章：流式执行——实时看见研究助手正在如何思考与推进]]
  - [[raw/lessons/LangGraph/08-send-and-parallel|第 8 章：并行与分发——让研究助手同时处理多个检索任务]]
  - [[raw/lessons/LangGraph/09-subgraphs-and-composition|第 9 章：子图与组合——把研究助手拆成可复用模块]]
  - [[raw/lessons/LangGraph/10-deployment-and-beyond|第 10 章：部署与扩展——把研究助手整理成本地可运行应用]]
- 全系列统一采用现代 LangGraph 主线：`StateGraph`、`MessagesState`、`add_messages`、`ToolNode`、`tools_condition`、`MemorySaver` / `SqliteSaver`、`interrupt()`、`Command(resume=...)`、`stream()`、`Send`、子图组合、`langgraph dev`
- 保持贯穿项目主线为 Research Copilot，并使其从 v0 递进成长到 v8
- 新增概念支撑页：
  - [[wiki/concepts/state-graph|StateGraph]]
  - [[wiki/concepts/langgraph-api-map|LangGraph API 学习地图]]
- 依据 [[wiki/concepts/series-navigation-consistency|系列课程导航一致性检查]] 完成系列导航校验：
  - 校验 `series.part` 已连续覆盖 1-10
  - 校验第 1 章路线图与学习路线表和真实文件一致
  - 校验各章 prerequisites、相关页面与“下一步学习”链条
  - 校验第 10 章明确标注系列结束，不再指向不存在的下一章
- 更新了 [[wiki/index.md]]（概念页统计 17 → 19，学习笔记统计 56 → 64，LangGraph 系列 2 篇 → 10 篇）

## [2026-04-22] lint | LLM-Training 系列课程导航链接全面修复

- 发现并修复了「大模型从零训练」系列课程中 **9 处导航不一致**
- 核心问题：01 概览的路线路图（§4.4 和 §5）与实际文件顺序完全不符；每章末尾的"下一步学习"链接中文件名/标题与实际文件不匹配
- 修复内容：
  - [[raw/lessons/LLM-Training/01-llm-training-overview]]：重写路线图和学习路线表，修复"下一步学习"链接
  - [[raw/lessons/LLM-Training/02-data-engineering]]：修复下一章链接（`03-tokenization` → `03-tokenizer-design`）
  - [[raw/lessons/LLM-Training/03-tokenizer-design]]：修复下一章链接（`04-model-architecture` → `04-modern-llm-architecture`）
  - [[raw/lessons/LLM-Training/04-modern-llm-architecture]]：修复下一章链接（`05-training-pipeline` → `05-distributed-training`）
  - [[raw/lessons/LLM-Training/06-training-optimization]]：修复下一章链接（`07-finetuning-and-alignment` → `07-pretraining-practice`）
  - [[raw/lessons/LLM-Training/07-pretraining-practice]]：修复下一章链接（`08-sft-and-alignment` → `08-supervised-finetuning`）
  - [[raw/lessons/LLM-Training/08-supervised-finetuning]]：修复下一章链接（`09-reinforcement-learning-alignment` → `09-rlhf-dpo`）
- 积累错误经验：记录到 [[wiki/concepts/series-navigation-consistency|系列课程导航一致性检查]]

## [2026-04-19] learn | 大模型从零训练系列（全部 10 章）

- 创建了完整学习系列「大模型从零训练」，共 10 篇笔记
- 笔记路径：`raw/lessons/LLM-Training/`
- 系列结构：
  - **模块一：基础与数据工程**
    - [[raw/lessons/LLM-Training/01-llm-training-overview|第 1 章：大模型训练全景]] — Pretrain→SFT→RLHF 全流程、nano-LLM 项目设计
    - [[raw/lessons/LLM-Training/02-data-engineering|第 2 章：数据工程]] — 数据来源、清洗管道、质量过滤、去重、数据配比
    - [[raw/lessons/LLM-Training/03-tokenizer-design|第 3 章：分词器设计]] — BPE/WordPiece/Unigram、SentencePiece 中文分词器
  - **模块二：模型架构与训练**
    - [[raw/lessons/LLM-Training/04-modern-llm-architecture|第 4 章：现代 LLM 架构]] — SwiGLU、RMSNorm、RoPE、GQA、nano-LLM 架构设计
    - [[raw/lessons/LLM-Training/05-distributed-training|第 5 章：分布式训练]] — DP/TP/PP、ZeRO/FSDP/DeepSpeed
    - [[raw/lessons/LLM-Training/06-training-optimization|第 6 章：训练优化技术]] — 混合精度、Flash Attention、梯度累积、学习率调度
    - [[raw/lessons/LLM-Training/07-pretraining-practice|第 7 章：预训练实战]] — nano-LLM 完整训练代码
  - **模块三：对齐与部署**
    - [[raw/lessons/LLM-Training/08-supervised-finetuning|第 8 章：SFT]] — 指令数据、LoRA/QLoRA
    - [[raw/lessons/LLM-Training/09-rlhf-dpo|第 9 章：RLHF 与 DPO]] — 奖励模型、PPO、DPO
    - [[raw/lessons/LLM-Training/10-evaluation-deployment|第 10 章：评估与部署]] — Benchmark、量化、vLLM
- 更新了 [[wiki/index.md]]（学习笔记统计 44 → 54，新增完整大模型从零训练系列 10 篇）

## [2026-04-19] learn | 大模型从零训练系列——第 6 章：训练优化技术

- 创建了学习笔记：[[raw/lessons/LLM-Training/06-training-optimization|第 6 章：训练优化技术]]
- 笔记路径：`raw/lessons/LLM-Training/06-training-optimization.md`
- 系列名：大模型从零训练（第 6 篇，advanced）
- 核心内容：
  - 混合精度训练：FP32/FP16/BF16 对比、三大核心技术（FP32 主权重、损失缩放、FP32 累加）、BF16 vs FP16 选型、PyTorch 代码
  - Flash Attention：标准 Attention 的 O(N²) 显存瓶颈、分块计算原理、Online Softmax 两遍扫描、PyTorch SDPA 使用方式
  - 梯度累积：等效增大 batch size 的原理、代码实现、loss 除以 accumulation_steps 的原因
  - 学习率调度：Cosine with Warmup 数学公式、各模型超参数参考表（GPT-3/LLaMA/Mistral/nano-LLM）
  - 梯度裁剪与训练稳定性：全局范数裁剪、训练不稳定性排查清单
  - nano-LLM 完整训练配置总结表 + 整合训练循环代码
- 更新了 [[wiki/index.md]]（学习笔记统计 44 → 45，新增大模型从零训练系列目录）

---

## [2026-04-18] learn | 科研开源工具全景指南

用户要求搜索并整理 25+ 款科研相关开源/免费工具的最新信息。

**研究方法**：12 路并行搜索（MiniMax web_search），覆盖：
- 学术写作工具：GPT Academic, STORM, Writefull, Grammarly
- 论文阅读工具：ChatPaper, Elicit, SciSpace, ScholarAI, PaperQA2
- 自动化研究：AI-Researcher, GPT Researcher
- 文献发现：Semantic Scholar API, ResearchRabbit, Connected Papers
- 数据分析：PandasAI
- 图表绘制：Draw.io, Detikzify, img2img-turbo
- 文献管理：Zotero 插件（Better BibTeX, Jasminum 等）
- Obsidian 插件：Zotero Integration, Citations, Dataview, Templater
- LaTeX 工具：latexdiff, Overleaf CLI
- 提示词：Awesome ChatGPT Prompts

**新建文件**：
- [[raw/lessons/research-tools/research-open-source-tools-guide]]（beginner，含 23 个工具详细说明 + 工具组合推荐 + 免费基础工具链）

**更新了** [[wiki/index.md]]（学习笔记统计 43 → 44）

---

## [2026-04-18] research | Agent Memory 方向文献调研

- 创建了研究方向文档 [[research/01-topics/agent-memory]]（记忆分类体系、研究脉络、创新空间分析）
- 创建了研究灵感文件 [[research/_inbox/agent-memory-ideas]]（5 个创新想法）
- 创建了 research/02-papers/ 子目录（_surveys, _foundations, long-term-memory, episodic-memory 等）
- 创建了综述论文笔记：[[research/02-papers/_surveys/agent-memory-survey-zhang2024]]、[[research/02-papers/_surveys/lifelong-learning-llm-agent-zheng2026]]
- 创建了经典论文笔记：[[research/02-papers/_foundations/generative-agents-park2023]]、[[research/02-papers/_foundations/memgpt-packer2023]]、[[research/02-papers/_foundations/reflexion-shinn2023]]
- 更新了 [[wiki/index.md]]（新增 Agent Memory 研究方向）

---

## [2026-04-08] create | 初始化 LLM Wiki 系统

- 创建了 Karpathy 风格的三层目录结构（raw/, wiki/, _templates/）
- 编写了 CLAUDE.md Schema 配置文件
- 创建了笔记模板（concept, entity, summary）
- 创建了 wiki/index.md 和 wiki/log.md

## [2026-04-08] ingest | Karpathy Obsidian 工作流

- 处理了 [[raw/Karpathy-Obsidian-LLM-Wiki.md]]
- 创建了概念页：[[wiki/concepts/knowledge-compilation]]、[[wiki/concepts/LLM Wiki]]
- 创建了实体页：[[wiki/entities/Andrej Karpathy]]、[[wiki/entities/Obsidian]]
- 更新了 [[wiki/index.md]]（2 概念 + 2 实体）

## [2026-04-08] update | 新增「学习」工作流（Learn）

- 在 CLAUDE.md Schema 中新增第四大操作流程：Learn（学习）
- 创建了 `wiki/lessons/` 目录，用于存放结构化学习笔记
- 创建了学习笔记模板 [[_templates/lesson.md]]
- 新增 `lesson` 页面类型及专属 Frontmatter 字段（difficulty, prerequisites, topic, status, series）
- 定义了完整的学习流程：主题分析 → 多源搜索 → 内容组织 → 思考题 → 写入 Wiki
- 更新了 [[wiki/index.md]]，新增学习笔记统计和目录

## [2026-04-08] update | 优化学习工作流——内容深度与并行写作

基于用户反馈，对 CLAUDE.md 中 Learn 工作流进行三项关键改进：
- **内容深度规范**：要求公式必须配有逐步拆解、符号说明、数值示例和自然语言解读，禁止直接堆砌公式
- **内容丰富度要求**：每个核心概念至少 2 种理解角度，重要概念必须有具体示例
- **并行写作策略**：大型学习任务拆分为多个子智能体并行写作，每个子智能体独立负责一个章节，避免上下文被撑爆

## [2026-04-09] learn | Transformer 深度学习系列（8 篇重写）

用户对旧版 Transformer 笔记不满意，要求重写为更完整、更深度的系列。

**执行策略**：
- 调用 planner 制定 8 篇章节规划和并行写作策略
- 批次 1-2（第 1-2 篇）串行执行，第 2 篇为系列核心（Self-Attention 完整手推）
- 批次 3（第 3-5 篇）三路并行写作（MHA / 位置编码 / FFN+残差+LN）
- 批次 4（第 6 篇）串行（需要综合前面所有知识）
- 批次 5（第 7-8 篇）两路并行写作（训练推理 / 现代变体）

**新建文件**（路径：`wiki/lessons/Transformer/`）：
- [[01-why-transformer]] — 为什么需要 Transformer（beginner）
- [[02-self-attention]] — Self-Attention 核心引擎（intermediate，含完整手推）
- [[03-multi-head-attention]] — Multi-Head Attention（intermediate）
- [[04-positional-encoding]] — 位置编码（intermediate）
- [[05-ffn-residual-layernorm]] — FFN、残差连接与 LayerNorm（intermediate）
- [[06-encoder-decoder]] — 编码器-解码器架构（intermediate）
- [[07-training-inference]] — 训练与推理（advanced）
- [[08-beyond-original]] — BERT、GPT 与现代变体（advanced）

**与旧笔记的区别**：
- 覆盖范围从 2 篇扩展到 8 篇，涵盖完整 Transformer 架构
- 第 2 篇 Self-Attention 包含**从输入到输出的完整手推**（不断裂）
- 每篇至少 2 个直觉类比、3 道思考题、代码视角
- 增加了训练/推理、现代变体等工程实践内容

**更新了** [[wiki/index.md]]（学习笔记统计 0 → 8）

## [2026-04-09] create | 从课程提炼概念页：残差连接与 Layer Normalization

- 阅读了 [[05-ffn-residual-layernorm|第 5 章课程笔记]]
- 创建了概念页：[[wiki/concepts/residual-connection]]（残差连接核心概念提炼）
- 创建了概念页：[[wiki/concepts/layer-normalization]]（LayerNorm 核心概念提炼）
- 更新了 [[wiki/index.md]]（概念页统计 2 → 4）

## [2026-04-09] create | 从课程提炼概念页：位置编码与前馈网络

- 阅读了 [[04-positional-encoding|第 4 章课程笔记]]，提炼位置编码核心概念
- 阅读了 [[05-ffn-residual-layernorm|第 5 章课程笔记]]，提炼 FFN 核心概念
- 创建了概念页：[[wiki/concepts/positional-encoding]]（正弦编码公式、线性性质、方案对比）
- 创建了概念页：[[wiki/concepts/feed-forward-network]]（FFN 角色、升维-激活-降维、参数占比）
- 更新了 [[wiki/index.md]]（概念页统计 4 → 6）

## [2026-04-09] create | 从课程提炼概念页：自回归生成、解码策略、KV Cache

- 阅读了 [[06-encoder-decoder|第 6 章课程笔记]] 和 [[07-training-inference|第 7 章课程笔记]]
- 创建了概念页：[[wiki/concepts/autoregressive-generation]]（自回归生成范式、Teacher Forcing、曝光偏差）
- 创建了概念页：[[wiki/concepts/beam-search]]（解码策略对比：贪心/Beam Search/Top-k/Top-p/Temperature）
- 创建了概念页：[[wiki/concepts/kv-cache]]（KV Cache 原理、加速机制、内存代价、PagedAttention）
- 更新了 [[wiki/index.md]]（概念页统计 6 → 9）

## [2026-04-09] create | 从课程提炼概念页：Cross-Attention、Masked Attention、Encoder-Decoder

- 阅读了 [[06-encoder-decoder|第 6 章课程笔记]]
- 创建了概念页：[[wiki/concepts/cross-attention]]（跨序列注意力：Q 来自解码器，K/V 来自编码器）
- 创建了概念页：[[wiki/concepts/masked-attention]]（因果遮罩：上三角 -inf 技巧，训练时 vs 推理时差异）
- 创建了概念页：[[wiki/concepts/encoder-decoder]]（编码器-解码器架构：理解与生成的分工、三种注意力位置）
- 补充了 [[wiki/concepts/self-attention]] 的索引条目（文件已存在但 index 中遗漏）
- 更新了 [[wiki/index.md]]（概念页统计 9 → 13）

## [2026-04-09] create | 从课程提炼模型实体页：BERT、GPT、T5

- 阅读了 [[08-beyond-original|第 8 章课程笔记]]
- 创建了实体页：[[wiki/entities/BERT]]（Encoder-Only，MLM + NSP 双向预训练）
- 创建了实体页：[[wiki/entities/GPT]]（Decoder-Only，自回归语言模型，规模法则与 In-Context Learning）
- 创建了实体页：[[wiki/entities/T5]]（Encoder-Decoder，Text-to-Text 统一框架，Span Corruption）
- 更新了 [[wiki/index.md]]（实体页统计 2 → 5）

## [2026-04-09] create | 从课程提炼概念页：Scaling Law 与混合专家模型

- 阅读了 [[08-beyond-original|第 8 章课程笔记]]
- 创建了概念页：[[wiki/concepts/scaling-law]]（规模法则：参数量/数据量/计算量的幂律关系、涌现能力、In-Context Learning）
- 创建了概念页：[[wiki/concepts/mixture-of-experts]]（MoE：Router + Expert 稀疏激活、与 Dense Transformer 对比、负载均衡挑战）
- 更新了 [[wiki/index.md]]（概念页统计 13 → 15）

## [2026-04-09] create | 从课程提炼对比分析页：架构变体与 Dense vs MoE

- 阅读了 [[08-beyond-original|第 8 章课程笔记]]
- 创建了对比页：[[wiki/comparisons/encoder-only-vs-decoder-only-vs-encoder-decoder]]（三种 Transformer 架构变体：设计哲学、核心对比表、优劣势、选型指南）
- 创建了对比页：[[wiki/comparisons/dense-vs-moe]]（Dense vs MoE：激活方式、计算量、推理成本、训练难度、适用场景）
- 更新了 [[wiki/index.md]]（对比页统计 0 → 2）

## [2026-04-09] create | 创建人物实体页：Vaswani、Bahdanau、Jay Alammar

- 阅读了 [[01-why-transformer|第 1 章课程笔记]] 和 [[02-self-attention|第 2 章课程笔记]]
- 创建了实体页：[[wiki/entities/Ashish Vaswani]]（Transformer 论文第一作者，提出"只用 Attention，不用递归"）
- 创建了实体页：[[wiki/entities/Dzmitry Bahdanau]]（注意力机制先驱，2014 年提出 Bahdanau Attention）
- 创建了实体页：[[wiki/entities/Jay Alammar]]（技术教育家，The Illustrated Transformer 等可视化教程作者）
- 三个页面间建立了交叉引用（wikilink）
- 更新了 [[wiki/index.md]]（实体页统计 5 → 8）

## [2026-04-09] create | 创建 Transformer 实体页

- 综合了 [[01-why-transformer|第 1 章]]、[[06-encoder-decoder|第 6 章]]、[[08-beyond-original|第 8 章]] 课程内容
- 创建了实体页：[[wiki/entities/Transformer]]（基本信息表、核心组件、三大革命、架构变体、关键超参数）
- 作为知识库中 Transformer 相关内容的中心入口页面
- 更新了 [[wiki/index.md]]（实体页统计 +1）

## [2026-04-09] create | 从课程提炼对比分析页：RNN vs Transformer、位置编码对比、LayerNorm vs BatchNorm

- 阅读了 [[01-why-transformer|第 1 章课程笔记]]、[[04-positional-encoding|第 4 章课程笔记]]、[[05-ffn-residual-layernorm|第 5 章课程笔记]]
- 创建了对比页：[[wiki/comparisons/rnn-vs-transformer]]（10 维度对比表、发展脉络、优势场景分析）
- 创建了对比页：[[wiki/comparisons/positional-encoding-comparison]]（正弦/可学习/RoPE/ALiBi 四方案对比、选型指南）
- 创建了对比页：[[wiki/comparisons/layernorm-vs-batchnorm]]（8 维度对比表、Transformer 选择 LayerNorm 的原因、Pre-LN vs Post-LN）
- 更新了 [[wiki/index.md]]（对比页统计 2 → 5）

## [2026-04-09] create | 创建三篇综合概述页

- 阅读了 [[01-why-transformer|第 1 章课程笔记]]、[[06-encoder-decoder|第 6 章课程笔记]]、[[07-training-inference|第 7 章课程笔记]]、[[08-beyond-original|第 8 章课程笔记]]
- 创建了概述页：[[wiki/overviews/transformer-architecture-overview]]（Transformer 架构全景：完整数据流、组件关系图、设计哲学、组件索引）
- 创建了概述页：[[wiki/overviews/transformer-training-inference-overview]]（训练与推理概述：训练关键步骤、推理策略、训练 vs 推理差异表）
- 创建了概述页：[[wiki/overviews/modern-llm-landscape]]（现代 LLM 格局：三条技术路线、效率优化、发展脉络 2017-2026、选型指南）
- 三篇概述页之间互相链接，并链接到已有的概念页、实体页和对比页
- 更新了 [[wiki/index.md]]（概述页统计 0 → 3）

## [2026-04-09] update | 知识库结构化完成——最终汇总

从 `raw/lessons/Transformer/` 的 8 篇课程中，系统性提取并生成了完整的结构化知识库。

**最终统计**：
- 概念页 16 个（+14 新建）
- 实体页 9 个（+7 新建）
- 对比页 5 个（+5 新建）
- 概述页 3 个（+3 新建）
- 学习笔记 8 篇（已在 raw/lessons/ 中）

**执行策略**：planner 制定方案 → 4 批次并行子智能体执行 → 主智能体统一更新索引和日志

**知识网络结构**：
- 核心骨架：self-attention → multi-head-attention → encoder-decoder → Transformer 实体
- 横向扩展：positional-encoding、FFN、residual-connection、layer-normalization
- 推理链路：autoregressive-generation → beam-search → kv-cache
- 现代扩展：scaling-law → mixture-of-experts
- 三大模型：BERT、GPT、T5
- 对比分析：RNN vs Transformer、位置编码方案、归一化方法、架构变体、Dense vs MoE
- 全局视角：架构概述、训练推理概述、现代 LLM 格局

所有页面通过 [[wikilink]] 互相链接，形成可导航的知识网络。

## [2026-04-09] learn | 肉鸽塔防挂机游戏开发系列——第 3 章：Roguelike 核心机制

- 创建了学习笔记：[[03-roguelike-core]]（Berlin Interpretation、可控随机性、Run-Based 结构、Meta-Progression、协同效应）
- 笔记路径：`raw/lessons/game/03-roguelike-core.md`
- 系列名：肉鸽塔防挂机游戏开发（第 3 篇，intermediate）
- 涵盖五大核心概念：柏林诠释与现代变通、种子/权重池/保底机制、单局结构设计、跨局成长系统、Synergy 元素标签系统
- 更新了 [[wiki/index.md]]（学习笔记统计 8 → 9）

## [2026-04-09] learn | 肉鸽塔防挂机游戏开发系列（14 篇完整系列）

用户想学习做独立的手机小游戏（肉鸽 + 塔防 + 挂机类型），目标平台为微信小游戏。

**执行策略**：
- 调用 planner 制定 14 篇章节规划和并行写作策略
- 用户确认：有编程经验、目标微信小游戏、参考《随机地牢》风格
- Phase 2A（基础层 3 篇）并行写作：01-03
- Phase 2B（机制层 5 篇）并行写作：04-08
- Phase 3（实战层 4 篇）并行写作：09-12
- Phase 4（发布层 2 篇）并行写作：13-14
- 补全笔记 10 和 14 的后半部分

**新建文件**（路径：`raw/lessons/game/`）：
- [[01-game-dev-fundamentals]] — 游戏开发基础（beginner）
- [[02-game-engine-selection]] — 游戏引擎选型（beginner）
- [[03-roguelike-core]] — Roguelike 核心机制（intermediate）
- [[04-procedural-map-generation]] — 程序化地图生成（intermediate）
- [[05-tower-defense-core]] — 塔防核心机制设计（intermediate）
- [[06-idle-system-design]] — 挂机系统设计（intermediate）
- [[07-numerical-balance]] — 数值平衡设计（intermediate）
- [[08-combat-system]] — 战斗系统设计（intermediate）
- [[09-cocos-creator-practice]] — Cocos Creator 实战开发（intermediate）
- [[10-ui-interaction-design]] — UI 与交互设计（beginner）
- [[11-save-data-management]] — 存档与数据管理（intermediate）
- [[12-performance-optimization]] — 性能优化与调试（advanced）
- [[13-publishing-and-operations]] — 游戏发布与运营（intermediate）
- [[14-full-project-walkthrough]] — 完整项目实战（advanced）

**更新了** [[wiki/index.md]]（学习笔记统计 9 → 22）

## [2026-04-10] learn | 千万字男频神话灵异小说创作系列（18 篇完整系列）

用户想学习在番茄小说平台写千万字级别的男频神话灵异类长篇小说。
参考作品：《我有一座恐怖屋》《斩神》《神秘复苏》等。

**执行策略**：
- 调用 planner 制定 18 篇章节规划和 4 阶段并行写作策略
- Phase 0：多源搜索（MiniMax + WebSearch 并行搜索番茄规则、黄金三章、作品分析、伏笔技法、群像写法、神话体系等）
- Phase 1（4 路并行）：模块一（01-03 商业基础）+ 模块六（16-18 作品拆解）
- Phase 2（4 路并行）：模块二（04-06 世界观设定）+ 模块五（13-15 叙事技法）
- Phase 3（2 路并行）：模块三（07-09 人物塑造）+ 模块四（10-12 大纲结构）

**新建文件**（路径：`raw/lessons/novel-writing/`）：

#### 模块一：网文商业基础
- [[01-web-novel-fundamentals|第 1 章：网文写作全景]] — 行业概况、心态建设、常见误区（beginner）
- [[02-fanqie-platform-guide|第 2 章：番茄小说平台攻略]] — 签约规则、收入构成、等级体系、推荐算法（beginner）
- [[03-golden-three-chapters|第 3 章：黄金三章]] — 开篇写作公式、案例分析、灵异类特殊技巧（beginner）

#### 模块二：世界观设定
- [[04-worldbuilding-mythology|第 4 章：神话体系搭建]] — 中国神话素材、原创体系构建、四层世界观（intermediate）
- [[05-worldbuilding-supernatural|第 5 章：灵异规则与力量体系]] — 规则设计、等级体系、金手指、灵异事件模板（intermediate）
- [[06-worldbuilding-map-faction|第 6 章：地图架构与势力设计]] — 多层级地图、五大势力、势力平衡（intermediate）

#### 模块三：人物塑造
- [[07-protagonist-design|第 7 章：主角设计与成长弧光]] — 三种主角类型、弧光节奏、金手指结合（intermediate）
- [[08-ensemble-cast-design|第 8 章：群像塑造技法]] — 四级角色金字塔、视角切换、配角出彩（intermediate）
- [[09-antagonist-npc-design|第 9 章：反派与 NPC 立体化]] — 四级反派、NPC 快速立体化、关系网（intermediate）

#### 模块四：大纲与结构
- [[10-outline-mega-structure|第 10 章：千万字大纲架构]] — 三层大纲体系、五阶段规划（intermediate）
- [[11-multi-thread-narrative|第 11 章：多线叙事设计]] — 五种线型、四种交织技法、线头管理（intermediate）
- [[12-pacing-rhythm|第 12 章：节奏控制]] — 三级节奏、灵异特殊节奏、卷划分策略（intermediate）

#### 模块五：叙事技法
- [[13-foreshadowing-techniques|第 13 章：伏笔埋设与回收]] — 六种高级技法、回收技巧、伏笔管理（intermediate）
- [[14-climax-emotional-design|第 14 章：燃点与刀子设计]] — 五类燃点、五类刀子、燃刀节奏（intermediate）
- [[15-suspense-hook-design|第 15 章：悬念与断章]] — 三层悬念、五种断章法、灵异特殊技巧（intermediate）

#### 模块六：参考作品拆解
- [[16-analysis-horror-house|第 16 章：《恐怖屋》拆解]] — 经营+恐怖双线、氛围营造（intermediate）
- [[17-analysis-slash-god|第 17 章：《斩神》拆解]] — 多神话世界观、群像、燃刀设计（intermediate）
- [[18-analysis-mysterious-revival|第 18 章：《神秘复苏》拆解]] — 规则怪谈、拼图体系、伏笔误导（intermediate）

**更新了** [[wiki/index.md]]（学习笔记统计 22 → 40）

## [2026-04-10] learn | 新增 AI 辅助游戏开发教程

- 创建了 [[raw/lessons/game/15-ai-assisted-game-dev|第 15 章：AI 辅助微信小游戏开发实战]]
- 内容涵盖：Claude Code 环境搭建、cocos-mcp-server 配置、CLAUDE.md 最佳实践、5 个实战场景、效率对比
- 更新了 [[wiki/index.md]]（学习笔记 40 → 41）

## [2026-04-11] learn | 微信小游戏爆火机制深度分析

用户要求深入分析近几年爆火的微信小游戏（抓大鹅、羊了个羊、向僵尸开炮等）的爆火原因。

**研究方法**：多维度并行搜索（MiniMax web_search × 6 轮），覆盖以下方向：
- 羊了个羊/抓大鹅爆火原因分析
- 向僵尸开炮商业模式拆解
- 微信小游戏行业数据与用户画像
- 社交裂变机制与变现模式
- 更多爆款案例数据（寻道大千、咸鱼之王）
- 成瘾心理学原理

**新建文件**：
- [[raw/lessons/game/wechat-minigame-viral-analysis]]（intermediate，约5000字）

**核心发现**：
1. 小游戏已是百亿级市场（2024年收入398亿，月活5亿）
2. 50%+用户从未玩过游戏，开辟全新用户群体
3. 两种爆款路径：社交裂变型（短期爆发）vs 买量运营型（长线收入）
4. 六大核心机制：低门槛×情绪驱动×社交裂变×随机性×即时反馈×混合变现
5. 心理学底层：多巴胺回路、沉没成本、"差一点"认知偏差、Hook Model

**更新了** [[wiki/index.md]]（学习笔记 41 → 42）

## [2026-04-11] learn | 微信爆款小游戏爆火机制深度调研 v2

用户要求深入分析微信小游戏爆火原因，写入 game 目录下新文件。

**研究方法**：8路并行搜索（MiniMax web_search），覆盖：
- 羊了个羊/抓大鹅/向僵尸开炮爆火原因
- 寻道大千/咸鱼之王/无尽冬日市场数据
- 小游戏用户画像与行业数据
- Hook上瘾模型与多巴胺机制
- IAA/IAP/混合变现模式
- 2025-2026行业趋势与平台政策

**新建文件**：
- [[raw/lessons/game/minigame-viral-deep-dive]]（intermediate，约7000字）

**与之前 v1 版本的区别**：
- 从3款扩展到6款爆款深度拆解（新增寻道大千、咸鱼之王、无尽冬日）
- 新增两条爆款路径的系统性对比矩阵
- 新增完整的 Hook 上瘾模型分析
- 新增认知偏差利用的6种方式
- 新增买量经济模型（ROI/LTV/CAC）
- 新增2026年最新平台激励政策对比
- 新增路径选择决策树

**更新了** [[wiki/index.md]]（学习笔记 42 → 43）

## [2026-04-13] research | 初始化学术论文工作流

用户目标：在本科阶段发表顶会论文。需要从零开始建立研究方向选择、文献综述、实验管理、论文写作的完整工作流。

**调研内容**：
- AI辅助学术论文写作工作流（选题→投稿全流程）
- 2025-2026 CV/AI热门研究方向分析（CVPR/ICCV/NeurIPS）
- 适合本科生低算力条件的研究方向推荐
- 免费GPU资源与开源工具箱策略

**核心发现**：
1. 首选方向：**模型压缩与高效部署**（算力需求低、创新空间大、工业界需求高）
2. 次选方向：图像分割/检测改进、视频理解轻量化
3. 慎入方向：多模态大模型、文生图（算力门槛极高）
4. 关键成功因素：选题比努力更重要，选对方向事半功倍

**工作流设计**：
- 新增 `research/` 目录结构（5个子目录 + inbox）
- 新增 5 个研究专用模板（paper-note, experiment-log, research-idea, research-topic, paper-writing）
- 在 CLAUDE.md 新增第五大工作流：**Research（研究）**
- 包含顶会论文 AI 辅助提示词模板（选题/文献综述/方法设计/论文写作）

**创建的文件**：
- [[_templates/paper-note]] — 论文阅读笔记模板
- [[_templates/experiment-log]] — 实验日志模板
- [[_templates/research-idea]] — 研究想法记录模板
- [[_templates/research-topic]] — 研究方向规划模板
- [[_templates/paper-writing]] — 论文写作模板
- `research/01-topics/` — 研究方向目录
- `research/02-papers/` — 论文笔记目录
- `research/03-experiments/` — 实验日志目录
- `research/04-writing/` — 论文草稿目录
- `research/05-submission/` — 投稿目录
- `research/_inbox/` — 灵感暂存目录

**更新了** [[wiki/index.md]]（新增研究工作区章节）

## [2026-04-13] research | 开启模型压缩与高效部署方向

用户选定研究方向：**模型压缩与高效部署（Model Compression & Efficient Deployment）**

**核心论文笔记创建**（8篇，覆盖经典+最新工作）：
- [[02-papers/mobilevit|MobileViT]]（ICLR 2022）— CNN+ViT混合轻量化，入门必读
- [[02-papers/edgevit|EdgeViT]]（ECCV 2022）— LGL瓶颈，稀疏全局注意力
- [[02-papers/nextvit|Next-ViT]]（ICLR 2022）— 工业部署优化，分解注意力
- [[02-papers/efficientvit|EfficientViT]]（ICCV 2023）— 内存效率洞察，Cascaded Attention
- [[02-papers/dynamicvit|DynamicViT]]（NeurIPS 2021）— 动态Token稀疏化
- [[02-papers/aiqvit|AIQViT]]（AAAI 2025）— 后训练量化最新进展
- [[02-papers/partial-attention|Partial Attention]]（ECCV 2024）— 稀疏注意力
- [[02-papers/efficientvim|EfficientViM]]（CVPR 2025）— Vision Mamba最新方向

**研究方向文档**：
- [[01-topics/model-compression-efficient-deployment]] — 方向总览，包含技术路线分析、创新空间、潜在切入点
- [[01-topics/reading-plan]] — 4周阅读计划，精读顺序和每篇目标

**灵感记录**：
- [[_inbox/first-reading-ideas]] — MobileViT知识蒸馏初步想法（阅读时产生）

**技术路线总结**：
1. 架构设计（MobileViT/EdgeViT/Next-ViT/EfficientViT）
2. 动态稀疏化（DynamicViT/Partial Attention）
3. 量化（AIQViT）
4. 知识蒸馏
5. 状态空间模型（EfficientViM/Vision Mamba）

**适合本科生的切入点**：
- 剪枝+蒸馏组合优化
- 面向特定任务的轻量化（分割/检测）
- EfficientViM改进（竞争小，前沿）
- 低比特量化+架构搜索联合设计

## [2026-04-15] research | Agent 蜂群动态协作拓扑——调研与创新架构设计

用户提出"做一个 Agent 蜂群项目能否发论文"的想法。经过全面调研和严格审查，完成了创新架构设计。

**调研范围**（10+ 论文/框架）：
- 经典框架：MetaGPT (ICLR 2024), AutoGen (Microsoft), ChatDev
- 协作机制：MachineSoM (社会心理学视角), Multi-Agent Collaboration Survey (arXiv:2501.06322)
- 失败分析：MAST (NeurIPS 2025, 14种失败模式)
- 群体智能+LLM：Model Swarms (Google, 权重层), MASOIE (算法层)
- 安全与拓扑：NetSafe (拓扑安全性), AgentCoord (可视化协作)
- 自组织：SoA (自组织Agent), MDTeamGPT (自进化医疗框架)

**识别的研究空白**：
1. **拓扑是静态的**（核心Gap）— 所有框架使用预定义固定拓扑，无人研究任务驱动的动态适配
2. **群体智能"行为层"空白** — 权重层和算法层已探索，行为层完全空白
3. **失败诊断了但没治疗** — MAST 停留在诊断层面

**提出架构：SwarmTopo v2**（已通过 architect 严格审查）：
- 环境信号层（解决LLM自评校准性问题）
- 软拓扑适配（连续参数空间，解决离散跳跃问题）
- 改进版状态机（补充缺失路径）
- Coordinator轮值制（解决冷启动和单点故障）

**审查反馈整合**：
- ⚠️ 致命风险：创新性可能被占先（需第1-2周严格文献搜索确认）
- ⚠️ 关键改进：新增消融A4（vs简单规则），叙事重心从"群体智能"转向"动态拓扑适配"
- ✅ 可行性：API成本$30-50可控，3-4个月时间可行

**新建文件**：
- [[01-topics/agent-swarm-dynamic-topology]] — 完整研究方案（文献调研+Gap分析+架构设计+实验方案+路线图）

**更新了** [[wiki/index.md]]（新增方向2：Agent蜂群动态协作拓扑）

## [2026-04-15] research | Agent 蜂群文献搜索 + 创新性确认 + Claude Code 指导文档

用户要求搜索关键论文确认创新性，并编写 Claude Code 指导文档。

**文献搜索完成**（覆盖15+篇论文）：
- 发现最强竞品：**AgentNet** (arXiv:2504.00587, 2025.04) — 已做动态DAG拓扑
- 其他竞品：S-Agents、AgentVerse、MDAgents、CAMEL — 均未覆盖群体智能行为层

**创新性判定**：从"高"调整为"中"
- ❌ 不能声称"首次动态拓扑"（AgentNet 已做）
- ✅ "群体智能行为层"在 LLM Agent 中仍然完全空白（无竞品）
- 论文叙事调整：从"首次动态拓扑"改为"群体智能行为层 + 环境信号驱动软适配"

**项目实施计划**（planner 制定）：
- 框架选型：自建轻量框架（~2000行），不用 AutoGen/LangGraph
- 4阶段时间线：验证(2w) → 框架(3w) → 完整系统(2w) → 实验(3w) → 写作(4w)
- API成本优化后 $35-45（在$50预算内）
- 第一周每天具体行动计划

**创建的文件**：
- [[01-topics/CLAUDE-SwarmTopo]] — Claude Code 项目指导文档
  - 包含：项目概述、架构设计、代码规范、实验设计、任务清单、技术细节
  - 用于在项目代码目录中启动 Claude Code 时自动读取

**更新的文件**：
- [[01-topics/agent-swarm-dynamic-topology]] — 更新竞品分析、创新性判定、Baseline增加AgentNet

## [2026-04-16] research | SwarmTopo 14周完整实施方案

用户完成项目准备工作（目录 + CLAUDE.md），要求制定从项目创建到论文发表的完整实施方案。

**创建的文件**：
- [[01-topics/implementation-plan]] — SwarmTopo 14 周完整实施方案
  - Phase 0（W1-W2）：验证期 — 极简 50 题实验 + Go/No-Go 决策
  - Phase 1（W3-W5）：核心框架 — 4 个组件实现 + 20 题集成测试
  - Phase 2（W6-W7）：完整系统 — 4 个 Baseline + 30 题测试
  - Phase 3（W8-W10）：全面实验 — 3 基准 × 5 方法 + 5 组消融
  - Phase 4（W11-W14）：论文写作 — 初稿 + 修改 + 投稿
  - 附录：文件依赖顺序、预算追踪、风险检查点、用户/Claude 分工表

**特色内容**：
- 每个任务配有可直接复制粘贴的 Claude Code 提示词
- 6 个风险检查点（CP1-CP6）+ 失败应对策略
- 总预算控制在 $50 以内（预计 $36-48）
- 6 个关键决策门：Go/No-Go、集成测试、统计显著性等

## [2026-05-07] learn | 新建多智能体综述论文阅读包

- 新建了 [[raw/paper/multi-agent/]] 目录，并下载 6 篇与 LLM 多智能体相关的综述论文 PDF
- 创建了 [[raw/paper/multi-agent/00-reading-manual|多智能体综述论文阅读手册]]，按“先全景、再协作、再通信、再工程、最后专题延伸”的顺序组织阅读路线
- 本次筛选优先保留了“高质量 + 较新 + 适合建立基础认知”的 general survey，并把过于垂直的多机器人、自动驾驶等 survey 暂时排除在主线外
- 在 [[wiki/index.md]] 中新增“多智能体综述阅读包（2026-05-07）”入口，便于从研究工作区直接跳转到阅读手册与论文原文

## [2026-05-07] create | 固化论文全文翻译学习笔记模板

- 新建了 [[_templates/paper-translation-lesson|paper-translation-lesson]]，用于后续把综述论文整理成“全文中文翻译 + 学习辅助解释”的系列笔记
- 模板中固化了 `paper_meta` 字段，以及“正文翻译 / 译者注（非原文）/ 理解提示（非原文）/ 易混点（非原文）/ 术语对照 / 思考题 / 上下篇导航”结构
- 明确规定所有新增解释必须单独标注为非原文，避免与原文翻译混写

## [2026-05-07] learn | 完成第 1 篇多智能体综述的学习友好版全文中文精读系列

- 新建了 [[raw/lessons/Multi-Agent-Survey/01-progress-and-challenges/00-overview|Multi-Agent Survey 中文精读系列总览（第 1 篇）]]，整理系列目标、术语约定、非原文标注规范与分篇导航
- 基于 [[raw/paper/multi-agent/01-large-language-model-based-multi-agents-survey-of-progress-and-challenges-2024.pdf]]，完成了第 1 篇综述论文的分篇全文中文翻译笔记：
  - [[raw/lessons/Multi-Agent-Survey/01-progress-and-challenges/01-survey-of-progress-and-challenges-introduction]]
  - [[raw/lessons/Multi-Agent-Survey/01-progress-and-challenges/02-survey-of-progress-and-challenges-background]]
  - [[raw/lessons/Multi-Agent-Survey/01-progress-and-challenges/03-survey-of-progress-and-challenges-dissecting-llm-ma]]
  - [[raw/lessons/Multi-Agent-Survey/01-progress-and-challenges/04-survey-of-progress-and-challenges-applications]]
  - [[raw/lessons/Multi-Agent-Survey/01-progress-and-challenges/05-survey-of-progress-and-challenges-supporting-tools-and-challenges]]
  - [[raw/lessons/Multi-Agent-Survey/01-progress-and-challenges/06-survey-of-progress-and-challenges-references]]
- 按原论文结构完整覆盖了摘要、导论、背景、系统拆解、应用、框架与基准、挑战、结论以及参考文献，满足“全文包含”要求
- 在分篇笔记中补入了统一的术语对照、图表语义说明与译者注，并严格使用“译者注（非原文）/ 理解提示（非原文）/ 易混点（非原文）”区分原文与补充内容
- 在 [[wiki/index.md]] 中补入”学习友好版中文精读系列”入口，并将学习笔记统计 81 → 88

## [2026-05-15] learn | 完成第 4 篇多智能体综述”Beyond Self-Talk”的学习友好版中文精读系列

- 新建了 [[raw/lessons/Multi-Agent-Survey/04-beyond-self-talk/00-beyond-self-talk-essence|第 4 篇-00 论文精华总览]]，提炼了论文的两级通信分析框架（系统间通信 + 系统内通信），并总结了 7 个精华判断和研究地图
- 基于论文 “Beyond Self-Talk: A Communication-Centric Survey of LLM-Based Multi-Agent Systems”（arXiv:2502.14321, 2025），完成了第 4 篇综述论文的分篇全文中文翻译笔记：
  - [[raw/lessons/Multi-Agent-Survey/04-beyond-self-talk/01-beyond-self-talk-background|01 摘要、导论与背景]] — 覆盖 Section 1 + 2，建立”通信为中心”的立论基础
  - [[raw/lessons/Multi-Agent-Survey/04-beyond-self-talk/02-beyond-self-talk-system-level|02 系统间通信：架构、目标与协议]] — 覆盖 Section 3，分析五种架构类型、三种通信目标和三种新兴协议
  - [[raw/lessons/Multi-Agent-Survey/04-beyond-self-talk/03-beyond-self-talk-system-internal|03 系统内通信：策略、范式、对象与内容]] — 覆盖 Section 4，分析微观通信机制的四个维度
  - [[raw/lessons/Multi-Agent-Survey/04-beyond-self-talk/04-beyond-self-talk-challenges-and-conclusion|04 挑战、未来方向与结论]] — 覆盖 Section 5 + 6，识别六大挑战方向与研究路线图
- 严格遵循了 Claudia 翻译笔记规范：frontmatter 完整（含 paper_meta 全部字段）、原文翻译与非原文补充严格分离（使用 note/tip/warning callout）、每篇包含术语对照表与思考题
- 建立了与第 3 篇”协作机制”的交叉引用：在 essence 中对比了两级通信框架与五维协作框架的互补关系，在 challenges 章中增加了跨框架思考题
- 在 [[wiki/index.md]] 中补入第 4 篇学习友好版中文精读系列入口，并将学习笔记统计 112 → 117

## [2026-05-15] learn | 完成第 5 篇多智能体综述"LLMs Working in Harmony"的学习友好版中文精读系列

- 新建了 [[raw/lessons/Multi-Agent-Survey/05-llms-working-in-harmony/00-llms-harmony-essence|第 5 篇-00 论文精华总览]]，提炼了论文的四柱技术框架（Architecture/Memory/Planning/Frameworks），并总结了 7 个精华判断和技术选型速查表
- 基于论文 "LLMs Working in Harmony: A Survey on the Technological Aspects of Building Effective LLM-Based Multi Agent Systems"（arXiv:2504.01963, 2025），完成了第 5 篇综述论文的分篇全文中文翻译笔记：
  - [[raw/lessons/Multi-Agent-Survey/05-llms-working-in-harmony/01-llms-harmony-introduction-and-architecture|01 导论与架构]] — 覆盖 Section I + II-A，拆解 CMD/CoA/Agent Forest/MoA 四种架构
  - [[raw/lessons/Multi-Agent-Survey/05-llms-working-in-harmony/02-llms-harmony-planning|02 规划框架]] — 覆盖 II-B，拆解 AdaPlanner/ChatCoT/KnowAgent/RAP/ToT/ReAct 六种规划方法
  - [[raw/lessons/Multi-Agent-Survey/05-llms-working-in-harmony/03-llms-harmony-memory|03 记忆框架]] — 覆盖 II-C，拆解 VecDB/RAG/ChatDB/MemoryBank/RET-LLM/SCM 六种记忆方案
  - [[raw/lessons/Multi-Agent-Survey/05-llms-working-in-harmony/04-llms-harmony-frameworks-and-conclusion|04 框架工具与结论]] — 覆盖 II-D + III-V，拆解 AutoGen/CAMEL/CrewAI/MetaGPT/LangGraph 五种框架
- 在 [[wiki/index.md]] 中补入第 5 篇学习友好版中文精读系列入口，并将学习笔记统计 117 → 122

## [2026-05-15] learn | 完成第 6 篇多智能体综述"Creativity in LLM-based MAS"的学习友好版中文精读系列

- 新建了 [[raw/lessons/Multi-Agent-Survey/06-creativity/00-creativity-mas-essence|第 6 篇-00 论文精华总览]]，提炼了论文的四大框架维度（工作流/技术/Persona/评估），并总结了 8 个精华判断和研究地图
- 基于论文 "Creativity in LLM-based Multi-Agent Systems: A Survey"（EMNLP 2025, 正式会议论文），完成了第 6 篇综述论文的分篇全文中文翻译笔记：
  - [[raw/lessons/Multi-Agent-Survey/06-creativity/01-creativity-mas-introduction-and-workflow|01 导论与工作流]] — 覆盖 Section 1-2，三阶段工作流（Planning/Process/Decision Making）与主动性谱系
  - [[raw/lessons/Multi-Agent-Survey/06-creativity/02-creativity-mas-techniques|02 创造力技术]] — 覆盖 Section 3，发散探索/迭代精炼/协作合成三大技术范式
  - [[raw/lessons/Multi-Agent-Survey/06-creativity/03-creativity-mas-persona|03 Persona 设计]] — 覆盖 Section 4，Coarse/Medium-Coarse/Fine 三层粒度与三种生成方式
  - [[raw/lessons/Multi-Agent-Survey/06-creativity/04-creativity-mas-evaluation|04 评估方法]] — 覆盖 Section 5，客观指标（6种）+ TTCT 主观改编 + 交互评估（CSI等）
  - [[raw/lessons/Multi-Agent-Survey/06-creativity/05-creativity-mas-challenges-and-conclusion|05 挑战与结论]] — 覆盖 Section 6-7 + Limitations，四大挑战 + 五个自身局限
- 在 [[wiki/index.md]] 中补入第 6 篇学习友好版中文精读系列入口，并将学习笔记统计 122 → 128

## [2026-05-16] learn | 完成三篇多智能体幻觉方向论文的翻译笔记

### G²CP 论文精读系列（6 篇）

- 新建了 [[raw/lessons/G2CP/00-essence|G2CP 精华总结]]，提炼了图接地通信协议的核心贡献、7 种执行语义、三大属性证明和实验结果（准确率+34%/token-73%/级联错误0%）
- 基于论文 "G²CP: A Graph-Grounded Communication Protocol for Verifiable and Efficient Multi-Agent Reasoning"（AAMAS 2026, arXiv:2602.13370），完成了分篇全文中文翻译笔记：
  - [[raw/lessons/G2CP/01-introduction|01 引言]] — 自然语言通信三大问题与四类歧义
  - [[raw/lessons/G2CP/02-related-work|02 相关工作]] — ACL 演进（KQML/FIPA/承诺语义）
  - [[raw/lessons/G2CP/03-protocol|03 协议定义]] — 形式化定义、操作语义、三大属性、承诺语义
  - [[raw/lessons/G2CP/04-architecture|04 多智能体架构]] — 四种智能体角色、运行时引擎、LLM 操作选择
  - [[raw/lessons/G2CP/05-experiments|05 实验评估]] — 500 合成场景、21 真实案例、理论分析

### DebUnc 论文精读系列（5 篇）

- 新建了 [[raw/lessons/DebUnc/00-essence|DebUnc 精华总结]]，提炼了不确定性感知辩论的核心贡献、三种度量、Attention Scaling 和 Oracle 分析
- 基于论文 "DebUnc: Improving Large Language Model Agent Communication With Uncertainty Metrics"（arXiv:2407.06426, 2024），完成了分篇全文中文翻译笔记：
  - [[raw/lessons/DebUnc/01-introduction|01 引言]] — LLM 幻觉问题与 DebUnc 动机
  - [[raw/lessons/DebUnc/02-related-work|02 相关工作]] — 不确定性度量与多智能体辩论研究
  - [[raw/lessons/DebUnc/03-method|03 方法]] — Mean Token Entropy/TokenSAR/Oracle、置信度转换、Attention Scaling
  - [[raw/lessons/DebUnc/04-experiments|04 实验与结果]] — Mistral-7B/Llama-3-8B 评估、Oracle 分析

### InEx 论文精读系列（6 篇）

- 新建了 [[raw/lessons/InEx/00-essence|InEx 精华总结]]，提炼了内省+跨模态协作双模块设计、TVER/VE-MHA 机制、跨模态共识流程
- 基于论文 "InEx: Hallucination Mitigation via Introspection and Cross-Modal Multi-Agent Collaboration"（arXiv 预印本, 2025），完成了分篇全文中文翻译笔记：
  - [[raw/lessons/InEx/01-introduction|01 引言]] — MLLM 幻觉问题、三类方法局限、InEx 框架概述
  - [[raw/lessons/InEx/02-related-work|02 相关工作]] — 幻觉缓解与多智能体协作方法
  - [[raw/lessons/InEx/03-preliminaries|03 预备知识]] — 熵理论、TVER 定义、VE-MHA 机制
  - [[raw/lessons/InEx/04-method|04 方法论]] — In/Ex 双模块完整实现、迭代验证流程
  - [[raw/lessons/InEx/05-experiments|05 实验与结果]] — 通用/幻觉基准、消融、鲁棒性分析

- 在 [[wiki/index.md]] 中补入三个论文精读系列入口，并将学习笔记统计 128 → 145

## [2026-05-20] create | 新增 AI 面试 RAG 深度笔记（2 篇）

- 创建了 raw/lessons/AI-Interview-Prep/03a-rag-indexing.md — RAG 索引与文档处理深入
- 创建了 raw/lessons/AI-Interview-Prep/03b-rag-retrieval.md — RAG 检索优化深入
