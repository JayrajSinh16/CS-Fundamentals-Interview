# Virtual Memory & Page Replacement

Virtual memory is a memory management technique that provides an abstraction of main memory, allowing programs to use more memory than physically available. Page replacement algorithms decide which pages to swap out when physical memory is full.

## Key Concepts

- **Virtual Memory**: Abstraction that gives each process its own address space larger than physical RAM
- **Paging**: Memory management scheme dividing memory into fixed-size blocks (pages)
- **Page Fault**: Interrupt when accessing page not currently in physical memory
- **Page Replacement**: Algorithm to choose which page to remove when memory is full
- **Working Set**: Set of pages actively used by a process in recent time window
- **Thrashing**: Excessive page swapping that degrades system performance

## Virtual Memory Architecture

```
Virtual Address Space:      Physical Memory:        Secondary Storage:
┌─────────────────┐        ┌─────────────────┐     ┌─────────────────┐
│ Code Segment    │        │ Page Frame 0    │     │                 │
├─────────────────┤   →    ├─────────────────┤     │  Swap Space     │
│ Data Segment    │        │ Page Frame 1    │  ↔  │                 │
├─────────────────┤        ├─────────────────┤     │   (Pages not    │
│ Heap            │        │ Page Frame 2    │     │   in memory)    │
├─────────────────┤        ├─────────────────┤     │                 │
│ Stack           │        │      ...        │     └─────────────────┘
└─────────────────┘        └─────────────────┘

Page Table maps virtual pages to physical frames
```

## Page Replacement Algorithms

### 1. First-In-First-Out (FIFO)
**Replace the oldest page in memory**

```c
#include <queue>

class FIFO_PageReplacement {
private:
    queue<int> page_queue;
    unordered_set<int> pages_in_memory;
    int capacity;
    
public:
    FIFO_PageReplacement(int cap) : capacity(cap) {}
    
    bool access_page(int page) {
        if (pages_in_memory.find(page) != pages_in_memory.end()) {
            return true;  // Page hit
        }
        
        // Page fault
        if (page_queue.size() == capacity) {
            int oldest = page_queue.front();
            page_queue.pop();
            pages_in_memory.erase(oldest);
        }
        
        page_queue.push(page);
        pages_in_memory.insert(page);
        return false;  // Page fault occurred
    }
};

// Example trace
int pages[] = {1, 3, 0, 3, 5, 6, 3};
// With 3 frames: 9 5 4 | 5 4 7 → Page faults at positions 0,1,2,4,5
```

### 2. Least Recently Used (LRU)
**Replace the page that hasn't been used for the longest time**

```c
class LRU_PageReplacement {
private:
    list<int> recent_pages;  // Most recent at front
    unordered_map<int, list<int>::iterator> page_map;
    int capacity;
    
public:
    LRU_PageReplacement(int cap) : capacity(cap) {}
    
    bool access_page(int page) {
        auto it = page_map.find(page);
        
        if (it != page_map.end()) {
            // Page hit - move to front
            recent_pages.erase(it->second);
            recent_pages.push_front(page);
            page_map[page] = recent_pages.begin();
            return true;
        }
        
        // Page fault
        if (recent_pages.size() == capacity) {
            int lru_page = recent_pages.back();
            recent_pages.pop_back();
            page_map.erase(lru_page);
        }
        
        recent_pages.push_front(page);
        page_map[page] = recent_pages.begin();
        return false;
    }
};
```

### 3. Optimal (OPT)
**Replace page that will be used farthest in future (theoretical)**

```c
int optimal_page_replacement(int pages[], int n, int capacity) {
    unordered_set<int> frames;
    int page_faults = 0;
    
    for (int i = 0; i < n; i++) {
        if (frames.find(pages[i]) != frames.end()) {
            continue;  // Page hit
        }
        
        page_faults++;
        
        if (frames.size() < capacity) {
            frames.insert(pages[i]);
        } else {
            // Find page used farthest in future
            int farthest = i;
            int page_to_remove = -1;
            
            for (int page : frames) {
                int next_use = n;  // If not found, assume never used
                for (int j = i + 1; j < n; j++) {
                    if (pages[j] == page) {
                        next_use = j;
                        break;
                    }
                }
                
                if (next_use > farthest) {
                    farthest = next_use;
                    page_to_remove = page;
                }
            }
            
            frames.erase(page_to_remove);
            frames.insert(pages[i]);
        }
    }
    
    return page_faults;
}
```

### 4. Clock Algorithm (Second Chance)
**Approximation of LRU using reference bits**

```c
class ClockPageReplacement {
private:
    struct Page {
        int page_number;
        bool reference_bit;
        bool valid;
    };
    
    vector<Page> frames;
    int clock_hand;
    int capacity;
    
public:
    ClockPageReplacement(int cap) : capacity(cap), clock_hand(0) {
        frames.resize(cap);
        for (int i = 0; i < cap; i++) {
            frames[i].valid = false;
        }
    }
    
    bool access_page(int page) {
        // Check if page is already in memory
        for (int i = 0; i < capacity; i++) {
            if (frames[i].valid && frames[i].page_number == page) {
                frames[i].reference_bit = true;  // Set reference bit
                return true;  // Page hit
            }
        }
        
        // Page fault - find victim using clock algorithm
        while (true) {
            if (!frames[clock_hand].valid) {
                // Empty frame found
                break;
            }
            
            if (frames[clock_hand].reference_bit) {
                frames[clock_hand].reference_bit = false;  // Give second chance
            } else {
                break;  // Victim found
            }
            
            clock_hand = (clock_hand + 1) % capacity;
        }
        
        // Replace page
        frames[clock_hand].page_number = page;
        frames[clock_hand].reference_bit = true;
        frames[clock_hand].valid = true;
        clock_hand = (clock_hand + 1) % capacity;
        
        return false;  // Page fault occurred
    }
};
```

## Performance Comparison Example

```
Page Reference String: 1, 2, 3, 4, 1, 2, 5, 1, 2, 3, 4, 5
Number of Frames: 3

FIFO:
Step | Pages | Frames    | Fault?
-----|-------|-----------|-------
1    | 1     | [1,-,-]   | Yes
2    | 2     | [1,2,-]   | Yes  
3    | 3     | [1,2,3]   | Yes
4    | 4     | [4,2,3]   | Yes (replace 1)
5    | 1     | [4,1,3]   | Yes (replace 2)
6    | 2     | [4,1,2]   | Yes (replace 3)
7    | 5     | [5,1,2]   | Yes (replace 4)
8    | 1     | [5,1,2]   | No
9    | 2     | [5,1,2]   | No
10   | 3     | [5,3,2]   | Yes (replace 1)
11   | 4     | [5,3,4]   | Yes (replace 2)
12   | 5     | [5,3,4]   | No

Total Page Faults: 9

LRU:
Total Page Faults: 10

Optimal:
Total Page Faults: 7 (theoretical minimum)
```

## Belady's Anomaly

```c
// FIFO can have more page faults with more frames!
// Example: Reference string 1,2,3,4,1,2,5,1,2,3,4,5

// 3 frames (FIFO): 9 page faults
// 4 frames (FIFO): 10 page faults (anomaly!)

// LRU and Optimal don't exhibit this anomaly
```

## Working Set Model

```c
class WorkingSet {
private:
    int window_size;
    queue<pair<int, int>> access_history;  // (page, time)
    unordered_set<int> current_working_set;
    
public:
    void access_page(int page, int time) {
        // Remove pages outside window
        while (!access_history.empty() && 
               time - access_history.front().second > window_size) {
            int old_page = access_history.front().first;
            access_history.pop();
            
            // Check if page still in window
            bool still_in_window = false;
            queue<pair<int, int>> temp = access_history;
            while (!temp.empty()) {
                if (temp.front().first == old_page) {
                    still_in_window = true;
                    break;
                }
                temp.pop();
            }
            
            if (!still_in_window) {
                current_working_set.erase(old_page);
            }
        }
        
        // Add current page
        access_history.push({page, time});
        current_working_set.insert(page);
    }
    
    int get_working_set_size() {
        return current_working_set.size();
    }
};
```

## Common Interview Questions

### Q1: What is the difference between FIFO and LRU page replacement?
**Answer**: 
- **FIFO**: Replaces oldest page based on arrival time, simple but can suffer from Belady's anomaly
- **LRU**: Replaces least recently used page based on access time, better performance but more complex to implement
LRU considers actual usage patterns while FIFO only considers arrival order.

### Q2: Why is the Optimal algorithm not practically implementable?
**Answer**: The Optimal algorithm requires knowledge of future page references, which is impossible in real systems. It's used as a theoretical benchmark to compare other algorithms. However, some approximations exist for specific scenarios where access patterns are predictable.

### Q3: What is Belady's anomaly and which algorithms suffer from it?
**Answer**: Belady's anomaly occurs when increasing the number of page frames results in more page faults instead of fewer. Only FIFO suffers from this anomaly. LRU and Optimal are "stack algorithms" and don't exhibit this behavior because they maintain inclusion property.

### Q4: How does the Clock algorithm approximate LRU?
**Answer**: Clock algorithm uses reference bits to give pages a "second chance." When a page is accessed, its reference bit is set. During replacement, it scans circularly, clearing reference bits of encountered pages until finding one with reference bit 0. This approximates LRU by giving recently used pages another chance.

### Q5: What factors affect the choice of page replacement algorithm?
**Answer**: 
- **Performance requirements**: LRU generally performs better than FIFO
- **Hardware support**: Some algorithms need hardware reference bits
- **Implementation complexity**: FIFO is simplest, LRU is more complex
- **Memory overhead**: LRU requires additional data structures
- **Real-time constraints**: Simple algorithms may be preferred for predictable timing

---
[← Back to Main Guide](./README.md)
