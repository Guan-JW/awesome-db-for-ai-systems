# Paper Notes: AI Agent Memory Systems

> How do AI agents store, retrieve, and manage knowledge across sessions?
> Key themes: short-term context, long-term persistent memory, memory distillation (converting fuzzy AI context → precise structured DB records).

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

**解决什么问题**：LLM 上下文窗口有限（早期 ~4K token），无法进行跨 session 的长文档分析和持久化对话记忆，导致 agent 每次对话都"失忆"。

**核心思路**：把 LLM 类比为操作系统进程，上下文窗口 = RAM，外部存储 = 磁盘：
- **Main context**（RAM）= LLM 实时处理的内容
- **Archival memory**（慢存储）= 向量数据库，按语义相似度 ANN 检索
- **Recall memory**（缓存）= KV / 关系 DB，按时间/关键词检索对话历史

Agent 通过 **function call（类比系统调用/中断）** 主动在层级间换入换出数据，无需人工干预。

**实验结果**：两个场景验证——① 分析超长文档（远超 context window）；② 多 session 持久化对话记忆，对话信息跨会话保留。MemGPT 可以"记住"几周前对话中的事项。

**与本 survey 的关联**：MemGPT 是 DB-for-AI 的最直接示范——它将向量 DB 和关系 DB 作为 agent 记忆的第一公民，定义了 archival memory = 向量 DB、recall memory = 关系 DB 的架构范式，是本 survey 第一章的核心论文。

**小结**：MemGPT 把向量数据库变成了 LLM 的"外部硬盘"，第一次系统性地证明了数据库对 AI agent 的不可替代性。

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

**解决什么问题**：如何让 AI agent 表现出可信的、类人的长期行为——包括记住过去发生的事、从经验中建立高层认知、与其他 agent 产生社会性协作涌现。

**核心思路**：25 个 agent 在一个类《模拟人生》的小镇沙盒中运行，每个 agent 有三层机制：
- **Memory stream**（记忆流）= append-only 事件日志，记录所有观测、对话、行动
- **检索机制** = 三维评分加权：时效性（recency, 指数衰减）+ 重要性（importance, LLM 打分 1-10）+ 相关性（relevance, 余弦相似度）
- **Reflection 机制** = LLM 定期从原始观测中归纳出高层洞察（如"Klaus 喜欢绘画"）并写回记忆，把 unstructured log 蒸馏成 structured knowledge

**实验结果**：用户只给了"在小镇举办情人节舞会"的初始想法，25 个 agent 自主完成了邀请、计划、参加的全流程——涌现行为无需显式编程。用户评测中 agent 行为被认为比无记忆基线更真实可信。

**与本 survey 的关联**：Generative Agents 把向量检索作为记忆系统的基础，首次大规模实验展示了"数据库驱动的 agent 社会"。Reflection 机制本质是从 unstructured log 向 structured knowledge 的自动蒸馏——对应本 survey 中"记忆压缩与知识提炼"这一研究方向。

**小结**：25 个 AI agent 靠一个向量数据库"记忆流"就实现了可信的社会行为涌现，直接证明了数据库设计决定 agent 行为质量。

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
