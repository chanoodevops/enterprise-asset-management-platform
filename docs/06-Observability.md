# 06 — Observability

## Observability Pillars

```
┌─────────────────────────────────────────────────────────────────┐
│                    OBSERVABILITY STACK                          │
├─────────────────┬─────────────────┬─────────────────────────────┤
│     LOGS        │    METRICS      │        TRACES               │
│                 │                 │                             │
│  What happened  │  How is it      │  Where is time              │
│  (Loki)         │  (Prometheus)   │  spent? (Tempo)             │
│                 │                 │                             │
│  • Structured   │  • Counters     │  • Distributed tracing      │
│  • Searchable   │  • Histograms   │  • Request flow             │
│  • Correlated   │  • Gauges       │  • Latency breakdown        │
└─────────────────┴─────────────────┴─────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                      VISUALIZATION                              │
│                     Grafana Dashboards                          │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐       │
│  │Overview  │  │ Service  │  │  Infra   │  │ Business │       │
│  │ Dashboard│  │ Dashboards│ │ Metrics  │  │ Metrics  │       │
│  └──────────┘  └──────────┘  └──────────┘  └──────────┘       │
└─────────────────────────────────────────────────────────────────┘
```

## Key Metrics (RED Method)

### Request Metrics

| Metric | Description | Target |
|--------|-------------|--------|
| **Rate** | Requests per second | 100–10,000 rps |
| **Errors** | Error rate (4xx + 5xx) | < 0.1% |
| **Duration** | Request latency (p50/p99) | < 50ms / < 200ms |

### Service Health Metrics

| Metric | Description | Threshold |
|--------|-------------|-----------|
| `http_request_duration_seconds` | Request latency histogram | p99 < 200ms |
| `http_requests_total` | Total request count | — |
| `http_requests_errors_total` | Error count | < 0.1% |
| `service_up` | Service health check | 1 = healthy |

### Infrastructure Metrics

| Metric | Description | Threshold |
|--------|-------------|-----------|
| `node_cpu_seconds_total` | CPU utilization | < 80% |
| `node_memory_MemAvailable_bytes` | Memory available | > 20% free |
| `container_network_receive_bytes_total` | Network I/O | — |
| `container_fs_usage_bytes` | Disk usage | < 85% |

## Alerting

### Alert Severity Levels

| Severity | Response Time | Notification | Example |
|----------|---------------|--------------|---------|
| **P0 — Critical** | < 5 min | PagerDuty + Slack + Phone | Complete outage |
| **P1 — High** | < 15 min | PagerDuty + Slack | Service degradation |
| **P2 — Medium** | < 1 hour | Slack | Warning threshold |
| **P3 — Low** | Next business day | Email | Informational |

### Alert Rules

```yaml
# Example: High Error Rate Alert
- alert: HighErrorRate
  expr: |
    rate(http_requests_errors_total[5m]) /
    rate(http_requests_total[5m]) > 0.01
  for: 5m
  labels:
    severity: P1
  annotations:
    summary: "High error rate on {{ $labels.service }}"
    description: "Error rate is {{ $value | humanizePercentage }}"

# Example: High Latency Alert
- alert: HighLatency
  expr: |
    histogram_quantile(0.99,
      rate(http_request_duration_seconds_bucket[5m])
    ) > 0.2
  for: 5m
  labels:
    severity: P1
  annotations:
    summary: "High p99 latency on {{ $labels.service }}"
```

## Distributed Tracing

### Trace Propagation

```
┌────────┐     ┌─────────┐     ┌──────────┐     ┌──────────┐
│ Client │────▶│ Gateway │────▶│ Service  │────▶│ Database │
│        │     │         │     │   A      │     │          │
└────────┘     └─────────┘     └──────────┘     └──────────┘
     │              │               │                │
     ▼              ▼               ▼                ▼
  TraceID: abc123
  SpanID:  001     002           003              004
  Parent:  -       001           002              003
```

### Trace Context Headers

| Header | Format | Purpose |
|--------|--------|---------|
| `traceparent` | `00-{trace-id}-{span-id}-{flags}` | W3C trace context |
| `tracestate` | `vendor=value,...` | Vendor-specific data |

## Logging Standards

### Structured Log Format

```json
{
  "timestamp": "2026-07-10T14:32:15.123Z",
  "level": "info",
  "service": "user-service",
  "trace_id": "abc123def456",
  "span_id": "789ghi012",
  "message": "User created successfully",
  "context": {
    "user_id": "usr_123",
    "email": "user@example.com",
    "duration_ms": 45
  }
}
```

### Log Levels

| Level | When to Use | Example |
|-------|-------------|---------|
| `ERROR` | Unrecoverable errors | Database connection lost |
| `WARN` | Recoverable issues | Retry attempt failed |
| `INFO` | Business events | Order placed |
| `DEBUG` | Development debugging | Request payload |
| `TRACE` | Extremely verbose | Function entry/exit |

## Dashboards

### Overview Dashboard

- System health status (up/down)
- Request rate and error rate
- Latency percentiles (p50, p95, p99)
- Active alerts
- Resource utilization summary

### Service Dashboard

- Per-service request metrics
- Dependency graph
- Database query performance
- Cache hit/miss ratio
- Queue depth and processing rate

### Business Dashboard

- Active users (DAU/MAU)
- Transaction volume
- Revenue metrics
- Feature adoption rates
- SLA compliance

## On-Call Runbook

### Incident Response Process

1. **Detect** — Alert fires, on-call engineer notified
2. **Acknowledge** — Acknowledge alert within 5 minutes
3. **Triage** — Assess severity and impact
4. **Mitigate** — Apply immediate fix (rollback, scale, etc.)
5. **Resolve** — Root cause identified and fixed
6. **Review** — Post-incident review within 48 hours

### Common Issues

| Symptom | Likely Cause | Action |
|---------|--------------|--------|
| High error rate | Bad deployment | Rollback to previous version |
| High latency | Resource exhaustion | Scale up pods/nodes |
| Memory leak | Application bug | Restart pod, file ticket |
| Connection pool exhaustion | Too many connections | Increase pool size, investigate |
