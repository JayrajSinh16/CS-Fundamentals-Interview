# Memory Allocation Techniques

Memory allocation is the process of reserving a portion of computer memory for program use. Understanding different allocation strategies, their trade-offs, and implementation details is crucial for system design and performance optimization.

## Key Concepts

- **Static Allocation**: Memory allocated at compile time
- **Dynamic Allocation**: Memory allocated at runtime
- **Memory Pool**: Pre-allocated memory blocks for efficient allocation
- **Fragmentation**: Wasted memory due to allocation patterns
- **Coalescing**: Merging adjacent free blocks
- **Compaction**: Moving allocated blocks to eliminate fragmentation

## Memory Layout

```
High Memory
┌─────────────────┐
│      Stack      │ ← Grows downward (local variables, function calls)
│        ↓        │
├─────────────────┤
│                 │
│   Free Space    │
│                 │
├─────────────────┤
│        ↑        │
│      Heap       │ ← Grows upward (dynamic allocation)
├─────────────────┤
│   Uninitialized │ ← BSS segment (zero-initialized globals)
│   Data (BSS)    │
├─────────────────┤
│   Initialized   │ ← Data segment (initialized globals)
│     Data        │
├─────────────────┤
│      Text       │ ← Code segment (program instructions)
│   (Code/Text)   │
└─────────────────┘
Low Memory
```

## Static vs Dynamic Allocation

### Static Allocation
```c
// Compile-time allocation
int global_array[1000];           // Global static allocation
static int static_var = 42;      // Static storage duration

void function_example() {
    int local_array[100];         // Stack allocation (automatic)
    static int func_static = 0;   // Static allocation in function
    
    // Memory size known at compile time
    // Automatic cleanup when scope ends
    // Fast allocation/deallocation
    // Limited flexibility
}
```

### Dynamic Allocation
```c
#include <stdlib.h>

void dynamic_allocation_example() {
    // Runtime allocation
    int* dynamic_array = (int*)malloc(1000 * sizeof(int));
    if (dynamic_array == NULL) {
        fprintf(stderr, "Memory allocation failed\n");
        return;
    }
    
    // Use the memory
    for (int i = 0; i < 1000; i++) {
        dynamic_array[i] = i * i;
    }
    
    // Manual cleanup required
    free(dynamic_array);
    dynamic_array = NULL;  // Prevent dangling pointer
    
    // More flexible but slower
    // Risk of memory leaks
    // Fragmentation issues
}
```

## Heap Management Algorithms

### 1. First Fit
**Allocate first block that is large enough**

```c
typedef struct Block {
    size_t size;
    int is_free;
    struct Block* next;
} Block;

Block* heap_start = NULL;

void* first_fit_malloc(size_t size) {
    Block* current = heap_start;
    
    // Search for first suitable block
    while (current != NULL) {
        if (current->is_free && current->size >= size) {
            current->is_free = 0;
            
            // Split block if it's much larger
            if (current->size > size + sizeof(Block) + 16) {
                Block* new_block = (Block*)((char*)current + sizeof(Block) + size);
                new_block->size = current->size - size - sizeof(Block);
                new_block->is_free = 1;
                new_block->next = current->next;
                
                current->size = size;
                current->next = new_block;
            }
            
            return (char*)current + sizeof(Block);
        }
        current = current->next;
    }
    
    // No suitable block found, allocate new one
    return allocate_new_block(size);
}

void first_fit_free(void* ptr) {
    if (ptr == NULL) return;
    
    Block* block = (Block*)((char*)ptr - sizeof(Block));
    block->is_free = 1;
    
    // Coalesce with next block if free
    if (block->next && block->next->is_free) {
        block->size += sizeof(Block) + block->next->size;
        block->next = block->next->next;
    }
    
    // Coalesce with previous block (requires additional tracking)
    coalesce_with_previous(block);
}
```

### 2. Best Fit
**Allocate smallest block that is large enough**

```c
void* best_fit_malloc(size_t size) {
    Block* current = heap_start;
    Block* best_block = NULL;
    size_t smallest_suitable_size = SIZE_MAX;
    
    // Search for best fitting block
    while (current != NULL) {
        if (current->is_free && current->size >= size) {
            if (current->size < smallest_suitable_size) {
                smallest_suitable_size = current->size;
                best_block = current;
            }
        }
        current = current->next;
    }
    
    if (best_block != NULL) {
        best_block->is_free = 0;
        
        // Split if necessary
        if (best_block->size > size + sizeof(Block) + 16) {
            split_block(best_block, size);
        }
        
        return (char*)best_block + sizeof(Block);
    }
    
    return allocate_new_block(size);
}

// Best fit reduces wasted space but is slower due to full search
```

### 3. Worst Fit
**Allocate largest available block**

```c
void* worst_fit_malloc(size_t size) {
    Block* current = heap_start;
    Block* worst_block = NULL;
    size_t largest_size = 0;
    
    // Search for largest block
    while (current != NULL) {
        if (current->is_free && current->size >= size) {
            if (current->size > largest_size) {
                largest_size = current->size;
                worst_block = current;
            }
        }
        current = current->next;
    }
    
    if (worst_block != NULL) {
        worst_block->is_free = 0;
        
        // Split block (likely to leave large remainder)
        if (worst_block->size > size + sizeof(Block) + 16) {
            split_block(worst_block, size);
        }
        
        return (char*)worst_block + sizeof(Block);
    }
    
    return allocate_new_block(size);
}

// Worst fit aims to leave large usable blocks but causes fragmentation
```

### 4. Next Fit
**First fit starting from last allocation point**

```c
static Block* last_allocated = NULL;

void* next_fit_malloc(size_t size) {
    Block* start = (last_allocated != NULL) ? last_allocated : heap_start;
    Block* current = start;
    
    // Search from last allocation point
    do {
        if (current->is_free && current->size >= size) {
            current->is_free = 0;
            last_allocated = current;
            
            if (current->size > size + sizeof(Block) + 16) {
                split_block(current, size);
            }
            
            return (char*)current + sizeof(Block);
        }
        
        current = current->next;
        if (current == NULL) {
            current = heap_start;  // Wrap around
        }
    } while (current != start);
    
    return allocate_new_block(size);
}

// Next fit is faster than first fit but can cause more fragmentation
```

## Advanced Allocation Strategies

### Memory Pools (Fixed-Size Allocation)
```c
typedef struct MemoryPool {
    void* pool_start;
    size_t block_size;
    size_t num_blocks;
    unsigned char* free_bitmap;
} MemoryPool;

MemoryPool* create_memory_pool(size_t block_size, size_t num_blocks) {
    MemoryPool* pool = malloc(sizeof(MemoryPool));
    
    pool->block_size = block_size;
    pool->num_blocks = num_blocks;
    
    // Allocate pool memory
    pool->pool_start = malloc(block_size * num_blocks);
    
    // Allocate and initialize bitmap
    size_t bitmap_size = (num_blocks + 7) / 8;  // Round up to bytes
    pool->free_bitmap = calloc(bitmap_size, 1);  // All blocks initially free
    
    return pool;
}

void* pool_alloc(MemoryPool* pool) {
    // Find first free block
    for (size_t i = 0; i < pool->num_blocks; i++) {
        size_t byte_index = i / 8;
        size_t bit_index = i % 8;
        
        if (!(pool->free_bitmap[byte_index] & (1 << bit_index))) {
            // Block is free, mark as allocated
            pool->free_bitmap[byte_index] |= (1 << bit_index);
            
            return (char*)pool->pool_start + (i * pool->block_size);
        }
    }
    
    return NULL;  // Pool is full
}

void pool_free(MemoryPool* pool, void* ptr) {
    if (ptr < pool->pool_start) return;
    
    size_t offset = (char*)ptr - (char*)pool->pool_start;
    size_t block_index = offset / pool->block_size;
    
    if (block_index >= pool->num_blocks) return;
    
    // Mark block as free
    size_t byte_index = block_index / 8;
    size_t bit_index = block_index % 8;
    pool->free_bitmap[byte_index] &= ~(1 << bit_index);
}

// Benefits: O(1) allocation, no fragmentation, cache-friendly
// Drawbacks: Fixed size, potential waste for varied sizes
```

### Buddy System
```c
#define MAX_ORDER 10  // 2^10 = 1024 byte maximum block

typedef struct BuddySystem {
    void* memory_start;
    size_t total_size;
    struct Block* free_lists[MAX_ORDER + 1];
} BuddySystem;

size_t get_order(size_t size) {
    size_t order = 0;
    size_t block_size = 1;
    
    while (block_size < size && order < MAX_ORDER) {
        block_size <<= 1;
        order++;
    }
    
    return order;
}

void* buddy_alloc(BuddySystem* buddy, size_t size) {
    size_t order = get_order(size);
    
    // Find smallest available block
    for (size_t current_order = order; current_order <= MAX_ORDER; current_order++) {
        if (buddy->free_lists[current_order] != NULL) {
            Block* block = buddy->free_lists[current_order];
            buddy->free_lists[current_order] = block->next;
            
            // Split block down to required size
            while (current_order > order) {
                current_order--;
                
                // Create buddy block
                size_t buddy_size = 1 << current_order;
                Block* buddy_block = (Block*)((char*)block + buddy_size);
                buddy_block->size = buddy_size;
                buddy_block->next = buddy->free_lists[current_order];
                buddy->free_lists[current_order] = buddy_block;
            }
            
            block->size = 1 << order;
            return (char*)block + sizeof(Block);
        }
    }
    
    return NULL;  // No memory available
}

void buddy_free(BuddySystem* buddy, void* ptr) {
    Block* block = (Block*)((char*)ptr - sizeof(Block));
    size_t order = get_order(block->size);
    
    // Coalesce with buddy blocks
    while (order < MAX_ORDER) {
        size_t buddy_offset = ((char*)block - (char*)buddy->memory_start) ^ (1 << order);
        Block* buddy_block = (Block*)((char*)buddy->memory_start + buddy_offset);
        
        // Check if buddy is free and same size
        if (!is_free_in_list(buddy->free_lists[order], buddy_block)) {
            break;  // Buddy not free
        }
        
        // Remove buddy from free list
        remove_from_free_list(&buddy->free_lists[order], buddy_block);
        
        // Merge with buddy
        if (buddy_block < block) {
            block = buddy_block;
        }
        
        order++;
        block->size = 1 << order;
    }
    
    // Add merged block to appropriate free list
    block->next = buddy->free_lists[order];
    buddy->free_lists[order] = block;
}
```

### Slab Allocation
```c
typedef struct Slab {
    void* memory;
    size_t object_size;
    size_t objects_per_slab;
    struct Slab* next;
    unsigned char* allocation_bitmap;
} Slab;

typedef struct SlabCache {
    size_t object_size;
    Slab* partial_slabs;   // Slabs with some free objects
    Slab* full_slabs;      // Slabs with no free objects
    Slab* empty_slabs;     // Slabs with all objects free
} SlabCache;

Slab* create_slab(size_t object_size, size_t slab_size) {
    Slab* slab = malloc(sizeof(Slab));
    slab->object_size = object_size;
    slab->objects_per_slab = slab_size / object_size;
    slab->memory = malloc(slab_size);
    slab->next = NULL;
    
    size_t bitmap_size = (slab->objects_per_slab + 7) / 8;
    slab->allocation_bitmap = calloc(bitmap_size, 1);
    
    return slab;
}

void* slab_alloc(SlabCache* cache) {
    Slab* slab = cache->partial_slabs;
    
    if (slab == NULL) {
        // Create new slab
        slab = create_slab(cache->object_size, 4096);  // 4KB slab
        cache->partial_slabs = slab;
    }
    
    // Find free object in slab
    for (size_t i = 0; i < slab->objects_per_slab; i++) {
        size_t byte_index = i / 8;
        size_t bit_index = i % 8;
        
        if (!(slab->allocation_bitmap[byte_index] & (1 << bit_index))) {
            // Object is free
            slab->allocation_bitmap[byte_index] |= (1 << bit_index);
            
            // Check if slab is now full
            if (is_slab_full(slab)) {
                // Move to full list
                cache->partial_slabs = slab->next;
                slab->next = cache->full_slabs;
                cache->full_slabs = slab;
            }
            
            return (char*)slab->memory + (i * cache->object_size);
        }
    }
    
    return NULL;  // Should not happen if properly managed
}

// Benefits: Reduces fragmentation, good cache locality, fast allocation
// Used in kernel object allocation (kmalloc)
```

## Garbage Collection Basics

### Reference Counting
```c
typedef struct Object {
    int ref_count;
    void* data;
    void (*destructor)(void*);
} Object;

Object* create_object(void* data, void (*destructor)(void*)) {
    Object* obj = malloc(sizeof(Object));
    obj->ref_count = 1;
    obj->data = data;
    obj->destructor = destructor;
    return obj;
}

void retain_object(Object* obj) {
    if (obj != NULL) {
        obj->ref_count++;
    }
}

void release_object(Object* obj) {
    if (obj != NULL) {
        obj->ref_count--;
        if (obj->ref_count == 0) {
            if (obj->destructor) {
                obj->destructor(obj->data);
            }
            free(obj);
        }
    }
}

// Problem: Cannot handle circular references
// A -> B -> A will never be freed
```

### Mark and Sweep (Simplified)
```c
typedef struct GCObject {
    int marked;
    struct GCObject* next;
    void* data;
} GCObject;

static GCObject* all_objects = NULL;
static GCObject** roots = NULL;
static int num_roots = 0;

void mark_phase() {
    // Clear all marks
    GCObject* obj = all_objects;
    while (obj != NULL) {
        obj->marked = 0;
        obj = obj->next;
    }
    
    // Mark from roots
    for (int i = 0; i < num_roots; i++) {
        mark_recursive(roots[i]);
    }
}

void mark_recursive(GCObject* obj) {
    if (obj == NULL || obj->marked) return;
    
    obj->marked = 1;
    
    // Mark all objects referenced by this object
    // (Implementation depends on object structure)
    mark_references(obj);
}

void sweep_phase() {
    GCObject** current = &all_objects;
    
    while (*current != NULL) {
        GCObject* obj = *current;
        
        if (!obj->marked) {
            // Unreachable object - free it
            *current = obj->next;
            free(obj->data);
            free(obj);
        } else {
            current = &obj->next;
        }
    }
}

void garbage_collect() {
    mark_phase();
    sweep_phase();
}
```

## Performance Comparison

```c
// Benchmark different allocation strategies
#include <time.h>

void benchmark_allocation() {
    const int NUM_ALLOCS = 100000;
    clock_t start, end;
    
    // First Fit benchmark
    start = clock();
    for (int i = 0; i < NUM_ALLOCS; i++) {
        void* ptr = first_fit_malloc(rand() % 1000 + 1);
        if (i % 2 == 0) first_fit_free(ptr);  // Free some randomly
    }
    end = clock();
    printf("First Fit: %f seconds\n", ((double)(end - start)) / CLOCKS_PER_SEC);
    
    // Best Fit benchmark
    start = clock();
    for (int i = 0; i < NUM_ALLOCS; i++) {
        void* ptr = best_fit_malloc(rand() % 1000 + 1);
        if (i % 2 == 0) best_fit_free(ptr);
    }
    end = clock();
    printf("Best Fit: %f seconds\n", ((double)(end - start)) / CLOCKS_PER_SEC);
    
    // Memory Pool benchmark (fixed size)
    MemoryPool* pool = create_memory_pool(512, NUM_ALLOCS);
    start = clock();
    for (int i = 0; i < NUM_ALLOCS; i++) {
        void* ptr = pool_alloc(pool);
        if (i % 2 == 0) pool_free(pool, ptr);
    }
    end = clock();
    printf("Memory Pool: %f seconds\n", ((double)(end - start)) / CLOCKS_PER_SEC);
}

/*
Typical Results:
First Fit: ~0.15 seconds (moderate speed, moderate fragmentation)
Best Fit: ~0.45 seconds (slower, less fragmentation)
Memory Pool: ~0.02 seconds (fastest, no fragmentation for fixed sizes)
*/
```

## Common Interview Questions

### Q1: What is the difference between internal and external fragmentation?
**Answer**: 
- **Internal Fragmentation**: Wasted space within allocated blocks (e.g., allocating 64-byte block for 50-byte request leaves 14 bytes unused)
- **External Fragmentation**: Free memory exists but isn't contiguous enough for allocation requests

Internal fragmentation is fixed-size waste per allocation; external fragmentation prevents large allocations despite sufficient total free memory.

### Q2: Compare First Fit, Best Fit, and Worst Fit allocation strategies.
**Answer**: 
- **First Fit**: Fast allocation, moderate fragmentation, simple implementation
- **Best Fit**: Slower allocation (full search), minimizes wasted space, may create many small unusable blocks
- **Worst Fit**: Slower allocation, creates large free blocks but causes more overall fragmentation

Best fit minimizes immediate waste but may not be optimal long-term due to small fragment creation.

### Q3: How does the Buddy System work and what are its advantages?
**Answer**: Buddy System splits memory into power-of-2 sized blocks. When allocating, it finds the smallest suitable block and splits larger blocks as needed. When freeing, it merges with "buddy" blocks (adjacent blocks of same size from same split).

**Advantages**: Fast allocation/deallocation, automatic coalescing, bounded fragmentation
**Disadvantages**: Internal fragmentation (only power-of-2 sizes), may waste space for non-power-of-2 requests

### Q4: What is a memory pool and when would you use it?
**Answer**: Memory pool pre-allocates a large block and divides it into fixed-size chunks. Objects are allocated from this pool.

**Use cases**: 
- Known object sizes (e.g., network packets, database records)
- Frequent allocation/deallocation of same-sized objects
- Real-time systems requiring predictable allocation time
- Reducing fragmentation in embedded systems

**Benefits**: O(1) allocation, no fragmentation, cache-friendly, predictable performance

### Q5: How do garbage collectors handle circular references?
**Answer**: Reference counting cannot handle circular references because objects in cycles maintain non-zero reference counts.

**Solutions**:
- **Mark and Sweep**: Traces from roots, marks reachable objects, sweeps unmarked objects
- **Weak References**: Break cycles by using non-owning references
- **Cycle Detection**: Specialized algorithms to detect and break reference cycles

Modern garbage collectors use tracing algorithms (mark-and-sweep, copying, generational) rather than pure reference counting.

---
[← Back to Main Guide](./README.md)
