# 07 — Disaster Recovery

## Recovery Objectives

| Metric | Target | Description |
|--------|--------|-------------|
| **RPO** | < 5 minutes | Maximum acceptable data loss |
| **RTO** | < 30 minutes | Maximum acceptable downtime |
| **MTTR** | < 30 minutes | Mean time to recovery |

## Disaster Scenarios

| Scenario | Impact | Probability | Recovery Strategy |
|----------|--------|-------------|-------------------|
| Single pod failure | Minimal | High | Auto-restart, self-healing |
| Node failure | Low | Medium | Pod rescheduling |
| AZ failure | High | Low | Multi-AZ failover |
| Region failure | Critical | Very Low | Cross-region failover |
| Data corruption | Critical | Low | Point-in-time restore |
| Security breach | Critical | Low | Incident response plan |

## Backup Strategy

### Database Backups

| Type | Frequency | Retention | Storage |
|------|-----------|-----------|---------|
| Automated snapshot | Daily | 30 days | S3 (cross-region) |
| WAL archival | Continuous | 7 days | S3 |
| Manual backup | Before migrations | 90 days | S3 |
| Logical dump | Weekly | 30 days | S3 |

### Backup Verification

```
┌─────────────────────────────────────────────────────────┐
│                BACKUP VERIFICATION                      │
├─────────────────────────────────────────────────────────┤
│  Daily:   Automated restore test to staging            │
│  Weekly:  Full integrity check on backups              │
│  Monthly: DR drill with full recovery                  │
│  Annual:  Third-party audit of backup procedures       │
└─────────────────────────────────────────────────────────┘
```

## Failover Architecture

### Multi-AZ (Primary)

```
┌─────────────────────────────────────────────────────────┐
│                     REGION: us-east-1                    │
├────────────────────┬────────────────────┬───────────────┤
│     AZ-1a          │      AZ-1b         │    AZ-1c      │
│  ┌──────────┐     │  ┌──────────┐     │ ┌──────────┐  │
│  │   EKS    │     │  │   EKS    │     │ │   EKS    │  │
│  │  Nodes   │     │  │  Nodes   │     │ │  Nodes   │  │
│  └────┬─────┘     │  └────┬─────┘     │ └────┬─────┘  │
│       │           │       │           │      │         │
│  ┌────┴─────┐     │  ┌────┴─────┐     │ ┌────┴─────┐  │
│  │   RDS    │     │  │   RDS    │     │ │   RDS    │  │
│  │  Primary │─────│──│ Replica  │     │ │ Standby  │  │
│  └──────────┘     │  └──────────┘     │ └──────────┘  │
└────────────────────┴────────────────────┴───────────────┘
```

### Cross-Region (DR)

```
┌──────────────────────────┐      ┌──────────────────────────┐
│   PRIMARY: us-east-1     │      │   DR: us-west-2          │
├──────────────────────────┤      ├──────────────────────────┤
│   ┌────────────────┐    │      │   ┌────────────────┐    │
│   │  Application   │    │      │   │  Application   │    │
│   │    Cluster     │    │      │   │    Cluster     │    │
│   └───────┬────────┘    │      │   └───────┬────────┘    │
│           │             │      │           │             │
│   ┌───────┴────────┐    │  ▲   │   ┌───────┴────────┐    │
│   │  Primary DB    │────│──│───│──▶│  Replica DB    │    │
│   └────────────────┘    │  │   │   └────────────────┘    │
│                         │  │   │                         │
│   ┌────────────────┐    │  │   │   ┌────────────────┐    │
│   │  Object Store  │────│──│───│──▶│  Object Store  │    │
│   │  (S3)          │    │  │   │   │  (Replica)     │    │
│   └────────────────┘    │  │   │   └────────────────┘    │
└──────────────────────────┘      └──────────────────────────┘
                              │
                     Cross-Region Replication
```

## Recovery Procedures

### Scenario 1: Pod Failure (Automatic)

```
1. Kubernetes detects pod failure
2. ReplicaSet creates new pod
3. Load balancer routes traffic to new pod
4. Time: < 30 seconds
```

### Scenario 2: Node Failure (Automatic)

```
1. Node health check fails
2. Taint applied to node
3. Pods evicted and rescheduled
4. New node provisioned (if autoscaling)
5. Time: < 2 minutes
```

### Scenario 3: AZ Failure (Semi-Auto)

```
1. Multi-AZ load balancer detects AZ failure
2. Traffic rerouted to healthy AZs
3. RDS failover to standby AZ
4. Verify service health
5. Time: < 5 minutes
```

### Scenario 4: Region Failure (Manual)

```
1. Declare disaster (RTO clock starts)
2. Promote DR region read replicas to primary
3. Update DNS to point to DR region
4. Verify all services operational
5. Communicate status to stakeholders
6. Time: < 30 minutes
```

### Scenario 5: Data Corruption (Manual)

```
1. Stop writes to affected database
2. Identify corruption scope and time
3. Restore to point-in-time before corruption
4. Replay WAL logs to recover recent valid data
5. Verify data integrity
6. Resume operations
7. Time: < 30 minutes
```

## Communication Plan

| Phase | Audience | Channel | Message |
|-------|----------|---------|---------|
| Detection | On-call team | PagerDuty | Incident detected |
| Triage | Engineering | Slack #incidents | Impact assessment |
| Escalation | Leadership | Email/Phone | Business impact |
| Resolution | All stakeholders | Status page | Resolution update |
| Post-mortem | Engineering | Meeting | Root cause analysis |

## DR Testing Schedule

| Test Type | Frequency | Scope | Participants |
|-----------|-----------|-------|--------------|
| Pod restart | Continuous | Single service | Automated |
| Node drain | Monthly | Worker node | Platform team |
| AZ failover | Quarterly | Full stack | Platform team |
| Region failover | Annually | Full platform | All teams |
| Tabletop exercise | Semi-annually | Incident response | Leadership + Eng |
