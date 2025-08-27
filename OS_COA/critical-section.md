# Critical Section Problem

The critical section problem involves coordinating multiple processes/threads accessing shared resources to prevent race conditions. It's fundamental to understanding synchronization in concurrent programming.

## Key Concepts

- **Critical Section**: Code segment that accesses shared variables and must not be executed by more than one process simultaneously
- **Race Condition**: Output depends on the unpredictable timing of process execution
- **Mutual Exclusion**: Only one process can be in critical section at any time
- **Progress**: If no process is in critical section and some want to enter, one must be allowed
- **Bounded Waiting**: Limit on how long a process waits to enter critical section
- **Atomic Operation**: Indivisible operation that completes without interruption

## Critical Section Structure

```c
// General structure of critical section
do {
    // Entry section - request permission to enter
    acquire_lock();
    
    // Critical section - access shared resources
    shared_variable++;
    
    // Exit section - release permission
    release_lock();
    
    // Remainder section - non-critical work
    other_work();
    
} while (true);
```

## Requirements for Critical Section Solution

### 1. Mutual Exclusion
Only one process can execute in critical section at any time
```c
// BAD: Race condition example
int counter = 0;

void increment() {
    counter++;  // Non-atomic: load, increment, store
}

// If two threads call increment() simultaneously:
// Thread 1: load counter (0) → increment (1) → store (1)
// Thread 2: load counter (0) → increment (1) → store (1)
// Result: counter = 1 (should be 2)
```

### 2. Progress
If critical section is empty and processes want to enter, selection cannot be postponed indefinitely
```c
// BAD: Violates progress
bool turn = 0;
if (turn == my_id) {
    // enter critical section
}
// If other process never wants to enter, this process waits forever
```

### 3. Bounded Waiting
There must be a bound on waiting time after a process requests entry
```c
// BAD: Unbounded waiting
while (test_and_set(&lock)) {
    // Process may wait forever if unlucky with timing
}
```

## Classic Solutions

### 1. Peterson's Algorithm (Two Processes)
```c
// Shared variables
bool flag[2] = {false, false};  // Intent to enter
int turn = 0;                   // Whose turn it is

// Process Pi (i = 0 or 1)
void critical_section_process(int i) {
    do {
        // Entry section
        flag[i] = true;           // I want to enter
        turn = 1 - i;             // Give turn to other process
        while (flag[1-i] && turn == 1-i) {
            // Wait if other wants to enter and it's their turn
        }
        
        // Critical section
        // ... access shared resource ...
        
        // Exit section
        flag[i] = false;          // I'm leaving
        
        // Remainder section
        // ... other work ...
        
    } while (true);
}
```

### 2. Bakery Algorithm (N Processes)
```c
// Shared variables
bool choosing[n] = {false};
int number[n] = {0};

void bakery_algorithm(int i) {
    do {
        // Entry section
        choosing[i] = true;
        number[i] = max(number[0], number[1], ..., number[n-1]) + 1;
        choosing[i] = false;
        
        for (int j = 0; j < n; j++) {
            while (choosing[j]);  // Wait if j is choosing number
            while ((number[j] != 0) && 
                   ((number[j] < number[i]) || 
                    ((number[j] == number[i]) && (j < i)))) {
                // Wait if j has smaller number or same number but smaller id
            }
        }
        
        // Critical section
        // ... access shared resource ...
        
        // Exit section
        number[i] = 0;
        
        // Remainder section
        // ... other work ...
        
    } while (true);
}
```

### 3. Hardware Solutions

#### Test-and-Set
```c
// Atomic test-and-set instruction
bool test_and_set(bool *target) {
    bool rv = *target;
    *target = true;
    return rv;
}

// Usage
bool lock = false;

void critical_section() {
    while (test_and_set(&lock)) {
        // Busy wait (spin lock)
    }
    
    // Critical section
    
    lock = false;  // Release lock
}
```

#### Compare-and-Swap
```c
// Atomic compare-and-swap instruction
int compare_and_swap(int *value, int expected, int new_value) {
    int temp = *value;
    if (*value == expected) {
        *value = new_value;
    }
    return temp;
}

// Usage for mutex
int lock = 0;

void acquire() {
    while (compare_and_swap(&lock, 0, 1) != 0) {
        // Busy wait
    }
}

void release() {
    lock = 0;
}
```

## Modern Solutions

### Pthread Mutex
```c
#include <pthread.h>

pthread_mutex_t mutex = PTHREAD_MUTEX_INITIALIZER;

void* thread_function(void* arg) {
    pthread_mutex_lock(&mutex);
    
    // Critical section
    shared_variable++;
    
    pthread_mutex_unlock(&mutex);
    
    return NULL;
}
```

### Spinlock vs Mutex
```c
// Spinlock - busy waiting
typedef struct {
    volatile int locked;
} spinlock_t;

void spin_lock(spinlock_t *lock) {
    while (__sync_lock_test_and_set(&lock->locked, 1)) {
        // Busy wait - wastes CPU cycles
    }
}

// Mutex - blocking
void mutex_lock(pthread_mutex_t *mutex) {
    // If lock unavailable, thread is blocked (context switch)
    // CPU can run other threads
}
```

## Common Interview Questions

### Q1: What is a race condition and how can it be prevented?
**Answer**: A race condition occurs when multiple processes/threads access shared data concurrently and the final result depends on the timing of their execution. It can be prevented by:
- Using mutex/semaphores for mutual exclusion
- Making operations atomic
- Proper synchronization mechanisms
- Avoiding shared mutable state when possible

### Q2: Why doesn't this simple solution work for critical section?
```c
bool lock = false;
if (!lock) {
    lock = true;
    // critical section
    lock = false;
}
```
**Answer**: This creates a race condition because checking and setting the lock are separate operations. Two processes can both see lock=false simultaneously, both set it to true, and both enter the critical section. Need atomic test-and-set operation.

### Q3: What are the advantages and disadvantages of Peterson's algorithm?
**Answer**: 
**Advantages**: Pure software solution, guarantees mutual exclusion, progress, and bounded waiting
**Disadvantages**: Only works for 2 processes, requires busy waiting (wastes CPU), assumes atomic reads/writes, doesn't work on modern architectures due to instruction reordering

### Q4: When would you use a spinlock vs a mutex?
**Answer**: 
**Spinlock**: Short critical sections, multiprocessor systems, when context switching overhead is higher than spinning time
**Mutex**: Long critical sections, single processor, when you want to allow other threads to run while waiting

### Q5: What is the ABA problem in concurrent programming?
**Answer**: ABA problem occurs in lock-free programming when a value changes from A to B and back to A between reads. A thread may think nothing changed when actually significant changes occurred. Solutions include:
- Using version numbers/timestamps
- Hazard pointers
- Compare-and-swap with pointer and counter

---
[← Back to Main Guide](./README.md)
