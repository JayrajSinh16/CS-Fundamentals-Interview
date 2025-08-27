# CPU Scheduling Algorithms

CPU scheduling determines which process gets CPU time and for how long. Different algorithms optimize for various metrics like response time, throughput, or fairness. Understanding these algorithms is essential for system performance optimization.

## Key Concepts

- **Preemptive**: Scheduler can interrupt running process to give CPU to another process
- **Non-preemptive**: Process runs until completion or voluntarily gives up CPU
- **Turnaround Time**: Total time from process arrival to completion
- **Waiting Time**: Time process spends in ready queue
- **Response Time**: Time from arrival to first CPU allocation
- **CPU Utilization**: Percentage of time CPU is busy executing processes

## Scheduling Algorithms

### 1. First Come First Served (FCFS)
**Non-preemptive scheduling based on arrival time**

```
Process  Arrival  Burst
   P1      0       8
   P2      1       4  
   P3      2       2

Gantt Chart: |P1(0-8)|P2(8-12)|P3(12-14)|

Waiting Times: P1=0, P2=7, P3=10  Average=5.67
Turnaround Times: P1=8, P2=11, P3=12  Average=10.33
```

### 2. Shortest Job First (SJF)
**Schedules process with shortest burst time first**

```
Same processes as above:

Non-preemptive SJF:
Gantt Chart: |P1(0-8)|P3(8-10)|P2(10-14)|
Average Waiting Time: 3.33

Preemptive SJF (SRTF):
At t=1: P2 arrives (4 < 7 remaining), preempt P1
At t=2: P3 arrives (2 < 3 remaining), preempt P2
Gantt Chart: |P1(0-1)|P3(2-4)|P2(4-8)|P1(8-15)|
```

### 3. Round Robin (RR)
**Time-sharing with fixed time quantum**

```
Time Quantum = 3

Process  Arrival  Burst
   P1      0       8
   P2      1       4
   P3      2       2

Gantt Chart: |P1(0-3)|P2(3-6)|P3(6-8)|P1(8-11)|P2(11-12)|P1(12-14)|

Waiting Times: P1=6, P2=7, P3=4  Average=5.67
```

### 4. Priority Scheduling
**Schedules based on process priority (lower number = higher priority)**

```
Process  Arrival  Burst  Priority
   P1      0       8      2
   P2      1       4      1
   P3      2       2      3

Non-preemptive Priority:
Gantt Chart: |P1(0-8)|P2(8-12)|P3(12-14)|

Preemptive Priority:
At t=1: P2 has higher priority, preempt P1
Gantt Chart: |P1(0-1)|P2(1-5)|P1(5-12)|P3(12-14)|
```

### 5. Multilevel Queue Scheduling
**Separate queues for different process types**

```
System Processes (Priority 0) → Round Robin (q=1)
Interactive Processes (Priority 1) → Round Robin (q=2)  
Batch Processes (Priority 2) → FCFS

Higher priority queues served first
```

## Performance Metrics Calculation

### Example Calculation
```
Process  Arrival  Burst  Completion  Turnaround  Waiting
   P1      0       5       8           8          3
   P2      2       3       5           3          0  
   P3      4       1       9           5          4

Formulas:
Turnaround Time = Completion Time - Arrival Time
Waiting Time = Turnaround Time - Burst Time
Response Time = First CPU Time - Arrival Time

Average Turnaround Time = (8+3+5)/3 = 5.33
Average Waiting Time = (3+0+4)/3 = 2.33
CPU Utilization = (Total Burst Time)/(Total Time) = 9/9 = 100%
Throughput = Number of processes/Total time = 3/9 = 0.33 processes/unit
```

## Algorithm Comparison

| Algorithm | Advantages | Disadvantages | Best Use Case |
|-----------|------------|---------------|---------------|
| **FCFS** | Simple, fair | Convoy effect, poor response time | Batch systems |
| **SJF** | Optimal average waiting time | Starvation of long processes | Known burst times |
| **Round Robin** | Fair, good response time | Higher overhead, poor for I/O bound | Interactive systems |
| **Priority** | Important tasks get preference | Starvation, priority inversion | Real-time systems |
| **Multilevel** | Flexible, different algorithms per queue | Complex, tuning required | General purpose OS |

## Common Interview Questions

### Q1: What is the convoy effect in FCFS scheduling?
**Answer**: Convoy effect occurs when short processes wait for a long process to complete, similar to cars stuck behind a slow truck. If a CPU-intensive process arrives first, all subsequent I/O-bound processes must wait, leading to poor system utilization and increased average waiting time.

### Q2: Why is SJF optimal for average waiting time?
**Answer**: SJF minimizes average waiting time because shorter jobs complete quickly, reducing the total waiting time of all processes. Mathematically, if you sort jobs by burst time and schedule them in increasing order, you get the minimum possible average waiting time (proven by exchange argument).

### Q3: What are the advantages and disadvantages of Round Robin scheduling?
**Answer**: 
**Advantages**: Fair CPU allocation, good response time for interactive processes, prevents starvation
**Disadvantages**: Higher context switching overhead, performance depends on time quantum choice (too small = overhead, too large = poor response time), not optimal for batch processes

### Q4: How do you choose the optimal time quantum for Round Robin?
**Answer**: 
- **Too small**: High context switching overhead, CPU utilization decreases
- **Too large**: Becomes similar to FCFS, poor response time
- **Optimal**: Usually 10-100ms, should be larger than context switch time but smaller than typical burst time
- Rule of thumb: 80% of processes should complete within one quantum

### Q5: What is priority inversion and how can it be solved?
**Answer**: Priority inversion occurs when a high-priority process waits for a resource held by a low-priority process, while a medium-priority process runs. Solutions include:
- **Priority Inheritance**: Low-priority process temporarily inherits high priority
- **Priority Ceiling**: Process acquires highest priority of any resource it might need
- **Disable preemption**: During critical sections (simple but reduces concurrency)

---
[← Back to Main Guide](./README.md)
