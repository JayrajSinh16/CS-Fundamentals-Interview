# Load Balancers

Load balancers distribute incoming requests across multiple servers to ensure optimal resource utilization, minimize response time, and avoid server overload.

## 🎯 What is Load Balancing?

```mermaid
graph TD
    A[Clients] --> B[Load Balancer]
    B --> C[Server 1]
    B --> D[Server 2]
    B --> E[Server 3]
    B --> F[Server N]
    
    C --> G[(Database)]
    D --> G
    E --> G
    F --> G
    
    style B fill:#87CEEB
    style C fill:#98FB98
    style D fill:#98FB98
    style E fill:#98FB98
    style F fill:#98FB98
```

Load balancing solves several problems:
- **Single Point of Failure**: Eliminates server SPOF
- **Scalability**: Enables horizontal scaling
- **Performance**: Optimizes resource utilization
- **Availability**: Provides redundancy and failover

## 🔄 Load Balancing Algorithms

### 1. Round Robin
Distributes requests sequentially across servers.

```mermaid
sequenceDiagram
    participant C as Client
    participant LB as Load Balancer
    participant S1 as Server 1
    participant S2 as Server 2
    participant S3 as Server 3
    
    C->>LB: Request 1
    LB->>S1: Forward
    
    C->>LB: Request 2
    LB->>S2: Forward
    
    C->>LB: Request 3
    LB->>S3: Forward
    
    C->>LB: Request 4
    LB->>S1: Forward (cycle repeats)
```

**Pros:** Simple, fair distribution
**Cons:** Doesn't consider server capacity or current load

### 2. Weighted Round Robin
Assigns weights based on server capacity.

```mermaid
graph LR
    A[Load Balancer] --> B[Server 1<br/>Weight: 3]
    A --> C[Server 2<br/>Weight: 2] 
    A --> D[Server 3<br/>Weight: 1]
    
    style B fill:#98FB98
    style C fill:#87CEEB
    style D fill:#FFE4B5
```

**Distribution Pattern:** 
- Server 1: 50% of requests (3/6)
- Server 2: 33% of requests (2/6)  
- Server 3: 17% of requests (1/6)

### 3. Least Connections
Routes to server with fewest active connections.

```mermaid
graph TD
    A[New Request] --> B{Check Connections}
    B --> C[Server 1: 5 connections]
    B --> D[Server 2: 3 connections] 
    B --> E[Server 3: 7 connections]
    
    B --> F[Route to Server 2<br/>Least connections]
    
    style D fill:#ccffcc
    style F fill:#ccffcc
```

**Best for:** Long-lived connections, varying request processing times

### 4. Weighted Least Connections
Combines least connections with server weights.

```
Score = Active Connections / Weight

Server 1: 10 connections / 3 weight = 3.33
Server 2: 8 connections / 2 weight = 4.0
Server 3: 3 connections / 1 weight = 3.0

→ Route to Server 3 (lowest score)
```

### 5. IP Hash
Routes based on client IP hash.

```mermaid
graph LR
    A[Client IP: 192.168.1.100] --> B[Hash Function]
    B --> C[Hash: 123456]
    C --> D[123456 % 3 = 0]
    D --> E[Route to Server 1]
    
    style B fill:#DDA0DD
    style E fill:#98FB98
```

**Pros:** Session affinity without sticky sessions
**Cons:** Uneven distribution if client IPs cluster

### 6. Least Response Time
Routes to server with fastest response time.

```mermaid
sequenceDiagram
    participant LB as Load Balancer
    participant S1 as Server 1
    participant S2 as Server 2
    participant S3 as Server 3
    
    Note over LB,S3: Health Check Responses
    LB->>S1: Health Check
    S1->>LB: 50ms
    
    LB->>S2: Health Check  
    S2->>LB: 30ms
    
    LB->>S3: Health Check
    S3->>LB: 80ms
    
    Note over LB: Route next request to Server 2
```

## 📍 Load Balancer Types by Layer

### Layer 4 (Transport Layer)
Routes based on IP and port information only.

```mermaid
graph TD
    A[Client Request<br/>TCP/UDP] --> B[L4 Load Balancer]
    B --> C[Check IP:Port]
    C --> D[Route Decision]
    D --> E[Backend Server]
    
    style B fill:#ffcccc
```

**Characteristics:**
- Fast (no packet inspection)
- Protocol agnostic
- Cannot make content-based decisions
- Lower latency

**Example Configuration (HAProxy):**
```
frontend main
    bind *:80
    mode tcp
    default_backend webservers

backend webservers
    mode tcp
    balance roundrobin
    server web1 10.0.1.1:80 check
    server web2 10.0.1.2:80 check
```

### Layer 7 (Application Layer)
Routes based on application data (HTTP headers, URLs, etc.).

```mermaid
graph TD
    A[HTTP Request] --> B[L7 Load Balancer]
    B --> C[Parse HTTP Headers]
    C --> D[Check URL Path]
    D --> E{Route Decision}
    
    E -->|/api/*| F[API Servers]
    E -->|/static/*| G[Static Servers]
    E -->|/admin/*| H[Admin Servers]
    
    style B fill:#ccffcc
```

**Characteristics:**
- Content-aware routing
- SSL termination
- Request modification
- Higher latency (packet inspection)

**Example Configuration (Nginx):**
```nginx
upstream api_servers {
    server 10.0.1.1:8080;
    server 10.0.1.2:8080;
}

upstream static_servers {
    server 10.0.2.1:80;
    server 10.0.2.2:80;
}

server {
    listen 80;
    
    location /api/ {
        proxy_pass http://api_servers;
    }
    
    location /static/ {
        proxy_pass http://static_servers;
    }
}
```

## 🏗️ Load Balancer Architectures

### Single Load Balancer
```mermaid
graph TD
    A[Internet] --> B[Load Balancer]
    B --> C[Web Server 1]
    B --> D[Web Server 2]
    B --> E[Web Server 3]
    
    style B fill:#ffcccc
```

**Pros:** Simple, cost-effective
**Cons:** Single point of failure

### High Availability Load Balancers
```mermaid
graph TD
    A[Internet] --> B[Primary LB]
    A -.-> C[Secondary LB<br/>Standby]
    
    B --> D[Web Server 1]
    B --> E[Web Server 2]
    B --> F[Web Server 3]
    
    B <-.-> C
    
    style B fill:#ccffcc
    style C fill:#ffffcc
```

**Implementation:** VRRP, heartbeat monitoring, automatic failover

### Multi-Tier Load Balancing
```mermaid
graph TD
    A[Internet] --> B[Global Load Balancer<br/>DNS-based]
    B --> C[Regional LB<br/>US East]
    B --> D[Regional LB<br/>US West]
    B --> E[Regional LB<br/>Europe]
    
    C --> F[Local LB 1]
    C --> G[Local LB 2]
    
    F --> H[Web Servers]
    G --> I[Web Servers]
    
    style B fill:#87CEEB
    style C fill:#98FB98
    style D fill:#98FB98
    style E fill:#98FB98
```

## 🔍 Health Checks

### Active Health Checks
Load balancer proactively checks server health.

```mermaid
sequenceDiagram
    participant LB as Load Balancer
    participant S1 as Server 1
    participant S2 as Server 2
    
    loop Every 5 seconds
        LB->>S1: GET /health
        S1->>LB: 200 OK
        
        LB->>S2: GET /health
        S2--xLB: Timeout/Error
    end
    
    Note over LB: Mark S2 as unhealthy
    Note over LB: Stop routing to S2
```

**Configuration Example:**
```yaml
health_check:
  interval: 5s
  timeout: 3s
  retries: 3
  path: /health
  expected_codes: [200, 201]
```

### Passive Health Checks
Monitor actual request failures to determine health.

```mermaid
graph TD
    A[Monitor Request Results] --> B{Failure Rate > Threshold?}
    B -->|Yes| C[Mark Server Unhealthy]
    B -->|No| D[Keep Server Active]
    
    C --> E[Remove from Pool]
    E --> F[Retry After Interval]
    
    style C fill:#ffcccc
    style D fill:#ccffcc
```

## ⚙️ Advanced Features

### SSL Termination
```mermaid
graph LR
    A[Client<br/>HTTPS] --> B[Load Balancer<br/>SSL Termination]
    B --> C[Backend<br/>HTTP]
    B --> D[Backend<br/>HTTP]
    
    A -.-> E[Encrypted<br/>Connection]
    B -.-> F[Unencrypted<br/>Connection]
    
    style B fill:#87CEEB
    style E fill:#ffcccc
    style F fill:#ccffcc
```

**Benefits:**
- Reduced backend server load
- Centralized certificate management
- Better performance

### Session Persistence (Sticky Sessions)
```mermaid
sequenceDiagram
    participant C as Client
    participant LB as Load Balancer
    participant S1 as Server 1
    participant S2 as Server 2
    
    C->>LB: First Request
    LB->>S1: Route to S1
    S1->>LB: Response + Session Cookie
    LB->>C: Response + Sticky Cookie
    
    C->>LB: Subsequent Request + Cookie
    Note over LB: Cookie indicates S1
    LB->>S1: Route to same server
```

**Implementation Methods:**
- Cookie-based
- IP-based
- URL parameter-based

### Rate Limiting
```mermaid
graph TD
    A[Incoming Request] --> B[Check Rate Limit]
    B --> C{Within Limit?}
    C -->|Yes| D[Forward Request]
    C -->|No| E[Return 429<br/>Too Many Requests]
    
    style D fill:#ccffcc
    style E fill:#ffcccc
```

**Rate Limiting Algorithms:**
- Token Bucket
- Leaky Bucket
- Fixed Window
- Sliding Window

## 🌐 Cloud Load Balancers

### AWS Application Load Balancer (ALB)
```mermaid
graph TD
    A[Internet Gateway] --> B[ALB]
    B --> C[Target Group 1<br/>EC2 Instances]
    B --> D[Target Group 2<br/>Lambda Functions]
    B --> E[Target Group 3<br/>IP Addresses]
    
    B --> F[Listener Rules]
    F --> G[Path-based Routing]
    F --> H[Host-based Routing]
    F --> I[Header-based Routing]
    
    style B fill:#FF9900
```

### Google Cloud Load Balancer
```mermaid
graph TD
    A[Global Load Balancer] --> B[Regional Backend 1]
    A --> C[Regional Backend 2]
    A --> D[Regional Backend 3]
    
    B --> E[Instance Group]
    C --> F[Instance Group]
    D --> G[Instance Group]
    
    style A fill:#4285F4
```

## 📊 Performance Metrics

### Key Metrics to Monitor

| Metric | Description | Target |
|--------|-------------|---------|
| **Latency** | Request processing time | < 100ms |
| **Throughput** | Requests per second | Business dependent |
| **Error Rate** | Failed requests % | < 0.1% |
| **Connection Count** | Active connections | Monitor trends |
| **Backend Health** | Healthy servers % | > 90% |

### Capacity Planning
```
Current: 10,000 RPS
Peak multiplier: 3x
Growth rate: 50% yearly

Year 1 peak: 30,000 RPS
Year 2 peak: 45,000 RPS
Year 3 peak: 67,500 RPS

With safety buffer (20%): 81,000 RPS
```

## 🛠️ Implementation Examples

### HAProxy Configuration
```
global
    daemon
    maxconn 4096

defaults
    mode http
    timeout connect 5000ms
    timeout client 50000ms
    timeout server 50000ms

frontend web_frontend
    bind *:80
    default_backend web_servers

backend web_servers
    balance roundrobin
    option httpchk GET /health
    server web1 10.0.1.1:8080 check
    server web2 10.0.1.2:8080 check
    server web3 10.0.1.3:8080 check
```

### Nginx Load Balancing
```nginx
upstream backend {
    least_conn;
    server 10.0.1.1:8080 weight=3;
    server 10.0.1.2:8080 weight=2;
    server 10.0.1.3:8080 weight=1;
    server 10.0.1.4:8080 backup;
}

server {
    listen 80;
    location / {
        proxy_pass http://backend;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

## 🎯 Best Practices

### 1. Algorithm Selection
- **Round Robin**: Equal servers, similar request patterns
- **Least Connections**: Varying connection durations
- **IP Hash**: Session affinity requirements
- **Weighted**: Mixed server capacities

### 2. Health Check Strategy
- Use application-specific health endpoints
- Check critical dependencies (database, cache)
- Implement circuit breaker patterns
- Monitor health check performance

### 3. Monitoring & Alerting
- Track response times and error rates
- Monitor backend server health
- Set up alerts for load balancer failures
- Use distributed tracing for request flow

### 4. Security Considerations
- Rate limiting and DDoS protection
- SSL/TLS termination best practices
- Access control and IP whitelisting
- Regular security updates

## ⚖️ Trade-offs

| Aspect | Layer 4 | Layer 7 |
|--------|---------|---------|
| **Performance** | Higher | Lower |
| **Features** | Basic | Advanced |
| **Complexity** | Simple | Complex |
| **Cost** | Lower | Higher |
| **Flexibility** | Limited | High |

---

**Key Takeaway**: Load balancers are critical for building scalable, highly available systems. Choose the right algorithm and type based on your specific requirements and traffic patterns.

---
[← Back to Main Guide](./README.md) | [← Previous: Consistency Patterns](./consistency-patterns.md) | [Next: Reverse Proxy →](./reverse-proxy.md)
