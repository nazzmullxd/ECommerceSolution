# Packet 10: Module Extraction Scorecards And Strangler Plans

## Packet Metadata

| Field | Value |
| --- | --- |
| Status | Blocked - PHASE0/PHASE1/PHASE2/PHASE3/PHASE4/PHASE5-GATE |
| Depends On | Packets 01-02 and 08-09 plus candidate owner evidence; `P6-EXTRACT-001` |
| Decision Type | Architecture decision review only; no service implementation instructions |
| Manual Review | Architecture review board, candidate/data/API/event owners, security/privacy, SRE/on-call, DB/commerce/payment, release and finance/cost review |
| Produces | Nine independent scorecards and `Remain In Monolith` or `Extraction ADR Required`, `evidence/10-completion.md` |

## Objective

Apply one rigorous scorecard separately to Catalog, Inventory, Cart, Order, Payment, Search, AI/RAG, Reporting and Notifications, with the default verdict `Remain In Monolith`; define strangler/fallback planning only for candidates that objectively pass hard gates and require a separate ADR.

## Read First And Prerequisites

- Packet 01 triggers/owners and Packet 02 completed monolith-first production measurements.
- Packet 08 stable event contracts and Packet 09 AI evidence where applicable.
- Phase 5 CI/CD/observability/on-call/security/backup/cost maturity and candidate module runbooks/data ownership.

## In Scope

- Independent candidate dossier: business pain, load/release/team/operational isolation, data/API/event/transaction/dependency boundary, security/compliance, cost/quotas, reliability/SLO/on-call and monolith remedies tried.
- Hard gates, weighted score, confidence/evidence freshness, risks and default verdict.
- For `Extraction ADR Required` only: conceptual strangler stages, contract/data migration/dual-read comparison, no-dual-write approach, event/backfill, routing, observability/security, fallback/reintegration, abort/rollback and cost.
- Decision expiry/review triggers and explicit rejection reasons.

## Out Of Scope

- Service code/project/repository, network API/event implementation, database split, queue/cloud, IaC/deployment/CI, dual writes, distributed transaction, schema migration, team creation or approved extraction ADR itself.

## Ownership And Boundaries

- **Owns:** score methodology, candidate evidence dossier and planning verdict.
- **Reads:** module production/organization/cost/security/contracts and monolith optimization evidence.
- **Must not own/mutate:** module data/API/events, architecture implementation, team/on-call, budget, cloud or risk acceptance.

## Evidence Required

| Evidence | Required Proof |
| --- | --- |
| Entry Evidence | Stable Phase 5 operations and candidate owner/data/runbook/SLO/security/cost maturity. |
| Measured Need Evidence | Sustained independent scale/deploy/release/incident/compliance pain after monolith remedies. |
| Design Evidence | Clear data/API/event/consistency/security/migration/fallback/cost and operational isolation. |
| Monolith-First Evidence | Multiple attempted optimizations with before/after production results and remaining threshold breach. |
| Extraction/Cloud Evidence | Separate approved ADR, dedicated team/on-call, budget/quotas, contract tests and reversible strangler plan. |

## Scorecard And Hard Gates

Score each dimension `0` absent, `1` weak, `2` moderate, `3` strong with direct evidence and confidence:

- Independent scaling and capacity profile.
- Independent deployment/release-frequency need.
- Stable data ownership with no shared writes.
- Stable API and versioned event boundaries.
- Operational/compliance/failure isolation benefit.
- Dedicated team/on-call/security/data ownership.
- Monolith-first remedies exhausted.
- Cost/complexity/business benefit positive.
- Migration/rollback/fallback feasible and tested in design.

Hard gates must all pass regardless of score: stable Phase 5, named dedicated ownership, clear exclusive data writes, no unresolved distributed transaction, contract/version/idempotency/observability/security, funded cost, and reversible fallback. Threshold/weights require `P6-EXTRACT-001` approval. Failing any hard gate means `Remain In Monolith`.

## Candidate Baseline Verdicts

| Candidate | Required boundary focus | Current default |
| --- | --- | --- |
| Catalog | Product/category/variant/image ownership; Inventory availability remains separate. | Remain In Monolith |
| Inventory | Warehouse/stock/reservation/transfer; checkout consistency/oversell risk. | Remain In Monolith |
| Cart | Cart/items/guest merge; pricing/stock excluded. | Remain In Monolith |
| Order | Order snapshots/status; payment/inventory distributed consistency risk. | Remain In Monolith |
| Payment | Provider events/reconciliation; no card data; order settlement boundary. | Remain In Monolith |
| Search | Derived index/ranking; Catalog source, DB keyword fallback. | Remain In Monolith |
| AI/RAG | Knowledge/index/retrieval/evaluation; role filters and disable fallback. | Remain In Monolith |
| Reporting | Derived read models/exports; stale/unavailable fallback. | Remain In Monolith |
| Notifications | Records/delivery attempts; Outbox source and in-app fallback. | Remain In Monolith |

## Data And Contract Design

For each candidate document exclusive writer/database boundary, data lifecycle/retention/backup/deletion, synchronous APIs/timeouts/idempotency/auth/version/SLO, events/ordering/replay/schema/privacy, source/derived copies, transaction/compensation/reconciliation and fallback owner.

No shared-table writes across future service boundary. Temporary reads during strangler are explicit/time-bound; no permanent shared database as "microservice."

## API And UI Contract

No service API/UI implementation. Conceptual contract must preserve existing public API/UI through facade/interface during migration, with compatibility, errors, latency, auth and feature fallback. Customer-visible behavior cannot depend on extraction status.

## Likely File Areas To Inspect

- Candidate module source/data/API/events/tests/runbooks/SLO/incidents/cost and Phase 5/6 evidence.
- Update only Phase 6 scorecard/evidence; no project/service files.

## Architecture And Prohibited Dependencies

- No service implementation instruction until separate ADR is approved after this packet.
- Avoid synchronous call chains in checkout; preserve local transaction until compensation/reconciliation proven.
- No shared writes, dual-write, 2PC, global ordering or "exactly once."
- Strangler starts behind existing interface/facade and preserves monolith fallback until parity/exit criteria.

## Database Changes And Migrations

None. Conceptual data migration must define snapshot/backfill/change capture/event replay/validation/cutover/rollback/retention without implementation. Separate migration ADR/manual gate required.

## Security, Logging, Audit, And Failure Rules

- Scorecards cannot omit identity/service auth, authorization/data scope, encryption/secrets/IAM, audit, privacy/deletion, dependency compromise and incident ownership.
- Evidence uses aggregates/references, no private production data/secrets.
- Network timeout/partial outage/version skew/replay/duplicate/data drift/provider/broker failure and fallback must be designed.
- A high score does not override failed hard gate or unaccepted security/cost risk.

## Planning Or Implementation Steps

1. Approve score dimensions/weights/threshold/hard gates/decision authority/expiry.
2. Build separate current-state dossier for each of nine candidates; do not copy conclusions.
3. Verify monolith remedies and production measurements for each.
4. Score with evidence/confidence and challenge by architecture/security/SRE/data/cost owners.
5. Record `Remain In Monolith` or `Extraction ADR Required`; missing evidence defaults to remain.
6. For ADR-required candidate only, outline conceptual strangler/data/API/event/security/operations/cost/fallback/abort stages.
7. Do not create implementation prompt; hand candidate to separate ADR process.

## Test And Review Matrix

| Review | Required Cases |
| --- | --- |
| Independence | Each candidate scored separately with current direct evidence and confidence. |
| Hard gates | Ownership/data writes/transactions/contracts/team/on-call/security/cost/fallback all pass or remain. |
| Failure | Network/broker/service/DB/version/duplicate/order/partial cutover and monolith fallback. |
| Migration | Shadow/dual-read comparison, no dual-write, backfill/replay/validation/cutover/abort/reintegrate. |
| Cost/operations | CI/CD/compute/data transfer/logs/metrics/queues/on-call/incidents/quotas and business benefit. |
| Scope | No code/project/API/broker/database split/cloud/service implementation instruction. |

## Verification Commands

```powershell
rg -n "Catalog|Inventory|Cart|Order|Payment|Search|AI/RAG|Reporting|Notification|Remain In Monolith|Extraction ADR" docs/implementation-guides/phase-6
git diff --check
git status --short
```

## Acceptance Criteria

- Nine candidates have independent evidence-backed scorecards and hard-gate results.
- Missing/weak/stale evidence or any failed hard gate produces `Remain In Monolith`.
- Any `Extraction ADR Required` includes conceptual reversible strangler/fallback/cost/security/operations plan but no implementation instruction.
- No service/cloud/database split/project/API/event code is added.

## Manual Approval Gates

Manual architecture review board approval is mandatory for score model, each candidate verdict, any ADR referral, data/API/event boundary, transaction/compensation, security/privacy, team/on-call, cost/quotas, migration/fallback and decision expiry.

## Stop Conditions

Stop for blanket score across modules, preference/forecast instead of evidence, failed hard gate, shared writes/dual-write/2PC, missing monolith fallback, unknown team/on-call/cost, service/cloud implementation instruction, or extraction declared approved without separate ADR.

## Rollback / Forward-Fix And Handoff

Documentation-only. Expired/changed evidence reverts verdict to `Remain In Monolith`. ADR referral can be withdrawn without runtime effect. If future extraction fails, strangler ADR must route back to monolith and reconcile data before teardown. Hand service/AWS triggers to Packet 13 and risk to 14.

## Copy-Ready Constrained Prompt

```text
Execute Phase 6 Packet 10 as architecture-review documentation only after prior gates, Packets 01-02/08-09 and P6-EXTRACT-001. Apply the approved scorecard separately to Catalog, Inventory, Cart, Order, Payment, Search, AI/RAG, Reporting and Notifications using current production evidence. Default every candidate to Remain In Monolith; only output Extraction ADR Required when every hard gate passes. Do not write service/project/API/event/database migration/cloud implementation instructions or approve extraction. Add evidence/10-completion.md with nine verdicts and named ARB review.
```
