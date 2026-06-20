# Packet 04: API Contract Baseline

## Packet Metadata

| Field | Value |
| --- | --- |
| Status | Not Started |
| Depends On | Packet 02 Approved |
| Produces | `artifacts/api-contract-baseline.md` |
| Human Reviewers | API owner, solution architect, security reviewer |

## Objective

Define canonical REST conventions and a complete MVP endpoint inventory so later agents can implement vertical slices without inventing routes, response/error shapes, authorization, ownership, idempotency, or pagination rules.

## Read First

- Approved Packet 01-02 artifacts
- `docs/roadmap/phase-0.md`, section 6
- `docs/architecture/system-design.md`, canonical API conventions
- Phase 1-4 roadmap API sections for compatibility, without expanding Phase 0 scope
- `docs/architecture/security-scalability-review.md`

## In Scope

- `/api/v1` route rules, nouns/commands, JSON casing, timestamps, money, IDs, status codes, Problem Details, pagination/filter/sort, correlation, idempotency, concurrency, compatibility, deprecation, auth/ownership metadata, rate-limit expectations, and endpoint inventory.

## Out Of Scope

- Controllers, OpenAPI generation, DTO code, validators, middleware, package selection, or implementation.
- Final provider-specific payment/webhook payloads.
- Trusting client prices, discounts, stock, shipping, tax, roles, or authorization context.

## Required Artifact

Create `artifacts/api-contract-baseline.md` containing:

1. General REST and JSON conventions.
2. Resource/command route naming and versioning/deprecation policy.
3. Request, single-resource, collection, async, and no-content response shapes.
4. Problem Details catalog for validation, authentication, forbidden, not found, conflict/concurrency, business rule, rate limit, and unexpected errors.
5. Pagination/filtering/sorting rules and allowlist requirements.
6. `X-Correlation-Id`, `Idempotency-Key`, conditional/concurrency, content type, and webhook-signature header rules.
7. Idempotency ownership, request fingerprint, replay result, retention, conflict, and retry semantics.
8. Endpoint inventory grouped by Auth/User, Product, Cart, Checkout, Inventory, Order, Payment/Webhook, Admin, Customer Experience, and AI/RAG.
9. For every endpoint: method, route, purpose, auth, role/permission, ownership, request, response, status codes, idempotency, rate-limit class, audit, and phase.
10. Compatibility rules for additive/breaking changes and internal versus public contracts.

## Questions To Answer

- Which ID format is exposed publicly, or should implementation keep IDs opaque?
- Which business commands use POST subresources versus PATCH state changes?
- Which endpoints require idempotency and who stores the result?
- When should ownership denial return 403 versus privacy-preserving 404?
- How are list filter/sort allowlists documented per endpoint?
- Which webhook fields remain provider-specific and deferred?

## Contract And Safety Rules

| Concern | Rule |
| --- | --- |
| Architecture | API is presentation; business rules live in Core/application services. |
| Data | Money is decimal amount plus currency; timestamps are UTC ISO 8601; IDs are opaque. |
| API/UI | `/api/v1`, camelCase JSON, Problem Details, max page size 100. |
| Security | Every endpoint declares auth, permission, ownership, abuse/rate, and audit behavior. |
| Logging/Audit | Correlation IDs are length-limited/untrusted; never log auth headers, cookies, tokens, secrets, full addresses, or sensitive bodies. |
| Failure | Safe retries require idempotency; external callbacks require signature/replay/duplicate rules. |

## Required Diagrams

- Mermaid request lifecycle: request, correlation, auth, validation, authorization/ownership, application service, persistence/outbox, response.
- Mermaid idempotent command flow including first request, replay, key conflict, and retained result.
- Mermaid API version/deprecation decision flow.

## Step-By-Step Instructions

1. Normalize existing routes across Phase 0-4 documents.
2. Define canonical request/response/error/header conventions.
3. Define opaque ID, money, timestamp, pagination, filter, and sort behavior.
4. Define idempotency and concurrency semantics before endpoint inventory.
5. Inventory endpoints by module and phase; do not pull enterprise features into MVP.
6. Add auth, permission, ownership, audit, rate-limit class, and failure metadata to every endpoint.
7. Mark provider-specific payloads and unresolved contracts explicitly.
8. Add examples that contain no real secrets or personal data.
9. Draw and explain required diagrams.
10. Run consistency/security review and request approval.

## Expected File Areas

- Create/update only `artifacts/api-contract-baseline.md` and Packet 04 status.

## Verification And Evidence

| Check | Evidence |
| --- | --- |
| Module coverage | Endpoint groups cover all approved modules. |
| Security metadata | Every endpoint has auth/ownership/audit/rate entries. |
| Error consistency | Problem Details fields and status mappings are canonical. |
| Retry safety | Risky commands and callbacks have idempotency/replay rules. |
| Compatibility | Additive/breaking/deprecation policy exists. |
| Scope | Only Phase 0 docs changed. |

```powershell
rg -n "/api/v1|Problem Details|X-Correlation-Id|Idempotency-Key|ownership|rate" docs/implementation-guides/phase-0/artifacts/api-contract-baseline.md
rg -n "client.*price|client.*stock|authorization header|cookie|token" docs/implementation-guides/phase-0/artifacts/api-contract-baseline.md
git diff --check
git diff --name-only
```

Build/test evidence may reuse Packet 01. Record documentation validation as the packet test evidence.

## Review Checklist

- Route names and API version are consistent.
- Every endpoint declares authorization and ownership.
- Errors do not leak resource existence or internals.
- Pagination/filter/sort are bounded and allowlisted.
- Idempotency covers money/stock/risky retries.
- Examples contain no sensitive data.

## Acceptance Criteria

- Artifact and three diagrams exist.
- Endpoint inventory has no missing required metadata.
- API, architecture, and security reviewers approve conventions.
- Later phases can add detail without changing the baseline silently.
- No application/API code changed.

## Manual Review And Stop Conditions

Stop if route conventions conflict across approved phases, a sensitive endpoint lacks an owner, or idempotency storage/ownership is undefined. Record the conflict in Packet 09 candidates.

## Rollback

Revert only the API artifact and Packet 04 status.

## Handoff

Packets 06-07 consume the auth/error/audit metadata; Packet 09 records the approved API baseline as a cross-phase contract.

## Copy-Ready Execution Prompt

```text
Execute Phase 0 Packet 04 from docs/implementation-guides/phase-0/04-api-contract-baseline.md after Packet 02 approval. Create only artifacts/api-contract-baseline.md and the allowed status update. Normalize all approved MVP routes, define Problem Details and header/idempotency/versioning rules, build the complete endpoint metadata inventory, add the three diagrams, run security/scope checks, and stop on contract conflicts rather than inventing behavior.
```

