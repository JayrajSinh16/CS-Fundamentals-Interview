# Cache Memory & Performance

Cache memory is a small, fast storage located between the CPU and main memory that stores frequently accessed data and instructions. Understanding cache behavior is crucial for system performance optimization and computer architecture interviews.

## Key Concepts

- **Cache Hit**: Data found in cache (fast access)
- **Cache Miss**: Data not in cache, must fetch from memory (slow)
- **Hit Rate**: Percentage of memory accesses served by cache
- **Miss Rate**: Percentage of memory accesses that miss cache
- **Temporal Locality**: Recently accessed data likely to be accessed again
- **Spatial Locality**: Nearby data likely to be accessed soon

## Cache Hierarchy

```
CPU Registers (1 cycle, ~32 words)
    ↓
L1 Cache (1-2 cycles, 32-64 KB)
    ↓
L2 Cache (4-12 cycles, 256 KB - 1 MB)
    ↓
L3 Cache (12-40 cycles, 1-32 MB)
    ↓
Main Memory (100-300 cycles, GBs)
    ↓
Secondary Storage (10,000+ cycles, TBs)
```

## Cache Organization Types

### 1. Direct-Mapped Cache
**Each memory block maps to exactly one cache line**

```c
typedef struct {
    int valid;
    unsigned int tag;
    char data[BLOCK_SIZE];
} CacheLine;

typedef struct {
    CacheLine lines[CACHE_SIZE / BLOCK_SIZE];
    int hits;
    int misses;
} DirectMappedCache;

int access_direct_mapped(DirectMappedCache* cache, unsigned int address) {
    // Extract fields from address
    unsigned int block_offset = address & (BLOCK_SIZE - 1);
    unsigned int index = (address >> BLOCK_OFFSET_BITS) & (NUM_LINES - 1);
    unsigned int tag = address >> (BLOCK_OFFSET_BITS + INDEX_BITS);
    
    CacheLine* line = &cache->lines[index];
    
    if (line->valid && line->tag == tag) {
        // Cache hit
        cache->hits++;
        return line->data[block_offset];
    } else {
        // Cache miss
        cache->misses++;
        
        // Load block from memory
        load_block_from_memory(line, address);
        line->valid = 1;
        line->tag = tag;
        
        return line->data[block_offset];
    }
}

// Example: 32-bit address, 64-byte blocks, 1024 lines
// Address breakdown: [18-bit tag][10-bit index][6-bit offset]
// Simple but prone to conflict misses
```

### 2. Set-Associative Cache
**Each memory block can map to one of several cache lines in a set**

```c
#define WAYS_PER_SET 4  // 4-way set associative

typedef struct {
    CacheLine ways[WAYS_PER_SET];
    int lru_counter[WAYS_PER_SET];
} CacheSet;

typedef struct {
    CacheSet sets[NUM_SETS];
    int global_counter;
    int hits;
    int misses;
} SetAssociativeCache;

int access_set_associative(SetAssociativeCache* cache, unsigned int address) {
    unsigned int block_offset = address & (BLOCK_SIZE - 1);
    unsigned int set_index = (address >> BLOCK_OFFSET_BITS) & (NUM_SETS - 1);
    unsigned int tag = address >> (BLOCK_OFFSET_BITS + SET_INDEX_BITS);
    
    CacheSet* set = &cache->sets[set_index];
    
    // Check all ways in the set
    for (int way = 0; way < WAYS_PER_SET; way++) {
        CacheLine* line = &set->ways[way];
        
        if (line->valid && line->tag == tag) {
            // Cache hit
            cache->hits++;
            set->lru_counter[way] = cache->global_counter++;
            return line->data[block_offset];
        }
    }
    
    // Cache miss - find LRU way
    cache->misses++;
    int lru_way = find_lru_way(set);
    
    CacheLine* victim = &set->ways[lru_way];
    load_block_from_memory(victim, address);
    victim->valid = 1;
    victim->tag = tag;
    set->lru_counter[lru_way] = cache->global_counter++;
    
    return victim->data[block_offset];
}

int find_lru_way(CacheSet* set) {
    int lru_way = 0;
    int oldest_time = set->lru_counter[0];
    
    for (int way = 1; way < WAYS_PER_SET; way++) {
        if (set->lru_counter[way] < oldest_time) {
            oldest_time = set->lru_counter[way];
            lru_way = way;
        }
    }
    
    return lru_way;
}

// Reduces conflict misses compared to direct-mapped
// More complex hardware, slower than direct-mapped
```

### 3. Fully Associative Cache
**Any memory block can map to any cache line**

```c
typedef struct {
    CacheLine lines[TOTAL_CACHE_LINES];
    int lru_order[TOTAL_CACHE_LINES];
    int global_counter;
    int hits;
    int misses;
} FullyAssociativeCache;

int access_fully_associative(FullyAssociativeCache* cache, unsigned int address) {
    unsigned int block_offset = address & (BLOCK_SIZE - 1);
    unsigned int tag = address >> BLOCK_OFFSET_BITS;
    
    // Search all cache lines
    for (int i = 0; i < TOTAL_CACHE_LINES; i++) {
        CacheLine* line = &cache->lines[i];
        
        if (line->valid && line->tag == tag) {
            // Cache hit
            cache->hits++;
            cache->lru_order[i] = cache->global_counter++;
            return line->data[block_offset];
        }
    }
    
    // Cache miss - find LRU line
    cache->misses++;
    int lru_line = 0;
    int oldest_time = cache->lru_order[0];
    
    for (int i = 1; i < TOTAL_CACHE_LINES; i++) {
        if (cache->lru_order[i] < oldest_time) {
            oldest_time = cache->lru_order[i];
            lru_line = i;
        }
    }
    
    CacheLine* victim = &cache->lines[lru_line];
    load_block_from_memory(victim, address);
    victim->valid = 1;
    victim->tag = tag;
    cache->lru_order[lru_line] = cache->global_counter++;
    
    return victim->data[block_offset];
}

// No conflict misses, maximum flexibility
// Expensive hardware, slow comparison for large caches
```

## Replacement Policies

### LRU (Least Recently Used)
```c
// True LRU for 4-way set
typedef struct {
    int access_order[4];  // 0 = most recent, 3 = least recent
} LRU_State;

void update_lru_on_hit(LRU_State* lru, int way) {
    int old_position = -1;
    
    // Find current position of the accessed way
    for (int i = 0; i < 4; i++) {
        if (lru->access_order[i] == way) {
            old_position = i;
            break;
        }
    }
    
    // Shift all ways that were more recent
    for (int i = old_position; i > 0; i--) {
        lru->access_order[i] = lru->access_order[i-1];
    }
    
    // Put accessed way at most recent position
    lru->access_order[0] = way;
}

int get_lru_victim(LRU_State* lru) {
    return lru->access_order[3];  // Least recently used
}
```

### Pseudo-LRU (Tree-Based)
```c
// For 4-way set associative cache
typedef struct {
    int tree_bits;  // 3 bits for 4-way: bit 0 (root), bits 1-2 (leaves)
} PseudoLRU_State;

void update_pseudo_lru(PseudoLRU_State* plru, int way) {
    switch (way) {
        case 0: plru->tree_bits |= 0b001; plru->tree_bits |= 0b010; break;
        case 1: plru->tree_bits |= 0b001; plru->tree_bits &= ~0b010; break;
        case 2: plru->tree_bits &= ~0b001; plru->tree_bits |= 0b100; break;
        case 3: plru->tree_bits &= ~0b001; plru->tree_bits &= ~0b100; break;
    }
}

int get_pseudo_lru_victim(PseudoLRU_State* plru) {
    // Traverse tree following 0 bits to find victim
    if (!(plru->tree_bits & 0b001)) {
        return (plru->tree_bits & 0b010) ? 0 : 1;
    } else {
        return (plru->tree_bits & 0b100) ? 2 : 3;
    }
}

// Approximates LRU with much less hardware
```

### Random Replacement
```c
#include <stdlib.h>

int random_replacement(int num_ways) {
    return rand() % num_ways;
}

// Simple hardware, surprisingly effective in practice
// No pathological access patterns
```

## Write Policies

### Write-Through
```c
void write_through(Cache* cache, unsigned int address, int data) {
    if (is_cache_hit(cache, address)) {
        // Update cache
        update_cache_line(cache, address, data);
    }
    
    // Always write to main memory
    write_to_memory(address, data);
}

// Pros: Memory always consistent, simple
// Cons: High memory traffic, slower writes
```

### Write-Back
```c
typedef struct {
    int valid;
    int dirty;  // Modified since loaded from memory
    unsigned int tag;
    char data[BLOCK_SIZE];
} WriteBackCacheLine;

void write_back(WriteBackCache* cache, unsigned int address, int data) {
    if (is_cache_hit(cache, address)) {
        // Update cache and mark dirty
        update_cache_line(cache, address, data);
        mark_dirty(cache, address);
    } else {
        // Cache miss - may need to write back dirty line
        WriteBackCacheLine* victim = find_victim_line(cache, address);
        
        if (victim->valid && victim->dirty) {
            // Write back dirty data
            write_back_to_memory(victim);
        }
        
        // Load new block and update
        load_block_from_memory(victim, address);
        update_cache_line(cache, address, data);
        victim->dirty = 1;
    }
}

// Pros: Reduced memory traffic, faster writes
// Cons: Complex, inconsistent memory until write-back
```

## Cache Performance Analysis

### Performance Metrics
```c
typedef struct {
    long long total_accesses;
    long long hits;
    long long misses;
    long long compulsory_misses;  // First access to block
    long long capacity_misses;    // Cache too small
    long long conflict_misses;    // Limited associativity
} CacheStatistics;

double calculate_hit_rate(CacheStatistics* stats) {
    return (double)stats->hits / stats->total_accesses;
}

double calculate_miss_rate(CacheStatistics* stats) {
    return (double)stats->misses / stats->total_accesses;
}

double calculate_average_access_time(CacheStatistics* stats, 
                                   int cache_hit_time, 
                                   int memory_access_time) {
    double hit_rate = calculate_hit_rate(stats);
    double miss_rate = calculate_miss_rate(stats);
    
    return hit_rate * cache_hit_time + 
           miss_rate * (cache_hit_time + memory_access_time);
}

// Example calculation:
// Hit rate: 95%, Cache hit time: 2 cycles, Memory access: 100 cycles
// Average access time = 0.95 * 2 + 0.05 * (2 + 100) = 1.9 + 5.1 = 7 cycles
```

### Cache Simulator
```c
typedef struct {
    unsigned int address;
    char operation;  // 'R' for read, 'W' for write
} MemoryTrace;

void simulate_cache_performance(MemoryTrace trace[], int trace_length) {
    DirectMappedCache cache = {0};
    
    for (int i = 0; i < trace_length; i++) {
        unsigned int addr = trace[i].address;
        char op = trace[i].operation;
        
        if (op == 'R') {
            access_direct_mapped(&cache, addr);
        } else if (op == 'W') {
            // Handle write according to write policy
            write_through_access(&cache, addr);
        }
        
        // Print periodic statistics
        if (i % 10000 == 0) {
            double hit_rate = (double)cache.hits / (cache.hits + cache.misses);
            printf("After %d accesses: Hit rate = %.2f%%\n", 
                   i + 1, hit_rate * 100);
        }
    }
    
    // Final statistics
    int total = cache.hits + cache.misses;
    printf("Final Results:\n");
    printf("Total accesses: %d\n", total);
    printf("Hits: %d (%.2f%%)\n", cache.hits, 
           (double)cache.hits / total * 100);
    printf("Misses: %d (%.2f%%)\n", cache.misses, 
           (double)cache.misses / total * 100);
}
```

## Cache-Conscious Programming

### Example 1: Matrix Multiplication Optimization
```c
// Cache-unfriendly version
void matrix_multiply_naive(int n, double A[n][n], double B[n][n], double C[n][n]) {
    for (int i = 0; i < n; i++) {
        for (int j = 0; j < n; j++) {
            for (int k = 0; k < n; k++) {
                C[i][j] += A[i][k] * B[k][j];  // B access is not cache-friendly
            }
        }
    }
}

// Cache-friendly version with blocking
void matrix_multiply_blocked(int n, double A[n][n], double B[n][n], double C[n][n]) {
    int block_size = 64;  // Tune based on cache size
    
    for (int ii = 0; ii < n; ii += block_size) {
        for (int jj = 0; jj < n; jj += block_size) {
            for (int kk = 0; kk < n; kk += block_size) {
                // Work on block
                for (int i = ii; i < min(ii + block_size, n); i++) {
                    for (int j = jj; j < min(jj + block_size, n); j++) {
                        for (int k = kk; k < min(kk + block_size, n); k++) {
                            C[i][j] += A[i][k] * B[k][j];
                        }
                    }
                }
            }
        }
    }
}

// Performance improvement: 2-10x speedup for large matrices
```

### Example 2: Array Traversal
```c
// Cache-unfriendly: column-wise access
void sum_columns_bad(int rows, int cols, int matrix[rows][cols]) {
    for (int j = 0; j < cols; j++) {
        int sum = 0;
        for (int i = 0; i < rows; i++) {
            sum += matrix[i][j];  // Non-sequential memory access
        }
        printf("Column %d sum: %d\n", j, sum);
    }
}

// Cache-friendly: row-wise access
void sum_columns_good(int rows, int cols, int matrix[rows][cols]) {
    int* column_sums = calloc(cols, sizeof(int));
    
    for (int i = 0; i < rows; i++) {
        for (int j = 0; j < cols; j++) {
            column_sums[j] += matrix[i][j];  // Sequential memory access
        }
    }
    
    for (int j = 0; j < cols; j++) {
        printf("Column %d sum: %d\n", j, column_sums[j]);
    }
    
    free(column_sums);
}
```

### Example 3: Data Structure Layout
```c
// Cache-unfriendly: Array of Structures (AoS)
typedef struct {
    float x, y, z;    // Position
    float r, g, b;    // Color
    float nx, ny, nz; // Normal
} Vertex_AoS;

void process_positions_aos(Vertex_AoS vertices[], int count) {
    for (int i = 0; i < count; i++) {
        // Only need position, but cache line loads entire vertex
        process_position(vertices[i].x, vertices[i].y, vertices[i].z);
    }
}

// Cache-friendly: Structure of Arrays (SoA)
typedef struct {
    float* positions_x;
    float* positions_y;
    float* positions_z;
    float* colors_r;
    float* colors_g;
    float* colors_b;
    float* normals_x;
    float* normals_y;
    float* normals_z;
} VertexData_SoA;

void process_positions_soa(VertexData_SoA* vertices, int count) {
    for (int i = 0; i < count; i++) {
        // Cache line contains only position data - more efficient
        process_position(vertices->positions_x[i], 
                        vertices->positions_y[i], 
                        vertices->positions_z[i]);
    }
}
```

## Common Interview Questions

### Q1: Explain the difference between L1, L2, and L3 caches.
**Answer**: 
- **L1 Cache**: Closest to CPU, fastest (1-2 cycles), smallest (32-64KB), split into instruction and data caches
- **L2 Cache**: Unified cache, larger (256KB-1MB), slower (4-12 cycles), may be per-core or shared
- **L3 Cache**: Largest (1-32MB), slowest (12-40 cycles), typically shared among all cores
Each level trades off speed for capacity, forming a memory hierarchy that exploits locality.

### Q2: What is the difference between cache hit and cache miss?
**Answer**: 
- **Cache Hit**: Requested data found in cache, fast access (1-2 cycles for L1)
- **Cache Miss**: Data not in cache, must fetch from next level (L2, L3, or main memory)

Hit rate = hits/(hits+misses). Higher hit rates mean better performance. Miss penalty is the additional time to fetch data from slower memory levels.

### Q3: Compare direct-mapped, set-associative, and fully associative caches.
**Answer**: 
- **Direct-mapped**: Each memory block maps to exactly one cache line. Fast, simple hardware, but prone to conflict misses
- **Set-associative**: Memory blocks map to one of N lines in a set. Reduces conflict misses, moderate complexity
- **Fully associative**: Any memory block can go anywhere. No conflict misses but expensive comparison hardware

Trade-off between hardware complexity, access time, and miss rate.

### Q4: What are the three types of cache misses?
**Answer**: 
- **Compulsory (Cold) Misses**: First access to a memory block, unavoidable
- **Capacity Misses**: Cache too small to hold all needed data, would occur even in fully associative cache
- **Conflict (Collision) Misses**: Due to limited associativity, occur when multiple blocks map to same location

Increasing cache size reduces capacity misses; increasing associativity reduces conflict misses.

### Q5: Explain write-through vs write-back cache policies.
**Answer**: 
- **Write-through**: All writes go to both cache and main memory immediately. Simple, consistent memory, but high memory traffic
- **Write-back**: Writes only update cache, memory updated when line is evicted. Needs dirty bit to track modifications. Reduces memory traffic but more complex

Write-back is generally preferred for performance, write-through for simplicity and consistency.

---
[← Back to Main Guide](./README.md)
