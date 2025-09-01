# SQL vs NoSQL Databases

Understanding when to use SQL vs NoSQL databases is crucial for system design interviews. Each has distinct advantages and use cases.

## 🏗️ Database Types Overview

```mermaid
graph TD
    A[Database Types] --> B[SQL/Relational]
    A --> C[NoSQL]
    
    B --> D[MySQL]
    B --> E[PostgreSQL]
    B --> F[Oracle]
    B --> G[SQL Server]
    
    C --> H[Document<br/>MongoDB, CouchDB]
    C --> I[Key-Value<br/>Redis, DynamoDB]
    C --> J[Column-Family<br/>Cassandra, HBase]
    C --> K[Graph<br/>Neo4j, Neptune]
    
    style B fill:#87CEEB
    style C fill:#98FB98
```

## 🗃️ SQL Databases (RDBMS)

### Characteristics
- **ACID Properties**: Atomicity, Consistency, Isolation, Durability
- **Schema-based**: Predefined structure with tables, rows, columns
- **SQL Language**: Standardized query language
- **Relationships**: Foreign keys and joins
- **Vertical Scaling**: Scale up with more powerful hardware

### ACID Properties Deep Dive

```mermaid
graph TD
    A[ACID Properties] --> B[Atomicity<br/>All or Nothing]
    A --> C[Consistency<br/>Valid State]
    A --> D[Isolation<br/>Concurrent Safety]
    A --> E[Durability<br/>Permanent Storage]
    
    B --> F[Transaction Success:<br/>All operations complete]
    B --> G[Transaction Failure:<br/>All operations rollback]
    
    C --> H[Data integrity rules<br/>always enforced]
    
    D --> I[Concurrent transactions<br/>don't interfere]
    
    E --> J[Committed data survives<br/>system failures]
    
    style A fill:#87CEEB
```

### When to Use SQL Databases

```mermaid
flowchart TD
    A[Consider SQL When] --> B[Complex Relationships]
    A --> C[ACID Compliance Required]
    A --> D[Complex Queries Needed]
    A --> E[Well-defined Schema]
    A --> F[Strong Consistency Required]
    
    B --> G[E-commerce: Products,<br/>Orders, Customers]
    C --> H[Banking: Transactions,<br/>Account balances]
    D --> I[Analytics: Joins,<br/>Aggregations, Reporting]
    E --> J[Enterprise: Structured<br/>business data]
    F --> K[Financial: No data<br/>inconsistency tolerance]
```

### SQL Example Schema
```sql
-- E-commerce database schema
CREATE TABLE users (
    user_id SERIAL PRIMARY KEY,
    email VARCHAR(255) UNIQUE NOT NULL,
    first_name VARCHAR(100),
    last_name VARCHAR(100),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE products (
    product_id SERIAL PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    description TEXT,
    price DECIMAL(10,2),
    category_id INTEGER REFERENCES categories(category_id),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE orders (
    order_id SERIAL PRIMARY KEY,
    user_id INTEGER REFERENCES users(user_id),
    total_amount DECIMAL(10,2),
    status VARCHAR(50),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE order_items (
    order_item_id SERIAL PRIMARY KEY,
    order_id INTEGER REFERENCES orders(order_id),
    product_id INTEGER REFERENCES products(product_id),
    quantity INTEGER,
    unit_price DECIMAL(10,2)
);
```

## 🚀 NoSQL Databases

### Types of NoSQL Databases

#### 1. Document Databases
Store data in document format (JSON, BSON).

```mermaid
graph TD
    A[Document Database] --> B[Flexible Schema]
    A --> C[Nested Data]
    A --> D[JSON/BSON Format]
    
    E[Example: MongoDB] --> F[Collections]
    F --> G[Documents]
    G --> H[Fields]
    
    style A fill:#98FB98
    style E fill:#47A248
```

**MongoDB Example:**
```json
{
  "_id": "user123",
  "name": "John Doe",
  "email": "john@example.com",
  "address": {
    "street": "123 Main St",
    "city": "New York",
    "country": "USA"
  },
  "orders": [
    {
      "order_id": "order456",
      "total": 99.99,
      "items": ["item1", "item2"]
    }
  ],
  "preferences": {
    "theme": "dark",
    "notifications": true
  }
}
```

#### 2. Key-Value Stores
Simple key-value pairs with high performance.

```mermaid
graph LR
    A[Key-Value Store] --> B[user:123]
    B --> C[User Data JSON]
    
    A --> D[session:abc]
    D --> E[Session Data]
    
    A --> F[cache:product:456]
    F --> G[Product Info]
    
    style A fill:#FFE4B5
```

**Redis Example:**
```redis
SET user:123 '{"name":"John","email":"john@example.com"}'
GET user:123

HSET user:123 name "John" email "john@example.com"
HGET user:123 name

SETEX session:abc 3600 '{"user_id":123,"role":"admin"}'
```

#### 3. Column-Family (Wide Column)
Data stored in column families, excellent for time-series data.

```mermaid
graph TD
    A[Column Family] --> B[Row Key]
    B --> C[Column Family 1]
    B --> D[Column Family 2]
    
    C --> E[Column 1:1]
    C --> F[Column 1:2]
    
    D --> G[Column 2:1]
    D --> H[Column 2:2]
    
    style A fill:#DDA0DD
```

**Cassandra Example:**
```sql
-- Time series data for IoT sensors
CREATE TABLE sensor_data (
    sensor_id text,
    timestamp timestamp,
    temperature float,
    humidity float,
    pressure float,
    PRIMARY KEY (sensor_id, timestamp)
) WITH CLUSTERING ORDER BY (timestamp DESC);

INSERT INTO sensor_data (sensor_id, timestamp, temperature, humidity, pressure)
VALUES ('sensor_001', '2024-01-15 10:30:00', 23.5, 45.2, 1013.25);
```

#### 4. Graph Databases
Optimized for relationships and connections.

```mermaid
graph TD
    A[User: Alice] -->|FOLLOWS| B[User: Bob]
    B -->|FOLLOWS| C[User: Charlie]
    A -->|LIKES| D[Post: Tech News]
    B -->|COMMENTED| D
    C -->|SHARED| D
    
    E[User: Dave] -->|FOLLOWS| A
    E -->|LIKES| F[Post: Travel Blog]
    
    style A fill:#87CEEB
    style B fill:#87CEEB
    style C fill:#87CEEB
    style E fill:#87CEEB
```

**Neo4j Example:**
```cypher
// Create users and relationships
CREATE (alice:User {name: 'Alice', age: 30})
CREATE (bob:User {name: 'Bob', age: 25})
CREATE (alice)-[:FOLLOWS]->(bob)

// Find friends of friends
MATCH (user:User {name: 'Alice'})-[:FOLLOWS]->()-[:FOLLOWS]->(fof)
WHERE NOT (user)-[:FOLLOWS]->(fof)
RETURN fof.name
```

## ⚖️ Detailed Comparison

### Performance Characteristics

| Aspect | SQL | NoSQL |
|--------|-----|-------|
| **Scalability** | Vertical (Scale Up) | Horizontal (Scale Out) |
| **Consistency** | Strong (ACID) | Eventual (BASE) |
| **Schema** | Fixed, Rigid | Flexible, Dynamic |
| **Queries** | Complex Joins, SQL | Simple Queries, API calls |
| **Transactions** | Full ACID support | Limited transaction support |
| **Data Integrity** | High (Foreign keys) | Application-level |

### CAP Theorem Application

```mermaid
graph TD
    A[CAP Theorem] --> B[Consistency]
    A --> C[Availability]
    A --> D[Partition Tolerance]
    
    E[SQL Databases] --> F[CA Systems<br/>Single node]
    E --> G[CP Systems<br/>Distributed SQL]
    
    H[NoSQL Databases] --> I[AP Systems<br/>High availability]
    H --> J[CP Systems<br/>Strong consistency]
    
    style E fill:#87CEEB
    style H fill:#98FB98
```

## 🎯 Use Case Scenarios

### E-commerce Platform Decision Tree

```mermaid
flowchart TD
    A[E-commerce Requirements] --> B{Data Relationships?}
    B -->|Complex| C[User-Product-Order relationships]
    C --> D[SQL Database<br/>PostgreSQL]
    
    B -->|Simple| E{Consistency Requirements?}
    E -->|Strong| F[Financial transactions]
    F --> D
    
    E -->|Eventual| G{Scale Requirements?}
    G -->|High| H[Product catalog, reviews]
    H --> I[NoSQL Database<br/>MongoDB]
    
    G -->|Low| J[Small business]
    J --> D
```

### Real-world Examples

#### Netflix Architecture
```mermaid
graph TD
    A[Netflix Data Strategy] --> B[MySQL<br/>Billing, Account info]
    A --> C[Cassandra<br/>Viewing history, metadata]
    A --> D[Elasticsearch<br/>Search, recommendations]
    A --> E[Redis<br/>Session data, caching]
    
    style B fill:#87CEEB
    style C fill:#DDA0DD
    style D fill:#FFE4B5
    style E fill:#98FB98
```

#### Facebook/Meta Architecture
```mermaid
graph TD
    A[Facebook Data Strategy] --> B[MySQL<br/>User profiles, relationships]
    A --> C[Cassandra<br/>Messages, feeds]
    A --> D[Memcached<br/>Caching layer]
    A --> E[Graph Database<br/>Social connections]
    
    style B fill:#87CEEB
    style C fill:#DDA0DD
    style D fill:#98FB98
    style E fill:#FFE4B5
```

## 📊 Performance Benchmarks

### Read Performance
```
Benchmark: 1M records, simple queries

MySQL: 
- Simple SELECT: 1,000 QPS
- Complex JOIN: 100 QPS

MongoDB:
- Find by ID: 10,000 QPS
- Complex aggregation: 500 QPS

Redis:
- GET operation: 100,000 QPS
- Hash operations: 80,000 QPS

Cassandra:
- Point queries: 15,000 QPS
- Range queries: 5,000 QPS
```

### Write Performance
```
Benchmark: Insert operations

MySQL:
- Single INSERT: 5,000 QPS
- Batch INSERT: 20,000 QPS

MongoDB:
- Single INSERT: 15,000 QPS
- Bulk INSERT: 100,000 QPS

Cassandra:
- Single WRITE: 25,000 QPS
- Batch WRITE: 200,000 QPS
```

## 🔧 Migration Strategies

### SQL to NoSQL Migration

```mermaid
sequenceDiagram
    participant A as Analysis Phase
    participant B as Design Phase
    participant C as Implementation
    participant D as Validation
    
    A->>A: Identify data patterns
    A->>A: Analyze query patterns
    A->>B: Requirements document
    
    B->>B: Choose NoSQL type
    B->>B: Design data model
    B->>C: Migration plan
    
    C->>C: Dual-write setup
    C->>C: Data migration
    C->>C: Application updates
    C->>D: Validation tests
    
    D->>D: Performance testing
    D->>D: Data consistency checks
```

### Polyglot Persistence Strategy

```mermaid
graph TD
    A[Application Layer] --> B[User Service<br/>PostgreSQL]
    A --> C[Product Catalog<br/>MongoDB]
    A --> D[Session Store<br/>Redis]
    A --> E[Analytics<br/>Cassandra]
    A --> F[Search<br/>Elasticsearch]
    A --> G[Recommendations<br/>Neo4j]
    
    style A fill:#87CEEB
```

## 🎯 Decision Framework

### Database Selection Criteria

```mermaid
flowchart TD
    A[Database Selection] --> B{Data Structure}
    B -->|Structured| C{Relationships?}
    B -->|Semi-structured| D[Document DB]
    B -->|Unstructured| E[Object Storage]
    
    C -->|Complex| F[Relational DB]
    C -->|Simple| G{Scale?}
    
    G -->|High| H[NoSQL]
    G -->|Low| I[SQL]
    
    J{Consistency?} --> K[Strong: SQL]
    J --> L[Eventual: NoSQL]
    
    M{Query Complexity?} --> N[Complex: SQL]
    M --> O[Simple: NoSQL]
```

### Specific Use Case Mapping

| Use Case | Database Type | Example |
|----------|---------------|---------|
| **Financial Systems** | SQL | PostgreSQL |
| **Content Management** | Document | MongoDB |
| **Real-time Analytics** | Column-family | Cassandra |
| **Caching** | Key-value | Redis |
| **Social Networks** | Graph | Neo4j |
| **IoT/Time Series** | Time-series | InfluxDB |
| **Full-text Search** | Search Engine | Elasticsearch |

## 🚀 Modern Trends

### NewSQL Databases
Combine benefits of SQL and NoSQL.

```mermaid
graph TD
    A[NewSQL] --> B[ACID Transactions]
    A --> C[Horizontal Scaling]
    A --> D[SQL Interface]
    
    E[Examples] --> F[CockroachDB]
    E --> G[TiDB]
    E --> H[VoltDB]
    
    style A fill:#FFE4B5
```

### Multi-Model Databases
Support multiple data models in one system.

```mermaid
graph TD
    A[Multi-Model DB] --> B[Document]
    A --> C[Graph]
    A --> D[Key-Value]
    A --> E[Column]
    
    F[Examples] --> G[ArangoDB]
    F --> H[CosmosDB]
    F --> I[OrientDB]
    
    style A fill:#DDA0DD
```

## 📚 Best Practices

### For SQL Databases
1. **Normalize properly** but denormalize for performance when needed
2. **Use indexes wisely** - they speed reads but slow writes
3. **Optimize queries** with EXPLAIN plans
4. **Use read replicas** for scaling reads
5. **Implement connection pooling**

### For NoSQL Databases
1. **Design for your access patterns** - queries should drive schema
2. **Denormalize data** to avoid joins
3. **Use appropriate consistency levels**
4. **Plan for eventual consistency**
5. **Monitor performance metrics**

### Hybrid Approach Example
```python
class UserService:
    def __init__(self):
        self.postgres = PostgreSQLConnection()  # User profiles
        self.redis = RedisConnection()          # Session data
        self.mongodb = MongoConnection()        # User preferences
    
    def get_user(self, user_id):
        # Get core user data from PostgreSQL
        user = self.postgres.query("SELECT * FROM users WHERE id = %s", user_id)
        
        # Get session data from Redis
        session = self.redis.get(f"session:{user_id}")
        
        # Get preferences from MongoDB
        preferences = self.mongodb.find_one({"user_id": user_id})
        
        return {
            "user": user,
            "session": session,
            "preferences": preferences
        }
```

---

## 🔗 Related Topics

### Database Section
- [Database Replication](./database-replication.md) - Master-slave and master-master replication strategies
- [Sharding Strategies](./sharding-strategies.md) - Horizontal partitioning for scalability
- [ACID vs BASE](./acid-vs-base.md) - Transaction models comparison
- [Database Federation](./database-federation.md) - Distributed database architectures

### Related System Design Concepts
- [CAP Theorem](./cap-theorem.md) - Consistency, Availability, Partition tolerance trade-offs
- [Consistency Patterns](./consistency-patterns.md) - Strong vs eventual consistency models
- [Performance Metrics](./performance-metrics.md) - Database performance monitoring

**Key Takeaway**: There's no one-size-fits-all database solution. Choose based on your specific requirements for consistency, scalability, query complexity, and data relationships. Many successful systems use multiple database types (polyglot persistence) to optimize for different use cases.

---

[← Back to Main Guide](./README.md) | [← Previous: Message Queues](./message-queues.md) | [Next: Database Replication →](./database-replication.md)
