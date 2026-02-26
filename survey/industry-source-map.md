# Industry & Practice Source Map (for Survey)

> 目标：把“非论文证据”系统化，避免只停留在学术视角。
> 使用规则：每个结论至少两类来源交叉验证（官方文档 + 开源 issue/benchmark/案例）。

## 证据分级

- **A 级（强证据）**：官方技术文档、API 文档、可复现实验/代码、公开 benchmark
- **B 级（中证据）**：官方博客、架构解读、厂商案例复盘
- **C 级（辅助证据）**：公众号/社区文章/二次解读

## 一、闭源平台（大厂成熟做法）

1. **Azure AI Search（RAG + Agentic Retrieval）** — A/B

   - 文档：https://learn.microsoft.com/en-us/azure/search/retrieval-augmented-generation-overview
   - Foundry IQ 博客：https://techcommunity.microsoft.com/blog/azure-ai-foundry-blog/foundry-iq-unlocking-ubiquitous-knowledge-for-agents/4470812
   - 可用于章节：`6.RAG`、`3.Vector DB`、`9.DB MAS / Enterprise`
2. **Databricks Mosaic AI Vector Search** — A

   - 文档：https://docs.databricks.com/aws/en/vector-search/vector-search
   - 可用于章节：`6.RAG`、`3.Vector DB`、`9.DB MAS / Enterprise`
3. **Snowflake Cortex / AI & ML** — A

   - 文档：https://docs.snowflake.com/en/guides-overview-ai-features
   - 可用于章节：`9.DB MAS / Enterprise`、`11.Governance`
4. **MongoDB Atlas Vector Search** — A

   - 文档：https://www.mongodb.com/docs/atlas/atlas-vector-search/vector-search-overview/
   - 可用于章节：`3.Vector DB`、`8.DB Agent`、`9.DB MAS / Enterprise`
5. **Elasticsearch kNN / Hybrid Search** — A

   - 文档：https://www.elastic.co/guide/en/elasticsearch/reference/current/knn-search.html
   - 可用于章节：`6.RAG`、`3.Vector DB`

## 二、开源基础设施（可复现）

1. **Milvus（云原生向量数据库）** — A

   - 架构文档：https://milvus.io/docs/architecture_overview.md
   - 代码：https://github.com/milvus-io/milvus
2. **Qdrant（多租户、别名切换、WAL）** — A

   - 文档：https://qdrant.tech/documentation/concepts/collections/
   - 代码：https://github.com/qdrant/qdrant
3. **Weaviate（动态索引、向量 + 倒排）** — A

   - 文档：https://docs.weaviate.io/weaviate/concepts/indexing
   - 代码：https://github.com/weaviate/weaviate
4. **OpenSearch Vector Search（语义/混合/多模态/RAG）** — A

   - 文档：https://docs.opensearch.org/latest/vector-search/
   - 代码：https://github.com/opensearch-project/OpenSearch
5. **LangGraph（长期运行 agent 的持久状态编排）** — A/B

   - 文档：https://docs.langchain.com/oss/python/langgraph/overview
   - 代码：https://github.com/langchain-ai/langgraph

## 三、框架与工程生态（把 DB 用起来）

1. **LlamaIndex 存储层指南** — A

   - 文档：https://developers.llamaindex.ai/python/framework/module_guides/storing/
   - 可用于章节：`5.Agent Memory`、`6.RAG`
2. **Pinecone 学习中心（向量数据库工程要点）** — B

   - 文档：https://www.pinecone.io/learn/vector-database/
   - 可用于章节：`3.Vector DB`、`9.DB MAS / Enterprise`

## 四、章节映射（按三层架构）

> 以下映射对应重组后的 survey/outline.md 章节编号。

- **Chapter 3 (Vector DB — Layer 1)**: Use Milvus, Qdrant, Weaviate architecture docs; Elasticsearch/OpenSearch/MongoDB operational knobs as production-side evidence.
- **Chapter 4 (Training Data & Embedding — Layer 1)**: Use Databricks Delta Sync, embedding lifecycle docs.
- **Chapter 5 (Agent Memory — Layer 2)**: Use LlamaIndex storage guides, Mem0 docs.
- **Chapter 6 (RAG — Layer 2)**: Use Azure AI Search, Databricks, Elastic evidence to argue why agentic + hybrid retrieval is becoming mainstream.
- **Chapter 7 (Runtime State — Layer 2)**: Use LangGraph checkpointing docs, vLLM/SGLang KV cache management.
- **Chapter 8 (DB Agent — Layer 3)**: Use Spider/BIRD benchmarks and enterprise NL2SQL deployment cases.
- **Chapter 9 (DB MAS — Layer 3)**: Use AutoGen, AgentScope, Foundry IQ as multi-agent platform evidence.
- **Chapter 11 (Governance — Cross-cutting)**: Use ACL, permission inheritance, billing/usage observability details from Azure, Databricks, Qdrant.

## 五、中文讲解与行业传播（辅助理解）

建议长期跟踪（公众号/社区）：

- 机器之心
- DataFunTalk
- 腾讯云开发者
- 阿里云开发者社区
- InfoQ 中文（AI / 数据基础设施专题）

使用建议：

- 只摘取“问题定义、术语解释、案例线索”
- 关键结论务必回链到 A/B 级来源

## 五、后续...

- 每周至少补 5 条来源（其中 A 级不少于 3 条）
- 每条来源补三行：`讲了什么 / 能支撑哪章 / 有什么局限`
- 对有时效性的云厂商能力（预览版、beta）标注日期与状态
