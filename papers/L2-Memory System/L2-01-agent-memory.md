# [Layer 2 · Memory] Paper Notes: AI Agent Memory Systems

> How do AI agents store, retrieve, and manage knowledge across sessions?
> Key themes: short-term context, long-term persistent memory, memory distillation (converting fuzzy AI context → precise structured DB records).
> **三层定位**：中间层记忆系统——管理 Agent 运行时产生的动态记忆数据。

---

## Entry Template

```
### [ShortName] Full Title

- **Authors**: ...
- **Venue**: ...
- **PDF/Code**: ...

**Summary**: ...
**DB / Storage Used**: ...
**Key Contribution**: ...
```

---

## Papers

### [MemGPT] MemGPT: Towards LLMs as Operating Systems

- **Authors**: Charles Packer et al.
- **Venue**: NeurIPS 2023 (Workshop)
- **PDF**: https://arxiv.org/abs/2310.08560
- **Code**: https://github.com/cpacker/MemGPT

**Summary**:
> Treats the LLM as an OS process. Implements a virtual context management system with main context (in-context), external storage (archival memory, recall memory backed by a database), and function calls to move data between tiers.

**DB / Storage Used**:
> Archival memory: vector database (embedding + ANN search). Recall memory: key-value / relational store for conversation history.

**Key Contribution**:
> First to systematically define a memory hierarchy (analogous to CPU registers / RAM / disk) for LLM agents, backed by explicit database abstractions.

**Limitations / Open Questions**:
> Memory eviction policy is simple; no consistency guarantees across agents.

#### 阅读笔记

这篇的核心想法是把 OS 的存储层级搬到 LLM 上来：上下文窗口 = RAM，向量数据库 = 磁盘，agent 通过 function call 在两层之间换入换出数据。

当时（2023 年）context window 还只有 4K，跨 session 记忆基本不可能。MemGPT 用了两种外部存储：archival memory 放向量库做语义检索，recall memory 放关系型 DB 按时间/关键词查历史对话。

实验是在两个场景上验的——超长文档分析（远超 context window）和多 session 持久记忆。效果还行，但记忆替换策略比较简单，多 agent 之间也没有一致性保证，后续工作应该有改进空间。

这篇对 survey 第 5 章很重要，可以作为"agent memory = 外挂数据库"这个范式的起始案例。

---

### [Generative Agents] Generative Agents: Interactive Simulacra of Human Behavior

- **Authors**: Joon Sung Park et al.
- **Venue**: UIST 2023
- **PDF**: https://arxiv.org/abs/2304.03442
- **Code**: https://github.com/joonspk-research/generative_agents

**Summary**:
> Simulates believable human behavior with 25 agents. Each agent has a memory stream (flat log of observations), a retrieval mechanism (recency + importance + relevance scoring), and a reflection mechanism that synthesizes higher-level insights.

**DB / Storage Used**:
> Flat append-only event log + vector similarity retrieval. "Reflection" distills raw observations into structured summaries — an early form of memory distillation.

**Key Contribution**:
> Introduces the *memory stream + reflection* paradigm, which is the conceptual basis for "memory distillation".

#### 阅读笔记

这篇做了个挺有意思的实验：25 个 AI agent 在虚拟小镇里生活，每个 agent 有个 append-only 的"记忆流"记录看到的一切。检索时不只按相似度，而是用 recency（时效性，指数衰减）+ importance（重要性，LLM 打 1-10 分）+ relevance（语义相关度）三个维度加权检索。

更有意思的是反思机制（reflection）：agent 定期从一堆零散记忆里归纳出高层结论，比如"Klaus 喜欢画画"，然后把归纳结果也写回记忆流。相当于把散乱的原始 log 提炼成结构化知识。

最终效果：研究者只给了一个"办情人节舞会"的 prompt，25 个 agent 自发完成了邀请、策划、参加的全过程。用户测评认为有反思机制的 agent 行为比无记忆的要真实很多。

对我们 survey 的意义：数据库的设计（怎么存、怎么检索、怎么聚合）直接影响了 agent 行为的质量。这个可以放第 5 章讨论"记忆压缩和知识提炼"的时候用。

---

### [Mem0] Mem0: Building Production-Ready AI Agents with Scalable Long-Term Memory

- **Authors**: Mem0 AI Team
- **Venue**: arXiv 2025
- **PDF**: https://arxiv.org/abs/2504.19413
- **Code**: https://github.com/mem0ai/mem0

**Summary**:
> Production memory layer for AI agents. Extracts facts from conversations using LLM, stores them in a hybrid store (vector DB + graph DB + key-value), and serves them at inference time.

**DB / Storage Used**:
> Vector DB (Qdrant/Chroma) + graph DB (Neo4j) + KV store (Redis). Demonstrates the need for **multi-store hybrid architecture** in production.

**Key Contribution**:
> Shows practical memory distillation: LLM call converts unstructured dialogue → structured facts → database entries.

---

### [A-MEM] A-MEM: Agentic Memory for LLM Agents

- **Authors**: Northwestern University
- **Venue**: arXiv 2025
- **PDF**: https://arxiv.org/abs/2502.12110
- **Code**: https://github.com/meeting-minutes/A-MEM

**Summary**:
> Dynamic memory network inspired by the Zettelkasten note-taking system. Each memory note has structured attributes and cross-links; the agent autonomously evolves the network over time.

**DB / Storage Used**:
> Graph-structured memory store; each note is a node with explicit relational links.

**Key Contribution**:
> Memory is not just stored but *actively organized and interconnected*, closer to a knowledge graph than a flat vector store.

---

## 补充资料：行业实践与技术博客

- **LangChain Memory 模块文档**：https://python.langchain.com/docs/modules/memory/  
  LangChain 的 memory 模块是目前用得最多的 agent 记忆方案之一。文档里把 ConversationBufferMemory、ConversationSummaryMemory、VectorStoreRetrieverMemory 等几种实现都列出来了，可以快速理解工程上都有哪些记忆模式。看完再回去读 MemGPT 会发现概念对得上。

- **Letta（MemGPT 项目）官方博客**：https://www.letta.com/blog  
  MemGPT 团队后来注册了 Letta 这个公司继续做，博客上有不少关于分层记忆架构工程落地的文章。比论文里讲得更接地气，包括怎么选向量库、记忆更新的触发时机、跨 session 的状态同步等等。

- **Mem0 项目文档和博客**：https://docs.mem0.ai/  
  跟论文里读的那篇 Mem0 配合着看。项目文档直接有代码示例，用起来挺简单的——几行代码就能给 agent 加上跨 session 记忆。他们还专门写了一篇 blog 讲为什么需要"记忆层"而不是把所有聊天记录塞进 prompt。

- 知乎和公众号上搜"AI Agent 记忆设计"、"大模型长期记忆"可以找到一些实践分享。印象比较深的一个观点是：大部分团队现阶段的 agent 记忆其实就是"检索历史对话然后塞进 system prompt"，还远远算不上真正的记忆系统。这个批评挺中肯的，跟 MemGPT 论文里的 motivation 也对得上。

- Cognee（https://github.com/topoteretes/cognee ）是另一个做 agent memory 的开源项目，思路偏知识图谱方向，跟 A-MEM 有点像。star 数不算多但代码写得比较清晰，想了解图结构记忆怎么实现的可以翻翻。

---

## 多模态记忆与 DB 原生记忆方案

> 参看 `papers/cross-cutting/multimodal-data-management.md` 和 `papers/L1-Storage Engine/L1-04-ai-native-db-seekdb.md`

上面几篇（MemGPT、Mem0、A-MEM）都是在应用层实现 agent 记忆，底层拼装向量库+图库+KV 库。另一个思路是把记忆直接做进数据库内核：

- **seekdb PowerMem**：OceanBase 的 seekdb 数据库内置了 Agent 记忆存储模块。记忆的存取在数据库事务内完成，天然有 ACID 保证——不会出现多 Agent 并发更新时的记忆不一致问题。相比 Mem0 那种应用层方案，少了跨系统同步的复杂度，但跟具体数据库绑定了。

- **多模态记忆**是另一个值得关注的 gap：目前所有 agent memory 系统都只处理文本。但具身智能场景里，agent 需要记住"在哪个位置看到了什么物体"（视觉+空间记忆），这需要数据库能同时管理向量、GIS 和结构化数据的混合查询。

---

## 工程案例：OpenClaw 的记忆架构

> **来源**：字节跳动技术团队公众号，2026-02-09
> https://mp.weixin.qq.com/s/Sx52DN9kktgri77z6cj3vg
> OpenClaw（原名 Clawdbot）是当下最火的开源个人 AI Agent 框架（GitHub 161K stars）。

OpenClaw 的记忆实现值得仔细看，因为它是目前**用户量最大的 agent 记忆系统实际落地**，而不是学术原型。

### 四类上下文分层

OpenClaw 把 Agent 的上下文分成四类，这个分类跟我们三层架构的映射关系很清晰：

| OpenClaw 上下文                   | 性质               | 对应三层         | 存储方式                            |
| :-------------------------------- | :----------------- | :--------------- | :---------------------------------- |
| 长期知识 (Durable Knowledge)      | 持久化事实和偏好   | L1 / L2 边界     | `MEMORY.md` + 日志文件              |
| 任务记忆 (Task Memory)            | 多步任务的中间产物 | L2 Runtime State | 日期文件 `memory/YYYY-MM-DD.md`     |
| 会话历史 (Conversational History) | 完整对话记录       | L2 Short-term    | `sessions/<id>.jsonl` (append-only) |
| 外部资源 (External Resources)     | 代码库、文档、API  | L1 Storage       | 文件系统 + 外部索引                 |

### 记忆分层设计

记忆不是一坨——OpenClaw 明确区分了两层核心结构：
- **日常日志层**：`memory/YYYY-MM-DD.md`，时序性信息，append-only
- **核心记忆层**：`MEMORY.md`，提炼后的稳定事实和偏好

实验性的扩展更有意思——`bank/` 目录按语义分类：
- `world.md`（客观事实）
- `experience.md`（agent 自身经历）
- `opinions.md`（主观判断 + 置信度 + 证据指向）
- `entities/`（实体信息库，每个实体一个文件）

这种分层跟 Generative Agents 的 memory stream + reflection 在思路上是一致的，但 OpenClaw 把它落到了文件系统级别的具体结构上。

### 两套 Backend 方案

特别值得关注的是 OpenClaw 提供了**两种完全不同的记忆存储方案**，这直接映射到我们 survey 讨论的核心问题——agent 记忆该用什么数据库？

**方案 A：文件 + SQLite Backend**
- 记忆本体：Markdown 文件是 source of truth（人可读、可版本控制）
- 索引后端：SQLite 充当索引层
  - `fts5` 扩展做 BM25 全文检索
  - `sqlite-vec` 扩展做向量相似度搜索
  - 两者混合检索实现语义+关键词的联合查询
- 表结构：`files`（文件元数据）→ `chunks`（分块+向量）→ `chunks_fts`（FTS5 虚拟表）→ `chunks_vec`（向量虚拟表）→ `embedding_cache`（避免重复 embedding）

**方案 B：LanceDB Plugin**
- 完全独立的链路，取代了文件+SQLite 整套体系
- LanceDB 作为嵌入式向量数据库，自带存储 + embedding + 检索
- 三个 agent tool：`memory_recall`（搜索）、`memory_store`（保存）、`memory_forget`（删除）
- 生命周期钩子实现自动化：
  - `before_agent_start`：自动检索相关记忆注入 prompt（auto-recall）
  - `agent_end`：自动从对话中抽取值得记住的信息写入 LanceDB（auto-capture）
- 记忆分类：用 `MemoryCategory` 枚举自动分类（启发式规则匹配）

### 自动记忆刷新（Auto Memory Flush）

一个工程细节很有启发：会话历史过长、即将被压缩（compaction）之前，系统自动触发一个静默 agent turn，提示模型把当前重要上下文存入 Memory 文件。这实际上是一种**数据库层面的 checkpoint 前持久化**——跟 vLLM 的 PagedAttention swap 逻辑有异曲同工之处：在资源不够（context window 溢出）之前，先把关键数据落盘。

### 对 survey 的价值

这个案例对我们 survey 来说有三层价值：

1. **验证三层架构的解释力**：OpenClaw 的四类上下文可以无缝映射到我们的 L1/L2 分层，说明三层框架不是空中楼阁，能解释真实系统的设计选择。

2. **回答"用什么数据库"的问题**：两套 backend 方案（SQLite+FTS5+sqlite-vec vs LanceDB）是"agent 记忆该用什么 DB"这个问题的活生生的 A/B 测试。SQLite 方案更透明（Markdown 作为 source of truth），LanceDB 方案更自动化（钩子驱动的 auto-capture/recall）。

3. **嵌入式 DB 在 agent 记忆中的位置**：SQLite 和 LanceDB 都是嵌入式数据库，不需要独立服务进程。这跟 seekdb 的嵌入式模式定位一致——个人 agent 的记忆不需要 Milvus 集群那种重型方案，轻量嵌入式方案更实际。

### LanceDB 的特性

LanceDB 基于 Lance 列式格式，几个特性跟 agent memory 场景很匹配：
- 本地优先（local-first），嵌入式部署，无服务
- 多模态存储（图片、文档、音视频）
- 多类别索引 + 混合检索（标量 + 向量 + 全文）
- 数据版本管理内置（基于 Lance 格式的 append-only 版本链）

生态项目值得跟踪：
- **lance-graph**：基于 Lance 的图查询引擎（支持 Cypher），可以做 agent 的知识图谱
- **lance-context**：基于 Lance 的多模态、带版本的上下文存储，专门为 agent workflow 设计
