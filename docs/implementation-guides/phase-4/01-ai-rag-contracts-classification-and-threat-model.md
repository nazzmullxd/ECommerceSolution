# Packet 01: AI/RAG Contracts, Classification, And Threat Model

## Packet Metadata

| Field | Value |
| --- | --- |
| Status | Blocked - PHASE0/PHASE1/PHASE2/PHASE3-GATE |
| Depends On | Approved Phase 0-3 contracts; `P4-GATES`, `P4-VIS-001` |
| Layer Ownership | AI/Search Core contracts and policy vocabulary; no provider implementation |
| Manual Review | Pre-code architecture, security/privacy, Identity, Catalog, Support, product, and legal/policy-owner review |
| Produces | Approved AI contract/threat baseline and `evidence/01-completion.md` |

## Objective

Freeze the allowed AI use cases, provider/retrieval boundaries, classification and visibility model, trust boundaries, threat mitigations, permission inventory, and fail-closed rules before any entity or provider code is created.

## Read First And Prerequisites

- Approved Phase 0 cross-phase contracts and threat model.
- Approved Phase 1 identity/permission/audit/rate contracts, Phase 2 Outbox, and Phase 3 Catalog/Search/Support/UI contracts.
- Phase 4 roadmap, RAG architecture, security review, and this package README.

## In Scope

- Framework-neutral contracts for completion, embeddings, vector/keyword search, indexing, retrieval, prompt construction, safety decision, conversation, feedback, and evaluation.
- Data classification, visibility lattice, role/permission mapping, endpoint purpose scopes, source-type allowlist, and approved/excluded use cases.
- Threat model for prompt injection, indirect injection, poisoning, sensitive disclosure, broken access control, cross-user conversations, excessive agency, model/provider failure, resource abuse, stale sources, and logging/privacy.
- Contract-level result/error/fallback/reason codes, correlation/causation, deterministic clock/ID abstractions where already approved.

## Out Of Scope

- Entities/migrations, provider algorithms, prompts, endpoints/UI, model/provider packages, AWS resources, customer enablement, business actions, private customer/order/payment/support retrieval, or exact numeric thresholds without evidence.

## Ownership And Read-Only Contracts

- **Owns:** AI use-case policy, AI-specific DTOs/interfaces, source classification vocabulary, retrieval scope decisions, safety outcomes, and threat/control register.
- **Reads:** identity/role/permission context and source-module public/authorized projections through approved interfaces.
- **Must not own/mutate:** identity/roles, Product visibility, Support/Order/Payment data, policy truth, AuditLog, Outbox, or source module entities.

## Data Contract

Define immutable contract shapes for `AiOperationContext`, `RetrievalScope`, `SearchQuery`, `SearchCandidate`, `RetrievedSource`, `GroundingDecision`, `AssistantResult`, `Citation`, `FallbackReason`, and provider results. Contracts must carry source/version/chunk/visibility/status metadata and safe diagnostics, never raw authorization tokens or client-provided roles.

Visibility is a set of explicit allowed levels, not numeric role comparison. Unknown role, permission, visibility, source type, status, or endpoint purpose is denied. Document exact permission codes through the Phase 1 change process.

## API Contract

No feature endpoint is implemented. Define shared Phase 4 response metadata and Problem Details extensions only:

- `correlationId`, operation type, fallback flag/reason, confidence when meaningful, and citations safe for that caller.
- Stable fallback reasons such as `NO_APPROVED_SOURCE`, `LOW_CONFIDENCE`, `RESTRICTED_TOPIC`, `FEATURE_DISABLED`, `PROVIDER_UNAVAILABLE`, `STALE_SOURCE`, and `RATE_LIMITED`.
- Clients cannot submit visibility, role, prompt template, provider/model, ranking weights, source allowlist, or confidence threshold.

## UI Acceptance States

Document shared states from the README and prohibit UI claims such as "verified" or "policy decision." Grounded answers show approved source labels; fallback/refusal clearly offers the allowed next action without pretending the AI checked an account/order.

## Likely File Areas To Inspect

- Existing Core conventions/result types/interfaces/permission constants and Phase 1-3 source contracts.
- Infrastructure provider/composition conventions, API Problem Details/OpenAPI, Web shared UI states, and Tests architecture/security areas.
- Exact paths must be chosen only after repository inspection during execution.

## Architecture And Prohibited Dependencies

- Core contracts contain no EF, HTTP, MVC, provider SDK, cloud, tokenizer, vector-database, or hosted-service dependency.
- Provider output is untrusted and cannot grant authorization, select visibility, execute a command, or declare itself grounded.
- Prompt text is not an authorization boundary. Pre-retrieval filtering and post-generation validation are separate defenses.
- No generic "ask the database" or tool/action interface exists in Phase 4.

## Database Changes And Migrations

None. Packet 02 owns the data model. Stop if implementation requires schema or provider package.

## Security, Logging, Audit, And Failure Rules

- Produce a threat table with asset, actor, trust boundary, attack, impact, preventive control, detective control, test, owner, and residual risk.
- Classify embeddings/indexed text as derived copies retaining the source's classification; embeddings are never assumed anonymous.
- Define banned log fields: raw sensitive prompt/source, private message, token/cookie/header, credentials, full customer/order/payment data, and provider secret.
- Define mandatory audit actions: knowledge lifecycle, visibility/threshold/provider/config changes, indexing/reindex, admin assistant, retrieval-log access, disable/enable.
- Every ambiguity or unavailable dependency fails closed with a typed fallback/error and correlation ID.

## Implementation Steps

1. Verify prerequisite approvals and reconcile exact Identity/Catalog/Support/Outbox contracts.
2. Inventory approved/excluded use cases and source types; map each endpoint purpose to users, visibility, permissions, and forbidden data.
3. Define framework-neutral interfaces/DTOs/result/fallback/citation/source metadata.
4. Build the data-classification and role/visibility matrix with deny-unknown behavior.
5. Threat-model direct/indirect injection, poisoning, leakage, BOLA, agency, abuse, stale data, provider and logging failures.
6. Define architecture dependency tests and contract test expectations.
7. Obtain named pre-code reviewers; record accepted decisions and unresolved blockers.
8. Implement contracts/tests only after approval, then run checks and obtain post-test review.

## Test Matrix

| Test | Required Cases |
| --- | --- |
| Architecture | Core has no forbidden provider/EF/HTTP/cloud dependencies. |
| Scope | Every endpoint purpose maps to allowed source types/visibility; unknown denied. |
| Contracts | Deterministic result/fallback/citation metadata; client cannot inject role/provider/threshold. |
| Classification | Public through SuperAdminOnly; CustomerPrivate/PaymentSensitive excluded. |
| Threats | Direct/indirect injection, poisoning, leakage, BOLA, excessive agency, abuse, stale/provider/log failure mapped to tests. |
| Privacy | Banned fields absent from DTOs/log contracts/evidence. |

## Verification Commands

```powershell
dotnet build ECommerceSolution.slnx
dotnet test ECommerceSolution.slnx --no-build
rg -n "IAi|IEmbedding|IVector|IRetrieval|Visibility|FallbackReason|CustomerPrivate|PaymentSensitive" ECommerce.Core ECommerce.Infrastructure ECommerce.API ECommerce.Web ECommerce.Tests
git diff --check
git diff --stat
git status --short
```

## Acceptance Criteria

- Approved/excluded use cases, source types, visibility, permissions, ownership, and no-private-index rules are unambiguous.
- Provider/retrieval/prompt/safety/conversation/evaluation contracts are framework-neutral and deny unknown input.
- Threat model covers OWASP LLM-style and application risks with controls, adversarial tests, owners, and residual risk.
- No endpoint/entity/provider/AWS behavior is implemented and all named reviews/evidence exist.

## Security Stop Conditions

Stop for missing prerequisite approval, numeric role hierarchy, retrieve-then-filter design, private/order/payment/support indexing, provider output as auth/grounding proof, model actions, broad database tool, sensitive logging, unresolved high/critical threat, or unapproved permission/source type.

## Rollback / Forward-Fix And Handoff

Contracts are additive drafts until approved. If a later conflict appears, pause dependent packets, version the contract/ADR, and rerun contract tests; do not silently reinterpret visibility. Hand approved contracts/threat controls to Packets 02-12.

## Copy-Ready Coding Prompt

```text
Execute Phase 4 Packet 01 only after PHASE0/PHASE1/PHASE2/PHASE3-GATE and pre-code reviewers approve P4-VIS-001. Inspect the repository, then add only framework-neutral AI/RAG contracts, classification/visibility/use-case policy, threat controls, architecture/contract tests, and evidence. Add no entities, migrations, endpoints, providers, SDKs, models, cloud calls, private-data indexing, or action tools. Deny unknown roles/visibility/source types and stop on any authorization, privacy, or high-risk ambiguity. Add evidence/01-completion.md and obtain post-test review.
```
