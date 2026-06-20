# Packet 08: Integration Event Catalog And Outbox Evolution

## Packet Metadata

| Field | Value |
| --- | --- |
| Status | Blocked - PHASE0/PHASE1/PHASE2/PHASE3/PHASE4/PHASE5-GATE |
| Depends On | Packets 01-02 and applicable 03-07 Approved; stable Outbox; `P6-EVENT-001` |
| Decision Type | Planning first; local outbox/event contracts before external broker decision |
| Manual Review | Domain/data owners, distributed-systems, security/privacy, Outbox/worker, API compatibility, operations/SRE, DB, cost and consumer owners |
| Produces | Versioned event catalog/compatibility/replay and broker decision plan, `evidence/08-completion.md` |

## Objective

Govern versioned integration facts with explicit ownership/schema/idempotency/ordering/retry/duplicate/poison/compatibility rules, preserve the database outbox, and define evidence required before any SQS/EventBridge adapter decision.

## Read First And Prerequisites

- Approved domain/outbox/event/idempotency/correlation/privacy/retention and worker/dead-letter/incident evidence.
- Packet 01 owners/triggers and event needs from Promotions/Loyalty/Warehouse/Reporting/AI.

## In Scope

- Event envelope and catalog for Catalog/Product, Inventory reserve/release/transfer, Order placed/status, Payment initiated/confirmed/failed, Shipment/Refund future, Review, Promotion, Loyalty, Reporting and AI knowledge/index outcomes.
- Producer meaning/transaction/timing, schema/version/additive compatibility/deprecation, consumers/purpose/data class/retention.
- Event/aggregate IDs/version, occurred/recorded time, correlation/causation, idempotency/replay key, duplicate/out-of-order/gap/poison/retry/backoff/dead-letter/reconciliation.
- Outbox lease/publish attempt/provider ID, consumer inbox/receipt if needed, replay/backfill controls, local contract tests and outbox-to-SQS/EventBridge decision criteria.

## Out Of Scope

- SQS/EventBridge resource/SDK, broker implementation, event-driven replacement of local transactions, event sourcing, global ordering, exactly-once claim, distributed saga framework, service extraction, raw entity/PII/payment/AI payload.

## Ownership And Boundaries

- **Owns:** producing module owns event name/meaning/schema/version/publication; Integration catalog owns governance metadata/compatibility.
- **Reads:** committed business state through outbox payload creation and consumer needs.
- **Must not own/mutate:** source business state, consumer side effects, module commands, broker/cloud or transaction boundaries.

## Evidence Required

| Evidence | Required Proof |
| --- | --- |
| Entry Evidence | Stable transactional Outbox/worker/idempotent consumers/monitoring/runbooks. |
| Measured Need Evidence | Consumer count/throughput/latency/integration/isolation/backlog need beyond local handlers. |
| Design Evidence | Envelope/catalog/schemas/compatibility/privacy/retry/replay/operations/cost/migration/fallback. |
| Monolith-First Evidence | Local publisher/consumer contract/replay/duplicate/order/load and production outbox measurements. |
| Extraction/Cloud Evidence | Separate SQS/EventBridge ADR/service-introduction gate only if triggers pass. |

## Data And Contract Design

Envelope: `eventId`, `eventType` (`Domain.Concept.PastAction.vN`), schema version, aggregate type/ID/version, occurred/recorded UTC, producer/release, correlation/causation, idempotency/replay key, data classification and payload. Avoid generic event with ambiguous payload.

Payload is minimal immutable fact/snapshot/reference, never command, secret/card data/token/raw private prompt/unnecessary PII. Consumer contract states accepted versions, ordering key, idempotent effect key, retryable/terminal errors, max age and reconciliation.

## API And UI Contract

No public event management API. Admin dead-letter/replay, if later approved, is bounded by event type/date/ID, permission/two-person approval/reason/dry-run/rate/audit and cannot edit payload. UI shows safe metadata only.

## Likely File Areas To Inspect

- Core domain/integration events and ownership, Infrastructure Outbox/worker/handlers, module projections, API admin operations, Tests contract/replay/failure.
- Planning docs only now.

## Architecture And Prohibited Dependencies

- Business write and outbox insert share local transaction where possible. External broker never replaces outbox.
- Delivery is at-least-once; consumers are idempotent. Do not claim exactly-once.
- Ordering is scoped to one aggregate/key; no global order.
- Strong checkout/payment/inventory consistency stays local; events notify derived/integration work after commit.

## Database Changes And Migrations

No migration now. Future envelope/outbox/inbox/receipt fields require retention/index/lease/concurrency/storage/cleanup/query-plan/provider migration and Phase 5 review. Do not add an inbox without demonstrated consumer need.

## Security, Logging, Audit, And Failure Rules

- Schema/data classification and consumer authorization minimize payload; encrypt/protect transport later without assuming broker security is enough.
- Log IDs/type/version/aggregate/retry/status/duration/correlation, not full sensitive payload.
- Audit manual dead-letter/replay/drop/schema/route changes.
- Duplicate/out-of-order/gap/poison/expired/unknown version/publish uncertain failure enters safe idempotent retry/dead-letter/reconciliation; no blind replay.

## Planning Or Implementation Steps

1. Inventory events/owners/consumers/purpose/consistency/data class and delete unjustified events.
2. Approve envelope/naming/version/compatibility/deprecation and minimal payload schemas.
3. Define producer transaction/outbox and consumer idempotency/order/retry/poison/reconciliation.
4. Define contract fixtures/schema compatibility/replay/load/privacy and operational runbooks/alerts.
5. Measure local outbox throughput/lag/consumer count/failure and optimize monolith worker first.
6. Score local vs SQS vs EventBridge purpose/cost/quotas/operations/security/migration/fallback in Packet 13.
7. Approve catalog/local contracts; require separate ADR before any broker adapter implementation.

## Test And Review Matrix

| Test | Required Cases |
| --- | --- |
| Contract | Producer/consumer golden schema, additive compatibility, unknown/new/old version, deprecation. |
| Delivery | Duplicate, out-of-order, gap, delayed, expired, poison, crash before/after effect/ack. |
| Idempotency | Repeated event/replay/consumer receipt, same ID changed payload rejected. |
| Ordering | Aggregate versions, stale event, parallel aggregates, no global assumption. |
| Privacy/security | Minimal payload, unauthorized consumer/admin replay, no secret/card/PII/prompt. |
| Migration/operations | Local fallback, broker unavailable/uncertain publish, backlog/dead-letter/reconciliation/rollback. |

## Verification Commands

```powershell
rg -n "OutboxMessage|EventType|AggregateId|Correlation|Causation|Idempotency|Retry|DeadLetter|IntegrationEvent" docs/roadmap docs/implementation-guides/phase-2 docs/implementation-guides/phase-6
git diff --check
git status --short
```

## Acceptance Criteria

- Every event has producer-owned past-tense meaning, versioned minimal schema, consumers, privacy/retention and compatibility.
- At-least-once/idempotency/order/retry/poison/replay/reconciliation behavior is explicit and tested in plan.
- Strong transaction flows remain local and Outbox remains durable publish intent.
- SQS/EventBridge require measured trigger and separate ADR; no broker/service/cloud code now.

## Manual Approval Gates

Manual approval is mandatory for event meaning/schema/version/deprecation, payload data/privacy/retention, producer transaction, consumer idempotency/order, replay/dead-letter, Outbox/inbox migration, broker choice/cost/quotas/security/operations and service extraction.

## Stop Conditions

Stop for event as command, raw entity/private/secret/card/prompt payload, exactly-once/global-order claim, dual write, broker replacing outbox/local transaction, non-idempotent consumer, blind replay, unknown owner/consumer, SQS/EventBridge/service instruction without ADR or implementation without approval.

## Rollback / Forward-Fix And Handoff

Planning-only. Future event versions are additive and old consumers remain during migration; stop publisher route/consumer and use local handler/outbox backlog on defect. Never mutate published event; issue corrected version/fact and reconcile. Hand broker decision to Packets 10/13, retention to 12 and risks to 14.

## Copy-Ready Constrained Prompt

```text
Execute Phase 6 Packet 08 as planning only after prior gates, Packets 01-02/applicable feature plans and P6-EVENT-001. Update only the versioned event catalog/envelope/schema/ownership/idempotency/ordering/retry/duplicate/poison/replay/privacy/Outbox/migration/fallback design and evidence. Do not write event/broker code, use commands/raw entities/private payloads, claim exactly-once/global order, replace local transactions/outbox, or add SQS/EventBridge/services/cloud. Require measured need and separate ADR for any broker. Add evidence/08-completion.md with producer/consumer/security review.
```
