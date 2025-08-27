# Process vs Thread

A process is an independent program in execution with its own memory space, while a thread is a lightweight unit of execution within a process that shares the process's memory space. Understanding this distinction is fundamental to operating systems and concurrent programming.

## Key Concepts

- **Process**: Independent execution unit with separate memory space, file descriptors, and system resources
- **Thread**: Lightweight execution unit within a process, shares memory space and resources with other threads
- **Memory Isolation**: Processes have separate address spaces, threads share the same address space
- **Context Switching**: Thread switching is faster than process switching due to shared memory
- **Communication**: Processes use IPC mechanisms, threads can communicate through shared memory
- **Fault Isolation**: Process crash doesn't affect other processes, thread crash can crash entire process

## Comparison Table

| Aspect | Process | Thread |
|--------|---------|--------|
| **Memory Space** | Separate address space | Shared address space |
| **Creation Time** | High overhead | Low overhead |
| **Context Switch** | Expensive (save/restore memory maps) | Cheap (save/restore registers) |
| **Communication** | IPC (pipes, sockets, shared memory) | Shared variables, memory |
| **Fault Tolerance** | Isolated (crash doesn't affect others) | Crash affects entire process |
| **Resource Sharing** | Minimal (explicit IPC) | High (automatic sharing) |

## Practical Example

```c
// Process Creation (fork())
#include <unistd.h>
#include <sys/wait.h>

int main() {
    pid_t pid = fork();
    
    if (pid == 0) {
        // Child process
        printf("Child process: PID = %d\n", getpid());
        exit(0);
    } else {
        // Parent process
        printf("Parent process: PID = %d, Child PID = %d\n", getpid(), pid);
        wait(NULL);  // Wait for child to finish
    }
    return 0;
}

// Thread Creation (pthread)
#include <pthread.h>

void* thread_function(void* arg) {
    printf("Thread ID: %lu\n", pthread_self());
    return NULL;
}

int main() {
    pthread_t thread;
    pthread_create(&thread, NULL, thread_function, NULL);
    pthread_join(thread, NULL);  // Wait for thread to finish
    return 0;
}
```

## Memory Layout Comparison

```
PROCESS MEMORY LAYOUT:
Process 1: [Code|Data|Heap|Stack]
Process 2: [Code|Data|Heap|Stack]
(Completely separate memory spaces)

THREAD MEMORY LAYOUT:
Process: [Code|Data|Heap] + [Stack1|Stack2|Stack3]
(Shared code, data, heap; separate stacks)
```

## Common Interview Questions

### Q1: Why is thread creation faster than process creation?
**Answer**: Thread creation is faster because:
- No need to create new address space (shares parent's memory)
- No need to copy memory mappings and file descriptors
- Only need to allocate new stack space and initialize thread control block
- Process creation involves duplicating entire process image and setting up new memory management structures

### Q2: When would you use processes vs threads?
**Answer**: 
**Use Processes when:**
- Need fault isolation (web servers, where one request crash shouldn't affect others)
- Security isolation required
- Different programs/applications
- Distributed systems across different machines

**Use Threads when:**
- Need to share data frequently (GUI applications)
- Performance is critical (game engines, media processing)
- Tasks are closely related and can benefit from shared memory

### Q3: What happens when a thread crashes vs when a process crashes?
**Answer**:
- **Thread crash**: Usually terminates the entire process and all threads within it (segmentation fault affects whole process)
- **Process crash**: Only affects that specific process, other processes continue running unaffected
- This is why multi-process architectures (like Chrome's process-per-tab) provide better stability

### Q4: Explain the difference between user-level and kernel-level threads.
**Answer**:
- **User-level threads**: Managed by user-space library, kernel sees only one process. Fast switching but no true parallelism on multi-core systems
- **Kernel-level threads**: Managed by OS kernel, true parallelism possible. Slower switching but better for CPU-intensive tasks
- **Hybrid model**: Many-to-many mapping provides benefits of both approaches

### Q5: What is the difference between multithreading and multiprocessing?
**Answer**:
- **Multithreading**: Multiple threads within single process, shared memory, faster communication, less fault tolerance
- **Multiprocessing**: Multiple processes, isolated memory, slower communication via IPC, better fault tolerance
- Choice depends on whether you prioritize performance (multithreading) or reliability (multiprocessing)

---
[← Back to Main Guide](./README.md)
