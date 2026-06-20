# Packet 08: Similar Product Recommendations

## Packet Metadata

| Field | Value |
| --- | --- |
| Status | Blocked - PHASE0/PHASE1/PHASE2/PHASE3-GATE |
| Depends On | Packets 06-07 Approved; approved Catalog projection; `P4-RECO-001`, `P4-CUSTOMER-ENABLEMENT-GATE` |
| Layer Ownership | Recommendations owns product-to-product similarity ranking only |
| Manual Review | Mandatory pre-code and post-test Catalog, AI security/privacy, ranking, performance, accessibility, and UX review; manual security approval before enablement |
| Produces | Non-personalized similar-product slice and `evidence/08-completion.md` |

## Objective

Recommend safe public alternatives using only current product characteristics and deterministic product-to-product similarity, with strict exclusions and an empty/same-category fallback rather than sensitive profiling.

## Read First And Prerequisites

- Approved Product/Variant/Attribute/Image/Rating visibility and search/index contracts.
- Packets 06-07 retrieval/ranking/failure/evaluation patterns.
- Resolved similarity feature/weight/fallback/availability/rate/UI decisions.

## In Scope

- Inputs: public product ID; category, public attributes/options/tags, approved displayed price range, public text embedding similarity, and optional approved rating aggregate as a bounded tie-breaker.
- Exclude current, hidden, draft, deleted, private, stale, unauthorized, and unavailable products under approved Catalog policy.
- Deterministic feature normalization/weights/dedup/tie-break/max results, current Catalog hydration, same-category or empty fallback, API/UI, rate/performance/evaluation/adversarial tests.

## Out Of Scope

- Customer/user ID input, click/order/wishlist/support/profile/payment/location/device history, collaborative filtering, sensitive categories, paid/sponsored ranking, popularity without an approved privacy-safe metric, cross-sell business rule, or AI-generated explanation.

## Ownership And Read-Only Contracts

- **Owns:** versioned similarity feature contract, score composition, exclusions, fallback and evaluation.
- **Reads:** one public source product and current public Catalog candidate projections/derived index.
- **Must not own/mutate:** Catalog/Product/price/stock/rating truth, customer identity/history, Cart/Checkout, or ranking policy from clients.

## Data Contract

Internal candidate features must identify source/version and normalize missing values deterministically. Public output uses existing Product summary plus optional safe reason tags such as shared category/attribute; never expose embedding/internal weights/sensitive inferred traits.

## API Contract

`GET /api/v1/products/{productId}/similar?pageSize={bounded}`

- Anonymous/customer/admin receive the same public non-personalized result for the same source/catalog/index version.
- Validate public source visibility, bounded result count, rate/timeout/cancellation.
- Missing/hidden source follows Catalog not-found policy; no safe match returns empty list; vector failure uses approved same-category deterministic fallback.

## UI Acceptance States

- Product detail: loading with stable dimensions, recommendations, empty/no safe match, source unavailable, fallback mode if user-facing wording approved, dependency failure, rate limited, feature disabled.
- Cards reuse Phase 3 product accessibility/availability/image/price behavior and never imply personal surveillance.

## Likely File Areas To Inspect

- Core recommendation query/feature/ranking contracts and Catalog read interfaces.
- Infrastructure candidate/retrieval composition, API endpoint/OpenAPI, Web product-detail component.
- Tests golden scenarios, privacy/visibility/adversarial/performance/UI/accessibility.

## Architecture And Prohibited Dependencies

- No customer context is accepted or resolved. Authentication may affect only generic rate/audit handling, never ranking.
- Current Catalog hydration is required before response; index features cannot revive hidden/stale products.
- The recommendation result is presentation only and never affects price, stock, discounts, checkout, or authorization.

## Database Changes And Migrations

None expected. No behavior/profile table, recommendation cache containing customer IDs, vector extension, or new source truth.

## Security, Logging, Audit, And Failure Rules

- Adversarial tests cover hidden product IDs, client feature/weight injection, scraping/rate abuse, malicious indexed text, sensitive inferred attribute, stale index, and candidate enumeration.
- Log source product ID, ranking version, candidate/result count, fallback/status/duration/correlation; no customer identity/history, raw feature/vector/text, or hidden candidate IDs.
- Failure returns empty/approved deterministic fallback, not invented product or provider-generated explanation.

## Implementation Steps

1. Obtain mandatory pre-code approval for features/weights/ties/exclusions/availability/fallback/result/rate/performance/API/UI.
2. Define versioned feature extraction and deterministic similarity contract.
3. Retrieve prefiltered candidates and compute/merge scores; hydrate/recheck current Catalog.
4. Implement bounded API and accessible product-detail states.
5. Add golden 10+ scenarios, missing feature/tie/fallback, visibility/privacy, stale/failure/performance, and adversarial tests.
6. Record evaluation, obtain post-test review, and obtain customer-enablement approval or keep disabled.

## Test Matrix

| Test | Required Cases |
| --- | --- |
| Similarity | Category/attribute/price/tag/text combinations; missing fields; ties; deterministic order. |
| Exclusions | Self, hidden/draft/deleted/private/stale/unavailable policy/current Catalog change. |
| Privacy | Same result across anonymous/users; no identity/history features/storage/logs. |
| Failure | Vector/index failure, source missing, no match, Catalog dependency, timeout/rate. |
| Adversarial | Hidden ID, client weight/feature, poisoned text, sensitive inference, enumeration/scrape. |
| UI/accessibility | Loading/results/empty/fallback/error/disabled; product-card keyboard/reflow/labels. |

## Verification Commands

```powershell
dotnet build ECommerceSolution.slnx
dotnet test ECommerceSolution.slnx --no-build
rg -n "Similar|Recommendation|CustomerId|OrderHistory|Wishlist|Embedding|RankingVersion" ECommerce.Core ECommerce.Infrastructure ECommerce.API ECommerce.Web ECommerce.Tests
git diff --check
git status --short
```

## Acceptance Criteria

- Ranking uses approved public product-to-product features only and is deterministic/versioned.
- Current Catalog exclusions prevent hidden/stale/unauthorized/unavailable products according to approved rule.
- No identity/history/profiling/sponsored/checkout dependency exists; fallback is same-category deterministic or empty.
- API/UI/performance/privacy/adversarial/evaluation tests and required reviews pass.
- Customer feature remains disabled without signed enablement gate.

## Security Stop Conditions

Stop for user/customer input or history, sensitive inference, hidden product leak, client weights/features, provider explanation, popularity without approved source, stale index response, recommendation affecting commerce truth, missing adversarial tests, or unsigned enablement.

## Rollback / Forward-Fix And Handoff

Disable the recommendation component/route and show no recommendations; product detail remains functional. Version feature/ranking changes and rerun Packet 11 evaluation before enablement. Hand evaluation outcomes to Packet 11.

## Copy-Ready Coding Prompt

```text
Execute Phase 4 Packet 08 only after prior gates, Packets 06-07, P4-RECO-001, and mandatory pre-code review. Implement only deterministic product-to-product similarity from approved public Catalog features, strict exclusions/current hydration, bounded API, accessible UI, same-category-or-empty fallback, and privacy/performance/adversarial tests. Never accept/store user history, infer sensitive traits, trust client weights, expose hidden products/internal vectors, affect checkout, add provider/cloud/schema, or enable customers without the signed gate. Add evidence/08-completion.md and obtain reviews.
```
