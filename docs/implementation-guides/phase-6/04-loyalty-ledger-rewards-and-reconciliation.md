# Packet 04: Loyalty Ledger, Rewards, And Reconciliation

## Packet Metadata

| Field | Value |
| --- | --- |
| Status | Blocked - PHASE0/PHASE1/PHASE2/PHASE3/PHASE4/PHASE5-GATE |
| Depends On | Packet 01 Approved; stable Order/Payment/Refund/Identity/Audit contracts; `P6-LOYALTY-001` |
| Decision Type | Planning first; future monolith ledger module only after finance/security approval |
| Manual Review | Product/loyalty, finance/accounting, order/payment/refund, security/fraud, privacy, DB/audit, support and operations review |
| Produces | Loyalty ledger/reward/reconciliation plan and `evidence/04-completion.md` |

## Objective

Design an append-oriented, auditable Loyalty ledger for earning, redemption, expiry, reversal and reward fulfillment with idempotency, reconciliation, fraud controls and refund/cancellation integration.

## Read First And Prerequisites

- Approved Order/Payment confirmation/failure/refund/cancellation events, money/discount and customer identity/merge/deletion contracts.
- Accounting treatment, point value/expiry/market rules, support correction authority and privacy/legal decisions.
- Packet 01 approved outcome and owner.

## In Scope

- LoyaltyAccount, immutable LedgerEntry, PointsLot/expiry allocation if needed, Reward/Redemption, ReconciliationCase and adjustment approval concepts.
- Earn pending/available/reversed/expired, redeem reserve/commit/release, FIFO/approved expiry, refund partial/full reversal, cancellation/payment failure, account merge/closure.
- Idempotency by source business event/action, balance derived from ledger, concurrency/negative-balance policy, reconciliation and fraud/rate/velocity controls.
- Customer statement and admin/support adjustment/approval/audit, events, retention, phased rollout and rollback.

## Out Of Scope

- Mutable balance without ledger, cryptocurrency/cash equivalence, transferable points, external loyalty vendor, multi-currency conversion, unapproved monetary liability/accounting, autonomous fraud denial, service extraction or paid provider.

## Ownership And Boundaries

- **Owns:** LoyaltyAccount, ledger/lots, reward/redemption, expiry/reversal/adjustment/reconciliation and balance projection.
- **Reads:** verified Order/Payment/Refund/Customer events/queries through approved contracts.
- **Must not own/mutate:** Order/Payment/Refund truth, Product price, Customer identity, promotion rules or Notification delivery.
- Ledger is source of loyalty truth; cached/projected balance is rebuildable.

## Evidence Required

| Evidence | Required Proof |
| --- | --- |
| Entry Evidence | Stable payment/order/refund/idempotency/outbox/audit and approved program/accounting owner. |
| Measured Need Evidence | Program value/liability/volume/fraud/support/reconciliation expectations. |
| Design Evidence | Ledger semantics, lifecycle, accounting/refund/fraud/privacy/API/event/test/migration/rollout. |
| Monolith-First Evidence | Future invariant/property/concurrency/replay/reconciliation and production ledger balance proof. |
| Extraction/Cloud Evidence | Not applicable; remain monolith. |

## Data And Contract Design

Every LedgerEntry records ID/account/type, signed points, source type/ID/version, idempotency key, status/effective/expiry/reversal link, actor/reason/correlation and timestamps. Entries are append-only; correction creates a compensating entry.

Define available/pending/redeemable balance formula, lot consumption/expiry, zero/negative behavior, source uniqueness and reconciliation equation. Reward redemption reserves points before external fulfillment and commits/releases idempotently.

## API And UI Contract

- Customer balance/statement/reward list/redemption uses owner scope, bounded pagination and clear pending/expiry/reversal state.
- Admin/support adjustment/reconciliation uses explicit permission, reason, expected version, limits/two-person approval where material and mandatory audit.
- Checkout redemption is server-side and stores immutable applied points/value snapshot; client cannot set points value or balance.

## Likely File Areas To Inspect

- Core Identity/Order/Payment/Refund/Money/Audit/Outbox contracts, future Infrastructure mappings, API/Web account/admin/checkout, Tests.
- Documentation only now.

## Architecture And Prohibited Dependencies

- Start in modular monolith to keep order/refund/points transaction/compensation understandable.
- No direct mutation of aggregate balance; derive/rebuild projection from ledger.
- Event consumers are idempotent and handle duplicate/out-of-order source versions.
- Reporting does not issue/reverse points.

## Database Changes And Migrations

No migration now. Future schema requires append/unique source/idempotency/concurrency/index/precision constraints, provider review, reconciliation query plan, backup/retention and Phase 5 migration gate.

## Security, Logging, Audit, And Failure Rules

- Prevent owner/BOLA, replay/double earn/redeem, concurrent overspend, insider adjustment, account farming, velocity abuse, refund/chargeback exploitation and source-event forgery.
- Log account/entry/source IDs, type/status/points/reason code/correlation; not customer PII/payment payload.
- Mandatory audit for program/rate/reward/expiry/adjustment/reconciliation/access changes.
- Notification/reward provider failure cannot silently consume points; use reserve/commit/release and reconciliation.

## Planning Or Implementation Steps

1. Approve program economics, point precision/value/status/expiry/accounting/refund/chargeback/legal/privacy.
2. Define append-only entities, invariants, projections, lots, idempotency and reconciliation equation.
3. Define Order/Payment/Refund/Checkout event/command transaction and duplicate/out-of-order behavior.
4. Define fraud/velocity/limits, privileged adjustment/approval/audit and support procedures.
5. Define customer/admin API/UI, retention/deletion/legal hold and statement transparency.
6. Define migration/seed, feature flag/canary, shadow ledger/reconciliation, rollback/forward-fix and tests.
7. Obtain planning approval before a separate bounded monolith implementation packet.

## Test And Review Matrix

| Test | Required Cases |
| --- | --- |
| Ledger | Earn/pending/available/redeem/expire/reverse/adjust; sum/projection invariant. |
| Commerce | Paid/failed/cancelled/partial/full refund/chargeback, promotion interaction and immutable snapshot. |
| Idempotency | Duplicate/out-of-order event, retry, concurrent redemption/last points, source uniqueness. |
| Fraud/access | BOLA, velocity/account farming, self-adjust/approval, excessive adjustment, replay/forged event. |
| Reconciliation | Projection drift, missing/duplicate entry, provider failure, reserve timeout and correction. |
| Rollout | Shadow ledger comparison, canary, flag off, no historical mutation, rollback/forward-fix. |

## Verification Commands

```powershell
rg -n "Order|Payment|Refund|Idempotency|Outbox|Audit|Loyalty|Ledger|Reward" docs/roadmap docs/implementation-guides/phase-2 docs/implementation-guides/phase-6
git diff --check
git status --short
```

## Acceptance Criteria

- Loyalty truth is an append-only idempotent ledger with deterministic balance/expiry/reversal/reconciliation.
- Payment/order/refund integration and customer/admin permissions/audit/fraud controls are unambiguous.
- No client balance/value, mutable-only total, historical rewrite, sensitive log or unapproved financial rule exists.
- Design remains monolith-first and implementation/cloud/extraction is absent.

## Manual Approval Gates

Manual approval is mandatory for program economics/accounting/expiry/refund, ledger precision/invariants, fraud/limits, customer privacy/retention, privileged adjustments/two-person approval, migration, shadow reconciliation and rollout.

## Stop Conditions

Stop for absent refund/accounting contract, mutable balance without ledger, client points/value, duplicate/negative/concurrency ambiguity, historical edit, self-approved adjustment, private data leak, unbounded liability/fraud, external service/cloud or implementation without separate approval.

## Rollback / Forward-Fix And Handoff

Planning-only. Future launch begins shadow/no-redemption, then canary; flag stops new earn/redeem while preserving ledger. Correct with compensating entries, never edits. Reconcile before re-enable. Hand events to Packet 08, reports to 06, access to 07 and risks to 14.

## Copy-Ready Constrained Prompt

```text
Execute Phase 6 Packet 04 as planning only after prior gates, Packet 01 and P6-LOYALTY-001. Update only the append-only Loyalty ledger/reward/earn/redeem/expiry/reversal/refund/idempotency/reconciliation/fraud/audit/API/data/test/rollout design and evidence. Do not write code/migrations, mutate historical balances, trust client points, invent accounting/refund rules, use private profiling, create services/cloud, or self-approve. Keep loyalty in the monolith and require compensating entries. Add evidence/04-completion.md with finance/commerce/security review.
```
