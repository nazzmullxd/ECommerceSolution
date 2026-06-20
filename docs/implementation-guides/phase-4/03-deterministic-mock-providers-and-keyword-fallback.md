# Packet 03: Deterministic Mock Providers And Keyword Fallback

## Packet Metadata

| Field | Value |
| --- | --- |
| Status | Blocked - PHASE0/PHASE1/PHASE2/PHASE3-GATE |
| Depends On | Packet 01 Approved; Packet 02 contracts available; `P4-MOCK-001` |
| Layer Ownership | Infrastructure owns local provider mechanics; Core owns contracts and safety decisions |
| Manual Review | Pre-code AI architecture/test review; post-test determinism and dependency review |
| Produces | Deterministic local completion/embedding/vector/keyword adapters and `evidence/03-completion.md` |

## Objective

Implement predictable offline providers that exercise the real contracts and failure paths without a model download, external network, secret, provider SDK, probabilistic output, or false claim of AI quality.

## Read First And Prerequisites

- Packet 01 provider/result/scope contracts and threat controls.
- Packet 02 provider-neutral document/embedding persistence decision if persistence is included.
- Approved Phase 3 Catalog keyword-search contract and repository package/dependency rules.

## In Scope

- Deterministic mock completion provider driven by explicit fixtures/retrieved source text.
- Deterministic local embedding function with documented normalization, fixed dimension/version, stable hash, and empty/oversize behavior.
- Local vector-search adapter with deterministic similarity/tie-breaker and mandatory server scope filters.
- Database-backed keyword fallback using typed allowlisted criteria and public/authorized source filters.
- Provider contract tests, repeatability tests, failure injection, cancellation/timeouts, and test-only fixtures.

## Out Of Scope

- Real LLM/local model, model download, internet call, provider SDK, generative free-form answer, Bedrock/OpenSearch/pgvector, GPU, approximate nearest-neighbor engine, quality claim, production benchmark, or customer enablement.

## Ownership And Read-Only Contracts

- **Owns:** local provider implementation/version metadata, deterministic algorithm, test fixtures, similarity output, and simulated failures.
- **Reads:** approved SearchDocument projections supplied with an already-authorized scope.
- **Must not own/mutate:** authorization, publication, source truth, confidence/fallback policy, knowledge lifecycle, conversation, or business modules.

## Data Contract

- Completion input contains fixed system policy ID/version and retrieved source IDs/snippets; mock output must be traceable to fixtures/snippets or return no-answer.
- Embedding metadata includes provider=`DeterministicMock`, algorithm version, dimension, input hash, vector hash, status, and no raw private input.
- Search candidates return document ID/source version/chunk, score components, and deterministic tie-break key.
- Exact algorithm/dimension/normalization and score range are versioned and approved; a change requires reindex and evaluation, not silent replacement.

## API Contract

No public endpoint. Provider interfaces are internal. They honor cancellation, bounded input/result count, typed failure codes, and no exception detail leakage. Keyword fallback uses the same typed retrieval result contract as vector search.

## UI Acceptance States

No feature UI. Test/demo tooling must label provider output `Deterministic Mock` and must not present it as intelligent, verified, or production-ready.

## Likely File Areas To Inspect

- Core provider/search contracts and configuration abstractions.
- Infrastructure AI/Search adapter registration, local query implementation, clocks/failure injector.
- Tests contract fixtures/snapshots/architecture/integration areas.

## Architecture And Prohibited Dependencies

- Implementations live in Infrastructure; Core has no algorithm/provider dependency.
- No random seed based on runtime/process, current time, culture-dependent ordering, network, environment-specific hash, or global mutable fixture.
- Provider receives scope filters and must enforce them in candidate selection; it cannot broaden scope.
- Mock completion may compose/copy bounded retrieved facts deterministically but cannot invent unsupported content.

## Database Changes And Migrations

No new migration beyond Packet 02. Stop if an algorithm requires provider-specific vector schema/extension. In-memory/test representation is acceptable only behind the same contract and with integration coverage for approved persistence.

## Security, Logging, Audit, And Failure Rules

- Treat all source/query text as untrusted and bounded. Deterministic does not mean safe; scope/publication filters still apply.
- Log provider/algorithm version, operation, counts, score range, duration, status, and correlation ID; not raw query/source/vector/full output.
- Simulate no response, malformed metadata, cancellation, timeout, embedding failure, vector failure, keyword failure, and partial candidate data.
- Provider failures return typed results; request orchestration decides fallback and never retries unboundedly.

## Implementation Steps

1. Approve deterministic algorithm/version/dimension/token normalization/similarity/tie-break and fixture format.
2. Implement mock completion with explicit supported fixtures/source extraction and no-answer default.
3. Implement deterministic embedding and vector adapter with bounds/cancellation/filter enforcement.
4. Implement typed database keyword fallback using approved source filters.
5. Register local providers explicitly; fail startup on ambiguous/missing provider configuration.
6. Add shared contract, determinism across repeated runs/cultures, filter, failure, cancellation, and architecture tests.
7. Run offline/network-dependency and package/diff checks; obtain post-test review.

## Test Matrix

| Test | Required Cases |
| --- | --- |
| Determinism | Same version/input produces byte-equivalent metadata/vector/output/order across repeats/cultures. |
| Bounds | Empty/oversize input, max candidates, invalid dimension, cancellation, timeout. |
| Scope | Visibility/publication/source type filters applied before candidate return; unknown denied. |
| Grounding | Mock answer uses supplied source fixture only; absent evidence returns no-answer. |
| Failure | Completion/embedding/vector/keyword unavailable/malformed/partial; typed safe result. |
| Dependencies | No network/provider SDK/model/cloud/secret; Core remains clean. |

## Verification Commands

```powershell
dotnet build ECommerceSolution.slnx
dotnet test ECommerceSolution.slnx --no-build
dotnet list ECommerceSolution.slnx package --vulnerable --include-transitive
rg -n "HttpClient|Bedrock|OpenSearch|OpenAI|Azure|Ollama|Random|DeterministicMock|Embedding|Vector" ECommerce.Core ECommerce.Infrastructure ECommerce.Tests
git diff --check
git status --short
```

## Acceptance Criteria

- All local providers are offline, deterministic, versioned, bounded, cancellation-aware, and contract-compatible.
- Missing evidence returns no-answer; providers cannot broaden authorization scope or invent business decisions.
- Keyword fallback works through the same safe result contract and all failure modes are reproducible.
- No SDK/model/cloud/network/secret/provider-specific schema is introduced; tests/evidence/reviews pass.

## Security Stop Conditions

Stop for network/model/provider dependency, nondeterministic ordering/hash, scope filter after candidate return, free-form invented answer, raw text/vector logging, client algorithm control, unbounded input/result/retry, provider-specific Core type, ambiguous provider selection, or customer-facing enablement.

## Rollback / Forward-Fix And Handoff

Disable provider registration and use explicit feature-disabled/keyword fallback. Algorithm changes require a version bump, reindex, contract/evaluation rerun, and forward-fix; never reinterpret stored vectors under a new version. Hand stable providers to Packets 05-12.

## Copy-Ready Coding Prompt

```text
Execute Phase 4 Packet 03 only after prior gates, Packet 01, P4-MOCK-001, and pre-code review. Implement only offline deterministic mock completion/embedding/vector providers, typed keyword fallback, configuration, failure injection, and contract/security tests in approved layers. Add no model download, network/provider SDK, cloud service, random behavior, provider-specific Core/schema, free-form unsupported answer, or customer UI. Enforce supplied scope filters before candidate return, label mock behavior, add evidence/03-completion.md, and obtain post-test review.
```
