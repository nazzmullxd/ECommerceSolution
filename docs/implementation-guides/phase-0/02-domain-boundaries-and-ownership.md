# Packet 02: Domain Boundaries And Ownership

## Packet Metadata

| Field | Value |
| --- | --- |
| Status | Not Started |
| Depends On | Packet 01 Approved |
| Produces | `artifacts/domain-contracts.md` |
| Human Reviewers | Solution architect, product owner, security reviewer |

## Objective

Define stable modular-monolith boundaries, data/rule ownership, allowed dependencies, module interactions, commands, queries, events, and data classifications so later implementation agents do not invent cross-module behavior.

## Read First

- Approved Packet 01 artifacts
- `ECommerce Platform Requirements Roadmap.md`, sections 6, 8, 11-13
- `docs/roadmap/phase-0.md`, sections 4, 7-10
- `docs/architecture/system-design.md`
- `docs/architecture/security-scalability-review.md`
- `docs/ai-rag/rag-architecture.md`

## In Scope

- Auth, User, Product/Catalog, Inventory, Cart, Checkout, Order, Payment, Admin, Customer Experience, Operations/Outbox, and AI/RAG boundaries.
- Module ownership and allowed read/command/event interactions.
- Internal domain event versus versioned integration event rules.
- Public, customer-private, staff/internal, payment-sensitive, audit-sensitive, and AI-indexed classifications.

## Out Of Scope

- Physical microservices, separate databases, code namespaces, repositories, or project moves.
- Final endpoint schemas, table schemas, provider choices, and implementation classes.
- Allowing Admin or AI/RAG to bypass source-module rules.

## Required Artifact

Create `artifacts/domain-contracts.md` containing:

1. Bounded-context map and beginner explanation.
2. Module responsibility matrix: owns, may read, may command, emits, consumes, and must not own.
3. Canonical aggregate candidates and source-of-truth owner.
4. Allowed dependency matrix and forbidden dependency examples.
5. Command/query/event catalog with owner and expected consumer.
6. Event naming/versioning/idempotency expectations.
7. Data-classification matrix and permitted module access.
8. Cross-module workflow descriptions for registration, product publish, cart-to-order, payment callback, inventory reservation, notification/outbox, and RAG indexing.
9. Known consistency boundary: strong transaction, eventual consistency, snapshot, or derived/read model.
10. Boundary risks and unresolved ownership decisions.

## Questions To Answer

- Is Checkout a separate coordination module or part of Cart for MVP documentation?
- Does Payment own payment status while Order owns order status, and how are transitions coordinated?
- Which module owns Address versus immutable AddressSnapshot?
- Which events are internal-only and which may enter the outbox?
- Which data may AI/RAG copy, and which classifications are excluded?
- Which Admin operations are orchestration over module services rather than data ownership?

## Contract And Safety Rules

| Concern | Rule |
| --- | --- |
| Architecture | Modular monolith first; Core abstractions point inward; Infrastructure implements adapters. |
| Data | One source-of-truth owner per entity/rule; snapshots and indexes are explicitly derived. |
| API/UI | Presentation calls application services; UI hiding is not authorization. |
| Security | Authorization and data classification apply before cross-module data is returned. |
| Logging/Audit | Events/logs carry IDs and classifications, not full sensitive payloads. |
| Failure | Cross-module eventual work uses outbox/retry; money/stock workflows require explicit compensation. |

## Required Diagrams

- Mermaid bounded-context map.
- Mermaid module dependency graph using arrows with an explicit legend.
- Mermaid cart-to-order interaction overview marking transaction/event boundaries.
- Mermaid data-classification flow for public, private, payment, audit, and AI-indexed data.

## Step-By-Step Instructions

1. Normalize module names against the glossary and roadmap; record aliases rather than duplicating modules.
2. Assign every Phase 0 entity and business rule to one source-of-truth owner.
3. Identify commands and queries that cross module boundaries.
4. Define events, producer, consumer, payload classification, idempotency, and delivery expectation.
5. Mark strong-consistency, snapshot, derived-data, and eventual-consistency boundaries.
6. Document Admin as authorized orchestration and AI/RAG as derived/indexed data only.
7. Build allowed/forbidden dependency matrices.
8. Draw diagrams with unambiguous arrow semantics.
9. Review every boundary against critical journeys and threats from Packet 01.
10. Record conflicts and seek human approval before later packet approval.

## Expected File Areas

- Create/update only `docs/implementation-guides/phase-0/artifacts/domain-contracts.md`.
- Update Packet 02 status in `README.md` after review.

## Verification And Evidence

| Check | Evidence |
| --- | --- |
| Entity ownership | Every Phase 0 entity maps to exactly one owner. |
| Dependency coverage | Matrix lists allowed and forbidden edges. |
| Workflow coverage | Seven required cross-module flows are documented. |
| Classification | Payment/private data cannot flow to public/AI contexts. |
| Diagram quality | Mermaid fences balanced; arrows explained. |
| Scope | Diff contains only allowed Phase 0 docs. |

Suggested commands:

```powershell
rg -n "Auth|User|Product|Inventory|Cart|Checkout|Order|Payment|Admin|AI/RAG|Outbox" docs/implementation-guides/phase-0/artifacts/domain-contracts.md
rg -n "source of truth|must not|forbidden|eventual|transaction" docs/implementation-guides/phase-0/artifacts/domain-contracts.md
git diff --check
git diff --name-only
```

Build/test evidence may reuse Packet 01 because this is documentation-only; record that fact.

## Review Checklist

- Module names match the canonical glossary.
- Data and rule ownership are explicit.
- Checkout/payment/inventory/order responsibilities do not overlap silently.
- Admin and AI/RAG cannot bypass owners.
- Event semantics and transaction boundaries are understandable.
- No microservice or separate-database requirement was introduced.

## Acceptance Criteria

- Required artifact and four diagrams exist.
- Every entity/rule/flow has an owner and consistency mode.
- Forbidden dependencies and sensitive-data exclusions are explicit.
- Product owner, architect, and security reviewer approve the boundaries.
- No unresolved ownership blocker remains for Packets 03-08.

## Manual Review And Stop Conditions

Stop when two modules claim the same source-of-truth rule, a workflow requires an undefined owner, or a proposed boundary creates a distributed transaction. Record the conflict; do not solve it by introducing a service.

## Rollback

Revert only `artifacts/domain-contracts.md` and Packet 02 status. Preserve Packet 01 and unrelated changes.

## Handoff

Packets 03-08 receive the approved module names, owners, interactions, classifications, event rules, and consistency boundaries.

## Copy-Ready Execution Prompt

```text
Execute Phase 0 Packet 02 from docs/implementation-guides/phase-0/02-domain-boundaries-and-ownership.md after Packet 01 is approved. Create only artifacts/domain-contracts.md and the allowed status update. Assign one source-of-truth owner to every entity and rule, define allowed/forbidden dependencies and classified data flows, add the required Mermaid diagrams, run verification, and stop for human review on any ownership conflict. Do not create code or microservices.
```
