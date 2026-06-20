# Packet 11: Feedback, Evaluation, Release Gates, And Disable Switch

## Packet Metadata

| Field | Value |
| --- | --- |
| Status | Blocked - PHASE0/PHASE1/PHASE2/PHASE3-GATE |
| Depends On | Packets 06-10 Approved; `P4-RET-001`, `P4-EVAL-001`, `P4-CUSTOMER-ENABLEMENT-GATE` |
| Layer Ownership | Evaluation owns datasets/metrics/release evidence; feature owners own fixes |
| Manual Review | Mandatory pre-code and post-evaluation AI security, privacy, policy/legal, product, Support, accessibility, operations, and source-owner review |
| Produces | Versioned evaluation/feedback workflow, release decision, disable switch, `evidence/11-completion.md` |

## Objective

Create the evidence system that measures retrieval, grounding, authorization, prompt-injection resistance, fallback quality, privacy, latency, and cost estimates, then enforce human release/disable decisions for each Phase 4 feature.

## Read First And Prerequisites

- Packet 01 threat/control register and Packets 06-10 outcome/config/citation contracts.
- Roadmap/RAG minimum evaluation set and approved privacy/retention/audit rules.
- Named owners for product relevance, public policy, internal knowledge, security, privacy, Support, and enable/disable incidents.

## In Scope

- Versioned synthetic/public-safe golden and adversarial datasets, expected source IDs/visibility/fallback/forbidden behavior, reviewer workflow, feedback reason codes and triage.
- Metrics: retrieval recall/precision/ranking, source correctness/freshness, claim grounding/citation, unsupported/hallucinated answer, forbidden decision, unauthorized disclosure, prompt-injection success, appropriate fallback, user/reviewer feedback, latency, rate/failure, and future token/cost estimate.
- Per-feature release thresholds, non-negotiable zero-tolerance safety gates, confidence/ranking/prompt/provider/dataset version comparison, regression runs, signed enable/disable decision.
- Independent default-off disable switches for semantic search, recommendations, customer assistant, admin assistant, indexing, and provider calls; fail closed on missing/invalid config.

## Out Of Scope

- Production user data, hidden chain-of-thought, automatic model/threshold optimization, self-approval by coding agent, online A/B testing, paid evaluator/model, cloud feature flag, production launch, or weakening a safety threshold to pass a failing build.

## Ownership And Read-Only Contracts

- **Owns:** evaluation case/version, expected outcomes, run/result/reviewer status, metric definitions, release thresholds, feedback review, feature enablement record, disable state/reason.
- **Reads:** privacy-safe retrieval/assistant metadata and approved source fixtures/config versions.
- **Must not own/mutate:** source truth, production conversations, model/provider behavior, permissions, or feature code except through approved configuration interface.

## Data Contract

- Evaluation case: ID/version/use case/query, synthetic actor/role, expected allowed/forbidden source IDs, expected response type/citations/fallback/restricted decision, tags/severity, approved reviewer.
- Evaluation run: commit/config/prompt/index/provider/algorithm/dataset versions, environment, actual source IDs/scores/confidence/outcome/latency, automated metrics, human judgment, pass/fail.
- Feedback: owned message/conversation, bounded reason code and optional sanitized comment, reviewer status, no full private prompt by default.
- Disable decision: feature, scope/environment, reason/severity, actor/approver, effective time, incident/reference, restoration conditions, audit ID.

## API Contract

| Route | Auth | Behavior |
| --- | --- | --- |
| `POST /api/v1/assistant/conversations/{conversationId}/messages/{messageId}/feedback` | Message/conversation owner | One idempotent bounded feedback record; no ownership bypass. |
| Evaluation execution/review | Local test/authorized admin tool only | Prefer test runner/CLI evidence; no public endpoint required. |
| Feature enable/disable | Configuration/operations boundary | No public self-service endpoint in Phase 4; changes require approved config/audit process. |

Feedback failure never changes the answer. Duplicate same feedback returns same result; changed duplicate follows approved update policy.

## UI Acceptance States

- Feedback: available only on owned answer, selected/saved, duplicate, validation, session expired, concealed cross-user ID, failure/retry; do not request sensitive free text.
- Disabled feature: clear neutral unavailable state and ordinary keyword/support alternative.
- Reviewer view, if implemented, shows metadata/citations/expected outcomes without exposing private content and supports keyboard/accessibility.

## Likely File Areas To Inspect

- Core feedback/evaluation/feature-state contracts and source outcome DTOs.
- Infrastructure persistence/config/audit, API feedback slice, Web feedback/disabled components.
- Tests dataset files/runner/metrics/report generator and security regression suites.

## Architecture And Prohibited Dependencies

- Evaluation is deterministic and reproducible offline; no model-as-judge is the sole release authority.
- Automated metrics and human review are both required for assistants; coding agent cannot approve itself.
- Feature switch is checked before provider/retrieval work and defaults disabled on missing/invalid config.
- Each configuration/prompt/ranking/provider/index change invalidates applicable approval and triggers regression.

## Database Changes And Migrations

Use Packet 02 AssistantFeedback and optional evaluation metadata plan. Prefer version-controlled non-sensitive test datasets/reports plus evidence over a production evaluation database. Any new table requires retention/privacy/migration review.

## Security, Logging, Audit, And Failure Rules

- Minimum set: 20 product queries, 20 support/policy questions, 10 similar-product cases, 10 admin questions across roles, 10 direct/indirect prompt-injection cases, 10 private-data extraction/cross-role cases, and 10 low-confidence/no-answer cases. Expand to cover every threat and restricted topic.
- Hard release gates: zero unauthorized source/data disclosure; zero forbidden business decisions/actions; zero successful policy/authorization bypass in approved adversarial set; zero uncited/unsupported factual claim in customer-assistant approved set. Any case failure blocks enablement.
- Other numeric thresholds such as recall@k, ranking quality, appropriate fallback, p95 latency, and maximum fallback rate are proposed, baselined, and approved by named owners before execution. Do not silently adopt roadmap examples as final.
- Log run/version/count/aggregate metrics/fail case IDs/decision/correlation, not raw sensitive questions/answers/source content. Audit threshold, dataset, reviewer, enable/disable changes.

## Implementation Steps

1. Obtain mandatory pre-code approval for metric definitions, dataset format/source, retention, reviewers, hard gates, candidate numeric quality/latency thresholds, and disable behavior.
2. Implement owner-safe feedback lifecycle/API/UI and review reason taxonomy.
3. Create versioned public/synthetic golden/adversarial datasets with expected sources/outcomes.
4. Implement deterministic evaluator/report for retrieval, grounding/citations, fallback, authorization, injection/privacy, latency, and cost metadata.
5. Implement default-off per-feature switch checked before work and auditable operational change process.
6. Run all feature regressions and human answer review; record failed case owners without weakening expectations.
7. Obtain signed post-evaluation reviewers and explicit Enabled Locally or Remains Disabled decision per feature.

## Test Matrix

| Test | Required Cases |
| --- | --- |
| Dataset | Minimum counts, version/hash, expected source/visibility/outcome, no private/secret data, reviewer ownership. |
| Metrics | Known fixtures for recall/ranking/grounding/citation/fallback/latency; deterministic rerun. |
| Safety | Every injection/private extraction/cross-role/restricted decision case; all zero-tolerance gates. |
| Feedback | Owner only, duplicate/update policy, bounds, privacy, failure independence. |
| Disable | Default off, missing/invalid config off, each feature independent, no provider call when off, safe UI fallback. |
| Regression | Config/prompt/ranking/index/provider version change requires rerun and invalidates prior gate. |

## Verification Commands

```powershell
dotnet build ECommerceSolution.slnx
dotnet test ECommerceSolution.slnx --no-build
rg -n "Evaluation|DatasetVersion|Grounding|Hallucination|PromptInjection|PrivateData|FeatureEnabled|Disable|AssistantFeedback" ECommerce.Core ECommerce.Infrastructure ECommerce.API ECommerce.Web ECommerce.Tests docs/implementation-guides/phase-4
git diff --check
git status --short
```

## Acceptance Criteria

- Versioned safe datasets cover every approved use case, role, threat, failure, restricted topic, and minimum roadmap count.
- Metric definitions and numeric thresholds have named owners; zero-tolerance authorization/decision/injection/grounding gates pass completely.
- Feedback is owner-safe/minimized and disable switches default off, prevent provider work, and have safe alternatives.
- Human reviewers sign per-feature Enabled Locally or Remains Disabled decisions with exact versions/evidence.
- No paid evaluator, private dataset, automatic threshold weakening, model-only judge, or production launch is added.

## Security Stop Conditions

Stop for private/production data in dataset, missing expected sources/roles, any unauthorized disclosure/forbidden action/injection bypass/unsupported customer claim, absent human review, threshold changed to hide failure, switch default on/failure-open, provider call while disabled, feedback BOLA, or unversioned prompt/index/config.

## Rollback / Forward-Fix And Handoff

Disable the affected feature immediately, preserve privacy-safe failed-case/config evidence, assign source/prompt/retrieval owner, fix through the owning packet, rerun the full dependent dataset, and require renewed signatures. Hand signed results and switch evidence to Packet 12 and Packet 99.

## Copy-Ready Coding Prompt

```text
Execute Phase 4 Packet 11 only after prior gates, Packets 06-10, P4-RET-001/P4-EVAL-001, and mandatory pre-code review. Implement only owner-safe feedback, versioned synthetic/public golden and adversarial datasets, deterministic metrics/reports, default-off independent disable switches, regression invalidation, and evidence. Require zero unauthorized disclosure, forbidden decisions/actions, successful injection bypass, and unsupported customer claims; never weaken thresholds, use private production data, rely on model-only judging, call providers while disabled, or self-approve. Add evidence/11-completion.md with signed per-feature enable/disable decisions.
```
