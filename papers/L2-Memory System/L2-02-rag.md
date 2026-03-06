# [Layer 2 · Memory] Paper Notes: RAG & Retrieval Pipelines

> How do retrieval systems bridge LLM knowledge gaps using external databases?
> Key themes: dense retrieval, adaptive retrieval, graph-structured retrieval.
> **三层定位**：中间层记忆系统——RAG 检索管线管理 AI 运行时的动态上下文组装与知识注入。

---

## Papers

### [RAG] Retrieval-Augmented Generation for Knowledge-Intensive NLP Tasks

- **Authors**: Patrick Lewis et al. (Facebook AI)
- **Venue**: NeurIPS 2020
- **PDF**: https://arxiv.org/abs/2005.11401
- **Code**: https://github.com/huggingface/transformers/tree/main/examples/research_projects/rag

**Summary**:
> Combines a parametric generator (seq2seq LM) with a non-parametric retriever (dense passage retrieval over Wikipedia). At inference time, the model retrieves top-k passages and conditions generation on them.

**DB / Storage Used**:
> Wikipedia dump encoded as dense vectors (FAISS index, ~21M passages). The retriever is a bi-encoder (question encoder + passage encoder). The vector index acts as the "external memory database".

**Key Contribution**:
> Foundational paper that establishes the RAG paradigm: external vector DB replaces parametric knowledge storage. Sets the template for all subsequent RAG systems.

#### 阅读笔记

RAG 开山之作。把整个 Wikipedia（2100 万段落）编码成向量塞进 FAISS，问答时先检索 top-k 段落、再把段落拼上问题喂给生成器。用一个 BART-large 就干过了参数量大得多的 T5-11B，关键是答案可以追溯到具体段落。

这篇定义了一个基本范式：向量数据库充当 LLM 的外部知识库。我们 survey 第 6 章讨论 RAG 时，基本上就是在讨论"这个向量数据库该怎么设计、怎么维护"。

---

### [Self-RAG] Self-RAG: Learning to Retrieve, Generate, and Critique through Self-Reflection

- **Authors**: Asai et al. (University of Washington)
- **Venue**: ICLR 2024
- **PDF**: https://arxiv.org/abs/2310.11511
- **Code**: https://github.com/AkariAsai/self-rag

**Summary**:
> Trains a single LM to adaptively retrieve passages on demand, generate output with inline "reflection tokens" (e.g., [Retrieve], [IsREL], [IsSUP], [IsUSE]), and self-evaluate quality — all in one forward pass.

**DB / Storage Used**:
> Same dense retrieval over external corpus (Wikipedia / custom). The key innovation is not the DB itself but *when and whether to query it*.

**Key Contribution**:
> First to make retrieval a *learned, conditional* behavior. The model decides when the DB is worth querying, producing more accurate and fact-grounded outputs than always-retrieve baselines.

#### 阅读笔记

这篇解决的问题很实在：标准 RAG 每次生成都去查一遍库，但有些问题根本不需要查（"法国首都在哪"），查了反而引入噪声。Self-RAG 给 LLM 加了一组"反思 token"，让模型自己决定要不要检索、检索的内容是否相关、生成的回答是否有据等。

在 ASQA、FactScore 等任务上都比直接 RAG 强。而且在不需要检索的题目上，它自动跳过检索，不浪费时间。

对 survey 来说，这篇说明的是"什么时候该访问数据库"本身就需要一套策略。不是一直查就是好的。

---

### [GraphRAG] From Local to Global: A Graph RAG Approach to Query-Focused Summarization

- **Authors**: Edge et al. (Microsoft Research)
- **Venue**: arXiv 2024
- **PDF**: https://arxiv.org/abs/2404.16130
- **Code**: https://github.com/microsoft/graphrag

**Summary**:
> Builds a knowledge graph from source documents (entity extraction → community detection → hierarchical summarization), then answers queries by traversing relevant graph communities. Specifically designed for "global" queries that require synthesizing information across many documents.

**DB / Storage Used**:
> Knowledge graph (nodes = entities/claims, edges = relations). Graph community summaries stored as additional text DB. Vector index over summaries for local queries.

**Key Contribution**:
> Addresses the fundamental RAG failure mode: standard RAG cannot synthesize themes across an entire corpus. Graph structure enables "global" comprehension.

#### 阅读笔记

这篇解决的痛点很明确：普通 RAG 做不了全局性问题。你问"这 1000 篇文档的主要话题有哪些"，向量检索只能给你几个最相关的片段，根本看不到整体。

GraphRAG 的做法是预处理阶段先用 LLM 从文档里抽实体和关系建成知识图谱，然后用 Leiden 算法做社区检测、对每个社区生成摘要。查询时如果是全局问题就遍历所有社区摘要，如果是局部问题就退化为普通向量检索。

人类评估比标准 RAG 在"全面性"和"多样性"上好得多（64-72% vs 28-36%）。但代价也不小，预处理阶段的 LLM 调用量挺大的。

对 survey 来说这篇很关键，说明了向量数据库不是万能的，有些场景确实需要图数据库才行。可以放在第 6 章讨论不同存储后端各自的适用范围。

---

### [RAG Survey] Retrieval-Augmented Generation for Large Language Models: A Survey

- **Authors**: Gao et al.
- **Venue**: arXiv 2023
- **PDF**: https://arxiv.org/abs/2312.10997

**Summary**:
> Comprehensive survey organizing RAG into three paradigms: Naive RAG (retrieve-then-read), Advanced RAG (pre/post retrieval processing), and Modular RAG (pipeline components). Covers indexing, retrieval, generation, and evaluation.

**DB / Storage Used**:
> N/A (survey paper). Covers all DB types used in RAG: dense vector indexes, sparse BM25, hybrid search, knowledge graphs.

**Key Contribution**:
> Best reference for the "RAG & Retrieval Systems" chapter of the survey.

#### 阅读笔记

综述性论文。把 RAG 分成三代：Naive RAG（简单的 retrieve-then-read）、Advanced RAG（加了 query rewriting、reranking、上下文压缩）、Modular RAG（完全组件化，模块可插拔）。还总结了 RAGAS 等评估方法。

没有自有实验，但整理得比较全面。我们 survey 第 6 章可以在此基础上补一个"数据库视角"：每代 RAG 架构对底层存储有什么不同的要求？比如 Naive RAG 只需要向量检索，Advanced RAG 需要元数据过滤和重排，Modular RAG 可能需要混合存储。

---

## 补充资料：行业实践与技术博客

工程上做 RAG 跟论文里讲的差距挺大的，下面这些资料偏实战，踩坑经验比较多。

- LlamaIndex 的 RAG 最佳实践博客：https://www.llamaindex.ai/blog  
  他们自己踩了很多坑总结出来的，比如 chunk size 该怎么选、metadata filtering 的重要性、reranking 到底能提升多少。个人觉得比很多论文的实验部分更有参考价值。

- LangChain RAG Cookbook：https://python.langchain.com/docs/tutorials/rag/  
  手把手教你搭 RAG pipeline，从文档切分到检索到生成。代码能直接跑，适合想快速复现的同学。不过 LangChain 版本迭代太快，隔几个月 API 可能就变了。

- **Pinecone RAG Guide**（https://www.pinecone.io/learn/retrieval-augmented-generation/ ）——Pinecone 出的 RAG 教程，虽然私心推自家向量库，但对 RAG 整体架构的介绍很清晰，尤其是 chunk + embed + upsert + query 这套流程的图画得很直观。

- 知乎上有不少 RAG 实践经验帖，搜"RAG 踩坑"或者"检索增强生成 实际效果"。社区里反复提到的问题包括：检索回来的结果不相关、chunk 边界切断了语义、长文档上召回率低等等。这些问题在 Self-RAG 和 GraphRAG 论文里都有对应的解决方案，但工程上怎么 trade-off 还是得看场景。

- Jerry Liu（LlamaIndex 创始人）在 Twitter/X 上经常发 RAG 相关的讨论串，内容质量高且紧跟最新进展，推荐关注：https://x.com/jerryjliu0

- 公众号"夕小瑶科技说"等 AI 公众号有时候会发 RAG 的中文实践总结，对英文资料看着费劲的同学比较友好。

---

## 延伸方向：多模态 RAG 与库内 RAG

> 参看 `papers/cross-cutting/multimodal-data-management.md` 和 `papers/L1-Storage Engine/L1-04-ai-native-db-seekdb.md`

### 多模态 RAG

传统 RAG 只检索文本段落。实际应用中越来越多需要检索图表、公式、图片甚至视频片段。代表性工作：
- **MuRAG**（Chen et al. 2022）：支持图文混合检索和生成
- **VisRAG**（Yu et al. 2024）：视觉增强的 RAG pipeline

数据库层面的挑战在于：同一个 chunk 可能同时有文本向量和图片向量，混合检索需要跨模态的统一排序。

### 库内 RAG（In-Database RAG）

另一个值得关注的趋势是把 RAG 管线从应用层搬进数据库。seekdb 的 PowerRAG 是目前比较完整的实现——文档入库时自动完成解析、分块、Embedding、索引，查询时一条 SQL 就能跑完 retrieve → rerank → generate 的全流程（通过 AI_EMBED、AI_COMPLETE、AI_RERANK 三个库内函数）。

这个"Doc In, Data Out"的思路跟存储过程类似，减少了数据在应用层和存储层之间的搬运。不过缺点也很明显：跟特定 LLM 服务绑定了，灵活性不如 LangChain/LlamaIndex。
