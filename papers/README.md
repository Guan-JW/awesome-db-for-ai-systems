# Paper Notes Index

Reading notes organized by the three-layer architecture (底层存储 → 中间层记忆 → 上层执行).

---

## Layer 1 · DB as the Storage Engine (Bottom Layer)

| File                                                 | Topic                                | Key Papers                                 |
| ---------------------------------------------------- | ------------------------------------ | ------------------------------------------ |
| [L1-01-vector-db.md](L1-01-vector-db.md)             | Vector Database Systems              | Milvus, FAISS, HNSW, VBASE                 |
| [L1-02-data-management.md](L1-02-data-management.md) | Training Data & Embedding Management | Learned Index, SemDeDup, DB-for-AI Surveys |

## Layer 2 · DB as the Memory System (Middleware Layer)

| File                                             | Topic                                  | Key Papers                             |
| ------------------------------------------------ | -------------------------------------- | -------------------------------------- |
| [L2-01-agent-memory.md](L2-01-agent-memory.md)   | Agent Memory Systems                   | MemGPT, Generative Agents, Mem0, A-MEM |
| [L2-02-rag.md](L2-02-rag.md)                     | RAG & Retrieval Pipelines              | RAG, Self-RAG, GraphRAG, RAG Survey    |
| [L2-03-runtime-state.md](L2-03-runtime-state.md) | Runtime State, KV Cache & Coordination | LangGraph *(待补充: vLLM, SGLang 等)*  |

## Layer 3 · DB as the Execution Engine (Top Layer)

| File                                                               | Topic                  | Key Papers                                          |
| ------------------------------------------------------------------ | ---------------------- | --------------------------------------------------- |
| [L3-01-text-to-sql.md](L3-01-text-to-sql.md)                       | DB Agent (Text-to-SQL) | BIRD, Spider, MAC-SQL, DIN-SQL, DAIL-SQL            |
| [L3-02-multi-agent-db.md](L3-02-multi-agent-db.md)                 | DB Multi-Agent Systems | CoALA, AutoGen, AgentScope, Thucy, AutoTQA          |
| [L3-03-enterprise-integration.md](L3-03-enterprise-integration.md) | Enterprise Integration | Azure AI Search, Databricks, Elasticsearch, MongoDB |
