# Sharding vs Replication

Sharding and replication are fundamental techniques for scaling databases. Sharding distributes data across multiple servers for horizontal scaling, while replication creates copies for redundancy and read performance.

## Key Points

- **Sharding**: Horizontal partitioning of data across multiple databases for scaling write operations. Distributes both data and load across multiple servers independently. Each shard contains subset of total data based on partitioning key. Enables linear scaling of storage capacity and write throughput.
- **Replication**: Creating copies of data on multiple servers for redundancy and read scaling. All replicas contain complete copy of data for fault tolerance. Primary handles writes while replicas can serve read operations. Improves read performance and provides disaster recovery capabilities.
- **Sharding**: Scales write capacity and storage by distributing load across multiple nodes. Each shard can handle independent write operations simultaneously. Storage capacity grows linearly with number of shards added. More complex to implement but provides true horizontal scaling.
- **Replication**: Improves read performance and availability through multiple data copies. Read operations can be distributed across multiple replica servers. Provides fault tolerance if primary server fails. Simpler to implement than sharding but doesn't scale write operations.
- **Shard key**: Determines how data is distributed across shards, critical for performance. Good shard key ensures even distribution and avoids hot spots. Should be immutable and frequently used in queries. Poor shard key choice can lead to unbalanced shards and performance issues.
- **Replication lag**: Delay between primary and replica updates, affects consistency. Asynchronous replication has shorter lag but potential data loss. Synchronous replication ensures consistency but impacts performance. Applications must handle potential data staleness in replicas.

## Example

```sql
-- Sharding Example
-- User data sharded by user_id
-- Shard 1 (user_id 1-1000):
CREATE TABLE users_shard1 (
    user_id INT PRIMARY KEY CHECK (user_id BETWEEN 1 AND 1000),
    name VARCHAR(100),
    email VARCHAR(100)
);

-- Shard 2 (user_id 1001-2000):
CREATE TABLE users_shard2 (
    user_id INT PRIMARY KEY CHECK (user_id BETWEEN 1001 AND 2000),
    name VARCHAR(100), 
    email VARCHAR(100)
);

-- Application determines shard based on user_id
function getShardForUser(user_id) {
    if (user_id <= 1000) return 'shard1';
    else return 'shard2';
}

-- Replication Example (Master-Slave)
-- Primary (Master) Database - handles all writes
INSERT INTO users (user_id, name, email) VALUES (1, 'John', 'john@email.com');
UPDATE users SET name = 'John Doe' WHERE user_id = 1;

-- Secondary (Slave) Databases - handle reads
-- Replica 1:
SELECT * FROM users WHERE user_id = 1;

-- Replica 2: 
SELECT COUNT(*) FROM users WHERE created_at > '2024-01-01';

-- Geographic Replication
-- US Region (Primary):
INSERT INTO products (id, name, price) VALUES (1, 'Laptop', 999.99);

-- EU Region (Replica):
-- Data replicated asynchronously
SELECT * FROM products WHERE id = 1; -- May have slight delay

-- Sharding with Replication (Combined)
-- Shard 1 with 2 replicas:
-- Primary: Shard1-Master (writes)
-- Replicas: Shard1-Replica1, Shard1-Replica2 (reads)

-- Shard 2 with 2 replicas:  
-- Primary: Shard2-Master (writes)
-- Replicas: Shard2-Replica1, Shard2-Replica2 (reads)
```

**Explanation**: Sharding distributes data across servers to scale writes and storage, while replication creates copies to scale reads and improve availability.

## Architecture Patterns

```
SHARDING:
Client → Router → [Shard1] [Shard2] [Shard3]
         ↓
    Determines which shard
    based on shard key

REPLICATION:
Client → [Primary] → [Replica1]
         ↓           [Replica2]
    Writes go to primary,
    reads can go to replicas

COMBINED (Sharded + Replicated):
Client → Router → Shard1[Primary→Replica]
                 Shard2[Primary→Replica]
                 Shard3[Primary→Replica]
```

## Common Interview Questions

### Q1: What's the difference between horizontal and vertical sharding?
**Answer**:
- **Horizontal Sharding (Row-based)**: Split table rows across servers
  ```sql
  -- Users 1-1000 → Server 1
  -- Users 1001-2000 → Server 2
  ```
- **Vertical Sharding (Column-based)**: Split table columns across servers
  ```sql
  -- Basic info (id, name) → Server 1  
  -- Extended info (bio, preferences) → Server 2
  ```
Horizontal is more common for scaling; vertical is used for separating hot/cold data.

### Q2: What are the challenges of sharding?
**Answer**: Key challenges include:
- **Cross-shard queries**: JOINs across shards are expensive
- **Rebalancing**: Moving data when adding/removing shards
- **Hot spots**: Uneven data distribution causing bottlenecks
- **Complexity**: Application logic must handle shard routing
- **Consistency**: Maintaining ACID properties across shards

### Q3: How do you choose a good shard key?
**Answer**: Good shard key characteristics:
- **High cardinality**: Many unique values for even distribution
- **Query-friendly**: Commonly used in WHERE clauses
- **Immutable**: Shouldn't change after record creation
- **Balanced**: Avoids hot spots

Examples:
- **Good**: user_id, hash(email), customer_id
- **Bad**: timestamp (creates hot spots), boolean fields (low cardinality)

### Q4: What's the difference between synchronous and asynchronous replication?
**Answer**:
- **Synchronous**: Primary waits for replica acknowledgment before committing
  - Pros: Strong consistency, no data loss
  - Cons: Higher latency, reduced availability if replica fails
  
- **Asynchronous**: Primary commits immediately, updates replicas later
  - Pros: Better performance, higher availability
  - Cons: Potential data loss, eventual consistency

### Q5: How do you handle failover in master-slave replication?
**Answer**: Failover process:
1. **Detection**: Monitor master health (heartbeat, timeout)
2. **Election**: Choose new master from replicas (most up-to-date)
3. **Promotion**: Promote chosen replica to master
4. **Redirect**: Update application to use new master
5. **Recovery**: When old master recovers, make it a replica

```sql
-- Example failover scenario
-- Before: Master (Server1) → Replica (Server2)
-- After failover: Master (Server2) → Replica (Server1)
```

Automated failover reduces downtime but requires careful coordination to prevent split-brain scenarios.

---
[← Back to Main Guide](./README.md)
