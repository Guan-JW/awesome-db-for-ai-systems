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
