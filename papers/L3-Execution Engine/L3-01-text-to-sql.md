# [Layer 3 · Execution] Paper Notes: DB Agent — Text-to-SQL & NL Interfaces

> How do AI systems translate natural language into executable SQL, and what does this reveal about AI-DB interaction challenges?
> **三层定位**：上层执行引擎——数据库作为 AI 应用直接操作的对象，DB Agent 通过自然语言与数据库交互。

---

## Benchmarks

### [Spider] Spider: A Large-Scale Human-Labeled Dataset for Complex and Cross-Domain Semantic Parsing and Text-to-SQL Task

- **Authors**: Yu et al. (Yale)
- **Venue**: EMNLP 2018
- **PDF**: https://arxiv.org/abs/1809.08887
- **Code**: https://github.com/taoyds/spider

**Summary**:
> 10,181 questions across 200 databases with complex, cross-domain SQL. Established the standard benchmark for NL2SQL for 5+ years.

**Key Contribution**:
> Defined the evaluation framework (exact match accuracy, execution accuracy) used across all subsequent NL2SQL work.

---

### [BIRD] Can LLM Already Serve as A Database Interface? A BIg Bench for Large-Scale Database Grounded Text-to-SQLs

- **Authors**: Li et al. (HKUST)
- **Venue**: NeurIPS 2023
- **PDF**: https://arxiv.org/abs/2305.03111
- **Code**: https://github.com/AlibabaResearch/DAMO-ConvAI/tree/main/bird

**Summary**:
> 12,751 question-SQL pairs over large (up to 33.4 GB) real-world databases with dirty data. Introduces "execution efficiency" as an evaluation metric. Much harder than Spider.

**Key Contribution**:
> Reflects real enterprise DB challenges: external knowledge needed, noisy data, efficiency constraints.

#### 阅读笔记

NeurIPS 2023 的 benchmark。跟 Spider 比最大的区别是"真实"：95 个真实数据库共 33.4 GB，数据里有 NULL、拼写错误、格式不一致。而且部分问题需要领域知识才能答对，比如"BMI>30 算肥胖"这种信息在 schema 里是找不到的。

结果相当惊人：ChatGPT 只有 40.08%，人类标注员 92.96%。差了 52 分。GPT-4 当时也就 50% 左右。说明 AI 在面对真实脏数据库的时候能力还差得远。

这篇对理解我们 survey 的 motivation 很重要：Text-to-SQL 不只是语法翻译问题，AI 需要理解数据库的内容和业务语义。可以放第 8 章讨论 DB Agent 面临的挑战时引用。

---

### [Spider 2.0] Spider 2.0: Evaluating Language Agents on Real-World Enterprise Text-to-SQL Workflows

- **Authors**: Lei et al.
- **Venue**: arXiv 2024
- **PDF**: https://arxiv.org/abs/2411.07763
- **Code**: https://github.com/xlang-ai/spider2

**Summary**:
> Enterprise-scale benchmark: 631 workflows requiring multi-step SQL across BigQuery, Snowflake, DuckDB. Tasks require 10-100 SQL steps. Current best model achieves <20% success rate.

**Key Contribution**:
> Represents the frontier: DB agents in real enterprise settings with multi-step planning, credential management, and real cloud warehouses.

---

## LLM-based Methods

### [DIN-SQL] DIN-SQL: Decomposed In-Context Learning of Text-to-SQL with Self-Correction

- **Authors**: Pourreza & Rafiei (University of Alberta)
- **Venue**: NeurIPS 2023
- **PDF**: https://arxiv.org/abs/2304.11015
- **Code**: https://github.com/MohammadrezaPourreza/Few-shot-NL2SQL-with-prompting

**Summary**:
> Decomposes NL2SQL into 4 sub-tasks (schema linking, classification, SQL generation, self-correction), each handled by a separate GPT-4 prompt. Achieves 82.8% on Spider with GPT-4.

**Key Contribution**:
> Established decomposition as the key principle for LLM-based Text-to-SQL.

---

### [MAC-SQL] MAC-SQL: A Multi-Agent Collaborative Framework for Text-to-SQL

- **Authors**: Wang et al. (Tencent)
- **Venue**: COLING 2025
- **PDF**: https://arxiv.org/abs/2312.11242
- **Code**: https://github.com/wbbeyourself/MAC-SQL

**Summary**:
> Three specialized agents: Selector (schema linking), Decomposer (breaks complex questions), Refiner (fixes SQL errors). Agents communicate via a shared state/message database.

**Key Contribution**:
> Demonstrates multi-agent architecture for DB interfaces; the shared message store acts as coordination DB.

#### 阅读笔记

腾讯的工作，三个 agent 分工合作：Selector 做 schema 裁剪（从全量 schema 里选相关子集），Decomposer 把复杂问题拆成子问题逐步生成 SQL，Refiner 执行 SQL、分析报错、自动修复。三者通过一个共享消息库传递中间状态。

基于 GPT-4 在 BIRD test set 上达到 59.59%（当时排第一），消融实验显示每个 agent 都有贡献（去掉 Refiner 下降 ~5%，去掉 Decomposer 下降 ~3%）。他们还开源了 SQL-Llama（LLaMA-2 7B fine-tune），7B 模型做到 43.94%，小模型里最好的。

对 survey 比较有启发的一点是：这三个 agent 同时在"用"数据库（查企业数据）和"靠"数据库做协调（共享消息库充当了 staging table 的角色）。数据库在这里扮演了双重角色，第 8、9 章都可以引用。

---

### [DAIL-SQL] Efficient In-Context Learning for Text-to-SQL

- **Authors**: Gao et al. (Alibaba DAMO)
- **Venue**: VLDB 2024
- **PDF**: https://arxiv.org/abs/2308.15363
- **Code**: https://github.com/BeachWang/DAIL-SQL

**Summary**:
> Systematic study of example selection and masking strategies for few-shot Text-to-SQL. Achieves 83.6% on Spider test set.

**Key Contribution**:
> Shows the critical impact of *how* examples are stored and retrieved from the few-shot example database.

---

---

## Agentic Search vs. DB Agent：读写光谱上的位置

> **来源**：Weaviate Blog "Building A Legal RAG App in 36 Hours" (2026-02-26)
> https://weaviate.io/blog/legal-rag-app

这篇博客有个概念比较有启发：**Agentic Search**。跟我们讨论的 DB Agent（Text-to-SQL、MAC-SQL 这些）不太一样。

### 区别在哪

| 维度     | Agentic Search                                                 | DB Agent                                                 |
| :------- | :------------------------------------------------------------- | :------------------------------------------------------- |
| 操作权限 | **只读**：只做检索、过滤、重排                                 | **读写**：可以写数据、改 schema、管理索引                |
| 推理目标 | 决定**怎么搜**（选哪几个集合、用什么过滤条件、要不要拆子查询） | 决定**执行什么操作**（生成 SQL/DML、做优化、跑管理命令） |
| 交互对象 | 主要跟向量库/搜索引擎交互                                      | 主要跟关系型数据库交互                                   |
| 风险等级 | 低（最多查错了）                                               | 高（写错了可能破坏数据）                                 |
| 典型系统 | Weaviate Query Agent, Azure AI Search Agentic Retrieval        | DIN-SQL, MAC-SQL, Spider 2.0 Agent                       |

Weaviate 的 Query Agent 具体做了这些事：
1. **Schema 感知**：自动检查可用的 collection 和字段定义，决定查哪个 collection
2. **查询拆解**：把复杂问题拆成多个子查询（比如"2024年服务协议里的通知期限是多少"会加上日期过滤）
3. **结构化过滤**：构建精确的 filter + aggregation，不只靠向量相似度
4. **重排**：用 Rerank Sub-agent 对召回结果按实际相关性重排
5. **答案合成**：用 Answer Sub-agent 基于筛选后的上下文生成答案

两种模式：Search Mode（发现式检索，返回来源让用户自己看）和 Ask Mode（直接给出有来源引用的答案）。

### 为什么这个区别对 survey 重要

这其实划出了一条清晰的分界线：**数据库在 AI 系统中是「被查询」还是「被操控」**。

- Agentic Search 把数据库当作一组**工具**（tool），agent 只决定检索策略，不动数据本身。
- DB Agent 把数据库当作一个**被管理的对象**，agent 不仅查数据还能改数据。

从数据库角度看，这两种模式对存储层的要求完全不同：
- Agentic Search 需要丰富的元数据（schema、collection 信息、字段描述）来辅助查询规划，但不需要 DML 能力
- DB Agent 需要事务支持、权限控制、操作回滚——因为写操作不可逆

这个读写光谱在我们 survey 里可以作为第 8 章开头的分类框架。

### 多模态方面的亮点

Weaviate 这个 legal app 用了多向量模型（ColModernVBERT）直接把 PDF 页面编码成视觉 token——不走 OCR，不切文本 chunk，直接把每页 PDF 当图片做 multivector embedding。然后用 Muvera 压缩这些多向量表示，降内存降延迟。

这是一个非常具体的**多模态数据入库方式**：文档不再是"先 OCR 提文字、再做 embedding"，而是"保留原始视觉布局、直接做图像级向量化"。对于法律文档这种格式复杂的场景（表格、条款编号、缩进层级），视觉编码显然比 OCR+chunking 保留了更多信息。

数据库层面的启发：向量库需要支持 multivector（一个文档对应多个向量而不是一个），并且需要高效的 multivector 压缩和检索。Weaviate 用 Muvera 做的压缩（ksim=4, dprojections=16, repetitions=20）是这个方向的实际工程方案。

---

## 补充资料：行业实践与技术博客

- Spider Leaderboard：https://yale-lily.github.io/spider  
  所有做 Text-to-SQL 的人都会盯的排行榜。可以快速看到当前 SOTA 是多少、哪些方法排在前面。不过注意 Spider 1.0 上的分数已经比较卷了（>90%），现实中的 Text-to-SQL 远没有这么高，看 BIRD 的榜单更有参考价值。

- **Defog.ai 的博客和开源模型**：https://defog.ai/blog/  
  做开源 Text-to-SQL 的初创公司。他们开源了 SQLCoder（基于 Code Llama fine-tune），配套的博客记录了做数据、调模型、处理 schema 信息的各种实战经验。比如怎么处理几百张表的 schema 塞不进 context window 的问题，写得挺实在的。

- 知乎和公众号上搜"Text-to-SQL 落地"能看到不少来自甲方和乙方的吐槽。反复出现的问题有：用户问的问题太模糊 LLM 瞎猜、schema 注释缺失导致选错表、多表 JOIN 生成的 SQL 慢得离谱、安全性问题（用户是否能通过自然语言注入执行恶意 SQL）……基本上都是论文里不怎么讨论但实际很头疼的问题。

- BIRD 排行榜（https://bird-bench.github.io/ ）比 Spider 更接近真实场景，目前最好的方法也就 70% 左右。排行榜页面还有各种模型的详细对比，包括不同难度级别和不同数据库引擎上的表现。

- Vanna（https://github.com/vanna-ai/vanna ）是一个比较火的开源 Text-to-SQL 项目，思路是先让用户提供一批 question-SQL 样例训练 RAG 检索，推理时检索相似样例做 few-shot。代码简洁，几分钟就能跑起来一个 demo，适合想快速搭个原型的同学。
