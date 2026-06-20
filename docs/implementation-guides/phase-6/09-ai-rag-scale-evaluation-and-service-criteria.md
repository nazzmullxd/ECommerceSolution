# Packet 09: AI/RAG Scale, Evaluation, And Service Criteria

## Packet Metadata

| Field | Value |
| --- | --- |
| Status | Blocked - PHASE0/PHASE1/PHASE2/PHASE3/PHASE4/PHASE5-GATE |
| Depends On | Packets 01-02 and 07-08 Approved; stable Phase 4/5 AI controls; `P6-AI-001` |
| Decision Type | Planning first; AI/RAG remains monolith module by default |
| Manual Review | AI security/safety, privacy, knowledge owners, Identity/access, SRE, evaluation, provider/cost, data/index, product/support and architecture review |
| Produces | AI/RAG scale/evaluation/provider-resilience/service-trigger plan and `evidence/09-completion.md` |

## Objective

Scale Phase 4 AI/RAG through index lifecycle, retrieval optimization, evaluation operations, injection/poisoning monitoring, rate limit and cost controls, and provider resilience while preserving authorization/grounding and defining strict evidence for any future separate service review.

## Read First And Prerequisites

- Approved Phase 4 source governance/index/retrieval/conversation/evaluation/disable evidence and Phase 5 security/telemetry/rates/incidents/cost.
- Packet 01 measured AI demand and Packet 02 monolith optimization evidence.
- Packet 07 enterprise access and Packet 08 knowledge/index events.

## In Scope

- Index capacity/sharding/partition-by-source-or-visibility only after measurement, generation/version/alias/atomic activation/reindex/backfill/compaction/deletion/retention/rebuild/DR.
- Retrieval latency/top-k/filter/rerank/cache-safe-embedding/query budgets while pre-filtering authorization and current-source checks.
- Knowledge approval/expiry/poison review at scale; evaluation dataset/version/coverage/reviewer/automated and human regression operations.
- Prompt injection/private extraction/unauthorized retrieval/unsafe answer/fallback/provider/rate/cost/latency monitoring and disable/incidents.
- Provider abstraction timeout/retry/circuit/fallback/model/version/dimension/token/cost/quota/residency/secret/IAM and migration comparison; no paid call.
- Separate-service criteria and scorecard inputs only.

## Out Of Scope

- Bedrock/OpenSearch/provider SDK/call/account, model download, private customer/order/payment/support indexing, autonomous actions, sensitive personalization, weaker grounding for latency, automatic threshold tuning, AI service implementation or cloud resource.

## Ownership And Boundaries

- **Owns:** AI/Search derived indexes, knowledge/retrieval/assistant/evaluation metadata, prompt/provider/ranking/config versions and safety decisions.
- **Reads:** approved source snapshots/events and server-derived Identity scope.
- **Must not own/mutate:** source Catalog/Knowledge/Identity/Order/Payment/Support data or business actions.
- Remains a modular-monolith module behind Phase 4 interfaces until Packet 10 and separate ADR approve otherwise.

## Evidence Required

| Evidence | Required Proof |
| --- | --- |
| Entry Evidence | Phase 4 safety/evaluation/disable plus Phase 5 telemetry/rate/incident/cost controls stable. |
| Measured Need Evidence | Query/index volume, latency, queue lag, provider quotas/cost, release/team isolation pain and trends. |
| Design Evidence | Index/evaluation/security/provider/resilience/privacy/cost/migration/rollback and service criteria. |
| Monolith-First Evidence | Optimized module production latency/cost/safety/load/reindex/provider-failure results. |
| Extraction/Cloud Evidence | Dedicated owner/on-call, scorecard/ADR, data/API/event boundary, cost/quotas and fallback. |

## Data And Contract Design

Track index/source/model/embedding/prompt/retrieval/evaluation/config generation and compatibility. Index partitions never broaden visibility; unknown partition/role fails closed. Provider change requires dimension/version/reindex, golden/adversarial evaluation and signed enablement.

Cost record stores operation/provider/model/version/token-or-unit estimate/latency/status/fallback/rate-card version/currency with privacy-safe IDs, not full prompt/source. Evaluation case/result remains versioned/synthetic or approved public-safe.

## API And UI Contract

- Existing search/customer/admin assistant APIs remain compatible, bounded and feature-flagged; no direct provider endpoint.
- Responses preserve citations/fallback/restricted-topic/disabled/rate/stale states. No performance optimization removes source checks.
- Admin index/evaluation/provider config actions require enterprise permission, approval/audit and no secret display.

## Likely File Areas To Inspect

- Phase 4 AI Core/Infrastructure/API/Web/Tests/evidence and Phase 5 telemetry/rates/incidents/cost.
- Planning only now; no provider/cloud files.

## Architecture And Prohibited Dependencies

- Authorization before retrieval and approved current source grounding are invariant under scale.
- Cache only approved non-private embeddings/query-independent derived artifacts with version/invalidation; never cache authorization decision/private answer across users.
- Provider fallback cannot answer from memory or bypass policy.
- Extraction must not copy all source data or create shared DB writes; it remains unapproved.

## Database Changes And Migrations

No migration now. Future index metadata/partition/checkpoint/evaluation schema requires provider/privacy/retention/capacity/migration review. Vector provider/schema remains behind Infrastructure and Packet 13 gate.

## Security, Logging, Audit, And Failure Rules

- Monitor direct/indirect injection, source poisoning, role/source probing, private extraction, cross-conversation, unsafe reports, fallback changes and cost/rate abuse with bounded privacy-safe signals.
- Audit knowledge/index/prompt/model/provider/threshold/permission/disable/reindex/retrieval-log access.
- Provider/index/evaluation/log DB failure uses approved fallback/disable and never fabricates answer.
- No full prompt/answer/source/vector/token/secret/private customer data in logs/evidence.

## Planning Or Implementation Steps

1. Validate Phase 4/5 gates and collect production AI traffic/latency/fallback/safety/index/provider/cost trends.
2. Define index lifecycle/capacity/reindex/partition/retention/rebuild/DR triggers and authorization tests.
3. Define retrieval/cache/rate/timeout/circuit/provider fallback optimizations and budgets.
4. Operationalize evaluation dataset ownership/coverage/reviewer/release regression and unsafe incident process.
5. Define provider comparison/migration/reindex/cost/quota/residency/IAM/secrets without integration.
6. Try/measure bounded monolith improvements under full adversarial regression.
7. Feed persistent evidence to Packet 10; otherwise record `Remain In Monolith`.

## Test And Review Matrix

| Review | Required Cases |
| --- | --- |
| Index | Full/incremental/reindex/partial/alias switch/delete/retention/rebuild/partition and visibility isolation. |
| Retrieval | Load/latency/top-k/filter/rerank/cache/fallback with role/source/stale/current checks. |
| Adversarial | Direct/indirect injection, poisoning, cross-role/user, private extraction, cost/rate abuse and provider manipulation. |
| Evaluation | Dataset version/coverage/drift, human review, hard safety gates, provider/prompt/index regression. |
| Provider | Timeout/rate/quota/malformed/version/dimension/cost spike/failover/disable/reindex/rollback. |
| Service criteria | Independent load/deploy/team/data/operations/cost and safe monolith fallback evidence. |

## Verification Commands

```powershell
rg -n "SearchDocument|EmbeddingRecord|Retrieval|Evaluation|PromptInjection|Provider|Confidence|Fallback|FeatureEnabled|CostEstimate" docs/ai-rag docs/implementation-guides/phase-4 docs/implementation-guides/phase-6
git diff --check
git status --short
```

## Acceptance Criteria

- AI scale plan preserves pre-retrieval authorization, approved-source grounding, hard evaluation gates and privacy.
- Index/provider/evaluation/cost/rate/resilience changes are versioned/reversible and require re-evaluation.
- Monolith-first options and production measurements precede any service/cloud review.
- No provider/model/cloud/private index/action/service implementation is authorized.

## Manual Approval Gates

Manual approval is mandatory for index partition/lifecycle, evaluation thresholds/dataset/review, prompt/model/provider/version, cost/rate/quota/residency/IAM/secrets, cache, safety monitoring/disable/re-enable and service-extraction ADR.

## Stop Conditions

Stop for absent Phase 4 safety evidence, weaker auth/grounding for speed, private indexing/cache, full prompt logs, automatic threshold weakening, provider/cloud call, no cost/quota/owner, action agency, service implementation or extraction without Packet 10 ADR.

## Rollback / Forward-Fix And Handoff

Planning-only. Future changes use versioned generation/alias/feature flags; disable provider/feature and return to keyword/support/manual path on regression, preserve prior index/config, rerun full evaluation. Hand measured service criteria to Packet 10 and AWS triggers to 13.

## Copy-Ready Constrained Prompt

```text
Execute Phase 6 Packet 09 as planning only after prior gates, Packets 01-02/07-08 and P6-AI-001. Update only AI/RAG index lifecycle/retrieval/evaluation/injection monitoring/rate/cost/provider resilience/privacy/migration/rollback/service-criteria documentation and evidence. Do not write provider/service/cloud code, download models, index private data, weaken authorization/grounding/evaluation, log prompts, or enable paid services. Measure monolith-first options and default Remain In Monolith. Add evidence/09-completion.md with AI security/cost/operations review.
```
