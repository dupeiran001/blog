+++
title = "Quantifying the Cost of Software-Managed Synchronization on Multi-Host CXL Memory"
description = "An end-to-end characterization of software-managed coherence, synchronization, and application scalability on multi-host CXL shared memory."
draft = false
+++

*SC26 (The International Conference for High Performance Computing, Networking, Storage, and Analysis)*
November 15–20, 2026 · Chicago, Illinois, USA

**Peiran Du**¹, Weishu Deng¹, Yang Song¹, Chenfei Liao¹, Yujie Yang¹, Ren Wang², Hui Lu¹, Jia Rao¹

¹ The University of Texas at Arlington  
² Intel Labs

## Resources

- **Code and reproducibility:** [GitHub — cxl-shm-bench](https://github.com/dupeiran001/cxl-shm-bench)
- **Archived research artifact:** [Zenodo — 10.5281/zenodo.21753320](https://doi.org/10.5281/zenodo.21753320)
- **Paper PDF:** *TBD*
- **Paper DOI:** *TBD*

## Abstract

Current Compute Express Link (CXL) platforms enable multi-host shared, byte-addressable memory without cross-host hardware cache coherence or atomic operations, forming a hardware-supported distributed shared memory (DSM). However, this CXL-DSM model requires software-managed cross-host coherence and synchronization, introducing significant challenges for scaling parallel applications beyond raw CXL memory performance. This paper presents an end-to-end characterization of this cost stack on a real four-host CXL platform, spanning memory bandwidth and latency, publish/observe visibility primitives, synchronization mechanisms, and application-level behavior. We show that cross-host visibility incurs ∼1 µs latency and lock acquisition costs 3–5 µs, significantly exceeding local hardware synchronization. These costs dominate in synchronization-intensive workloads and prevent scalable multi-host execution, with the volume of republished shared state, rather than the count of lock or barrier operations, as the deciding factor. In contrast, workloads with read-dominated or partitioned, owner-local writes can achieve net speedup by leveraging additional cores and cache capacity. Across microbenchmarks, graph workloads, and a concurrent hash table, application scalability tracks remote-write intensity, synchronization frequency, and data placement. Our results highlight fundamental limits of non-coherent shared memory and provide practical guidance for designing and deploying applications on current multi-host non-coherent CXL systems.

## Overview

Compute Express Link (CXL) allows multiple hosts to map and access the same pool of byte-addressable memory using ordinary loads and stores. On current multi-host systems, however, the hosts do not share hardware cache coherence or cross-host atomic operations. Applications must explicitly publish modified cache lines and invalidate stale copies before observing updates made by other hosts.

This paper investigates the performance implications of that software-managed visibility model. Using a real four-host CXL platform, we measure the cost of memory accesses, cross-host publish/observe operations, locks, barriers, and software atomics, then evaluate how those costs affect parallel applications. The central question is not simply how fast CXL memory is, but **which sharing and synchronization patterns can scale across hosts**.

## Key Findings

- **Cross-host visibility has a measurable cost.** Software-managed coherence takes approximately **0.9–1.5 µs per one-way update**, depending on the publish/observe mechanism.
- **Synchronization is much slower than host-local primitives.** Cross-host lock acquisition costs approximately **3–5 µs**, while software-implemented atomic operations take approximately **3 µs**.
- **Workload behavior matters more than synchronization counts alone.** The volume of shared state that must be published and refreshed across hosts is a major determinant of scalability.
- **Read-dominated and partitioned workloads benefit most.** Triangle counting scales effectively with additional hosts, and a concurrent hash table achieves near-linear scaling when updates are distributed across mostly independent buckets. In contrast, synchronization-heavy BFS scales poorly, while PageRank variants are limited by repeated publication and refresh of shared state.

## Experimental Scope

The evaluation uses a four-host CXL distributed shared memory system with a CXL switch and shared Type-3 memory devices. It combines memory bandwidth and latency microbenchmarks, software visibility measurements, centralized and decentralized synchronization primitives, graph workloads (BFS, PageRank, PageRank-SpMV, and triangle counting), and a concurrent hash table.

## Artifact and Reproducibility

The [cxl-shm-bench repository](https://github.com/dupeiran001/cxl-shm-bench) contains the implementations, measurement scripts, collected data, and instructions for reproducing the experiments and figures in the paper. Its components cover memory microbenchmarks, synchronization primitives, multi-host graph kernels, a concurrent hash table, and supporting CXL memory-management utilities.

An archival copy of the artifact is available on [Zenodo](https://doi.org/10.5281/zenodo.21753320).
