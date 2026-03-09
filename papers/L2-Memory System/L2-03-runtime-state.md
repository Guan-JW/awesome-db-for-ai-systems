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
5. **Auto Memory Flush**: 当 context window 即将溢出时，如何自动将关键上下文持久化？OpenClaw 的做法是在会话压缩前触发一个静默 agent turn——跟 vLLM PagedAttention 的 swap 逻辑异曲同工。

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

## Papers

### [vLLM / PagedAttention] Efficient Memory Management for Large Language Model Serving with PagedAttention

- **Authors**: Woosuk Kwon, Zhuohan Li, Siyuan Zhuang, Ying Sheng, Lianmin Zheng et al. (UC Berkeley)
- **Venue**: SOSP 2023
- **PDF**: https://arxiv.org/abs/2309.06180
- **Code**: https://github.com/vllm-project/vllm

**Summary**:
> Proposes PagedAttention, an attention algorithm inspired by virtual memory paging in OS. KV cache is stored in non-contiguous physical blocks, mapped via a page table. Eliminates memory fragmentation and enables flexible KV cache sharing within and across requests.

**DB / Storage Used**:
> KV cache stored in GPU memory as paged blocks; block manager acts as a memory allocator (analogous to OS virtual memory manager). Supports copy-on-write for shared prefixes.

**Key Contribution**:
> Near-zero KV cache memory waste; 2-4× throughput improvement over FasterTransformer and Orca. De-facto standard for LLM serving.

#### 阅读笔记

SOSP 2023 best paper，LLM serving 领域影响力最大的一篇。

问题很直接：LLM 推理时每个请求要维护一份 KV Cache，大小随生成 token 数动态增长。传统做法预分配连续内存块，结果碎片特别严重，显存利用率经常只有 20-40%。

PagedAttention 照搬了 OS 虚拟内存的思路：把 KV Cache 切成固定大小的 block（page），用页表映射，不要求物理连续。还支持 copy-on-write，多个请求共用同一段 system prompt 时前缀 KV Cache 可以共享，修改时才拷贝。

在 LLaMA-13B 和 OPT-175B 上测试，吞吐量比 FasterTransformer 高 2-4 倍。现在几乎所有开源 LLM 推理引擎都在用 vLLM 或者参考了它的内存管理方式。

这篇对我们 survey 第 7 章是核心参考。分页、页表、copy-on-write 这些全是 DB/OS 的老概念，但拿到 GPU 上管 KV Cache 效果非常好，是跨领域技术迁移的典型。

---

### [SGLang] SGLang: Efficient Execution of Structured Language Model Programs

- **Authors**: Lianmin Zheng, Liangsheng Yin, Zhiqiang Xie et al. (UC Berkeley / Stanford)
- **Venue**: arXiv 2023 (NSDI 2025)
- **PDF**: https://arxiv.org/abs/2312.07104
- **Code**: https://github.com/sgl-project/sglang

**Summary**:
> A system for efficient execution of complex LLM programs (multi-turn chat, agent control, structured output). Introduces RadixAttention for automatic KV cache reuse across requests sharing common prefixes, using a radix tree (trie) data structure.

**DB / Storage Used**:
> Radix tree (前缀树) as an in-memory index over KV cache blocks. Enables O(prefix_length) lookup for cache reuse.

**Key Contribution**:
> RadixAttention enables automatic, fine-grained KV cache sharing; up to 6.4× higher throughput than vLLM on multi-call LLM programs.

#### 阅读笔记

vLLM 解决了单请求的内存碎片问题，SGLang 进一步解决了多请求之间的 KV Cache 复用问题。

实际 agent 应用里经常出现这种模式：每次调用 LLM 都带同一段 system prompt 和 few-shot examples，只有最后的用户输入不同。SGLang 用 radix tree（基数树）来索引已经算好的 KV Cache，新请求来了先沿树查最长匹配前缀，命中的部分直接复用。树本身还负责缓存的插入和驱逐。

在 agent 控制、few-shot learning、RAG 管线、多轮对话等场景上比 vLLM 快 2.1-6.4 倍。

从数据库角度看，radix tree 就是文件系统和内存数据库里常用的索引结构，拿来管 KV Cache 的复用其实很自然。在 survey 第 7 章可以用这个例子说明传统索引结构在 AI 推理场景中的新应用。

---

### [Mooncake] Mooncake: A KVCache-centric Disaggregated Architecture for LLM Serving

- **Authors**: Ruoyu Qin, Zheming Li, Weiran He, Mingxing Zhang et al. (Moonshot AI / Tsinghua)
- **Venue**: arXiv 2024
- **PDF**: https://arxiv.org/abs/2407.00079
- **Code**: *(内部系统，Kimi 的线上推理平台)*

**Summary**:
> Production serving platform for Kimi (月之暗面). Separates prefill and decoding into different clusters, and builds a disaggregated KV cache pool using underutilized CPU/DRAM/SSD resources of the GPU cluster. A KVCache-centric scheduler balances throughput and latency SLOs.

**DB / Storage Used**:
> Disaggregated KV cache pool: GPU HBM → CPU DRAM → SSD, with a centralized scheduler. Essentially a distributed, tiered cache store.

**Key Contribution**:
> First production system treating KV cache as a first-class distributed storage resource rather than ephemeral GPU memory. Up to 525% throughput increase over baseline in long-context scenarios.

#### 阅读笔记

月之暗面（Kimi 团队）的生产级系统论文。面对的现实是：Kimi 的长上下文（几十万 token）负载非常重，单个请求的 KV Cache 就可能吃掉一整块 GPU 的显存。

核心做法是 prefill 和 decode 分离部署到不同 GPU 集群。中间用一个分布式 KV Cache 存储池来传递中间结果，这个池由 GPU HBM → CPU DRAM → SSD 三层组成，还有集中式调度器根据延迟要求决定 KV Cache 放哪层。过载时还会提前拒绝无法满足 SLO 的请求。

论文报告模拟负载下吞吐提升 525%，真实 Kimi 负载下多处理 75% 的请求。长上下文（>100K token）场景优势更明显。

说白了这套系统就是一个"分布式缓存数据库"：分层存储、缓存替换策略、数据分片、全局调度，跟做了几十年的数据库 buffer pool 管理很像。survey 第 7 章讲 KV Cache 管理系统时可以重点讨论。

---

### [CacheGen] CacheGen: KV Cache Compression and Streaming for Fast Large Language Model Serving

- **Authors**: Yuhan Liu, Hanchen Li et al. (University of Chicago)
- **Venue**: SIGCOMM 2024
- **PDF**: https://arxiv.org/abs/2310.07240
- **Code**: https://github.com/UChi-JCL/CacheGen

**Summary**:
> Compresses KV cache into compact bitstream representations using a custom tensor encoder that exploits the distributional properties of KV tensors. Adapts compression level dynamically based on available bandwidth to balance latency and quality.

**DB / Storage Used**:
> KV cache stored as compressed bitstreams on remote storage; streamed to GPU on demand. Essentially a columnar compression scheme for tensor data.

**Key Contribution**:
> 3.5-4.3× KV cache size reduction; 3.2-3.7× faster context loading with negligible quality loss. Enables efficient KV cache sharing across network.

#### 阅读笔记

发在 SIGCOMM 2024。切入点是 KV Cache 的网络传输瓶颈：如果多个请求共享同一段上下文（比如同一份 RAG 检索结果），理论上可以复用已有的 KV Cache 避免重复计算，但 KV Cache 体积太大（LLaMA-7B 的 4K 上下文就有几百 MB），走网络传太慢。

CacheGen 利用 KV 张量在不同 layer 和 head 上值域分布差异大这个特性，做了一套专用的压缩编码器。还能根据当前网络带宽动态调压缩率，带宽好就轻压保质量，带宽差就重压或者干脆重算。

实测 3.5-4.3 倍压缩，上下文加载延迟降 3.2-3.7 倍，输出质量基本不受影响。

这个思路跟数据库里的列存压缩策略有相通的地方——不同列用不同编码。survey 第 7 章可以提一下。

---

### [DistServe] DistServe: Disaggregating Prefill and Decoding for Goodput-optimized Large Language Model Serving

- **Authors**: Yinmin Zhong, Shengyu Liu et al. (Peking University)
- **Venue**: OSDI 2024
- **PDF**: https://arxiv.org/abs/2401.09670

**Summary**:
> Disaggregates prefill (compute-intensive) and decoding (memory-bound) phases to separate GPU pools. Co-optimizes resource allocation and parallelism strategy per phase independently, minimizing TTFT (time to first token) and TPOT (time per output token) jointly.

**DB / Storage Used**:
> KV cache migrated between prefill and decoding clusters via optimized transfer; placement strategy based on cluster bandwidth topology.

**Key Contribution**:
> 7.4× more requests served or 12.6× tighter SLO compared to monolithic systems. Proves that disaggregation (storage/compute separation, a DB concept) applies to LLM serving.

#### 阅读笔记

OSDI 2024，跟 Mooncake 思路类似都是 prefill-decode 分离，但学术味更浓一些，有比较系统的建模和优化。

核心观察是 prefill 阶段计算密集（适合大 GPU、少显存），decode 阶段访存密集（适合小 GPU、大显存），混在一起跑互相拖后腿。拆开后可以分别优化并行策略和资源分配。KV Cache 在两类节点间迁移，调度器根据集群拓扑优化传输路径。

在 LLaMA-70B 等模型上测试，同样延迟要求下能服务 7.4 倍请求，或者同样请求量下延迟收紧 12.6 倍。提升很显著。

这类 prefill-decode 分离架构，其实跟数据库领域早就在做的 OLTP-OLAP 分离、读写分离是一个思路。KV Cache 在集群间的传输和放置策略也是分布式数据管理的经典问题。可以放 survey 第 7 章一起讨论。

---

## 补充资料：行业实践与技术博客

这个方向工程实践跑得比论文快很多，很多优化技巧是在开源项目的 issue 和 blog 里先出现的。

- **vLLM 官方文档和博客**：https://docs.vllm.ai/  
  PagedAttention 论文读完之后一定要看 vLLM 的文档，实际部署时的参数调优（`gpu_memory_utilization`、`max_num_seqs`、`block_size` 等）全靠文档。社区的 GitHub Discussions 里也有大量关于不同模型、不同硬件上的经验分享。

- **SGLang 官方博客**：https://lmsys.org/blog/  
  LMSYS 团队（做 Chatbot Arena 那帮人）搞的项目，博客上有 RadixAttention 的可视化讲解，比论文好理解很多。还有关于 structured output 加速的文章也值得一看。

- 知乎上搜"大模型推理优化"或者"vLLM 部署经验"能找到很多帖子。社区里讨论得比较热的话题包括：什么时候该用 continuous batching、不同量化方式对吞吐的影响、多卡推理怎么配 tensor parallel。有些帖子写得非常详细，带 benchmark 数据的那种。

- **Anyscale（Ray）博客上关于 LLM Serving 的系列文章**：https://www.anyscale.com/blog  
  Ray 团队写了不少关于分布式 LLM 推理的实战文章，包括怎么用 Ray Serve 部署 vLLM、如何做 auto-scaling。他们还有跟 vLLM 团队合作的性能对比测试，数据比较可信。

- FlexGen（https://github.com/FMInference/FlexGen ）是另一个值得了解的推理优化项目，主打单卡跑大模型，把 KV Cache 卸载到 CPU/SSD。虽然现在用的人不多了，但 offloading 的思路跟 Mooncake 那篇异曲同工。

- 公众号"机器之心"和"量子位"经常转发推理优化相关的文章，虽然比较偏新闻报道，但可以用来追踪这个领域最新的开源项目和技术动态。
