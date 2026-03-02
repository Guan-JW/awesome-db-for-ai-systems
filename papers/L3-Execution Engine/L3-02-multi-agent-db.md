# [Layer 3 · Execution] Paper Notes: DB Multi-Agent Systems

> How do multi-agent systems use databases as both coordination infrastructure and application target?
> Key insight: as agent complexity grows, in-memory shared state is insufficient — persistent, queryable databases become essential.
> **三层定位**：上层执行引擎——多个 Agent 协作操作数据库，DB 既是协调工具又是应用对象。

---

## Papers

### [CoALA] Cognitive Architectures for Language Agents

- **Authors**: Sumers et al. (Princeton)
- **Venue**: TMLR 2024
- **PDF**: https://arxiv.org/abs/2309.02427

**Summary**:
> Proposes a unified framework for language agents along three dimensions: Memory (working/episodic/semantic/procedural), Action space (grounding/retrieval/reasoning/learning), and Decision-making cycle (propose→evaluate→select→execute). Maps each memory type to a database category.

**DB / Storage Used**:
> Conceptual mapping: working memory → context window (no persistent DB); episodic memory → append-only log / vector DB; semantic memory → KG / relational DB; procedural memory → code store.

**Key Contribution**:
> Provides formal taxonomy for mapping agent memory types to database categories — essential theoretical foundation for the survey.

**DB Mapping**:
```
Working memory   → LLM context window (no persistent DB)
Episodic memory  → Append-only log / vector DB (retrieval by similarity)
Semantic memory  → Knowledge graph / relational DB
Procedural memory → Code store / retrieval-augmented code DB
```

#### 阅读笔记

Princeton 的框架论文，借鉴认知科学（Soar、ACT-R）来给 LLM Agent 做分类。核心贡献是把 agent 记忆分成四类，每类刚好对应一种数据库：
- 工作记忆 → context window（不需持久 DB）
- 情节记忆 → append-only log / 向量库
- 语义记忆 → 知识图谱 / 关系库
- 过程记忆 → 代码仓库

综述性质，没有自有实验，但回顾了 ReAct、Voyager、Generative Agents 等几十个系统，用这个框架统一描述。

对我们 survey 来说这篇是理论基础：每种记忆类型需要什么样的数据库？分类体系可以直接用来组织第 5 章和第 9 章的材料。

---

### [AutoGen] AutoGen: Enabling Next-Gen LLM Applications via Multi-Agent Conversation

- **Authors**: Qingyun Wu et al. (Microsoft)
- **Venue**: ICLR 2024
- **PDF**: https://arxiv.org/abs/2308.08155
- **Code**: https://github.com/microsoft/autogen

**Summary**:
> Framework where agents communicate through a shared conversation history (the DB). Supports customizable agent roles, human-in-the-loop, and tool use.

**DB / Storage Used**:
> Conversation history stored in-memory (dict); persistent backends optional. No built-in long-term memory DB — a known limitation.

**Key Contribution**:
> De-facto standard for multi-agent application development; reveals the gap in built-in persistent storage.

#### 阅读笔记

微软的多 agent 框架，ICLR 2024。把 agent 间交互抽象成"对话"：每个 agent 有角色和系统提示，通过 send/receive message 通信，底层就是共享的 in-memory dict。三种模式：纯 LLM 对话、human-in-loop、LLM+代码执行。

用数十行 Python 就能搭出之前要几百行才能实现的多 agent 应用。在数学推理、代码生成、问答等任务上都验证过。

但有个明显的缺陷：对话历史全在内存里，没有内置的持久化方案。session 一断全丢。也没有跨 session 记忆，没有并发写入保护。作为目前最流行的 multi-agent 框架，连最基本的持久化都缺失，恰好说明了这个领域对数据库支持的需求。我们 survey 可以拿这个当 motivation。

---

### [AgentScope] AgentScope: A Flexible yet Robust Multi-Agent Platform

- **Authors**: Gao et al. (Alibaba)
- **Venue**: arXiv 2024
- **PDF**: https://arxiv.org/abs/2402.14034
- **Code**: https://github.com/modelscope/agentscope

**Summary**:
> Production multi-agent platform. Introduces a typed message passing system and a Monitor (global KV store for metrics/state sharing across agents).

**DB / Storage Used**:
> Monitor: in-memory KV store with quota tracking. Explicit file / model storage backends.

**Key Contribution**:
> One of the few frameworks with explicit consideration for inter-agent *database-backed* state sharing.

---

### [LangGraph] LangGraph: Building Stateful, Multi-Agent Applications with LLMs

- **Authors**: LangChain Team
- **Code**: https://github.com/langchain-ai/langgraph

**Summary**:
> Graph-based workflow engine for multi-agent LLM apps. Each node is an agent; edges represent control flow. State is a typed dict persisted in a configurable DB (SQLite / PostgreSQL / Redis by default).

**DB / Storage Used**:
> Checkpointer: SQLite (default), PostgreSQL, or Redis. State snapshots enable time-travel debugging and fault tolerance.

**Key Contribution**:
> First major framework to make *persistent, queryable state* a first-class citizen — not an afterthought.
