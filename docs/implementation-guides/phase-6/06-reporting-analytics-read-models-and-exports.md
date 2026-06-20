# Packet 06: Reporting, Analytics Read Models, And Exports

## Packet Metadata

| Field | Value |
| --- | --- |
| Status | Blocked - PHASE0/PHASE1/PHASE2/PHASE3/PHASE4/PHASE5-GATE |
| Depends On | Packets 01-02 Approved; source/event/access contracts; `P6-REPORT-001` |
| Decision Type | Planning first; monolith read models before warehouse/service |
| Manual Review | Business metrics, finance, privacy/security, DB/performance, module/event owners, exports/access, retention, operations and cost review |
| Produces | Reporting/read-model/export/data-warehouse trigger plan and `evidence/06-completion.md` |

## Objective

Design privacy-safe, rebuildable reporting projections that protect transactional workloads, expose explicit freshness/quality, control exports and define objective criteria for a future analytics store only after measured harm.

## Read First And Prerequisites

- Approved source data/event semantics, Phase 3 dashboard/reporting, Phase 5 query/load/privacy/access/retention evidence.
- Packet 01 measured reporting need and Packet 02 monolith scaling outcomes.
- Named metric business owner and data/privacy owner.

## In Scope

- Metric catalog for sales/order/payment/inventory/support/review/promotion/loyalty/warehouse/operations with formula/source/status/currency/timezone/granularity/owner.
- Read models/projections, watermark/source version/idempotency/replay/rebuild/backfill/late/duplicate/out-of-order/correction, refresh schedule/lag/freshness/completeness and reconciliation.
- Query/API/UI bounded filters/group/sort/pagination, privacy-safe aggregation/small-cell rules, permissioned async exports with expiry/download audit.
- Transactional load isolation, indexes/replicas/read-model storage candidates, performance budget, retention/deletion/legal hold and future warehouse criteria/cost/rollback.

## Out Of Scope

- Accounting ledger truth, checkout/payment/inventory decision, unrestricted SQL/query builder, raw customer behavior profiling, production data lake/warehouse, service/cloud, real-time guarantee, client formula or unapproved export.

## Ownership And Boundaries

- **Owns:** derived reporting models, projection checkpoints, metric definitions, freshness/quality and export job metadata.
- **Reads:** approved source events/projections.
- **Must not own/mutate:** transactional entities/status/money/stock/customer identity, source event meaning or audit truth.
- Source modules own corrections; reporting replays/rebuilds.

## Evidence Required

| Evidence | Required Proof |
| --- | --- |
| Entry Evidence | Stable source/event contracts, metric owner, privacy/retention/access and measured reporting demand. |
| Measured Need Evidence | Query load/latency/locks/data volume/export demand and transactional impact. |
| Design Evidence | Metric/projection/freshness/reconciliation/privacy/export/API/test/rollout/rollback. |
| Monolith-First Evidence | Read-model before/after load, correctness/lag/rebuild/security and production measurement. |
| Extraction/Cloud Evidence | Warehouse/service only if thresholds exceeded and Packet 10/13 approvals exist. |

## Data And Contract Design

Metric definition includes ID/version/name/purpose/owner, numerator/denominator, source events/statuses, currency/timezone, dimensions/filters, aggregation, late/correction rule, privacy threshold, refresh target and test fixture.

Projection record carries source aggregate/version/event ID, calculation version, watermark, refreshedAtUtc, status/current-stale-failed, quality/reconciliation state and no unnecessary PII. Export stores query/version/requester/purpose/status/count/checksum/expiry/access audit, not permanent unrestricted copies.

## API And UI Contract

- Admin/report APIs require explicit permissions/purpose, bounded date/rows/dimensions, allowlists, async export for large work, rate/concurrency and audit.
- Responses show `asOfUtc`, freshness/partial/quality/currency/timezone and safe empty/unavailable state; failures never become zero.
- Export links are short-lived/authorized, non-guessable, rechecked at download and never public/cacheable.

## Likely File Areas To Inspect

- Source event/read contracts, Reporting Core/Infrastructure queries/projectors, API/Web dashboard/export, audit/storage/outbox and tests.
- Planning docs only now.

## Architecture And Prohibited Dependencies

- Read model is derived and rebuildable; no transaction path calls Reporting for truth.
- Start in monolith DB/schema/process with workload isolation and bounded workers; consider replica/store only after evidence.
- No cross-module table ownership or huge synchronous export.
- Cache/report freshness is explicit; private exports are not shared cache.

## Database Changes And Migrations

No migration now. Future read-model/checkpoint/export schema requires owner/event version/idempotency/index/partition/retention/query-plan/provider/backup and Phase 5 migration review. Source schema is not altered for report convenience without owner evidence.

## Security, Logging, Audit, And Failure Rules

- Enforce least privilege, small-cell suppression/aggregation, field allowlists, purpose, row/date limits, rate and export expiry.
- Log report/metric/version/requester ID/query categories/row count/status/duration/correlation; not raw filters containing PII or exported data.
- Audit report/export access/download/delete/permission/formula changes.
- Projection lag/failure/duplicate/out-of-order/correction shows stale/partial and retries/rebuilds without blocking commerce.

## Planning Or Implementation Steps

1. Approve metric catalog/formulas/source statuses/currency/timezone/privacy/freshness/owner.
2. Collect production query/load/export evidence and select smallest read-model/projection.
3. Define events/checkpoints/idempotency/replay/backfill/late/correction/reconciliation/rebuild.
4. Define API/UI/export/access/rate/expiry/audit/retention and safe storage.
5. Define performance/capacity/index/migration/worker/rollout/kill/rebuild tests.
6. Define objective warehouse/service triggers: sustained DB harm, data volume/history/query diversity, team/on-call and cost.
7. Obtain planning approval before bounded monolith read-model implementation.

## Test And Review Matrix

| Test | Required Cases |
| --- | --- |
| Metrics | Known fixtures, statuses, currency/timezone/boundary/correction and formula version. |
| Projection | Duplicate/out-of-order/late/missing/poison/replay/backfill/rebuild/checkpoint crash. |
| Freshness | Current/stale/partial/failed/no-data; never false zero or transaction dependency. |
| Privacy/access | Role/purpose/BOLA/small cell/PII allowlist/export link expiry/recheck/audit. |
| Performance | Source load protected, bounded queries/exports, lag/throughput/storage/retention. |
| Rollout | Shadow compare/reconcile/canary/flag off/rebuild/rollback and source corrections. |

## Verification Commands

```powershell
rg -n "Dashboard|Reporting|Metric|ReadModel|Projection|Export|Freshness|AsOf|Outbox|Audit" docs/roadmap docs/implementation-guides/phase-3 docs/implementation-guides/phase-6
git diff --check
git status --short
```

## Acceptance Criteria

- Metrics/formulas/source/freshness/privacy/access/retention and owners are deterministic and testable.
- Read models/projections are idempotent/replayable/rebuildable and never transaction truth.
- Exports are bounded, async, authorized, expiring and audited with no privacy leak.
- Future warehouse/service has objective measured triggers and remains unapproved.

## Manual Approval Gates

Manual approval is mandatory for formulas/finance interpretation, source/event changes, PII/small-cell/export, retention/legal hold, schema/index/worker, replica/store/warehouse trigger, cost, rollout/reconciliation and any extraction.

## Stop Conditions

Stop for unknown metric owner/source, reporting as checkout/payment/stock truth, raw PII/unbounded query/export, missing freshness, false zero, non-idempotent projection, transactional DB harm without mitigation, warehouse/service/cloud request without evidence/approval or implementation without separate prompt.

## Rollback / Forward-Fix And Handoff

Planning-only. Future read models run shadow, compare/reconcile, then canary; disable queries/exports and rebuild derived data on defect without source mutation. Hand events to Packet 08, permissions to 07, retention to 12 and extraction criteria to 10.

## Copy-Ready Constrained Prompt

```text
Execute Phase 6 Packet 06 as planning only after prior gates, Packets 01-02 and P6-REPORT-001. Update only metric/read-model/projection/freshness/reconciliation/privacy/export/API/data/test/migration/rollout/rollback and objective warehouse criteria documentation/evidence. Do not write code/migrations, query unrestricted PII, overload source tables, make reports transaction truth, create warehouse/service/cloud, or self-approve formulas. Keep read models rebuildable in the monolith first. Add evidence/06-completion.md with business/privacy/DB review.
```
