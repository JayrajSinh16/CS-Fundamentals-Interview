# Semaphores & Mutex

Semaphores and mutexes are synchronization primitives used to control access to shared resources in concurrent programming. They provide higher-level abstractions than hardware-based solutions for solving critical section problems.

## Key Concepts

- **Semaphore**: Integer variable with atomic wait() and signal() operations
- **Binary Semaphore**: Semaphore with values 0 or 1, similar to mutex
- **Counting Semaphore**: Can have any non-negative integer value
- **Mutex**: Mutual exclusion lock, binary semaphore with ownership concept
- **Critical Section**: Code that accesses shared resources
- **Deadlock**: Two or more processes waiting indefinitely for each other

## Semaphore Operations

### Basic Structure
```c
typedef struct {
    int value;              // Semaphore counter
    struct process *list;   // Queue of waiting processes
} semaphore;

// Atomic operations
void wait(semaphore *S) {   // Also called P() or down()
    S->value--;
    if (S->value < 0) {
        // Add this process to S->list
        block();  // Block the process
    }
}

void signal(semaphore *S) { // Also called V() or up()
    S->value++;
    if (S->value <= 0) {
        // Remove a process P from S->list
        wakeup(P);  // Wake up process P
    }
}
```

### Implementation Example
```c
#include <semaphore.h>
#include <pthread.h>

sem_t semaphore;
int shared_resource = 0;

void* producer(void* arg) {
    for (int i = 0; i < 10; i++) {
        // Produce item
        int item = i;
        
        // Wait for empty slot
        sem_wait(&semaphore);
        
        // Add item to buffer
        shared_resource = item;
        printf("Produced: %d\n", item);
        
        // Signal that item is available
        sem_post(&semaphore);
        
        sleep(1);
    }
    return NULL;
}

int main() {
    sem_init(&semaphore, 0, 1);  // Initialize with value 1
    
    pthread_t prod_thread;
    pthread_create(&prod_thread, NULL, producer, NULL);
    
    pthread_join(prod_thread, NULL);
    sem_destroy(&semaphore);
    return 0;
}
```

## Types of Semaphores

### 1. Binary Semaphore (Mutex-like)
```c
// Binary semaphore for mutual exclusion
sem_t binary_sem;
sem_init(&binary_sem, 0, 1);  // Initialize to 1

void critical_section() {
    sem_wait(&binary_sem);    // Acquire (value becomes 0)
    
    // Critical section - only one process can be here
    shared_variable++;
    
    sem_post(&binary_sem);    // Release (value becomes 1)
}
```

### 2. Counting Semaphore (Resource Management)
```c
// Counting semaphore for resource pool
#define NUM_RESOURCES 3
sem_t resource_sem;
sem_init(&resource_sem, 0, NUM_RESOURCES);  // 3 resources available

void use_resource() {
    sem_wait(&resource_sem);  // Acquire resource (decrements count)
    
    // Use the resource
    printf("Using resource, %d remaining\n", sem_value);
    sleep(2);  // Simulate work
    
    sem_post(&resource_sem);  // Release resource (increments count)
}
```

## Mutex vs Semaphore

### Mutex (Mutual Exclusion)
```c
#include <pthread.h>

pthread_mutex_t mutex = PTHREAD_MUTEX_INITIALIZER;

void* thread_function(void* arg) {
    pthread_mutex_lock(&mutex);
    
    // Critical section - guaranteed exclusive access
    shared_counter++;
    printf("Counter: %d\n", shared_counter);
    
    pthread_mutex_unlock(&mutex);
    return NULL;
}

// Mutex characteristics:
// 1. Only the thread that locked it can unlock it (ownership)
// 2. Cannot be used for signaling between threads
// 3. Typically used for protecting shared data
```

### Semaphore vs Mutex Comparison

| Feature | Mutex | Semaphore |
|---------|-------|-----------|
| **Purpose** | Mutual exclusion | Signaling & counting |
| **Ownership** | Yes (thread that locks must unlock) | No (any thread can signal) |
| **Initial Value** | Unlocked (1) | Any non-negative value |
| **Use Case** | Protecting shared data | Resource counting, signaling |
| **Recursive Locking** | Possible (with recursive mutex) | Not applicable |

## Classic Synchronization Problems

### 1. Producer-Consumer with Bounded Buffer
```c
#define BUFFER_SIZE 5

sem_t empty_slots;    // Count of empty slots
sem_t full_slots;     // Count of full slots  
sem_t mutex;          // Mutual exclusion for buffer

int buffer[BUFFER_SIZE];
int in = 0, out = 0;

void producer() {
    for (int i = 0; i < 20; i++) {
        int item = produce_item();
        
        sem_wait(&empty_slots);   // Wait for empty slot
        sem_wait(&mutex);         // Enter critical section
        
        buffer[in] = item;        // Add item to buffer
        in = (in + 1) % BUFFER_SIZE;
        
        sem_post(&mutex);         // Exit critical section
        sem_post(&full_slots);    // Signal new item available
    }
}

void consumer() {
    for (int i = 0; i < 20; i++) {
        sem_wait(&full_slots);    // Wait for item
        sem_wait(&mutex);         // Enter critical section
        
        int item = buffer[out];   // Remove item from buffer
        out = (out + 1) % BUFFER_SIZE;
        
        sem_post(&mutex);         // Exit critical section
        sem_post(&empty_slots);   // Signal empty slot available
        
        consume_item(item);
    }
}

int main() {
    sem_init(&empty_slots, 0, BUFFER_SIZE);  // Initially all empty
    sem_init(&full_slots, 0, 0);             // Initially no items
    sem_init(&mutex, 0, 1);                  // Binary semaphore
    
    // Create producer and consumer threads
    // ...
}
```

### 2. Readers-Writers Problem
```c
sem_t write_mutex;    // Controls access to shared data
sem_t read_count_mutex; // Controls access to reader count
int read_count = 0;

void reader() {
    sem_wait(&read_count_mutex);
    read_count++;
    if (read_count == 1) {
        sem_wait(&write_mutex);  // First reader blocks writers
    }
    sem_post(&read_count_mutex);
    
    // Reading shared data
    read_data();
    
    sem_wait(&read_count_mutex);
    read_count--;
    if (read_count == 0) {
        sem_post(&write_mutex);  // Last reader unblocks writers
    }
    sem_post(&read_count_mutex);
}

void writer() {
    sem_wait(&write_mutex);
    
    // Writing shared data
    write_data();
    
    sem_post(&write_mutex);
}
```

## Advanced Mutex Features

### Recursive Mutex
```c
pthread_mutexattr_t attr;
pthread_mutex_t recursive_mutex;

pthread_mutexattr_init(&attr);
pthread_mutexattr_settype(&attr, PTHREAD_MUTEX_RECURSIVE);
pthread_mutex_init(&recursive_mutex, &attr);

void recursive_function(int n) {
    pthread_mutex_lock(&recursive_mutex);
    
    if (n > 0) {
        recursive_function(n - 1);  // Same thread can lock again
    }
    
    pthread_mutex_unlock(&recursive_mutex);
}
```

### Timed Mutex
```c
struct timespec timeout;
clock_gettime(CLOCK_REALTIME, &timeout);
timeout.tv_sec += 5;  // 5 second timeout

if (pthread_mutex_timedlock(&mutex, &timeout) == 0) {
    // Got the lock within timeout
    // Critical section
    pthread_mutex_unlock(&mutex);
} else {
    // Timeout occurred
    printf("Could not acquire lock within timeout\n");
}
```

## Common Interview Questions

### Q1: What's the difference between a binary semaphore and a mutex?
**Answer**: 
- **Binary Semaphore**: Can be signaled by any thread, used for signaling between threads, no ownership concept
- **Mutex**: Has ownership (only locking thread can unlock), used for mutual exclusion, cannot be used for signaling
Example: Use semaphore for producer-consumer signaling, mutex for protecting shared data

### Q2: Can a semaphore value be negative?
**Answer**: Internally yes, but the semaphore abstraction shows it as non-negative. When value becomes negative, it represents the number of processes waiting. For example, if value is -3, it means 3 processes are blocked waiting for the semaphore.

### Q3: What happens if you call sem_post() more times than sem_wait()?
**Answer**: The semaphore value keeps increasing beyond its initial value. This can lead to more processes entering critical section than intended. It's a programming error that can violate mutual exclusion or resource limits.

### Q4: How do you prevent priority inversion with semaphores?
**Answer**: Priority inversion occurs when high-priority process waits for low-priority process holding a semaphore. Solutions:
- **Priority Inheritance**: Low-priority process temporarily inherits high priority
- **Priority Ceiling Protocol**: Process gets highest priority of any semaphore it might acquire
- **Avoid blocking in high-priority threads**

### Q5: What is a spurious wakeup and how do you handle it?
**Answer**: Spurious wakeup occurs when a waiting thread wakes up without the condition being met (due to signals, system calls). Handle by always checking condition in a loop:
```c
while (condition_not_met) {
    sem_wait(&semaphore);
}
```
Instead of just using if statement for condition checking.

---
[← Back to Main Guide](./README.md)
