# Classic Synchronization Problems

Classic synchronization problems are well-known scenarios that demonstrate common challenges in concurrent programming. These problems help understand different synchronization techniques and their trade-offs.

## Key Concepts

- **Producer-Consumer**: Synchronizing data production and consumption with bounded buffer
- **Readers-Writers**: Multiple readers vs exclusive writers access to shared data
- **Dining Philosophers**: Avoiding deadlock when multiple processes need multiple resources
- **Sleeping Barber**: Service provider with limited capacity and waiting customers
- **Cigarette Smokers**: Complex resource dependencies and signaling
- **Bounded Buffer**: Fixed-size buffer shared between producers and consumers

## 1. Producer-Consumer Problem

### Problem Description
Multiple producer threads generate data items and place them in a shared buffer. Multiple consumer threads remove items from the buffer. Buffer has fixed size and must handle concurrent access safely.

### Solution with Semaphores
```c
#define BUFFER_SIZE 10

// Shared variables
int buffer[BUFFER_SIZE];
int in = 0, out = 0;

// Semaphores
sem_t empty;      // Count of empty slots (initially BUFFER_SIZE)
sem_t full;       // Count of full slots (initially 0)
sem_t mutex;      // Mutual exclusion for buffer access (initially 1)

void producer() {
    while (true) {
        int item = produce_item();
        
        sem_wait(&empty);     // Wait for empty slot
        sem_wait(&mutex);     // Enter critical section
        
        buffer[in] = item;    // Add item to buffer
        in = (in + 1) % BUFFER_SIZE;
        
        sem_post(&mutex);     // Exit critical section
        sem_post(&full);      // Signal item is available
    }
}

void consumer() {
    while (true) {
        sem_wait(&full);      // Wait for available item
        sem_wait(&mutex);     // Enter critical section
        
        int item = buffer[out]; // Remove item from buffer
        out = (out + 1) % BUFFER_SIZE;
        
        sem_post(&mutex);     // Exit critical section
        sem_post(&empty);     // Signal empty slot
        
        consume_item(item);
    }
}

// Initialization
sem_init(&empty, 0, BUFFER_SIZE);
sem_init(&full, 0, 0);
sem_init(&mutex, 0, 1);
```

## 2. Readers-Writers Problem

### Problem Description
Multiple processes want to access shared data. Readers only read data (multiple readers can read simultaneously). Writers modify data (must have exclusive access, no readers or other writers).

### First Readers-Writers Solution (Readers Priority)
```c
sem_t write_mutex;        // Mutual exclusion for writers
sem_t read_count_mutex;   // Mutual exclusion for read_count
int read_count = 0;       // Number of current readers

void reader() {
    while (true) {
        // Entry section
        sem_wait(&read_count_mutex);
        read_count++;
        if (read_count == 1) {
            sem_wait(&write_mutex);  // First reader blocks writers
        }
        sem_post(&read_count_mutex);
        
        // Reading section
        read_data();
        
        // Exit section
        sem_wait(&read_count_mutex);
        read_count--;
        if (read_count == 0) {
            sem_post(&write_mutex);  // Last reader allows writers
        }
        sem_post(&read_count_mutex);
        
        // Remainder section
        other_work();
    }
}

void writer() {
    while (true) {
        // Entry section
        sem_wait(&write_mutex);
        
        // Writing section
        write_data();
        
        // Exit section
        sem_post(&write_mutex);
        
        // Remainder section
        other_work();
    }
}
```

### Second Readers-Writers Solution (Writers Priority)
```c
sem_t read_mutex;         // Mutual exclusion for readers
sem_t write_mutex;        // Mutual exclusion for writers
sem_t reader_queue;       // Queue for readers when writer is waiting
int read_count = 0;
int write_count = 0;

void writer() {
    sem_wait(&write_count_mutex);
    write_count++;
    if (write_count == 1) {
        sem_wait(&reader_queue);  // Block new readers
    }
    sem_post(&write_count_mutex);
    
    sem_wait(&write_mutex);
    
    // Writing
    write_data();
    
    sem_post(&write_mutex);
    
    sem_wait(&write_count_mutex);
    write_count--;
    if (write_count == 0) {
        sem_post(&reader_queue);  // Allow readers
    }
    sem_post(&write_count_mutex);
}
```

## 3. Dining Philosophers Problem

### Problem Description
Five philosophers sit around a circular table. Each philosopher thinks and eats. To eat, a philosopher needs both left and right chopsticks. Only one philosopher can use each chopstick at a time.

### Solution 1: Avoid Deadlock by Ordering
```c
#define N 5
sem_t chopstick[N];

void philosopher(int i) {
    while (true) {
        think();
        
        // Pick up chopsticks in order to avoid deadlock
        if (i < (i + 1) % N) {
            sem_wait(&chopstick[i]);        // Left chopstick
            sem_wait(&chopstick[(i + 1) % N]); // Right chopstick
        } else {
            sem_wait(&chopstick[(i + 1) % N]); // Right chopstick
            sem_wait(&chopstick[i]);        // Left chopstick
        }
        
        eat();
        
        sem_post(&chopstick[i]);
        sem_post(&chopstick[(i + 1) % N]);
    }
}
```

### Solution 2: Maximum N-1 Philosophers
```c
sem_t chopstick[N];
sem_t dining_room;  // Allow only N-1 philosophers to try eating

void philosopher(int i) {
    while (true) {
        think();
        
        sem_wait(&dining_room);    // Enter dining room
        sem_wait(&chopstick[i]);   // Left chopstick
        sem_wait(&chopstick[(i + 1) % N]); // Right chopstick
        
        eat();
        
        sem_post(&chopstick[i]);
        sem_post(&chopstick[(i + 1) % N]);
        sem_post(&dining_room);    // Leave dining room
    }
}

// Initialize
for (int i = 0; i < N; i++) {
    sem_init(&chopstick[i], 0, 1);
}
sem_init(&dining_room, 0, N - 1);  // Allow N-1 philosophers
```

## 4. Sleeping Barber Problem

### Problem Description
Barber shop has one barber, one barber chair, and N chairs for waiting customers. If no customers, barber sleeps. If barber is busy, customer waits or leaves if no chairs available.

### Solution
```c
#define CHAIRS 3

sem_t customers;      // Number of customers waiting
sem_t barber;         // Barber availability
sem_t access_seats;   // Mutual exclusion for waiting chairs
int waiting = 0;      // Number of customers waiting

void barber() {
    while (true) {
        sem_wait(&customers);     // Sleep until customer arrives
        
        sem_wait(&access_seats);
        waiting--;                // One less customer waiting
        sem_post(&barber);        // Barber is ready
        sem_post(&access_seats);
        
        cut_hair();               // Cut customer's hair
    }
}

void customer() {
    sem_wait(&access_seats);
    
    if (waiting < CHAIRS) {
        waiting++;                // Sit in waiting chair
        sem_post(&customers);     // Wake up barber
        sem_post(&access_seats);
        
        sem_wait(&barber);        // Wait for barber
        get_haircut();            // Get hair cut
    } else {
        sem_post(&access_seats);
        leave();                  // No chairs available, leave
    }
}
```

## 5. Cigarette Smokers Problem

### Problem Description
Three smokers and one agent. Each smoker has infinite supply of one ingredient (tobacco, paper, or matches). Agent randomly places two different ingredients on table. Smoker with third ingredient picks them up, makes cigarette, and smokes.

### Solution
```c
sem_t agent;          // Agent semaphore
sem_t tobacco;        // Tobacco smoker semaphore
sem_t paper;          // Paper smoker semaphore
sem_t match;          // Match smoker semaphore

void agent_process() {
    while (true) {
        sem_wait(&agent);
        
        int random = rand() % 3;
        if (random == 0) {
            sem_post(&tobacco);   // Signal tobacco smoker
        } else if (random == 1) {
            sem_post(&paper);     // Signal paper smoker
        } else {
            sem_post(&match);     // Signal match smoker
        }
    }
}

void smoker_with_tobacco() {
    while (true) {
        sem_wait(&tobacco);
        make_cigarette();
        sem_post(&agent);         // Signal agent
        smoke();
    }
}

void smoker_with_paper() {
    while (true) {
        sem_wait(&paper);
        make_cigarette();
        sem_post(&agent);
        smoke();
    }
}

void smoker_with_matches() {
    while (true) {
        sem_wait(&match);
        make_cigarette();
        sem_post(&agent);
        smoke();
    }
}
```

## Common Interview Questions

### Q1: Why is ordering important in the dining philosophers problem?
**Answer**: Without ordering, deadlock can occur when all philosophers pick up their left chopstick simultaneously. By imposing a consistent ordering (like always picking lower-numbered chopstick first), we break the circular wait condition that's necessary for deadlock.

### Q2: What's the difference between the first and second readers-writers solutions?
**Answer**: 
- **First solution (Readers priority)**: Readers never wait for other readers, but writers may starve if readers keep arriving
- **Second solution (Writers priority)**: Writers get priority, but readers may starve if writers keep arriving
- **Third solution**: Fair solution where neither readers nor writers starve

### Q3: In the sleeping barber problem, why do we need the access_seats semaphore?
**Answer**: The `access_seats` semaphore provides mutual exclusion for the `waiting` variable. Without it, multiple customers could read and modify `waiting` simultaneously, causing race conditions and incorrect behavior (customers might think chairs are available when they're not).

### Q4: How would you modify producer-consumer to handle multiple producers and consumers?
**Answer**: The given solution already handles multiple producers and consumers correctly. The `mutex` semaphore ensures only one thread accesses the buffer at a time, while `empty` and `full` semaphores coordinate between all producers and consumers regardless of their number.

### Q5: What happens if you swap the order of sem_wait() calls in producer-consumer?
**Answer**: Swapping order can cause deadlock:
```c
// WRONG order - can deadlock
sem_wait(&mutex);    // Get exclusive access first
sem_wait(&empty);    // Then wait for empty slot
```
If buffer is full, producer holds mutex and waits for empty slot, but consumer can't run to create empty slot because it can't get mutex. Always acquire resource semaphores before mutual exclusion semaphores.

---
[← Back to Main Guide](./README.md)
