# System Design Interview Questions

This guide covers system design interview questions commonly asked at **Samsung**, **Oracle**, and **Fastenal**, along with detailed approaches and solutions.

## 🏢 Company-Specific Questions

### Samsung Interview Questions

#### 1. Design a Distributed Cache System
**Context**: Samsung's cloud services need efficient caching for global applications.

##### Clarifying Questions
- What type of data will be cached? (User sessions, API responses, computed results)
- What's the expected cache size? (GB, TB, PB scale)
- What's the read/write ratio? (Typically 80:20 for cache)
- What's the required latency? (<1ms for local, <50ms for distributed)
- Geographic distribution requirements? (Global, regional)

##### Requirements
**Functional:**
- Store and retrieve key-value pairs
- Support TTL (Time To Live)
- Cache eviction policies (LRU, LFU, Random)
- Distributed across multiple nodes

**Non-Functional:**
- Low latency: <1ms for reads, <5ms for writes
- High availability: 99.9%
- Scalability: Handle millions of requests/second
- Consistency: Eventual consistency acceptable

##### High-Level Design

```mermaid
graph TB
    A[Client Applications] --> B[Load Balancer]
    B --> C[Cache Proxy Layer]
    C --> D[Consistent Hashing Ring]
    
    D --> E[Cache Node 1]
    D --> F[Cache Node 2]
    D --> G[Cache Node 3]
    D --> H[Cache Node N]
    
    E --> I[Memory<br/>LRU Eviction]
    F --> J[Memory<br/>LRU Eviction]
    G --> K[Memory<br/>LRU Eviction]
    H --> L[Memory<br/>LRU Eviction]
    
    M[Monitoring] --> N[Metrics Collection]
    M --> O[Health Checks]
    
    style C fill:#87CEEB
    style D fill:#98FB98
```

##### Detailed Design

**Consistent Hashing Implementation:**
```python
import hashlib
import bisect

class ConsistentHash:
    def __init__(self, nodes=None, replicas=3):
        self.replicas = replicas
        self.ring = {}
        self.sorted_keys = []
        
        if nodes:
            for node in nodes:
                self.add_node(node)
    
    def _hash(self, key):
        return int(hashlib.sha256(key.encode()).hexdigest(), 16)
    
    def add_node(self, node):
        for i in range(self.replicas):
            virtual_key = self._hash(f"{node}:{i}")
            self.ring[virtual_key] = node
            bisect.insort(self.sorted_keys, virtual_key)
    
    def get_node(self, key):
        if not self.ring:
            return None
        
        hash_key = self._hash(key)
        idx = bisect.bisect_right(self.sorted_keys, hash_key)
        if idx == len(self.sorted_keys):
            idx = 0
        return self.ring[self.sorted_keys[idx]]
```

**Cache Node Architecture:**
```mermaid
graph TD
    A[Cache Request] --> B[Request Router]
    B --> C[Local Cache Check]
    C --> D{Cache Hit?}
    D -->|Yes| E[Return Data]
    D -->|No| F[Check Remote Nodes]
    F --> G[Update Local Cache]
    G --> E
    
    H[Eviction Manager] --> I[LRU Policy]
    I --> J[Remove Least Used]
    
    style D fill:#ffffcc
    style E fill:#ccffcc
```

##### API Design
```http
# Cache Operations
PUT /cache/{key}
Content-Type: application/json
TTL: 3600

{
  "value": "cached_data",
  "metadata": {
    "created_at": "2024-01-15T10:30:00Z"
  }
}

GET /cache/{key}
Response: 200 OK / 404 Not Found

DELETE /cache/{key}
Response: 204 No Content

# Administrative
GET /cache/stats
GET /cache/health
POST /cache/flush
```

##### Scale Estimations
```
Assumptions:
- 100M requests/day
- Average key size: 250 bytes
- Average value size: 1KB
- Cache hit ratio: 85%

QPS: 100M / (24 * 3600) = 1,157 requests/second
Peak QPS (3x): 3,471 requests/second

Memory per node: 64GB
Total cache size needed: 100GB
Number of nodes: 2 (with replication factor 2)

Network bandwidth:
- Reads: 3,471 * 0.85 * 1KB = 2.9 MB/s
- Writes: 3,471 * 0.15 * 1KB = 0.5 MB/s
```

---

#### 2. Design a Real-time Analytics Dashboard
**Context**: Samsung needs to monitor IoT device performance across global manufacturing plants.

##### Clarifying Questions
- How many devices are we monitoring? (Millions of IoT devices)
- What metrics do we track? (Temperature, humidity, performance, errors)
- What's the data ingestion rate? (10K events/second per factory)
- Real-time requirements? (Dashboard updates every 5 seconds)
- Historical data retention? (1 year for detailed, 5 years aggregated)

##### Requirements
**Functional:**
- Ingest real-time IoT data streams
- Process and aggregate metrics
- Display real-time dashboards
- Alert on anomalies
- Support historical queries

**Non-Functional:**
- Handle 1M events/second globally
- Dashboard latency: <5 seconds
- 99.95% availability
- Horizontal scalability

##### Architecture Design

```mermaid
graph TB
    A[IoT Devices] --> B[Message Queue<br/>Apache Kafka]
    B --> C[Stream Processing<br/>Apache Flink]
    C --> D[Real-time Store<br/>Redis]
    C --> E[Time Series DB<br/>InfluxDB]
    
    F[Dashboard API] --> D
    F --> E
    G[Web Dashboard] --> F
    
    H[Alerting Service] --> I[Notification Queue]
    C --> H
    
    J[Batch Processing<br/>Apache Spark] --> E
    J --> K[Data Warehouse<br/>BigQuery]
    
    style B fill:#DDA0DD
    style C fill:#98FB98
    style D fill:#FFE4B5
```

---

### Oracle Interview Questions

#### 3. Design a Notification System for Millions of Users
**Context**: Oracle Cloud needs to send notifications across multiple channels.

##### System Architecture

```mermaid
graph TB
    A[Application Services] --> B[Notification API Gateway]
    B --> C[Message Queue<br/>Apache Kafka]
    
    C --> D[Email Service]
    C --> E[SMS Service] 
    C --> F[Push Service]
    C --> G[In-App Service]
    
    D --> H[Email Providers<br/>SendGrid, SES]
    E --> I[SMS Providers<br/>Twilio, SNS]
    F --> J[Push Providers<br/>FCM, APNS]
    
    K[User Preferences DB] --> B
    L[Template Service] --> D
    L --> E
    L --> F
    
    M[Analytics DB] --> N[Delivery Tracking]
    D --> N
    E --> N
    F --> N
    
    style C fill:#DDA0DD
    style B fill:#87CEEB
```

##### Notification Flow
```mermaid
sequenceDiagram
    participant A as Application
    participant API as Notification API
    participant Q as Message Queue
    participant PS as Push Service
    participant SMS as SMS Service
    participant E as Email Service
    
    A->>API: Send Notification Request
    API->>API: Validate & Enrich
    API->>Q: Queue Message
    
    par Push Notification
        Q->>PS: Process Push
        PS->>PS: Send to FCM/APNS
    and SMS Notification
        Q->>SMS: Process SMS
        SMS->>SMS: Send via Twilio
    and Email Notification
        Q->>E: Process Email
        E->>E: Send via SendGrid
    end
    
    Note over PS,E: Track delivery status
```

---

#### 4. Design a Global File Storage System (Dropbox-like)
**Context**: Oracle Cloud Storage competing with Dropbox/Google Drive.

##### Architecture Overview

```mermaid
graph TB
    A[Client Apps] --> B[CDN/Edge Servers]
    B --> C[API Gateway]
    C --> D[File Service]
    C --> E[Metadata Service]
    C --> F[Sync Service]
    
    D --> G[Object Storage<br/>S3/OCI]
    E --> H[Metadata DB<br/>PostgreSQL]
    F --> I[Version Control DB]
    
    J[Search Service] --> K[Elasticsearch]
    L[Sharing Service] --> M[Permissions DB]
    
    N[Sync Queue] --> O[Real-time Sync]
    
    style G fill:#FFE4B5
    style B fill:#87CEEB
    style N fill:#DDA0DD
```

##### File Upload Flow
```mermaid
sequenceDiagram
    participant C as Client
    participant API as API Gateway
    participant FS as File Service
    participant S3 as Object Storage
    participant DB as Metadata DB
    participant SQ as Sync Queue
    
    C->>API: Upload Request
    API->>FS: Generate Upload URL
    FS->>S3: Get Presigned URL
    S3->>FS: Return URL
    FS->>API: Return Upload URL
    API->>C: Upload URL
    
    C->>S3: Upload File Directly
    S3->>FS: Upload Complete Webhook
    FS->>DB: Save File Metadata
    FS->>SQ: Queue Sync Event
    
    SQ->>C: Notify Other Devices
```

---

### Fastenal Interview Questions

#### 5. Design an Inventory Management System
**Context**: Fastenal needs to track millions of items across thousands of locations.

##### Requirements Analysis
**Functional:**
- Track inventory levels across locations
- Handle purchase orders and receipts
- Support inventory transfers
- Generate reorder alerts
- Barcode/RFID scanning support

**Non-Functional:**
- Handle 10K transactions/second
- Real-time inventory updates
- 99.9% availability
- Support for 3000+ locations

##### System Architecture

```mermaid
graph TB
    A[Point of Sale] --> B[API Gateway]
    A1[Warehouse Scanners] --> B
    A2[Mobile Apps] --> B
    
    B --> C[Inventory Service]
    B --> D[Order Service]
    B --> E[Location Service]
    
    C --> F[Inventory DB<br/>Sharded by Location]
    D --> G[Order DB]
    E --> H[Location DB]
    
    I[Event Stream<br/>Kafka] --> J[Analytics Service]
    I --> K[Reorder Service]
    I --> L[Reporting Service]
    
    C --> I
    D --> I
    
    M[External Systems] --> N[EDI Gateway]
    N --> D
    
    style F fill:#FFE4B5
    style I fill:#DDA0DD
```

##### Database Schema

```mermaid
erDiagram
    Location ||--o{ Inventory : "has"
    Product ||--o{ Inventory : "tracked_in"
    Product ||--o{ OrderItem : "ordered"
    Order ||--o{ OrderItem : "contains"
    Supplier ||--o{ Product : "supplies"
    
    Location {
        int location_id PK
        varchar location_code UK
        varchar name
        varchar address
        varchar type
        boolean is_active
    }
    
    Product {
        int product_id PK
        varchar sku UK
        varchar name
        varchar description
        decimal unit_price
        varchar category
        int supplier_id FK
    }
    
    Inventory {
        int inventory_id PK
        int location_id FK
        int product_id FK
        int quantity_on_hand
        int quantity_reserved
        int reorder_point
        int max_quantity
        timestamp last_updated
    }
    
    Order {
        int order_id PK
        int location_id FK
        varchar order_type
        varchar status
        timestamp created_at
        decimal total_amount
    }
    
    OrderItem {
        int order_item_id PK
        int order_id FK
        int product_id FK
        int quantity
        decimal unit_price
    }
```

---

#### 6. Design a Supply Chain Tracking System
**Context**: Track products from suppliers through warehouses to customers.

##### End-to-End Tracking Flow

```mermaid
graph LR
    A[Supplier] --> B[Manufacturing]
    B --> C[Warehouse]
    C --> D[Distribution Center]
    D --> E[Local Store]
    E --> F[Customer]
    
    G[RFID/Barcode] --> H[Tracking Events]
    H --> I[Event Processing]
    I --> J[Location Updates]
    
    K[Real-time Dashboard] --> J
    L[Customer Portal] --> J
    
    style H fill:#DDA0DD
    style I fill:#98FB98
```

---

#### 7. Design a Warehouse Management System
**Context**: Optimize warehouse operations for Fastenal's distribution centers.

##### Warehouse Layout Optimization

```mermaid
graph TD
    A[Receiving Dock] --> B[Inspection Area]
    B --> C[Put-away Queue]
    C --> D[Storage Locations]
    
    E[Pick Request] --> F[Route Optimization]
    F --> G[Pick Path Generation]
    G --> H[Picker Assignment]
    
    D --> I[Pick Locations]
    I --> J[Packing Area]
    J --> K[Shipping Dock]
    
    L[WMS Algorithm] --> F
    L --> M[Inventory Positioning]
    
    style F fill:#87CEEB
    style L fill:#98FB98
```

## 🎯 General Interview Tips

### 1. Clarification Questions Framework
Always start with these categories:
- **Scale**: How many users/requests/data?
- **Features**: What specific functionality is needed?
- **Performance**: Latency and throughput requirements?
- **Consistency**: Strong vs eventual consistency needs?

### 2. Design Process
1. **Requirements Gathering** (5 minutes)
2. **Capacity Estimation** (5 minutes)
3. **High-Level Design** (10 minutes)
4. **Detailed Design** (15 minutes)
5. **Scale & Bottlenecks** (10 minutes)

### 3. Common Follow-up Questions

#### For Samsung (Hardware/IoT Focus)
- "How would you handle device firmware updates?"
- "Design for offline-first mobile applications"
- "Handle real-time sensor data processing"

#### For Oracle (Enterprise/Database Focus)
- "How would you ensure ACID properties at scale?"
- "Design database migration with zero downtime"
- "Handle multi-tenant architecture challenges"

#### For Fastenal (Supply Chain/Inventory Focus)
- "How would you handle seasonal demand spikes?"
- "Design for just-in-time inventory management"
- "Handle supplier integration challenges"

### 4. Technical Deep Dives

#### Database Selection Framework
```mermaid
flowchart TD
    A[Data Requirements] --> B{ACID needed?}
    B -->|Yes| C[SQL Database]
    B -->|No| D{Scale > 1TB?}
    D -->|Yes| E[NoSQL Database]
    D -->|No| F[SQL Database]
    
    C --> G[PostgreSQL/MySQL]
    E --> H{Data Model?}
    H -->|Document| I[MongoDB]
    H -->|Key-Value| J[Cassandra/DynamoDB]
    H -->|Graph| K[Neo4j]
    H -->|Time Series| L[InfluxDB]
```

#### Caching Strategy Decision Tree
```mermaid
flowchart TD
    A[Caching Need] --> B{Data Type?}
    B -->|Static| C[CDN]
    B -->|Dynamic| D{Update Frequency?}
    D -->|High| E[Cache-Aside]
    D -->|Medium| F[Write-Through]
    D -->|Low| G[Write-Behind]
    
    H{Consistency?} --> I[Strong: Write-Through]
    H --> J[Eventual: Cache-Aside]
```

## 📊 Estimation Templates

### Storage Calculation
```
Daily Active Users: X
Average data per user per day: Y
Retention period: Z years

Total storage = X × Y × 365 × Z
With replication factor 3: Total × 3
With growth (20% yearly): Total × (1.2^Z)
```

### Bandwidth Calculation
```
Peak QPS: X requests/second
Average response size: Y KB
Peak bandwidth = X × Y KB/s

Convert to Mbps: (X × Y × 8) / 1024
```

### Server Calculation
```
Peak QPS: X
Server capacity: Y QPS
Number of servers needed: X / Y
With redundancy (N+1): (X / Y) + 1
```

---

**Success Tips**: 
1. Practice drawing diagrams quickly
2. Think out loud during the interview
3. Always consider trade-offs
4. Start simple, then add complexity
5. Ask about constraints and edge cases
