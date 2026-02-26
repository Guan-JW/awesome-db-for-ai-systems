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

**解决什么问题**：LLM Agent 研究层出不穷，但每个系统有自己的术语（"记忆"、"工具"、"规划"），无法横向比较，也没有统一的设计原则。

**核心思路**：借鉴认知科学和经典 AI 认知架构（Soar, ACT-R），提出 CoALA 框架：
- **记忆模块**：工作记忆（上下文 window）、情节记忆（事件日志）、语义记忆（知识库）、过程记忆（代码/策略）
- **动作空间**：外部动作（物理/数字环境）、内部动作（检索/推理/学习）
- **决策循环**：规划阶段（提案+评估+选择）→ 执行阶段 → 循环

把 LLM 类比为概率产生式系统（probabilistic production system）。回顾性分析了 ReAct、Voyager、Generative Agents、Tree of Thoughts 等数十个 agent 系统，统一放入框架描述。

**实验结果**：综述性框架论文，无自有实验。通过案例分析展示框架的表达能力，并用框架揭示各系统的设计取舍。

**与本 survey 的关联**：本 survey 最重要的理论基础——CoALA 的内存分类直接对应不同数据库类型，为 survey 的核心章节提供了分类体系骨架。每种记忆类型背后都需要特定 DB 支撑。

**小结**：CoALA 把 LLM Agent 的"记忆"分成四类，每类都对应一种数据库——这正是我们 survey 的核心论点之一：什么数据库支撑了什么类型的 AI 记忆。

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

**解决什么问题**：多智能体协作系统开发门槛高——每个系统要手写 agent 间通信逻辑，难以统一管理 LLM 调用、工具调用、人工介入。

**核心思路**：把多智能体抽象为"可对话 Agent"的有向图——每个 agent 有角色 + 系统提示，通过 send/receive message 接口通信（底层是共享对话历史的 in-memory dict，实为"临时 DB"）。三种执行模式：纯 LLM 自主对话、LLM + human-in-loop、LLM + 代码执行。只需配置角色和初始消息，框架自动接管调度。

**实验结果**：在数学推理（Math）、代码生成（HumanEval）、在线决策、问答等任务上验证，用数十行 Python 实现原本需要数百行的多 agent 应用。

**与本 survey 的关联**：AutoGen 暴露了关键 DB 缺口——对话历史只在内存中，没有内置持久化、没有跨 session 记忆、没有并发写入保护。这是本 survey 的核心 motivation：成熟的多智能体框架显然需要数据库来支撑。

**小结**：AutoGen 是最流行的多智能体框架，但它"对话历史在内存"的做法在长期任务中完全不可持久——这正是我们 survey 想解决的核心问题。

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
