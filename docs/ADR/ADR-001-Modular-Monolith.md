# ADR-001

## Title

Use Modular Monolith for MVP

---

## Status

Accepted

---

## Date

2026-07-10

---

# Context

The Enterprise Asset Management Platform is being developed by a single engineer as a portfolio project.

The project aims to demonstrate enterprise software architecture while remaining achievable within approximately six months.

Several architectural approaches were considered:

- Monolith
- Modular Monolith
- Microservices

---

# Decision

The MVP will adopt a Modular Monolith Architecture.

The application will be organized into clearly separated business modules while remaining a single deployable application.

---

# Reasons

Advantages

- Faster development
- Easier debugging
- Easier deployment
- Lower AWS cost
- Easier local development
- Simpler testing
- Reduced operational complexity

---

# Consequences

Positive

- Faster delivery
- Easier onboarding
- Lower infrastructure cost
- Simpler CI/CD
- Easier troubleshooting

Negative

- Entire application deployed together
- Scaling occurs at application level
- Module boundaries must be carefully maintained

---

# Alternatives Considered

## Traditional Monolith

Pros

- Very simple

Cons

- Poor separation
- Difficult long-term maintenance

Rejected.

---

## Microservices

Pros

- Independent deployment
- Independent scaling
- Technology flexibility

Cons

- Operational complexity
- Service discovery
- Distributed transactions
- Higher AWS cost
- Increased DevOps effort

Rejected for MVP.

Future versions may extract modules into microservices when business growth justifies the additional complexity.

---

# Decision Owner

Solution Architect

---

# Related Documents

- Project Brief
- PRD
- Solution Architecture
