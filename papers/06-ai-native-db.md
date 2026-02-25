# Paper Notes: AI-Native Database Design

> How should database systems be redesigned or extended to natively support AI workloads?
> Core challenges: multimodal data, model-in-database inference, AI-generated data deduplication, learned index structures, consistency under AI writes.

---

## Open Problems (from meeting discussion)

1. **Multimodal data management**: Text, images, audio, video in one store — no unified mature solution yet
2. **AI-generated data deduplication**: LLM outputs may be semantically duplicate; traditional hash-based dedup fails
3. **Consistency under concurrent AI writes**: Multi-agent systems may write conflicting facts simultaneously
4. **Memory distillation pipeline**: Convert fuzzy LLM context → precise structured DB records at scale
5. **KV store limitations**: Current AI systems heavily use KV DBs but they lack query expressiveness for complex agent state

---

## Papers

### [Learned Index] The Case for Learned Index Structures

- **Authors**: Kraska et al. (Google)
- **Venue**: SIGMOD 2018
- **PDF**: https://arxiv.org/abs/1712.01208
- **Code**: https://github.com/learnedsystems/RMI

**Summary**:
> Replaces B-tree / hash indexes with learned models (neural networks) that predict record positions. 2× faster lookups with 100× smaller index size on read-heavy workloads.

**Key Contribution**:
> Proof of concept that ML can replace core DB data structures — foundational for "AI-native DB" discussion.

---

### [VBASE] Unifying Online Vector Similarity Search and Relational Queries

*(Also listed in vector-db.md — cross-reference)*

- **Venue**: OSDI 2023
- **Key Contribution**: Hybrid queries (vector similarity + SQL predicates) without index rebuild.

---

### [SemDeDup] SemDeDup: Data-efficient Learning at Web-scale through Semantic Deduplication

- **Authors**: Abbas et al. (Meta AI)
- **Venue**: arXiv 2023
- **PDF**: https://arxiv.org/abs/2303.09540
- **Code**: https://github.com/facebookresearch/SemDeDup

**Summary**:
> Clusters embeddings via k-means; within each cluster, removes near-duplicates based on cosine similarity threshold. Applied to LAION-2B, removes 50% of data while matching or exceeding full-data performance.

**DB / Storage Used**:
> Embedding store (FAISS) + cluster assignment store.

**Key Contribution**:
> Directly addresses the **AI-generated data deduplication** problem raised in the meeting: semantic (not syntactic) dedup via vector DB.

---

### [AI-Native DB Survey] Data Management for Large Language Models: A Survey

- **Authors**: Zhao et al. (Renmin University)
- **Venue**: arXiv 2023→2024
- **PDF**: https://arxiv.org/abs/2312.01700

**Summary**:
> Surveys DB challenges introduced by LLMs: training data management, inference data management (KV cache), and application-level data management (RAG, memory). Proposes a unified "LLM-DB co-design" agenda.

**Key Contribution**:
> Most directly aligned survey with our research scope — must read. Defines the gap our survey should fill.

---

### [DB for AI Survey] Database Technology for the Era of Large Language Models

- **Authors**: Zhou et al. (Tsinghua)
- **Venue**: VLDB Journal 2024
- **PDF**: https://arxiv.org/abs/2402.02643
- **⚠️ ID 待确认**: 原 ID `2402.01117` 为 DTS-SQL（文不对题）。VLDB Journal 版本可能无 arxiv 预印本，可查 https://doi.org/10.1007/s00778-024-00864-x

**Summary**:
> Two perspectives: (1) DB for AI — how databases support LLM training, storage, RAG, agents; (2) AI for DB — how LLMs improve DB systems (NL2SQL, query optimization, self-tuning).

**Key Contribution**:
> Comprehensive framing; our survey focuses on (1) DB for AI but should be aware of (2) AI for DB as context.
