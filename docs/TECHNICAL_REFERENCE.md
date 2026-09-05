# Technical Reference for AI-Assisted ERP Engineering

This document is a **reference surface**, not a mandatory stack. Hazim's ERP Engineering Skills are architecture- and vendor-aware, but intentionally not locked to one framework.

The engineering rule is simple:

> Choose technology based on the domain guarantees, operational requirements and team constraints — not because a framework name looks modern.

---

# 1. Core Programming Languages

## TypeScript / JavaScript
Recommended for:
- full-stack web applications,
- API layers,
- event consumers,
- admin applications,
- BFF layers,
- real-time UI workflows.

Common ecosystem:
- Node.js
- NestJS
- Fastify
- Express
- Next.js
- React
- Zod
- Drizzle / Prisma / TypeORM

Key cautions for ERP:
- ORM abstraction must not hide critical locking semantics,
- monetary values must not use JavaScript floating-point arithmetic as financial truth,
- transaction boundaries must be explicit,
- domain validation must not exist only in the frontend.

## Java / Kotlin
Recommended for:
- large transactional backends,
- long-lived enterprise services,
- strongly modeled domain systems,
- complex integrations.

Common ecosystem:
- Spring Boot
- Spring Security
- Hibernate / JPA
- jOOQ
- Flyway / Liquibase

## C# / .NET
Recommended for:
- enterprise applications,
- Microsoft-centered environments,
- strongly typed business applications,
- Windows/AD-heavy customers.

Common ecosystem:
- ASP.NET Core
- Entity Framework Core
- Dapper
- FluentValidation

## Python
Recommended for:
- automation,
- data processing,
- reporting,
- AI/ML services,
- integration workers,
- rapid internal services.

Common ecosystem:
- FastAPI
- Django
- SQLAlchemy
- Pydantic
- Celery

For ledger-critical hot paths, ensure transaction semantics and concurrency behavior are deliberately designed and tested.

## Go
Recommended for:
- infrastructure-facing services,
- high-throughput APIs,
- workers,
- networking-heavy services,
- small operational binaries.

---

# 2. Frontend

Common choices:
- React
- Next.js
- Vue
- Nuxt
- Angular
- Svelte / SvelteKit

ERP frontend requirements commonly include:
- large virtualized data grids,
- keyboard-heavy workflows,
- optimistic edits only where safe,
- conflict detection,
- draft persistence,
- robust validation,
- permissions reflected in UI without relying on UI for security,
- stale-data strategy,
- pagination and server-side filtering,
- accessible forms,
- printable/exportable documents,
- internationalization and RTL where required.

Frontend state is not the accounting or inventory source of truth.

---

# 3. Relational Database

## PostgreSQL
Strong default for new ERP workloads because it provides:
- ACID transactions,
- MVCC,
- row-level locks,
- serializable isolation,
- constraints,
- triggers,
- row-level security,
- JSONB where justified,
- window functions,
- rich indexing,
- logical/physical backup options,
- WAL/PITR ecosystem.

Important primitives:

```sql
BEGIN;
SELECT ... FOR UPDATE;
-- validate invariant
-- write ledger movement
-- write journal entries
-- write outbox record
COMMIT;
```

Do not assume `SELECT FOR UPDATE` automatically solves every race condition. Locks must cover the authoritative rows that actually participate in the invariant.

At `SERIALIZABLE` isolation, applications must be prepared to retry serialization failures safely.

## MySQL / MariaDB
Can support ERP systems when isolation behavior, constraints, locking, migrations, recovery and operational tooling are understood by the team.

## SQL Server
Strong enterprise option, especially in Microsoft ecosystems.

---

# 4. Data Types

## Money
Use exact numeric types.

Examples:

```sql
NUMERIC(20, 6)
DECIMAL(20, 6)
```

Precision and scale depend on business requirements.

Avoid using binary floating point as the authoritative representation for money.

## Quantity
Quantities may require exact decimal precision depending on UOM and industry.

## Time
Store explicit timestamps and business dates separately where semantics differ.

Examples:
- document_date
- posting_date
- event_occurred_at
- recorded_at
- created_at

Do not confuse "when the real-world event occurred" with "when the database received it."

---

# 5. API Protocols

## HTTPS / TLS
All production network traffic carrying credentials, financial data or personal data should use encrypted transport.

Preferred:
- TLS 1.3 where supported
- TLS 1.2 minimum for legacy compatibility policies where necessary

## REST + JSON
Useful for:
- public APIs,
- mobile/web backend APIs,
- partner integrations,
- conventional CRUD/read models.

Document with OpenAPI.

## OpenAPI 3.x
Use for:
- request/response schemas,
- status codes,
- authentication definitions,
- compatibility review,
- generated SDKs where appropriate.

## GraphQL
Useful where client-driven querying materially improves product design.

Audit:
- authorization per resolver/resource,
- query complexity,
- N+1,
- data leakage,
- batching,
- introspection policy.

## gRPC
Useful for internal service-to-service contracts where strongly typed schemas and efficient transport matter.

---

# 6. Authentication & Authorization Protocols

## OAuth 2.0
Use for delegated authorization where appropriate.

## OpenID Connect
Use for identity/authentication on top of OAuth 2.0.

## Sessions
Server-side sessions can be a strong choice for ERP web applications.

Typical controls:
- secure cookies,
- HttpOnly,
- SameSite policy,
- CSRF protection where relevant,
- server-side invalidation,
- device/session visibility where required.

## JWT
Use deliberately.

Consider:
- short lifetime,
- signing-key rotation,
- issuer/audience validation,
- revocation strategy,
- no sensitive mutable authorization assumptions embedded for excessive periods.

---

# 7. Authorization Architecture

Possible layers:

```text
Authentication
    ↓
Tenant / legal entity boundary
    ↓
Role / permission decision
    ↓
Resource ownership or scope
    ↓
Field-level restrictions where required
    ↓
Segregation-of-duties rule
    ↓
Database enforcement for critical boundaries
```

Useful mechanisms:
- RBAC
- ABAC
- PostgreSQL RLS
- policy engines
- explicit application authorization services

RLS does not remove the need to audit bypass roles, raw SQL, security-definer functions, object storage, exports and background jobs.

---

# 8. Idempotency

Business commands exposed to retries should define stable idempotency semantics.

Example conceptual key:

```text
company_id + operation_type + business_document_id + line_or_action_id
```

A strong implementation generally includes:
- durable unique constraint,
- request identity,
- result persistence or terminal status,
- retry-safe behavior,
- timeout-after-commit test.

Do not rely only on in-memory caches for idempotency of financial or inventory operations.

---

# 9. Transactional Outbox

Use when a database transaction must trigger an external effect that cannot participate in the same ACID transaction.

Example:

```text
BEGIN
  update ERP state
  write ledger
  write accounting journal
  insert outbox_event
COMMIT

worker reads outbox_event
  → sends webhook / email / queue message
  → retries safely
  → records delivery status
```

Use for:
- payment integrations,
- shipping,
- WhatsApp/SMS/email,
- ecommerce sync,
- customs,
- external accounting connectors,
- bank integrations.

---

# 10. Messaging

Common technologies:
- Apache Kafka
- RabbitMQ
- NATS
- AWS SQS/SNS
- Google Pub/Sub
- Azure Service Bus

Plan for:
- duplicate delivery,
- ordering,
- poison messages,
- retries,
- dead-letter handling,
- replay,
- schema evolution,
- consumer idempotency,
- observability.

"Exactly once" must not be assumed merely because a broker uses the phrase in one part of its architecture.

---

# 11. Webhooks

Recommended controls:
- HTTPS
- signed payloads
- timestamp
- replay window
- event ID
- idempotent receiver
- exponential retry
- delivery log
- endpoint rotation
- secret rotation

Typical signature approach:

```text
HMAC-SHA256(secret, timestamp + "." + raw_body)
```

Exact protocol should be documented and versioned.

---

# 12. Object Storage

Common systems:
- Amazon S3
- MinIO
- Google Cloud Storage
- Azure Blob Storage
- S3-compatible providers

Controls:
- private buckets by default,
- signed short-lived URLs,
- MIME allowlist,
- file-size limits,
- malware scanning,
- authorization before issuing download URL,
- retention/lifecycle,
- orphan cleanup,
- encryption.

---

# 13. Cache

Redis is useful for:
- sessions,
- rate limiting,
- ephemeral locks when carefully designed,
- derived caches,
- job metadata,
- temporary computation.

Do not make Redis the sole source of financial or inventory truth unless the architecture deliberately provides durable ledger guarantees elsewhere.

For each cache define:
- source of truth,
- TTL,
- invalidation,
- stale tolerance,
- recovery/rebuild behavior.

---

# 14. Search

Possible systems:
- PostgreSQL full-text search
- Elasticsearch
- OpenSearch
- Meilisearch
- Typesense

If search is derived from ERP data:
- canonical DB remains authoritative unless explicitly designed otherwise,
- updates must be replayable,
- drift must be detectable,
- index rebuild must be possible.

---

# 15. Observability

## OpenTelemetry
Use for vendor-neutral tracing, metrics and logs instrumentation where appropriate.

Protocol:
- OTLP/gRPC
- OTLP/HTTP

Recommended correlation dimensions:
- request_id
- trace_id
- tenant_id
- user/service identity where safe
- business_document_id
- operation type
- queue message/event ID

Never put secrets or sensitive financial payloads indiscriminately into logs.

## Metrics
Common stack:
- Prometheus
- Grafana
- cloud-native equivalents

Track both technical and business health.

Examples:
- DB latency
- lock wait
- deadlocks
- queue lag
- API p95/p99
- posting failures
- reconciliation mismatches
- inventory-negative attempts
- outbox backlog
- failed login rate

---

# 16. Containers

## Docker / OCI
Useful for reproducible packaging.

Audit:
- pinned base images,
- minimal runtime image,
- non-root execution,
- no secrets baked into image,
- health checks,
- image scanning,
- deterministic builds where practical.

## Kubernetes
Use when operational scale/availability justifies the complexity.

Do not introduce Kubernetes merely to make a small ERP appear enterprise-grade.

---

# 17. Reverse Proxy / Gateway

Common:
- Caddy
- NGINX
- Envoy
- Traefik
- cloud API gateways

Responsibilities may include:
- TLS termination,
- routing,
- compression,
- rate limiting,
- request-size limits,
- access logging,
- WAF integration.

Authorization still belongs in the application/domain architecture where business context is required.

---

# 18. Infrastructure as Code

Common:
- Terraform
- OpenTofu
- Pulumi
- cloud-native templates

Benefits:
- repeatability,
- reviewable change,
- environment reconstruction,
- disaster recovery,
- drift awareness.

---

# 19. CI/CD

Common:
- GitHub Actions
- GitLab CI
- Azure DevOps
- Jenkins
- CircleCI

Recommended gates:

```text
format/lint
→ typecheck
→ unit tests
→ integration tests
→ migration validation
→ security scans
→ dependency scan
→ secret scan
→ build artifact
→ provenance/signing where required
→ staged deployment
→ smoke/health verification
```

---

# 20. Software Supply Chain

Controls to evaluate:
- lockfiles
- dependency pinning
- Dependabot/Renovate-style updates
- vulnerability scanning
- SBOM
- provenance
- trusted CI runners
- branch protection
- required reviews
- signed releases where required
- restricted production credentials

Relevant standards/concepts to study:
- SLSA
- OpenSSF Scorecard
- NIST SSDF
- CycloneDX
- SPDX

---

# 21. Security Verification

Use multiple approaches:
- code review
- threat modeling
- SAST
- DAST
- dependency scanning
- secret scanning
- configuration review
- negative authorization tests
- penetration testing for appropriate risk tiers

Relevant reference families:
- OWASP ASVS
- OWASP Top 10
- OWASP API Security Top 10
- NIST SSDF

Automated tools do not prove business-logic security.

---

# 22. Testing Layers

## Unit
Pure calculations and local domain logic.

## Integration
Database, queues, object storage, real framework boundaries.

## E2E
Critical user/business workflows.

## Property-based
Invariants across generated sequences.

Example properties:

```text
Every posted journal balances.
Every stock issue decreases exactly the allowed authoritative balance.
Rejected quality stock never becomes allocatable without an allowed transition.
Retrying a business command does not duplicate its effect.
Tenant A cannot access Tenant B through any supported access path.
```

## Concurrency
Exercise real collisions.

## Load
Measure saturation, latency, lock behavior and headroom.

## Failure injection
Kill/restart dependencies around transaction boundaries.

## Restore drill
Actually restore backups and reconcile resulting ledgers.

---

# 23. Database Migration Tooling

Common:
- Flyway
- Liquibase
- Prisma Migrate
- Alembic
- EF Core migrations
- framework migration tools

Critical rules:
- migrations are reviewed,
- destructive changes are deliberate,
- rollback/forward-fix strategy exists,
- schema and application compatibility is planned,
- large-table changes are tested against realistic data volume,
- manual database objects are not silently destroyed by ORM-generated migrations.

---

# 24. ERP Accounting Kernel

Core concepts:
- chart of accounts
- double entry
- journals
- fiscal periods
- document date vs posting date
- reversals
- dimensions/cost centers
- AP/AR subledgers
- GRNI
- inventory valuation
- FX
- tax
- reconciliation

Invariant:

```text
for every posted journal:
SUM(debit) == SUM(credit)
```

This should be protected by design, not merely checked by a dashboard query.

---

# 25. Inventory Costing

Possible valuation methods:
- FIFO
- moving weighted average
- standard cost
- specific identification where appropriate

Operational issue policies may include:
- FEFO
- FIFO
- location priority
- lot restrictions

Operational picking policy and financial valuation policy are separate concepts and may need separate engines.

---

# 26. Manufacturing

Concepts the planning/audit skills recognize:
- BOM
- BOM revisions/effectivity
- routing
- work centers
- material issue
- backflush
- WIP
- partial completion
- scrap
- rework
- co/by-products
- indirect cost
- production variance
- lot/serial traceability
- quality gate

---

# 27. Recovery

A serious ERP should define:
- backup type
- backup frequency
- offsite copy
- encryption
- retention
- RPO
- RTO
- PITR where required
- restore procedure
- restore test cadence
- reconciliation after restore

A backup is an input. A successful restore is evidence.

---

# 28. Recommended Architectural Default for a Small/Medium ERP

This is an example, not a rule:

```text
Web UI:          TypeScript + React/Next.js
Backend:         TypeScript + NestJS OR Java/Spring Boot OR .NET
Database:        PostgreSQL
Cache/session:   Redis
Object storage:  S3-compatible
API:             REST/JSON + OpenAPI
Auth:            OIDC / secure server sessions
Async effects:   Transactional Outbox + worker/queue
Observability:   OpenTelemetry + metrics + structured logs
Packaging:       Docker
CI/CD:           GitHub Actions or equivalent
Backups:         automated DB backups + PITR where required + restore drills
```

For many small businesses a disciplined modular monolith is preferable to premature microservices.

---

# 29. Final Technology Rule

Never let architecture vocabulary substitute for operational proof.

```text
"We use PostgreSQL"  ≠ concurrency is correct
"We use Redis"       ≠ caching is safe
"We use JWT"         ≠ authorization is correct
"We use RLS"         ≠ tenant isolation is complete
"We use Docker"      ≠ deployment is reliable
"We have CI"         ≠ releases are safe
"We have backups"    ≠ recovery works
"We have 1,000 tests"≠ the right invariants are tested
```

The skills exist to force the AI to connect technology choices to the guarantees the business actually needs.
