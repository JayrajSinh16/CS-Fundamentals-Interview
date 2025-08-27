# Process States & State Transitions

Process states represent different stages in a process's lifecycle from creation to termination. Understanding these states and transitions is crucial for operating system management and debugging system performance issues.

## Key Concepts

- **New**: Process is being created, memory allocated, process control block initialized
- **Ready**: Process loaded in memory, waiting for CPU allocation by scheduler
- **Running**: Process currently executing instructions on CPU (only one per CPU core)
- **Waiting/Blocked**: Process waiting for I/O operation or event completion
- **Terminated**: Process execution completed, resources being deallocated
- **Suspended**: Process moved to secondary storage due to memory shortage

## State Transition Diagram

```
    NEW
     ↓ (admitted)
   READY ←──────────────── READY SUSPENDED
     ↓ ↑ (dispatch)         ↑ ↓ (suspend)
     ↓ ↑ (timeout)          ↑ ↓ (resume)
   RUNNING                  ↑ ↓
     ↓ ↑ (I/O complete)     ↑ ↓
     ↓ ↑ (event occurs)     ↑ ↓
   WAITING ←──────────────── WAITING SUSPENDED
     ↓ (process terminates)
  TERMINATED
```

## Detailed State Descriptions

### New State
- Process creation initiated
- Memory allocation in progress
- Process Control Block (PCB) being initialized
- Not yet ready for execution

### Ready State
- Process fully loaded in main memory
- Waiting in ready queue for CPU time
- Has all resources except CPU
- Can immediately start execution when scheduled

### Running State
- Process currently executing on CPU
- Only one process per CPU core can be in this state
- Can transition to ready, waiting, or terminated
- Preemption can move it back to ready

### Waiting/Blocked State
- Process cannot continue until some event occurs
- Common reasons: I/O operation, semaphore wait, child process completion
- Process voluntarily gives up CPU
- Must wait for specific event notification

### Terminated State
- Process execution completed (normal or abnormal termination)
- Resources being deallocated
- Exit status may be available to parent process
- PCB eventually removed from system

## Process Control Block (PCB) Information

```c
struct process_control_block {
    int process_id;                 // Unique process identifier
    enum process_state state;       // Current state
    int priority;                   // Scheduling priority
    struct cpu_context context;     // CPU registers, program counter
    struct memory_info memory;      // Memory allocation details
    struct file_descriptor_table files; // Open files
    int parent_pid;                 // Parent process ID
    struct process_times times;     // Creation, execution times
};
```

## Practical Example: Process State Monitoring

```bash
# Linux commands to observe process states
ps aux  # Shows current process states
# STAT column meanings:
# R - Running
# S - Sleeping (waiting)
# D - Uninterruptible sleep (I/O)
# Z - Zombie (terminated but not cleaned up)
# T - Stopped

# Example output:
# USER  PID  %CPU  %MEM  STAT  COMMAND
# root   1   0.0   0.1    S    /sbin/init
# user  1234 25.0  5.2    R    ./cpu_intensive_app
# user  1235  0.0  1.1    S    sleep 100
```

## State Transition Examples

### Example 1: I/O Operation
```
Process reading file:
RUNNING → (read() system call) → WAITING → (I/O complete) → READY → RUNNING
```

### Example 2: Time Slice Expiry
```
Process in round-robin scheduling:
RUNNING → (time quantum expires) → READY → (scheduled again) → RUNNING
```

### Example 3: Process Creation
```
Parent creating child:
NEW → (fork() completes) → READY → (scheduled) → RUNNING
```

## Common Interview Questions

### Q1: What is the difference between Ready and Waiting states?
**Answer**: 
- **Ready**: Process has all resources except CPU and can run immediately when CPU becomes available
- **Waiting**: Process lacks some resource (usually I/O completion) and cannot run even if CPU is available
Ready processes compete for CPU, while waiting processes wait for events.

### Q2: Can a process go directly from Running to Waiting state?
**Answer**: Yes, when a running process makes a system call that blocks (like reading from disk, waiting for user input, or waiting on a semaphore). The process voluntarily gives up the CPU because it cannot continue execution until the event occurs.

### Q3: What causes a process to move from Running to Ready state?
**Answer**: 
- **Preemption**: Higher priority process becomes ready
- **Time slice expiry**: In time-sharing systems with round-robin scheduling
- **Interrupts**: System interrupts that cause context switching
This is involuntary transition - the process is forced to give up CPU.

### Q4: What is a zombie process and how is it related to process states?
**Answer**: A zombie process is in terminated state but its PCB still exists because the parent hasn't read its exit status using wait(). The child has finished execution but remains in process table until parent acknowledges termination. This prevents resource leaks and allows parent to get child's exit status.

### Q5: Explain the concept of suspended states.
**Answer**: Suspended states occur when processes are moved to secondary storage due to:
- **Memory shortage**: System needs to free up RAM
- **Administrative reasons**: System administrator suspends processes
- **Performance tuning**: Moving low-priority processes out of memory
Suspended processes can be either ready-suspended or waiting-suspended, and must be resumed before execution.

---
[← Back to Main Guide](./README.md)
