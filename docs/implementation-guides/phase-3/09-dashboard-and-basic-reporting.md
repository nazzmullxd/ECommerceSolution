# Packet 09: Dashboard And Basic Reporting

## Packet Metadata

| Field | Value |
| --- | --- |
| Status | Blocked - PHASE0/PHASE1/PHASE2-GATE |
| Depends On | Packet 01 Approved; approved Product/Order/Payment/Inventory/Support/Review read contracts; `P3-REPORT-001` |
| Layer Ownership | Reporting owns derived queries/read models only; source modules own transactional truth |
| Risk Review | Mandatory pre-code and post-test human review by security/privacy, database/performance, commerce, operations, and UX owners |
| Produces | Bounded dashboard/reporting vertical slice, `evidence/09-completion.md` |

## Objective

Provide a small, privacy-safe operational dashboard and basic reports that tolerate stale/partial data, stay within query budgets, and never become transactional truth or overload the commerce database.

## Read First And Prerequisites

- Approved Phase 1 privileged access/audit and Phase 2-3 source-module read contracts/evidence.
- Packet 01 and resolved `P3-REPORT-001` metrics, freshness, date boundary/timezone, query budget, and read-model decisions.
- Phase 3 roadmap dashboard/reporting, authorization, security, failure, logging, testing, and UI sections.

## In Scope

- Approved metrics only: order counts/value by public operational status, payment success/failure count, low-stock count, open support count, pending-review count, and recent operational activity if privacy-approved.
- Bounded date ranges, allowlisted grouping/filtering, UTC storage with approved display timezone, freshness/as-of metadata, and cancellation/timeouts.
- Direct optimized aggregate queries for small data by default; introduce DashboardMetric/read model only with measured evidence and owner approval.
- Admin dashboard and basic summary API/UI, partial/unavailable states, permission checks, and access audit.

## Out Of Scope

- Data warehouse, BI platform, arbitrary query builder/export, customer profiling, cohort/LTV/attribution, financial accounting truth, real-time SLA, predictive analytics, cross-customer drill-down, or advanced Phase 6 analytics.

## Ownership And Read-Only Contracts

- **Owns:** reporting query definitions, optional derived DashboardMetric rows, freshness/error metadata, and no source business state.
- **Reads:** approved aggregate projections from Orders, Payments, Inventory, Support, Reviews, and Catalog.
- **Must not own/mutate:** transactional entities, statuses, totals, stock, payment, ticket/review state, CustomerProfile, or module migrations without owner review.

## API Contract

| Route | Auth | Behavior |
| --- | --- | --- |
| `GET /api/v1/admin/dashboard/summary` | `dashboard.read` | Fixed metric set, per-metric status, `asOfUtc`, and freshness. |
| `GET /api/v1/admin/reports/commerce-summary` | `reports.read` | Bounded date range/group/filter summary with no row-level PII. |
| `GET /api/v1/admin/reports/operations-summary` | `reports.read` | Low-stock/open-ticket/pending-review aggregates only. |

Reject unknown filters/groupings/sorts, excessive date range/page size, and invalid time boundaries with Problem Details. Return a successful partial envelope only when its contract clearly marks failed metrics; do not convert authorization or total database failure to fake zeroes.

## UI Acceptance States

- Dashboard: loading skeleton with stable dimensions, complete success, empty/zero, stale/as-of marker, individual unavailable metric, total failure, forbidden, and retry.
- Reports: date/filter validation, loading, no data, success, partial/stale, timeout, unavailable, and accessible table alternative to any chart.
- Values include units/currency/timezone; errors never reveal SQL/internal details, and privileged data is not placed in page source for unauthorized roles.

## Likely File Areas To Inspect

- Source-module application read contracts and Phase 1 authorization/audit services.
- Infrastructure query implementations, query plans/index evidence, optional read-model mapping/refresh worker.
- API DTOs/endpoints/OpenAPI and Web dashboard/report views/components/styles/scripts.
- Unit, integration, authorization, privacy, query-plan/performance, partial-failure, UI/accessibility, and audit tests.

## Architecture And Prohibited Dependencies

- Reporting uses read-only query services and projections. Dashboard cards do not become dependencies of checkout, payment, inventory, or status decisions.
- Avoid one giant cross-module EF query. Compose bounded aggregates or a reviewed read model; controllers/views do not query DbContext.
- No cache for transactional truth. Dashboard cache/read model may be stale only with explicit `asOfUtc`/freshness and invalidation/TTL policy.

## Database Changes And Migrations

Default: no new entity. Add indexes only after captured query plan/timing and source-owner migration review. Add DashboardMetric/read-model table only if direct queries exceed approved budgets; document source event/watermark, rebuild, duplicate handling, consistency, and no-source-of-truth guarantee.

## Security, Logging, And Failure Rules

- Separate dashboard/report permissions and audit privileged access with actor, report/metric set, normalized date range, result, and correlation ID.
- Aggregate/minimize output; suppress small-cell or identifying drill-down when privacy policy requires. Never expose full address, contact, ticket/review text, payment provider detail, or customer list.
- Parameterize queries, allowlist controls, apply timeouts/cancellation/range limits, and rate-limit expensive endpoints.
- Log query/report name, normalized filters, duration/row or metric count, freshness, timeout/result, and correlation; never SQL parameters containing PII, result rows, or full request bodies.
- Failed/stale metrics are labelled, not zeroed. Reporting outage never blocks commerce.

## Implementation Steps

1. Obtain mandatory pre-code approval for exact formulas/source statuses, currency/timezone, date bounds, privacy threshold, permissions, freshness, query budgets, direct/read-model choice, exact routes/UI.
2. Define fixed metric/report DTOs and read interfaces with per-metric status/as-of metadata.
3. Implement bounded read-only queries; capture plans/timings before any index/read-model proposal.
4. If evidence requires, obtain separate schema approval and implement rebuildable derived read model.
5. Implement permission/audit/rate/timeout controls, exact APIs, and accessible dashboard/report UI.
6. Add formula, source-state, authorization, privacy, injection/filter, timeout/partial/stale, load, audit, UI/accessibility tests.
7. Run checks, capture query evidence, and obtain mandatory post-test human approval.

## Test Matrix

| Test | Required Cases |
| --- | --- |
| Formula | Approved source statuses, currency/timezone/date boundaries, zero/no data, known fixture totals. |
| Authorization/privacy | Customer/support/default admin denied as designed; aggregate-only; no small-cell/PII leakage; access audit. |
| Query safety | Allowlist/range/page; parameterization; cancellation/timeout; rate limit; captured budget/plan. |
| Consistency | Direct or read-model freshness; replay/rebuild if used; stale/partial/error not converted to zero. |
| Independence | Reporting failure does not affect catalog/cart/checkout/order/payment. |
| UI | Loading/empty/success/partial/stale/error/forbidden, units, table alternative, keyboard/screen-reader. |

## Verification Commands

```powershell
dotnet build ECommerceSolution.slnx
dotnet test ECommerceSolution.slnx --no-build
rg -n "dashboard.read|reports.read|DashboardMetric|asOfUtc|Freshness|GroupBy|Timeout" ECommerce.Core ECommerce.Infrastructure ECommerce.API ECommerce.Web ECommerce.Tests
git diff --check
git status --short
```

## Acceptance Criteria

- Every metric has approved formula/source/status/timezone/currency/freshness and a deterministic fixture test.
- Reporting is bounded, read-only, permissioned, audited, privacy-safe, measured, and independent of commerce truth.
- Partial/stale/unavailable states are explicit in API/UI; no failure becomes a misleading zero.
- Any index/read model has evidence, source-owner approval, rebuild behavior, and migration tests.
- Mandatory pre-code and post-test human approvals and evidence exist.

## Security Stop Conditions

Stop for missing human review/formula, arbitrary query/filter, unbounded date/query, PII or small-cell leakage, weak permission/audit, SQL details, cross-module writes, reporting as transactional dependency, fake zero on failure, unmeasured index/read model, or destructive migration.

## Rollback / Forward-Fix And Handoff

Disable dashboard/report endpoints or stale refresh worker without changing source modules. Derived data may be discarded and rebuilt; never repair source truth through Reporting. Hand approved projections/performance evidence to Packet 11 and Phase 5 observability/capacity planning.

## Copy-Ready Coding Prompt

```text
Execute Phase 3 Packet 09 only after all prerequisite gates, Packet 01, approved source read contracts, P3-REPORT-001, and mandatory pre-code human review. Implement only fixed bounded privacy-safe aggregate queries, optional evidence-justified rebuildable read model, exact permissioned/audited APIs, accessible UI, freshness/partial states, budgets, and tests. Never mutate source modules, expose PII/arbitrary queries, use reporting as commerce truth, or add schema/indexes without measured owner approval. Add evidence/09-completion.md and query evidence, then obtain mandatory post-test review.
```
