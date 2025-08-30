# CAP Theorem

The CAP Theorem, also known as Brewer's Theorem, is a fundamental principle in distributed systems that states you can only guarantee two out of three properties simultaneously.

## 🔺 The Three Properties

```mermaid
graph TD
    A[CAP Theorem] --> B[Consistency]
    A --> C[Availability] 
    A --> D[Partition Tolerance]
    
    B --> E["All nodes see the<br/>same data simultaneously"]
    C --> F["System remains operational<br/>even if some nodes fail"]
    D --> G["System continues despite<br/>network failures"]
    
    style A fill:#87CEEB
    style B fill:#ffcccc
    style C fill:#ccffcc  
    style D fill:#ffffcc
```

### Consistency (C)
All nodes see the same data at the same time. When a write occurs, all subsequent reads must return the updated value.

### Availability (A)
The system remains operational and responsive. Every request receives a response (success or failure).

### Partition Tolerance (P)
The system continues to function despite network failures that prevent communication between nodes.

## ⚖️ The Three Combinations

Since network partitions are inevitable in distributed systems, you typically choose between **CP** or **AP**.

### CP - Consistency + Partition Tolerance
System remains consistent during network partitions but sacrifices availability.

```mermaid
sequenceDiagram
    participant C as Client
    participant N1 as Node 1
    participant N2 as Node 2
    
    Note over N1,N2: Network Partition Occurs
    C->>N1: Write Request
    N1--xN2: Cannot sync with Node 2
    N1->>C: Error - Cannot guarantee consistency
    
    Note over N1,N2: Partition Healed
    C->>N1: Write Request
    N1->>N2: Sync data
    N2->>N1: Ack
    N1->>C: Success
```

**Examples:** Traditional RDBMS (PostgreSQL, MySQL), HBase, MongoDB

### AP - Availability + Partition Tolerance
System remains available during partitions but may serve stale data.

```mermaid
sequenceDiagram
    participant C as Client
    participant N1 as Node 1
    participant N2 as Node 2
    
    Note over N1,N2: Network Partition Occurs
    C->>N1: Write Request
    N1->>C: Success (local write)
    
    C->>N2: Read Request
    N2->>C: Stale Data (not yet synced)
    
    Note over N1,N2: Partition Healed - Eventual Consistency
    N1->>N2: Sync latest data
```

**Examples:** Cassandra, DynamoDB, CouchDB, DNS

### CA - Consistency + Availability
Perfect consistency and availability, but no partition tolerance.

```mermaid
graph LR
    A[Client] --> B[Single Node System]
    B --> C[(Database)]
    
    style B fill:#ffcccc
    style C fill:#ffcccc
```

**Examples:** Single-node databases, LDAP

> **Note:** CA systems don't exist in truly distributed environments since network partitions are inevitable.

## 🏢 Real-World Examples

### Banking System (CP System)
```mermaid
graph TD
    A[ATM Request] --> B{Network Partition?}
    B -->|No| C[Process Transaction]
    B -->|Yes| D[Reject Transaction]
    
    C --> E[Update All Replicas]
    E --> F[Return Success]
    
    D --> G[Return Error:<br/>'Service Unavailable']
    
    style D fill:#ffcccc
    style G fill:#ffcccc
```

**Reasoning:** Better to be unavailable than show incorrect account balance.

### Social Media Feed (AP System)
```mermaid
graph TD
    A[User Posts] --> B{Network Partition?}
    B -->|No| C[Update All Regions]
    B -->|Yes| D[Update Local Region]
    
    C --> E[Immediate Global Consistency]
    D --> F[Eventual Consistency]
    
    F --> G[Sync when partition heals]
    
    style D fill:#ccffcc
    style F fill:#ffffcc
```

**Reasoning:** Users can tolerate seeing slightly stale posts, but the service must remain available.

## 📊 Detailed Comparison

| System Type | Consistency | Availability | Partition Tolerance | Use Case |
|-------------|-------------|--------------|-------------------|----------|
| **RDBMS** | Strong | High | Low | Financial transactions |
| **Cassandra** | Eventual | High | High | Social media, IoT |
| **MongoDB** | Strong | Medium | High | Content management |
| **Redis Cluster** | Eventual | High | High | Caching, sessions |
| **Zookeeper** | Strong | Medium | High | Configuration management |

## 🔄 Consistency Models

### Strong Consistency
```mermaid
sequenceDiagram
    participant C as Client
    participant N1 as Node 1
    participant N2 as Node 2
    participant N3 as Node 3
    
    C->>N1: Write(x=5)
    N1->>N2: Replicate(x=5)
    N1->>N3: Replicate(x=5)
    N2->>N1: Ack
    N3->>N1: Ack
    N1->>C: Write Complete
    
    Note over C,N3: All subsequent reads return x=5
```

### Eventual Consistency
```mermaid
sequenceDiagram
    participant C1 as Client 1
    participant C2 as Client 2
    participant N1 as Node 1
    participant N2 as Node 2
    
    C1->>N1: Write(x=5)
    N1->>C1: Write Complete
    
    C2->>N2: Read(x)
    N2->>C2: x=old_value
    
    Note over N1,N2: Background sync
    N1->>N2: Replicate(x=5)
    
    C2->>N2: Read(x)
    N2->>C2: x=5
```

## 🎯 Design Decisions Framework

### When to Choose CP (Consistency + Partition Tolerance)

```mermaid
flowchart TD
    A[Design Decision] --> B{Data Correctness Critical?}
    B -->|Yes| C{Can tolerate downtime?}
    C -->|Yes| D[Choose CP]
    C -->|No| E[Consider eventual consistency with compensation]
    B -->|No| F[Choose AP]
    
    D --> G[Examples: Banking, Inventory]
    E --> H[Examples: E-commerce with reservation]
    F --> I[Examples: Social media, Content]
    
    style D fill:#ffcccc
    style F fill:#ccffcc
```

### Implementation Patterns

#### CP Implementation - Two-Phase Commit
```mermaid
sequenceDiagram
    participant C as Coordinator
    participant N1 as Node 1
    participant N2 as Node 2
    participant N3 as Node 3
    
    Note over C,N3: Phase 1 - Prepare
    C->>N1: Prepare(transaction)
    C->>N2: Prepare(transaction)
    C->>N3: Prepare(transaction)
    
    N1->>C: Vote Yes
    N2->>C: Vote Yes
    N3->>C: Vote No
    
    Note over C,N3: Phase 2 - Abort (due to No vote)
    C->>N1: Abort
    C->>N2: Abort
    C->>N3: Abort
```

#### AP Implementation - Gossip Protocol
```mermaid
graph TD
    A[Node A updates] --> B[Node A tells Node B]
    B --> C[Node B tells Node C]
    C --> D[Node C tells Node D]
    A --> E[Node A tells Node E]
    E --> F[Node E tells Node F]
    
    style A fill:#ffcccc
    style B fill:#ffffcc
    style C fill:#ccffcc
    style D fill:#ccffcc
    style E fill:#ffffcc
    style F fill:#ccffcc
```

## 🧮 Mathematical Perspective

### Availability Calculation
```
Availability = MTBF / (MTBF + MTTR)

Where:
- MTBF = Mean Time Between Failures
- MTTR = Mean Time To Recovery

Example:
- MTBF = 8760 hours (1 year)
- MTTR = 1 hour
- Availability = 8760 / (8760 + 1) = 99.99%
```

### Consistency Lag
```
Consistency Lag = Network Delay + Processing Time + Queue Wait

Typical values:
- Same datacenter: 1-5ms
- Cross-region: 50-200ms
- Cross-continent: 150-300ms
```

## 🏗️ Architectural Patterns

### Multi-Master Replication (AP)
```mermaid
graph TD
    A[Client 1] --> B[Master 1]
    C[Client 2] --> D[Master 2]
    E[Client 3] --> F[Master 3]
    
    B <--> D
    D <--> F
    F <--> B
    
    B --> G[(Data Store)]
    D --> H[(Data Store)]
    F --> I[(Data Store)]
    
    style B fill:#ccffcc
    style D fill:#ccffcc
    style F fill:#ccffcc
```

### Consensus-Based (CP)
```mermaid
graph TD
    A[Client] --> B[Leader]
    B --> C[Follower 1]
    B --> D[Follower 2]
    B --> E[Follower 3]
    
    C --> F{Majority?}
    D --> F
    E --> F
    
    F -->|Yes| G[Commit]
    F -->|No| H[Reject]
    
    style B fill:#ffcccc
    style G fill:#ccffcc
    style H fill:#ffcccc
```

## 🔧 Practical Implementations

### Database Configurations

#### MySQL Cluster (CP)
```sql
-- Synchronous replication
SET GLOBAL sync_binlog = 1;
SET GLOBAL innodb_flush_log_at_trx_commit = 1;

-- Wait for slave acknowledgment
CHANGE MASTER TO MASTER_HOST='master_ip',
                 MASTER_USER='repl_user',
                 MASTER_PASSWORD='password',
                 MASTER_LOG_FILE='mysql-bin.000001',
                 MASTER_LOG_POS=4;
```

#### Cassandra (AP)
```yaml
# cassandra.yaml
consistency_level: ONE  # Fast writes
read_repair_chance: 0.1  # Eventual consistency
hinted_handoff_enabled: true  # Handle temporary failures
```

## 📚 Extended Concepts

### PACELC Theorem
Extension of CAP that considers latency:

```
If there is a Partition:
  Choose between Availability and Consistency
Else (no partition):
  Choose between Latency and Consistency
```

```mermaid
graph TD
    A[PACELC] --> B[Partition Scenario]
    A --> C[Normal Operation]
    
    B --> D[PA/C Choice]
    C --> E[E/LC Choice]
    
    D --> F[High Availability<br/>vs<br/>Strong Consistency]
    E --> G[Low Latency<br/>vs<br/>Strong Consistency]
    
    style A fill:#87CEEB
    style F fill:#ffffcc
    style G fill:#ffffcc
```

## 🎯 Interview Questions & Answers

### Q: "How would you design a globally distributed chat application?"

**Answer Framework:**
1. **Clarify Requirements**: Real-time messaging, global users, message ordering
2. **Choose AP**: Users expect availability over perfect consistency
3. **Design**: Regional clusters with eventual consistency
4. **Handle Edge Cases**: Conflict resolution, message ordering

### Q: "Why can't we have all three CAP properties?"

**Answer**: In a distributed system, network partitions are inevitable. When they occur, you must choose between:
- Waiting for all nodes (sacrificing availability)
- Proceeding with subset of nodes (sacrificing consistency)

## 🚀 Best Practices

1. **Design for your use case**: Don't default to CP or AP - choose based on business requirements
2. **Measure real-world performance**: Theory vs practice can differ significantly
3. **Plan for partition scenarios**: Have clear procedures for handling splits
4. **Monitor consistency lag**: Set alerting on replication delays
5. **Test partition scenarios**: Regularly test failure modes

---

**Key Takeaway**: CAP Theorem forces you to make explicit trade-offs. The best choice depends on your specific use case and business requirements.
