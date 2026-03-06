# [Layer 1 · Storage] Paper Notes: Vector Database Systems

> How should vector stores be designed and optimized for AI workloads requiring semantic similarity search at scale?
> **三层定位**：底层存储引擎——向量数据库是 AI 系统最核心的静态数据基础设施之一。

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

Milvus 这篇是 SIGMOD 2021 的 system paper，讲的是怎么从零设计一个面向向量检索的完整数据库，不只是搜索引擎。

我觉得它解决的最根本的问题是：FAISS 这类库只管"搜"，不管"存"。没有持久化、没有崩溃恢复、没法多人并发写入。一旦要上生产环境，就得自己在外面包一层数据管理逻辑，非常折腾。Milvus 把这些能力内建了。

几个比较关键的设计点：
- 存储计算分离，查询节点可以独立扩缩容
- 分层存储：热数据放内存、温数据落 SSD、冷数据扔对象存储。这个跟传统 DB 的 buffer pool 思路基本一样
- 提供 WAL（预写日志），崩溃后能恢复。这个在向量库里比较少见
- 一个 collection 可以选不同索引类型（HNSW、IVF、DiskANN），不用换系统

论文里给的数据是十亿级向量上比 FAISS+PostgreSQL 组合快 5-10 倍，p99 延迟 <10ms。不过这个对比有点不公平，毕竟 FAISS+PG 本来就不是为一起用设计的。

放在 survey 里，Milvus 可以作为第 3 章（Vector DB）的核心案例，用来说明"为什么 AI 系统需要专用向量数据库而不是在现有 DB 上打补丁"。

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

---

## 补充资料：行业实践与技术博客

下面是一些论文之外的参考材料，主要来自各向量数据库厂商的工程博客和中文社区讨论，读论文啃不动的时候可以先看这些建立直觉。

- **Pinecone 官方学习指南 "What is a Vector Database?"**  
  https://www.pinecone.io/learn/vector-database/  
  非常适合入门，配图清晰，把向量数据库和传统数据库的区别讲得很到位。缺点是作为商业公司文档有一定倾向性，对自家产品的局限性一笔带过。

- Qdrant 技术博客里有一篇关于 HNSW 参数调优的文章（https://qdrant.tech/articles/filtrable-hnsw/ ），讲了 `ef_construct`、`M` 这些参数怎么影响召回率和延迟。做过向量检索的都知道调这些参数是玄学，这篇给了一些经验值，比较实用。

- Zilliz（Milvus 背后的公司）的工程博客 https://zilliz.com/blog 上干货不少，比较推荐他们关于十亿级向量分布式检索的几篇实战文章。虽然免不了夹带私货推自家方案，但工程细节比论文里写得详细多了。

- 知乎上搜"向量数据库选型"能找到好几个高赞回答，比较有参考价值的是对比 Milvus / Qdrant / Weaviate / pgvector 各自适合什么场景的讨论。大体结论是：小规模用 pgvector 省事，中等规模 Qdrant 性能好还轻量，大规模上 Milvus 生态成熟但运维成本高。不过知乎帖子时效性强，具体版本对比可能已经过时了。

- pgvector 的 GitHub README（https://github.com/pgvector/pgvector ）值得快速过一遍。在已有 PostgreSQL 基础设施的团队里，加个扩展就能做向量检索，不需要额外引入一套新系统。Ann Benchmarks 上的性能数据也在不断改善。

---

## 补充：多模统一检索与 AI 原生方案

> 参看 `papers/L1-Storage Engine/L1-04-ai-native-db-seekdb.md`

除了上面这些"纯向量"或"向量扩展"方案，还有一条路线是把向量能力直接内建到多模数据库里：

- **seekdb (OceanBase)**：在一个引擎里原生支持向量索引（HNSW/IVF）+ 全文索引（BM25）+ 关系型 + JSON + GIS。一条 SQL 里可以同时做向量召回和关键词召回，然后用 RRF 或 LLM 重排合并结果。跟 pgvector 相比，seekdb 的全文和混合检索能力更完整；跟 Elasticsearch 相比，seekdb 有完整的 ACID 事务和多表 JOIN。
- 这种"全包型"方案的取舍跟专用向量库正好相反：灵活性和部署简单性好，但在极端向量规模（百亿级）下的性能能否追上 Milvus 还需要观察。
