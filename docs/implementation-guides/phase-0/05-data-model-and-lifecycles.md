# Packet 05: Data Model And Lifecycles

## Packet Metadata

| Field | Value |
| --- | --- |
| Status | Not Started |
| Depends On | Packet 02 Approved |
| Produces | `artifacts/data-model.md` |
| Human Reviewers | Data architect, solution architect, security reviewer |

## Objective

Define the conceptual MVP data model, ownership, relationships, invariants, lifecycles, indexes, concurrency, snapshots, retention classes, and migration assumptions without creating schemas or EF Core migrations.

## Read First

- Approved Packet 01-02 artifacts
- `ECommerce Platform Requirements Roadmap.md`, domain model groups
- `docs/roadmap/phase-0.md`, section 7
- Phase 1-4 roadmap data-model sections for consistency
- `docs/architecture/system-design.md`
- `docs/ai-rag/rag-architecture.md`

## In Scope

- Conceptual entities for identity/access, user, catalog, inventory, cart/checkout, order/payment, customer experience, AI/RAG, and operations.
- Aggregate ownership, relationships, invariants, status/lifecycle rules, unique/index candidates, concurrency, immutable snapshots, deletion/retention classes, and provider-neutral migration assumptions.

## Out Of Scope

- EF classes/configuration, SQL, migrations, physical column types, provider-specific features, production engine selection, seed credentials, or data import.
- Advanced promotions, multi-warehouse implementation, refunds/returns detail, or analytics warehouse schema.

## Required Artifact

Create `artifacts/data-model.md` containing:

1. Modeling principles and source-of-truth rules.
2. Conceptual ER diagram grouped by module.
3. Entity catalog: owner, purpose, key relationships, important fields/value objects, classification, lifecycle, and phase.
4. Aggregate/invariant table.
5. Canonical status/state-transition tables for account/session, product/variant/image, cart/checkout/reservation, order/payment/webhook/outbox, review/ticket/notification, and knowledge/index/conversation records.
6. Unique constraints and index candidates with query/use-case reason.
7. Optimistic concurrency and transaction-boundary candidates.
8. Immutable snapshot rules for order lines, addresses, totals, currency, product names/SKUs, and payment references.
9. Soft-delete/hard-delete/archive policy by entity class.
10. Retention classification with unresolved legal durations clearly deferred.
11. Migration assumptions: provider neutrality, review, rollback/forward-fix, backup before risky change, and no automatic production migration.
12. Data-model risks and unresolved decisions.

## Questions To Answer

- Which conceptual entities are MVP versus later-phase placeholders?
- Which records are mutable, append-only, immutable snapshots, derived, or rebuildable?
- What concurrency token is conceptually required for stock, cart, order, payment, and knowledge publishing?
- Which uniqueness rules prevent duplicates/replay?
- Which data must be retained, anonymized, archived, or securely deleted?
- Should Address remain User-owned while AddressSnapshot is Checkout/Order-owned?

## Contract And Safety Rules

| Concern | Rule |
| --- | --- |
| Architecture | Entity/rule ownership follows Packet 02; Infrastructure owns physical persistence choices. |
| Data | Money includes currency; UTC timestamps; stock cannot go below zero; card data is never modeled. |
| API/UI | Public DTOs need not mirror entities; private fields are not exposed by default. |
| Security | Classify PII, payment, audit, internal, and AI-indexed data; apply minimization. |
| Logging/Audit | Audit/history records store necessary metadata, not secrets or full sensitive snapshots. |
| Failure | Transactions, concurrency, idempotency, snapshots, and outbox prevent partial/duplicate corruption. |

## Required Diagrams

- Mermaid ER diagram grouped by domain ownership.
- Mermaid order/payment/inventory lifecycle diagram.
- Mermaid data lifecycle diagram: active, archived/expired, anonymized/deleted, retained audit.

## Step-By-Step Instructions

1. Build the entity inventory from all approved roadmap phases and mark owning phase.
2. Assign every entity to the Packet 02 source-of-truth owner.
3. Define relationships and aggregate boundaries without physical schema detail.
4. Document invariants and legal status transitions.
5. Identify unique/index candidates from API queries and replay risks.
6. Define concurrency and transaction candidates for money, stock, state, and publishing.
7. Define immutable snapshots and derived/rebuildable records.
8. Classify deletion, archival, and retention behavior; mark legal durations blocked where unknown.
9. Add provider-neutral migration assumptions and high-risk review rules.
10. Draw diagrams, cross-check with API/domain contracts, and request approval.

## Expected File Areas

- Create/update only `artifacts/data-model.md` and Packet 05 status.

## Verification And Evidence

| Check | Evidence |
| --- | --- |
| Entity coverage | Every roadmap entity is included or explicitly deferred. |
| Ownership | Every entity has one owner matching Packet 02. |
| Lifecycle | High-risk records have legal state transitions. |
| Integrity | Unique, index, concurrency, snapshot, and transaction candidates exist. |
| Privacy | Classification/deletion/retention are explicit. |
| Scope | No schema, migration, or source file changed. |

```powershell
rg -n "Owner|Classification|Lifecycle|Unique|Index|Concurrency|Snapshot|Retention" docs/implementation-guides/phase-0/artifacts/data-model.md
rg -n "card number|CVV|plaintext token|raw secret" docs/implementation-guides/phase-0/artifacts/data-model.md
git diff --check
git diff --name-only
```

Build/test evidence may reuse Packet 01; record documentation-only validation.

## Review Checklist

- Conceptual and physical design are not confused.
- Entity ownership matches domain contracts.
- No raw card/secret model exists.
- Stock, payment, order, webhook, refresh token, and outbox integrity are addressed.
- Snapshots protect historical order truth.
- Unknown retention law is visible as a blocker.

## Acceptance Criteria

- Artifact and three diagrams exist.
- Every entity has owner, classification, lifecycle, and phase.
- High-risk invariants and concurrency/idempotency constraints are documented.
- Data, architecture, and security reviewers approve the model.
- No database implementation was created.

## Manual Review And Stop Conditions

Stop if an entity has multiple owners, state transitions permit impossible money/stock outcomes, retention requires unknown legal interpretation, or a provider-specific choice would leak into Core.

## Rollback

Revert only the data-model artifact and Packet 05 status.

## Handoff

Packets 06-09 consume approved identity entities, classifications, lifecycle rules, constraints, snapshots, and migration assumptions.

## Copy-Ready Execution Prompt

```text
Execute Phase 0 Packet 05 from docs/implementation-guides/phase-0/05-data-model-and-lifecycles.md after Packet 02 approval. Create only artifacts/data-model.md and the allowed status update. Produce the conceptual entity/aggregate/lifecycle/index/concurrency/snapshot/retention design and required diagrams, remain provider-neutral, never model card data or secrets, verify traceability and scope, and stop for human review on ownership or legal-retention conflicts.
```
