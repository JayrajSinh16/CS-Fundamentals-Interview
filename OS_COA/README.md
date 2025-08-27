# Operating Systems & Computer Architecture Interview Guide 🖥️

A comprehensive guide covering essential OS and Computer Architecture concepts for technical interviews at top companies.

## 📋 Table of Contents

### Operating Systems

#### Process Management
- [Process vs Thread](./process-vs-thread.md)
- [Process States & State Transitions](./process-states.md)
- [CPU Scheduling Algorithms](./cpu-scheduling.md)
- [Inter-Process Communication](./inter-process-communication.md)

#### Synchronization & Deadlocks
- [Critical Section Problem](./critical-section.md)
- [Semaphores & Mutex](./semaphores-mutex.md)
- [Classic Synchronization Problems](./classic-sync-problems.md)
- [Deadlock Management](./deadlock-management.md)

#### Memory Management
- [Memory Allocation Techniques](./memory-allocation.md)
- [Virtual Memory & Page Replacement](./virtual-memory.md)
- [Thrashing](./thrashing.md)
- [Fragmentation](./fragmentation.md)

#### File Systems
- [File Organization Methods](./file-organization.md)
- [Directory Structures](./directory-structures.md)
- [Disk Scheduling Algorithms](./disk-scheduling.md)

### Computer Architecture

#### Basic Concepts
- [Von Neumann vs Harvard Architecture](./von-neumann-harvard.md)
- [RISC vs CISC](./risc-vs-cisc.md)
- [Endianness](./endianness.md)
- [Instruction Cycle](./instruction-cycle.md)

#### Memory Hierarchy
- [Cache Memory & Mapping Techniques](./cache-memory.md)
- [Cache Performance](./cache-performance.md)
- [Memory Types (RAM/ROM)](./memory-types.md)

#### CPU & Performance
- [Pipelining & Hazards](./pipelining-hazards.md)
- [Addressing Modes](./addressing-modes.md)
- [Interrupts](./interrupts.md)

#### I/O & Storage
- [I/O Techniques](./io-techniques.md)
- [RAID Levels](./raid-levels.md)

## 🎯 How to Use This Guide

1. **Start with Process Management** - Foundation of operating systems
2. **Master Synchronization** - Critical for concurrent programming
3. **Understand Memory Management** - Essential for performance optimization
4. **Learn Computer Architecture** - Hardware understanding for system design
5. **Practice Numerical Examples** - Especially scheduling and page replacement algorithms

## 💡 Interview Tips

### For Operating Systems:
- **Know scheduling algorithms** with time complexities
- **Understand deadlock scenarios** and prevention strategies
- **Explain memory management** concepts clearly
- **Practice numerical problems** for CPU scheduling and page replacement
- **Use real-world examples** to demonstrate understanding

### For Computer Architecture:
- **Understand performance metrics** (CPI, throughput, latency)
- **Know cache behavior** and performance implications
- **Explain pipelining benefits** and hazard handling
- **Understand I/O techniques** and their trade-offs
- **Be familiar with RAID configurations** for data storage

## 🏢 Company Focus

This guide covers topics frequently asked at:

### Operating Systems Focus:
- **Google** - Process synchronization, memory management, distributed systems
- **Amazon** - CPU scheduling, deadlock handling, system performance
- **Oracle** - Memory allocation, file systems, database-OS interaction
- **Samsung** - Embedded systems, real-time scheduling, memory optimization

### Computer Architecture Focus:
- **Intel/AMD** - Cache design, pipelining, instruction sets
- **ARM** - RISC architecture, low-power design, embedded systems
- **NVIDIA** - Memory hierarchy, parallel processing, GPU architecture
- **Apple** - System optimization, cache performance, custom silicon

## 📊 Quick Reference

### Most Important Algorithms to Remember:
- **CPU Scheduling**: Round Robin, SJF, Priority
- **Page Replacement**: LRU, FIFO, Optimal
- **Disk Scheduling**: SCAN, C-SCAN, SSTF
- **Deadlock Detection**: Wait-for graph, Banker's algorithm

### Key Performance Metrics:
- **CPU Utilization** = (Total time - Idle time) / Total time
- **Throughput** = Number of processes completed / Time period
- **Turnaround Time** = Completion time - Arrival time
- **Cache Hit Ratio** = Cache hits / (Cache hits + Cache misses)

## 🔗 Related Topics

- **System Design** - Understanding OS concepts helps in distributed system design
- **Database Systems** - Memory management and file systems are crucial
- **Network Programming** - Process communication and synchronization
- **Embedded Systems** - Real-time OS and hardware architecture knowledge

---

*Each topic includes definitions, key concepts, practical examples, and common interview questions with detailed answers.*

## 📝 Study Schedule Recommendation

### Week 1: Process Management
- Day 1-2: Process vs Thread, Process States
- Day 3-4: CPU Scheduling Algorithms
- Day 5-7: Practice numerical problems and IPC

### Week 2: Synchronization & Memory
- Day 1-3: Critical sections, Semaphores, Classic problems
- Day 4-5: Deadlock management
- Day 6-7: Memory allocation and virtual memory

### Week 3: File Systems & Architecture Basics
- Day 1-2: File organization and directory structures
- Day 3: Disk scheduling algorithms
- Day 4-5: Von Neumann, RISC vs CISC
- Day 6-7: Instruction cycle and endianness

### Week 4: Advanced Architecture & Review
- Day 1-3: Cache memory and performance
- Day 4-5: Pipelining and I/O techniques
- Day 6-7: RAID levels and comprehensive review
