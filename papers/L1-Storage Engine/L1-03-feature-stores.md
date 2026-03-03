# [Layer 1 · Storage] Paper Notes: Feature Stores

> How do ML systems ensure data consistency between training and serving, and what database technologies power this "feature engineering layer"?
> **三层定位**：底层存储引擎——**特征存储**作为连接原始数据与模型推理的关键桥梁，解决了 AI 工程化落地中的 T-S Skew (Training-Serving Skew) 问题。

---

## Key Concepts

-   **Training-Serving Skew**: The critical problem where feature logic or data timing differs between model training (batch, offline) and interference serving (real-time, online), leading to degraded model performance.
-   **Point-in-Time Correctness (Time Travel)**: The ability to query feature values *exactly as they were* at a specific past timestamp to prevent data leakage during training set generation.
-   **Dual-Store Architecture**: The standard design pattern separating high-throughput batch storage (Offline Store) from low-latency key-value serving (Online Store).

---

## Papers / Systems

### [Feast] Feast: A Feature Store for Machine Learning

-   **Authors**: Feast Community (originally Gojek/Google Cloud)
-   **Project**: https://feast.dev/
-   **Code**: https://github.com/feast-dev/feast

**Summary**:
> The leading open-source feature store. Originally developed at Gojek, now a standard. Manages the ETL from raw data to online serving, ensuring consistency.

**Architecture**:
> **Dual-Store Design**:
> 1.  **Offline Store** (e.g., Snowflake, BigQuery, Parquet): Stores historical data for training. Optimized for scan throughput.
> 2.  **Online Store** (e.g., Redis, DynamoDB, Cassandra): Stores the latest feature values. Optimized for single-entity low-latency lookup (<10ms).
> 3.  **Registry**: Central catalog of feature definitions and metadata.

**Key Contribution**:
> **Point-in-Time Correctness**: Implements "AS OF JOIN" logic at scale. When generating training data, it joins label timestamps with feature values valid *at that exact second*, preventing future data from leaking into the past.
> **Materialization Engine**: Automatically syncs feature updates from the Offline Store (or streams) to the Online Store.

#### 阅读笔记

Feast 解决的核心痛点是"模型上线变傻"：离线训练时候用的 SQL 写得好好的，上线时候换成 Java/Go 重新实现一遍逻辑，结果写错了或者数据有延迟，导致线上效果大打折扣。 feature store 把这个逻辑固化下来，同一套定义，生成 offline dataframe 和 online api。

最厉害的是 Time Travel：做历史数据回测的时候，比如我要预测"上个月用户点没点广告"，当时的"用户最近7天点击率"必须是上个月那个时间点的状态，不能用了今天的。Feast 自动处理了这个 AS OF JOIN，在数据库层面非常繁琐，它给封装好了。

---

### [Tecton] Tecton: The Enterprise Feature Platform

-   **Authors**: Built by the team that created Uber Michelangelo
-   **Website**: https://www.tecton.ai/

**Summary**:
> Commercial evolution of the Feature Store concept. Adds "On-Demand Features" (calculated at request time) and complex streaming aggregations.

**Key Contribution**:
> **Feature Pipeline Orchestration**: Manages the entire DAG of feature transformations (batch, stream, real-time). The "DevOps for Data" approach.

---

## Database Perspective

From a database system view, a Feature Store is a specialized **Materialized View Manager**:
1.  **View Definition**: Users define features (views) in Python/DSL.
2.  **View Maintenance**: The system eagerly updates the Online Store (Materialized View) for low latency.
3.  **Consistency**: It guarantees eventual consistency between the Offline "source of truth" and the Online "cache".
4.  **Query Interface**: Supports both high-throughput scans (training) and point lookups (serving).

This bridges the gap between **AP (Analytical Processing)** and **TP (Transactional Processing/Serving)** in AI workloads.
