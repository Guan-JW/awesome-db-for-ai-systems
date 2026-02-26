# Survey Outline: Databases for AI Systems

---

## Abstract (Draft)

The rapid proliferation of large language model (LLM)-based AI systems — including chatbots, autonomous agents, and multi-agent frameworks — has surfaced a critical gap: the absence of a unified, mature data management layer. Current AI systems rely on an ad-hoc mixture of key-value stores, vector databases, and file systems that were not designed for the unique requirements of AI workloads. These requirements include managing multimodal data, supporting semantic similarity retrieval, maintaining consistency under concurrent AI writes, and enabling the "distillation" of unstructured agent context into queryable structured records.

This survey systematically examines the role of databases in modern AI systems across five dimensions: (1) agent memory architectures, (2) retrieval-augmented generation pipelines, (3) vector database systems and algorithms, (4) natural language interfaces to databases, and (5) enterprise AI database integration. We analyze existing solutions, identify open challenges, and propose a research agenda toward AI-native database design.

---

## 1. Introduction

- 1.1 The rise of LLM-based AI systems
- 1.2 Why data management matters — beyond storage
- 1.3 The gap: no unified DB layer for AI systems
- 1.4 Scope and contributions of this survey
- 1.5 Survey organization

## 2. Background & Taxonomy

- 2.1 Types of AI systems (chatbots, RAG apps, agents, multi-agent)
- 2.2 Data access patterns in AI systems (read-heavy retrieval, write-on-update, event sourcing)
- 2.3 Database types used today (KV, Vector, Relational, Graph, Time-series)
- 2.4 Taxonomy of DB roles in AI systems (Figure: taxonomy.png)

## 3. Agent Memory Systems

- 3.1 Memory architectures for LLM agents (CoALA framework)
- 3.2 Short-term context management
- 3.3 Long-term persistent memory (MemGPT, Generative Agents)
- 3.4 Memory distillation: from fuzzy context to structured records (Mem0, A-MEM)
- 3.5 Multi-agent shared memory
- 3.6 Open challenges: consistency, deduplication, eviction policies

## 4. Retrieval-Augmented Generation (RAG)

- 4.1 Canonical RAG architecture and the retrieval DB
- 4.2 Advanced retrieval strategies (self-RAG, corrective RAG, adaptive RAG)
- 4.3 Graph-based RAG
- 4.4 Multimodal RAG
- 4.5 Data pipeline challenges: chunking, embedding, indexing, staleness

## 5. Vector Database Systems

- 5.1 ANN algorithms: HNSW, IVF, DiskANN
- 5.2 System architecture: Milvus, Weaviate, Qdrant, pgvector
- 5.3 Hybrid search: vector + scalar predicates (VBASE)
- 5.4 Scalability, consistency, and durability
- 5.5 Benchmarking vector databases

## 6. Natural Language Interfaces to Databases

- 6.1 Text-to-SQL: datasets (Spider, BIRD, Spider 2.0) and methods (DIN-SQL, DAIL-SQL, MAC-SQL)
- 6.2 DB agents: multi-step planning over databases
- 6.3 NL interfaces beyond SQL (NL2KV, NL2Graph)
- 6.4 Evaluation challenges in enterprise settings

## 7. Multi-Agent Systems & Database Coordination

- 7.1 Shared state patterns (blackboard, event sourcing, pub/sub)
- 7.2 Framework analysis: AutoGen, LangGraph, AgentScope
- 7.3 Consistency and isolation in multi-agent DB access
- 7.4 Agent OS: toward a DB-centric agent runtime

## 8. Enterprise AI Database Integration

- 8.1 AI + OLTP: transactional AI systems
- 8.2 AI + OLAP: analytical AI (Text-to-SQL at warehouse scale)
- 8.3 AI + Time-series: monitoring and forecasting
- 8.4 AI + Graph: knowledge reasoning
- 8.5 AI + Streaming: real-time AI inference on event streams

## 9. AI-Native Database Design

- 9.1 Learned index structures
- 9.2 In-database ML inference
- 9.3 Semantic deduplication at scale
- 9.4 Multimodal storage
- 9.5 Toward a unified AI-native DB: open research agenda

## 10. Conclusion

- 10.1 Summary of findings
- 10.2 Key open challenges
- 10.3 Future research directions
