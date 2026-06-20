# Packet 09: Performance, Load, Capacity, And Caching

## Packet Metadata

| Field | Value |
| --- | --- |
| Status | Blocked - PHASE0/PHASE1/PHASE2/PHASE3/PHASE4-GATE |
| Depends On | Packets 01-04 and 06 Design Approved; `P5-SLO-001`, `P5-RATE-001`, `P5-PERF-001` |
| Ownership | Performance owner coordinates; each module/database/provider owner owns bottlenecks and budgets |
| Manual Review | Mandatory workload, rate-limit, payment/webhook, inventory, database, cache, AI, test-data, staging capacity and cost review |
| Produces | Performance budgets/load/capacity/cache plan and `evidence/09-completion.md` |

## Objective

Define reproducible local and staging performance tests for critical journeys, establish measured budgets/capacity margins, identify database/worker/provider bottlenecks, and enforce cache boundaries that never make stale data authoritative for money, stock, permissions or private state.

## Read First And Prerequisites

- Packet 01 traffic/critical-journey/owner inventory, Packet 03 candidate SLIs/SLOs, Packet 04 limits, Packet 06 topology.
- Approved idempotency/concurrency/payment webhook/outbox/indexing/read-model contracts and test-data fixtures.
- Expected business traffic/peaks/data size/geography/client mix and future topology assumptions; mark unknowns explicitly.

## In Scope

- Baseline budgets for catalog list/detail/search, cart/merge, checkout/limited stock, payment webhook, order read, admin dashboard/report, database, Outbox/notification/index worker and enabled AI.
- Workload model with arrival/concurrency/think time/ramp/duration/data cardinality/read-write mix/cold-warm cache/dependency latency/error injection and test environment hardware/tool/version.
- Local smoke/baseline/stress/concurrency/endurance/recovery tests using synthetic data and deterministic mock providers; staging capacity/soak/failure plan.
- Query plans/index/lock/deadlock/pool/thread/GC/memory/CPU/disk/network/worker backlog analysis; capacity estimate and headroom.
- Explicit cache eligibility/key/TTL/size/invalidation/freshness/privacy/failure/fallback and stampede controls.

## Out Of Scope

- Production load test, real payment charge/provider abuse, customer data, denial-of-service, paid testing platform, infrastructure scaling/provisioning, performance code change without owner packet, premature distributed cache/CDN, or final SLO from roadmap guesses.

## Ownership And Operational Contracts

- **Owns:** test scenario/model/dataset/environment/report schema, performance budget/capacity evidence and cache policy matrix.
- **Reads:** approved APIs/business invariants/SLIs/telemetry/topology and query outcomes.
- **Must not own/mutate:** business rules, production state, rates/SLO risk acceptance, schema/index/cache implementation, or cloud capacity.

Every scenario records purpose/owner, route/journey, setup/data size, identities, concurrency/arrival/ramp/duration, dependency/mock state, expected status/invariants, budget/abort threshold, metrics/query capture, cleanup/idempotency, result and bottleneck/action owner.

## Evidence Required By Tier

| Tier | Required Evidence |
| --- | --- |
| Design Evidence | Approved workload/budget/scenario/tool/data/hardware/cache/capacity/error/abort design and owner matrix. |
| Local Verification Evidence | Reproducible synthetic baseline/concurrency/stress/recovery reports with environment/tool/version/raw safe summaries and invariants. |
| Staging Evidence | Production-like dataset/topology/config artifact capacity/soak/failure/cache/rate tests and headroom recommendation. |
| Production Approval Evidence | Expected traffic/capacity/quotas/autoscaling/cache/SLO/rates accepted with monitoring and scale/rollback owner; no production load run required by default. |

## API And UI Contract

- Load clients follow approved API/auth/idempotency/CSRF/signature contracts and bounded test identities.
- UI budgets include server response plus agreed critical render/accessibility behavior, but no synthetic metric is presented as real-user monitoring.
- Test-only webhook signatures/providers are isolated and cannot reach production; duplicate/delayed/invalid callbacks preserve Phase 2 semantics.

## Likely File Areas To Inspect

- API/Web endpoints, Core services, Infrastructure queries/DbContexts/pools/workers/providers/cache abstractions, migrations/indexes, Tests performance fixtures and telemetry.
- Add no load tool/package/script until tool/version/security review during future execution.

## Architecture And Prohibited Dependencies

- Cache only rebuildable read projections such as public Catalog/category/image metadata, approved public policy/FAQ and explicitly stale dashboard summaries.
- Never cache as truth: checkout totals, current price for payment, payment/order settlement status, final inventory/reservations, auth/session/refresh tokens, authorization/permission, private customer/support data, webhook replay, audit requirement.
- Search/report cache failure cannot affect transaction truth. Invalidation uncertainty returns fresh read/stale-labelled safe response or disables cache.
- Performance optimizations remain in owner modules; no cross-module direct table/query shortcut.

## Database Changes And Migrations

No migration in this packet. Index/query/schema changes require measured query plan evidence, owner/database review, migration/lock/storage/write-cost analysis and rerun. Do not create speculative indexes.

## Security, Logging, Audit, And Failure Rules

- Synthetic credentials/data only; rate and test source are allowlisted in isolated environments without disabling server security.
- Abort on data corruption, overselling, duplicate charge/order/effect, signature/auth bypass, environment instability, excessive error/resource use or accidental non-test target.
- Results/logs exclude secrets/tokens/cookies/headers/full PII/payment payload/AI prompt and high-cardinality raw IDs.
- Cache keys exclude secrets and avoid cross-user/tenant leakage; private responses are not shared-cacheable.

## Implementation Steps

1. Resolve expected traffic/data/peaks/regions/tool/hardware/budgets/headroom and test ownership; label candidate values.
2. Define scenario matrix for all required routes/workers/providers with invariants and abort thresholds.
3. Define safe synthetic fixture/setup/cleanup/idempotency/signature and accidental-target protection.
4. Define telemetry/query/lock/resource capture and report comparison to baseline.
5. Define cache policy matrix and prove prohibited truth/private data are excluded.
6. Execute local free-tool baseline/concurrency/stress/recovery only after review; record actual environment/results.
7. Propose owner fixes/indexes/rates/budgets, rerun; define staging soak/capacity/failure evidence and approval.

## Test And Review Matrix

| Scenario | Required Cases |
| --- | --- |
| Catalog/search | List/detail/filter/sort/pagination/keyword/semantic fallback, cold/warm cache and visibility. |
| Cart/checkout | Guest/auth/merge, concurrent last stock, price change, idempotent submit, reservation conflict/expiry. |
| Payment/webhook | Valid/invalid/duplicate/delayed bursts, signature/replay/idempotency, reconciliation; mock only. |
| Admin/database | Dashboard/report/support queries, large data, index/query plan, lock/deadlock/pool/timeout. |
| Workers/AI | Outbox backlog/retry/dead-letter/drain, indexing generation, retrieval/fallback/rate/provider failure. |
| Cache/recovery | Hit/miss/expiry/invalidate/stampede/store failure/stale label; no truth/private/auth cache. |

## Verification Commands

```powershell
dotnet build ECommerceSolution.slnx
dotnet test ECommerceSolution.slnx --no-build
rg -n "IMemoryCache|IDistributedCache|Cache|AsNoTracking|Include\(|ToList|GroupBy|Semaphore|Concurrency|Idempotency" ECommerce.Core ECommerce.Infrastructure ECommerce.API ECommerce.Web ECommerce.Tests
git diff --check
git status --short
```

Load commands are added only after selecting and approving an installed free tool and confirming the isolated target.

## Acceptance Criteria

- Every required journey/worker/provider has reproducible workload, budget, invariants, abort thresholds, telemetry and owner.
- Local results report actual environment/data/tool/version and no commerce/security invariant fails.
- Cache matrix excludes transaction/auth/private truth and covers freshness/invalidation/stampede/failure.
- Any optimization/index/rate/SLO/capacity proposal is evidence-based and manually reviewed; staging/production remain pending.

## Manual Approval Gates

Manual approval is mandatory for test target/data/tool/intensity, auth/rate exceptions, payment/webhook scenarios, limited-stock concurrency, database plans/indexes, cache boundaries, AI/provider tests, staging capacity/soak, final rates/SLOs and production capacity.

## Stop Conditions

Stop for unknown target/owner, production/customer data, real charge/provider abuse, disabled security, missing idempotency/signature, oversell/duplicate effect/data corruption, unsafe load, unbounded test, cache of money/stock/auth/private truth, speculative index, roadmap target claimed measured, or local result promoted to staging/production.

## Rollback / Forward-Fix And Handoff

Abort test and disable test workload/cache experiment on invariant/resource failure; preserve safe evidence and restore isolated synthetic state through approved fixture. Fix in owning module, rerun regression/performance and review. Hand baselines/threshold candidates/capacity risks to Packets 10/12/99.

## Copy-Ready Future Execution Prompt

```text
Execute Phase 5 Packet 09 only after prior gates, Packets 01-04/06, P5-PERF-001/P5-RATE-001, and mandatory module/security review. Define and, only on an approved isolated synthetic target, run bounded local free-tool tests for catalog/cart/checkout/mock webhook/dashboard/DB/workers/AI plus safe cache behavior. Do not target production, use customer data/real payments, disable auth/signatures/idempotency, add speculative indexes/cache/cloud, or cache money/stock/auth/private truth. Abort on invariant failure, record actual environment/results as local only, leave staging/production pending, and add evidence/09-completion.md.
```
