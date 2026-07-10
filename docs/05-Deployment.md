# 05 — Deployment

## Deployment Strategy

> **GitOps-First** — All deployments are declarative, version-controlled, and auditable via Git.

## Deployment Pipeline

```
┌─────────────────────────────────────────────────────────────────┐
│                     CI PIPELINE (GitHub Actions)                │
├─────────────────────────────────────────────────────────────────┤
│  ┌──────┐   ┌──────────┐   ┌──────────┐   ┌──────────┐        │
│  │ Lint │──▶│  Test    │──▶│  Build   │──▶│  Scan    │        │
│  │      │   │          │   │          │   │          │        │
│  │ ESLint│  │ Unit +   │   │ Docker   │   │ Trivy +  │        │
│  │ Prettier│ Integration│  │ Images   │   │ Snyk     │        │
│  └──────┘   └──────────┘   └──────────┘   └──────────┘        │
│                                                │                │
│                                                ▼                │
│                                      ┌──────────────┐          │
│                                      │  Push to ECR │          │
│                                      └──────┬───────┘          │
│                                             │                  │
└─────────────────────────────────────────────┼──────────────────┘
                                              │
                                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                   CD PIPELINE (ArgoCD)                          │
├─────────────────────────────────────────────────────────────────┤
│  ┌──────────────┐   ┌──────────────┐   ┌──────────────┐        │
│  │  Git Sync    │──▶│  Diff Check  │──▶│   Deploy     │        │
│  │              │   │              │   │              │        │
│  │  Detect      │   │  Show changes│   │  Apply to    │        │
│  │  manifest    │   │  between     │   │  target      │        │
│  │  changes     │   │  desired vs  │   │  cluster     │        │
│  │              │   │  actual      │   │              │        │
│  └──────────────┘   └──────────────┘   └──────────────┘        │
│                                               │                 │
│                                               ▼                 │
│                                     ┌──────────────┐           │
│                                     │  Health Check│           │
│                                     │  + Rollback  │           │
│                                     └──────────────┘           │
└─────────────────────────────────────────────────────────────────┘
```

## Environments

| Environment | Branch | Auto-Deploy | Approval | Traffic |
|-------------|--------|-------------|----------|---------|
| `dev` | `feature/*` | ✅ Yes | None | Internal testing |
| `staging` | `main` | ✅ Yes | Auto-merge | Pre-prod validation |
| `production` | `release/*` | ❌ Manual | Required | Live traffic |
| `dr` | `release/*` | ❌ Manual | Required | Failover only |

## Deployment Patterns

### Blue-Green Deployment

```
┌──────────────┐        ┌──────────────┐
│   Blue       │        │   Green      │
│   (Current)  │        │   (New)      │
│              │        │              │
│   Traffic ──▶│        │   Traffic ──▶│
│              │        │              │
│   100%       │        │   0%         │
└──────────────┘        └──────────────┘
                              │
                              │ Switch
                              ▼
┌──────────────┐        ┌──────────────┐
│   Blue       │        │   Green      │
│   (Old)      │        │   (Current)  │
│              │        │              │
│   Traffic ──▶│        │   Traffic ──▶│
│              │        │              │
│   0%         │        │   100%       │
└──────────────┘        └──────────────┘
```

### Canary Deployment

```
┌─────────────────────────────────────────┐
│         Traffic Distribution            │
├─────────────────────────────────────────┤
│  v1.0.0  ████████████████████░░░░  80%  │
│  v1.1.0  ████░░░░░░░░░░░░░░░░░░  20%  │
├─────────────────────────────────────────┤
│  Monitor → Increase → Full Rollout      │
└─────────────────────────────────────────┘
```

## Infrastructure as Code

### Directory Structure

```
infrastructure/
├── environments/
│   ├── dev/
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   └── terraform.tfvars
│   ├── staging/
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   └── terraform.tfvars
│   └── production/
│       ├── main.tf
│       ├── variables.tf
│       └── terraform.tfvars
├── modules/
│   ├── vpc/
│   ├── eks/
│   ├── rds/
│   ├── redis/
│   └── monitoring/
└── shared/
    └── remote-state.tf
```

## Rollback Procedures

| Scenario | Automated | Manual | Time to Rollback |
|----------|-----------|--------|-------------------|
| Failed health check | ✅ | ❌ | < 2 minutes |
| Error rate spike | ✅ | ❌ | < 5 minutes |
| Performance degradation | ❌ | ✅ | < 15 minutes |
| Data corruption | ❌ | ✅ | < 30 minutes |

## Deployment Checklist

- [ ] Code reviewed and approved
- [ ] All tests passing
- [ ] Security scan clean
- [ ] Database migrations tested
- [ ] Feature flags configured
- [ ] Monitoring dashboards updated
- [ ] Runbooks updated
- [ ] Rollback plan documented
- [ ] Communication sent to stakeholders
