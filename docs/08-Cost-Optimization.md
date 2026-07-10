# 08 — Cost Optimization

## Cost Principles

> **FinOps Culture** — Every engineer owns cost. Optimize for value, not just spend.

## Cost Overview

| Category | Monthly Budget | Current Spend | Status |
|----------|---------------|---------------|--------|
| Compute (EKS) | $20,000 | — | 🟢 On Track |
| Database (RDS) | $10,000 | — | 🟢 On Track |
| Cache (Redis) | $3,000 | — | 🟢 On Track |
| Storage (S3) | $2,000 | — | 🟢 On Track |
| Networking | $5,000 | — | 🟢 On Track |
| Monitoring | $2,000 | — | 🟢 On Track |
| **Total** | **$42,000** | — | 🟢 |

## Optimization Strategies

### 1. Right-Sizing

```
┌─────────────────────────────────────────────────────────┐
│                   RIGHT-SIZING                           │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  Current:  m5.2xlarge (8 vCPU, 32 GB)  × 10           │
│  Actual:   30% CPU, 45% Memory average                  │
│                                                         │
│  Optimized: m5.xlarge (4 vCPU, 16 GB)  × 6             │
│  Savings:   ~$800/month                                 │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

### 2. Auto-Scaling

| Resource | Min | Max | Target | Strategy |
|----------|-----|-----|--------|----------|
| EKS Pods | 2 | 50 | 70% CPU | HPA |
| EKS Nodes | 3 | 20 | 60% CPU | Cluster Autoscaler |
| RDS | — | — | — | Read replicas on-demand |

### 3. Spot Instances

| Workload | Spot Savings | Availability | Risk |
|----------|-------------|--------------|------|
| Stateless services | 60–70% | 95% | Low |
| Batch processing | 60–70% | 95% | Low |
| Databases | — | — | Not recommended |
| Monitoring | 60–70% | 95% | Medium |

### 4. Reserved Instances

| Resource | Commitment | Discount | Recommendation |
|----------|------------|----------|----------------|
| EKS Control Plane | 1 year | 30% | ✅ Yes |
| RDS (PostgreSQL) | 1 year | 35% | ✅ Yes |
| ElastiCache | 1 year | 30% | ✅ Yes |
| EC2 (baseline) | 1 year | 40% | ✅ Yes |

### 5. Storage Optimization

| Strategy | Savings | Implementation |
|----------|---------|----------------|
| Lifecycle policies | 40–60% | Move old data to Glacier |
| Compression | 50–70% | Enable compression at rest |
| Deduplication | 20–30% | Application-level |
| Delete unused | 100% | Automated cleanup |

## Cost Allocation

### Tagging Strategy

| Tag | Required | Example | Purpose |
|-----|----------|---------|---------|
| `env` | ✅ | `production`, `staging` | Environment tracking |
| `team` | ✅ | `platform`, `payments` | Team ownership |
| `service` | ✅ | `user-service`, `api-gateway` | Service identification |
| `cost-center` | ✅ | `engineering`, `data` | Department billing |
| `project` | ❌ | `migration-v2` | Project tracking |

### Cost Dashboard Metrics

| Metric | Description | Alert Threshold |
|--------|-------------|-----------------|
| Daily spend | Total AWS spend | > $2,000 |
| Cost per service | Allocated cost | > budget + 20% |
| Cost per request | Unit economics | Increasing trend |
| Unused resources | Idle resources | > $500/month |

## Monthly Cost Review

### Review Process

```
┌─────────────────────────────────────────────────────────┐
│              MONTHLY COST REVIEW                         │
├─────────────────────────────────────────────────────────┤
│  1. Automated report generation (AWS Cost Explorer)     │
│  2. Team leads review service costs                     │
│  3. Identify optimization opportunities                 │
│  4. Implement top 3 optimizations                       │
│  5. Track progress vs. budget                           │
│  6. Update forecasts                                    │
└─────────────────────────────────────────────────────────┘
```

### Optimization Backlog

| Priority | Item | Est. Savings | Effort | Status |
|----------|------|-------------|--------|--------|
| P1 | Right-size EKS nodes | $2,000/mo | Low | 📋 Planned |
| P2 | Enable spot instances | $5,000/mo | Medium | 📋 Planned |
| P3 | S3 lifecycle policies | $800/mo | Low | 📋 Planned |
| P4 | Reserved instances | $3,000/mo | Low | 📋 Planned |
| P5 | Optimize data transfer | $1,000/mo | High | 📋 Planned |

## Cost Alerts

| Alert | Threshold | Action |
|-------|-----------|--------|
| Daily spend > $2,000 | Warning | Slack notification |
| Daily spend > $3,000 | Critical | Page on-call |
| Service over budget | +20% | Team notification |
| Unused resources > $500 | Monthly | Auto-cleanup or alert |
| Forecast > budget | +10% | Leadership notification |

## FinOps Maturity Model

| Level | Stage | Description | Status |
|-------|-------|-------------|--------|
| 1 | Inform | Visibility into spend | ✅ Current |
| 2 | Optimize | Cost reduction initiatives | 🎯 Target |
| 3 | Operate | Automated cost management | 📅 Future |

## Tools & Resources

| Tool | Purpose | Access |
|------|---------|--------|
| AWS Cost Explorer | Cost visualization | All engineers |
| Kubecost | K8s cost allocation | Platform team |
| Infracost | IaC cost estimation | CI pipeline |
| CloudHealth | Cost optimization | Finance team |
