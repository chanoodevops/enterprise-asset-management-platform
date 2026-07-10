# 03 — Technology Stack

## Overview

| Layer | Technology | Rationale |
|-------|-----------|-----------|
| **Language** | TypeScript / Go | Type safety, performance, ecosystem |
| **Runtime** | Node.js 20+ / Go 1.22+ | LTS support, performance |
| **Framework** | NestJS (API), Next.js (Web) | Enterprise patterns, React ecosystem |
| **Database** | PostgreSQL 16 | ACID compliance, JSON support |
| **Cache** | Redis 7 (Valkey) | In-memory performance, pub/sub |
| **Search** | Elasticsearch 8 / OpenSearch | Full-text search, analytics |
| **Message Queue** | Apache Kafka (MSK) | Event streaming, durability |
| **Container** | Docker, containerd | Industry standard |
| **Orchestration** | Kubernetes (EKS) | Declarative infrastructure |
| **IaC** | Terraform + Helm | Multi-cloud, templating |
| **CI/CD** | GitHub Actions + ArgoCD | GitOps workflow |
| **Monitoring** | Grafana Stack (Prometheus, Loki, Tempo) | Unified observability |
| **Tracing** | OpenTelemetry + Tempo | Vendor-neutral instrumentation |
| **Secrets** | HashiCorp Vault | Dynamic secrets, encryption |
| **Service Mesh** | Istio | mTLS, traffic management |

## Frontend Stack

| Component | Technology | Version |
|-----------|-----------|---------|
| Framework | Next.js | 14+ |
| Language | TypeScript | 5.4+ |
| UI Library | React | 18+ |
| State Management | Zustand / React Query | Latest |
| Styling | Tailwind CSS | 3.4+ |
| Testing | Vitest + Playwright | Latest |
| Build Tool | Turbopack | Latest |

## Backend Stack

| Component | Technology | Version |
|-----------|-----------|---------|
| Runtime | Node.js | 20 LTS |
| Framework | NestJS | 10+ |
| Language | TypeScript | 5.4+ |
| ORM | Prisma | 5+ |
| Validation | Zod | 3.23+ |
| Auth | Passport.js + JWT | Latest |
| Testing | Jest + Supertest | Latest |

## Infrastructure Stack

| Component | Technology | Provider |
|-----------|-----------|----------|
| Compute | EKS (Kubernetes) | AWS |
| Load Balancer | ALB / NLB | AWS |
| CDN | CloudFront | AWS |
| DNS | Route 53 | AWS |
| Object Storage | S3 | AWS |
| Database | RDS PostgreSQL | AWS |
| Cache | ElastiCache Redis | AWS |
| Secrets | AWS Secrets Manager / Vault | AWS / Self-hosted |
| CI/CD | GitHub Actions | GitHub |
| GitOps | ArgoCD | Self-hosted on K8s |

## Development Tools

| Tool | Purpose |
|------|---------|
| VS Code | IDE |
| Docker Desktop | Local containerization |
| Tilt / Skaffold | Local K8s development |
| Postman / Bruno | API testing |
| pgAdmin / DBeaver | Database management |
| Terraform Cloud | State management |

## Version Matrix

| Dependency | Min Version | Target Version | Notes |
|------------|-------------|----------------|-------|
| Node.js | 20.0.0 | 20.11.x | LTS |
| Go | 1.22.0 | 1.22.x | Latest stable |
| TypeScript | 5.4.0 | 5.5.x | Strict mode |
| Kubernetes | 1.28 | 1.30 | EKS |
| Terraform | 1.7.0 | 1.8.x | AWS provider 5.x |
| Helm | 3.14 | 3.15+ | Chart management |
