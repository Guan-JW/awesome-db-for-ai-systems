# [Layer 3 · Execution] Paper Notes: Enterprise AI Integration Patterns

> How do enterprise AI systems integrate with existing database infrastructure?
> Key themes: RAG over enterprise data lakes, multi-source federated retrieval, data governance for AI outputs.
> **三层定位**：上层执行引擎——数据库在企业 AI 应用中的集成模式与治理实践。

---

## Open Problems

1. **Schema heterogeneity**: Enterprise data spans hundreds of schemas; how does AI navigate them?
2. **Access control**: Retrieved data must respect row-level security and column-level permissions
3. **Data freshness**: Enterprise KB changes continuously — how to keep vector indexes in sync?
4. **Auditability**: AI-generated answers based on retrieved data must be traceable to sources

---

## Industry Practices (non-paper evidence)

### [Azure AI Search + Foundry IQ] Reusable enterprise knowledge layer

- **Type**: Official product docs + official blog
- **Links**:
	- https://learn.microsoft.com/en-us/azure/search/retrieval-augmented-generation-overview
	- https://techcommunity.microsoft.com/blog/azure-ai-foundry-blog/foundry-iq-unlocking-ubiquitous-knowledge-for-agents/4470812

**What it does**:
> Evolves from classic single-query RAG to agentic retrieval (query decomposition, parallel source access, iterative retrieval, citations). Foundry IQ packages this as reusable knowledge bases that multiple agents can share.

**Enterprise value**:
> Solves duplicated “one team one RAG pipeline” construction. Keeps permission inheritance and governance in the retrieval layer instead of rebuilding them in each app.

**Limitation / Caution**:
> Some capabilities are in preview; for conservative production rollout, teams may still use classic RAG path first.

**简单说**：企业在把 RAG 从“每个团队自己拼”改成“统一知识层复用”。

---

### [Databricks Mosaic AI Vector Search] Delta-native retrieval stack

- **Type**: Official product docs
- **Link**: https://docs.databricks.com/aws/en/vector-search/vector-search

**What it does**:
> Builds vector index directly from Delta tables, supports auto-sync updates, metadata filtering, reranking, endpoint ACLs, and usage/cost monitoring.

**Enterprise value**:
> Reduces ETL split-brain between data lake and vector store. Teams can keep one data backbone and add retrieval features on top.

**Limitation / Caution**:
> Different endpoint types have explicit limits and trade-offs (capacity/latency/sync behavior). Some storage-optimized capabilities are preview-labeled.

**一句话总结**：Databricks 做的是让向量检索贴着数据湖跑，而不是另起一套。

---

### [Elasticsearch / OpenSearch] Hybrid retrieval as default production path

- **Type**: Official technical docs
- **Links**:
	- https://www.elastic.co/guide/en/elasticsearch/reference/current/knn-search.html
	- https://docs.opensearch.org/latest/vector-search/

**What it does**:
> Supports ANN + lexical hybrid search, pre-filtered kNN, rescoring, and chunk-level retrieval with nested vectors.

**Enterprise value**:
> Fits real corpora where exact identifiers (SKU, policy code, ticket id) and semantic meaning must both be matched.

**Limitation / Caution**:
> Tuning cost is real: `num_candidates`, quantization, filter strategy, and reranking settings materially affect latency and quality.

**备注**：线上很少纯向量检索，关键词+向量混合基本是标配。

---

### [MongoDB Atlas Vector Search] Document DB + vector retrieval convergence

- **Type**: Official product docs
- **Link**: https://www.mongodb.com/docs/atlas/atlas-vector-search/vector-search-overview/

**What it does**:
> Offers ANN/ENN search, hybrid search, and pre-filtering in the same document database workflow.

**Enterprise value**:
> For teams already using document DB heavily, this lowers migration friction and simplifies developer workflow.

**Limitation / Caution**:
> Strongly depends on embedding quality and schema discipline; retrieval quality can degrade if metadata fields are loosely managed.

不少团队不会单独引入向量库，而是在现有文档库里加向量能力。

---

### [Weaviate Query Agent] Agentic Search for Enterprise Retrieval

- **Type**: Official product + engineering blog
- **Links**:
	- https://weaviate.io/blog/legal-rag-app
	- https://docs.weaviate.io/agents/query

**What it does**:
> Query Agent 自动分析 collection schema、拆分复杂查询、构建精确过滤条件（日期/类型/管辖区）、调用 rerank sub-agent 做重排、最后合成带引用来源的回答。两种模式：Search Mode（发现式检索）和 Ask Mode（直接生成答案）。

**Enterprise value**:
> 对法律/金融这种需要精确性的场景，避免了 naive RAG 的"语义相似但不相关"问题（比如把 2022 年条款当 2024 年返回）。Agent 自动做的元数据过滤在传统 RAG 里需要开发者手写大量规则。

**跟 DB Agent 的区别**:
> Agentic Search 是只读的——agent 只决定检索策略，不会修改数据。DB Agent（如 Text-to-SQL 系统）可以执行写操作。这是一个重要的能力分界线，决定了对底层数据库的安全和事务需求。

**多模态亮点**:
> 用 multivector 模型（ColModernVBERT）直接对 PDF 页面做视觉编码，不走 OCR。Muvera 压缩把 multivector 降到可用的内存和延迟范围内。这是目前看到的最干净的"文档直接入向量库"方案。

---

## How to use these sources in survey chapters (三层架构版)

- **Chapter 3 (Vector DB — Layer 1)**: Use Elasticsearch/OpenSearch/MongoDB operational knobs as production-side evidence beyond ANN algorithms. Weaviate multivector + Muvera compression as multimodal vector storage case.
- **Chapter 6 (RAG — Layer 2)**: Use Azure/Databricks/Elastic evidence to argue why agentic + hybrid retrieval is becoming mainstream.
- **Chapter 8 (DB Agents — Layer 3)**: Weaviate Query Agent as the Agentic Search exemplar; contrast with DB Agents (read-only vs. read-write spectrum).
- **Chapter 9 (DB MAS — Layer 3)**: Use Foundry IQ and Databricks as two concrete "platformized knowledge layer" cases.
- **Chapter 11 (Governance — Cross-cutting)**: Use ACL, permission inheritance, and billing/usage observability details as practical governance evidence.
