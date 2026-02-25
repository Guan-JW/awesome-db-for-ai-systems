# Paper Notes: Vector Database Systems

> How should vector stores be designed and optimized for AI workloads requiring semantic similarity search at scale?

---

## Key Concepts

- **ANN (Approximate Nearest Neighbor)**: Most vector DB operations are ANN, trading exact accuracy for speed
- **HNSW**: Graph-based ANN index; best recall/latency trade-off; widely adopted (Weaviate, Qdrant, pgvector)
- **IVF (Inverted File)**: Cluster-based ANN; better memory efficiency; used in FAISS
- **DiskANN**: Disk-based graph index; enables billion-scale search on a single machine
- **Hybrid Search**: Combining vector similarity + scalar/keyword filtering (increasingly common in production)

---

## Entry Template

```
### [ShortName] Full Title

- **Authors**: ...
- **Venue**: ...
- **PDF/Code**: ...

**Summary**: ...
**Index Type**: ...
**Key Contribution**: ...
```

---

## Papers

### [FAISS] Billion-scale similarity search with GPUs

- **Authors**: Johnson et al. (Meta AI)
- **Venue**: IEEE Trans. Big Data 2021
- **PDF**: https://arxiv.org/abs/1702.08734
- **Code**: https://github.com/facebookresearch/faiss

**Summary**:
> GPU-accelerated similarity search library. Implements IVF, HNSW, PQ, and their combinations. De-facto standard for offline ANN search.

**Index Type**: IVF + PQ (quantization for compression), HNSW

**Key Contribution**:
> GPU batching for ANN; quantization to fit billion-scale indexes in memory.

---

### [HNSW] Efficient and Robust ANN Search Using Hierarchical Navigable Small World Graphs

- **Authors**: Malkov & Yashunin (Yandex)
- **Venue**: IEEE Trans. PAMI 2020
- **PDF**: https://arxiv.org/abs/1603.09320
- **Code**: https://github.com/nmslib/hnswlib

**Summary**:
> Graph-based ANN. Builds a multi-layer small-world graph. Query traverses from top (coarse) to bottom (fine-grained) layers. Currently the dominant in-memory ANN index.

**Key Contribution**:
> Best recall-latency trade-off among ANN algorithms; now the default index in Pinecone, Weaviate, Qdrant, pgvector.

---

### [Milvus] Milvus: A Purpose-Built Vector Data Management System

- **Authors**: Wang et al. (Zilliz)
- **Venue**: SIGMOD 2021
- **PDF**: https://dl.acm.org/doi/10.1145/3448016.3457550
- **Code**: https://github.com/milvus-io/milvus

**Summary**:
> First system paper on a production vector database. Addresses log-structured storage, multi-index type support, cloud-native deployment, and consistency.

**DB / Storage Used**:
> Tiered storage: hot data in-memory → warm data on SSD → cold data in object storage. WAL for write consistency.

**Key Contribution**:
> Defines the architecture of a purpose-built vector DB: separation of storage/compute, tiered storage (memory → SSD → object storage), WAL for consistency.

#### 阅读笔记

**解决什么问题**：通用数据库（MySQL、PostgreSQL）或纯向量库（FAISS）都不够用——前者不支持高效向量检索，后者缺乏数据管理能力（持久化、一致性、并发写入、多种索引类型切换）。生产级 AI 系统需要一个专门为向量设计的"完整数据库"。

**核心思路**：Milvus 的四个关键设计决策：
- **存储计算分离**：查询节点可独立扩展，不受写入瓶颈影响，云原生部署
- **多级存储分层**：热数据 in-memory（毫秒延迟）→ 温数据 SSD（秒级召回）→ 冷数据对象存储（按需加载），大幅降低成本
- **多索引类型支持**：同一个 collection 可按场景选择 HNSW（高召回率）、IVF（低内存）、DiskANN（十亿级磁盘索引），用户无需更换系统
- **WAL + 一致性保证**：Write-Ahead Log 确保向量插入的原子性和崩溃恢复，首次将关系 DB 的 ACID 部分引入向量 DB

**实验结果**：SIGMOD 2021 实验展示：在十亿级向量数据集上，Milvus 比 FAISS+PostgreSQL 组合快 5-10×；支持每秒数百万次查询（QPS），延迟 <10ms（p99）。

**与本 survey 的关联**：Milvus 是 RAG/Agent 记忆系统在生产环境中最常用的向量 DB 之一，定义了 purpose-built vector DB 的标准架构——多级存储、多索引类型、WAL 一致性。理解 Milvus 就理解了为什么 AI 系统需要专用向量数据库而不是通用 DB 打补丁。

**小结**：Milvus 是第一个完整的生产级向量数据库 system paper——它首次回答了"如何像运营关系数据库一样运营向量数据库"的工程问题。

---

### [VBASE] Unifying Online Vector Similarity Search and Relational Queries via Relaxed Monotonicity

- **Authors**: Zhang et al. (PKU / Microsoft)
- **Venue**: OSDI 2023
- **PDF**: https://www.usenix.org/conference/osdi23/presentation/zhang-qianxi
- **Code**: https://github.com/microsoft/MSVBASE

**Summary**:
> Enables SQL-like predicates on top of vector similarity search without rebuilding indexes. Introduces "relaxed monotonicity" as the theoretical bridge between ANNS and relational operators.

**Key Contribution**:
> Critical paper for **hybrid search**: allows queries like `SELECT * FROM docs WHERE category='finance' ORDER BY embedding <-> query_vec LIMIT 10` efficiently.

---

### [VectorDB Survey] Survey of Vector Database Management Systems

- **Authors**: Han et al. (Purdue)
- **Venue**: VLDB Journal 2024
- **PDF**: https://arxiv.org/abs/2310.14021

**Summary**:
> Comprehensive taxonomy and benchmark of vector DBs: pure vector (Pinecone, Weaviate, Qdrant, Milvus), vector-extended relational (pgvector, VSAG), and in-memory libraries (FAISS, hnswlib).

**Key Contribution**:
> Best reference for the "Vector Database Systems" chapter of the survey.
