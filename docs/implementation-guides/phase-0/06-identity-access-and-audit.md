# Packet 06: Identity, Access, And Audit

## Packet Metadata

| Field | Value |
| --- | --- |
| Status | Not Started |
| Depends On | Packets 02, 04, And 05 Approved |
| Produces | `artifacts/identity-access-audit.md` |
| Human Reviewers | Security reviewer, solution architect, product owner |

## Objective

Define authentication, session/token, authorization, permission, ownership, admin, audit, and bootstrap contracts so Phase 1 can implement identity without making security architecture decisions.

## Read First

- Approved Packets 01-05 artifacts
- `docs/roadmap/phase-0.md`, section 8
- `docs/roadmap/phase-1-mvp-foundation.md`, auth/access/audit sections
- `docs/architecture/system-design.md`, canonical identity model
- `docs/architecture/security-scalability-review.md`

## In Scope

- Canonical `ApplicationUser` identity model, CustomerProfile separation, ASP.NET Core Identity direction, cookie/bearer boundaries, refresh/session concepts, roles/permissions/policies, ownership, admin bootstrap, lockout/recovery/MFA roadmap, audit requirements, and negative-access behavior.

## Out Of Scope

- Identity/JWT/cookie code, hashing configuration values, secrets, migrations, real email, external identity providers, or production IAM.
- Separate admin-user entity.
- Granting specialized future staff roles before their phase requires them.

## Required Artifact

Create `artifacts/identity-access-audit.md` containing:

1. Identity/account/profile boundaries and canonical terminology.
2. Authentication flows for registration, login, current session, refresh, logout/revocation, password reset, email confirmation, lockout, and disabled account.
3. Web cookie and API bearer-token decision table, including CSRF implications and token storage prohibitions.
4. Access/refresh/session expiry, rotation, hashing, replay detection, revocation-family, and security-stamp expectations.
5. Role evolution: Anonymous, Customer, Admin, SuperAdmin first; specialized staff later.
6. Permission catalog and policy/ownership matrix by module/action.
7. Broken-access-control rules: server-side policy, object ownership, 403/404 behavior, list-query filtering, and negative tests.
8. Super-admin bootstrap and emergency-access constraints without credentials.
9. Admin MFA production gate/compensating-control decision.
10. Audit-event catalog: actor, action, target, result, reason, correlation, redacted metadata, retention class, and fail-closed/fail-open rule.
11. Authentication/authorization failure matrix and user-enumeration-safe responses.
12. Phase 1 implementation decisions and blockers.

## Questions To Answer

- Confirm secure Web cookie plus API bearer-token responsibilities and whether both are needed in MVP.
- What are proposed access/refresh expiries, and who approves them?
- Which role/permission can assign or revoke each role without self-escalation?
- How is first super admin bootstrapped locally without hardcoded credentials?
- Which sensitive actions fail if audit persistence fails?
- Is admin MFA required before production or earlier?

## Contract And Safety Rules

| Concern | Rule |
| --- | --- |
| Architecture | Core owns access concepts/policies; Infrastructure owns Identity, token signing, cookies, and persistence. |
| Data | Store password hashes via Identity and refresh-token hashes only; minimize profile/identity duplication. |
| API/UI | UI hiding is convenience only; every endpoint/query enforces policy and ownership server-side. |
| Security | Least privilege, no self-escalation, short-lived access, rotation/reuse detection, generic login failure. |
| Logging/Audit | Never log passwords, tokens, cookies, auth headers, reset codes, or full PII; sensitive security/admin actions are audited. |
| Failure | Missing secrets fail startup; token replay revokes the family; sensitive unaudited actions fail safely. |

## Required Diagrams

- Mermaid registration/login/session flow.
- Mermaid access-token/refresh rotation and reuse-detection sequence.
- Mermaid authorization decision flow: authenticate, role/permission, ownership/query filter, allow/deny/audit.
- Mermaid sensitive admin action plus audit transaction flow.

## Step-By-Step Instructions

1. Reconcile Phase 0 cookie guidance with Phase 1 API bearer guidance and record the canonical boundary.
2. Define identity/profile/role/permission/session concepts and owners.
3. Describe every auth lifecycle flow and safe failure result.
4. Define token/session storage, rotation, replay, revocation, and expiry decisions as proposed/approved.
5. Create role evolution and permission/action matrices.
6. Define object ownership and privacy-preserving list/detail access rules.
7. Define super-admin bootstrap and self-escalation controls.
8. Create audit catalog and audit-failure policy.
9. Draw required flows and derive negative security test expectations.
10. Request explicit security and product approval.

## Expected File Areas

- Create/update only `artifacts/identity-access-audit.md` and Packet 06 status.

## Verification And Evidence

| Check | Evidence |
| --- | --- |
| Flow coverage | Registration through revocation/recovery documented. |
| Role/permission coverage | Every privileged action has a server-side policy. |
| Ownership | Detail and list query behavior prevent BOLA/IDOR. |
| Token safety | Hashing, rotation, replay, revocation, and expiry defined. |
| Audit | Sensitive actions and failure policy defined. |
| Scope | No credentials, code, config, or migrations changed. |

```powershell
rg -n "register|login|refresh|rotation|reuse|revok|permission|ownership|bootstrap|audit" docs/implementation-guides/phase-0/artifacts/identity-access-audit.md
rg -n "password|token|cookie|authorization header|secret" docs/implementation-guides/phase-0/artifacts/identity-access-audit.md
git diff --check
git diff --name-only
```

The second search is a manual redaction review, not an automatic failure. Build/test evidence may reuse Packet 01.

## Review Checklist

- Identity and profile ownership are distinct.
- Cookie/bearer/CSRF responsibilities are clear.
- Refresh tokens are never stored plaintext.
- Role/permission/ownership checks are server-side.
- Self-escalation and user enumeration are prevented.
- Audit failures have explicit behavior.

## Acceptance Criteria

- Artifact and four diagrams exist.
- Security reviewer approves auth/session/access/audit contracts.
- Product owner approves role capabilities and privacy behavior.
- Phase 1 has no unresolved high-risk identity decision except explicitly blocked values.
- No authentication implementation or secret was added.

## Manual Review And Stop Conditions

Stop if cookie/bearer behavior remains contradictory, bootstrap requires hardcoded credentials, a role can grant itself privilege, plaintext tokens are proposed, or a sensitive action has no audit-failure rule.

## Rollback

Revert only the identity/access artifact and Packet 06 status.

## Handoff

Packet 07 consumes auth/admin trust controls. Packet 09 records the approved identity, permission, ownership, and audit contracts for Phase 1.

## Copy-Ready Execution Prompt

```text
Execute Phase 0 Packet 06 from docs/implementation-guides/phase-0/06-identity-access-and-audit.md after Packets 02, 04, and 05 are approved. Create only artifacts/identity-access-audit.md and the allowed status update. Reconcile cookie/bearer guidance, define auth/session/token/RBAC/ownership/bootstrap/audit contracts and diagrams, prohibit plaintext secrets and self-escalation, run verification, and stop for security review on any contradiction.
```

