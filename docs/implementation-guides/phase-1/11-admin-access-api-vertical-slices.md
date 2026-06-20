# Packet 11: Admin User And Access-Control API Vertical Slices

## Packet Metadata

| Field | Value |
| --- | --- |
| Status | Blocked - PHASE0-GATE |
| Depends On | Packets 07-09 Approved |
| Layer Ownership | Core admin/access queries/commands; Infrastructure persistence; API policy/mapping |
| Produces | Admin user list, role assignment, effective permissions APIs/tests, `evidence/11-completion.md` |
| Human Review | Security reviewer, product owner, API owner |

## Objective

Expose the minimum Phase 1 admin foundation APIs with least privilege, privacy-safe user listing, audited role changes, self-escalation prevention, bounded queries, and exhaustive negative authorization tests.

## Read First And Prerequisites

- Approved Phase 0 API/identity/security contracts.
- Packet 07 access policies/bootstrap, Packet 08 audit, Packet 09 authentication.
- Phase 1 roadmap sections 7-8, 12, 16-19.

## In Scope

- `GET /api/v1/admin/users`
- `POST /api/v1/admin/users/{userId}/roles`
- `GET /api/v1/admin/permissions/me`
- Admin query/command services, permission policies, pagination/filter/sort allowlists, minimal DTOs, role assignment validation/concurrency/idempotency, audit, OpenAPI, tests.

## Out Of Scope

- General user editing/deletion, password reset, audit-log reader, permission CRUD, role CRUD UI, full admin portal, impersonation, bulk operations, specialized future staff roles, or later-phase admin features.

## Likely File Areas To Inspect

- Core access/admin queries/commands/results/pagination contracts.
- Infrastructure identity/access query implementations.
- API admin controllers/endpoints/policies/mappers/OpenAPI.
- Authorization, BOLA/IDOR, pagination, audit, and concurrency tests.

## Architecture And Prohibited Dependencies

- API requires named permission policies and calls application services; no role-table/DbContext access in controllers.
- Core pagination/query contracts remain HTTP/EF-neutral.
- Infrastructure applies authorization-compatible data minimization/filtering; UI/client filtering is not security.
- Do not grant SuperAdmin behavior through an unreviewed `Admin` role shortcut.

## API Contract

| Method/Route | Authorization | Contract |
| --- | --- | --- |
| `GET /api/v1/admin/users` | `users.read` | Page 1/20 default, max 100; allowlisted email/role/status/sort; minimal safe identity/profile summary. |
| `POST /api/v1/admin/users/{userId}/roles` | SuperAdmin or approved `users.roles.assign` policy | Explicit add/remove command, target/role validation, no self-escalation, audit; canonical conflict/forbidden behavior. |
| `GET /api/v1/admin/permissions/me` | Admin/SuperAdmin | Current effective permission codes/version only; no internal mappings/other users. |

Clarify whether one route handles add/remove or separate command routes before coding. Follow approved API contract; do not invent verbs ad hoc.

## Database Changes And Migrations

None expected.

## Security, Logging, And Failure Rules

- Anonymous 401; authenticated customer/admin without permission 403; privacy-sensitive target may use 404 per contract.
- List returns minimum fields; no password hash, stamp, lockout internals, token/session, audit metadata, full phone/address, or unnecessary PII.
- Filters/sorts are allowlisted and page size bounded.
- Role changes re-check current actor permission, target, protected roles, self-escalation, concurrency, and session/permission invalidation.
- Sensitive role write and audit follow Packet 08 mandatory transaction rule.
- Log IDs/action/result/correlation, not full profile or request body.

## Implementation Steps

1. Confirm route shape for add/remove and list field/filter/sort allowlists.
2. Define framework-neutral paged admin user query and minimal DTO.
3. Implement permission-protected list query and API tests.
4. Define role assignment command/result with concurrency/idempotency expectation.
5. Implement role endpoint using Packet 07 service and Packet 08 audit transaction.
6. Implement current effective permissions endpoint without exposing storage mappings.
7. Add policies/metadata/OpenAPI and comprehensive negative authorization tests.
8. Test self-escalation, protected-admin, stale-session invalidation, enumeration/PII, and duplicate/concurrent assignment.
9. Inspect logs/responses, run all checks, create evidence, obtain security/product approval.

## Test Matrix

| Test | Required Cases |
| --- | --- |
| User list | Authorized; anonymous/customer/admin-without-permission denied; page/max/filter/sort allowlists; minimal fields. |
| Role assignment | Authorized; unauthorized; self-escalation; protected/last admin; invalid user/role; duplicate; concurrent; audit failure. |
| Effective permissions | Admin/SuperAdmin; customer denied; stable codes only; change invalidation reflected. |
| Privacy | No tokens/hashes/stamps/audit/full PII; safe 403/404 behavior. |
| Architecture | Thin controller; no direct EF/Identity table access. |

## Verification Commands

```powershell
dotnet build ECommerceSolution.slnx
dotnet test ECommerceSolution.slnx --no-build
rg -n "PasswordHash|SecurityStamp|TokenHash|RefreshToken|Address|PhoneNumber|DbContext|UserManager" ECommerce.API
git diff --check
git diff --stat
git status --short
```

Review matches for DTO leakage and controller framework access.

## Acceptance Criteria

- Three admin/access endpoints match approved contracts.
- Permission policies deny by default and negative tests cover every role.
- User list is bounded, allowlisted, and privacy-minimal.
- Role changes prevent self-escalation/protected-admin loss and are transactionally audited.
- Effective permission changes invalidate/refresh access as designed.
- No broad admin CRUD/UI/audit reader/future role/later feature or migration was added.
- Evidence and security/product/API approval exist.

## Security Stop Conditions

Stop on role-only privilege shortcut, fail-open permission lookup, self-escalation, ability to remove all recovery admins, PII/internal-field exposure, unbounded queries, role change without required audit, direct EF controller access, or schema drift.

## Rollback And Handoff

Do not strand the environment without a recovery admin. Revert endpoint/code changes carefully and preserve role/audit history. Handoff the complete foundation to Packet 12.

## Copy-Ready Coding Prompt

```text
Execute Phase 1 Packet 11 from docs/implementation-guides/phase-1/11-admin-access-api-vertical-slices.md only after PHASE0-GATE and Packets 07-09 approval. Implement only the approved admin user list, role assignment/removal command route, and current effective-permissions endpoint with thin controllers, named permission policies, bounded privacy-minimal queries, self-escalation/protected-admin controls, mandatory audit, OpenAPI, and exhaustive negative tests. Add evidence/11-completion.md, run checks, and stop on privilege, PII, query, audit, recovery-admin, direct EF, or schema concerns.
```
