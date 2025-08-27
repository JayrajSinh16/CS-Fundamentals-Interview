# CAP Theorem

CAP Theorem states that in a distributed database system, you can guarantee at most two out of three properties: Consistency, Availability, and Partition tolerance. This fundamental principle guides distributed system design decisions.

## Key Points

- **Consistency (C)**: All nodes see the same data simultaneously across distributed system. Requires coordination between nodes to maintain identical state. Strong consistency guarantees immediate visibility of updates everywhere. Essential for applications requiring strict data accuracy like financial systems.
- **Availability (A)**: System remains operational and responsive to requests even during failures. Means every request receives a response without guarantee of most recent data. Critical for user-facing applications that cannot tolerate downtime. Often prioritized in web applications and real-time systems.
- **Partition Tolerance (P)**: System continues despite network failures between nodes. Network partitions are inevitable in distributed systems due to hardware/software failures. System must function even when nodes cannot communicate. Fundamental requirement for any truly distributed database system.
- **Trade-off**: Must choose two properties when network partitions occur in distributed systems. During normal operation, all three can be maintained simultaneously. Partition forces choice between consistency (reject operations) and availability (allow potentially inconsistent operations). Design decision affects application behavior during network failures.
- **CP Systems**: Consistent but may be unavailable during partitions, prioritize data accuracy. Choose to block operations rather than risk inconsistent data. Common in financial and transactional systems where correctness is paramount. Examples include traditional RDBMS with synchronous replication.
- **AP Systems**: Available but may have inconsistent data during partitions, prioritize service continuity. Continue serving requests even with potential temporary inconsistencies. Eventually achieve consistency when partition heals. Popular in web applications and social media platforms.

## Example

```sql
-- Traditional RDBMS (CA System)
-- Strong consistency and availability, but not partition tolerant
CREATE TABLE users (id INT PRIMARY KEY, name VARCHAR(50), email VARCHAR(100));

-- All replicas must agree before write completes
INSERT INTO users VALUES (1, 'John Doe', 'john@email.com');
-- If network partition occurs, system becomes unavailable

-- NoSQL Examples:

-- MongoDB (CP System)
-- Chooses consistency over availability during partitions
db.users.insert({_id: 1, name: "John Doe", email: "john@email.com"}, 
                {writeConcern: {w: "majority"}});
-- Waits for majority of replicas to acknowledge write
-- During partition, minority nodes become unavailable

-- Cassandra (AP System) 
-- Chooses availability over consistency during partitions
INSERT INTO users (id, name, email) VALUES (1, 'John Doe', 'john@email.com');
-- Write succeeds even if some replicas are unreachable
-- Data may be inconsistent across replicas temporarily

-- DynamoDB (AP with eventual consistency)
-- Available during partitions, eventual consistency
put_item(
    TableName='users',
    Item={'id': 1, 'name': 'John Doe', 'email': 'john@email.com'}
)
-- Write succeeds immediately, consistency achieved later
```

**Explanation**: Different database systems make different CAP trade-offs based on their design goals and use cases.

## CAP Theorem Scenarios

```
Normal Operation (No Partition):
Node A ↔ Node B ↔ Node C
All three properties can be maintained

Network Partition Occurs:
Node A ↔ Node B    |    Node C (isolated)

CP Choice (Consistency + Partition Tolerance):
- Reject writes to maintain consistency
- System becomes unavailable for writes
- Example: Traditional RDBMS with strict replication

AP Choice (Availability + Partition Tolerance):
- Accept writes on both sides
- Allow temporary inconsistency
- Example: NoSQL systems like Cassandra
```

## Common Interview Questions

### Q1: Why can't a distributed system guarantee all three CAP properties?
**Answer**: During a network partition, you must choose:
- **Consistency**: Reject operations to prevent inconsistent data (system becomes unavailable)
- **Availability**: Continue operations on both sides (data becomes inconsistent)

You cannot maintain both consistency and availability simultaneously during a partition because nodes cannot communicate to coordinate.

### Q2: How do modern databases handle CAP trade-offs?
**Answer**: Modern systems often provide configurable trade-offs:
- **MongoDB**: Configurable read/write concerns (CP by default, can tune toward AP)
- **Cassandra**: Tunable consistency levels (eventual, one, quorum, all)
- **DynamoDB**: Strongly consistent vs eventually consistent reads
- **PostgreSQL**: Synchronous vs asynchronous replication

### Q3: What's the difference between strong consistency and eventual consistency?
**Answer**:
- **Strong Consistency**: All reads receive the most recent write immediately
  ```sql
  -- Write to all replicas before acknowledging
  WRITE(x = 5) → All replicas = 5 → ACK
  READ(x) → Returns 5 (guaranteed)
  ```
- **Eventual Consistency**: System will become consistent over time
  ```sql
  -- Write to one replica, propagate asynchronously
  WRITE(x = 5) → Replica A = 5 → ACK
  READ(x) → May return old value initially, eventually returns 5
  ```

### Q4: Can you give examples of CP vs AP systems in practice?
**Answer**:
- **CP Systems**: 
  - Traditional RDBMS (PostgreSQL, MySQL with sync replication)
  - MongoDB (with majority write concern)
  - Redis Cluster
  
- **AP Systems**:
  - Cassandra
  - DynamoDB
  - Riak
  - CouchDB

### Q5: How does CAP Theorem relate to ACID properties?
**Answer**: 
- **CAP Consistency**: All nodes see same data (distributed consistency)
- **ACID Consistency**: Database satisfies integrity constraints (single-node consistency)

They address different aspects:
- ACID focuses on single-node transaction properties
- CAP focuses on distributed system trade-offs during network failures

A distributed system can be ACID-compliant on each node while making CAP trade-offs across nodes.

---
[← Back to Main Guide](./README.md)
