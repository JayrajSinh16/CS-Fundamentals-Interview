# WhatsApp System Design

Design a real-time messaging platform like WhatsApp that supports billions of users with end-to-end encryption and multimedia messaging.

## 📋 Problem Statement

Design WhatsApp that supports:
- Real-time messaging between users
- Group chats with up to 256 members
- Media sharing (images, videos, documents)
- Voice and video calls
- Status updates (Stories)
- End-to-end encryption
- Message delivery confirmations

## 🎯 Requirements

### Functional Requirements
- **1-to-1 Messaging**: Private conversations between users
- **Group Messaging**: Groups with up to 256 members
- **Media Sharing**: Photos, videos, documents, voice messages
- **Real-time Delivery**: Messages delivered instantly
- **Message Status**: Sent, delivered, read indicators
- **Online Presence**: Show user online/offline status
- **Message History**: Store and sync message history
- **Push Notifications**: Notify users of new messages

### Non-Functional Requirements
- **Scale**: 2+ billion users, 100 billion messages daily
- **Latency**: <100ms message delivery
- **Availability**: 99.99% uptime
- **Security**: End-to-end encryption
- **Global**: Worldwide service
- **Storage**: 1 year message retention

## 📊 Capacity Estimation

### User & Message Statistics
```
Total Users: 2B
Daily Active Users: 1.5B
Messages per user per day: 50
Total daily messages: 75B

Message breakdown:
- Text messages: 60B (80%)
- Images: 10B (13%)
- Videos: 3B (4%)
- Documents: 2B (3%)
```

### Storage Requirements
```
Text messages: 60B × 100 bytes = 6TB/day
Images: 10B × 200KB = 2PB/day
Videos: 3B × 2MB = 6PB/day
Documents: 2B × 500KB = 1PB/day

Total daily storage: ~9PB/day
Annual storage: 9PB × 365 = 3.3EB/year

With compression and deduplication: ~1EB/year
```

### Bandwidth Requirements
```
Peak messages: 1M messages/second
Average message size: 2KB
Peak bandwidth: 2GB/second
```

## 🏗️ High-Level Architecture

```mermaid
graph TB
    A[Mobile Clients] --> B[CDN/Edge Servers]
    B --> C[Load Balancer]
    C --> D[API Gateway]
    
    D --> E[Message Service]
    D --> F[User Service]
    D --> G[Media Service]
    D --> H[Notification Service]
    D --> I[Presence Service]
    
    E --> J[Message Queue<br/>Kafka]
    E --> K[(Message DB<br/>Cassandra)]
    
    F --> L[(User DB<br/>PostgreSQL)]
    G --> M[Object Storage<br/>S3]
    H --> N[Push Service<br/>FCM/APNS]
    I --> O[Redis Cache]
    
    P[WebSocket Servers] --> Q[Connection Pool]
    A <--> P
    
    style P fill:#87CEEB
    style J fill:#DDA0DD
    style O fill:#98FB98
```

## 🗄️ Database Design

### User Entity
```mermaid
erDiagram
    User {
        bigint user_id PK
        varchar phone_number UK
        varchar name
        varchar profile_picture_url
        timestamp last_seen
        varchar status_message
        boolean is_online
        timestamp created_at
        varchar public_key
        varchar device_id
    }
```

### Message Entity
```mermaid
erDiagram
    Message {
        uuid message_id PK
        bigint sender_id FK
        bigint receiver_id FK
        varchar chat_id FK
        text content
        varchar message_type
        varchar media_url
        timestamp created_at
        timestamp delivered_at
        timestamp read_at
        boolean is_deleted
        varchar encryption_key_id
    }
```

### Complete ER Diagram
```mermaid
erDiagram
    User ||--o{ Message : "sends"
    User ||--o{ GroupMember : "joins"
    User ||--o{ Contact : "has"
    Group ||--o{ Message : "contains"
    Group ||--o{ GroupMember : "has"
    Message ||--o{ MessageStatus : "tracks"
    
    User {
        bigint user_id PK
        varchar phone_number UK
        varchar name
        varchar profile_picture_url
        timestamp last_seen
        varchar status_message
        boolean is_online
        varchar public_key
        varchar device_id
        timestamp created_at
    }
    
    Message {
        uuid message_id PK
        bigint sender_id FK
        varchar chat_id
        text encrypted_content
        varchar message_type
        varchar media_url
        timestamp created_at
        boolean is_deleted
        varchar reply_to_message_id
    }
    
    Group {
        varchar group_id PK
        varchar group_name
        varchar description
        varchar group_picture_url
        bigint created_by FK
        timestamp created_at
        int max_members
        varchar group_type
    }
    
    GroupMember {
        varchar group_id FK
        bigint user_id FK
        varchar role
        timestamp joined_at
        boolean is_admin
    }
    
    Contact {
        bigint user_id FK
        bigint contact_user_id FK
        varchar contact_name
        timestamp added_at
        boolean is_blocked
    }
    
    MessageStatus {
        uuid message_id FK
        bigint user_id FK
        varchar status
        timestamp status_time
    }
```

## 🔄 Core Workflows

### Message Sending Flow
```mermaid
sequenceDiagram
    participant A as User A
    participant WS as WebSocket Server
    participant MS as Message Service
    participant Q as Message Queue
    participant DB as Message DB
    participant NS as Notification Service
    participant B as User B
    
    A->>WS: Send Message
    WS->>MS: Process Message
    
    Note over MS: Generate message_id
    Note over MS: Encrypt content
    
    MS->>DB: Store Message
    MS->>Q: Queue for Delivery
    
    par Delivery Path
        Q->>WS: Deliver to User B
        WS->>B: Real-time Message
        B->>WS: Delivery Confirmation
        WS->>MS: Update Status
    and Notification Path
        Q->>NS: Send Push Notification
        NS->>B: Push Notification
    end
    
    MS->>WS: Confirm Sent
    WS->>A: Message Sent ✓
```

### Group Message Flow
```mermaid
sequenceDiagram
    participant U as User
    participant MS as Message Service
    participant GS as Group Service
    participant Q as Message Queue
    participant M1 as Member 1
    participant M2 as Member 2
    participant MN as Member N
    
    U->>MS: Send Group Message
    MS->>GS: Get Group Members
    GS->>MS: Return Member List
    
    Note over MS: Create message copies
    Note over MS: Encrypt for each member
    
    MS->>Q: Queue Multiple Deliveries
    
    par Parallel Delivery
        Q->>M1: Deliver Message
        Q->>M2: Deliver Message
        Q->>MN: Deliver Message
    end
    
    Note over MS: Track delivery status
    MS->>U: Group Message Sent ✓
```

### Online Presence Flow
```mermaid
sequenceDiagram
    participant U as User
    participant WS as WebSocket Server
    participant PS as Presence Service
    participant C as Redis Cache
    participant F as Friends
    
    U->>WS: Connect
    WS->>PS: User Online
    PS->>C: Update Status
    
    Note over PS: Get user's contacts
    PS->>F: Broadcast Online Status
    
    loop Heartbeat
        U->>WS: Ping
        WS->>PS: Update Last Seen
        PS->>C: Refresh TTL
    end
    
    Note over U: User disconnects
    WS->>PS: User Offline
    PS->>C: Update Status
    PS->>F: Broadcast Offline Status
```

## 🔐 End-to-End Encryption

### Signal Protocol Implementation
WhatsApp uses the Signal Protocol for E2E encryption.

```mermaid
graph TD
    A[Key Exchange] --> B[Identity Keys<br/>Long-term]
    A --> C[Signed Pre-keys<br/>Medium-term]
    A --> D[One-time Pre-keys<br/>Single use]
    
    E[Session Setup] --> F[Double Ratchet]
    F --> G[Root Key]
    F --> H[Chain Keys]
    
    I[Message Encryption] --> J[Message Key]
    I --> K[AES-256 Encryption]
    
    style F fill:#87CEEB
    style K fill:#98FB98
```

### Key Exchange Process
```mermaid
sequenceDiagram
    participant A as Alice
    participant S as Server
    participant B as Bob
    
    Note over A,B: Initial Setup
    A->>S: Upload Identity Key + Pre-keys
    B->>S: Upload Identity Key + Pre-keys
    
    Note over A,B: Alice wants to message Bob
    A->>S: Request Bob's Pre-key Bundle
    S->>A: Bob's Keys
    
    Note over A: Generate Shared Secret
    Note over A: Derive Encryption Keys
    
    A->>S: Encrypted Message + Alice's Keys
    S->>B: Forward Message Bundle
    
    Note over B: Derive Same Shared Secret
    Note over B: Decrypt Message
    
    B->>A: Encrypted Reply (E2E established)
```

## 📱 Real-time Communication

### WebSocket Connection Management
```mermaid
graph TD
    A[Client Apps] --> B[Connection Load Balancer]
    B --> C[WebSocket Server 1]
    B --> D[WebSocket Server 2]
    B --> E[WebSocket Server N]
    
    C --> F[Connection Pool<br/>10K connections]
    D --> G[Connection Pool<br/>10K connections]
    E --> H[Connection Pool<br/>10K connections]
    
    I[Message Routing] --> J[User-Server Mapping<br/>Redis]
    J --> K[Route to Correct Server]
    
    style B fill:#87CEEB
    style J fill:#98FB98
```

### Message Delivery Guarantees
```mermaid
stateDiagram-v2
    [*] --> Sent
    Sent --> Delivered : Received by server
    Delivered --> Read : Opened by recipient
    
    Sent --> Failed : Network error
    Failed --> Retry : Exponential backoff
    Retry --> Sent : Retry attempt
    Retry --> Cancelled : Max retries exceeded
```

## 🚀 Scalability Solutions

### Database Sharding Strategy

#### Message Sharding by Chat ID
```mermaid
graph TD
    A[Message Request] --> B[Shard Router]
    B --> C{Hash chat_id}
    
    C --> D[Shard 1<br/>chat_id % 4 = 0]
    C --> E[Shard 2<br/>chat_id % 4 = 1]
    C --> F[Shard 3<br/>chat_id % 4 = 2]
    C --> G[Shard 4<br/>chat_id % 4 = 3]
    
    style B fill:#87CEEB
    style D fill:#FFE4B5
    style E fill:#FFE4B5
    style F fill:#FFE4B5
    style G fill:#FFE4B5
```

#### Time-based Partitioning
```python
def get_message_partition(timestamp):
    # Partition by month
    year_month = timestamp.strftime("%Y-%m")
    return f"messages_{year_month}"

# Examples:
# 2024-01 messages -> messages_2024_01
# 2024-02 messages -> messages_2024_02
```

### Caching Strategy
```mermaid
graph TD
    A[Message Request] --> B[L1: Local Cache<br/>Recent messages]
    B -->|Miss| C[L2: Redis Cache<br/>Frequently accessed]
    C -->|Miss| D[L3: Database<br/>Persistent storage]
    
    E[Cache Warming] --> F[Preload Recent Chats]
    E --> G[Preload Active Groups]
    
    style B fill:#ccffcc
    style C fill:#98FB98
    style D fill:#FFE4B5
```

## 📊 Media Handling

### Media Upload Flow
```mermaid
sequenceDiagram
    participant U as User
    participant API as API Gateway
    participant MS as Media Service
    participant S3 as Object Storage
    participant CDN as CDN
    participant R as Recipient
    
    U->>API: Upload Media Request
    API->>MS: Generate Upload URL
    MS->>S3: Get Presigned URL
    S3->>MS: Return Upload URL
    MS->>API: Upload URL + media_id
    API->>U: Upload URL
    
    U->>S3: Upload Media Directly
    S3->>MS: Upload Complete Webhook
    
    Note over MS: Generate thumbnails
    Note over MS: Compress media
    
    MS->>CDN: Distribute to Edge Servers
    MS->>API: Media Ready
    
    Note over U: Send message with media_id
    U->>R: Message with media reference
    R->>CDN: Download Media
```

### Media Storage Optimization
```mermaid
graph TD
    A[Original Media] --> B[Processing Pipeline]
    B --> C[Multiple Formats]
    
    C --> D[Thumbnail: 150x150]
    C --> E[Preview: 640x640]
    C --> F[High Quality: Original]
    
    D --> G[Hot Storage<br/>Frequently accessed]
    E --> H[Warm Storage<br/>Recent content]
    F --> I[Cold Storage<br/>Archive]
    
    style G fill:#FF6B6B
    style H fill:#4ECDC4
    style I fill:#45B7D1
```

## 🔔 Push Notifications

### Notification Architecture
```mermaid
graph TD
    A[Message Event] --> B[Notification Service]
    B --> C{User Online?}
    
    C -->|Yes| D[Real-time WebSocket]
    C -->|No| E[Push Notification]
    
    E --> F[FCM for Android]
    E --> G[APNS for iOS]
    
    H[Notification Preferences] --> I[User Settings]
    I --> J[Mute Groups]
    I --> K[Do Not Disturb]
    I --> L[Custom Ringtones]
    
    style B fill:#87CEEB
    style F fill:#34A853
    style G fill:#000000
```

### Smart Notification Batching
```python
class NotificationBatcher:
    def __init__(self):
        self.batch_window = 30  # seconds
        self.pending_notifications = {}
    
    def add_notification(self, user_id, message):
        if user_id not in self.pending_notifications:
            self.pending_notifications[user_id] = []
            # Schedule batch send
            schedule_task(self.send_batch, user_id, delay=self.batch_window)
        
        self.pending_notifications[user_id].append(message)
    
    def send_batch(self, user_id):
        messages = self.pending_notifications.pop(user_id, [])
        if messages:
            count = len(messages)
            if count == 1:
                text = f"New message from {messages[0].sender}"
            else:
                text = f"{count} new messages"
            
            send_push_notification(user_id, text)
```

## 📈 Performance Optimizations

### Message Compression
```python
import gzip
import json

def compress_message(message_data):
    # Convert to JSON and compress
    json_data = json.dumps(message_data).encode('utf-8')
    compressed = gzip.compress(json_data)
    
    # Only use compression if it reduces size significantly
    if len(compressed) < len(json_data) * 0.8:
        return compressed, True
    return json_data, False

def decompress_message(data, is_compressed):
    if is_compressed:
        return json.loads(gzip.decompress(data).decode('utf-8'))
    return json.loads(data.decode('utf-8'))
```

### Connection Pooling
```mermaid
graph TD
    A[10M Users] --> B[Connection Distribution]
    B --> C[Server 1: 100K connections]
    B --> D[Server 2: 100K connections]
    B --> E[Server N: 100K connections]
    
    F[Connection Optimization] --> G[HTTP/2 for API calls]
    F --> H[WebSocket for real-time]
    F --> I[Connection reuse]
    
    style C fill:#98FB98
    style D fill:#98FB98
    style E fill:#98FB98
```

## 🌍 Global Architecture

### Multi-Region Deployment
```mermaid
graph TB
    A[Global Load Balancer] --> B[North America]
    A --> C[Europe]
    A --> D[Asia Pacific]
    A --> E[South America]
    
    B --> F[Message Servers]
    B --> G[Media CDN]
    B --> H[Regional Database]
    
    C --> I[Message Servers]
    C --> J[Media CDN]
    C --> K[Regional Database]
    
    H <--> L[Cross-Region Sync]
    K <--> L
    
    style A fill:#87CEEB
    style L fill:#DDA0DD
```

## 🎯 API Design

### Core Endpoints
```http
# Authentication
POST /api/v1/auth/verify-phone
POST /api/v1/auth/verify-code

# Messaging
POST /api/v1/messages
GET /api/v1/messages/{chat_id}
PUT /api/v1/messages/{message_id}/status

# Groups
POST /api/v1/groups
POST /api/v1/groups/{group_id}/members
DELETE /api/v1/groups/{group_id}/members/{user_id}

# Media
POST /api/v1/media/upload
GET /api/v1/media/{media_id}

# Presence
PUT /api/v1/presence/online
PUT /api/v1/presence/offline
```

### WebSocket Events
```json
{
  "type": "message",
  "data": {
    "message_id": "uuid",
    "chat_id": "chat123",
    "sender_id": "user456",
    "content": "encrypted_content",
    "timestamp": "2024-01-15T10:30:00Z",
    "message_type": "text"
  }
}

{
  "type": "typing",
  "data": {
    "chat_id": "chat123",
    "user_id": "user456",
    "is_typing": true
  }
}

{
  "type": "presence",
  "data": {
    "user_id": "user456",
    "status": "online",
    "last_seen": "2024-01-15T10:30:00Z"
  }
}
```

## ⚖️ Trade-offs & Design Decisions

### Trade-offs Made

| Decision | Benefit | Cost |
|----------|---------|------|
| **End-to-End Encryption** | Strong privacy | Cannot search message content |
| **Real-time Delivery** | Great UX | High infrastructure cost |
| **Phone Number as ID** | Easy discovery | Privacy concerns |
| **Media Compression** | Reduced bandwidth | Processing overhead |

### Bottlenecks & Solutions

1. **WebSocket Connections**: Use connection pools and horizontal scaling
2. **Message Delivery**: Implement efficient routing and caching
3. **Media Storage**: Use CDN and tiered storage
4. **Database Writes**: Shard by chat_id and use write-through caching

## 🔧 Technology Stack

- **Backend**: Erlang/OTP (high concurrency)
- **Database**: Cassandra (messages), PostgreSQL (users)
- **Cache**: Redis (presence, recent messages)
- **Message Queue**: Apache Kafka
- **Storage**: Amazon S3, CloudFront CDN
- **Push Notifications**: FCM, APNS
- **Monitoring**: Custom tools, Prometheus

---

**Key Insights**: WhatsApp's architecture prioritizes real-time communication, privacy through encryption, and efficient resource utilization. The use of Erlang enables handling millions of concurrent connections with minimal resources.
