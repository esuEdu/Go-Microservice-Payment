# Go Microservice Payment

A production-grade payments backend written in Go, built from an empty folder up to a set of microservices running on Kubernetes.

This repository is my code for the course **[Go Microservices Full Course: Build a Payments System From Scratch](https://www.youtube.com/watch?v=AVy0hy4qSnQ)** by [CodeWise](https://www.youtube.com/@withcodewise). The course has 24 episodes and about 20 hours of content.

> **Status:** work in progress. I'm building this episode by episode. See the [roadmap](#roadmap) for progress.

## What it does

- **Double-entry ledger** in PostgreSQL with append-only entries and derived balances, so money is never created or destroyed by accident
- **Safe money movement** using row locks, deadlock handling, retries with jitter and idempotency keys (with Redis as a fast path)
- **Auth** with hashed passwords, signed JWTs and refresh-token rotation
- **Service-to-service calls over gRPC**, with a REST/JSON **API gateway** in front
- **Event-driven flows** with NATS JetStream and the transactional outbox pattern
- **Transfers as sagas**, using wallet holds and an orchestrator
- **Notifications** with retries and a dead-letter queue
- **Outbound webhooks** that are signed, with SSRF protection
- **Multi-currency support** with FX conversion
- **Fraud and risk checks**, with a manual review API
- **Reconciliation and reporting**, using leader election and CSV statements
- **Security hardening** with mTLS, least-privilege database roles and an audit log

## Tech stack

| Area | Tools |
| --- | --- |
| Language | Go |
| Database | PostgreSQL, [sqlc](https://sqlc.dev/), [pgx](https://github.com/jackc/pgx) |
| Cache | Redis |
| Messaging | NATS JetStream |
| RPC | gRPC (HTTP/JSON at the gateway) |
| Observability | OpenTelemetry, Prometheus, Grafana |
| Testing | Go test, testcontainers, the race detector, k6 |
| Delivery | Multi-stage distroless Docker images, Docker Compose, GitHub Actions |
| Deployment | Kubernetes, Helm |

## Architecture

```
                 ┌──────────────┐
  client ──HTTP──▶  API Gateway │
                 └──────┬───────┘
                        │ gRPC
        ┌───────────────┼────────────────┐
        ▼               ▼                ▼
   ┌─────────┐    ┌──────────┐     ┌──────────┐
   │  Auth   │    │  Wallet  │◀────│ Transfer │  (saga orchestrator)
   └─────────┘    │ (ledger) │     └────┬─────┘
                  └────┬─────┘          │
                       │ outbox         │ outbox
                       ▼                ▼
                 ┌────────────────────────────┐
                 │       NATS JetStream       │
                 └──────┬──────────────┬──────┘
                        ▼              ▼
                ┌──────────────┐ ┌──────────┐
                │ Notification │ │ Webhooks │
                └──────────────┘ └──────────┘

   PostgreSQL · Redis · OpenTelemetry → Prometheus / Grafana
```

This is the target design. It will be refined as the services get built.

## Getting started

### Prerequisites

- [Go](https://go.dev/dl/) (latest stable)
- [Docker](https://www.docker.com/) with Docker Compose
- Optional, for later episodes: [grpcurl](https://github.com/fullstorydev/grpcurl), [k6](https://k6.io/), [kind](https://kind.sigs.k8s.io/) or minikube, and [Helm](https://helm.sh/)

### Running locally

Setup and run instructions will be added here as the local infrastructure and services are built.

## Roadmap

- [ ] 01: Project setup, local infrastructure, graceful shutdown
- [ ] 02: Designing a double-entry ledger
- [ ] 03: Type-safe SQL with sqlc + pgx
- [ ] 04: Config, logging and errors
- [ ] 05: The wallet API
- [ ] 06: Moving money safely (locks, deadlocks, retries)
- [ ] 07: Idempotency keys
- [ ] 08: The auth service
- [ ] 09: gRPC between services
- [ ] 10: The API gateway
- [ ] 11: Events with NATS JetStream and the transactional outbox
- [ ] 12: Transfer service and sagas
- [ ] 13: Notification service
- [ ] 14: Observability
- [ ] 15: Testing like a pro
- [ ] 16: Shipping it (health checks, Docker, CI, Compose)
- [ ] 17: Outbound webhooks
- [ ] 18: Multi-currency and FX
- [ ] 19: Fraud and risk checks
- [ ] 20: Reconciliation and reporting
- [ ] 21: Security hardening
- [ ] 22: Performance
- [ ] 23: Kubernetes
- [ ] 24: Chaos engineering and the final demo

## Credits

- Course: [Go Microservices Full Course: Build a Payments System From Scratch](https://www.youtube.com/watch?v=AVy0hy4qSnQ) by CodeWise
- Course page with chapters: <https://www.fiston.dev/courses/build-a-production-grade-payments-backend-in-go>
- Reference implementation: <https://github.com/dev-marker/payments>
