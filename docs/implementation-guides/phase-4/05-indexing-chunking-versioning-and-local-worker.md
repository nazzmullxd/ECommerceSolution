# Packet 05: Indexing, Chunking, Versioning, And Local Worker

## Packet Metadata

| Field | Value |
| --- | --- |
| Status | Blocked - PHASE0/PHASE1/PHASE2/PHASE3-GATE |
| Depends On | Packets 01-04 Approved; Phase 2 Outbox Approved; `P4-CHUNK-001` |
| Layer Ownership | Indexing owns derived generations; Catalog/Knowledge own sources |
| Manual Review | Mandatory pre-code and post-test AI security, source-owner, database, Outbox/worker, and operations review |
| Produces | Versioned local indexing pipeline and `evidence/05-completion.md` |

## Objective

Build an idempotent local indexing pipeline that reads only approved source projections, chunks deterministically, records lineage/hashes, generates derived embeddings, and atomically activates a complete source generation while preserving the previous valid generation on failure.

## Read First And Prerequisites

- Packets 01-04 contracts/entities/providers/publication and poisoning controls.
- Approved Phase 2 Outbox worker lease/retry/deduplication/dead-letter rules.
- Approved Phase 3 public Catalog visibility/version/image/attribute/search handoff.

## In Scope

- Product and KnowledgeArticle indexing requests/events, idempotency/deduplication, source snapshot query, deterministic chunking/metadata/source IDs/hashes, embedding generation, completeness checks, generation switch, deactivation, reindex, retry, dead-letter/manual retry, and local worker.
- Product fields limited to approved public title/description/category/brand/tags/public attributes/variant options/price range/public image alt metadata/availability policy.
- Knowledge fields limited to one approved published version and its classification/visibility/expiry.

## Out Of Scope

- Private customer/order/payment/support data, arbitrary table indexing, unapproved article/draft, file/URL ingestion, cloud queue/vector provider, parallel distributed workers beyond approved local lease, automatic source correction, or customer endpoint.

## Ownership And Read-Only Contracts

- **Owns:** indexing job/request metadata, derived SearchDocument/EmbeddingRecord generations, active-generation pointer/state, retry/dead-letter evidence.
- **Reads:** immutable source snapshots through Catalog/Knowledge application interfaces and deterministic providers.
- **Must not own/mutate:** Product/Variant/Inventory/policy/article source state, source visibility, Identity, Outbox source rows, or business events.

## Data Contract

- Stable source ID: versioned source type/source ID/source version/chunk number/generation, encoded without leaking private internals.
- Chunk metadata: title/section, deterministic order, content/source hash, classification/visibility/publication/expiry snapshot, source URL/route only if safe, provider/algorithm version.
- Chunking uses normalized heading/paragraph/sentence boundaries, bounded size, no orphan overlap leakage, deterministic ordering, and no split that loses policy conditions/citation context.
- Activation requires expected chunk count, all hashes/metadata/embeddings valid, source version still current/approved, and a transaction/atomic pointer switch.

## API Contract

| Route | Permission | Behavior |
| --- | --- | --- |
| `POST /api/v1/admin/search/indexing/sources/{sourceType}/{sourceId}` | `search.index` | Idempotently request index/reindex of one approved source. |
| `POST /api/v1/admin/search/reindex-requests` | `search.reindex` | Bounded source type/version scope; no "all database" arbitrary query. |
| `GET /api/v1/admin/search/indexing/jobs/{jobId}` | `search.index.read` | Safe status/count/error code; no source content/vector. |

Prefer `202 Accepted` with job ID for worker processing. Same key/same fingerprint returns same job; different fingerprint conflicts.

## UI Acceptance States

- Admin indexing status: queued, processing, complete/current, no changes/deduplicated, stale/reindex required, partial failed but prior active retained, dead-letter/manual review, source no longer approved, forbidden, rate limited.
- Never display vector/raw chunk/private content in general admin UI. Retry requires reason and permission.

## Likely File Areas To Inspect

- Core indexing commands/jobs/source snapshot interfaces/events/result contracts.
- Infrastructure Outbox/worker lease, source adapters, chunker, providers, persistence/generation switch.
- API/Web admin status/reindex slices and Tests unit/integration/concurrency/security/failure/UI.

## Architecture And Prohibited Dependencies

- Source modules expose approved read snapshots/events; Indexing never queries their tables directly or changes their state.
- Reuse Phase 2 Outbox/worker patterns; do not create a second queue framework.
- Worker rechecks authorization-independent source publication/visibility/current version before activation.
- A partially built generation is never queryable. Retrieval selects one complete active generation.

## Database Changes And Migrations

Use Packet 02 schema. Add job/generation fields/table only if Packet 02 explicitly approved or is amended through review. Capture indexes/query plans for pending jobs, source generations, active documents, and embedding lookup. No vector extension.

## Security, Logging, Audit, And Failure Rules

- Validate source type/ID through allowlists; no arbitrary reflection/table/type. Recheck source classification and poisoning review status.
- Log job/source IDs/version/generation, counts/hashes prefix/provider version/duration/status/error code/correlation/causation; never raw source/chunk/vector/prompt/private data.
- Audit reindex/deactivation/manual retry and source visibility/index-policy changes.
- Embedding/chunk/persistence/worker crash/lease expiry/source changed/partial activation failure preserves previous valid generation and records retry/dead-letter.

## Implementation Steps

1. Obtain mandatory pre-code approval for source snapshots/events, chunk algorithm/limits, source ID, generation activation, worker/idempotency/retry/dead-letter, routes/UI.
2. Define indexing job/generation lifecycle and typed source snapshot adapters.
3. Implement deterministic chunker/metadata/hash/source IDs with policy-context tests.
4. Implement product and knowledge adapters using approved fields only.
5. Implement embedding/persistence into inactive generation and atomic completeness/current-source switch.
6. Integrate approved Outbox/local worker lease/retry/dedupe/dead-letter/manual retry.
7. Add exact APIs/UI, authorization/audit/rate controls.
8. Add adversarial poisoning/source-race/duplicate/partial/crash/concurrency/privacy tests and obtain post-test review.

## Test Matrix

| Test | Required Cases |
| --- | --- |
| Chunking | Deterministic boundaries/order/hash; headings/tables/long paragraph/empty; policy condition remains cited. |
| Scope | Public Catalog fields only; published Knowledge only; private/draft/stale/deleted excluded. |
| Idempotency | Duplicate request/event; same source/hash no-op; changed version new generation. |
| Atomicity | Embedding/chunk/DB/crash/source-race/partial failure keeps old generation; complete switch only. |
| Worker | Lease expiry, bounded retry/backoff, dead-letter, manual retry permission/audit, cancellation. |
| Adversarial/privacy | Injection text remains data; hidden/encoded poison rejected/reviewed; no raw content/vector logs/API. |

## Verification Commands

```powershell
dotnet build ECommerceSolution.slnx
dotnet test ECommerceSolution.slnx --no-build
rg -n "Indexing|SearchDocument|EmbeddingRecord|Generation|Chunk|Outbox|BackgroundService|SourceType" ECommerce.Core ECommerce.Infrastructure ECommerce.API ECommerce.Web ECommerce.Tests
git diff --check
git status --short
```

## Acceptance Criteria

- Product/knowledge sources are read through approved snapshots and only allowed published/public-or-authorized fields are indexed.
- Chunk/source/hash/generation behavior is deterministic, idempotent, versioned, and atomically activated.
- Every partial/race/crash failure preserves the last complete valid generation; retries/dead-letter are bounded/audited.
- Adversarial, privacy, API/UI, worker, concurrency, and query/migration tests plus mandatory reviews pass.
- No private data, arbitrary source, second queue, cloud/vector extension, or customer endpoint is added.

## Security Stop Conditions

Stop for direct source-table query, arbitrary source type/table, draft/private/payment/support indexing, incomplete generation retrievable, non-versioned algorithm, client source/visibility control, unbounded reindex/retry, duplicate worker framework, raw content/vector logs, missing poison review, or missing human approval.

## Rollback / Forward-Fix And Handoff

Pause worker and keep current active generation. Mark defective generation inactive, preserve job evidence, fix/version algorithm, and reindex; never mutate source to repair an index. Hand complete generation/query contracts to Packet 06 and operational metrics to Packet 12.

## Copy-Ready Coding Prompt

```text
Execute Phase 4 Packet 05 only after all prior gates, Packets 01-04, approved Phase 2 Outbox/Phase 3 Catalog handoff, P4-CHUNK-001, and mandatory pre-code review. Implement only typed source adapters, deterministic chunk/hash/source IDs, inactive generation build, atomic complete activation, existing-Outbox local worker/retry/dead-letter, scoped APIs/UI, audit, and adversarial/failure tests. Never index private/draft/stale data, query source tables directly, expose raw chunks/vectors, create a second queue, or add cloud/vector services. Add evidence/05-completion.md and obtain post-test review.
```
