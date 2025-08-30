# Instagram System Design

Design a photo-sharing platform like Instagram that supports billions of users and handles millions of photo uploads daily.

## 📋 Problem Statement

Design Instagram that allows users to:
- Upload and share photos/videos
- Follow other users and view their content
- Like, comment, and share posts
- Generate personalized feeds
- Search for users and content

## 🎯 Requirements

### Functional Requirements
- **Photo/Video Upload**: Users can upload media content
- **Timeline Generation**: Users see posts from people they follow
- **Social Features**: Follow/unfollow, like, comment
- **Search**: Find users, hashtags, and content
- **Stories**: Temporary 24-hour content
- **Direct Messaging**: Private conversations

### Non-Functional Requirements
- **Scale**: 2 billion users, 500M daily active users
- **Read Heavy**: 100:1 read to write ratio
- **Storage**: 95TB of new photos daily
- **Availability**: 99.9% uptime
- **Latency**: Feed loads in <200ms
- **Global**: Worldwide content delivery

## 📊 Capacity Estimation

### User Statistics
```
Total Users: 2B
Daily Active Users: 500M
Photos uploaded per day: 95M
Average photo size: 1MB
Video uploads: 5M per day
Average video size: 10MB
```

### Storage Requirements
```
Photo storage per day: 95M × 1MB = 95TB
Video storage per day: 5M × 10MB = 50TB
Total daily storage: 145TB
Annual storage: 145TB × 365 = 53PB

With metadata and multiple sizes:
Total storage needed: 53PB × 3 = 159PB per year
```

### Bandwidth Requirements
```
Read QPS: 50,000 requests/second (peak)
Write QPS: 500 requests/second
Bandwidth for reads: 50,000 × 1MB = 50GB/s
Bandwidth for writes: 500 × 1MB = 500MB/s
```

## 🏗️ High-Level Architecture

```mermaid
graph TB
    A[Mobile/Web Clients] --> B[CDN]
    B --> C[Load Balancer]
    C --> D[API Gateway]
    
    D --> E[User Service]
    D --> F[Media Service] 
    D --> G[Feed Service]
    D --> H[Social Service]
    D --> I[Notification Service]
    
    E --> J[(User DB)]
    F --> K[(Media Metadata DB)]
    F --> L[Object Storage]
    G --> M[(Feed DB)]
    G --> N[Feed Cache]
    H --> O[(Graph DB)]
    I --> P[Message Queue]
    
    Q[Search Service] --> R[(Search Index)]
    
    style B fill:#87CEEB
    style L fill:#FFE4B5
    style N fill:#98FB98
    style P fill:#DDA0DD
```

## 🗄️ Database Design

### User Entity
```mermaid
erDiagram
    User {
        bigint user_id PK
        varchar username UK
        varchar email UK
        varchar full_name
        text bio
        varchar profile_picture_url
        timestamp created_at
        timestamp updated_at
        boolean is_verified
        int followers_count
        int following_count
        int posts_count
    }
```

### Post Entity
```mermaid
erDiagram
    Post {
        bigint post_id PK
        bigint user_id FK
        text caption
        varchar media_url
        varchar media_type
        int width
        int height
        timestamp created_at
        int likes_count
        int comments_count
        boolean is_deleted
        varchar location
    }
```

### Complete ER Diagram
```mermaid
erDiagram
    User ||--o{ Post : "creates"
    User ||--o{ Comment : "writes"
    User ||--o{ Like : "gives"
    User ||--o{ Follow : "follows"
    User ||--o{ Follow : "followed_by"
    Post ||--o{ Comment : "has"
    Post ||--o{ Like : "receives"
    Post ||--o{ PostTag : "tagged_with"
    Tag ||--o{ PostTag : "used_in"
    User ||--o{ Story : "creates"
    
    User {
        bigint user_id PK
        varchar username UK
        varchar email UK
        varchar full_name
        text bio
        varchar profile_picture_url
        timestamp created_at
        boolean is_verified
        int followers_count
        int following_count
        int posts_count
    }
    
    Post {
        bigint post_id PK
        bigint user_id FK
        text caption
        varchar media_url
        varchar media_type
        int width
        int height
        timestamp created_at
        int likes_count
        int comments_count
        varchar location
    }
    
    Comment {
        bigint comment_id PK
        bigint post_id FK
        bigint user_id FK
        text content
        timestamp created_at
        bigint parent_comment_id FK
    }
    
    Like {
        bigint like_id PK
        bigint post_id FK
        bigint user_id FK
        timestamp created_at
    }
    
    Follow {
        bigint follow_id PK
        bigint follower_id FK
        bigint following_id FK
        timestamp created_at
    }
    
    Tag {
        bigint tag_id PK
        varchar tag_name UK
        int usage_count
    }
    
    PostTag {
        bigint post_id FK
        bigint tag_id FK
        timestamp created_at
    }
    
    Story {
        bigint story_id PK
        bigint user_id FK
        varchar media_url
        varchar media_type
        timestamp created_at
        timestamp expires_at
        int views_count
    }
```

## 🔄 Core Workflows

### Photo Upload Flow
```mermaid
sequenceDiagram
    participant U as User
    participant A as App
    participant API as API Gateway
    participant MS as Media Service
    participant S3 as Object Storage
    participant DB as Database
    participant CDN as CDN
    
    U->>A: Select & Upload Photo
    A->>API: POST /upload with photo
    API->>MS: Process upload request
    
    Note over MS: Generate unique filename
    Note over MS: Resize to multiple formats
    
    MS->>S3: Store original + variants
    S3->>MS: Return URLs
    
    MS->>DB: Save metadata
    MS->>CDN: Preload popular formats
    
    MS->>API: Return post_id + URLs
    API->>A: Upload complete
    A->>U: Show success + preview
```

### Feed Generation Flow
```mermaid
sequenceDiagram
    participant U as User
    participant A as App
    participant API as API Gateway
    participant FS as Feed Service
    participant C as Cache
    participant DB as Database
    participant ML as ML Service
    
    U->>A: Open Instagram
    A->>API: GET /feed
    API->>FS: Get user feed
    
    FS->>C: Check cached feed
    
    alt Cache Hit
        C->>FS: Return cached posts
    else Cache Miss
        FS->>DB: Get following list
        FS->>DB: Get recent posts
        FS->>ML: Rank posts by relevance
        ML->>FS: Return ranked posts
        FS->>C: Cache ranked feed
    end
    
    FS->>API: Return feed data
    API->>A: Feed response
    A->>U: Display feed
```

### Social Interaction Flow
```mermaid
sequenceDiagram
    participant U as User
    participant A as App
    participant API as API Gateway
    participant SS as Social Service
    participant DB as Database
    participant NS as Notification Service
    participant Q as Message Queue
    
    U->>A: Like a post
    A->>API: POST /posts/{id}/like
    API->>SS: Process like
    
    par Update Database
        SS->>DB: Insert like record
        SS->>DB: Increment likes_count
    and Send Notification
        SS->>Q: Queue notification
        Q->>NS: Process notification
        NS->>A: Push notification to post owner
    end
    
    SS->>API: Success response
    API->>A: Update UI
    A->>U: Show liked state
```

## 🧠 Feed Generation Algorithm

### Timeline Generation Strategies

#### 1. Pull Model (Timeline on Read)
```mermaid
sequenceDiagram
    participant U as User
    participant FS as Feed Service
    participant DB as Database
    
    U->>FS: Request feed
    FS->>DB: Get user's following list
    FS->>DB: Get recent posts from each followed user
    Note over FS: Merge and sort posts by timestamp
    FS->>U: Return ranked feed
```

**Pros:** Consistent, real-time updates
**Cons:** Slow for users following many people

#### 2. Push Model (Timeline on Write)
```mermaid
sequenceDiagram
    participant PO as Post Owner
    participant FS as Feed Service
    participant DB as Database
    participant F1 as Follower 1
    participant F2 as Follower 2
    participant FN as Follower N
    
    PO->>FS: Create new post
    FS->>DB: Get all followers
    
    par Update All Feeds
        FS->>F1: Add to timeline
        FS->>F2: Add to timeline
        FS->>FN: Add to timeline
    end
```

**Pros:** Fast reads
**Cons:** Expensive writes for celebrities, storage overhead

#### 3. Hybrid Model (Instagram's Approach)
```mermaid
graph TD
    A[New Post Created] --> B{User Type?}
    B -->|Regular User| C[Push to Followers]
    B -->|Celebrity| D[Store in Celebrity Feed]
    
    E[User Requests Feed] --> F[Pull Own Timeline]
    F --> G[Pull Celebrity Posts]
    G --> H[Merge & Rank]
    H --> I[Return Feed]
    
    style C fill:#ccffcc
    style D fill:#ffffcc
    style H fill:#87CEEB
```

### Ranking Algorithm
```python
# Simplified Instagram ranking algorithm
def calculate_post_score(post, user):
    # Recency score (exponential decay)
    time_diff = current_time - post.created_at
    recency_score = exp(-time_diff / TIME_DECAY)
    
    # Relationship score
    if post.user_id in user.close_friends:
        relationship_score = 1.0
    elif post.user_id in user.following:
        relationship_score = 0.8
    else:
        relationship_score = 0.1
    
    # Engagement score
    engagement_rate = (post.likes + post.comments) / post.user.followers_count
    engagement_score = min(engagement_rate * 10, 1.0)
    
    # Content type preference
    if post.media_type == user.preferred_content_type:
        content_score = 1.0
    else:
        content_score = 0.8
    
    # Final score
    final_score = (
        recency_score * 0.3 +
        relationship_score * 0.4 +
        engagement_score * 0.2 +
        content_score * 0.1
    )
    
    return final_score
```

## 🗂️ Data Storage Strategy

### Object Storage Architecture
```mermaid
graph TD
    A[Upload Service] --> B{File Type}
    B -->|Image| C[Image Processing]
    B -->|Video| D[Video Processing]
    
    C --> E[Generate Multiple Sizes]
    D --> F[Generate Multiple Qualities]
    
    E --> G[Original: 1080x1080]
    E --> H[Large: 640x640]
    E --> I[Medium: 320x320]
    E --> J[Thumbnail: 150x150]
    
    F --> K[4K Quality]
    F --> L[HD Quality]
    F --> M[SD Quality]
    
    G --> N[S3 - Hot Storage]
    H --> N
    I --> N
    J --> N
    K --> O[S3 - Warm Storage]
    L --> O
    M --> O
    
    style N fill:#FF6B6B
    style O fill:#4ECDC4
```

### Database Sharding Strategy

#### User Sharding
```mermaid
graph TD
    A[User Request] --> B[Shard Router]
    B --> C{Hash user_id}
    C --> D[Shard 1<br/>user_id % 4 = 0]
    C --> E[Shard 2<br/>user_id % 4 = 1]
    C --> F[Shard 3<br/>user_id % 4 = 2]
    C --> G[Shard 4<br/>user_id % 4 = 3]
    
    style B fill:#87CEEB
    style D fill:#FFE4B5
    style E fill:#FFE4B5
    style F fill:#FFE4B5
    style G fill:#FFE4B5
```

#### Post Sharding (by creation time)
```mermaid
graph TD
    A[Post Creation] --> B{Creation Date}
    B --> C[2024 Q1<br/>Shard A]
    B --> D[2024 Q2<br/>Shard B]
    B --> E[2024 Q3<br/>Shard C]
    B --> F[2024 Q4<br/>Shard D]
    
    style C fill:#FFE4B5
    style D fill:#FFE4B5
    style E fill:#FFE4B5
    style F fill:#FFE4B5
```

## 🚀 Caching Strategy

### Multi-Level Caching
```mermaid
graph TD
    A[User Request] --> B[CDN Cache]
    B -->|Miss| C[Application Cache]
    C -->|Miss| D[Database Cache]
    D -->|Miss| E[Database]
    
    B -->|Hit| F[Return Cached Content]
    C -->|Hit| F
    D -->|Hit| F
    
    style B fill:#87CEEB
    style C fill:#98FB98
    style D fill:#FFE4B5
```

### Cache Types and TTL
| Cache Type | Content | TTL | Invalidation |
|------------|---------|-----|--------------|
| **CDN** | Images, Videos | 30 days | Version-based |
| **Feed Cache** | User timelines | 5 minutes | Event-driven |
| **User Cache** | Profile data | 1 hour | Update-based |
| **Metadata Cache** | Post details | 30 minutes | Event-driven |

## 📱 API Design

### Core Endpoints

#### User Management
```http
POST /api/v1/users/register
POST /api/v1/users/login
GET /api/v1/users/{user_id}
PUT /api/v1/users/{user_id}
GET /api/v1/users/{user_id}/posts
```

#### Post Management
```http
POST /api/v1/posts
GET /api/v1/posts/{post_id}
DELETE /api/v1/posts/{post_id}
GET /api/v1/feed
GET /api/v1/explore
```

#### Social Features
```http
POST /api/v1/users/{user_id}/follow
DELETE /api/v1/users/{user_id}/follow
POST /api/v1/posts/{post_id}/like
POST /api/v1/posts/{post_id}/comments
```

### Sample API Response
```json
{
  "post_id": "123456789",
  "user": {
    "user_id": "987654321",
    "username": "johndoe",
    "profile_picture": "https://cdn.instagram.com/profiles/johndoe.jpg"
  },
  "media": {
    "type": "image",
    "url": "https://cdn.instagram.com/posts/123456789.jpg",
    "width": 1080,
    "height": 1080,
    "thumbnails": {
      "small": "https://cdn.instagram.com/posts/123456789_150.jpg",
      "medium": "https://cdn.instagram.com/posts/123456789_320.jpg"
    }
  },
  "caption": "Beautiful sunset! #photography #nature",
  "likes_count": 1547,
  "comments_count": 89,
  "created_at": "2024-01-15T18:30:00Z",
  "location": "Malibu, CA"
}
```

## 🔧 Technology Stack

### Backend Services
- **API Gateway**: Kong/AWS API Gateway
- **Application**: Java/Python with Spring Boot/Django
- **Message Queue**: Apache Kafka
- **Cache**: Redis Cluster
- **Search**: Elasticsearch
- **ML Platform**: TensorFlow/PyTorch

### Databases
- **User Data**: PostgreSQL (sharded)
- **Posts Metadata**: Cassandra
- **Social Graph**: Neo4j/Amazon Neptune
- **Analytics**: Apache Spark + HDFS

### Infrastructure
- **Object Storage**: Amazon S3/Google Cloud Storage
- **CDN**: Cloudflare/AWS CloudFront
- **Container Orchestration**: Kubernetes
- **Monitoring**: Prometheus + Grafana

## 📊 Performance Optimizations

### Image Processing Pipeline
```mermaid
graph LR
    A[Original Upload] --> B[Async Processing Queue]
    B --> C[Resize Service]
    C --> D[Multiple Formats]
    D --> E[WebP, JPEG, PNG]
    E --> F[Upload to CDN]
    F --> G[Cache Warming]
    
    style B fill:#DDA0DD
    style F fill:#87CEEB
```

### Feed Precomputation
```python
# Pseudo-code for feed precomputation
class FeedPrecomputation:
    def precompute_feeds(self):
        # Run every 5 minutes
        active_users = get_active_users_last_hour()
        
        for user in active_users:
            feed = self.generate_feed(user)
            cache.set(f"feed:{user.id}", feed, ttl=300)
    
    def generate_feed(self, user):
        # Get posts from following
        following = get_user_following(user.id)
        posts = get_recent_posts(following, limit=1000)
        
        # Apply ML ranking
        ranked_posts = ml_service.rank_posts(posts, user)
        
        return ranked_posts[:50]  # Top 50 posts
```

## 🔒 Security Considerations

### Content Moderation
```mermaid
graph TD
    A[User Uploads Content] --> B[Automatic Scan]
    B --> C{Safe Content?}
    C -->|Yes| D[Publish Immediately]
    C -->|Suspicious| E[Queue for Review]
    C -->|Unsafe| F[Block Content]
    
    E --> G[Human Moderator]
    G --> H{Approve?}
    H -->|Yes| D
    H -->|No| F
    
    style D fill:#ccffcc
    style F fill:#ffcccc
```

### Authentication & Authorization
- **JWT tokens** for stateless authentication
- **OAuth 2.0** for third-party integrations
- **Rate limiting** to prevent abuse
- **HTTPS everywhere** for secure communication

## ⚖️ Trade-offs & Bottlenecks

### Trade-offs Made

| Decision | Benefit | Cost |
|----------|---------|------|
| **Hybrid Feed Model** | Balanced performance | Complex implementation |
| **NoSQL for Posts** | High scalability | Eventual consistency |
| **CDN for Media** | Fast global access | Storage costs |
| **Microservices** | Independent scaling | Operational complexity |

### Potential Bottlenecks

1. **Feed Generation**: For users following celebrities
2. **Image Processing**: High-resolution uploads
3. **Database Writes**: During viral content
4. **Search Performance**: Complex queries at scale

### Scaling Solutions

1. **Horizontal Scaling**: Add more servers
2. **Caching**: Multiple cache layers
3. **Async Processing**: Queue heavy operations
4. **Read Replicas**: Distribute read load

---

**Key Insights**: Instagram's success comes from optimizing for read-heavy workloads, efficient image delivery, and smart feed generation algorithms that balance relevance with performance.
