# Packet 10: Product Search, Filter, And Sort Experience

## Packet Metadata

| Field | Value |
| --- | --- |
| Status | Blocked - PHASE0/PHASE1/PHASE2-GATE |
| Depends On | Packets 01-05 Approved as applicable; Phase 2 Catalog read flow Approved; `P3-SEARCH-001` |
| Layer Ownership | Catalog owns keyword query/results; Search UI consumes public Catalog/attribute/rating/image projections |
| Risk Review | Pre-code and post-test Catalog, database/performance, security, accessibility, and UX review |
| Produces | Basic product discovery experience and Phase 4 handoff, `evidence/10-completion.md` |

## Objective

Turn the Phase 2 catalog listing into a predictable keyword/filter/sort experience with bounded queries, URL-stable navigation, accessible empty/error states, and a clean future handoff to semantic search.

## Read First And Prerequisites

- Approved Phase 2 Catalog visibility/pagination/query contracts and Phase 3 variant/attribute/image/review public projections.
- Packet 01 and resolved `P3-SEARCH-001` searchable fields, ranking, facets, sorts, page limits, and aggregate freshness.
- Phase 3 roadmap search/filter, security, failure, logging, testing, and UI sections.

## In Scope

- Normalized basic keyword search over approved public product fields, combined with allowlisted category/attribute/availability/price filters.
- Allowlisted sort choices such as relevance, name, displayed price, newest, and approved rating if freshness supports it.
- Stable cursor or offset pagination per Phase 0/2 contract, bounded page size/query length/filter count/date/range, and deterministic tie-breaker.
- Search/filter/sort MVC/Razor UX, query-string state, reset/empty/error behavior, and Phase 4 semantic adapter handoff contract.

## Out Of Scope

- Embeddings, Vector DB, AI/LLM, personalization, typo correction service, search analytics profiling, hidden/draft products, unrestricted SQL field/sort input, or cloud OpenSearch.

## Ownership And Read-Only Contracts

- **Owns:** Catalog remains owner of keyword search criteria, visibility, ranking contract, result projection, and pagination semantics.
- **Reads:** ProductVariant/Attribute, active ProductImage, approved Rating aggregate, and sellability display through approved read projections.
- **Must not own/mutate:** Review records, Inventory truth, ProductImage lifecycle, Cart, customer profile/history, or Phase 4 vector documents.

## API Contract

| Route | Auth | Behavior |
| --- | --- | --- |
| `GET /api/v1/products?q=&category=&attribute.&minPrice=&maxPrice=&availability=&sort=&page=&pageSize=` | Anonymous | Extends the single Phase 2 public listing contract with normalized allowlisted criteria. |

Return normalized criteria, items, pagination metadata, and optional facet counts only when budgets allow. Unknown filter/sort, invalid range, excessive query/page/filter count returns validation Problem Details. Internal ranking scores and hidden fields are not exposed unless a later compatibility decision approves them.

## UI Acceptance States

- Initial browse, typing/submitting, loading, results, no results with clear filter reset, invalid filter/range, dependency/timeout failure, retry, and pagination navigation.
- Query/filter/sort state is represented in the URL and survives refresh/back/forward without duplicating parameters.
- Filters use appropriate controls, removable active-filter summary, visible labels, keyboard operation, focus management, announced result count, and responsive layout without overlap.

## Likely File Areas To Inspect

- Catalog Core query criteria/projections/interfaces and public visibility rules.
- Infrastructure query/index/query-plan implementations and approved rating/image/attribute joins or composition.
- API product endpoint/DTO/OpenAPI and Web product list/filter/view-model/controller/styles/scripts.
- Unit, integration, API injection/limits, query-performance, UI navigation/accessibility, and regression tests.

## Architecture And Prohibited Dependencies

- Extend one public Catalog route/query contract; do not create a competing search source in Phase 3.
- Parse to typed criteria, allowlist server-side, parameterize provider queries, and apply deterministic tie-breaker/pagination.
- Client controls only presentation/request criteria; server enforces visibility and computes displayed results. Search availability is not checkout stock truth.

## Database Changes And Migrations

No table is expected. Add indexes only from captured query plans/timings and Catalog/database review. Do not add a vector column/index. Document database-provider-specific text matching and portable fallback before implementation.

## Security, Logging, And Failure Rules

- Enforce active/public product/variant/attribute/image visibility before result/facet composition.
- Bound and normalize query text without constructing SQL; apply rate limit, timeout, cancellation, and page/filter caps.
- Log normalized query metadata or one-way privacy-safe query fingerprint, filter/sort names, duration/result count/status/correlation; do not log raw sensitive queries, customer identity/history, SQL, or full responses.
- Dependency/rating/facet failure follows approved partial-versus-total behavior; never show hidden products or claim stock/price certainty when unavailable.

## Implementation Steps

1. Approve fields/matching/case/culture, visibility, filters/sorts, relevance/tie-breaker, facet/rating freshness, limits, database behavior, exact URL/UI, and Phase 4 contract.
2. Define typed query criteria/result/facet DTOs and compatibility tests.
3. Implement bounded Catalog query and compose only approved public projections.
4. Capture plans/timings; propose reviewed indexes only if evidence requires them.
5. Implement exact API and accessible URL-driven MVC search/filter/sort UI.
6. Define Phase 4 adapter boundary preserving typed filters and product IDs; add no AI code.
7. Add visibility/injection/limit/pagination/determinism/performance/failure/UI/accessibility tests.
8. Run checks, record evidence, and obtain post-test review.

## Test Matrix

| Test | Required Cases |
| --- | --- |
| Matching | Approved fields/case/culture; blank/long/special text; deterministic tie-breaker. |
| Filters/sorts | Every allowlisted option/composition; unknown/repeated/invalid ranges rejected. |
| Visibility | Hidden/deleted/private/unavailable policy; private attributes/images/reviews absent. |
| Pagination | Stable boundaries; max page; URL round-trip/back/forward; no duplicate/skip under fixture. |
| Safety/performance | Injection payload as data; timeout/cancellation/rate; query budget/plan/index evidence. |
| UI/accessibility | Browse/loading/results/empty/reset/error/pagination, labels/focus/keyboard/result announcement/responsive. |

## Verification Commands

```powershell
dotnet build ECommerceSolution.slnx
dotnet test ECommerceSolution.slnx --no-build
rg -n "ProductSearch|SearchCriteria|pageSize|sort|filter|IQueryable|FromSql|Vector|Embedding" ECommerce.Core ECommerce.Infrastructure ECommerce.API ECommerce.Web ECommerce.Tests
git diff --check
git status --short
```

## Acceptance Criteria

- One public Catalog contract provides deterministic, bounded, allowlisted keyword/filter/sort/pagination behavior.
- Active/public visibility is enforced and read-only projections do not become price/stock/review/image truth.
- URL-driven UI covers all states, works by keyboard/screen reader, and is responsive.
- Query safety/performance evidence and tests pass; no embeddings/vector/cloud dependency is introduced.
- Phase 4 handoff preserves typed criteria and stable product IDs without changing Phase 3 behavior.

## Security Stop Conditions

Stop for raw sort/filter SQL, unbounded query/page/facet, hidden product/data leakage, customer profiling, search as checkout truth, unstable pagination, sensitive raw-query logging, unmeasured index, vector/AI implementation, or inaccessible controls.

## Rollback / Forward-Fix And Handoff

Disable new criteria/UI while retaining the Phase 2 browse route compatibility. Remove only evidence-approved unused indexes through reviewed migration. Hand typed query, visibility, product-ID, filter, and evaluation fixture contracts to Phase 4 semantic search design.

## Copy-Ready Coding Prompt

```text
Execute Phase 3 Packet 10 only after all prerequisite gates, approved Phase 2 Catalog, relevant Phase 3 public projections, P3-SEARCH-001, and pre-code review. Extend only the existing public Catalog query with typed bounded keyword/filter/sort/pagination, evidence-based indexes, exact API, accessible URL-driven MVC UI, tests, and a contract-only Phase 4 handoff. Never add raw SQL input, hidden data, customer profiling, checkout truth, embeddings/vector/AI, or cloud services. Add evidence/10-completion.md, run checks, and obtain post-test review.
```
