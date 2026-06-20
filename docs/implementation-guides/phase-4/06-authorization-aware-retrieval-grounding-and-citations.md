# Packet 06: Authorization-Aware Retrieval, Grounding, And Citations

## Packet Metadata

| Field | Value |
| --- | --- |
| Status | Blocked - PHASE0/PHASE1/PHASE2/PHASE3-GATE |
| Depends On | Packets 01-05 Approved; `P4-CONFIDENCE-001`, `P4-CITATION-001` |
| Layer Ownership | Retrieval owns scope/ranking/grounding decisions; providers only return candidates/output |
| Manual Review | Mandatory pre-code and post-test AI security, authorization, privacy, source-owner, and UX review |
| Produces | Safe retrieval/prompt/citation/fallback foundation and `evidence/06-completion.md` |

## Objective

Implement the single retrieval foundation used by search and assistants: derive caller scope, pre-filter current approved sources, rank deterministically, calculate explainable confidence, build a data-delimited prompt, validate grounding/citations, log safe metadata, and fall back on uncertainty.

## Read First And Prerequisites

- Packets 01-05 contracts, threat controls, active-generation and provider behavior.
- Approved Phase 1 current-user/permission/audit/rate conventions and source visibility mappings.
- Evaluation owners and proposed confidence/citation decisions; customer use remains disabled.

## In Scope

- Endpoint-purpose-specific `RetrievalScope` derived server-side from identity/permissions.
- Query validation/normalization/category/restricted-topic detection, keyword/vector candidate retrieval, pre-retrieval filters, deterministic merge/rerank, source diversity/limits, confidence components, fallback policy, and citation construction.
- Fixed/versioned prompt builder that separates trusted system policy, untrusted user text, and untrusted retrieved data; no tools/actions.
- Post-generation support check mapping answer claims to retrieved chunks or rejecting/falling back.
- RetrievalLog metadata and correlation/causation without full sensitive prompts.

## Out Of Scope

- Feature endpoint/UI, provider-specific prompt syntax, model-memory policy answer, chain-of-thought storage/display, private source retrieval, automatic threshold tuning, business command/tool execution, or customer enablement.

## Ownership And Read-Only Contracts

- **Owns:** scope selection, candidate filters, score normalization/merge, confidence/fallback, prompt version, citations, grounding validation, retrieval outcome metadata.
- **Reads:** server user context, complete active SearchDocument generations, deterministic providers, approved source metadata.
- **Must not own/mutate:** identity/permissions, source content/lifecycle, provider configuration, Product/Support/Order/Payment, AuditLog, or evaluation approval.

## Data Contract

- Scope includes endpoint purpose, authenticated actor category, explicit allowed visibility/source types, publication/active/expiry rules, maximum candidates/chunks/tokens-or-characters, and prohibited categories.
- Confidence is an application score, not provider certainty. Record versioned formula components such as top relevance, margin/diversity, exact source support, freshness/completeness, and penalties; clamp/validate all provider scores.
- Citation contains source ID/version/chunk plus caller-safe title/section/route. Internal source IDs/titles never cross to lower visibility.
- Prompt builder marks retrieved text as quoted data and instructs no rule following from it; source ordering is deterministic.

## API Contract

Internal retrieval contract accepts normalized query plus server-derived purpose/context, not role/visibility/source IDs from the client. It returns authorized candidates, citations, confidence components, fallback reason, config/dataset/prompt versions, and safe diagnostics.

Common feature responses use grounded success or fallback. `NO_APPROVED_SOURCE`, `LOW_CONFIDENCE`, `RESTRICTED_TOPIC`, `STALE_SOURCE`, `PROVIDER_UNAVAILABLE`, and `FEATURE_DISABLED` are normal typed outcomes where appropriate, not fabricated answers.

## UI Acceptance States

Define reusable citation and fallback components: grounded answer/results with source labels, source unavailable/stale, low confidence, restricted topic, disabled feature, provider failure, rate limit, and manual/support next action. Never show raw score as a probability of truth without approved UX wording.

## Likely File Areas To Inspect

- Core retrieval/scope/policy/confidence/prompt/citation interfaces and current-user permissions.
- Infrastructure search providers/query filters/log persistence and API/Web shared response/view components.
- Tests golden/adversarial scope, prompt, score, citation, ownership, privacy, and failure areas.

## Architecture And Prohibited Dependencies

- Authorization is calculated before provider search and repeated in backing query/provider filters. In-memory post-filter is defense in depth only, never primary.
- Retrieval content and user query are untrusted data. Prompt text cannot loosen scope or source status.
- Provider answer never determines grounding; application validates citations/support and can only narrow to fallback.
- Do not persist chain-of-thought, hidden prompt, full prompt, or unrestricted source text in logs.

## Database Changes And Migrations

Use Packet 02 RetrievalLog schema. Any score-component/source child mapping change needs database/privacy review. No vector extension or source schema change.

## Security, Logging, Audit, And Failure Rules

- Deny unknown purpose/role/visibility/source/status. Recheck document status/version/current generation at retrieval time.
- Restricted policy/refund/payment/legal/account-security questions can retrieve approved public policy for a cited summary, but output must refuse a decision and escalate. Without exact evidence, fallback.
- Log actor pseudonymous ID/role snapshot, purpose, query hash/category, source IDs/versions/scores, confidence components/version, fallback/safety reason, latency, correlation; not full query/prompt/answer/source/private data.
- Any filter/provider/log persistence uncertainty returns safe fallback/error; mandatory privileged audit failures follow Phase 1 rules.

## Implementation Steps

1. Obtain mandatory pre-code approval for scope matrix, restricted categories, candidate limits, score normalization/merge, confidence formula/bands, citation display, prompt template/version, log fields.
2. Implement server scope resolver and backing pre-filter predicates with deny-unknown tests.
3. Implement normalized hybrid candidate merge/rank/diversity and explainable confidence components.
4. Implement citation builder and fixed data-delimited prompt with injection defenses.
5. Implement output grounding/restricted-data validation and central fallback policy.
6. Persist privacy-safe RetrievalLog metadata and propagate correlation/config versions.
7. Add adversarial direct/indirect injection, role leakage, stale/deleted, score manipulation, fake citation, unsupported claim, provider/log failure tests.
8. Run checks/evaluation fixtures and obtain mandatory post-test review; keep customer features disabled.

## Test Matrix

| Test | Required Cases |
| --- | --- |
| Authorization | Every role/visibility/purpose combination; unknown denied; backing query predicate verified. |
| Source state | Published/current/complete only; stale/expired/deleted/partial/old generation excluded. |
| Confidence | Formula components/version/bounds/ties; missing/weak/conflicting sources; low score fallback. |
| Grounding/citations | Every factual claim supported; fake/missing/wrong-visibility citation rejected; safe labels. |
| Adversarial | Direct/indirect injection, system-prompt request, encoded override, role/source/score injection, data extraction. |
| Privacy/failure | No full prompt/source/answer logs; provider/vector/keyword/log DB timeout/malformed scores fail closed. |

## Verification Commands

```powershell
dotnet build ECommerceSolution.slnx
dotnet test ECommerceSolution.slnx --no-build
rg -n "RetrievalScope|Confidence|Citation|Prompt|Fallback|Visibility|RetrievalLog|SystemPrompt|ChainOfThought" ECommerce.Core ECommerce.Infrastructure ECommerce.API ECommerce.Web ECommerce.Tests
git diff --check
git status --short
```

## Acceptance Criteria

- Scope is server-derived and applied before retrieval in every provider/backing query; unknown values fail closed.
- Only complete current approved authorized chunks reach deterministic prompt construction.
- Confidence/fallback/citations/grounding are centralized, versioned, explainable, and independently tested.
- Adversarial authorization/injection/fake-citation/private-data/provider-failure tests pass with privacy-safe retrieval logs.
- Mandatory reviews/evidence exist and no customer-facing feature is enabled.

## Security Stop Conditions

Stop for retrieve-then-filter primary design, client role/visibility/source/score/prompt control, stale/partial/private source, prompt as authorization, provider self-certifying grounding, unsupported claim without fallback, internal citation leak, chain-of-thought/full prompt logging, missing adversarial tests, or unapproved confidence/citation decision.

## Rollback / Forward-Fix And Handoff

Set the central feature/retrieval switch disabled and use keyword/manual support fallbacks. Version prompt/confidence/ranking changes, rerun all downstream evaluation, and never reinterpret old logs without their config version. Hand approved retrieval contract to Packets 07-12.

## Copy-Ready Coding Prompt

```text
Execute Phase 4 Packet 06 only after all prior gates, Packets 01-05, P4-CONFIDENCE-001/P4-CITATION-001, and mandatory pre-code security review. Implement only server-derived pre-filtered retrieval, deterministic hybrid ranking, versioned confidence/fallback, data-delimited prompt, citation/grounding validation, privacy-safe retrieval logs, shared UI states, and adversarial tests. Never accept client role/source/score/prompt, retrieve broad then filter, use stale/private data, answer unsupported/restricted decisions, log full prompts, or enable customer features. Add evidence/06-completion.md and obtain post-test review.
```
