# [Layer 1 · Storage] Paper Notes: Training Data & Embedding Management

> How should database systems manage training data, embeddings, and AI-native storage at scale?
> Core challenges: semantic deduplication, learned indexes, multimodal data, embedding lifecycle.
> **三层定位**：底层存储引擎——训练数据管理、Embedding 管理、AI 原生存储设计。

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

#### 阅读笔记

B-tree 用了 40 多年，所有数据用同一种树结构做索引。但很多真实数据其实有明显规律，比如时间戳基本单调递增、邮编按地区聚簇。Learned Index 的想法很直接：训练一个小模型来"记住"数据的分布，直接预测 key 在哪个位置，跳过树遍历。

在有序整数数据集上，比 B-tree 快 2 倍，索引体积缩小 100 倍。但只适合读多写少的场景，写入频繁的话模型得不断重训。

跟我们的研究方向稍微有点偏：我们主要看"DB 怎么服务 AI"，这篇其实是"AI 怎么改造 DB"。不过后面讨论 AI-Native DB（第 10 章）的时候需要引用，算是了解背景。

---

### [VBASE] Unifying Online Vector Similarity Search and Relational Queries

*(Also listed in L1-01-vector-db.md — cross-reference)*

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

#### 阅读笔记

互联网规模的训练集（LAION-2B 有 20 亿条图文对）里大量内容意思差不多但表述不同，传统哈希去重根本查不出来。这篇的做法是先用预训练模型把数据编码成向量，用 k-means 聚类，然后在每个簇内按余弦相似度删掉过于相似的样本。底层用 FAISS 做大规模向量检索和聚类。

比较有意思的结论是：在 LAION-2B 上删掉 50% 的数据之后，下游 CLIP 模型性能居然没降甚至略有提升。说明删掉的确实是冗余。

survey 第 4 章可以用这篇来说明：AI 时代的数据清洗需要的是"语义级"去重，而不是简单的字节比对，向量数据库在训练阶段就已经有用武之地了。

---

### [AI-Native DB Survey] Data Management for Large Language Models: A Survey

- **Authors**: Zhao et al. (Renmin University)
- **Venue**: arXiv 2023→2024
- **PDF**: https://arxiv.org/abs/2312.01700

**Summary**:
> Surveys DB challenges introduced by LLMs: training data management, inference data management (KV cache), and application-level data management (RAG, memory). Proposes a unified "LLM-DB co-design" agenda.

**Key Contribution**:
> Most directly aligned survey with our research scope — must read. Defines the gap our survey should fill.

#### 阅读笔记

人大的这篇综述，跟我们 survey 的选题重叠度最高。他们也是从数据管理角度看 LLM，分了训练数据管理、推理数据管理（KV Cache 之类的）、应用数据管理（RAG、Agent 记忆）三层。这个分法跟我们的三层架构其实很接近。

和我们的区别：他们偏"数据管理流程"讲得多，比如数据收集、清洗、去重流程；我们底下大纲更强调数据库系统本身的技术（存储引擎、索引结构、一致性保证）。写 survey 的时候需要在 related work 里把这个差异说清楚。

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

#### 阅读笔记

清华这篇拉得比较大，双向都看了："DB for AI"（数据库怎么服务 LLM）和"AI for DB"（LLM 怎么改进数据库自身，如 Text-to-SQL、自动调参）。

对我们来说主要参考前半部分。后面"AI for DB"可以在背景章节简单提一下就行，不需要展开。这篇帮我们明确了 scope：我们的 survey 应该聚焦在 "DB for AI" 这一侧。

---

## Feature Store 方向

> Feature Store 是训练数据基础设施的重要组成部分，但目前学术论文较少，主要以开源系统和行业实践为主。

### [Feast] Feast: An Open Source Feature Store

- **Code**: https://github.com/feast-dev/feast
- **Docs**: https://docs.feast.dev/
- **Type**: 开源系统

**Summary**:
> 最流行的开源 Feature Store。连接离线存储（数据湖/数据仓库）和在线存储（Redis / DynamoDB），保证训练时和线上推理时使用同一套特征定义，解决 training-serving skew 问题。

**DB / Storage Used**:
> Offline store: BigQuery, Snowflake, Redshift, file-based (Parquet)
> Online store: Redis, DynamoDB, SQLite, PostgreSQL
> Registry: S3 / GCS 上的元数据

**Key Contribution**:
> 定义了 Feature Store 的基本架构模式：离线存储（批量计算特征）、在线存储（毫秒级查询）、注册表（特征定义元数据）、物化过程（定期把离线特征同步到在线存储）。

#### 阅读笔记

Feature Store 学术论文不多，主要看系统文档和社区实践。Feast 的核心卖点是解决 training-serving skew：训练在离线数据仓库（BigQuery/Snowflake）上跑 Python 算特征，线上用 Java/Go 重写一套，两边逻辑不同步就导致模型在线效果变差。

Feast 的解法是搞一个声明式的特征定义（YAML 或 Python SDK），统一描述特征的来源和计算逻辑，然后提供"物化"（materialization）流程把离线算好的特征定期灌到在线存储（Redis/DynamoDB）。这样训练和线上用的特征来自同一份定义。

放在 survey 里属于第 4 章的内容。Feature Store 虽然不如向量数据库那么"AI 专属"，但它处理的离线/在线一致性、低延迟查询、特征版本管理，底下都是数据库层面的问题。

---

### [Databricks Feature Store / Feature Engineering]

- **Docs**: https://docs.databricks.com/en/machine-learning/feature-store/index.html
- **Type**: 商业平台

**Summary**:
> 基于 Delta Lake 和 Unity Catalog 构建的 Feature Store。特征表就是 Delta 表，天然支持版本控制、血缘追踪、权限管理和自动物化到在线存储。

**Key Contribution**:
> 把 Feature Store 从独立组件变成数据平台的"内置能力"——特征表和普通数据表在同一个 catalog 中管理。
