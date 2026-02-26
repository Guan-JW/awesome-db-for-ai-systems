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

**解决什么问题**：LLM 的知识储存在参数里，更新代价高、无法溯源、且存在幻觉。对于需要精确事实的任务（开放域问答、fact verification），纯参数记忆不够可靠。

**核心思路**：用"检索器 + 生成器"的两阶段架构替代纯参数记忆。检索器把整个 Wikipedia 编码成向量存入 FAISS，生成器在回答时先检索最相关的 k 个段落，再以这些段落作为 context 生成答案。Wikipedia = 可查询的向量数据库。

**实验结果**：在 Natural Questions、TriviaQA、WebQuestions 上超越同期最优的参数模型（T5-11B），且只用了小得多的生成器（BART-large）。关键是检索到的段落可以直接被人审查，增加了可解释性。

**与本 survey 的关联**：RAG 定义了一个核心范式：向量数据库作为 AI 的"外部知识库"。本 survey 的 RAG 章节本质上是在研究这个数据库该如何设计、如何管理、如何保持新鲜。

**小结**：RAG 把 Wikipedia 变成了一个可查询的向量数据库，证明了"外挂数据库"比"参数记忆"更适合知识密集型任务。

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

**解决什么问题**：标准 RAG 无脑检索，每次生成都查一遍数据库——这对简单问题（"法国首都是哪里"）是浪费，而且检索到的内容质量不保证。

**核心思路**：给 LLM 训练一套"反思 token"：[Retrieve] 决定是否需要检索，[IsREL] 判断检索到的段落是否相关，[IsSUP] 判断生成内容是否有文本支撑，[IsUSE] 综合评估答案质量。一次 forward pass 内完成检索决策、生成、评估。

**实验结果**：在 ASQA（开放域 QA）、FactScore（传记生成事实准确率）、PubHealth（健康声明验证）等任务上，Self-RAG 超越了 ChatGPT 和直接 RAG 基线。在不需要检索的任务上，Self-RAG 也不会强行检索，保持了生成效率。

**与本 survey 的关联**：Self-RAG 揭示了 DB 访问的"智能调度"问题——不是所有时刻都需要查数据库，按需访问才能提升效率和质量。这对本 survey 中"AI 系统如何高效使用数据库"这一问题有直接回答。

**小结**：Self-RAG 让模型学会了"要不要查数据库"，把 RAG 从无脑检索提升到了智能按需访问。

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

**解决什么问题**：传统 RAG 擅长回答"某个具体事实是什么"，但对于"整个文档集里讨论了哪些主要话题"这类全局性问题完全失效——因为向量检索只能找到局部相关片段，看不到整体。

**核心思路**：两阶段预处理 + 两种查询模式。预处理：① 用 LLM 从文档中抽取实体和关系构建知识图谱；② 用图社区检测（Leiden 算法）划分主题社区；③ 对每个社区生成摘要存入文本 DB。查询时：Global 模式遍历所有社区摘要生成宏观答案，Local 模式退化为向量检索精确片段。

**实验结果**：在 Podcast 转录和新闻文章数据集上，人类评估 GraphRAG 在"全面性"和"多样性"维度上显著优于标准 RAG（64-72% vs 28-36% 偏好率）。代价是预处理时间和 LLM 调用次数大幅增加。

**与本 survey 的关联**：GraphRAG 是本 survey 中"图数据库支撑 RAG"方向的核心案例——它把文档转化成图数据库，用图结构解决了向量 DB 无法处理的全局查询问题。

**小结**：GraphRAG 把文档变成知识图谱，解决了"RAG 看不见全局"的核心缺陷，证明图数据库在企业知识管理中不可缺少。

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

**解决什么问题**：RAG 领域在 2023-2024 年论文爆发，各种改进方法缺乏系统梳理，从业者难以把握整体脉络。

**核心思路**：提出 Naive→Advanced→Modular RAG 三代框架：
- **Naive RAG**：简单的检索-读取流程，问题在检索精度低、生成冗余
- **Advanced RAG**：加入 query rewriting（检索前）、reranking + context compression（检索后）
- **Modular RAG**：完全可插拔的组件架构，每个模块（检索器/重排器/生成器）可独立替换优化

每代框架都对应不同的数据库设计要求。

**实验结果**：综述性论文，无自有实验。总结了 RAGAS 等评估框架，给出 RAG 各模块的评测指标。

**与本 survey 的关联**：这是我们了解 RAG 领域全貌的最佳综述。我们的 survey 将从数据库视角切入，补充这篇综述在"存储层设计"上的空白——哪些数据库组件支撑了 RAG 的各个模块。

**小结**：最全的 RAG 综述，我们的研究将基于它，从"哪些数据库支撑 RAG 的各个组件"这个数据库视角提供新的贡献。
