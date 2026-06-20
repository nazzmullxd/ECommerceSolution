# Packet 02: Modular Monolith Scaling And Compatibility

## Packet Metadata

| Field | Value |
| --- | --- |
| Status | Blocked - PHASE0/PHASE1/PHASE2/PHASE3/PHASE4/PHASE5-GATE |
| Depends On | Packet 01 Approved; Phase 5 performance/observability/deployment evidence; `P6-SCALE-001` |
| Decision Type | Planning first; monolith optimization implementation only after separate packet approval |
| Manual Review | Module/data owners, DB/performance, security/privacy, operations/SRE, API, release and cost review |
| Produces | Monolith scale scorecard/backlog/compatibility plan and `evidence/02-completion.md` |

## Objective

Plan and prioritize measured modular-monolith scaling through query/index tuning, read models, workers, outbox throughput, safe caching, horizontal replicas, feature flags and API compatibility before any extraction decision.

## Read First And Prerequisites

- Packet 01 approved triggers/owners/priorities and Phase 5 load/query/capacity/SLO/telemetry evidence.
- Approved module data/API/event ownership, migration/release/feature-flag/caching/security contracts.

## In Scope

- Bottleneck baseline and target for Catalog, Cart/Checkout, Inventory, Orders/Payment, Admin/Support, Reporting, Notifications/Outbox, Search/AI.
- Query plans, measured indexes, projections/read models, batching/pagination/connection/transaction scope, N+1/materialization controls.
- Worker/outbox lease/batch/concurrency/backoff/dead-letter/consumer idempotency and throughput/lag.
- Safe cache matrix with source/TTL/size/key/invalidation/freshness/stampede/failure/privacy; horizontal statelessness/session/data-protection/shared-state and worker coordination.
- Feature-flag lifecycle/canary/kill/owner/expiry/audit, API additive/version/deprecation compatibility and old/new schema/event coexistence.
- Phased monolith-first experiments, budgets, production measurements and rollback.

## Out Of Scope

- Service extraction, external queue/cache/search/database, paid AWS, speculative index, shared mutable cache truth, breaking API, distributed transaction, multi-region, implementation before evidence/owner review.

## Ownership And Boundaries

- **Owns:** monolith scaling option/experiment/measurement/compatibility/rollback plan.
- **Reads:** module queries/events/workers/APIs and production signals.
- **Must not own/mutate:** source module data/rules, SLO/rate/cost acceptance, migration, infrastructure/cloud or extraction decision.

## Evidence Required

| Evidence | Required Proof |
| --- | --- |
| Entry Evidence | Approved production bottleneck/trigger and owner from Packet 01. |
| Measured Need Evidence | Query plans, latency/throughput/lag/resource/cardinality/lock trend and customer/business impact. |
| Design Evidence | Ranked monolith options, compatibility/security/cost/test/rollout/rollback. |
| Monolith-First Evidence | Before/after local/staging/production measurements and invariant/regression proof. |
| Extraction/Cloud Evidence | Not produced; persistent limitation becomes input to Packet 10 only. |

## Data And Contract Design

Each optimization record includes owner/module/query-or-worker/API, baseline/environment/data size, bottleneck hypothesis, target, design, data/consistency/privacy class, migration/index/cache/event impact, test, rollout flag, abort/rollback, measurements and residual limit.

Read models/indexes/caches are rebuildable and freshness-labelled. Source module remains authoritative. API/event compatibility matrix records producer/consumer/version, additive change, deprecation window and old/new coexistence.

## API And UI Contract

- Prefer additive API changes; breaking change requires approved new version, consumer inventory, migration/deprecation/rollback and tests.
- UI displays read-model/cache freshness/partial/unavailable state where relevant and never presents stale analytics as transaction truth.
- Pagination/filter/sort remain bounded/allowlisted and backward-compatible.

## Likely File Areas To Inspect

- Measured Core/Infrastructure/API/Web/worker/query/migration/tests and Phase 5 performance/telemetry evidence.
- No source changes in this planning packet unless a later explicitly approved monolith implementation prompt is issued.

## Architecture And Prohibited Dependencies

- Add indexes from measured plans only; include write/storage/lock costs.
- Move heavy reads to projections before a warehouse/service; read model events remain idempotent/rebuildable.
- Cache cannot authorize, price, reserve stock, settle payment, determine order truth or store private cross-user response.
- Horizontal replicas require stateless app/shared durable state and lease-safe workers.

## Database Changes And Migrations

No migration now. Every future index/read-model/schema change requires measured evidence, DB/module owner, provider plan, lock/write/storage analysis, expand-contract compatibility, backup/forward-fix and Phase 5 migration gate.

## Security, Logging, Audit, And Failure Rules

- Optimization must preserve authorization/ownership/audit/idempotency/concurrency/privacy and not expose broad direct table access.
- Metrics/logs use bounded safe dimensions; no PII/query parameters/secret payloads.
- Cache/read model/worker failure degrades to fresh source or explicit unavailable/stale state without corrupting transaction truth.
- Flag off/rollback path is tested before rollout; flags cannot hide missing security.

## Planning Or Implementation Steps

1. Confirm Packet 01 trigger and collect exact production plans/telemetry/data sizes/traffic/lag.
2. Reproduce baseline locally/staging with synthetic representative data where possible.
3. Rank query/index/read-model/worker/cache/horizontal/code/API options by impact/risk/cost.
4. Define invariant/security/compatibility/migration/feature-flag/rollback tests for smallest option.
5. Seek monolith implementation approval for one bounded change, not this whole packet.
6. Measure local/staging/production before/after and stop/rollback on regression.
7. Repeat options; only persistent measured limitation goes to Packet 10 scorecard.

## Test And Review Matrix

| Review | Required Cases |
| --- | --- |
| Query/index | Representative plan/data, read/write/lock/storage, pagination and regression. |
| Read model | Replay/rebuild/duplicate/out-of-order/freshness/partial failure and source independence. |
| Worker/outbox | Lease/concurrency/batch/backoff/dead-letter/poison/idempotency/lag recovery. |
| Cache | Hit/miss/expiry/invalidation/stampede/failure/privacy; prohibited truth excluded. |
| Horizontal/flags | Multiple replicas, sessions/data protection, worker coordination, canary/off/expiry/audit. |
| Compatibility | Old/new API/schema/event clients, additive/deprecation/rollback, no security drift. |

## Verification Commands

```powershell
dotnet build ECommerceSolution.slnx
dotnet test ECommerceSolution.slnx --no-build
rg -n "AsNoTracking|Include\(|GroupBy|ToList|Cache|BackgroundService|Outbox|Feature|ApiVersion|Concurrency|Idempotency" ECommerce.Core ECommerce.Infrastructure ECommerce.API ECommerce.Web ECommerce.Tests
git diff --check
git status --short
```

## Acceptance Criteria

- Every scaling proposal starts from measured production pain and tries smallest monolith option first.
- Data/API/event/transaction/auth/privacy compatibility and rollback are explicit.
- Read models/caches/workers remain derived/idempotent/rebuildable and never transaction truth.
- No speculative schema, external service, paid resource, extraction or multi-region instruction is added.
- Persistent residual limits are quantified for Packet 10, not interpreted as automatic extraction.

## Manual Approval Gates

Manual approval is mandatory for query/index/migration, read model/event, cache boundary, worker concurrency/outbox, horizontal state, API version/deprecation, feature flag rollout, production measurement and any handoff to extraction review.

## Stop Conditions

Stop for no measured trigger, speculative index/cache, authorization/transaction bypass, private cache/read model leak, non-idempotent worker, breaking API without plan, unsafe mixed version, external service/cloud request, or optimization result claimed without production evidence.

## Rollback / Forward-Fix And Handoff

Planning-only now. Future optimization rolls out behind approved flag/canary and reverts code/config/index only through migration-safe plan; preserve source truth. Failed change returns to baseline and evidence. Hand only measured persistent limitation to Packet 10; otherwise record `Remain In Monolith`.

## Copy-Ready Constrained Prompt

```text
Execute Phase 6 Packet 02 as planning only after prior gates, Packet 01 and P6-SCALE-001. Inspect measured Phase 5/production evidence and update only Phase 6 monolith scaling/query/read-model/worker/outbox/cache/horizontal/flag/API compatibility plans and evidence. Do not write code/migrations/services/cloud, add speculative indexes/cache, break APIs, or make derived data transaction truth. Require owner, test, migration, phased rollout and rollback for one future bounded monolith change. Quantify persistent limitations for Packet 10; otherwise conclude Remain In Monolith. Add evidence/02-completion.md.
```
