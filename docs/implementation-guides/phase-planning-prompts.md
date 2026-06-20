# Phase Implementation-Planning Prompts

## Purpose

Use these prompts one at a time, in phase order. Each prompt asks Codex or Claude to convert an approved roadmap phase into a modular instruction package. The generated package is documentation for later implementation; it must not contain application or deployment changes.

Do not run all seven prompts at once. Complete, review, and approve one phase instruction package before generating the next.

## Shared Output Standard

Every generated phase package must:

- Live under `docs/implementation-guides/phase-N/`.
- Include `README.md` as the phase index, dependency graph, execution order, progress checklist, and approval gate.
- Use numbered work-packet files such as `01-foundation.md`, `02-database.md`, and `99-phase-acceptance.md`.
- Keep one work packet small enough for one focused AI coding session and one reviewable pull request.
- State objective, prerequisites, source documents, dependencies, in-scope work, out-of-scope work, architecture rules, data/API/UI contracts, security rules, logging/audit rules, failure behavior, exact implementation steps, expected file areas, tests, commands, acceptance criteria, manual-review points, rollback notes, evidence, and handoff.
- End every work packet with a copy-ready execution prompt for Codex/Claude.
- Include a requirements-to-work-packet traceability matrix so no roadmap requirement is lost.
- Record unresolved decisions in a decision register. Use an approved roadmap default when one exists; otherwise mark the packet blocked instead of inventing a high-risk decision.
- Preserve .NET 10, ASP.NET Core on .NET 10, EF Core compatible with .NET 10, C#, Onion Architecture, modular-monolith-first design, Problem Details errors, correlation IDs, idempotency rules, privacy-safe logging, and local/free-first tooling.
- Tell the future coding agent to inspect the repository before naming exact files, package versions, commands, or migrations. Do not fabricate repository state.
- Require tests to be added with each behavior, not postponed until the end.
- Require `git diff`, build, test, and scope verification before a packet is marked complete.
- Never include secrets, credentials, paid-service provisioning, unrelated refactors, premature microservices, or instructions that bypass authorization, ownership, validation, audit, or security controls.

## Prompt 1: Phase 0 Discovery And System-Design Instructions

```text
Act as a senior AWS Solutions Architect and senior .NET 10 architect. Create a detailed, beginner-friendly Phase 0 instruction package. This is documentation work only: do not write application code, migrations, deployment code, or cloud resources.

Read completely before editing:
- ECommerce Platform Requirements Roadmap.md
- docs/roadmap/phase-0.md
- docs/architecture/system-design.md
- docs/architecture/security-scalability-review.md
- docs/ai-rag/rag-architecture.md
- docs/implementation-guides/feature-module-template.md
- docs/implementation-guides/ai-assisted-development-playbook.md
- docs/implementation-guides/phase-planning-prompts.md

Create `docs/implementation-guides/phase-0/`. Follow the Shared Output Standard in `phase-planning-prompts.md`.

Split Phase 0 into ordered work packets covering at minimum:
1. Repository and stakeholder discovery, business goals, personas, assumptions, glossary, constraints, and success metrics.
2. Domain decomposition, bounded contexts, module ownership, dependency rules, commands/queries/events, and data-classification boundaries.
3. C4 Context, Container, and Component diagrams, runtime/deployment views, trust boundaries, and diagram explanations.
4. Canonical API conventions, endpoint inventory, Problem Details, pagination/filtering/sorting, correlation IDs, idempotency, and compatibility/versioning policy.
5. Conceptual MVP data model, entity ownership, relationships, lifecycle/status rules, constraints, indexes, concurrency, snapshots, retention, and migration assumptions.
6. Authentication, session/token, authorization, permission, ownership, admin, audit, and bootstrap design.
7. OWASP threat model for auth, admin, file upload, checkout, payment, webhooks, AI/RAG, and sensitive logging, with mitigations and verification criteria.
8. Local/free-first development architecture, configuration/secrets, logging, monitoring, backup simulation, failure handling, and future AWS mapping without provisioning.
9. Architecture decision records, open-question resolution, cross-phase contract register, Phase 1 entry criteria, and final design approval review.

For every packet, define the exact design artifacts to create or update, questions to answer, Mermaid diagrams required, review checklist, pass/fail acceptance criteria, and evidence needed for approval. Phase 0 packets produce decisions and contracts, not code. Include a final traceability matrix mapping every Phase 0 roadmap section and approval item to a packet and artifact.

Update only Phase 0 implementation-guide documentation. Report created/changed files, assumptions, blocked decisions, risks, and the recommended first packet to execute.
```

## Prompt 2: Phase 1 MVP Foundation Instructions

```text
Act as a senior .NET 10 architect, application-security reviewer, and AWS Solutions Architect. Convert the approved Phase 1 roadmap into a novice-friendly, AI-executable implementation instruction package. Create documentation only; do not implement code yet.

Read completely before editing:
- ECommerce Platform Requirements Roadmap.md
- docs/roadmap/phase-0.md
- docs/roadmap/phase-1-mvp-foundation.md
- docs/architecture/system-design.md
- docs/architecture/security-scalability-review.md
- docs/implementation-guides/feature-module-template.md
- docs/implementation-guides/ai-assisted-development-playbook.md
- docs/implementation-guides/phase-0/README.md and all approved Phase 0 artifacts
- docs/implementation-guides/phase-planning-prompts.md

Create `docs/implementation-guides/phase-1/`. Follow the Shared Output Standard in `phase-planning-prompts.md` and treat approved Phase 0 contracts as authoritative.

Create ordered, independently reviewable work packets for:
1. Solution baseline audit, .NET 10 compatibility, project responsibilities, dependency tests, conventions, and local setup prerequisites.
2. Configuration, environment validation, user-secrets, database-provider decision, DbContext boundary, migration workflow, and seed-data rules.
3. Foundation result/error models, Problem Details, validation pipeline, exception handling, correlation IDs, and privacy-safe structured logging.
4. Identity/access domain entities and EF Core mappings: ApplicationUser, CustomerProfile, Role, Permission, UserRole, RolePermission, RefreshToken, and AuditLog.
5. ASP.NET Core Identity configuration, password policy, registration, login safety, account state, lockout, and user-enumeration protection.
6. Access-token service, refresh-token hashing/rotation/reuse detection, logout, revocation, expiry, and session failure behavior.
7. RBAC and permission policies, customer ownership rules, admin restrictions, super-admin bootstrap, self-escalation prevention, and negative authorization tests.
8. Audit service and mandatory-audit transactions for sensitive actions.
9. Auth, profile, admin-user, role, and permission API vertical slices with request/response contracts.
10. Integration-test infrastructure, security tests, API tests, migration verification, demo script, and Phase 1 acceptance gate.

Each packet must identify predecessor packets, expected project/layer ownership, likely file areas to inspect, prohibited dependencies, database changes, API contracts, test matrix, security stop conditions, and a copy-ready coding prompt. Do not choose exact NuGet versions without checking the installed SDK and official compatibility during that future packet. Do not add cart, order, payment, inventory, or AI implementation.

Update only Phase 1 implementation-guide documentation. Report created/changed files, unresolved Phase 1 decisions, security review points, and execution order.
```

## Prompt 3: Phase 2 MVP Commerce-Flow Instructions

```text
Act as a senior commerce architect, payment-security reviewer, AWS Solutions Architect, and .NET 10 architect. Convert Phase 2 into a detailed, dependency-safe implementation instruction package. Create documentation only; do not write application code, migrations, or payment integrations yet.

Read completely before editing:
- ECommerce Platform Requirements Roadmap.md
- docs/roadmap/phase-0.md
- docs/roadmap/phase-1-mvp-foundation.md
- docs/roadmap/phase-2-mvp-commerce-flow.md
- docs/architecture/system-design.md
- docs/architecture/security-scalability-review.md
- docs/implementation-guides/feature-module-template.md
- docs/implementation-guides/ai-assisted-development-playbook.md
- approved `docs/implementation-guides/phase-0/` and `phase-1/` indexes/contracts
- docs/implementation-guides/phase-planning-prompts.md

Create `docs/implementation-guides/phase-2/` using the Shared Output Standard. Preserve all Phase 1 security and API contracts.

Create small ordered work packets for:
1. Commerce statuses, value objects, money rules, error/result contracts, event contracts, and state-transition tests.
2. Public catalog browse/detail read contracts with visibility, filtering, sorting, pagination, and query-performance rules.
3. Cart and CartItem domain rules, persistence, authenticated ownership, concurrency, expiry, APIs, and tests.
4. Guest-cart identity, abuse controls, secure key/cookie design, login merge algorithm, idempotency, and conflict tests.
5. InventoryItem, InventoryTransaction, StockReservation, optimistic concurrency/transactions, expiry worker, release/confirm behavior, and overselling tests.
6. CheckoutSession and AddressSnapshot, server-side price/discount/shipping/tax recalculation, cart validation, reservation coordination, and idempotency.
7. Order, OrderItem snapshots, OrderStatusHistory, pending-order creation, legal transitions, ownership, and history APIs.
8. Payment abstraction and deterministic mock/sandbox adapter, hosted-payment assumptions, initiation, reconciliation, and no-card-data controls.
9. Signed webhook validation, replay window, provider-event uniqueness, duplicate callbacks, delayed callbacks, amount/currency verification, and audit evidence.
10. Transactional outbox, local worker, retries/backoff, deduplication, dead-letter behavior, notification handoff, and future SQS/EventBridge boundary.
11. Compensation and recovery flows for payment/order/inventory/outbox partial failures.
12. End-to-end cart-to-order tests, concurrency tests, security tests, failure injection, demo, and Phase 2 acceptance gate.

For high-risk packets, require explicit human approval before coding and after tests. Never trust client totals, discounts, stock, shipping, or tax. Never store card data. Define transaction boundaries and idempotency ownership precisely. End each packet with a constrained execution prompt and rollback/forward-fix notes.

Update only Phase 2 implementation-guide documentation. Report created/changed files, blocking commerce decisions, high-risk review gates, and execution order.
```

## Prompt 4: Phase 3 MVP Experience-Layer Instructions

```text
Act as a senior .NET 10 application architect, UX-aware commerce engineer, security reviewer, and AWS Solutions Architect. Convert Phase 3 into modular implementation instructions. Create documentation only; do not implement code or provision services.

Read completely before editing:
- ECommerce Platform Requirements Roadmap.md
- docs/roadmap/phase-0.md through docs/roadmap/phase-3-mvp-experience-layer.md
- docs/architecture/system-design.md
- docs/architecture/security-scalability-review.md
- docs/implementation-guides/feature-module-template.md
- docs/implementation-guides/ai-assisted-development-playbook.md
- approved instruction indexes/contracts for Phases 0-2
- docs/implementation-guides/phase-planning-prompts.md

Create `docs/implementation-guides/phase-3/` using the Shared Output Standard. Do not weaken Phase 1 identity or Phase 2 order/inventory/payment rules.

Create ordered work packets for:
1. Phase 3 statuses, permissions, API/UI conventions, ownership matrix, and cross-module read contracts.
2. ProductVariant, VariantOption, ProductAttribute, variant selection, price adjustment, variant inventory linkage, availability, and product-detail behavior.
3. ProductImage metadata, storage abstraction, local adapter, allowlisted formats, content validation, limits, safe names/paths, lifecycle, deletion, and upload tests.
4. Authenticated wishlist, duplicate prevention, ownership/privacy, unavailable products, APIs/UI, and tests; keep guest wishlist out of scope.
5. Review/Rating creation, verified-buyer rule, moderation lifecycle, edits/deletes, aggregate calculation, spam/abuse controls, APIs/UI, and tests.
6. SupportTicket/TicketMessage lifecycle, assignment, customer/support/admin visibility, internal notes, escalation, sensitive-content handling, APIs/UI, and IDOR tests.
7. Notification model, recipient ownership, in-app delivery, outbox integration, retries, mock email boundary, and failure behavior.
8. Admin order-management screens/services that reuse Phase 2 transitions, permissions, audit, concurrency, and negative tests.
9. Dashboard/basic-reporting query services, approved metrics, stale/partial results, privacy-safe aggregation, performance budgets, and no transactional truth dependency.
10. Product search/filter/sort UX, allowlists, empty states, pagination, and Phase 4 semantic-search handoff.
11. Integrated customer/admin/support UI flows, accessibility checks, security regression suite, demo, and Phase 3 acceptance gate.

Every packet must distinguish data ownership from read-only consumption and specify API plus UI acceptance states. Require human review for uploads, support visibility, moderation, privileged order changes, and reporting queries. End every packet with a copy-ready constrained coding prompt.

Update only Phase 3 implementation-guide documentation. Report created/changed files, unresolved UX/business rules, security review gates, and execution order.
```

## Prompt 5: Phase 4 AI-Enhanced MVP Instructions

```text
Act as a senior AI/RAG architect, application-security specialist, AWS Solutions Architect, and .NET 10 architect. Convert Phase 4 into a safe, local/free-first, AI-executable instruction package. Create documentation only; do not connect paid AI/cloud services or write application code.

Read completely before editing:
- ECommerce Platform Requirements Roadmap.md
- docs/roadmap/phase-0.md through docs/roadmap/phase-4-ai-enhanced-mvp.md
- docs/ai-rag/rag-architecture.md
- docs/architecture/system-design.md
- docs/architecture/security-scalability-review.md
- docs/implementation-guides/feature-module-template.md
- docs/implementation-guides/ai-assisted-development-playbook.md
- approved instruction indexes/contracts for Phases 0-3
- docs/implementation-guides/phase-planning-prompts.md

Create `docs/implementation-guides/phase-4/` using the Shared Output Standard. Mock providers and deterministic local behavior come first; Bedrock/OpenSearch remain future adapters only.

Create ordered work packets for:
1. AI/RAG contracts, provider abstractions, data classifications, visibility matrix, approved/excluded use cases, and threat model.
2. SearchDocument, EmbeddingRecord, KnowledgeArticle/Version, RetrievalLog, Conversation/Message/Feedback entities, constraints, retention, and EF mapping plan.
3. Deterministic mock completion/embedding/vector providers and keyword-search fallback with contract tests.
4. Knowledge article draft-review-approve-publish-archive-delete lifecycle, authorization, poisoning defenses, audit, and stale-content handling.
5. Product/knowledge indexing pipeline, chunking/metadata rules, source IDs, hashes, reindexing, partial failure, deduplication, and local worker.
6. Authorization-aware retrieval, pre-retrieval filters, confidence calculation, fallback policy, citations, prompt construction, and retrieval logs.
7. Hybrid semantic product search, ranking, hidden/unavailable product exclusion, rate limits, APIs, and evaluation tests.
8. Similar-product recommendations using product similarity only, exclusions, fallback, APIs, and tests.
9. Customer support assistant using approved public sources, unsupported-topic refusal, ticket escalation, anonymous restrictions, and source-grounding tests.
10. Admin knowledge assistant with role-scoped sources, privileged audit, cross-role leakage tests, and no customer/payment data indexing.
11. Feedback, evaluation dataset, human-review workflow, grounding/hallucination/prompt-injection/private-data metrics, release thresholds, and incident disable switch.
12. Provider/vector failure handling, abuse/rate limiting, observability, cost-estimate fields, future Bedrock/OpenSearch adapter contracts, and Phase 4 acceptance gate.

Require adversarial tests in every retrieval/assistant packet. The assistant must not answer sensitive policy, refund, payment, legal, or account-security questions from model memory. If approved evidence is absent or confidence is low, it must fallback. Require manual security approval before customer-facing enablement.

Update only Phase 4 implementation-guide documentation. Report created/changed files, unresolved safety thresholds, blocked provider decisions, evaluation gates, and execution order.
```

## Prompt 6: Phase 5 Production-Readiness Instructions

```text
Act as a senior AWS Solutions Architect, SRE, DevSecOps architect, incident commander, and .NET 10 architect. Convert Phase 5 into a detailed production-readiness instruction package. Create planning and implementation instructions only; do not write workflows/deployment code, provision AWS resources, or add secrets.

Read completely before editing:
- ECommerce Platform Requirements Roadmap.md
- docs/roadmap/phase-0.md through docs/roadmap/phase-5-production-readiness.md
- docs/architecture/system-design.md
- docs/architecture/security-scalability-review.md
- docs/ai-rag/rag-architecture.md
- docs/implementation-guides/feature-module-template.md
- docs/implementation-guides/ai-assisted-development-playbook.md
- approved instruction indexes/contracts for Phases 0-4
- docs/implementation-guides/phase-planning-prompts.md

Create `docs/implementation-guides/phase-5/` using the Shared Output Standard. Separate local verification, staging readiness, and future AWS execution. No production action may be implied complete merely because documentation exists.

Create ordered work packets for:
1. Production gap assessment, workload inventory, owners, environment matrix, configuration/secrets classification, and launch criteria.
2. Health/readiness/liveness checks and dependency degradation behavior.
3. Structured logging, correlation propagation, privacy redaction, metrics, traces, audit review, retention, dashboards, and service-level indicators/objectives.
4. HTTPS, headers, CORS, CSRF, auth/session/admin/MFA review, rate limits, file-upload hardening, dependency/static scans, and OWASP approval gates.
5. CI/CD design: branch policy, build, tests, scans, artifacts, migration review, approvals, smoke tests, stop rules, and rollback; no pipeline code yet.
6. Deployment topology for Web/API/worker, safe EF migrations, media, feature flags, rolling/blue-green options, rollback versus forward-fix, and future ECS/RDS/S3/CloudFront/WAF mapping.
7. Backup inventory, frequency/retention, RTO/RPO, local backup/restore drill, restore evidence, accidental deletion, and future AWS backup design.
8. Incident response roles and complete runbooks for application, database, checkout, payment callback, outbox, auth attack, admin compromise, upload abuse, and unsafe AI answers.
9. Performance budgets and local load-test plans for catalog, cart, checkout, webhook, dashboard, database, worker, and AI; include safe caching boundaries.
10. Monitoring/alert catalog, thresholds, ownership, severity, notification/escalation, alert testing, and false-positive review.
11. PII minimization/masking/retention/deletion, production access review, audit protection, secrets rotation, and compliance evidence.
12. AWS cost estimate worksheet, quotas, budgets/anomaly alerts, log/storage/AI controls, launch readiness review, go/no-go checklist, and Phase 5 acceptance gate.

Each packet must distinguish design evidence, local test evidence, staging evidence, and production approval evidence. Require manual approval for CI/CD, migrations, secrets, IAM, rate limits, payment behavior, restore drills, incident procedures, and go-live. End every packet with a constrained future execution prompt and rollback/stop conditions.

Update only Phase 5 implementation-guide documentation. Report created/changed files, unresolved operational ownership, production blockers, required drills, and execution order.
```

## Prompt 7: Phase 6 Scale And Enterprise Instructions

```text
Act as a principal enterprise architect, AWS Solutions Architect, distributed-systems reviewer, security architect, and .NET 10 architect. Convert Phase 6 into a modular, evidence-driven enterprise evolution instruction package. Create documentation only. Keep the modular monolith as the default and do not implement microservices, multi-region writes, or paid services.

Read completely before editing:
- ECommerce Platform Requirements Roadmap.md
- docs/roadmap/phase-0.md through docs/roadmap/phase-6-scale-enterprise.md
- docs/architecture/system-design.md
- docs/architecture/security-scalability-review.md
- docs/ai-rag/rag-architecture.md
- docs/implementation-guides/feature-module-template.md
- docs/implementation-guides/ai-assisted-development-playbook.md
- approved instruction indexes/contracts for Phases 0-5
- docs/implementation-guides/phase-planning-prompts.md

Create `docs/implementation-guides/phase-6/` using the Shared Output Standard. Every expansion must require stable MVP/Phase 5 controls, measured need, owner, cost review, migration plan, and rollback path.

Create ordered planning-first work packets for:
1. Enterprise-entry assessment, measurable scale triggers, ownership/capacity model, feature prioritization, and anti-overengineering gate.
2. Modular-monolith scaling: indexes, query plans, read models, workers, outbox throughput, safe caching, horizontal scaling, feature flags, and API compatibility.
3. Promotions/campaign/coupon module with server-side calculation, precedence/stacking, usage limits, expiry, abuse controls, approval, audit, refund effects, and tests.
4. Loyalty ledger, earning/redemption/expiry/reversal, idempotency, reconciliation, audit, fraud controls, and refund integration.
5. Multi-warehouse stock, allocation, reservations, transfers, fulfillment, concurrency, eventual-consistency boundaries, overselling prevention, and recovery.
6. Reporting/analytics read models, projections, refresh/freshness, privacy-safe metrics/exports, transactional-load protection, retention, and future warehouse criteria.
7. Enterprise RBAC/permissions, least privilege, separation of duties, sensitive-action approval, access reviews, emergency access, audit, and permission-sprawl controls.
8. Versioned integration-event catalog, ownership, schemas, idempotency, ordering, retries, duplicates, poison messages, compatibility, and outbox-to-SQS/EventBridge migration decision.
9. AI/RAG scale, index lifecycle, evaluation operations, prompt-injection monitoring, cost/rate limits, provider resilience, and criteria for separate-service consideration.
10. Module-extraction scorecard applied separately to Catalog, Inventory, Cart, Order, Payment, Search, AI/RAG, Reporting, and Notifications, with data/API/event boundary, risks, strangler migration, fallback, and cost.
11. Single-region/Multi-AZ first, replicas/CDN/backups, active-passive DR criteria, data residency, and explicit rejection criteria for premature active-active multi-region.
12. Data retention/compliance matrix, secure deletion, legal holds, PII minimization, audit/payment/AI log retention, access review, and evidence ownership.
13. Future AWS service mapping with introduction triggers, local equivalent, estimate, quota, security/operations burden, and approval gate.
14. Enterprise risk register, architecture review board checklist, phased rollout/canary/feature-flag strategy, rollback, and Phase 6 acceptance gate.

For every possible service extraction, default to "remain in the monolith" unless objective evidence satisfies the approved extraction criteria. Do not create service implementation instructions until a separate architecture decision is approved. End each packet with a constrained planning or implementation prompt appropriate to its approval state.

Update only Phase 6 implementation-guide documentation. Report created/changed files, rejected overengineering proposals, unresolved enterprise decisions, cost/security gates, and execution order.
```

## Recommended Use Order

1. Run Prompt 1 and review the generated Phase 0 package.
2. Resolve or explicitly accept Phase 0 defaults.
3. Approve Phase 0 before running Prompt 2.
4. Continue one phase at a time; never let a later prompt silently override an approved earlier contract.
5. During implementation, run only the execution prompt from the next approved work packet.
