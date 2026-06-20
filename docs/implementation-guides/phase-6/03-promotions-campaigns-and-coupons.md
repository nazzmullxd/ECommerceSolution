# Packet 03: Promotions, Campaigns, And Coupons

## Packet Metadata

| Field | Value |
| --- | --- |
| Status | Blocked - PHASE0/PHASE1/PHASE2/PHASE3/PHASE4/PHASE5-GATE |
| Depends On | Packet 01 Approved; stable Pricing/Checkout/Order/Payment contracts; `P6-PROMO-001` |
| Decision Type | Planning first; future bounded monolith module only after business/security/commerce approval |
| Manual Review | Product/pricing, finance/accounting, commerce/payment/refund, security/fraud, marketing, DB, audit, privacy and operations review |
| Produces | Promotion module contract/test/rollout plan and `evidence/03-completion.md` |

## Objective

Design a server-authoritative Promotions module for campaigns, rules and coupons with deterministic precedence/stacking/rounding, usage limits, abuse controls, approval/audit, immutable order snapshots and refund/cancellation effects.

## Read First And Prerequisites

- Approved money/currency/rounding, Catalog pricing, Checkout idempotency/recalculation, Order snapshots, Payment/refund and Audit contracts.
- Target market/legal/tax/discount representation and promotion approval authority.
- Packet 01 approved business outcome and owner/capacity.

## In Scope

- Promotion/Campaign/Coupon/Rule/Audience/StackingPolicy/CouponRedemption/PromotionEvaluation and immutable applied-discount snapshot concepts.
- Lifecycle draft/review/approved/scheduled/active/paused/expired/ended/cancelled with UTC clock/concurrency/audit.
- Eligibility by approved product/category/channel/customer segment only when privacy/legal approved; minimums, quantity, date, budget, global/per-customer/per-order usage.
- Fixed/percentage/free-shipping or explicitly approved types, caps, exclusions, precedence/stacking/conflict, allocation across lines, rounding and tax/shipping interaction.
- Server validation/evaluation/reservation/redemption/release/reconciliation, idempotency, rate/guess abuse, admin approval and phased rollout.

## Out Of Scope

- Client discount amount, dynamic AI pricing, sensitive profiling, marketplace/vendor funding, multi-currency conversion, tax/legal invention, gift cards, real refund implementation if absent, service extraction or paid campaign platform.

## Ownership And Boundaries

- **Owns:** promotion/campaign/coupon/rule/approval/usage/redemption/evaluation and applied promotion identity.
- **Reads:** trusted Catalog/pricing/customer eligibility/Checkout context through approved interfaces.
- **Must not own/mutate:** base Product price, tax/payment truth, Order item snapshot, CustomerProfile, refund workflow or Inventory.
- Checkout remains final total orchestrator and stores server-calculated immutable discount snapshots.

## Evidence Required

| Evidence | Required Proof |
| --- | --- |
| Entry Evidence | Stable checkout/payment/order/refund/audit and approved business need/owner. |
| Measured Need Evidence | Campaign/use-case/value/volume/abuse/latency expectations and current limitation. |
| Design Evidence | Rules/lifecycle/precedence/rounding/usage/approval/refund/security/API/data/test/rollout/rollback. |
| Monolith-First Evidence | Future deterministic unit/property/concurrency/integration/security/performance and production results. |
| Extraction/Cloud Evidence | Not applicable; module remains monolith. |

## Data And Contract Design

Define unique coupon code hash/lookup policy, display code handling, versioned rule definition using structured typed model (not executable expressions), schedule/clock, budget/usage counters, reservation/redemption idempotency, approval actor/reason, concurrency, and order snapshot fields.

Evaluation input is server-built with customer ID only when authorized, product/variant/category, quantities, trusted prices/currency, channel, shipping/tax context and current UTC. Output lists eligible/ineligible reason codes, applied rule/version, allocation and totals; client cannot set rule/version/amount/weights.

## API And UI Contract

- Customer coupon validate/apply is a Checkout command with rate/idempotency and generic invalid response that avoids private-code enumeration.
- Admin author/review/approve/schedule/pause/cancel endpoints use explicit commands, permissions, expected version, reason, two-person approval for high-risk campaigns and mandatory audit.
- UI shows exact eligibility/expiry/stacking terms, recalculated total and removal/conflict state; server revalidates at checkout/order.

## Likely File Areas To Inspect

- Core money/pricing/checkout/order/audit contracts, Infrastructure mappings/transactions, API/Web checkout/admin, Tests commerce/security/concurrency.
- Planning docs only now.

## Architecture And Prohibited Dependencies

- Promotions starts as a module in the same transaction boundary needed for checkout usage/redemption consistency.
- No arbitrary script/SQL/expression execution. Rules are allowlisted typed policies.
- Cache may hold public campaign display metadata, never usage/redemption/eligibility truth.
- Reporting/marketing read models do not decide discounts.

## Database Changes And Migrations

No migration now. Future schema requires unique/concurrency/index/usage constraints, money precision/rounding review, provider plan, backfill/compatibility/backup and Phase 5 migration gate. Coupon uniqueness and redemption limits need database enforcement plus service validation.

## Security, Logging, Audit, And Failure Rules

- Prevent code enumeration, brute force, self-approval, time/race usage bypass, replay, client amount/eligibility tampering and privacy segment leakage.
- Log campaign/promotion/rule IDs/version, coupon hash/reference, evaluation/redemption reason, amounts/currency, actor/correlation; not full code, PII or cart/payment payload.
- Mandatory audit for lifecycle/rules/budget/approval/manual redemption change.
- Failure before order commits releases reservation; refund/cancel reversal follows approved idempotent compensation and never invents refund state.

## Planning Or Implementation Steps

1. Resolve promotion types/precedence/stacking/rounding/tax/shipping/eligibility/usage/refund/accounting and approval policy.
2. Define entities/lifecycle/typed rule/value objects and transaction/idempotency boundaries.
3. Define deterministic evaluation/allocation examples and property invariants.
4. Define coupon storage/enumeration/rate/usage/concurrency/reconciliation/audit controls.
5. Define Checkout/Order/Refund event/API/UI contracts and immutable snapshots.
6. Define migration, feature-flag/canary, monitoring/kill, rollback/forward-fix and test matrix.
7. Obtain planning approval; issue a separate small monolith implementation prompt only afterward.

## Test And Review Matrix

| Test | Required Cases |
| --- | --- |
| Rules | Every type/eligibility/exclusion/minimum/cap/schedule/currency/rounding/allocation. |
| Stacking | Priority/exclusive/combinable/conflict/order independence and deterministic tie. |
| Usage | Global/customer/order limits, concurrent last use, reservation expiry, duplicate/idempotent retry. |
| Security | Guess/rate/enumeration/tamper/self-approve/segment leak/BOLA/admin separation. |
| Commerce | Price change, cart revalidation, immutable order snapshot, payment fail/cancel/refund partial/full. |
| Failure/rollout | DB/audit/outbox failure, stale rule, flag off, canary/rollback and reconciliation. |

## Verification Commands

```powershell
rg -n "Money|Currency|Rounding|Checkout|Discount|Coupon|Promotion|OrderItem|Refund|Audit|Idempotency" docs/roadmap docs/implementation-guides/phase-2 docs/implementation-guides/phase-6
git diff --check
git status --short
```

## Acceptance Criteria

- Server-side deterministic promotion semantics, snapshots, usage, idempotency, approval/audit and refund/cancel effects are unambiguous.
- Client cannot set discount/eligibility/rule; database/service concurrency prevents over-redemption.
- Security/privacy/finance/tax/commerce owners approve typed rules and tests.
- Design remains monolith-first with no code/migration/service/cloud now.

## Manual Approval Gates

Manual approval is mandatory for rule types/stacking/rounding/tax/shipping, eligibility/segmentation, budgets/usage, coupon security, high-value campaign approval, Checkout/Order/refund integration, migration, rollout and financial reconciliation.

## Stop Conditions

Stop for missing commerce/refund contract, client amount/eligibility, arbitrary rule execution, self-approval, undefined rounding/stacking/refund, sensitive profiling, unbounded usage/rate, weak concurrency/idempotency/audit, microservice/cloud request or implementation without separate approval.

## Rollback / Forward-Fix And Handoff

Planning-only. Future rollout defaults off, canaries campaigns, preserves applied order snapshots and pauses new evaluation/redemption on defect; do not rewrite historical orders. Reconcile/redress through approved financial process. Hand events to Packet 08, analytics to 06, permissions to 07 and risks to 14.

## Copy-Ready Constrained Prompt

```text
Execute Phase 6 Packet 03 as planning only after prior gates, Packet 01 and P6-PROMO-001. Update only the Promotions/Campaign/Coupon ownership, typed rule/lifecycle/stacking/rounding/usage/approval/audit/refund/API/data/test/migration/rollout/rollback design and evidence. Do not write code/migrations, trust client discounts, invent tax/refund rules, use sensitive profiling/arbitrary expressions, create services/cloud, or self-approve. Keep Checkout server-authoritative and the module in the monolith. Add evidence/03-completion.md with named finance/commerce/security review.
```
