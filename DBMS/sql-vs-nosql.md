# SQL vs NoSQL

SQL and NoSQL databases serve different use cases based on data structure, scalability requirements, and consistency needs. Understanding their differences helps in choosing the right database technology for specific applications.

## Key Points

- **SQL (Relational)**: Structured data with fixed schema, ACID compliance, complex queries, vertical scaling. Uses tables with rows and columns following relational model. Supports complex JOINs and transactions for data integrity. Mature ecosystem with standardized query language and extensive tooling.
- **NoSQL (Non-relational)**: Flexible schema, horizontal scaling, eventual consistency for big data needs. Designed for modern web applications with varying data structures. Optimized for specific data access patterns and use cases. Better suited for rapid development and agile methodologies.
- **Types of NoSQL**: Document (MongoDB), Key-Value (Redis), Column-family (Cassandra), Graph (Neo4j) databases. Each type optimized for different data models and access patterns. Document stores handle JSON-like structures, key-value for simple lookups. Graph databases excel at relationship-heavy data and traversals.
- **Trade-offs**: Consistency vs Availability, Structure vs Flexibility, Features vs Performance. SQL provides stronger guarantees but less flexibility and scalability. NoSQL offers better performance and scalability but weaker consistency. Choice depends on specific application requirements and constraints.
- **Use Cases**: SQL for transactions and complex queries, NoSQL for big data and rapid development. SQL suited for financial systems, ERP, and traditional business applications. NoSQL better for social media, IoT, analytics, and content management. Modern applications often use both (polyglot persistence).

## Example

```sql
-- SQL Database (PostgreSQL/MySQL)
-- Structured schema with relationships
CREATE TABLE customers (
    customer_id INT PRIMARY KEY,
    name VARCHAR(100),
    email VARCHAR(100) UNIQUE,
    created_at TIMESTAMP
);

CREATE TABLE orders (
    order_id INT PRIMARY KEY,
    customer_id INT,
    total DECIMAL(10,2),
    order_date TIMESTAMP,
    FOREIGN KEY (customer_id) REFERENCES customers(customer_id)
);

-- Complex joins and transactions
BEGIN TRANSACTION;
INSERT INTO customers (customer_id, name, email) VALUES (1, 'John Doe', 'john@email.com');
INSERT INTO orders (order_id, customer_id, total) VALUES (101, 1, 99.99);
COMMIT;

SELECT c.name, o.total, o.order_date
FROM customers c
JOIN orders o ON c.customer_id = o.customer_id
WHERE c.email = 'john@email.com';
```

```javascript
// NoSQL Database Examples

// MongoDB (Document Database)
// Flexible schema, embedded documents
db.customers.insertOne({
    _id: 1,
    name: "John Doe",
    email: "john@email.com",
    orders: [
        {
            order_id: 101,
            total: 99.99,
            order_date: new Date(),
            items: [
                {product: "Laptop", price: 99.99}
            ]
        }
    ]
});

// Query with flexible structure
db.customers.find({"orders.total": {$gt: 50}});

// Redis (Key-Value Database)
// Simple key-value operations
SET user:1:name "John Doe"
SET user:1:email "john@email.com"
HSET user:1:profile name "John Doe" email "john@email.com" age 30

// Cassandra (Column-family)
// Wide column store for time-series data
CREATE TABLE user_activity (
    user_id UUID,
    timestamp TIMESTAMP,
    activity TEXT,
    PRIMARY KEY (user_id, timestamp)
);

INSERT INTO user_activity (user_id, timestamp, activity)
VALUES (uuid(), now(), 'login');
```

**Explanation**: SQL provides structured data with relationships and ACID guarantees, while NoSQL offers flexibility and scalability for different data models.

## Comparison Table

| Aspect | SQL | NoSQL |
|--------|-----|-------|
| **Schema** | Fixed, predefined | Flexible, dynamic |
| **Scaling** | Vertical (more powerful hardware) | Horizontal (more servers) |
| **ACID** | Full ACID compliance | Eventual consistency |
| **Queries** | Complex SQL, JOINs | Simple queries, limited JOINs |
| **Use Cases** | Financial, ERP, CRM | Social media, IoT, analytics |
| **Examples** | PostgreSQL, MySQL, Oracle | MongoDB, Cassandra, Redis |

## Common Interview Questions

### Q1: When would you choose SQL over NoSQL?
**Answer**: Choose SQL when you need:
- **ACID transactions**: Banking, financial systems
- **Complex relationships**: Multiple table JOINs
- **Strong consistency**: Inventory management
- **Mature ecosystem**: Established tools and expertise
- **Complex queries**: Reporting and analytics

Example: E-commerce order processing with payments requires ACID properties.

### Q2: What are the main types of NoSQL databases?
**Answer**:
- **Document**: MongoDB, CouchDB (JSON-like documents)
- **Key-Value**: Redis, DynamoDB (simple key-value pairs)
- **Column-family**: Cassandra, HBase (wide column storage)
- **Graph**: Neo4j, Amazon Neptune (nodes and relationships)

Each type optimizes for different data access patterns and use cases.

### Q3: How do NoSQL databases achieve scalability?
**Answer**: NoSQL databases scale horizontally through:
- **Sharding**: Distributing data across multiple servers
- **Replication**: Creating copies for fault tolerance
- **Eventual consistency**: Allowing temporary inconsistencies
- **Partitioning**: Dividing data based on keys or geography

This contrasts with SQL's traditional vertical scaling approach.

### Q4: Can you use SQL with NoSQL databases?
**Answer**: Yes, many NoSQL databases now support SQL-like interfaces:
- **MongoDB**: MongoDB Query Language (MQL) and SQL connector
- **Cassandra**: CQL (Cassandra Query Language)
- **DynamoDB**: PartiQL
- **Elasticsearch**: SQL plugin

However, these interfaces may not support all SQL features like complex JOINs.

### Q5: What is polyglot persistence?
**Answer**: Polyglot persistence is using multiple database technologies within a single application:
```
E-commerce Application:
├── User profiles → PostgreSQL (relational data)
├── Product catalog → Elasticsearch (search)
├── Shopping cart → Redis (session data)
├── Recommendations → Neo4j (graph relationships)
└── Analytics → Cassandra (time-series data)
```

Each database is chosen based on specific data requirements and access patterns.

---
[← Back to Main Guide](./README.md)
