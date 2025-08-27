# Inter-Process Communication (IPC)

Inter-Process Communication enables processes to exchange data and synchronize their actions. Since processes have separate memory spaces, they need special mechanisms to communicate and coordinate their activities.

## Key Concepts

- **Shared Memory**: Processes share a common memory region for fast data exchange
- **Message Passing**: Processes communicate by sending/receiving messages through OS
- **Pipes**: Unidirectional communication channel between processes
- **Sockets**: Bidirectional communication, can work across networks
- **Signals**: Software interrupts for process notification
- **Synchronization**: Coordinating process execution to avoid race conditions

## IPC Mechanisms

### 1. Shared Memory
**Fast communication through shared memory region**

```c
// Producer Process
#include <sys/shm.h>
#include <sys/ipc.h>

int main() {
    key_t key = ftok("file", 65);
    int shmid = shmget(key, 1024, 0666|IPC_CREAT);
    char *shared_memory = (char*) shmat(shmid, NULL, 0);
    
    strcpy(shared_memory, "Hello from producer!");
    shmdt(shared_memory);  // Detach
    return 0;
}

// Consumer Process
int main() {
    key_t key = ftok("file", 65);
    int shmid = shmget(key, 1024, 0666);
    char *shared_memory = (char*) shmat(shmid, NULL, 0);
    
    printf("Data from shared memory: %s\n", shared_memory);
    shmdt(shared_memory);
    shmctl(shmid, IPC_RMID, NULL);  // Remove shared memory
    return 0;
}
```

### 2. Message Passing
**Communication through message queues**

```c
// Message Queue Example
#include <sys/msg.h>

struct message {
    long msg_type;
    char msg_text[100];
};

// Sender Process
int main() {
    key_t key = ftok("file", 65);
    int msgid = msgget(key, 0666 | IPC_CREAT);
    
    struct message msg;
    msg.msg_type = 1;
    strcpy(msg.msg_text, "Hello from sender!");
    
    msgsnd(msgid, &msg, sizeof(msg), 0);
    return 0;
}

// Receiver Process
int main() {
    key_t key = ftok("file", 65);
    int msgid = msgget(key, 0666);
    
    struct message msg;
    msgrcv(msgid, &msg, sizeof(msg), 1, 0);
    
    printf("Received: %s\n", msg.msg_text);
    msgctl(msgid, IPC_RMID, NULL);  // Remove message queue
    return 0;
}
```

### 3. Pipes
**Unidirectional communication channel**

```c
// Anonymous Pipe (between parent-child)
#include <unistd.h>

int main() {
    int pipefd[2];  // pipefd[0] = read end, pipefd[1] = write end
    char buffer[100];
    
    pipe(pipefd);
    
    if (fork() == 0) {
        // Child process - writer
        close(pipefd[0]);  // Close read end
        write(pipefd[1], "Hello from child!", 18);
        close(pipefd[1]);
    } else {
        // Parent process - reader
        close(pipefd[1]);  // Close write end
        read(pipefd[0], buffer, 100);
        printf("Parent received: %s\n", buffer);
        close(pipefd[0]);
        wait(NULL);
    }
    return 0;
}

// Named Pipe (FIFO)
mkfifo("/tmp/mypipe", 0666);  // Create named pipe

// Writer process
int fd = open("/tmp/mypipe", O_WRONLY);
write(fd, "Hello FIFO!", 12);
close(fd);

// Reader process  
int fd = open("/tmp/mypipe", O_RDONLY);
read(fd, buffer, 100);
close(fd);
```

### 4. Sockets
**Bidirectional communication, network capable**

```c
// Server Process
#include <sys/socket.h>
#include <netinet/in.h>

int main() {
    int server_fd = socket(AF_INET, SOCK_STREAM, 0);
    struct sockaddr_in address;
    
    address.sin_family = AF_INET;
    address.sin_addr.s_addr = INADDR_ANY;
    address.sin_port = htons(8080);
    
    bind(server_fd, (struct sockaddr *)&address, sizeof(address));
    listen(server_fd, 3);
    
    int client_socket = accept(server_fd, NULL, NULL);
    char *message = "Hello from server!";
    send(client_socket, message, strlen(message), 0);
    
    close(client_socket);
    close(server_fd);
    return 0;
}

// Client Process
int main() {
    int sock = socket(AF_INET, SOCK_STREAM, 0);
    struct sockaddr_in serv_addr;
    
    serv_addr.sin_family = AF_INET;
    serv_addr.sin_port = htons(8080);
    serv_addr.sin_addr.s_addr = inet_addr("127.0.0.1");
    
    connect(sock, (struct sockaddr *)&serv_addr, sizeof(serv_addr));
    
    char buffer[1024];
    read(sock, buffer, 1024);
    printf("Message from server: %s\n", buffer);
    
    close(sock);
    return 0;
}
```

### 5. Signals
**Software interrupts for process notification**

```c
#include <signal.h>

void signal_handler(int signum) {
    printf("Received signal %d\n", signum);
}

int main() {
    signal(SIGUSR1, signal_handler);  // Register handler
    
    pid_t child = fork();
    if (child == 0) {
        sleep(2);
        kill(getppid(), SIGUSR1);  // Send signal to parent
        exit(0);
    } else {
        pause();  // Wait for signal
        wait(NULL);
    }
    return 0;
}
```

## IPC Comparison

| Method | Speed | Complexity | Network Support | Synchronization |
|--------|-------|------------|-----------------|-----------------|
| **Shared Memory** | Fastest | Medium | No | Manual (semaphores) |
| **Message Queue** | Fast | Low | No | Built-in |
| **Pipes** | Medium | Low | No | None |
| **Sockets** | Slow | High | Yes | None |
| **Signals** | Fast | Low | No | Event-based |

## Producer-Consumer Example with Shared Memory

```c
// Shared data structure
struct shared_data {
    int buffer[10];
    int in;     // Next empty slot
    int out;    // Next full slot
    int count;  // Number of items
};

// Producer
void produce(struct shared_data *data, int item) {
    while (data->count == 10);  // Wait if buffer full
    
    data->buffer[data->in] = item;
    data->in = (data->in + 1) % 10;
    data->count++;
}

// Consumer
int consume(struct shared_data *data) {
    while (data->count == 0);  // Wait if buffer empty
    
    int item = data->buffer[data->out];
    data->out = (data->out + 1) % 10;
    data->count--;
    return item;
}
```

## Common Interview Questions

### Q1: What are the advantages and disadvantages of shared memory vs message passing?
**Answer**: 
**Shared Memory**: 
- Advantages: Fastest IPC, efficient for large data transfer
- Disadvantages: Manual synchronization needed, no built-in protection, complex programming

**Message Passing**: 
- Advantages: Built-in synchronization, easier to program, network support
- Disadvantages: Slower due to system calls, overhead for small messages

### Q2: When would you use pipes vs sockets?
**Answer**: 
**Pipes**: Use for local communication between related processes (parent-child), simple data transfer, shell command chaining
**Sockets**: Use for network communication, unrelated processes, client-server applications, need bidirectional communication

### Q3: What is the difference between anonymous and named pipes?
**Answer**: 
- **Anonymous pipes**: Created by pipe() system call, only between parent-child processes, exist only during process lifetime
- **Named pipes (FIFO)**: Have filesystem names, can be used by any processes knowing the name, persist until explicitly deleted

### Q4: How do you handle race conditions in shared memory?
**Answer**: Use synchronization primitives:
- **Semaphores**: Control access to shared resources
- **Mutex**: Mutual exclusion for critical sections
- **Memory barriers**: Ensure proper ordering of memory operations
- **Atomic operations**: For simple shared variables

### Q5: What happens if a process dies while holding a system resource (like message queue)?
**Answer**: Depends on the IPC mechanism:
- **Shared memory**: Remains until explicitly removed (can cause resource leaks)
- **Message queues**: May remain, need cleanup
- **Pipes**: Automatically cleaned up when all processes close them
- **Sockets**: Connections are terminated, but server sockets may remain
Best practice: Use signal handlers or atexit() for cleanup

---
[← Back to Main Guide](./README.md)
