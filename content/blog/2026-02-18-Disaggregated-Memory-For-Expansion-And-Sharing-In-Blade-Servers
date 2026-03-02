+++
title = "Disaggregated Memory for Expansion and Sharing in Blade Servers"
[extra]
bio = """ """
[[extra.authors]]
name = "Eric Morgan-Bronec (Leader)"
[[extra.authors]]
name = "Thomas Pinon (Scribe and Blogger)"
[[extra.authors]]
name = "Humoud Almutairi"
[[extra.authors]]
name = "Donovan Burk"
[[extra.authors]]
name = "Dustin Bajarin-Freitas"
[[extra.authors]]
name = "Max Leibowitz"
+++

# Motivation
This paper seeks to address the issue of the "Memory Capacity Wall". Datacenter memory requirements are rising rapidly due to an upwards trend in the number of CPU cores per socket, increased utilization of virtual machines, and heavier workloads. However, Physical constraints and rising memory prices make it difficult to justify simply adding more memory to these machines. Additionally, studies have shown that traditional provisioning methods have led to significant underutilization, inflating datacenter costs.

# Overview
The authors propose the idea of disaggregated memory, a design in which a memory "blade" acts as a shared memory resource that is made available to multiple servers. The blade consists of a protocol engine to interface with the backplane, a memory controller (ASIC or small CPU, and one or more channels of commodity DRAM DIMMs. The blade uses "superpages", which are 16 MB large, in order to keep mapping tables small and allow for fast lookups. In the interest of client isolation and safety, the memory controller translates incoming addresses into local addresses called remote machine memory addresses (RMMAs). The remote memory capacity is provisioned to the client servers based on the decisions of the clients' virtual machine monitors (VMMs) and higher-level management software.

# System Architecture
The researchers explored two distinct system architectures, page-swapping (PS) and fine-grained remote access (FGRA).

## Page-Swapping (PS)
This approach to the system design is intended to reduce hardware complexity while sacrificing some performance. In this case, the only piece of non-standard hardware is the memory blade itself. No hardware changes are required for the compute blades. The page-swapping architecture is based on the concept of leveraging existing virtual memory infrastructure to detect accesses to remote memory. When such an access is detected, data is pulled from the memory blade into the server's local memory with page-level (4KB in this study) granularity. This page-level approach takes advantage of access locality, allowing the PCIe memory migration overhead to be amortized. The page management is implemented at the VMM layer. In order to decouple the processes of swapping pages in from remote memory and evicting pages to remote memory, a pool of free local pages is maintained. 

## Fine-Grained Remote Access (FGRA)
This approach to the system design is intended to explore closer to the upper bounds of performance while allowing for some hardware changes to the client servers. The design uses custom  hardware called a "coherence filter" to to redirect cache fill requests from the client server to the remote memory blade. This design allows the remote memory space to be accessed directly by the client's operating system. Additionally, this design allows for cache-block granularity.

# Results
Tests were run on a system with only 75% of the required local memory available. The base case was with no remote memory present, so the system was forced to swap pages to the disk, harming performance greatly.

## Speedup
The addition of the remote memory blade produced significantly positive results, with speedups ranging from 4x to 320x. As a surprise to the researchers, the PS architecture actually outperformed the FGRA architecture. The key here is that every subsequent access to the page that was pulled from remote memory will only incur local memory delays. The amortization of the connection overhead  worked as planned.

## Power and Cost
There were significant power savings observed. The memory power draw of the standard fully-provisioned server was estimated to be 21 watts, while the total per-server memory power draw of the disaggregated memory solution was estimated to be 15 watts. In terms of cost, up to an 87% performance-per-dollar increase was seen in the best case scenario. These results came from an "ensemble-level memory sharing" experiment, where memory was further optimized to exploit the varying requirements of a cluster of servers over time.

# Class Discussions
- Why use coherence?
We discussed the rationale behind using coherence at all, since the coherence-less architecture in this case (PS) appeared to perform better. We talked about how the approach focused on exploiting locality instead. While the FGRA architecture had remote memory fetches driven by the coherence protocol, the PS architecture's fetches were simply driven by page faults. We talked about how coherence must be "all or nothing".

- When to use coherence?
As an expansion to the previous question, there was a question about when coherence can/should be used. The answer was that a memory controller must be involved, and the memory must be able to participate in the coherence network.

- Would PS or FGRA be better for today's workloads?
The answer to this question was FGRA, since it is similar to CXL.

- What might the consumer side look like?
We had a discussion about how these studies may translate into consumer products. Professor Hale talked about the possibilty of Thunderbolt-style memory expansion units. The main issue would be working with the inevitable latency. There was also a brief discussion of the possibility of memory extension over WiFi or even P2P networks. The feasibility of these ideas would depend heavily on radio speeds and the needs of the workloads in question.

- Is there research happening in relation to the interconnects?
We had a discussion about interconnect technologies and network types. The answer seems to be that CXL is the main focus, but Ethernet and Infiniband are also improving.

- Is there a limit on how many servers or racks can be connected together?
The answer seemed to be that the main constraints are pin bandwidth and cabling. All-to-all connections are generally unrealistic, and routed networks are more common.

# References
Kevin Lim, Jichuan Chang, Trevor Mudge, Parthasarathy Ranganathan, Steven K. Reinhardt, and Thomas F. Wenisch. 2009. Disaggregated memory for expansion and sharing in blade servers. SIGARCH Comput. Archit. News 37, 3 (June 2009), 267–278. https://doi.org/10.1145/1555815.1555789

# AI Disclosure
Gemini Pro 3 was used to expedite analysis of the paper, but this blog post was human-written.
