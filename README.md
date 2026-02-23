# Awesome Databases for AI Systems

> A curated survey of database systems, design patterns, and architectural practices in modern AI systems including multi-agent frameworks, RAG pipelines, and LLM applications.

## 🎯 About This Project

This repository serves as a comprehensive resource collection exploring the evolving role of databases in AI-native systems. As AI applications grow in complexity—from simple chatbots to sophisticated multi-agent systems—the database layer has become a critical component that goes far beyond traditional data storage.

**Research Focus:**
- How do databases enable coordination in multi-agent systems?
- What are the emerging database patterns for RAG (Retrieval-Augmented Generation)?
- How do vector databases integrate with traditional databases in AI architectures?
- What role do DB Agents play in AI system architectures?
- What are the performance and consistency trade-offs in AI-native data systems?

## 📚 Table of Contents

- [📄 Papers](#-papers)
- [🚀 Open Source Projects](#-open-source-projects)
- [🏢 Industry Practices](#-industry-practices)
- [🗄️ Database Categories](#️-database-categories)
- [🏗️ Design Patterns](#️-design-patterns)
- [💡 Use Cases](#-use-cases)
- [🤝 Contributing](#-contributing)

---

## 📄 Papers

### Multi-Agent Systems & Databases

*Coming soon - Research papers on database roles in multi-agent coordination, state management, and agent communication.*

### RAG & Vector Databases

*Coming soon - Papers on vector database architectures, hybrid search, and retrieval systems.*

### LLM Applications & Data Management

*Coming soon - Research on memory systems, context management, and data flows in LLM apps.*

### DB Agent & Natural Language Interfaces

```
NLIDB / Text-to-SQL 发展脉络

I. 基准数据与任务定义层（Benchmark Era）
   ├── 单轮跨库泛化
   │   └── Spider (2018)
   ├── 多轮上下文建模
   │   ├── SParC (2019)
   │   └── CoSQL (2019)
   ├── 多语言扩展
   │   └── MultiSpider (2023)
   └── 真实数据库执行场景
       └── BIRD (2023)

II. 结构建模与解码约束层（Neural Parsing Era）
   ├── Schema-aware 编码
   │   └── RAT-SQL (2020)
   └── 语法约束解码
       └── PICARD (2021)

III. LLM增强与可扩展系统层（LLM Scaling Era）
   ├── 大规模Schema路由
   │   └── DBCopilot (2025)
   ├── 交互式多轮LLM框架
   │   └── Interactive-T2S (2025)
   └── Agentic强化优化
       └── AGRO-SQL (2025)

IV. Agent化数据库系统层（Database Agent Era）
   ├── 端到端数据库Agent
   │   └── AskDB (2025)
   ├── 混合轻量+大模型Agent
   │   └── Hybrid Agentic System (2025)
   └── 自动特征工程Agent
       └── ReFuGe (2026)

V. 评测、生成与安全层（Evaluation & Security）
   ├── 自动对话生成与评测
   │   ├── Automated Evaluation of DB Agents (2025)
   │   └── Agent-based Dialogue Generation (2025)
   └── 安全性与SQL注入攻击
       └── SIGMOD 2026 Security Paper

VI. 范式抽象与理论层（Data Agent Abstraction）
   ├── NLIDB系统综述
   │   └── NLI4DB (2024)
   ├── Data Agent综述
   │   ├── Survey of Data Agents (2025)
   │   └── Data Agents: Levels (2026)
   └── 生成式数据库管理员
       └── Gen-DBA (2026)
```

- **Spider: A Large-Scale Human-Labeled Dataset for Complex and Cross-Domain Semantic Parsing and Text-to-SQL Task**. *EMNLP*, 2018.
  - **Brief summary:** Foundational cross-domain Text-to-SQL benchmark introducing complex multi-table SQL over unseen databases. Contains 10k+ questions across 200 databases with disjoint train/test schemas to evaluate schema generalization and compositional reasoning. It established the modern evaluation paradigm for NL→SQL systems.
  - **Link:** https://arxiv.org/abs/1809.08887
- **SParC: Cross-Domain Semantic Parsing in Context**. *ACL*, 2019.
  - **Brief summary:** Extends Spider to multi-turn conversational Text-to-SQL. Evaluates context tracking, dialogue state modeling, and sequential query refinement across turns, forming an early benchmark for conversational NLIDB systems.
  - **Link:** https://aclanthology.org/P19-1443/
- **CoSQL: A Conversational Text-to-SQL Challenge**. *EMNLP*, 2019.
  - **Brief summary:** Introduces a human-in-the-loop conversational benchmark where users and systems interact to iteratively construct SQL queries. Emphasizes clarification, correction, and dialogue-based query refinement.
  - **Link:** https://aclanthology.org/D19-1204/

* **MultiSpider: Multilingual Text-to-SQL Semantic Parsing Benchmark**. *AAAI*, 2023.

  - **Brief summary:** Multilingual extension of Spider covering seven languages. Evaluates cross-lingual generalization and multilingual NL→SQL transfer, expanding NLIDB research beyond English-only settings.

  - **Link:** https://doi.org/10.1609/aaai.v37i11.26499

- **BIRD: A Big Bench for Large-Scale Database-Grounded Text-to-SQL Evaluation**. *NeurIPS/Arxiv Preprint*, 2023.
  - **Brief summary:** Enterprise-scale benchmark emphasizing real database values, noisy data, and execution grounding. Moves beyond schema-only reasoning to value-aware SQL generation, reflecting realistic NLIDB deployment scenarios.
  - **Link:** https://bird-bench.github.io/

------

- **RAT-SQL: Relation-Aware Schema Encoding and Linking for Text-to-SQL Parsers**. *ACL*, 2020.
  - **Brief summary:** Proposes relation-aware self-attention to jointly encode schema graphs and natural language queries, significantly improving schema linking and cross-domain generalization on Spider. Influenced many structural encoding approaches in neural Text-to-SQL.
  - **Link:** https://arxiv.org/abs/1911.04942
- **PICARD: Parsing Incrementally for Constrained Auto-Regressive Decoding from Language Models**. *EMNLP*, 2021.
  - **Brief summary:** Introduces grammar-constrained decoding for LLM-based SQL generation, ensuring syntactic validity during autoregressive generation. Substantially improves execution accuracy and robustness on Spider-style benchmarks.
  - **Link:** https://aclanthology.org/2021.emnlp-main.779/

------

- **DBCopilot: Natural Language Querying over Massive Databases via Schema Routing**. *EDBT*, 2025.
  - **Brief summary:** Proposes a schema-routing framework to scale Text-to-SQL over large enterprise databases. Uses lightweight models for schema pruning and large LLMs for SQL synthesis, improving efficiency and scalability in real-world deployments.
  - **Link:** [arXiv](https://arxiv.org/abs/2312.03463)
- **Interactive-T2S: Multi-Turn Interactions for Text-to-SQL with Large Language Models**. *CIKM*, 2025.
  - **Brief summary:** Develops an interactive multi-turn Text-to-SQL framework with clarification strategies and dialogue state tracking. Enhances robustness for ambiguous or underspecified user queries.
  - **Link:** [ACM](https://dl.acm.org/doi/abs/10.1145/3746252.3761052)
- **AGRO-SQL: Agentic Group-Relative Optimization with High-Fidelity Data Synthesis**. *arXiv*, 2025.
  - **Brief summary:** Introduces an agentic reinforcement learning framework for Text-to-SQL combining high-fidelity synthetic data and Group-Relative Policy Optimization (GRPO). Improves compositional reasoning and generalization on complex SQL generation tasks.
  - **Link:** [arXiv](https://arxiv.org/abs/2512.23366)

------


- **AskDB: An LLM Agent for Natural Language Interaction with Relational Databases**. *arXiv*, 2025.
  - **Brief summary:** End-to-end LLM database agent supporting schema parsing, SQL generation, execution grounding, verification, and natural language explanation. Emphasizes execution-aware reasoning and unified pipeline design.
  - **Link:** [arXiv](https://arxiv.org/abs/2511.16131)
- **Hybrid Agentic System for Schema-Aware NL2SQL Generation**. *Cornerstone*, 2025.
  - **Brief summary:** Designs a hybrid architecture combining lightweight models for routine queries and large LLM agents for complex cases. Incorporates schema-aware distillation to balance efficiency and accuracy in production NL2SQL systems.
  - **Link:** [Cornerstone](https://cornerstone.lib.mnsu.edu/etds/1559/)
- **ReFuGe: Feature Generation for Prediction Tasks on Relational Databases with LLM Agents**. *arXiv*, 2026.
  - **Brief summary:** Proposes a collaborative multi-agent framework for automated feature engineering over relational databases, bridging NL-based database interaction and downstream predictive modeling tasks.
  - **Link:** [arXiv](https://arxiv.org/abs/2601.17735)

-----

- **Automated Evaluation of Database Conversational Agents**. *SciTePress*, 2025.
  - **Brief summary:** Proposes automated evaluation pipelines using synthetic test construction and evaluator agents to assess conversational robustness, query correctness, and dialogue consistency in LLM-based DB agents.
  - **Link:** [SciTePress](https://www.scitepress.org/Link.aspx?doi=10.5220/0013732900003985)
- **Automation of the Generation and Evaluation of Dialogues for Text-to-SQL Systems: An Agent-based Approach**. *PUC-Rio*, 2025.
  - **Brief summary:** Uses a dual-agent framework to automatically generate and evaluate multi-turn Text-to-SQL dialogues, addressing dataset scarcity and improving evaluation scalability.
  - **Link:** [PUC-Rio](https://www.maxwell.vrac.puc-rio.br/74368/74368.PDF)
- **Are Your LLM-based Text-to-SQL Models Secure? Exploring SQL Injection via Backdoor Attacks**. *SIGMOD*, 2026.
  - **Brief summary:** Analyzes security vulnerabilities in LLM-based Text-to-SQL systems, demonstrating susceptibility to SQL injection and backdoor attacks. Proposes mitigation strategies for secure deployment in production databases.
  - **Link:** [arXiv](https://arxiv.org/abs/2503.05445)

------

* **NLI4DB: A Systematic Review of Natural Language Interfaces for Databases**. Mengyi Liu, Jianqiu Xu. *Preprint*, 2024.

  - **Brief summary:** Comprehensive survey of NLIDB systems. Decomposes NL→query pipelines into preprocessing, semantic understanding, and translation. Covers rule-based, neural, and LLM-based approaches, benchmarks, evaluation metrics, and emerging paradigms such as Text-to-SQL, SQL2Text, and Speech2SQL.

  - **Link:** [arXiv](https://arxiv.org/abs/2503.02435)


- **A Survey of Data Agents: Emerging Paradigm or Overstated Hype?**. *arXiv*, 2025.
  - **Brief summary:** Surveys LLM-based data agents interacting with data systems through tool use and orchestration. Discusses architectural patterns, evaluation gaps, and limitations when integrating agents with database infrastructures.
  - **Link:** [arXiv](https://arxiv.org/abs/2510.23587)
- **Data Agents: Levels, State of the Art, and Open Problems**. *arXiv (SIGMOD 2026 Tutorial)*, 2026.
  - **Brief summary:** Proposes a six-level taxonomy of data agents spanning query answering, analytics, reasoning, orchestration, and autonomous decision-making. Frames open research problems for database-integrated LLM agents.
  - **Link:** [arXiv](https://arxiv.org/abs/2602.04261v1)
- **Gen-DBA: Generative Database Agents (Towards a Move 37 for Databases)**. *arXiv*, 2026.
  - **Brief summary:** Explores generative database agents capable of producing optimization strategies and tuning plans using transformer-based models and staged training. Extends agent concepts beyond query answering into database administration.
  - **Link:** [arXiv](https://arxiv.org/abs/2601.16409)

### Database Systems for AI

*Coming soon - Papers on AI-optimized databases, query systems, and storage engines.*

---

## 🚀 Open Source Projects

### Multi-Agent Frameworks

| Project | Database Used | Role | Stars | Notes |
|---------|---------------|------|-------|-------|
| *To be added* | | | | |

### RAG Frameworks

| Project | Database Used | Role | Stars | Notes |
|---------|---------------|------|-------|-------|
| *To be added* | | | | |

### Agent Memory Systems

| Project | Database Used | Role | Stars | Notes |
|---------|---------------|------|-------|-------|
| *To be added* | | | | |

### DB Agent & Text-to-SQL Systems

| Project | Database Support | Role in AI System | Stars | Notes |
|---------|------------------|-------------------|-------|-------|
| *To be added* | | | | |

### Vector Database Projects

| Project | Type | Primary Use Case | Stars | Notes |
|---------|------|------------------|-------|-------|
| *To be added* | | | | |

---

## 🏢 Industry Practices

### Case Studies

*Coming soon - Real-world implementations from companies building AI systems.*

### Architecture Patterns

*Coming soon - Common architectural patterns and their database implications.*

### Best Practices

*Coming soon - Lessons learned from production AI systems.*

---

## 🗄️ Database Categories

### Vector Databases
- **Specialized Vector DBs**: Pinecone, Weaviate, Qdrant, Milvus, Chroma
- **Vector Extensions**: pgvector, Redis Vector Search, Elasticsearch kNN

### Traditional Databases in AI Systems
- **Relational**: PostgreSQL, MySQL (with AI extensions)
- **NoSQL**: MongoDB, Redis, Cassandra
- **Graph**: Neo4j, TigerGraph

### Hybrid & AI-Native Systems
- Systems combining vector + relational + graph capabilities

---

## 🏗️ Design Patterns

### Agent State Management
*Coming soon - Patterns for managing agent state, conversation history, and context.*

### Memory Hierarchies
*Coming soon - Short-term vs long-term memory, caching strategies.*

### Multi-Agent Coordination
*Coming soon - Shared state, message passing, event sourcing patterns.*

### DB Agent as Component
*Coming soon - Integrating database agents (Text-to-SQL, NL2SQL) into multi-agent systems, their interaction patterns with other agents, and use cases in data-driven applications.*

### RAG Data Pipelines
*Coming soon - Ingestion, chunking, embedding, indexing, retrieval patterns.*

---

## 💡 Use Cases

### By Application Type
- **Chatbots & Conversational AI**
- **Multi-Agent Systems**
- **Code Assistants**
- **Research & Analysis Tools**
- **Creative AI Applications**

### By Database Role
- **Vector Search & Semantic Retrieval**
- **State & Session Management**
- **Agent Communication & Coordination**
- **Knowledge Graph & Reasoning**
- **Monitoring & Observability**

---

## 🤝 Contributing

We welcome contributions! This is an evolving survey and we encourage:

- **Papers**: Recent research on databases in AI systems
- **Projects**: Open source implementations with interesting database usage
- **Case Studies**: Production experiences and lessons learned
- **Patterns**: Novel design patterns or architectural insights

### How to Contribute

1. Fork this repository
2. Add your content following the existing format
3. Submit a pull request with a clear description
4. Ensure links are accessible and descriptions are accurate

### Submission Guidelines

- **Papers**: Include title, authors, venue, year, and brief summary
- **Projects**: Include GitHub link, star count, primary database used, and role description
- **Case Studies**: Provide context, architecture overview, and key takeaways

---

<!-- ## 📖 Citation

If you find this resource useful for your research, please consider citing:

```bibtex
@misc{awesome-db-for-ai-systems,
  title={Awesome Databases for AI Systems},
  author={[Jiawei Guan, Feng Zhang, Jiayi Li, Xueping Yang, Xiaoyong Du]},
  year={2026},
  howpublished={\url{https://github.com/Guan-JW/awesome-db-for-ai-systems}}
}
```

--- -->

## 📬 Contact & Discussion

- **Issues**: For suggestions or corrections, please open an issue

---

*This is a living document. Star and watch this repository for updates.*
