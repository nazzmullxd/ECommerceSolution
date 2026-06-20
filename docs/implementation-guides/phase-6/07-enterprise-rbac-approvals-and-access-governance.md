# Packet 07: Enterprise RBAC, Approvals, And Access Governance

## Packet Metadata

| Field | Value |
| --- | --- |
| Status | Blocked - PHASE0/PHASE1/PHASE2/PHASE3/PHASE4/PHASE5-GATE |
| Depends On | Packet 01 Approved; stable Phase 1/3/5 Identity/access/audit; `P6-RBAC-001` |
| Decision Type | Planning first; extend central Identity/Access module in monolith |
| Manual Review | Security/IAM, privacy, audit, HR/business role owners, module/data owners, operations/emergency access and compliance review |
| Produces | Enterprise permission/approval/access-review plan and `evidence/07-completion.md` |

## Objective

Design scalable least-privilege roles/permissions using the least privilege principle, separation of duties, sensitive-action approvals, access review, emergency access and audit controls without creating parallel identities or permission sprawl.

## Read First And Prerequisites

- Approved ApplicationUser/Role/Permission/UserRole/RolePermission/session/admin/audit/production access contracts.
- Packet 01 owner/RACI and Packets 03-06 sensitive actions/data/export scopes.
- HR/joiner-mover-leaver, MFA/JIT/break-glass, access review and compliance owner decisions.

## In Scope

- Role bundles: SuperAdmin, Admin, SupportAgent, InventoryManager, CatalogManager, MarketingManager, ReportingViewer, Auditor and approved future roles.
- Atomic permission catalog with code/category/description/owner/risk/data scope, assignment eligibility, expiry/review and incompatibilities.
- Separation of duties/two-person approval for roles/permissions, high-value campaigns, loyalty adjustments, inventory transfers/adjustments, sensitive exports/settings/emergency operations.
- Access request/approve/provision/review/revoke, joiner/mover/leaver, permission-version/session invalidation, emergency/break-glass with MFA/time/reason/monitoring/post-review.
- Audit/protected review, permission-sprawl metrics, orphan/unused/duplicate cleanup and UI/API governance.

## Out Of Scope

- Separate staff identity, external enterprise IdP/IAM integration, AWS IAM policy, ABAC engine, customer impersonation, permanent break-glass, role hierarchy granting implicit broad access, service extraction or compliance certification.

## Ownership And Boundaries

- **Owns:** central application roles/permissions/assignments/approvals/reviews/emergency access and policy decision.
- **Reads:** module action/data risk metadata and organization identity status through approved source.
- **Must not own/mutate:** business data/action outcome, AWS IAM, audit records, HR source or module approval semantics.
- Modules enforce policies server-side and mandatory audit; UI hiding is not authorization.

## Evidence Required

| Evidence | Required Proof |
| --- | --- |
| Entry Evidence | Stable central Identity/session/audit/admin access and named role/data owners. |
| Measured Need Evidence | New job functions/actions, access incidents, review burden/sprawl metrics. |
| Design Evidence | Permission/role/SoD/approval/review/emergency/session/audit/API/test/migration/rollout. |
| Monolith-First Evidence | Future negative policy tests, review/revoke/emergency drills and production access metrics. |
| Extraction/Cloud Evidence | Not applicable; application access remains central module; AWS IAM separately mapped. |

## Data And Contract Design

Permission codes are stable, action-specific and deny unknown. Role is a reviewed bundle, not an authorization shortcut. Assignment records include scope if approved, requester/approver/reason/start/expiry/review/status/version. Approval binds exact command/fingerprint/value/risk/window and cannot be reused for changed action.

Define incompatible duties and self-approval/self-escalation prohibitions. Emergency access is separate time-limited assignment with stronger authentication, alert/audit and mandatory post-use review.

## API And UI Contract

- Explicit request/approve/reject/assign/revoke/review/emergency commands with permission, expected version, reason and mandatory audit; no generic role patch.
- Effective permissions endpoint returns current actor-safe codes/version, not sensitive policy graph.
- Admin UI shows owner/risk/expiry/incompatibility/pending approval and confirmation; inaccessible data/actions never load server-side.

## Likely File Areas To Inspect

- Core Identity/Access/Audit/session/current-user/policy contracts, Infrastructure mappings/cache/invalidation, API/Web admin/approval/review and Tests security.
- Planning only now.

## Architecture And Prohibited Dependencies

- One canonical ApplicationUser and permission engine; no per-module staff tables.
- Role hierarchy cannot override explicit deny/incompatibility/data scope.
- Authorization decisions are fresh enough for sensitive actions; cache has version/invalidation and fail-closed behavior.
- AWS IAM/service identities are separate from application RBAC and require Packet 13.

## Database Changes And Migrations

No migration now. Future approval/scope/expiry/review/emergency schema requires unique/concurrency/index/retention, bootstrap/seed change, session invalidation and Phase 5 migration/security review.

## Security, Logging, Audit, And Failure Rules

- Prevent self-escalation, confused deputy, role/permission injection, stale permission cache, orphan assignment, approval replay, collusion gaps and emergency abuse.
- Log actor/target/permission/role/scope/action/result/reason code/correlation, not full PII or secrets.
- Mandatory audit failure blocks assignment/approval/emergency/sensitive action.
- Review/revoke/session invalidation failure fails closed for sensitive access and alerts owner.

## Planning Or Implementation Steps

1. Inventory enterprise actions/data/risk/owners and map atomic permissions.
2. Define role bundles without implied hierarchy; identify SoD/incompatible/self-approval rules.
3. Define request/approval/assignment/expiry/review/revoke/session invalidation lifecycle.
4. Define sensitive-action approval fingerprint/threshold/two-person and emergency access.
5. Define API/UI/audit/alert/access-review/sprawl metrics and retention.
6. Define seed/migration/negative tests/phased rollout/parallel policy comparison/rollback.
7. Obtain planning approval before bounded central access implementation.

## Test And Review Matrix

| Test | Required Cases |
| --- | --- |
| Role matrix | Every role/permission/action/data scope positive and negative; unknown deny. |
| Escalation | Self/peer escalation, assignment without authority, role hierarchy/confused deputy. |
| Approval | Same/different command fingerprint, threshold, expiry, duplicate/replay, self/SoD. |
| Lifecycle | Join/move/leave, expiry/review/revoke, session/cache invalidation and owner unavailable. |
| Emergency | MFA/reason/time/alert/audit/revoke/post-review, no permanent assignment. |
| Failure/rollout | Audit/DB/cache/notification failure, shadow policy comparison, flag/rollback no open access. |

## Verification Commands

```powershell
rg -n "ApplicationUser|Role|Permission|UserRole|RolePermission|Audit|MFA|Session|SuperAdmin|SupportAgent" docs/roadmap docs/implementation-guides/phase-1 docs/implementation-guides/phase-3 docs/implementation-guides/phase-6
git diff --check
git status --short
```

## Acceptance Criteria

- Atomic permissions/role bundles/data scopes/owners and deny-unknown behavior are explicit.
- SoD, sensitive approval, lifecycle/review/revoke/session invalidation and emergency access are fail-closed/audited/testable.
- One canonical Identity remains; application RBAC and AWS IAM are not conflated.
- Planning adds no role seed/code/migration/external IdP/cloud now.

## Manual Approval Gates

Manual approval is mandatory for permission catalog/role bundles/data scopes, SoD/approval thresholds, SuperAdmin/emergency/MFA, access review/retention, seed/migration/session invalidation, rollout and future IAM mapping.

## Stop Conditions

Stop for parallel staff identity, implicit broad hierarchy, self-escalation/approval, permanent emergency access, stale cache failure-open, missing audit, unknown owner/reviewer, unrestricted exports/admin data, AWS IAM conflation, service/cloud or implementation without approval.

## Rollback / Forward-Fix And Handoff

Planning-only. Future policy rollout runs shadow/negative tests and can revert bundle mappings without removing deny policies; revoke risky assignments/sessions first. Never rollback by granting broad access. Hand action approvals to Packets 03-06, events to 08, AI scope to 09 and retention to 12.

## Copy-Ready Constrained Prompt

```text
Execute Phase 6 Packet 07 as planning only after prior gates, Packet 01 and P6-RBAC-001. Update only enterprise permission/role/SoD/approval/access-review/revoke/session/emergency/audit/API/data/test/migration/rollout design and evidence. Do not add users/roles/seeds/code/migrations, create parallel identities, allow hierarchy/self-escalation/permanent break-glass, conflate AWS IAM, or provision services. Keep central deny-by-default application RBAC in the monolith. Add evidence/07-completion.md with security/privacy/module-owner review.
```
