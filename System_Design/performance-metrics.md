# Performance Metrics

Understanding and measuring performance metrics is crucial for building scalable systems. This guide covers key metrics, monitoring strategies, and optimization techniques.

## 📊 Key Performance Metrics

### Response Time Metrics

```mermaid
graph TD
    A[Response Time Components] --> B[Network Latency]
    A --> C[Server Processing Time]
    A --> D[Database Query Time]
    A --> E[External API Calls]
    
    B --> F[DNS Resolution: 1-100ms]
    B --> G[TCP Handshake: 1-3ms]
    B --> H[Data Transfer: Variable]
    
    C --> I[CPU Processing: 1-50ms]
    C --> J[Memory Access: <1ms]
    C --> K[Disk I/O: 1-10ms]
    
    style A fill:#87CEEB
    style C fill:#98FB98
```

#### Percentile Measurements
```
P50 (Median): 50% of requests complete within this time
P90: 90% of requests complete within this time  
P95: 95% of requests complete within this time
P99: 99% of requests complete within this time
P99.9: 99.9% of requests complete within this time
```

**Why Percentiles Matter:**
- Average can be misleading due to outliers
- P95/P99 show user experience for slowest requests
- Tail latencies often indicate system bottlenecks

### Throughput Metrics

#### Requests Per Second (QPS/RPS)
```mermaid
graph LR
    A[Load Testing Results] --> B[Peak QPS: 10,000]
    A --> C[Average QPS: 3,000]
    A --> D[Sustained QPS: 5,000]
    
    E[Capacity Planning] --> F[Current: 3,000 QPS]
    E --> G[Target: 15,000 QPS]
    E --> H[Growth: 5x capacity needed]
    
    style B fill:#ffcccc
    style G fill:#ccffcc
```

#### Bandwidth Utilization
```
Inbound Bandwidth = QPS × Average Request Size
Outbound Bandwidth = QPS × Average Response Size

Example:
- 5,000 QPS
- 2KB average request
- 50KB average response
- Inbound: 5,000 × 2KB = 10 MB/s
- Outbound: 5,000 × 50KB = 250 MB/s
```

### Error Rate Metrics

#### Error Classification
```mermaid
graph TD
    A[HTTP Status Codes] --> B[2xx Success]
    A --> C[4xx Client Errors]
    A --> D[5xx Server Errors]
    
    C --> E[400 Bad Request]
    C --> F[401 Unauthorized]
    C --> G[404 Not Found]
    C --> H[429 Rate Limited]
    
    D --> I[500 Internal Server Error]
    D --> J[502 Bad Gateway]
    D --> K[503 Service Unavailable]
    D --> L[504 Gateway Timeout]
    
    style B fill:#ccffcc
    style C fill:#ffffcc
    style D fill:#ffcccc
```

#### Error Rate Calculation
```python
def calculate_error_rates(metrics):
    total_requests = metrics['total_requests']
    
    error_rates = {
        'overall_error_rate': (metrics['4xx_count'] + metrics['5xx_count']) / total_requests,
        'client_error_rate': metrics['4xx_count'] / total_requests,
        'server_error_rate': metrics['5xx_count'] / total_requests,
        'critical_error_rate': metrics['5xx_count'] / total_requests
    }
    
    return {k: f"{v:.2%}" for k, v in error_rates.items()}

# Target Error Rates:
# Overall: <1%
# Server errors: <0.1% 
# Critical errors: <0.01%
```

## 🎯 System Resource Metrics

### CPU Metrics

```mermaid
graph TD
    A[CPU Metrics] --> B[CPU Utilization %]
    A --> C[Load Average]
    A --> D[CPU Steal Time]
    A --> E[Context Switches]
    
    B --> F[Target: <70% sustained]
    C --> G[1min, 5min, 15min averages]
    D --> H[Virtualization overhead]
    E --> I[Process scheduling efficiency]
    
    style B fill:#98FB98
    style F fill:#ccffcc
```

#### CPU Utilization Guidelines
```
Excellent: 0-50% (plenty of headroom)
Good: 50-70% (efficient utilization)
Warning: 70-85% (monitor closely)
Critical: 85-100% (immediate action needed)
```

### Memory Metrics

```mermaid
graph TD
    A[Memory Metrics] --> B[Memory Utilization]
    A --> C[Memory Bandwidth]
    A --> D[Cache Hit Ratios]
    A --> E[Garbage Collection]
    
    B --> F[RAM Usage: <80%]
    B --> G[Swap Usage: <10%]
    
    C --> H[Memory throughput]
    C --> I[Memory latency]
    
    D --> J[L1 Cache: >95%]
    D --> K[L2 Cache: >90%]
    D --> L[L3 Cache: >80%]
    
    style F fill:#ccffcc
    style G fill:#ffffcc
```

### Storage Metrics

#### Disk I/O Performance
```mermaid
graph LR
    A[Storage Metrics] --> B[IOPS]
    A --> C[Throughput]
    A --> D[Latency]
    A --> E[Queue Depth]
    
    B --> F[HDD: 100-200 IOPS]
    B --> G[SSD: 10,000+ IOPS]
    B --> H[NVMe: 100,000+ IOPS]
    
    C --> I[Sequential Read/Write MB/s]
    C --> J[Random Read/Write MB/s]
    
    style G fill:#ccffcc
    style H fill:#98FB98
```

## 🌐 Network Metrics

### Network Performance Indicators

```mermaid
graph TD
    A[Network Metrics] --> B[Latency]
    A --> C[Bandwidth]
    A --> D[Packet Loss]
    A --> E[Connection Metrics]
    
    B --> F[Round Trip Time RTT]
    B --> G[First Byte Time TTFB]
    
    C --> H[Upload Speed]
    C --> I[Download Speed]
    
    D --> J[Packet Drop Rate]
    D --> K[Retransmission Rate]
    
    E --> L[Connection Pool]
    E --> M[Active Connections]
    
    style B fill:#87CEEB
    style C fill:#98FB98
```

### Connection Pool Optimization
```python
class ConnectionPoolMetrics:
    def __init__(self):
        self.active_connections = 0
        self.idle_connections = 0
        self.max_connections = 100
        self.connection_wait_time = []
        self.connection_creation_time = []
    
    def calculate_pool_efficiency(self):
        total_connections = self.active_connections + self.idle_connections
        
        metrics = {
            'pool_utilization': self.active_connections / self.max_connections,
            'connection_efficiency': self.active_connections / total_connections if total_connections > 0 else 0,
            'average_wait_time': sum(self.connection_wait_time) / len(self.connection_wait_time) if self.connection_wait_time else 0,
            'pool_saturation': total_connections / self.max_connections
        }
        
        return metrics

# Optimal pool utilization: 60-80%
# Connection wait time: <10ms
# Pool saturation: <90%
```

## 📈 Application-Level Metrics

### Business Metrics

```mermaid
graph TD
    A[Business Metrics] --> B[User Engagement]
    A --> C[Transaction Metrics]
    A --> D[Feature Usage]
    A --> E[Revenue Impact]
    
    B --> F[Daily Active Users]
    B --> G[Session Duration]
    B --> H[Page Views per Session]
    
    C --> I[Transaction Success Rate]
    C --> J[Payment Processing Time]
    C --> K[Cart Abandonment Rate]
    
    D --> L[Feature Adoption Rate]
    D --> M[API Endpoint Usage]
    
    E --> N[Revenue per User]
    E --> O[Cost per Transaction]
    
    style A fill:#87CEEB
```

### Custom Application Metrics
```python
from prometheus_client import Counter, Histogram, Gauge
import time

# Define custom metrics
user_login_counter = Counter('user_logins_total', 'Total user logins')
request_duration = Histogram('request_duration_seconds', 'Request duration')
active_users = Gauge('active_users', 'Number of active users')
database_connections = Gauge('database_connections', 'Database connection pool')

class MetricsCollector:
    def __init__(self):
        self.metrics = {}
    
    def track_user_login(self, user_id, success=True):
        user_login_counter.inc()
        if success:
            # Track successful login metrics
            self.record_business_metric('successful_login', user_id)
        else:
            self.record_business_metric('failed_login', user_id)
    
    def track_request_duration(self, endpoint, duration):
        request_duration.labels(endpoint=endpoint).observe(duration)
    
    def update_active_users(self, count):
        active_users.set(count)
    
    def record_business_metric(self, metric_name, value):
        if metric_name not in self.metrics:
            self.metrics[metric_name] = []
        
        self.metrics[metric_name].append({
            'value': value,
            'timestamp': time.time()
        })
```

## 📊 Monitoring and Alerting

### Monitoring Architecture

```mermaid
graph TB
    A[Applications] --> B[Metrics Collection]
    C[Infrastructure] --> B
    D[Databases] --> B
    
    B --> E[Time Series DB<br/>Prometheus]
    E --> F[Visualization<br/>Grafana]
    E --> G[Alerting<br/>AlertManager]
    
    G --> H[PagerDuty]
    G --> I[Slack]
    G --> J[Email]
    
    K[Log Aggregation] --> L[ELK Stack]
    L --> M[Log-based Alerts]
    
    N[Distributed Tracing] --> O[Jaeger/Zipkin]
    
    style B fill:#98FB98
    style E fill:#87CEEB
    style G fill:#FFE4B5
```

### Alert Configuration Best Practices

#### SLI/SLO-Based Alerting
```yaml
# Example Prometheus alerting rules
groups:
  - name: slo_alerts
    rules:
      - alert: HighErrorRate
        expr: rate(http_requests_total{status=~"5.."}[5m]) / rate(http_requests_total[5m]) > 0.01
        for: 2m
        labels:
          severity: critical
        annotations:
          summary: "High error rate detected"
          description: "Error rate is {{ $value | humanizePercentage }}"
      
      - alert: HighLatency
        expr: histogram_quantile(0.95, rate(http_request_duration_seconds_bucket[5m])) > 0.5
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "High latency detected"
          description: "95th percentile latency is {{ $value }}s"
      
      - alert: LowThroughput
        expr: rate(http_requests_total[5m]) < 100
        for: 10m
        labels:
          severity: warning
        annotations:
          summary: "Low throughput detected"
```

## 📊 Performance Testing

### Load Testing Strategy

```mermaid
graph TD
    A[Performance Testing Types] --> B[Load Testing]
    A --> C[Stress Testing]
    A --> D[Spike Testing]
    A --> E[Volume Testing]
    A --> F[Endurance Testing]
    
    B --> G[Normal expected load]
    C --> H[Beyond normal capacity]
    D --> I[Sudden load increases]
    E --> J[Large amounts of data]
    F --> K[Extended periods]
    
    style A fill:#87CEEB
```

### Load Testing Implementation
```python
import asyncio
import aiohttp
import time
from dataclasses import dataclass
from typing import List

@dataclass
class LoadTestResult:
    response_time: float
    status_code: int
    success: bool
    timestamp: float

class LoadTester:
    def __init__(self, base_url: str, concurrent_users: int = 100):
        self.base_url = base_url
        self.concurrent_users = concurrent_users
        self.results: List[LoadTestResult] = []
    
    async def make_request(self, session: aiohttp.ClientSession, endpoint: str):
        start_time = time.time()
        try:
            async with session.get(f"{self.base_url}{endpoint}") as response:
                await response.text()
                response_time = time.time() - start_time
                
                result = LoadTestResult(
                    response_time=response_time,
                    status_code=response.status,
                    success=200 <= response.status < 300,
                    timestamp=start_time
                )
                self.results.append(result)
                return result
                
        except Exception as e:
            response_time = time.time() - start_time
            result = LoadTestResult(
                response_time=response_time,
                status_code=0,
                success=False,
                timestamp=start_time
            )
            self.results.append(result)
            return result
    
    async def run_load_test(self, endpoint: str, duration_seconds: int = 60):
        async with aiohttp.ClientSession() as session:
            start_time = time.time()
            
            while time.time() - start_time < duration_seconds:
                tasks = [
                    self.make_request(session, endpoint)
                    for _ in range(self.concurrent_users)
                ]
                await asyncio.gather(*tasks)
                await asyncio.sleep(1)  # 1 second between batches
    
    def analyze_results(self):
        if not self.results:
            return {}
        
        response_times = [r.response_time for r in self.results]
        successful_requests = [r for r in self.results if r.success]
        
        analysis = {
            'total_requests': len(self.results),
            'successful_requests': len(successful_requests),
            'error_rate': 1 - (len(successful_requests) / len(self.results)),
            'average_response_time': sum(response_times) / len(response_times),
            'p50_response_time': self.percentile(response_times, 0.5),
            'p95_response_time': self.percentile(response_times, 0.95),
            'p99_response_time': self.percentile(response_times, 0.99),
            'requests_per_second': len(self.results) / (max(r.timestamp for r in self.results) - min(r.timestamp for r in self.results))
        }
        
        return analysis
    
    def percentile(self, data: List[float], percentile: float) -> float:
        sorted_data = sorted(data)
        index = int(percentile * len(sorted_data))
        return sorted_data[min(index, len(sorted_data) - 1)]
```

## 🎯 Performance Optimization Strategies

### Database Performance Tuning

```mermaid
graph TD
    A[Database Optimization] --> B[Query Optimization]
    A --> C[Index Optimization]
    A --> D[Connection Pooling]
    A --> E[Caching Strategies]
    
    B --> F[Query Analysis]
    B --> G[Execution Plans]
    B --> H[Query Rewriting]
    
    C --> I[Index Selection]
    C --> J[Composite Indexes]
    C --> K[Index Maintenance]
    
    D --> L[Pool Sizing]
    D --> M[Connection Reuse]
    
    E --> N[Result Caching]
    E --> O[Query Caching]
    
    style A fill:#87CEEB
```

### Application Performance Optimization
```python
# Performance optimization techniques

# 1. Connection Pooling
from sqlalchemy import create_engine
from sqlalchemy.pool import QueuePool

engine = create_engine(
    'postgresql://user:pass@localhost/db',
    poolclass=QueuePool,
    pool_size=20,
    max_overflow=30,
    pool_pre_ping=True,
    pool_recycle=3600
)

# 2. Caching with TTL
import functools
import time

def timed_cache(ttl_seconds):
    def decorator(func):
        cache = {}
        
        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            key = str(args) + str(sorted(kwargs.items()))
            current_time = time.time()
            
            if key in cache:
                result, timestamp = cache[key]
                if current_time - timestamp < ttl_seconds:
                    return result
            
            result = func(*args, **kwargs)
            cache[key] = (result, current_time)
            return result
        
        return wrapper
    return decorator

# 3. Async processing for I/O operations
import asyncio
import aiohttp

async def fetch_data_async(urls):
    async with aiohttp.ClientSession() as session:
        tasks = [fetch_url(session, url) for url in urls]
        results = await asyncio.gather(*tasks)
        return results

async def fetch_url(session, url):
    async with session.get(url) as response:
        return await response.json()
```

## 📏 Performance Benchmarking

### Industry Standard Benchmarks

| System Component | Good Performance | Excellent Performance |
|------------------|------------------|--------------------|
| **Web Server** | 1,000 RPS | 10,000+ RPS |
| **Database** | 1,000 QPS | 10,000+ QPS |
| **Cache Hit Rate** | 80% | 95%+ |
| **API Response Time** | <200ms | <100ms |
| **Page Load Time** | <3 seconds | <1 second |
| **Error Rate** | <1% | <0.1% |

### Performance Testing Checklist

- [ ] Baseline performance measurements taken
- [ ] Load testing with expected traffic
- [ ] Stress testing beyond normal capacity
- [ ] Database query performance analyzed
- [ ] Caching effectiveness measured
- [ ] Memory usage patterns identified
- [ ] CPU utilization optimized
- [ ] Network latency minimized
- [ ] Error rates within acceptable limits
- [ ] Monitoring and alerting configured

---

**Key Takeaway**: Performance metrics provide the foundation for building scalable systems. Monitor what matters, set realistic targets, and continuously optimize based on real-world data.
