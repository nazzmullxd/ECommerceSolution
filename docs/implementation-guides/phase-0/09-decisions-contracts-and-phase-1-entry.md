# Packet 09: Decisions, Contracts, And Phase 1 Entry

## Packet Metadata

| Field | Value |
| --- | --- |
| Status | Not Started |
| Depends On | Packets 01-08 Approved |
| Produces | `artifacts/decision-register.md`, `artifacts/cross-phase-contract-register.md`, selected `artifacts/adr/*.md` |
| Human Reviewers | Product owner, solution architect, security reviewer, development lead, operator/cost owner |

## Objective

Resolve or govern open questions, record architecture decisions, freeze cross-phase contracts, and define a hard Phase 1 entry gate so future coding agents cannot silently reinterpret Phase 0.

## Read First

- All approved Phase 0 packet artifacts
- `ECommerce Platform Requirements Roadmap.md`
- `docs/roadmap/phase-0.md`, sections 12-14
- `docs/architecture/system-design.md`
- `docs/architecture/security-scalability-review.md`
- `docs/ai-rag/rag-architecture.md`
- `docs/implementation-guides/ai-assisted-development-playbook.md`

## In Scope

- Decision status/ownership, ADRs, defaults/deadlines, conflict resolution, cross-phase contracts, change-control process, Phase 1 prerequisites, implementation sequencing constraints, and approval evidence.

## Out Of Scope

- Implementing decisions in code, selecting packages, creating migrations, provisioning services, or approving Phase 1 code.
- Forcing decisions not required for Phase 1 when an explicit safe default/deadline exists.
- Treating an AI-generated decision as human approval.

## Required Artifacts

### `artifacts/decision-register.md`

For each question/decision record: ID, statement, options, decision/default, status (`Proposed`, `Accepted`, `Deferred`, `Rejected`, `Superseded`), rationale, consequences/trade-offs, security/cost impact, owner, approvers, required-by gate, evidence, date, and ADR link.

At minimum cover:

- Modular monolith and Onion Architecture.
- Current project layout and deferred `src/`/`tests/` move.
- Local/free-first adapter strategy.
- Local database default and production database decision timing.
- Same Web app for customer/admin MVP.
- Web cookie versus API bearer-token boundary.
- `/api/v1`, Problem Details, correlation, and idempotency baseline.
- Database outbox before SQS/EventBridge.
- Hosted/mock/sandbox payment and no card storage.
- Inventory reservation/expiry/concurrency.
- Local file abstraction and future S3.
- Mock/local AI/RAG, approved sources, authorization, fallback, and future Bedrock/OpenSearch.
- Future ECS Fargate before EKS and single-region first.
- Target market/currency/tax/shipping/retention blockers.

### `artifacts/adr/*.md`

Create an ADR only for a decision that is accepted or whose trade-offs need durable explanation. Use `ADR-NNNN-short-title.md` and include Context, Decision, Status, Options, Consequences, Security/Privacy, Cost/Operations, Validation, Revisit Trigger, and Links.

### `artifacts/cross-phase-contract-register.md`

Include contract ID, contract, owner, consumers/phases, invariant, allowed extension, breaking change, tests/evidence, and change approval. At minimum include:

- `ARCH-001`: Onion dependency direction.
- `IDENTITY-001`: `ApplicationUser` plus role/permission model.
- `API-001`: `/api/v1`, camelCase, Problem Details, correlation.
- `API-002`: idempotency for risky commands.
- `DATA-001`: one source-of-truth owner and immutable snapshots.
- `SEC-001`: explicit auth/ownership/negative tests and privacy-safe logs.
- `COMMERCE-001`: server-calculated trusted totals.
- `INVENTORY-001`: reservation/expiry/concurrency prevents overselling.
- `PAYMENT-001`: hosted provider, no card data, signed/replay-safe callback.
- `EVENT-001`: transactional outbox, versioned integration events, idempotent consumers.
- `FILES-001`: validated local abstraction, S3 later.
- `AI-001`: approved-source, authorization-aware, confidence/fallback RAG.
- `OPS-001`: local/free first; future AWS adapter replacement and cost approval.

Add a Phase 1 entry checklist that references evidence rather than yes/no opinion.

## Questions To Answer

- Which unresolved decisions truly block Phase 1?
- Which defaults can be accepted temporarily, with what revisit trigger?
- Who may approve a breaking cross-phase contract change?
- What evidence is required before an ADR becomes `Accepted`?
- What happens when a later phase discovers a conflict?

## Contract And Safety Rules

| Concern | Rule |
| --- | --- |
| Architecture | Accepted ADRs/contracts outrank packet instructions; breaking changes require explicit review. |
| Data/API/UI | Contracts state invariants and allowed extensions; no silent schema/route/identity drift. |
| Security | Security-reducing changes require security approval; high-risk blockers cannot be defaulted by AI. |
| Logging/Audit | Decision evidence excludes secrets/PII; approval history identifies role/date/reason. |
| Failure | A missing Phase 1-critical decision blocks entry; a deferred later decision has owner, default, deadline, and impact. |

## Required Diagrams

- Mermaid decision lifecycle: proposed, reviewed, accepted/deferred/rejected, superseded.
- Mermaid cross-phase contract change-control flow.
- Mermaid Phase 1 entry decision flow.

## Step-By-Step Instructions

1. Collect every open question, assumption, conflict, risk acceptance, and default from Packets 01-08 and source docs.
2. Deduplicate and assign decision IDs, owners, required-by gates, and statuses.
3. Identify accepted architecture choices needing ADRs; do not create ADRs for trivial wording.
4. Write ADRs with alternatives, consequences, security, cost, validation, and revisit triggers.
5. Build cross-phase contracts from approved domain/API/data/identity/security/operations artifacts.
6. Define allowed extensions versus breaking changes and approvers.
7. Map each Phase 1 entry criterion to artifact/evidence and blocker status.
8. Draw decision/change/entry diagrams.
9. Run consistency review across all artifacts.
10. Obtain named human approvals; do not self-approve.

## Expected File Areas

- Create/update only the decision register, cross-phase contract register, selected ADRs, and Packet 09 status under `docs/implementation-guides/phase-0/`.

## Verification And Evidence

| Check | Evidence |
| --- | --- |
| Decision completeness | Every open question/default/risk acceptance has an ID and owner. |
| ADR quality | Accepted durable decisions include trade-offs/revisit triggers. |
| Contract coverage | Required cross-phase contract IDs exist with owners and change rules. |
| Phase 1 gate | Every entry criterion links to approved evidence. |
| Approval | Human reviewer roles/dates recorded; AI is not approver. |
| Scope | Only Phase 0 docs changed. |

```powershell
rg -n "Proposed|Accepted|Deferred|Rejected|Superseded|Owner|Required By" docs/implementation-guides/phase-0/artifacts/decision-register.md
rg -n "ARCH-001|IDENTITY-001|API-001|API-002|DATA-001|SEC-001|COMMERCE-001|INVENTORY-001|PAYMENT-001|EVENT-001|FILES-001|AI-001|OPS-001" docs/implementation-guides/phase-0/artifacts/cross-phase-contract-register.md
rg -n "TBD|TODO|BLOCKED" docs/implementation-guides/phase-0/artifacts
git diff --check
git diff --name-only
```

Build/test evidence may reuse Packet 01. Record documentation-only scope.

## Review Checklist

- Decisions, defaults, assumptions, and blockers are distinct.
- Every blocker has owner and deadline/gate.
- ADRs explain alternatives and consequences.
- Contracts state invariants and change control.
- Phase 1 cannot start with unresolved identity/security/dependency blockers.
- Future AWS/payment/AI choices are not falsely approved.

## Acceptance Criteria

- Decision register, contract register, necessary ADRs, and three diagrams exist.
- Required contract IDs are complete and approved.
- Every Phase 1 entry criterion has evidence and owner.
- Critical blockers are resolved or Phase 1 remains explicitly blocked.
- Named human reviewers approve the registers.

## Manual Review And Stop Conditions

Stop if reviewers cannot resolve a source conflict, a high-risk decision lacks a security owner, Phase 1 requires a deferred critical contract, or someone asks the AI to mark decisions approved.

## Rollback

Revert only Packet 09 artifacts/status. Do not remove previously approved packet artifacts. Supersede accepted ADRs rather than deleting their history after use.

## Handoff

Packet 90 verifies requirement coverage. Packet 99 uses the decision/contract registers and Phase 1 entry evidence for final approval.

## Copy-Ready Execution Prompt

```text
Execute Phase 0 Packet 09 from docs/implementation-guides/phase-0/09-decisions-contracts-and-phase-1-entry.md only after Packets 01-08 are approved. Create the decision register, required cross-phase contract register, and only necessary ADRs under the allowed Phase 0 artifact paths. Resolve/deduplicate open questions, define owners/defaults/gates/change control, map Phase 1 entry evidence, add all three diagrams, run consistency/scope checks, and stop for named human approval. Never self-approve a decision.
```

