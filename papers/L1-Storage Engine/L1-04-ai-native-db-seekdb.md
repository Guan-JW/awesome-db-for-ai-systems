# [Cross-Cutting] OceanBase seekdb: AI-Native 统一数据库

> seekdb 是 OceanBase 推出的 AI 原生搜索数据库，将**关系型、向量、全文、JSON、GIS** 统一在单引擎内。
> 它同时覆盖三层架构——底层多模存储、中间层 Agent 记忆（PowerMem）与 RAG 管线（PowerRAG）、上层的 Fork Table 数据沙箱。
> 更特殊的一点是它的**云-边-端**三级部署能力（嵌入式/单机/分布式），这给具身智能、车载等场景带来了工程上的可行性。

---

## 一、定位：打通三层的"一体化 AI 数据库"

seekdb 跟我们 survey 里逐层讨论的那些专用系统（Milvus 只做向量、LangGraph 只管状态、vLLM 只管 KV Cache）思路不同，它试图在一个数据库里把存储 + 记忆 + 执行全包了。

对应到三层架构：

| 层           | seekdb 对应能力                                                 | 竞品对比                                      |
| :----------- | :-------------------------------------------------------------- | :-------------------------------------------- |
| L1 Storage   | 向量索引 (HNSW/IVF/DiskANN) + 全文索引 (BM25) + 关系存储 + JSON | Milvus (纯向量), pgvector (PG扩展)            |
| L2 Memory    | PowerMem (Agent 记忆存储) + PowerRAG (库内 RAG 管线)            | MemGPT (只管记忆), LangChain RAG (应用层拼装) |
| L3 Execution | Fork Table（数据分支/沙箱）+ AI Functions (库内推理)            | Neon branch (PG 层面), Chroma fork (向量层面) |

这种"全包"方案在工程上的好处很直接：少维护几个组件，ACID 一致性天然有保证，不用自己搞跨系统的数据同步。代价是每个方向的深度可能不如专用系统。

---

## 二、L1 层：多模存储与混合检索

### 混合检索架构

seekdb 的混合检索是在单条 SQL 里同时跑向量召回和关键词召回，然后在数据库内部做重排。支持三种重排策略：加权分数、RRF (Reciprocal Rank Fusion)、LLM 重排。

```sql
SET @parm = '{
    "query": { "query_string": { "fields": ["query","content"], "query": "hello" } },
    "knn":   { "field": "vector", "k": 3, "query_vector": [1,2,3] }
}';
SELECT json_pretty(DBMS_HYBRID_SEARCH.SEARCH('doc_table', @parm));
```

这跟 Elasticsearch 的做法类似，但 seekdb 多了完整的 ACID 事务和多表 JOIN 能力。对于 RAG 系统来说，"检索时能直接 JOIN 权限表做过滤"这件事其实非常实用，纯向量库做不到。

### 多模数据统一

一个表里可以同时有关系列、向量列、全文列、JSON 列、GIS 列，共用同一个存储引擎（LSM-Tree）。向量索引走 HNSW（内存）或 IVF-PQ（磁盘），全文索引走 BM25，标量过滤下压到存储层。

这个设计对多模态 AI 应用有直接意义——比如一个图文检索系统，图片向量、文字描述、结构化标签可以存在同一个表里，一条混合查询搞定，不用跨三个系统拼结果。

---

## 三、L2 层：PowerMem 与 PowerRAG

### PowerMem（Agent 记忆）

seekdb 内置了面向 Agent 的记忆存储模块。跟 MemGPT/Mem0 那样在应用层自己拼接不同，PowerMem 直接在数据库内核里做了记忆的存取和管理。记忆按实体维度组织，支持向量 + 全文混合检索历史记忆。

从数据库角度看，这相当于把 Mem0 的"向量库 + 图库 + KV 库"三件套合并成了一个带 MVCC 的统一存储。好处是记忆更新和查询天然有事务保证——不会出现"写了一半记忆，另一个 Agent 读到了不一致状态"这种问题。

### PowerRAG（库内 RAG）

传统 RAG 是在应用层完成"分块 → Embedding → 入库 → 检索 → 拼 prompt → 调 LLM"这个链条，seekdb 把整个 pipeline 拉进了数据库：

- `AI_EMBED(text)` 函数直接在 SQL 里做文本向量化
- `AI_COMPLETE(prompt)` 函数在 SQL 里调 LLM 做生成
- `AI_RERANK(results)` 函数做搜索结果重排

所谓"Doc In, Data Out"——喂进去一篇文档，数据库自动完成解析、分块、Embedding、入库，查询时一条 SQL 返回 RAG 结果。

这个思路跟传统 RDBMS 把存储过程内置类似，优点是减少了数据搬运和网络开销，缺点是跟具体 LLM 服务耦合了。

---

## 四、L3 层：Fork Table 数据沙箱

这是那篇微信推文的主角。

### 技术原理

Fork Table 实现了**数据库层面的 Git 分支**：基于某个时刻的一致性快照，毫秒级创建一个逻辑独立的分支表。

底层用的是 Copy-on-Write：
1. 前台：只复制元数据和表定义，记录 `fork_snapshot_scn`（分支诞生时刻），立即可查
2. 后台：利用 LSM-Tree 的特性，对已冻结的 SSTable 只增加 Macro Block 的引用计数（零拷贝），对混合新旧版本的 SSTable 通过快照迭代器重写
3. 隔离：逻辑隔离靠 `fork_snapshot_scn` 做绝对分割，物理隔离随 Compaction 自然完成

```sql
FORK TABLE production_features TO features_experiment_v2;
-- 在分支上做实验性修改...
-- 不影响源表
```

### 对 AI 系统的价值

这个特性在几个场景下有直接用处：
- **A/B 测试**：基于同一份生产数据瞬间创建两个分支，跑不同策略，结果隔离
- **特征工程版本化**：做完 feature engineering 后 Fork 一个快照表，后续训练锁定此表，实验可复现
- **Agent 工作区**：多 Agent 系统里每个 Agent Fork 自己的私有数据分支，避免记忆交叉污染
- **Vibe Coding 回滚**：AI 自动改数据前先 Fork，改坏了一键回到快照

从数据库研究角度看，Fork Table 是 Copy-on-Write 和 Snapshot Isolation 在 AI 数据工作流中的一个自然延伸。Neon（PostgreSQL 的 serverless 分支）和 Chroma（向量库）也有类似的 fork/branch 功能，但 seekdb 的事务一致性保证更强（完整的 ACID + LSM 快照）。

---

## 五、云-边-端部署与具身智能

seekdb 比较独特的一个卖点是部署弹性：

| 部署模式             | 资源需求   | 场景                 |
| :------------------- | :--------- | :------------------- |
| 嵌入式 (pip install) | 1C2G 可跑  | 端设备、机器人、车载 |
| 单机服务器           | 常规服务器 | 开发测试、中小规模   |
| 分布式 (OceanBase)   | 集群       | 大规模生产环境       |

对具身智能这种场景来说，机器人本地需要一个能跑向量检索 + 事务的轻量数据库，来管理感知数据、任务记忆、地图信息等。传统方案是 SQLite + FAISS 拼起来用，但没有多模混合搜索也没有内置 AI 能力。seekdb 嵌入式模式能在 1C2G 上跑起来，同时支持向量、全文、JSON，并且跟云端 OceanBase 分布式版本 schema 兼容，边端的数据可以同步上云，这个"端云一体"的思路在目前的竞品里比较少见。

---

## 六、跟 survey 的关系

seekdb 在我们 survey 中扮演的角色比较特别——它不是某一层的代表，而是**"统一 AI 数据库"这条技术路线的代表案例**。

可以在几个地方引用：

1. **第 3 章（Vector DB）**：作为"关系型数据库扩展向量能力"路线的案例，跟 pgvector 放在一起比较。seekdb 的差异点在于原生混合检索和库内 AI 函数。
2. **第 5 章（Agent Memory）**：PowerMem 可以作为"数据库原生记忆方案"的案例，跟 Mem0/MemGPT 的应用层方案做对比。
3. **第 10 章（AI-Native DB）**：Fork Table 和 AI Functions 是"AI-Native 数据库设计"最直接的例证。
4. **云-边-端讨论**：survey 目前完全没覆盖边缘部署，这块可以新开一个小节。

### 值得思考的问题

- 一体化方案在工程上简化了部署，但在学术上是否有独特的技术贡献？目前看到的更多是"把已有技术（HNSW、BM25、CoW）组合进一个引擎"，还没看到特别新的算法层创新。
- Fork Table 的 CoW 思路不算新（Btrfs、ZFS 都用），但应用到 AI 数据沙箱这个场景确实贴合当下需求。
- 嵌入式部署对具身智能有意义，但 1C2G 上能跑多大的向量库、延迟表现如何，目前公开数据不多，后续可以关注 benchmark。

---

## 参考来源

- OceanBase seekdb 官方文档：https://www.oceanbase.ai/docs/seekdb-overview/
- seekdb GitHub：https://github.com/oceanbase/seekdb
- Fork Table 推文：https://mp.weixin.qq.com/s/hWaS50RDu6HbNvz8csD6tQ
- Fork Table 文档：https://www.oceanbase.ai/docs/zh-CN/fork-table-overview
