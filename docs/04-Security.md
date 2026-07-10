# 04 — Security

## Security Posture

> **Zero-Trust Architecture** — Never trust, always verify. Every request is authenticated, authorized, and encrypted regardless of origin.

## Security Layers

```
┌─────────────────────────────────────────────────────┐
│                   PERIMETER                          │
│   WAF / DDoS Protection / Rate Limiting             │
├─────────────────────────────────────────────────────┤
│                   NETWORK                           │
│   VPC / Subnet Isolation / Security Groups          │
├─────────────────────────────────────────────────────┤
│                   TRANSPORT                         │
│   TLS 1.3 / mTLS Between Services                  │
├─────────────────────────────────────────────────────┤
│                   APPLICATION                       │
│   Auth / AuthZ / Input Validation / CORS            │
├─────────────────────────────────────────────────────┤
│                     DATA                            │
│   Encryption at Rest / Field-level Encryption       │
├─────────────────────────────────────────────────────┤
│                   IDENTITY                          │
│   IAM / RBAC / Service Accounts / API Keys          │
└─────────────────────────────────────────────────────┘
```

## Authentication & Authorization

### Authentication Flow

```
┌────────┐     ┌─────────┐     ┌──────────┐     ┌──────────┐
│ Client │────▶│  OIDC   │────▶│   IAM    │────▶│  Token   │
│        │     │ Provider│     │  Service │     │  Issuer  │
└────────┘     └─────────┘     └──────────┘     └──────────┘
                                                       │
                  ┌────────────────────────────────────┘
                  ▼
           ┌──────────────┐
           │   JWT Token  │
           │ ┌──────────┐ │
           │ │ Access   │ │  (15 min expiry)
           │ │ Token    │ │
           │ ├──────────┤ │
           │ │ Refresh  │ │  (7 day expiry)
           │ │ Token    │ │
           │ └──────────┘ │
           └──────────────┘
```

### Authorization Model

| Role | Permissions | Scope |
|------|-------------|-------|
| `platform-admin` | Full system access | Global |
| `team-admin` | Team management, deploy | Team-scoped |
| `developer` | Read/write services, deploy | Service-scoped |
| `viewer` | Read-only access | Resource-scoped |
| `service-account` | Programmatic access | API-scoped |

## Secret Management

| Secret Type | Storage | Rotation | Access |
|-------------|---------|----------|--------|
| Database credentials | Vault / AWS SM | Every 30 days | Service accounts |
| API keys | Vault / AWS SM | On-demand | Environment variables |
| TLS certificates | cert-manager | Auto-renewal | K8s secrets |
| Encryption keys | KMS | Annually | HSM-backed |

## Network Security

### VPC Architecture

```
┌─────────────────────────────────────────────────────┐
│                     VPC (10.0.0.0/16)               │
│  ┌─────────────────────────────────────────────┐    │
│  │         Public Subnets (10.0.0.0/20)        │    │
│  │   ALB / NAT Gateway / Bastion Host          │    │
│  └─────────────────────────────────────────────┘    │
│  ┌─────────────────────────────────────────────┐    │
│  │        Private Subnets (10.0.16.0/20)       │    │
│  │   EKS Worker Nodes / Application Pods       │    │
│  └─────────────────────────────────────────────┘    │
│  ┌─────────────────────────────────────────────┐    │
│  │        Data Subnets (10.0.32.0/20)          │    │
│  │   RDS / ElastiCache / MSK                   │    │
│  └─────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────┘
```

### Network Policies

- **Ingress**: Allow only from ALB and within service mesh
- **Egress**: Allow to internet via NAT, internal services unrestricted
- **Service-to-Service**: mTLS required, deny-by-default

## Data Security

| Data State | Protection | Standard |
|------------|-----------|----------|
| At Rest | AES-256 (KMS-managed) | AWS KMS |
| In Transit | TLS 1.3 | cert-manager |
| In Processing | Memory encryption (if available) | Runtime-dependent |
| Backup | Encrypted snapshots | AWS Backup |

## Compliance

| Framework | Status | Scope |
|-----------|--------|-------|
| SOC 2 Type II | 🔄 In Progress | Full platform |
| GDPR | ✅ Compliant | EU data subjects |
| HIPAA | ⏳ Planned | Healthcare workloads |
| PCI DSS | ⏳ Planned | Payment processing |

## Security Practices

### Code Security

- **SAST**: CodeQL, Semgrep in CI pipeline
- **SCA**: Dependabot, Snyk for dependency scanning
- **Container**: Trivy, Grype for image scanning
- **IaC**: Checkov, tfsec for Terraform scanning

### Operational Security

- **Audit Logging**: CloudTrail + application audit logs
- **Incident Response**: Runbooks for P0/P1 incidents
- **Penetration Testing**: Annual third-party assessment
- **Security Training**: Quarterly team training sessions

### Access Control

- **Just-in-Time (JIT)** access for production
- **MFA** required for all human access
- **Service accounts** with minimal permissions
- **API key** rotation on schedule
