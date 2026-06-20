# Packet 10: Admin Knowledge Assistant And Role Isolation

## Packet Metadata

| Field | Value |
| --- | --- |
| Status | Blocked - PHASE0/PHASE1/PHASE2/PHASE3-GATE |
| Depends On | Packets 04-06 Approved; Phase 1/3 staff permissions Approved; `P4-ADMIN-001` |
| Layer Ownership | Assistant owns answer metadata; Knowledge owns documents; Identity owns access |
| Manual Review | Mandatory pre-code and post-test AI security, IAM/RBAC, privacy, operations, audit, policy-owner, and UX review |
| Produces | Role-isolated internal knowledge assistant and `evidence/10-completion.md` |

## Objective

Provide an audited internal assistant over approved operational documents with explicit permission and visibility filters, zero customer/payment-data indexing, and adversarial proof that lower roles cannot retrieve higher-role sources.

## Read First And Prerequisites

- Packets 01, 04-06 classification/governance/retrieval/citation contracts.
- Approved Phase 1 effective-permission/role-change/session/audit rules and Phase 3 SupportAgent/Admin/SuperAdmin contracts.
- Approved internal document owners, visibility mapping, retrieval-log reviewer permission, retention and incident owners.

## In Scope

- Support/Admin/SuperAdmin assistant query and owned conversation UI using server-derived role plus explicit permission.
- Public, SupportOnly, AdminOnly, and SuperAdminOnly sources exactly as approved; role snapshot and fresh effective permission at each request.
- Grounded/cited answers, restricted decision boundary, low-confidence/no-source fallback, privileged mandatory audit, retrieval-log linkage, rate/turn limits, disable switch.
- Cross-role/source/conversation leakage and prompt-injection adversarial tests.

## Out Of Scope

- CustomerPrivate/order/payment/address/ticket-message indexing, database/table query, live operational records, customer impersonation, action/tool execution, stock/order/refund change, unrestricted SuperAdmin bypass, confidential prompt storage, provider/cloud, or customer-facing use.

## Ownership And Read-Only Contracts

- **Owns:** admin assistant conversation/message metadata, result/citations/fallback, role snapshot, retrieval/audit linkage.
- **Reads:** current server Identity permissions and current published authorized Knowledge sources.
- **Must not own/mutate:** roles/permissions, AuditLog, source documents, customer/commerce/support data, or operational actions.

## Data Contract

- `AssistantType=AdminKnowledge` plus actor ID, role/permission snapshot, retrieval scope/config/source IDs, confidence/fallback and audit ID.
- Visibility is explicit set intersection from effective permissions, not client `requestedVisibility`; remove that roadmap request field unless retained solely as a narrower allowlist that can never broaden server scope.
- Conversation is owner-scoped and reauthorized every turn. Role/permission changes invalidate or narrow future turns; prior response metadata remains protected under original classification.

## API Contract

| Route | Permission | Behavior |
| --- | --- | --- |
| `POST /api/v1/admin/assistant/questions` | `ai.admin.ask` plus visibility-specific access | Query authorized current internal sources; mandatory audit. |
| `GET /api/v1/admin/assistant/conversations/{conversationId}` | Owner or explicit `ai.admin.conversations.read` | Reauthorizes every source/message projection. |
| `GET /api/v1/admin/ai/retrieval-logs` | `ai.retrieval-logs.read` | Bounded metadata only; filters allowlisted; access itself audited. |

Client role/visibility/source/provider/prompt is ignored/rejected. Use Problem Details for auth/authorization; grounded/fallback semantics for content uncertainty; mandatory audit failure follows Phase 1 fail-safe rule.

## UI Acceptance States

- Authorized loading/grounded citations/fallback/restricted decision/source stale/provider failure/rate/disabled; forbidden role; role changed during conversation; concealed conversation; mandatory audit unavailable.
- Clearly label internal visibility and never render a higher-classification title/snippet/citation to a lower role, including page source, logs, autocomplete, error, count, or timing-sensitive list.

## Likely File Areas To Inspect

- Core admin assistant/scope/conversation policies and existing permission/audit contracts.
- Infrastructure retrieval/log persistence, API admin endpoints/OpenAPI, Web staff UI/navigation.
- Tests cross-role matrix, BOLA, audit atomicity, prompt injection, private-data absence, UI/accessibility.

## Architecture And Prohibited Dependencies

- Fresh effective permissions and endpoint purpose determine scope before every retrieval/turn.
- Admin assistant reads only Knowledge index; no repository for Order/Payment/Customer/SupportTicket or generic SQL/data tool.
- Audit and retrieval metadata are separate; privileged request requires mandatory AuditLog without storing full prompt.
- SuperAdmin is still constrained to approved published SuperAdminOnly sources and no business actions.

## Database Changes And Migrations

Use Packet 02 schema. No operational-data index, generic connector, or role-specific duplicate database. Any retrieval-log metadata change requires privacy/database review.

## Security, Logging, Audit, And Failure Rules

- Adversarial matrix attempts role claim, `requestedVisibility` escalation, source ID probing, cross-user conversation, indirect injection from internal content, prompt/config extraction, timing/count leakage, and requests for customer/order/payment/ticket data/actions.
- Audit actor, effective roles/permission version, operation, authorized scope, source IDs, confidence/fallback/result/correlation; never full prompt/answer/source/private operational data.
- Role changed/audit unavailable/source stale/provider failure returns deny/fallback and no higher-scope cache/history.

## Implementation Steps

1. Obtain mandatory pre-code approval for permission/visibility matrix, source owners, conversation/log access, role-change behavior, rates/retention, mandatory audit, routes/UI.
2. Implement fresh server scope resolution and owner-scoped conversation reauthorization.
3. Compose Packet 06 retrieval/grounding with internal citation redaction and restricted decision rules.
4. Implement mandatory audit/retrieval linkage and bounded retrieval-log review.
5. Implement exact APIs and accessible role-aware UI without client visibility escalation.
6. Add exhaustive role/source/conversation/adversarial/private-data/action/audit/failure/UI tests.
7. Obtain post-test security/operations/audit approval; keep production/provider use disabled.

## Test Matrix

| Test | Required Cases |
| --- | --- |
| Role matrix | Anonymous/customer denied; Support/Public+Support; Admin approved set; SuperAdmin approved all; unknown denied. |
| Reauthorization | Role removed/permission version changed mid-conversation; old source citation/history protected. |
| Grounding | Current published authorized sources only; claim citation; no/weak/conflict -> fallback. |
| Adversarial | Client visibility/role/source ID, cross-conversation, indirect injection, prompt extraction, timing/count/source-title leak. |
| Prohibited data/actions | Customer/order/payment/address/ticket content and every mutation/tool request refused/not indexed. |
| Audit/failure/UI | Mandatory audit rollback/deny, retrieval log access audit, provider/stale/rate/disabled, accessible states. |

## Verification Commands

```powershell
dotnet build ECommerceSolution.slnx
dotnet test ECommerceSolution.slnx --no-build
rg -n "admin/assistant|ai.admin|RetrievalLog|requestedVisibility|Order|Payment|CustomerProfile|TicketMessage|Audit" ECommerce.Core ECommerce.Infrastructure ECommerce.API ECommerce.Web ECommerce.Tests
git diff --check
git status --short
```

## Acceptance Criteria

- Every request/turn resolves fresh effective permission and prefilters exactly the approved visibility set.
- Lower roles cannot infer higher sources through results, citations, histories, logs, errors, counts, or timing fixtures.
- No customer/order/payment/address/ticket content is indexed/retrieved and no action/tool exists.
- Grounding/citation, mandatory audit, conversation ownership, rates/failures, exhaustive adversarial role matrix and UI/accessibility tests pass.
- Mandatory reviews/evidence exist; no provider/cloud/production enablement is added.

## Security Stop Conditions

Stop for client visibility/role/source control, numeric role hierarchy, stale permission cache broadening scope, cross-user history, higher-source metadata leak, generic database/operational connector, private/payment/support index, business action/tool, audit bypass/failure-open, full prompt logging, missing adversarial matrix, or unresolved high/critical issue.

## Rollback / Forward-Fix And Handoff

Disable admin assistant and retrieval-log UI while preserving protected audit metadata. Revoke affected source/permission, investigate exposure, version/fix scope/prompt, rerun full role matrix and Packet 11 evaluation before re-enable. Hand results to Packets 11-12.

## Copy-Ready Coding Prompt

```text
Execute Phase 4 Packet 10 only after all prior gates, Packets 04-06, approved staff permissions, P4-ADMIN-001, and mandatory pre-code review. Implement only fresh permission-derived role-scoped retrieval, owner-safe conversations, grounded internal citations, mandatory audit/retrieval-log review, bounded API/UI, disable/fallback, and exhaustive cross-role/adversarial tests. Never trust requestedVisibility/role/source, index customer/order/payment/ticket data, add generic data tools/actions, leak higher-source metadata, log prompts, or add providers/cloud. Add evidence/10-completion.md and obtain post-test approval.
```
