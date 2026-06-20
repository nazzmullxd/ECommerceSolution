# Packet 07: Hybrid Semantic Product Search

## Packet Metadata

| Field | Value |
| --- | --- |
| Status | Blocked - PHASE0/PHASE1/PHASE2/PHASE3-GATE |
| Depends On | Packet 06 Approved; Phase 3 Catalog/Search handoff Approved; `P4-SEARCH-001`, `P4-CUSTOMER-ENABLEMENT-GATE` |
| Layer Ownership | Catalog owns product truth; AI/Search owns derived ranking only |
| Manual Review | Mandatory pre-code and post-test Catalog, AI security, database/performance, abuse, accessibility, and UX review; manual security approval before enablement |
| Produces | Local hybrid product-search vertical slice and `evidence/07-completion.md` |

## Objective

Add intent-oriented product discovery by safely merging Phase 3 keyword results with deterministic local semantic candidates while preserving Catalog visibility, server-owned filters/ranking, stable pagination, fallback, and checkout authority.

## Read First And Prerequisites

- Approved Phase 3 keyword/search/filter/sort/product projection and Catalog visibility contracts.
- Packets 01-06 indexing/retrieval/provider/evaluation contracts.
- Resolved ranking/availability/page/rate decisions; customer feature remains disabled pending gate.

## In Scope

- Semantic query API and MVC/Razor experience composing keyword and vector candidates through Packet 06.
- Public product source fields only; server-side category/attribute/price/availability filters, deterministic score normalization/merge/dedup/tie-break/pagination.
- Hidden/draft/deleted/private exclusion; unavailable products excluded or clearly represented exactly as Catalog approves.
- Keyword-only degradation, bounded anonymous/auth rate limits, query performance/evaluation/adversarial tests, and result reason metadata safe for users.

## Out Of Scope

- Personalization, behavior/order/profile history, sponsored/business-secret weights, client ranking weights, checkout price/stock truth, SKU exposure unless already public, model provider, cloud search, typo service, or automatic customer enablement.

## Ownership And Read-Only Contracts

- **Owns:** semantic query orchestration, derived score components/ranking version, result dedupe, evaluation metrics.
- **Reads:** public Catalog search/result projection and complete public SearchDocument generation.
- **Must not own/mutate:** Product/Variant/Image/Rating/Inventory/price, Catalog visibility, customer behavior, Cart/Checkout, or source index content.

## Data Contract

Input uses query plus approved typed Phase 3 filters/page; client cannot submit score weights/source IDs/provider. Output returns existing public product projection, pagination, search mode (`Hybrid`/`KeywordFallback`), safe reason tags, and no raw embeddings/internal scores unless explicitly approved for admin diagnostics.

Stable pagination requires deterministic final tie-breaker and documented behavior when Catalog/index changes between pages; use the approved Phase 3 paging model rather than inventing another.

## API Contract

`POST /api/v1/search/semantic` may be used as roadmap-approved contract, or an approved compatibility ADR may extend the Phase 3 product-search route. Decide one before coding.

- Auth optional; anonymous and authenticated requests use only public product scope.
- Enforce query length, filter/sort/page allowlists, max page size/candidates, rate limit, timeout/cancellation.
- Empty/no relevant result returns successful empty list; vector failure can return keyword fallback with metadata; total failure returns safe Problem Details/fallback state.

## UI Acceptance States

- Initial, submitting/loading, hybrid results, keyword-fallback label, no results/reset filters, invalid filter, rate limited/retry-after, index stale/unavailable, total failure, feature disabled, and accessible pagination.
- Preserve URL/query/filter state where Phase 3 UX requires; result labels must not claim certainty or personalization.

## Likely File Areas To Inspect

- Core search orchestration/result contracts and approved Catalog query interface.
- Infrastructure retrieval adapters/query performance, API endpoint/OpenAPI/rate policy, Web search UI/view models.
- Tests evaluation fixtures, API/visibility/adversarial/load/UI/accessibility.

## Architecture And Prohibited Dependencies

- Catalog projection is returned only after fresh visibility checks; SearchDocument is candidate data, never final product truth.
- Final price/stock/purchasability remains Catalog/Cart/Checkout server logic. Search cache/index cannot authorize purchase.
- Semantic failure degrades to existing keyword search; it must not make product visibility broader.

## Database Changes And Migrations

No schema expected. Add index only from captured plan/timing and source-owner/database approval. No vector extension/provider schema.

## Security, Logging, Audit, And Failure Rules

- Apply public/active/current filters before candidate return and again when hydrating Catalog projection.
- Adversarial inputs include injection/encoded text, hidden product terms/IDs, overlong/filter explosion, score/source/provider manipulation, enumeration, and resource exhaustion.
- Log query hash/category, filter names, result count/mode/duration/fallback/rate/correlation, not raw sensitive query, customer identity/history, vector, hidden product, or full result.
- Vector/index/provider failure uses bounded keyword fallback; stale/missing Catalog item is omitted, never fabricated.

## Implementation Steps

1. Obtain mandatory pre-code approval for route compatibility, fields, merge formula/tie-break, availability, pagination, rate/query/performance budget, response/UI wording.
2. Implement typed request/orchestration using Packet 06 public scope and Phase 3 keyword query.
3. Merge/dedupe/rank candidates deterministically, then hydrate/filter through current Catalog projection.
4. Implement exact API, rate/timeout/cancellation, Problem Details/fallback metadata.
5. Implement accessible search UI states and preserve approved query/filter navigation.
6. Add golden relevance, visibility, paging, performance, fallback, and adversarial tests.
7. Record evaluation; obtain post-test and separate manual customer-enablement security approval or keep disabled.

## Test Matrix

| Test | Required Cases |
| --- | --- |
| Relevance | Roadmap 20 product questions plus exact/synonym/intent/no-match; deterministic ranking. |
| Visibility | Hidden/draft/deleted/private/stale/old generation omitted; Catalog changed after index. |
| Filters/paging | Every allowlist/range; invalid/repeated/excessive; stable dedupe/tie/page boundaries. |
| Failure/performance | Vector/index timeout/malformed score -> keyword; keyword failure; budget/rate/cancel. |
| Adversarial | Prompt/query injection, hidden ID enumeration, client weight/source/provider, filter bomb, scraping pattern. |
| UI/accessibility | Loading/results/fallback/empty/error/rate/disabled, keyboard/focus/labels/announcements/reflow. |

## Verification Commands

```powershell
dotnet build ECommerceSolution.slnx
dotnet test ECommerceSolution.slnx --no-build
rg -n "search/semantic|Hybrid|KeywordFallback|SearchCriteria|Visibility|pageSize|RateLimit" ECommerce.Core ECommerce.Infrastructure ECommerce.API ECommerce.Web ECommerce.Tests
git diff --check
git status --short
```

## Acceptance Criteria

- Hybrid ranking is deterministic/server-owned and composes the approved keyword/Catalog contract.
- Current Catalog visibility excludes hidden/deleted/draft/private products and checkout truth is untouched.
- Keyword degradation, limits/rates/performance, exact API/UI/accessibility states, relevance and adversarial tests pass.
- No personal data/profile, provider/cloud, client weights, vector schema, or hidden diagnostics are added.
- Customer surface remains disabled unless named manual security/product reviewers approve the evaluation gate.

## Security Stop Conditions

Stop for missing human review, SearchDocument returned as product truth, hidden product leak, client ranking/source/provider control, personal profiling, unstable/unbounded query, semantic failure broadening results, checkout price/stock reliance, missing adversarial evaluation, or enablement without signed gate.

## Rollback / Forward-Fix And Handoff

Disable semantic mode and preserve the Phase 3 keyword route/UI. Version ranking/index changes and rerun Packet 11 evaluation before re-enable. Hand safe product similarity candidates and evaluation fixtures to Packet 08.

## Copy-Ready Coding Prompt

```text
Execute Phase 4 Packet 07 only after prior gates, Packet 06, approved Phase 3 Catalog/Search, P4-SEARCH-001, and mandatory pre-code review. Implement only deterministic hybrid orchestration over public prefiltered candidates plus current Catalog hydration, bounded API/rate/timeout, keyword fallback, accessible UI, relevance/performance/adversarial tests, and evidence. Never use private behavior, expose hidden products/internal scores, trust client weights, make search checkout truth, add provider/cloud/schema, or enable customers without P4-CUSTOMER-ENABLEMENT-GATE. Add evidence/07-completion.md and obtain reviews.
```
