# Packet 04: Knowledge Lifecycle, Authorization, And Poisoning Defense

## Packet Metadata

| Field | Value |
| --- | --- |
| Status | Blocked - PHASE0/PHASE1/PHASE2/PHASE3-GATE |
| Depends On | Packets 01-02 Approved; Phase 1 permissions/audit Approved; `P4-KB-001` |
| Layer Ownership | Knowledge Management owns article/version truth; Indexing owns derived copies |
| Manual Review | Mandatory pre-code and post-test security, policy/legal, content governance, authorization, audit, and UX review |
| Produces | Safe knowledge authoring/publishing vertical slice and `evidence/04-completion.md` |

## Objective

Implement a governed draft-to-publication lifecycle that prevents self-approval, untrusted ingestion, stale or partial publication, role misclassification, and retrieval poisoning.

## Read First And Prerequisites

- Packets 01-02 classification/entity/lifecycle/threat contracts.
- Approved Identity permissions, mandatory audit transaction, concurrency, Problem Details, and Phase 2 Outbox.
- Approved public/support/admin policy owners and retention/review cadence.

## In Scope

- Draft, submit/review, approve, publish, reject, archive, restore-if-approved, and soft-delete commands with legal transitions.
- Immutable published versions; new edits create new draft versions while the prior complete published version remains active.
- Explicit author/reviewer separation, role/permission and visibility checks, source ownership, reason/concurrency/idempotency, mandatory audit, and review expiry.
- Typed text entry only for MVP, strict size/format/link rules, no file/URL ingestion, content hash, injection/poisoning review checklist, and stale/review-due behavior.
- Admin API and accessible MVC/Razor workflow with preview clearly separated from retrievable publication.

## Out Of Scope

- Arbitrary upload, remote URL fetch, web crawl, Office/PDF parsing, rich executable HTML, AI-written/auto-approved content, bulk import, provider ingestion, private support/order/customer data, or publication without human review.

## Ownership And Read-Only Contracts

- **Owns:** KnowledgeArticle/Version source content, classification, visibility, lifecycle, owner, author/reviewer, published version, review/expiry, and governance history.
- **Reads:** authenticated actor/permissions and audit/outbox interfaces.
- **Must not own/mutate:** SearchDocument/Embedding active generation directly except via indexing request/event; Identity roles; Catalog/policy outside managed articles; Support/Order/Payment data.

## Data Contract

- Published version is immutable and separately identified from current draft.
- Classification and visibility changes require a new review; increasing or decreasing visibility cannot bypass publication.
- Content is bounded plain text/approved Markdown subset rendered safely; embedded instructions remain content and cannot alter system behavior.
- Store source/content hash, reviewer/reason/timestamps, next review/expiry, concurrency version, and audit references.

## API Contract

| Route | Permission | Behavior |
| --- | --- | --- |
| `GET /api/v1/admin/knowledge/articles` | `knowledge.read` | Bounded filters by lifecycle/visibility/category/owner; no unauthorized content. |
| `POST /api/v1/admin/knowledge/articles` | `knowledge.author` | Creates draft with explicit classification/visibility. |
| `PUT /api/v1/admin/knowledge/articles/{articleId}/versions/{versionId}` | Author + permitted scope | Updates draft only with expected version. |
| `POST /api/v1/admin/knowledge/articles/{articleId}/review-submissions` | `knowledge.author` | Submits immutable candidate for independent review. |
| `POST /api/v1/admin/knowledge/articles/{articleId}/review-decisions` | `knowledge.review` | Approve/reject with reason; author cannot approve own version. |
| `POST /api/v1/admin/knowledge/articles/{articleId}/publications` | `knowledge.publish` | Publishes approved version and emits indexing request idempotently. |
| `POST /api/v1/admin/knowledge/articles/{articleId}/archives` | `knowledge.publish` | Archives active source and emits deactivation. |
| `DELETE /api/v1/admin/knowledge/articles/{articleId}` | `knowledge.delete` | Soft delete with reason/audit and index deactivation. |

Do not use generic status patch. Exact permission names must be approved through Phase 1.

## UI Acceptance States

- List/detail/editor: loading, empty, draft saved, validation, stale conflict, review pending, rejected reason, approved not yet indexed, published/current version, review due/stale, archived/deleted, unauthorized, audit/index request failure.
- Preview must say "not published/not retrievable"; source visibility and forbidden-data warning are always visible.
- Review/approve/publish/archive/delete use accessible confirmation, reason, expected version, and cannot be performed by hiding server checks.

## Likely File Areas To Inspect

- Core knowledge entities/lifecycle/policies/commands/events/permissions.
- Infrastructure mappings/repositories/audit/outbox transaction and API/Web admin slices.
- Tests lifecycle/authorization/BOLA/poisoning/rendering/concurrency/audit/UI/accessibility.

## Architecture And Prohibited Dependencies

- Knowledge Management is source of truth; Indexing consumes a versioned event/query and cannot alter source state.
- Author, reviewer, and publisher separation follows approved policy; SuperAdmin is not an automatic bypass.
- No raw HTML execution, URL fetch, file parser, auto-publish, AI approval, or direct index write from controllers.

## Database Changes And Migrations

Use Packet 02 mappings. Add only reviewed constraints/indexes if lifecycle execution proves missing. No separate table/migration without reconciling Packet 02 and owner review.

## Security, Logging, Audit, And Failure Rules

- Treat content as attacker-controlled. Review direct/indirect prompt injection, secret/PII inclusion, misleading policy, hidden text/links, role misclassification, and duplicate source.
- Mandatory audit and source mutation must be atomic; publication plus Outbox indexing request follows approved transaction rules.
- Log IDs/version/status/visibility/action/result/reason code/correlation, not full article/draft/reason text.
- Audit/indexing failure leaves prior published version active and new version unpublished. Expired/stale source is excluded by Retrieval until reapproved.

## Implementation Steps

1. Obtain mandatory pre-code approval for permission matrix, separation of duties, format/size/link rules, review cadence/expiry, exact lifecycle/routes/UI.
2. Implement lifecycle invariants and permission/ownership policies.
3. Implement owner/reviewer-scoped persistence with concurrency and existing mappings.
4. Implement mandatory audit plus idempotent publication/deactivation Outbox requests.
5. Implement exact APIs and accessible author/reviewer/publisher UI.
6. Add lifecycle, self-approval, cross-role, poisoning corpus, stale/source leak, concurrency, audit/outbox failure, API/UI/accessibility tests.
7. Run checks, record evidence, and obtain mandatory post-test review.

## Test Matrix

| Test | Required Cases |
| --- | --- |
| Lifecycle | Legal/illegal transitions; immutable published version; edit creates draft; prior version remains. |
| Authorization | Author/reviewer/publisher split; self-approval denied; visibility scope; BOLA; SuperAdmin explicit. |
| Poisoning | Direct/indirect instructions, hidden/encoded text, fake citations, secret/PII/payment/support content, misleading links. |
| Consistency | Publication/audit/outbox atomicity; duplicate command/idempotency; stale conflict. |
| Staleness | Review due/expired/archive/delete excluded; old version retained until complete replacement. |
| UI/privacy | Every state; safe rendering; no full content/reason in logs; keyboard/focus/errors. |

## Verification Commands

```powershell
dotnet build ECommerceSolution.slnx
dotnet test ECommerceSolution.slnx --no-build
rg -n "KnowledgeArticle|PendingReview|Published|knowledge\.|Outbox|Audit|Html.Raw|HttpClient" ECommerce.Core ECommerce.Infrastructure ECommerce.API ECommerce.Web ECommerce.Tests
git diff --check
git status --short
```

## Acceptance Criteria

- Only independently reviewed, approved, published, current content can request indexing; author cannot self-approve.
- Prior valid published version survives failed replacement/indexing, and stale/archive/delete deactivates retrieval safely.
- Permission/BOLA/concurrency/idempotency/audit/rendering/poisoning/adversarial tests and API/UI states pass.
- No upload/URL fetch/AI approval/private data/raw HTML/provider behavior is introduced.
- Mandatory pre-code and post-test reviews/evidence exist.

## Security Stop Conditions

Stop for missing human review, self-approval, generic status patch, direct controller index write, untrusted upload/URL, private/payment/support content, unsafe HTML, classification downgrade without review, publication without atomic audit/outbox, stale source retrievable, sensitive logging, or unresolved poisoning finding.

## Rollback / Forward-Fix And Handoff

Disable authoring/publication commands while keeping the last approved version. Archive/deactivate a compromised source through audited commands; preserve evidence for incident review. Forward-fix with a new version and full review, then hand approved version events to Packet 05.

## Copy-Ready Coding Prompt

```text
Execute Phase 4 Packet 04 only after all prior gates, Packets 01-02, P4-KB-001, and mandatory pre-code governance/security review. Implement only the governed KnowledgeArticle/version lifecycle, exact permissions/APIs/UI, immutable publication, stale handling, audit/outbox indexing requests, safe text rendering, and adversarial poisoning tests. Add no upload/URL fetch/AI approval/private data/direct index write/generic status patch. Preserve the last valid version on failure, add evidence/04-completion.md, and obtain mandatory post-test review.
```
