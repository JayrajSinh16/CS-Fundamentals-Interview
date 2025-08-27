# Deadlock Management

Deadlock is a situation where a set of processes are blocked because each process is holding a resource and waiting for another resource acquired by some other process. Understanding deadlock prevention, avoidance, detection, and recovery is crucial for system design.

## Key Concepts

- **Deadlock**: A situation where processes are permanently blocked, waiting for each other
- **Circular Wait**: Chain of processes where each waits for resource held by next
- **Resource Allocation Graph**: Visual representation of resource allocation and requests
- **Safe State**: System state where all processes can complete execution
- **Unsafe State**: State that may lead to deadlock (but not necessarily)

## Deadlock Conditions (Coffman Conditions)

### Four Necessary Conditions for Deadlock:

1. **Mutual Exclusion**: Resources cannot be shared
2. **Hold and Wait**: Process holds resources while waiting for others
3. **No Preemption**: Resources cannot be forcibly taken away
4. **Circular Wait**: Circular chain of processes waiting for resources

```c
// Example deadlock scenario
Process P1:                    Process P2:
lock(resource_A);             lock(resource_B);
// ... some work               // ... some work
lock(resource_B);  ← WAIT     lock(resource_A);  ← WAIT
// ... more work               // ... more work
unlock(resource_B);           unlock(resource_A);
unlock(resource_A);           unlock(resource_B);

// Both processes will wait forever!
```

## Resource Allocation Graph (RAG)

```
Process → Resource (Request Edge)
Resource → Process (Assignment Edge)

Example:
    P1 ──→ R1 ──→ P2
    ↑              ↓
    R2 ←────────── P3

If RAG has cycle AND single instance per resource type → Deadlock
If RAG has cycle AND multiple instances → Possibly deadlock
```

## Deadlock Prevention

### Method 1: Eliminate Mutual Exclusion
```c
// Use read-write locks instead of exclusive locks where possible
pthread_rwlock_t data_lock = PTHREAD_RWLOCK_INITIALIZER;

void reader_function() {
    pthread_rwlock_rdlock(&data_lock);  // Multiple readers allowed
    // Read shared data
    pthread_rwlock_unlock(&data_lock);
}

void writer_function() {
    pthread_rwlock_wrlock(&data_lock);  // Exclusive write access
    // Modify shared data
    pthread_rwlock_unlock(&data_lock);
}
```

### Method 2: Eliminate Hold and Wait
```c
// Acquire all resources at once
int acquire_all_resources(int* resources, int count) {
    // Try to acquire all resources atomically
    for (int i = 0; i < count; i++) {
        if (try_lock(resources[i]) != SUCCESS) {
            // Release all previously acquired resources
            for (int j = 0; j < i; j++) {
                unlock(resources[j]);
            }
            return FAILURE;
        }
    }
    return SUCCESS;
}

void process_with_prevention() {
    int needed_resources[] = {R1, R2, R3};
    
    if (acquire_all_resources(needed_resources, 3) == SUCCESS) {
        // Do work with all resources
        // Release all resources
        for (int i = 0; i < 3; i++) {
            unlock(needed_resources[i]);
        }
    } else {
        // Wait and retry later
        sleep(1);
        process_with_prevention();  // Retry
    }
}
```

### Method 3: Allow Preemption
```c
// Priority-based preemption
typedef struct {
    int process_id;
    int priority;
    Resource* held_resources[MAX_RESOURCES];
    int resource_count;
} ProcessInfo;

void preempt_if_necessary(ProcessInfo* high_priority, ProcessInfo* low_priority) {
    if (high_priority->priority > low_priority->priority) {
        // Preempt resources from low priority process
        for (int i = 0; i < low_priority->resource_count; i++) {
            Resource* res = low_priority->held_resources[i];
            release_resource(low_priority, res);
            allocate_resource(high_priority, res);
        }
        
        // Low priority process must restart
        restart_process(low_priority);
    }
}
```

### Method 4: Eliminate Circular Wait
```c
// Ordered resource allocation
enum ResourceOrder {
    RESOURCE_A = 1,
    RESOURCE_B = 2,
    RESOURCE_C = 3,
    RESOURCE_D = 4
};

void ordered_allocation_example() {
    // Always acquire resources in ascending order
    lock(RESOURCE_A);  // Order: 1
    lock(RESOURCE_B);  // Order: 2
    lock(RESOURCE_C);  // Order: 3
    
    // Do work
    
    // Release in reverse order (optional but good practice)
    unlock(RESOURCE_C);
    unlock(RESOURCE_B);
    unlock(RESOURCE_A);
}

// This prevents circular wait as all processes follow same order
```

## Deadlock Avoidance - Banker's Algorithm

```c
#define MAX_PROCESSES 5
#define MAX_RESOURCES 3

typedef struct {
    int allocation[MAX_PROCESSES][MAX_RESOURCES];     // Currently allocated
    int max_need[MAX_PROCESSES][MAX_RESOURCES];       // Maximum need
    int available[MAX_RESOURCES];                     // Available resources
    int need[MAX_PROCESSES][MAX_RESOURCES];           // Remaining need
} BankersAlgorithm;

void calculate_need(BankersAlgorithm* bank) {
    for (int i = 0; i < MAX_PROCESSES; i++) {
        for (int j = 0; j < MAX_RESOURCES; j++) {
            bank->need[i][j] = bank->max_need[i][j] - bank->allocation[i][j];
        }
    }
}

bool is_safe_state(BankersAlgorithm* bank) {
    int work[MAX_RESOURCES];
    bool finish[MAX_PROCESSES] = {false};
    int safe_sequence[MAX_PROCESSES];
    int count = 0;
    
    // Initialize work with available resources
    for (int i = 0; i < MAX_RESOURCES; i++) {
        work[i] = bank->available[i];
    }
    
    while (count < MAX_PROCESSES) {
        bool found = false;
        
        for (int p = 0; p < MAX_PROCESSES; p++) {
            if (!finish[p]) {
                // Check if process can complete with available resources
                bool can_complete = true;
                for (int j = 0; j < MAX_RESOURCES; j++) {
                    if (bank->need[p][j] > work[j]) {
                        can_complete = false;
                        break;
                    }
                }
                
                if (can_complete) {
                    // Process can complete, add its allocation to work
                    for (int k = 0; k < MAX_RESOURCES; k++) {
                        work[k] += bank->allocation[p][k];
                    }
                    
                    safe_sequence[count++] = p;
                    finish[p] = true;
                    found = true;
                }
            }
        }
        
        if (!found) {
            return false;  // Unsafe state
        }
    }
    
    printf("Safe sequence: ");
    for (int i = 0; i < MAX_PROCESSES; i++) {
        printf("P%d ", safe_sequence[i]);
    }
    printf("\n");
    
    return true;  // Safe state
}

bool request_resources(BankersAlgorithm* bank, int process_id, int request[]) {
    // Check if request is valid
    for (int i = 0; i < MAX_RESOURCES; i++) {
        if (request[i] > bank->need[process_id][i] || 
            request[i] > bank->available[i]) {
            return false;  // Invalid request
        }
    }
    
    // Temporarily allocate resources
    for (int i = 0; i < MAX_RESOURCES; i++) {
        bank->available[i] -= request[i];
        bank->allocation[process_id][i] += request[i];
        bank->need[process_id][i] -= request[i];
    }
    
    // Check if state is safe
    if (is_safe_state(bank)) {
        return true;  // Grant request
    } else {
        // Rollback allocation
        for (int i = 0; i < MAX_RESOURCES; i++) {
            bank->available[i] += request[i];
            bank->allocation[process_id][i] -= request[i];
            bank->need[process_id][i] += request[i];
        }
        return false;  // Deny request
    }
}

// Example usage
void bankers_example() {
    BankersAlgorithm bank = {
        .allocation = {{0,1,0}, {2,0,0}, {3,0,2}, {2,1,1}, {0,0,2}},
        .max_need = {{7,5,3}, {3,2,2}, {9,0,2}, {2,2,2}, {4,3,3}},
        .available = {3,3,2}
    };
    
    calculate_need(&bank);
    
    if (is_safe_state(&bank)) {
        printf("System is in safe state\n");
        
        int request[] = {1,0,2};
        if (request_resources(&bank, 1, request)) {
            printf("Request granted\n");
        } else {
            printf("Request denied - would lead to unsafe state\n");
        }
    } else {
        printf("System is in unsafe state\n");
    }
}
```

## Deadlock Detection

### Wait-for Graph Algorithm
```c
#define MAX_PROCESSES 10

typedef struct {
    bool wait_for[MAX_PROCESSES][MAX_PROCESSES];  // wait_for[i][j] = true if Pi waits for Pj
    int num_processes;
} WaitForGraph;

bool has_cycle_util(WaitForGraph* graph, int v, bool visited[], bool rec_stack[]) {
    visited[v] = true;
    rec_stack[v] = true;
    
    for (int u = 0; u < graph->num_processes; u++) {
        if (graph->wait_for[v][u]) {
            if (!visited[u] && has_cycle_util(graph, u, visited, rec_stack)) {
                return true;
            } else if (rec_stack[u]) {
                return true;  // Back edge found - cycle detected
            }
        }
    }
    
    rec_stack[v] = false;
    return false;
}

bool detect_deadlock(WaitForGraph* graph) {
    bool visited[MAX_PROCESSES] = {false};
    bool rec_stack[MAX_PROCESSES] = {false};
    
    for (int i = 0; i < graph->num_processes; i++) {
        if (!visited[i]) {
            if (has_cycle_util(graph, i, visited, rec_stack)) {
                return true;  // Deadlock detected
            }
        }
    }
    
    return false;  // No deadlock
}

void print_deadlock_cycle(WaitForGraph* graph) {
    // Find and print the processes involved in deadlock
    for (int i = 0; i < graph->num_processes; i++) {
        for (int j = 0; j < graph->num_processes; j++) {
            if (graph->wait_for[i][j]) {
                printf("P%d waits for P%d\n", i, j);
            }
        }
    }
}
```

## Deadlock Recovery

### Method 1: Process Termination
```c
typedef struct {
    int process_id;
    int priority;
    int resources_held;
    int computation_done;  // Percentage of work completed
} ProcessInfo;

// Terminate all deadlocked processes
void terminate_all_deadlocked(int deadlocked_processes[], int count) {
    for (int i = 0; i < count; i++) {
        printf("Terminating process P%d\n", deadlocked_processes[i]);
        kill_process(deadlocked_processes[i]);
        release_all_resources(deadlocked_processes[i]);
    }
}

// Terminate processes one by one until deadlock is broken
int terminate_minimum_cost(ProcessInfo processes[], int deadlocked[], int count) {
    // Sort by cost (priority, computation done, resources held)
    for (int i = 0; i < count - 1; i++) {
        for (int j = 0; j < count - i - 1; j++) {
            ProcessInfo p1 = processes[deadlocked[j]];
            ProcessInfo p2 = processes[deadlocked[j + 1]];
            
            // Lower priority, less work done, fewer resources = lower cost
            int cost1 = p1.priority + p1.computation_done + p1.resources_held;
            int cost2 = p2.priority + p2.computation_done + p2.resources_held;
            
            if (cost1 > cost2) {
                // Swap
                int temp = deadlocked[j];
                deadlocked[j] = deadlocked[j + 1];
                deadlocked[j + 1] = temp;
            }
        }
    }
    
    // Terminate lowest cost process first
    int victim = deadlocked[0];
    kill_process(victim);
    release_all_resources(victim);
    
    return victim;
}
```

### Method 2: Resource Preemption
```c
typedef struct {
    int resource_id;
    int holder_process;
    int requester_process;
    int preemption_cost;
} PreemptionCandidate;

int select_victim_for_preemption(PreemptionCandidate candidates[], int count) {
    int min_cost = INT_MAX;
    int victim = -1;
    
    for (int i = 0; i < count; i++) {
        if (candidates[i].preemption_cost < min_cost) {
            min_cost = candidates[i].preemption_cost;
            victim = i;
        }
    }
    
    return victim;
}

void preempt_resource(int resource_id, int from_process, int to_process) {
    printf("Preempting resource R%d from P%d to P%d\n", 
           resource_id, from_process, to_process);
    
    // Save state of victim process
    save_process_state(from_process);
    
    // Take away resource
    deallocate_resource(from_process, resource_id);
    allocate_resource(to_process, resource_id);
    
    // Rollback victim process to safe checkpoint
    rollback_process(from_process);
}
```

## Practical Example: Dining Philosophers Solution

```c
#include <pthread.h>
#include <semaphore.h>

#define N 5  // Number of philosophers

sem_t forks[N];
sem_t room;  // Limits philosophers in room to prevent deadlock

void* philosopher(void* arg) {
    int id = *(int*)arg;
    
    while (1) {
        // Think
        printf("Philosopher %d is thinking\n", id);
        sleep(rand() % 3);
        
        // Enter room (deadlock prevention)
        sem_wait(&room);
        
        // Pick up forks (ordered to prevent circular wait)
        int left = id;
        int right = (id + 1) % N;
        
        if (id % 2 == 0) {  // Even numbered philosophers
            sem_wait(&forks[left]);   // Pick up left fork first
            sem_wait(&forks[right]);  // Then right fork
        } else {  // Odd numbered philosophers
            sem_wait(&forks[right]);  // Pick up right fork first
            sem_wait(&forks[left]);   // Then left fork
        }
        
        // Eat
        printf("Philosopher %d is eating\n", id);
        sleep(rand() % 3);
        
        // Put down forks
        sem_post(&forks[left]);
        sem_post(&forks[right]);
        
        // Leave room
        sem_post(&room);
    }
    
    return NULL;
}

int main() {
    pthread_t philosophers[N];
    int ids[N];
    
    // Initialize semaphores
    for (int i = 0; i < N; i++) {
        sem_init(&forks[i], 0, 1);
        ids[i] = i;
    }
    sem_init(&room, 0, N-1);  // Allow only N-1 philosophers in room
    
    // Create philosopher threads
    for (int i = 0; i < N; i++) {
        pthread_create(&philosophers[i], NULL, philosopher, &ids[i]);
    }
    
    // Wait for threads
    for (int i = 0; i < N; i++) {
        pthread_join(philosophers[i], NULL);
    }
    
    return 0;
}
```

## Common Interview Questions

### Q1: What are the four conditions necessary for deadlock to occur?
**Answer**: The four Coffman conditions are:
1. **Mutual Exclusion**: At least one resource must be held in non-shareable mode
2. **Hold and Wait**: Processes must hold resources while waiting for additional resources
3. **No Preemption**: Resources cannot be forcibly removed from processes
4. **Circular Wait**: There must be a circular chain of processes waiting for resources
All four conditions must be present simultaneously for deadlock to occur.

### Q2: What is the difference between deadlock prevention and deadlock avoidance?
**Answer**: 
- **Prevention**: Ensures at least one of the four necessary conditions never occurs (static approach)
- **Avoidance**: Allows all four conditions but makes intelligent decisions to avoid unsafe states (dynamic approach)
Prevention is more restrictive but simpler; avoidance is more flexible but requires advance knowledge of resource requirements.

### Q3: Explain the Banker's Algorithm with an example.
**Answer**: Banker's Algorithm ensures system never enters unsafe state by checking if granting a resource request leaves the system in a safe state. 

Example with 3 processes, 3 resource types:
```
Available: [3,3,2]
Allocation: P0[0,1,0], P1[2,0,0], P2[3,0,2]
Max Need: P0[7,5,3], P1[3,2,2], P2[9,0,2]
Need: P0[7,4,3], P1[1,2,2], P2[6,0,0]
```

Safe sequence might be P1→P0→P2 if each can complete with available resources.

### Q4: How do you detect deadlock in a system?
**Answer**: Common methods include:
1. **Wait-for graph**: For single instance resources, create graph where edge P1→P2 means P1 waits for P2. Cycle indicates deadlock.
2. **Resource allocation graph**: For multiple instances, more complex analysis needed.
3. **Banker's algorithm detection**: Run periodically to check if system is in safe state.
Detection overhead must be balanced against detection frequency.

### Q5: What are the trade-offs between different deadlock handling approaches?
**Answer**: 
- **Prevention**: Simple but inefficient resource utilization, may reduce concurrency
- **Avoidance**: Better resource utilization but requires advance information and has runtime overhead
- **Detection + Recovery**: Allows deadlocks but has recovery costs (process termination/rollback)
- **Ignore (Ostrich approach)**: Lowest overhead but system may hang, used when deadlocks are rare

---
[← Back to Main Guide](./README.md)
