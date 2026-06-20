# Packet 09: Customer Support Assistant And Escalation

## Packet Metadata

| Field | Value |
| --- | --- |
| Status | Blocked - PHASE0/PHASE1/PHASE2/PHASE3-GATE |
| Depends On | Packets 04-07 Approved; Phase 3 Support contract Approved; `P4-ASSIST-001`, `P4-CUSTOMER-ENABLEMENT-GATE` |
| Layer Ownership | Assistant owns conversation/answer metadata; public Knowledge owns policy facts; Support owns tickets |
| Manual Review | Mandatory pre-code and post-test AI security, policy/legal, privacy, Support, abuse, accessibility, and product review; manual security approval before enablement |
| Produces | Grounded public support assistant and `evidence/09-completion.md` |

## Objective

Provide a local customer assistant that answers only from approved current public FAQ/help/shipping/return/refund-summary/product sources, cites support, refuses unsupported or decision-sensitive requests, and offers an explicit privacy-safe support-ticket path.

## Read First And Prerequisites

- Packets 04-07 publication/retrieval/confidence/citation/provider/search contracts.
- Approved Phase 3 customer SupportTicket ownership/category/UI and notification contracts.
- Approved public policy owners, restricted-topic taxonomy, anonymous policy, rate limits, retention, and enablement reviewers.

## In Scope

- Customer question endpoint and accessible conversation UI using public scope only for anonymous and authenticated users in Phase 4.
- Approved sources: public FAQ, shipping policy, return policy, approved refund-policy summary, public help, and public product information.
- Grounded answer with caller-safe citations; clarification only when safe; central low-confidence/no-source/stale/provider/disabled fallback.
- Restricted-topic behavior: cite-and-summarize approved policy without deciding refund/payment/legal/account-security eligibility/outcome; otherwise refuse/fallback.
- Explicit support escalation: anonymous sign-in/help path; authenticated user chooses to open Phase 3 ticket form with safe category and optional user-reviewed summary, not automatic ticket creation or raw transcript.
- Conversation ownership, rate/size/turn limits, retention/minimization, feedback hook, and adversarial tests.

## Out Of Scope

- Private order/account/payment/address/support context, authentication/account recovery, final policy/refund/payment/legal/fraud/safety decision, ticket/refund/order action, autonomous agent, transcript attachment, live chat, provider/cloud, or production enablement.

## Ownership And Read-Only Contracts

- **Owns:** customer assistant conversation/message metadata, grounded/fallback result, citations, safe escalation suggestion, evaluation linkage.
- **Reads:** current approved public sources and current authenticated actor only for conversation ownership/rate/escalation availability.
- **Must not own/mutate:** SupportTicket, Order/Payment/Account, policy source, CustomerProfile, authorization, refund, or any commerce state.

## Data Contract

- Anonymous conversations are request-scoped or use a reviewed high-entropy short-lived server identifier; never merge across users/devices or persist a broad profile.
- Authenticated conversations belong to the current user; role change does not broaden public customer scope.
- Store hashes/source IDs/confidence/fallback and only an approved bounded sanitized preview; no full sensitive transcript by default.
- Every factual answer sentence/claim must map to one or more returned public citations. Restricted decision request carries explicit refusal/next action.

## API Contract

| Route | Auth | Behavior |
| --- | --- | --- |
| `POST /api/v1/assistant/customer/questions` | Anonymous or Customer | Bounded question/optional owned conversation ID; public scope only. |
| `GET /api/v1/assistant/customer/conversations/{conversationId}` | Owner if persistence approved | Bounded safe history; anonymous history policy must be explicit. |
| `POST /api/v1/assistant/customer/conversations/{conversationId}/escalations` | Authenticated owner | Returns/prepares approved Support ticket navigation; does not auto-submit. |

Success includes answer, safe citations, confidence band/fallback, and next action. Restricted/unsupported/low confidence is a safe fallback response. Ownership violation is concealed. Rate limit returns `429`; disabled/provider unavailable follows approved safe response.

## UI Acceptance States

- Consent/expectation notice without marketing claims, question validation, loading/cancel, grounded answer/citations, clarification, low confidence/no source, restricted-topic refusal, provider unavailable, rate limited, session expired, conversation concealed, feature disabled.
- Escalation requires explicit user action, sign-in when needed, editable safe summary, warning not to include password/card data, and no automatic transcript transfer.
- Keyboard/focus/live-region/error/citation navigation/reflow/long text and screen-reader behavior must pass.

## Likely File Areas To Inspect

- Core assistant/conversation/escalation/restricted-topic services and Support query/command boundary.
- Infrastructure retrieval/provider/persistence, API endpoint/rate/OpenAPI, Web assistant/support handoff UI.
- Tests grounding/citations/ownership/privacy/adversarial/rate/failure/UI/accessibility/evaluation.

## Architecture And Prohibited Dependencies

- Endpoint purpose always resolves `Public` sources, even for staff/admin using this endpoint.
- Assistant cannot call Support/Order/Payment/Auth commands. Escalation is a user-confirmed handoff to existing Support UI/API.
- Conversation history is untrusted input and cannot alter system policy or authorize sources.
- No answer from model memory. Mock provider output without exact source support is rejected.

## Database Changes And Migrations

Use Packet 02 conversation/message schema only. No ticket transcript/link table or private-context index. Retention must be approved before persistent anonymous history.

## Security, Logging, Audit, And Failure Rules

- Detect/classify prompt injection, system-prompt extraction, source fabrication, private-data/order lookup, payment/refund/legal/account-security decision, credential/card submission, abusive/resource-heavy input, and cross-conversation IDs.
- Policy-sensitive question: retrieve exact approved current public source, provide citation-bound summary plus "cannot decide" boundary; absent/weak source always fallback/escalate.
- Log operation, pseudonymous actor/conversation ID, question hash/category, source IDs, confidence/fallback/rate/duration/correlation; never full question/answer/transcript, credentials, contact/order/payment/support data.
- Provider/retrieval/log persistence/escalation dependency failure does not fabricate answer or auto-create ticket.

## Implementation Steps

1. Obtain mandatory pre-code approval for anonymous behavior, sources/categories, restricted-topic matrix, citation wording, turn/size/rate/retention, escalation and exact API/UI.
2. Implement owner-safe conversation service and public-purpose retrieval orchestration.
3. Implement claim/citation validation, restricted-topic refusal/summary, central fallback, and disable check.
4. Implement exact API plus accessible UI and explicit Support handoff without automatic transcript.
5. Add golden public policy set and adversarial injection/private-data/cross-user/forbidden-decision/source-fabrication tests.
6. Run evaluation and failure/rate/privacy/log/accessibility tests; obtain post-test reviewers.
7. Keep disabled until Packet 11 thresholds and named manual security/product approval pass.

## Test Matrix

| Test | Required Cases |
| --- | --- |
| Grounding | Every answer claim cited; no source/weak/conflict/stale -> fallback; fake citation rejected. |
| Restricted | Refund/payment/legal/policy exception/account-security asks: cited summary only, no decision; no evidence -> refusal. |
| Authorization | Public scope for all; cross-user conversation concealed; anonymous isolation/expiry; no private source. |
| Adversarial | Direct/indirect/encoded injection, prompt/source extraction, role claim, order/payment lookup, credential/card text, data exfiltration. |
| Escalation | Anonymous sign-in, explicit confirmation, editable safe summary, no raw transcript/auto-ticket, Support ownership preserved. |
| Failure/UI | Provider/retrieval/log/support/rate/disabled; all accessible states and privacy-safe logs. |

## Verification Commands

```powershell
dotnet build ECommerceSolution.slnx
dotnet test ECommerceSolution.slnx --no-build
rg -n "assistant/customer|RestrictedTopic|Fallback|Citation|Conversation|SupportTicket|Order|Payment|Transcript" ECommerce.Core ECommerce.Infrastructure ECommerce.API ECommerce.Web ECommerce.Tests
git diff --check
git status --short
```

## Acceptance Criteria

- Assistant uses only current approved public sources and every factual claim has caller-safe citation support.
- Sensitive policy/refund/payment/legal/account-security requests never receive a model-memory or eligibility/outcome decision.
- Conversation ownership/anonymous isolation, limits/rates, privacy logs, adversarial tests, explicit no-transcript escalation, and accessible API/UI states pass.
- No private context/business action/provider/cloud is added.
- Feature remains disabled until Packet 11 and signed customer-enablement security/product approval pass.

## Security Stop Conditions

Stop for missing review, private/order/payment/support retrieval, model-memory answer, uncited claim, sensitive decision, automatic ticket/action/transcript transfer, cross-user conversation, prompt/source leak, full transcript logging, missing adversarial evaluation, or enablement without signed gate.

## Rollback / Forward-Fix And Handoff

Activate the central disable switch and route users to ordinary FAQ/Support flows. Preserve privacy-safe metadata for incident review, archive/disable affected source/prompt version, correct through governance/reindex/evaluation, and require renewed approval. Hand outcomes to Packet 11-12.

## Copy-Ready Coding Prompt

```text
Execute Phase 4 Packet 09 only after all prior gates, Packets 04-07, approved Phase 3 Support contract, P4-ASSIST-001, and mandatory pre-code review. Implement only a public-source grounded customer assistant, claim citations, restricted-topic refusal/cited summary boundary, owner-safe conversation, explicit no-transcript Support handoff, bounded API/UI, disable/fallback, and adversarial/accessibility tests. Never retrieve private/order/payment/support data, answer sensitive decisions from memory, auto-create actions, log transcripts, add provider/cloud, or enable without the signed gate. Add evidence/09-completion.md and obtain reviews.
```
