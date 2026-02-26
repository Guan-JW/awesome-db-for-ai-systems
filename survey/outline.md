# Survey Outline: Databases for AI Systems

> 三层架构版本

---

## Abstract (Draft)

The rapid proliferation of large language model (LLM)-based AI systems — including chatbots, autonomous agents, and multi-agent frameworks — has surfaced a critical gap: the absence of a unified, mature data management layer. Current AI systems rely on an ad-hoc mixture of key-value stores, vector databases, and file systems that were not designed for the unique requirements of AI workloads. These requirements include managing multimodal data, supporting semantic similarity retrieval, maintaining consistency under concurrent AI writes, and enabling the "distillation" of unstructured agent context into queryable structured records.

This survey systematically examines the role of databases in modern AI systems across three architectural layers: (1) DB as the Storage Engine — static data infrastructure including vector databases, training data management, and feature stores; (2) DB as the Memory System — runtime data management for agent memory, RAG pipelines, KV cache, and workflow state; and (3) DB as the Execution Engine — databases as application-layer participants including DB agents and multi-agent database systems. We analyze existing solutions, identify open challenges, and propose a research agenda toward AI-native database design.

---

## 1. Introduction

- 1.1 The rise of LLM-based AI systems
- 1.2 Why data management matters — beyond storage
- 1.3 The gap: no unified DB layer for AI systems
- 1.4 Three-layer taxonomy: Storage → Memory → Execution
- 1.5 Scope and contributions of this survey
- 1.6 Survey organization

## 2. Background & Taxonomy

- 2.1 Types of AI systems (chatbots, RAG apps, agents, multi-agent)
- 2.2 Data access patterns in AI systems (read-heavy retrieval, write-on-update, event sourcing)
- 2.3 Database types used today (KV, Vector, Relational, Graph, Time-series)
- 2.4 Three-layer taxonomy of DB roles in AI systems (Figure: taxonomy.png)

---

## Part I — DB as the Storage Engine (Bottom Layer)

> 数据库作为 AI 系统的静态数据仓库

## 3. Vector Database Systems

- 3.1 ANN algorithms: HNSW, IVF, DiskANN
- 3.2 System architecture: Milvus, Weaviate, Qdrant, pgvector
- 3.3 Hybrid search: vector + scalar predicates (VBASE)
- 3.4 Scalability, consistency, and durability
- 3.5 Benchmarking vector databases

## 4. Training Data & Embedding Management

- 4.1 Large-scale training data storage and versioning
- 4.2 Semantic deduplication for AI-generated data (SemDeDup)
- 4.3 Embedding lifecycle: generation, indexing, staleness, refresh
- 4.4 Learned index structures — ML meets DB internals
- 4.5 Feature stores: bridging training and serving

---

## Part II — DB as the Memory System (Middleware Layer)

> 数据库管理 AI 系统运行时动态产生的数据

## 5. Agent Memory Systems

- 5.1 Memory architectures for LLM agents (CoALA framework)
- 5.2 Short-term context management
- 5.3 Long-term persistent memory (MemGPT, Generative Agents)
- 5.4 Memory distillation: from fuzzy context to structured records (Mem0, A-MEM)
- 5.5 Multi-agent shared memory
- 5.6 Open challenges: consistency, deduplication, eviction policies

## 6. RAG & Retrieval Pipelines

- 6.1 Canonical RAG architecture and the retrieval DB
- 6.2 Advanced retrieval strategies (Self-RAG, Corrective RAG, Adaptive RAG)
- 6.3 Graph-based RAG (GraphRAG)
- 6.4 Multimodal RAG
- 6.5 From single-query RAG to agentic retrieval (enterprise evolution)
- 6.6 Data pipeline challenges: chunking, embedding, indexing, staleness

## 7. Runtime State, KV Cache & Coordination

- 7.1 KV cache management: offloading, sharing, lifecycle
- 7.2 Workflow state persistence: checkpoints, snapshots, time-travel (LangGraph)
- 7.3 Shared state patterns (blackboard, event sourcing, pub/sub)
- 7.4 Consistency and isolation in multi-agent DB access

---

## Part III — DB as the Execution Engine (Top Layer)

> 数据库作为应用层直接参与 AI 业务逻辑

## 8. DB Agents: Natural Language Interfaces to Databases

- 8.1 Text-to-SQL: datasets (Spider, BIRD, Spider 2.0) and methods (DIN-SQL, DAIL-SQL, MAC-SQL)
- 8.2 Single DB agent: multi-step planning over databases
- 8.3 NL interfaces beyond SQL (NL2KV, NL2Graph)
- 8.4 Evaluation challenges in enterprise settings

## 9. DB Multi-Agent Systems

- 9.1 Framework analysis: AutoGen, LangGraph, AgentScope
- 9.2 MAS for relational databases (Thucy, AutoTQA)
- 9.3 Agent OS: toward a DB-centric agent runtime
- 9.4 Enterprise AI database integration patterns

---

## Cross-cutting Concerns

## 10. AI-Native Database Design

- 10.1 Multimodal storage
- 10.2 In-database ML inference
- 10.3 Semantic deduplication at scale
- 10.4 Toward a unified AI-native DB: open research agenda

## 11. Enterprise Governance & Operations

- 11.1 Access control and permission inheritance for AI retrieval
- 11.2 Auditability and provenance tracking
- 11.3 Cost monitoring and capacity planning
- 11.4 Industry convergence: open-source vs. commercial platforms

## 12. Conclusion

- 12.1 Summary of findings across three layers
- 12.2 Key open challenges
- 12.3 Future research directions
