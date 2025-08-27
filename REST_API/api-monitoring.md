# API Monitoring

Monitoring ensures API reliability, performance, and business SLAs.

## What to Monitor

- Availability and uptime (SLA/SLO)
- Latency percentiles (p50/p95/p99)
- Error rates by code/class
- Throughput (RPS), saturation, resource usage
- Business KPIs (orders, signups)

## Techniques

- Health checks (/health, /ready, /live)
- Synthetic checks from multiple regions
- Distributed tracing (W3C Trace Context)
- Structured logs with correlation IDs
- Metrics and dashboards (Prometheus, Grafana)

## Alerting

- Define SLOs and error budgets
- Multi-signal alerts (error rate + latency)
- On-call runbooks and playbooks

## Example Health Endpoint

```http
GET /health
200 OK
{
  "status": "healthy",
  "dependencies": {
    "db": { "status": "ok", "latency_ms": 5 },
    "redis": { "status": "ok", "latency_ms": 1 }
  }
}
```

## Interview Questions

1. What are your key API SLOs?
2. How do you trace requests across services?
3. How do you detect partial outages?
4. What metrics predict incidents early?
5. How do you design actionable alerts?
