# Paper Notes: Text-to-SQL & Natural Language DB Interfaces

> How do AI systems translate natural language into executable SQL, and what does this reveal about AI-DB interaction challenges?

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

**解决什么问题**：Spider 等早期 Text-to-SQL 基准使用干净、小型的玩具数据库，与现实企业数据库严重脱节，导致高分模型在实际部署中大量失败。

**核心思路**：构建更真实的 benchmark，四项关键升级：
- **规模**：12,751 问答对，95 个真实数据库，总大小 33.4 GB，覆盖 37 个行业领域
- **脏数据**：数据库含 NULL、拼写错误、格式不一致等真实问题
- **外部知识**：正确答案需要领域知识（如"BMI > 30 算肥胖"），仅凭 schema 无法作答
- **新指标**：增加 SQL 执行效率评测（execution efficiency），不只看答案正确与否；引入 Value Linking 任务：将问题实体与数据库真实值关联

**实验结果**：ChatGPT（3.5）仅达 40.08% EX，而人类标注员达 92.96%——52 分差距揭示了巨大的能力鸿沟。GPT-4 当时约 50%，即使最强模型在真实 DB 场景下仍有大量失败。

**与本 survey 的关联**：BIRD 证明"Text-to-SQL = 数据库理解问题"，不只是 SQL 语法生成。AI agent 访问数据库需要理解 DB 的内容和领域知识，不能只靠 schema——正是本 survey 研究"AI 如何理解和使用 DB"的核心 motivation 之一。

**小结**：BIRD 证明 AI 在真实数据库面前几乎是文盲，ChatGPT 只有 40% 正确率，说明 agent 与数据库的交互远比我们想象的难。

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

**解决什么问题**：复杂 NL2SQL 任务（多表 JOIN、嵌套子查询、聚合运算）单一模型一次性生成很难准确，且出错后没有机制自动检测和修复。

**核心思路**：三个专职 agent 协同完成 Text-to-SQL，通过共享消息状态库（shared message DB）传递中间结果：
- **Selector agent** — schema 裁剪，从全量 schema 中选出与问题相关的子 schema，减少 LLM 负担
- **Decomposer agent** — 把复杂问题拆解为若干简单子问题，用 CoT 逐步生成 SQL 片段
- **Refiner agent** — 执行 SQL，若报错则分析错误日志并自动修复

共享消息库记录每步中间结果（类似 DB 的 staging table），保证各 agent 信息同步。同时开源 SQL-Llama（LLaMA-2 7B fine-tune），证明小模型也可接近 GPT-4 水平。

**实验结果**：基于 GPT-4 在 BIRD test set 达 59.59% EX（当时 SOTA）；SQL-Llama（7B）达 43.94%，是同参数量开源最优。消融实验：无 Decomposer 时下降约 3%，无 Refiner 时下降约 5%，每个 agent 均有明显贡献。

**与本 survey 的关联**：MAC-SQL 是多 agent 访问数据库的典型案例——agents 用 DB 协调自身（共享消息库），同时访问目标 DB（查询企业数据库）。这种双重 DB 角色正是本 survey 的核心研究场景之一。

**小结**：MAC-SQL 用三 agent + 共享状态库把 Text-to-SQL 正确率推到 60%，展示了 multi-agent 协同如何显著提升 AI 的 DB 访问能力。

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
