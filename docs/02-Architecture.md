# 02 — Architecture

## High-Level Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                        CLIENT LAYER                             │
│   ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐      │
│   │   Web    │  │  Mobile  │  │   CLI    │  │   API    │      │
│   │   Apps   │  │   Apps   │  │  Tools   │  │ Consumers│      │
│   └────┬─────┘  └────┬─────┘  └────┬─────┘  └────┬─────┘      │
└────────┼─────────────┼─────────────┼─────────────┼─────────────┘
         │             │             │             │
         ▼             ▼             ▼             ▼
┌─────────────────────────────────────────────────────────────────┐
│                      EDGE LAYER                                 │
│   ┌─────────────────────────────────────────────────────────┐   │
│   │              CDN / WAF / DDoS Protection               │   │
│   └─────────────────────┬───────────────────────────────────┘   │
│   ┌─────────────────────┴───────────────────────────────────┐   │
│   │                   API Gateway                           │   │
│   │            (Rate Limiting / Auth / Routing)             │   │
│   └─────────────────────┬───────────────────────────────────┘   │
└─────────────────────────┼───────────────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────────────┐
│                    SERVICE LAYER                                │
│   ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐      │
│   │  Auth    │  │  User    │  │  Order   │  │  Payment │      │
│   │ Service  │  │ Service  │  │ Service  │  │ Service  │      │
│   └────┬─────┘  └────┬─────┘  └────┬─────┘  └────┬─────┘      │
│        └──────┬───────┴──────┬──────┴──────┬──────┘            │
│               │              │             │                    │
│         ┌─────┴──────┐ ┌────┴─────┐ ┌─────┴──────┐            │
│         │  Service   │ │  Event   │ │  Message   │            │
│         │   Mesh     │ │  Stream  │ │   Queue    │            │
│         └────────────┘ └──────────┘ └────────────┘            │
└─────────────────────────┬───────────────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────────────┐
│                     DATA LAYER                                  │
│   ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐      │
│   │ Postgres │  │  Redis   │  │ S3       │  │ Elastic  │      │
│   │ (RDBMS)  │  │ (Cache)  │  │ (Store)  │  │ (Search) │      │
│   └──────────┘  └──────────┘  └──────────┘  └──────────┘      │
└─────────────────────────────────────────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────────────┐
│                 OBSERVABILITY LAYER                              │
│   ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐      │
│   │  Logs    │  │ Metrics  │  │  Traces  │  │  Alerts  │      │
│   │ (Loki)   │  │(Prom)    │  │(Tempo)   │  │(AlertMgr)│      │
│   └──────────┘  └──────────┘  └──────────┘  └──────────┘      │
└─────────────────────────────────────────────────────────────────┘
```

## Architecture Principles

| # | Principle | Description |
|---|-----------|-------------|
| 1 | **Loose Coupling** | Services communicate via well-defined APIs and events |
| 2 | **Single Responsibility** | Each service owns one bounded context |
| 3 | **Defense in Depth** | Multiple security layers, no single point of failure |
| 4 | **Infrastructure as Code** | All infrastructure is version-controlled and reproducible |
| 5 | **Observability by Default** | Every service emits logs, metrics, and traces |
| 6 | **Zero-Trust Networking** | mTLS between all services, no implicit trust |

## Key Components

### API Gateway

- **Technology**: Kong / AWS API Gateway
- **Responsibilities**: Rate limiting, authentication, request routing, response caching

### Service Mesh

- **Technology**: Istio / Linkerd
- **Responsibilities**: mTLS, traffic management, retry policies, circuit breaking

### Event Streaming

- **Technology**: Apache Kafka / AWS MSK
- **Responsibilities**: Async communication, event sourcing, CQRS support

### Container Orchestration

- **Technology**: Kubernetes (EKS/GKE)
- **Responsibilities**: Deployment, scaling, self-healing, resource management

## Data Flow

```
Client Request → CDN → WAF → API Gateway → Auth Check
                                              │
                                    ┌─────────┴─────────┐
                                    ▼                   ▼
                               Sync Response      Async Processing
                                    │                   │
                                    │              Event Stream
                                    │                   │
                                    ▼                   ▼
                              Data Service      Background Workers
                                    │                   │
                                    ▼                   ▼
                              Database              Database
                                    │                   │
                                    └─────────┬─────────┘
                                              ▼
                                     Response to Client
```

## Deployment Architecture

| Environment | Purpose | Infrastructure |
|-------------|---------|----------------|
| `dev` | Local development | Docker Compose / Tilt |
| `staging` | Pre-production validation | Kubernetes (single AZ) |
| `production` | Live traffic | Kubernetes (multi-AZ) |
| `dr` | Disaster recovery | Secondary region |
