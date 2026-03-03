# [Layer 2 · Memory] Paper Notes: KV Cache Management

> How do systems manage the massive, dynamic intermediate states (Key-Value Cache) generated during LLM inference?
> **三层定位**：中间层记忆系统——**KV Cache** 是大模型推理时的"运行时内存"（RAM），其管理效率直接决定了系统的吞吐量和延迟。

---

## Key Concepts

-   **KV Cache**: Intermediate attention key and value tensors generated during the prefill phase and reused/updated during the decode phase.
-   **Memory Fragmentation**: The primary bottleneck in early LLM serving. Pre-allocating contiguous memory for unknown output lengths wastes 60-80% of GPU memory.
-   **PagedAttention**: An algorithm inspired by OS virtual memory paging to allow non-contiguous storage of KV blocks.
-   **Disaggregation**: Separating compute (GPU) from memory (KV Cache storage) to allow independent scaling.

---

## Papers / Systems

### [vLLM] vLLM: Easy, Fast, and Cheap LLM Serving with PagedAttention

-   **Authors**: Kwon et al. (UC Berkeley)
-   **Venue**: SOSP 2023
-   **PDF**: https://arxiv.org/abs/2309.06180
-   **Code**: https://github.com/vllm-project/vllm

**Summary**:
> Addresses the memory fragmentation problem in LLM serving. Introduces **PagedAttention**, which divides the KV cache into fixed-size blocks (pages) and manages them via a block table, similar to OS virtual memory.

**Key Contribution**:
> **PagedAttention**:
> 1.  **Algorithm**: Attention kernel that can read/write non-contiguous blocks on the fly.
> 2.  **System**: Block Manager that handles allocation, freeing, and sharing of blocks.
>
> **Results**:
> -   **Zero Waste**: Eliminates internal external fragmentation.
> -   **Sharing**: Enables "Copy-on-Write" for parallel sampling/beam search (sharing the prompt blocks across multiple sequences).
> -   **Swapping**: Supports paging out blocks to CPU memory when GPU memory is full.

#### 阅读笔记

vLLM 是 2023 年 LLM 推理系统最重要的工作之一。它的核心洞察是：LLM 推理时的显存浪费问题，跟几十年前操作系统面临的内存碎片问题是一样的。

传统的做法是像 C 语言的 `malloc` 一样预分配一大块连续显存（因为你不知道用户会问多长），结果大部分都空着。vLLM 搞了个"显存版的分页机制"（PagedAttention）：把 KV Cache 切成小块（Token Block），用一张表（Block Table）记录逻辑块到物理块的映射。

这直接导致了吞吐量的暴涨（2-4倍），因为显存利用率上去了，Batch Size 就能开得更大。
从数据库视角看，vLLM 实现了一个**专用的内存数据库（In-Memory DB）**来管理 KV Cache，具备了 Buffer Pool 管理、逻辑/物理映射、甚至 Copy-on-Write（快照）等数据库核心特性。

---

### [Mooncake] Mooncake: A KVCache-centric Disaggregated Architecture for LLM Serving

-   **Authors**: Qin et al. (KVCache.AI / Tsinghua)
-   **Venue**: arXiv 2024
-   **PDF**: https://arxiv.org/abs/2407.00000 (Scenario based)
-   **Code**: https://github.com/kvcache-ai/Mooncake

**Summary**:
> Proposes a disaggregated architecture where the KV Cache is separated from the compute instance. Allows "Context Shift" (moving a session to another GPU without recomputing) and enables a global pool of shared memory.

**Architecture**:
> **KVCache Store**: A distributed storage system optimized for WORM (Write-Once-Read-Many) workloads.
> **Tiered Hierarchy**:
> -   Tier 0: On-chip GPU Memory (Hot)
> -   Tier 1: DRAM Pool (Warm - via RDMA)
> -   Tier 2: SSD / NVMe (Cold)

**Key Contribution**:
> **Separation of Compute and Memory**: Solves the resource skew problem (Prefill is compute-bound, Decode is memory-bound).
> **Global Scheduler**: Routes requests based on data locality (where the KV cache is), effectively implementing "Data-Aware Scheduling" from distributed databases.

#### 阅读笔记

如果说 vLLM 是单机操作系统的内存管理，Mooncake 就是**分布式文件系统 / 分布式共享内存**。

核心问题是：如果我把一个长对话请求调度到了另一台机器，那之前的 KV Cache 全都没了，得由新机器重算一遍（Prefill），极其浪费。Mooncake 把 KV Cache 单独拿出来做一个全局共享的池子，通过 RDMA 高速访问。

这就像从本地硬盘进化到了 NAS/S3：计算节点（GPU）变成了无状态的（Stateless），状态全在远端存储里。这让系统的弹性伸缩能力大大增强。
对于 survey 来说，Mooncake 完美展示了 L2 Memory 的极致形态——它真的是一个"数据库"了。

---
