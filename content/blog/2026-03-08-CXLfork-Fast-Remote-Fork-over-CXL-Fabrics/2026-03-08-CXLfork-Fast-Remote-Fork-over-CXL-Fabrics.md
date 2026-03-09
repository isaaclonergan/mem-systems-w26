+++
title = "CXLfork: Fast Remote Fork over CXL Fabrics"
[extra]
bio = """ """
[[extra.authors]]
name = "William Davis (Leader / Presentor)"
[[extra.authors]]
name = "Carlos Alvarado-Lopez (Leader / Presentor)"
[[extra.authors]]
name = "Suvrojyoti Paul (scribe)"
[[extra.authors]]
name = "James S. Tappert (blogger)"
+++

## Introduction
The rise of serverless computing (Function-as-a-Service) has brought the challenge of "cold starts" to the forefront of distributed systems research. When a function is invoked, the overhead of initializing its state can lead to significant latency. While local fork semantics can mitigate this on a single node, scaling across a cluster traditionally requires slow and memory-intensive remote cloning processes.

The paper *CXLfork: Fast Remote Fork over CXL Fabrics* introduces a novel remote fork interface designed for the emerging Compute Express Link (CXL) interconnect. By leveraging CXL's ability to provide shared, byte-addressable remote memory, CXLfork achieves near-zero serialization and zero-copy process cloning across nodes.
## Background and Motivation

### CXL's Advantage

The authors analyzed the access patterns of common FaaS workloads and categorized the data accesses into 3 types:
- **Init:** Data that is used for function initialization and is rarely accessed during execution.
- **Read-only:** Data that is only read during function execution.
- **Read/Write:** Data that is both written and read during function execution.

![Function Memory Footprint](./function-mem-footprint.png)

The results depicted above showed that on average the *Init* and *Read-only* functions made up the bulk of the memory footprint, being responsible for 72.2% and 23% of the footprint, respectively. These findings imply that remote forks can benefit significantly from storing the common states of the *Init* and *Read-only* functions in a CXL shared memory. Allowing cluster-wide memory deduplication and potentially increasing the number of function instances that can be run on a fixed local memory budget.

### Limitations of Existing Remote Fork Technologies
The paper analyzes two existing remote fork mechanisms, ***CRIU*** (*Checkpoint and Restore In Userspace*) and ***Mitosis***, and evaluates their performance when adapted to use shared CXL memory. 

#### ***CRIU:***
Referred to as a "*state-of-practice*" framework, ***CRIU*** does not utilize a cluster's network fabric. Instead, to create a remote fork, it uses Protocol Buffers to indirectly copy a process' state from one node to another through a serialization-deserialization process. The authors adapted this framework to CXL by caching the serialized OS state in shared CXL memory during the *checkpoint* phase and then deserializing to the target node during the *restore* phase. This process is shown in the image below:

![CRIU-CXL](./criu-cxl.png)

#### ***Mitosis:*** 
Referred to as a "*state-of-the-art*" framework, ***Mitosis*** utilizes the cluster's network fabric RDMA capabilities to accelerate the remote forking process. OS state is transferred similarly to ***CRIU's*** serialization-deserialization process, just using one-sided RDMA opererations to transfer the serialized OS state instead, the main difference being the forked process is executed without the parent process' memory pages available. This causes special page faults to trigger that will copy the required pages via remote paging. The authors adapted ***Mitosis*** to support CXL by replacing the RMDA operations with page copies over the shared CXL memory. The remote page faults and OS state transfers are also served through the CXL memory. This process is shown in the image below:

![Mitosis-CXL](./mitosis-cxl.png)

#### BERT Function Evaluation
These adaptations were evaluated by comparing the latency and memory usage during BERT function execution:

![BERT Function Results](./bert-results.png)

The authors found that ***CRIU's*** *restore* phase latency alone was 2.7x longer than the entirety of a local fork equivalent. Furthermore, ***CRIU's*** memory overhead was found to be 42x greater than a local fork. For ***Mitosis*** the findings were slightly better, but the latency overhead was still 2.6x longer than a local fork, and memory usage was 24x greater. 

These findings were the main motivations for designing a new remote fork interface for CXL fabrics. 

## Design of CXLfork

CXLfork addresses the challenges of remote cloning through three primary innovations:

- **1. Near-Zero Serialization** Instead of converting process state into a portable file format, CXLfork checkpoints process data and OS structures (like page tables) to CXL memory exactly as they are. To allow different OS instances to use these structures, CXLfork rebases them to index the CXL physical memory address space.
- **2. Direct State Mapping** By default, CXLfork maps the checkpointed state in CXL memory directly into the cloned process on the remote node. It uses Copy-on-Write (CoW) for any modifications, ensuring that the majority of the process remains shared and deduplicated across the cluster.
- **3. Fine-Grained Tiering** To manage the latency overhead of CXL memory (hundreds of nanoseconds), CXLfork provides tailored placement policies:
  - **Migrate-on-Write (MoW):** Localizes only the data that is being modified.
  - **Migrate-on-Access (MoA):** Localizes pages as they are accessed to minimize future CXL latency.
  - **Hybrid Tiering (HT):** Uses hardware "access" (A) bits to identify and proactively migrate "hot" pages to local memory.


## CXLporter: Scaling Serverless

The authors developed CXLporter, an autoscaler that utilizes CXLfork to manage serverless functions. It maintains a pool of "ghost containers"—empty shells that consume no CPU or local memory until a function is cloned into them on-demand via CXLfork. This approach allows for rapid scaling during load spikes without the memory pressure of keeping many "warm" containers idle.

## Evaluation and Results

The system was evaluated on an Intel Sapphire Rapids platform with an FPGA-based CXL memory device. Key findings include:

- **Latency:**
  - CXLfork is 2.26x faster than CRIU-CXL and 1.40x faster than Mitosis-CXL on average.
  - It is only 14% slower than a standard local fork.
- **Memory Efficiency:**
  - It reduces local memory consumption by 87% compared to CRIU and 61% compared to Mitosis.
- **Robustness**
  - Strong resistance to cache pollution from scans
  - Stable performance across diverse application behaviors
- **Cold Starts:**
  - CXLfork execution is 11x faster than a vanilla "cold" start.


## Class Discussion
!!! Need Scribe Notes !!!

## Conclusion

CXLfork demonstrates that shared memory fabrics can revolutionize traditional system interfaces. By providing a fast, memory-frugal remote fork, it enables serverless platforms to scale with high performance and high resource utilization, effectively bridging the gap between local and remote process cloning.

## References

- Alverti, C., et al. [*CXLfork: Fast Remote Fork over CXL Fabrics (ASPLOS '25*)](https://tianyin.github.io/pub/cxlfork.pdf)

# Generative AI Disclosure
* Gemini was used to generate a Markdown file template and to validate spelling and grammar.
