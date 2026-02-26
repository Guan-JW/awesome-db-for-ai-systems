# [Layer 2 · Memory] Paper Notes: Runtime State, KV Cache & Coordination

> How do databases manage runtime state for AI systems — including KV cache, workflow checkpoints, and multi-agent coordination state?
> **三层定位**：中间层记忆系统——管理 AI 系统运行时的临时状态与中间计算结果。

---

## Key Themes

- **KV Cache Management**: LLM 推理时产生的 Key-Value 中间结果的存储、卸载与共享
- **Workflow State Persistence**: 多步 Agent 工作流的断点续跑、状态快照与时间旅行调试
- **Coordination State**: 多 Agent 间的共享状态、消息传递、并发一致性

---

## Open Problems

1. **KV cache offloading**: GPU 内存不够时，如何高效地将 KV cache 卸载到 CPU/SSD/远程存储？
2. **KV cache sharing**: 多用户共用同一段 system prompt 时，如何共享对应的 KV cache 避免重复计算？
3. **State consistency**: 多 Agent 并发写入工作流状态时，如何保证一致性？
4. **Checkpoint granularity**: 状态快照的粒度如何选择——太细浪费存储，太粗恢复不精确？

---

## Systems & Frameworks

### [LangGraph] Building Stateful, Multi-Agent Applications with LLMs

- **Authors**: LangChain Team
- **Code**: https://github.com/langchain-ai/langgraph

**Summary**:
> Graph-based workflow engine for multi-agent LLM apps. Each node is an agent; edges represent control flow. State is a typed dict persisted in a configurable DB (SQLite / PostgreSQL / Redis by default).

**DB / Storage Used**:
> Checkpointer: SQLite (default), PostgreSQL, or Redis. State snapshots enable time-travel debugging and fault tolerance.

**Key Contribution**:
> First major framework to make *persistent, queryable state* a first-class citizen — not an afterthought.

**与三层架构的关联**：LangGraph 是中间层"状态持久化"的核心案例——它证明了长时间运行的 Agent 系统需要数据库来保存工作流状态，支持崩溃恢复和时间旅行调试。

---

## Papers (待补充)

> - KV Cache 卸载/共享方面的系统论文（如 PagedAttention / vLLM、SGLang、Mooncake 等）
> - 分布式 Agent 状态管理论文
> - AI 推理服务中的状态管理与调度论文
