# Survey Outline: Databases for AI Systems

> 三层架构版本

---

## Abstract (Draft)

LLM-based AI systems — chatbots, agents, multi-agent frameworks — increasingly need proper data management, but current practice is ad-hoc. Most systems cobble together KV stores, vector databases, and file systems that were not designed for AI workloads. Issues include multimodal data handling, semantic similarity retrieval, consistency under concurrent AI writes, and converting unstructured agent context into queryable records.

This survey examines database roles in AI systems across three layers: (1) Storage Engine — vector DBs, training data management, feature stores; (2) Memory System — agent memory, RAG pipelines, KV cache, workflow state; (3) Execution Engine — DB agents and multi-agent database systems. We review existing solutions, identify gaps, and discuss directions toward AI-native database design.

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
- 2.4 Three-layer taxonomy of DB roles: Storage, Memory, Execution
- 2.5 **The Unified Architecture** (Figure: Integrated Data Flow from Storage to Execution)

---

## Part I — DB as the Storage Engine (Bottom Layer)

> 数据库作为 AI 系统的静态数据仓库

## 3. Vector Database Systems
- 3.1 ANN algorithms: HNSW, IVF, DiskANN
- 3.2 System architecture: Milvus, Weaviate, Qdrant, pgvector
- 3.3 Hybrid search: vector + scalar predicates (VBASE)

## 4. Feature Stores & Training Data
- 4.1 Feature Stores: The "ETL for AI" (Feast, Tecton)
  - *Point-in-Time Correctness*
  - *Online/Offline Skew*
- 4.2 Training Data Management (SemDeDup, DataComp)
- 4.3 Embedding Versioning & Lifecycle

---

## Part II — DB as the Memory System (Middleware Layer)

> 数据库管理 AI 系统运行时动态产生的中间状态

## 5. Agent Memory Systems
- 5.1 The Memory Hierarchy: Context Window vs. Vector DB vs. Graph
- 5.2 Retrieval-Augmented Generation (RAG) as Memory Retrieval
- 5.3 Memory Distillation: From unstructured logs to structured facts (Mem0)

## 6. Runtime State & KV Cache
- 6.1 **KV Cache Management**: The "Virtual Memory" of LLM Serving
  - *PagedAttention (vLLM)*
  - *Disaggregated Architecture (Mooncake)*
- 6.2 Agent State Persistence (LangGraph Checkpoints)
- 6.3 Multi-Agent Coordination via Shared State

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
