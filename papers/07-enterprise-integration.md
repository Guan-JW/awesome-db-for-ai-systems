# Paper Notes: Enterprise AI Integration Patterns

> How do enterprise AI systems integrate with existing database infrastructure?
> Key themes: RAG over enterprise data lakes, multi-source federated retrieval, data governance for AI outputs.

---

## Open Problems

1. **Schema heterogeneity**: Enterprise data spans hundreds of schemas; how does AI navigate them?
2. **Access control**: Retrieved data must respect row-level security and column-level permissions
3. **Data freshness**: Enterprise KB changes continuously — how to keep vector indexes in sync?
4. **Auditability**: AI-generated answers based on retrieved data must be traceable to sources

---

## Papers

*(To be populated — focus on production deployments and system papers describing enterprise-scale RAG/agent deployments)*

### Suggested Papers to Add

- **Databricks Mosaic AI** — managed RAG platform, vector search integrated into Delta Lake
- **Glean** — enterprise search with LLM layer, multi-source federated retrieval
- **Microsoft Fabric** — unified analytics platform with copilot and vector search
- **Amazon Kendra / Q** — enterprise knowledge retrieval with access control
