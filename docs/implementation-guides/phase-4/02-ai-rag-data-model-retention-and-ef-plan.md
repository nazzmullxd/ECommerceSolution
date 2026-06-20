# Packet 02: AI/RAG Data Model, Retention, And EF Plan

## Packet Metadata

| Field | Value |
| --- | --- |
| Status | Blocked - PHASE0/PHASE1/PHASE2/PHASE3-GATE |
| Depends On | Packet 01 Approved; `P4-DATA-001`, `P4-RET-001` |
| Layer Ownership | AI/Search owns derived records; source modules retain truth |
| Manual Review | Pre-code database, privacy/security, AI architecture, audit, and retention-owner review; post-migration review |
| Produces | Approved entity/mapping/migration plan and `evidence/02-completion.md` |

## Objective

Define the provider-neutral relational model, lifecycle, constraints, indexes, concurrency, retention, and deletion/rebuild behavior for AI/RAG derived data without storing prohibited private content or making the index a source of truth.

## Read First And Prerequisites

- Packet 01 approved contracts/classification/threat model.
- Approved Phase 1 database/audit/identity and Phase 2 Outbox conventions.
- Approved source-module IDs/version/visibility/change-event contracts from Catalog and Knowledge Management.

## In Scope

- `SearchDocument`, `EmbeddingRecord`, `KnowledgeArticle`, `KnowledgeArticleVersion`, `RetrievalLog`, `AssistantConversation`, `AssistantMessage`, `AssistantFeedback`, and optional `AiProviderRequestLog` only if metadata cannot live in operational logs.
- Ownership, statuses, invariants, relationships, indexes, uniqueness, concurrency, source hashes, lineage, lifecycle, retention category, soft/hard deletion, and rebuild rules.
- EF Core mapping and migration plan after local provider/DbContext boundary is approved.

## Out Of Scope

- Provider-specific vector columns/extensions, full prompt/source archives, private customer/order/payment/support data, binary files, model output as policy truth, endpoints/UI, provider code, automatic legal retention values, or cloud schema.

## Ownership And Read-Only Contracts

- **Owns:** knowledge authoring/version metadata, derived chunks/embeddings, retrieval/conversation/message/feedback metadata, provider-call metadata, and retention state.
- **Reads:** stable source IDs/versions/classification from source modules and actor IDs from Identity.
- **Must not own/mutate:** Product/policy source text outside KnowledgeArticle, identity/role, Catalog visibility, Support/Order/Payment data, AuditLog, or Outbox records.

## Data Contract

Required decisions:

- `SearchDocument`: source type/ID/version, deterministic chunk ID/number, title/section, bounded retrievable content, content hash, visibility, approval/publication snapshot, active/stale/deleted/expiry/reindex state, index generation, timestamps/version.
- `EmbeddingRecord`: document ID, deterministic provider/model/version, dimension, vector or provider-neutral serialized local representation only after approval, vector hash/status/error code/timestamps. Never expose vector publicly.
- `KnowledgeArticle/Version`: slug/category/owner/visibility/lifecycle/current published version, immutable published content, author/reviewer/approval/rejection/archive/delete fields, source hash/version/concurrency.
- `RetrievalLog`: operation/assistant type, correlation ID, pseudonymous actor reference/role snapshot, query/content hashes and safe category, source IDs/versions/scores, confidence/fallback/safety reasons, latency, dataset/config versions, timestamps/retention state.
- `Conversation/Message/Feedback`: opaque ownership, assistant type, role snapshot, bounded sanitized preview only when approved, hashes, citations/confidence/fallback, feedback reason/status, timestamps/expiry/version.

Store score/source collections using a structured relational child model or reviewed JSON mapping, not ad hoc delimited strings.

## API Contract

No endpoint implementation. Define persistence DTO boundaries so public responses never expose internal visibility, raw content/vector, reviewer notes, query/prompt hashes, provider diagnostics, or retention metadata.

## UI Acceptance States

No UI implementation. Document lifecycle/retention states that later UI must render: draft, pending review, approved, published, rejected, archived, deleted, stale/reindex required, indexing failed, conversation expired, and feedback accepted/reviewed.

## Likely File Areas To Inspect

- Core entities/value objects/statuses/repository contracts and existing audit/clock/ID conventions.
- Infrastructure DbContext boundary, mappings/migrations, provider limitations, and Tests migration/constraint fixtures.
- Do not select exact paths/provider types until execution inspection.

## Architecture And Prohibited Dependencies

- Core entities do not contain EF attributes, provider vector types, JSON provider APIs, or cloud model names.
- SearchDocument/EmbeddingRecord are disposable rebuildable projections; no commerce or authorization decision reads them as truth.
- Source version completeness and active generation prevent mixed old/new chunk retrieval.
- Conversation ownership is explicit and query-enforced; predictable IDs and shared anonymous histories are prohibited.

## Database Changes And Migrations

Plan one reviewed migration only after reconciliation. Required constraints include unique source/type/version/chunk/generation, unique article/version and non-deleted slug, unique document/provider/model embedding, owner-scoped conversation/message/feedback keys, status/visibility/active indexes, retrieval timestamp/outcome indexes, and concurrency tokens. Provider portability and query plans must be documented. Destructive migration or vector extension requires separate approval.

## Security, Logging, Audit, And Failure Rules

- Derived content retains source classification; a hash/embedding does not downgrade it.
- Bounded previews require a privacy purpose and sanitizer; otherwise store only hash/metadata. Do not retain raw system prompts or provider requests/responses.
- Legal/privacy owner must approve retention and deletion rules. Until then, minimize and block automatic hard deletion that could violate evidence needs.
- Knowledge lifecycle and visibility changes use mandatory audit; retrieval logs are operational AI evidence, not a replacement for AuditLog.
- Failed mapping/migration stops startup/packet; partial index records remain inactive and rebuildable.

## Implementation Steps

1. Resolve database/DbContext/vector-representation and retention decisions with named owners.
2. Define lifecycle/state transition tables and data dictionary with prohibited fields.
3. Define entities/value objects and structured child records/provider-neutral interfaces.
4. Design constraints/indexes/concurrency/source-lineage/generation/retention behavior.
5. Review query patterns and provider portability; create migration only after approval.
6. Add mapping, lifecycle, constraint, owner-scope, migration fresh/upgrade, and privacy tests.
7. Scan schema/log/evidence for prohibited content; obtain post-migration review.

## Test Matrix

| Test | Required Cases |
| --- | --- |
| Lifecycle | Every legal/illegal status transition; published immutable; stale/archive/delete behavior. |
| Constraints | Duplicate chunk/article version/slug/embedding/feedback; ownership and foreign keys. |
| Generation | Partial version inactive; one active complete generation; old generation retained until switch. |
| Privacy | Prohibited private/payment/support/prompt/provider fields absent; bounded previews sanitized. |
| Retention | Expiry selection, legal hold/blocked hard delete if defined, source deletion propagation. |
| Migration | Fresh apply, representative upgrade, indexes/concurrency/provider portability. |

## Verification Commands

```powershell
dotnet build ECommerceSolution.slnx
dotnet test ECommerceSolution.slnx --no-build
rg -n "SearchDocument|EmbeddingRecord|KnowledgeArticle|RetrievalLog|AssistantConversation|AssistantMessage|AssistantFeedback|AiProviderRequestLog" ECommerce.Core ECommerce.Infrastructure ECommerce.Tests
git diff --check
git diff --stat
git status --short
```

## Acceptance Criteria

- All required entities have explicit owner, lineage, lifecycle, retention category, constraints, indexes, concurrency, and rebuild/deletion rules.
- Schema contains no private customer/order/payment/support data, secrets, raw prompts/provider payloads, or provider-specific dependency without approval.
- Active-generation completeness prevents partial/stale retrieval and owner scopes prevent cross-user conversation access.
- Fresh/upgrade migration and privacy/constraint tests pass with named reviews/evidence.

## Security Stop Conditions

Stop for unresolved database/retention owner, raw sensitive prompts/responses, private commerce/support fields, provider vector type in Core, broad anonymous conversation ownership, mixed partial generations, missing source lineage, weak uniqueness/concurrency, destructive migration, or unreviewed provider extension.

## Rollback / Forward-Fix And Handoff

Back up the local database before migration. Disable AI writes/retrieval and retain source/old generation on rollback. After data exists, forward-fix schema/status and rebuild derived indexes rather than deleting source/audit evidence. Hand approved entities/mappings to Packets 04-06 and 11-12.

## Copy-Ready Coding Prompt

```text
Execute Phase 4 Packet 02 only after all prior gates, Packet 01, P4-DATA-001/P4-RET-001, and pre-code database/privacy review. Inspect the actual DbContext/provider first; implement only provider-neutral AI/RAG entities, mappings, reviewed migration, lifecycle/constraints/indexes/concurrency/retention behavior, and tests. Store no private customer/order/payment/support data, secrets, raw prompts/provider payloads, or provider-specific vector type in Core. Keep derived data rebuildable and inactive until a complete generation exists. Add evidence/02-completion.md and obtain post-migration review.
```
