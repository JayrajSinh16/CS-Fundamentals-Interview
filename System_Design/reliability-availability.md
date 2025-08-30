# Reliability & Availability

Reliability and availability are critical aspects of system design that determine how well a system performs under various conditions and how accessible it remains to users.

## 🎯 Definitions

### Reliability
The probability that a system performs correctly during a specific time duration.

```
Reliability = Number of Successful Operations / Total Number of Operations
```

### Availability  
The percentage of time a system is operational and accessible.

```
Availability = Uptime / (Uptime + Downtime) × 100%
```

## 📊 Availability Classifications

### The Nines of Availability

```mermaid
graph TD
    A[Availability Levels] --> B[99% - Two Nines<br/>87.6 hours downtime/year]
    A --> C[99.9% - Three Nines<br/>8.76 hours downtime/year]
    A --> D[99.99% - Four Nines<br/>52.56 minutes downtime/year]
    A --> E[99.999% - Five Nines<br/>5.26 minutes downtime/year]
    A --> F[99.9999% - Six Nines<br/>31.5 seconds downtime/year]
    
    style B fill:#ffcccc
    style C fill:#ffffcc
    style D fill:#ccffcc
    style E fill:#ccffff
    style F fill:#e6ccff
```

### Downtime Impact Table

| Availability | Downtime/Year | Downtime/Month | Downtime/Week | Use Case |
|--------------|---------------|----------------|----------------|----------|
| **90%** | 36.5 days | 72 hours | 16.8 hours | Internal tools |
| **99%** | 3.65 days | 7.2 hours | 1.68 hours | Basic websites |
| **99.9%** | 8.76 hours | 43.2 minutes | 10.1 minutes | E-commerce |
| **99.99%** | 52.56 minutes | 4.32 minutes | 1.01 minutes | Financial services |
| **99.999%** | 5.26 minutes | 25.9 seconds | 6.05 seconds | Mission critical |

## 🏗️ Building Reliable Systems

### 1. Redundancy
Eliminate single points of failure by adding redundant components.

```mermaid
graph TD
    A[User Request] --> B[Load Balancer 1]
    A -.-> C[Load Balancer 2<br/>Standby]
    
    B --> D[Web Server 1]
    B --> E[Web Server 2]
    B --> F[Web Server 3]
    
    D --> G[Database Master]
    E --> G
    F --> G
    
    G --> H[Database Replica 1]
    G --> I[Database Replica 2]
    
    style C fill:#ffffcc
    style H fill:#ccffcc
    style I fill:#ccffcc
```

### 2. Graceful Degradation
System continues to operate with reduced functionality when components fail.

```mermaid
graph TD
    A[Full Service] --> B[Recommendation Engine Down]
    B --> C[Show Default Content]
    
    A --> D[Payment Service Down]
    D --> E[Queue Orders for Later]
    
    A --> F[Search Service Down]  
    F --> G[Show Cached Results]
    
    style A fill:#ccffcc
    style C fill:#ffffcc
    style E fill:#ffffcc
    style G fill:#ffffcc
```

### 3. Circuit Breaker Pattern
Prevent cascade failures by stopping calls to failing services.

```mermaid
stateDiagram-v2
    [*] --> Closed
    Closed --> Open : Failure threshold reached
    Open --> HalfOpen : Timeout elapsed
    HalfOpen --> Closed : Success
    HalfOpen --> Open : Failure
    
    note right of Closed
        Normal operation
        Requests pass through
    end note
    
    note right of Open
        Fail fast
        Return cached data
        or error immediately
    end note
    
    note right of HalfOpen
        Test if service
        is back online
    end note
```

#### Circuit Breaker Implementation
```python
import time
from enum import Enum

class CircuitState(Enum):
    CLOSED = "closed"
    OPEN = "open"
    HALF_OPEN = "half_open"

class CircuitBreaker:
    def __init__(self, failure_threshold=5, recovery_timeout=60, expected_exception=Exception):
        self.failure_threshold = failure_threshold
        self.recovery_timeout = recovery_timeout
        self.expected_exception = expected_exception
        
        self.failure_count = 0
        self.last_failure_time = None
        self.state = CircuitState.CLOSED
    
    def call(self, func, *args, **kwargs):
        if self.state == CircuitState.OPEN:
            if self._should_attempt_reset():
                self.state = CircuitState.HALF_OPEN
            else:
                raise Exception("Circuit breaker is OPEN")
        
        try:
            result = func(*args, **kwargs)
            self._on_success()
            return result
        except self.expected_exception as e:
            self._on_failure()
            raise e
    
    def _should_attempt_reset(self):
        return (time.time() - self.last_failure_time) >= self.recovery_timeout
    
    def _on_success(self):
        self.failure_count = 0
        self.state = CircuitState.CLOSED
    
    def _on_failure(self):
        self.failure_count += 1
        self.last_failure_time = time.time()
        
        if self.failure_count >= self.failure_threshold:
            self.state = CircuitState.OPEN
```

## 🔄 Failure Detection & Recovery

### Health Check Patterns

#### 1. Shallow Health Check
```python
@app.route('/health')
def shallow_health():
    return {"status": "healthy", "timestamp": time.time()}
```

#### 2. Deep Health Check
```python
@app.route('/health/deep')
def deep_health():
    checks = {
        "database": check_database_connection(),
        "cache": check_redis_connection(),
        "external_api": check_external_service(),
        "disk_space": check_disk_space()
    }
    
    all_healthy = all(checks.values())
    status_code = 200 if all_healthy else 503
    
    return {"status": "healthy" if all_healthy else "unhealthy", 
            "checks": checks}, status_code
```

### Monitoring & Alerting Architecture

```mermaid
graph TB
    A[Application Metrics] --> B[Metrics Collection<br/>Prometheus]
    C[Infrastructure Metrics] --> B
    D[Custom Business Metrics] --> B
    
    B --> E[Alerting Rules Engine]
    E --> F[Alert Manager]
    
    F --> G[PagerDuty]
    F --> H[Slack]
    F --> I[Email]
    
    J[Grafana Dashboard] --> B
    K[Log Aggregation<br/>ELK Stack] --> L[Log-based Alerts]
    
    style B fill:#98FB98
    style F fill:#FFE4B5
```

## 📈 Measuring Reliability

### Key Metrics

#### 1. Mean Time Between Failures (MTBF)
```
MTBF = Total Operational Time / Number of Failures

Example:
System runs for 8760 hours (1 year)
Experiences 4 failures
MTBF = 8760 / 4 = 2190 hours
```

#### 2. Mean Time To Recovery (MTTR)
```
MTTR = Total Downtime / Number of Incidents

Example:
4 incidents with downtimes: 30min, 45min, 60min, 25min
Total downtime = 160 minutes
MTTR = 160 / 4 = 40 minutes
```

#### 3. Service Level Indicators (SLIs)
- **Availability**: Percentage of successful requests
- **Latency**: Time to process requests
- **Throughput**: Requests handled per second
- **Error Rate**: Percentage of failed requests

#### 4. Service Level Objectives (SLOs)
```yaml
slos:
  availability:
    target: 99.9%
    measurement_window: 30_days
  
  latency:
    target: 95th_percentile < 200ms
    measurement_window: 24_hours
  
  error_rate:
    target: <0.1%
    measurement_window: 24_hours
```

### Error Budget Concept

```
Error Budget = 1 - SLO

For 99.9% availability SLO:
Error Budget = 1 - 0.999 = 0.1%

Monthly Error Budget = 30 days × 24 hours × 60 minutes × 0.001 = 43.2 minutes
```

```mermaid
graph LR
    A[Error Budget<br/>43.2 minutes/month] --> B{Budget Consumed?}
    B -->|< 50%| C[Normal Development<br/>New Features]
    B -->|50-80%| D[Focus on Reliability<br/>Reduce Risk]
    B -->|> 80%| E[Freeze Features<br/>Fix Issues]
    
    style C fill:#ccffcc
    style D fill:#ffffcc
    style E fill:#ffcccc
```

## 🌍 High Availability Architectures

### Multi-Region Setup

```mermaid
graph TB
    A[Global DNS<br/>Route 53] --> B[US-East Region]
    A --> C[US-West Region]
    A --> D[EU Region]
    
    B --> E[Load Balancer]
    B --> F[Auto Scaling Group]
    B --> G[RDS Multi-AZ]
    
    C --> H[Load Balancer]
    C --> I[Auto Scaling Group]
    C --> J[RDS Multi-AZ]
    
    D --> K[Load Balancer]
    D --> L[Auto Scaling Group]
    D --> M[RDS Multi-AZ]
    
    G <--> N[Cross-Region<br/>Replication]
    J <--> N
    M <--> N
    
    style A fill:#87CEEB
    style N fill:#DDA0DD
```

### Active-Active vs Active-Passive

#### Active-Active Configuration
```mermaid
graph TD
    A[Traffic Splitter] --> B[Region 1<br/>50% Traffic]
    A --> C[Region 2<br/>50% Traffic]
    
    B --> D[Database Cluster 1]
    C --> E[Database Cluster 2]
    
    D <--> F[Bidirectional<br/>Sync]
    E <--> F
    
    style B fill:#ccffcc
    style C fill:#ccffcc
```

**Pros:** Full resource utilization, no wasted capacity
**Cons:** Complex data synchronization, potential conflicts

#### Active-Passive Configuration
```mermaid
graph TD
    A[All Traffic] --> B[Primary Region<br/>Active]
    B -.-> C[Secondary Region<br/>Standby]
    
    B --> D[Primary Database]
    D --> E[Async Replication]
    E --> F[Standby Database]
    
    style B fill:#ccffcc
    style C fill:#ffffcc
    style F fill:#ffffcc
```

**Pros:** Simpler to manage, consistent data
**Cons:** Wasted resources, longer failover time

## 🔧 Fault Tolerance Patterns

### 1. Bulkhead Pattern
Isolate critical resources to prevent total system failure.

```mermaid
graph TD
    A[Shared Thread Pool<br/>Problem] --> B[One slow service<br/>blocks all]
    
    C[Bulkhead Solution] --> D[User Service<br/>Thread Pool]
    C --> E[Payment Service<br/>Thread Pool]
    C --> F[Notification Service<br/>Thread Pool]
    
    style B fill:#ffcccc
    style D fill:#ccffcc
    style E fill:#ccffcc
    style F fill:#ccffcc
```

### 2. Timeout Pattern
```python
import asyncio

async def call_with_timeout(service_call, timeout_seconds=5):
    try:
        result = await asyncio.wait_for(service_call(), timeout_seconds)
        return result
    except asyncio.TimeoutError:
        # Return cached data or default response
        return get_cached_response()
```

### 3. Retry Pattern with Exponential Backoff
```python
import time
import random

def exponential_backoff_retry(func, max_retries=3, base_delay=1):
    for attempt in range(max_retries):
        try:
            return func()
        except Exception as e:
            if attempt == max_retries - 1:
                raise e
            
            delay = base_delay * (2 ** attempt) + random.uniform(0, 1)
            time.sleep(delay)
```

## 📊 Real-World Examples

### Netflix Reliability Strategy

```mermaid
graph TD
    A[Netflix Approach] --> B[Chaos Engineering]
    A --> C[Circuit Breakers]
    A --> D[Graceful Degradation]
    A --> E[Multi-Region]
    
    B --> F[Chaos Monkey<br/>Random failures]
    B --> G[Chaos Kong<br/>Region failures]
    
    C --> H[Hystrix Library<br/>Fault tolerance]
    
    D --> I[Show cached content<br/>when recommendations fail]
    
    E --> J[Active-Active<br/>Global deployment]
    
    style A fill:#E50914
```

### Amazon's Reliability Principles

1. **Everything Fails**: Design for failure from day one
2. **Loose Coupling**: Services should be independent
3. **Fault Isolation**: Failures shouldn't propagate
4. **Gradual Rollouts**: Deploy changes incrementally

### Google's SRE Practices

```mermaid
graph TD
    A[Google SRE] --> B[Error Budgets]
    A --> C[Toil Reduction]
    A --> D[Automation]
    A --> E[Monitoring]
    
    B --> F[Balance reliability<br/>vs feature velocity]
    C --> G[Eliminate repetitive<br/>manual work]
    D --> H[Automate operations<br/>and responses]
    E --> I[Comprehensive<br/>observability]
    
    style A fill:#4285F4
```

## 🎯 Best Practices

### 1. Design for Failure
- Assume everything will fail
- Plan for graceful degradation
- Implement proper error handling
- Use timeouts and circuit breakers

### 2. Monitoring & Observability
- Comprehensive metrics collection
- Real-time alerting
- Distributed tracing
- Regular health checks

### 3. Testing Strategies
```mermaid
graph TD
    A[Testing Strategy] --> B[Unit Tests<br/>Component isolation]
    A --> C[Integration Tests<br/>Service interactions]
    A --> D[Load Tests<br/>Performance validation]
    A --> E[Chaos Tests<br/>Failure scenarios]
    
    style E fill:#ffcccc
```

### 4. Incident Response
- Clear escalation procedures
- Post-mortem analysis
- Blameless culture
- Continuous improvement

## ⚖️ Trade-offs

| Approach | Reliability Benefit | Cost |
|----------|-------------------|------|
| **Redundancy** | High fault tolerance | Increased infrastructure cost |
| **Multi-Region** | Geographic disaster recovery | Complexity & latency |
| **Circuit Breakers** | Prevents cascade failures | Increased code complexity |
| **Monitoring** | Fast issue detection | Operational overhead |

---

**Key Takeaway**: Reliability and availability are achieved through deliberate design choices, not accidents. Plan for failure, measure everything, and continuously improve based on real-world data.
