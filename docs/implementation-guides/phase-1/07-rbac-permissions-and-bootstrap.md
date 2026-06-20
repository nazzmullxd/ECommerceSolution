# Packet 07: RBAC, Permissions, And Super-Admin Bootstrap

## Packet Metadata

| Field | Value |
| --- | --- |
| Status | Blocked - PHASE0-GATE |
| Depends On | Packets 04-05 Approved; decisions `P1-SEC-001`, `P1-BOOT-001` |
| Layer Ownership | Core permissions/policies; Infrastructure stores/seeds; API policy composition later |
| Produces | RBAC/permission service, seed/bootstrap mechanism, tests, `evidence/07-completion.md` |
| Human Review | Security reviewer, product owner, solution architect |

## Objective

Implement least-privilege roles, permissions, assignment rules, policy checks, stable idempotent seeding, and a safe one-time super-admin bootstrap without self-escalation or hardcoded credentials.

## Read First And Prerequisites

- Approved Phase 0 identity/access/security contracts.
- Packet 04 entities/mappings and Packet 05 account service.
- Phase 1 roadmap sections 8, 12, 16-19, and 24.
- Existing Identity/DI/seeding configuration and future admin endpoint contracts.

## In Scope

- Phase 1 system roles: Customer, Admin, SuperAdmin.
- Stable permissions: `users.read`, `users.update`, `users.roles.assign`, `permissions.read`, `audit.read`; future catalog/inventory codes may be reserved metadata only if approved.
- Effective-permission resolution, assignment/removal service rules, self-escalation prevention, last-super-admin safety if required, cache/version invalidation concept.
- Idempotent role/permission/role-permission seed.
- One-time local super-admin bootstrap via secrets/env, with disable-after-success behavior and mandatory audit contract.

## Out Of Scope

- HTTP admin endpoints, admin UI, specialized Phase 3/6 roles, production IAM/MFA implementation, permission management UI, product/cart/order/payment permissions, or hardcoded account credentials.

## Likely File Areas To Inspect

- Core permission constants/contracts/policies/access-control services.
- Infrastructure Identity/access repositories, seeding/bootstrap/options/DI.
- API policy registration only when needed for later endpoint authorization tests.
- Unit/integration/security tests.

## Architecture And Prohibited Dependencies

- Core permission codes and business rules remain framework-neutral.
- Infrastructure translates roles/permissions to Identity/persistence.
- API policy handlers may call Core access services; controllers cannot query role tables directly.
- Do not use role checks alone for sensitive admin actions where a permission contract exists.
- Do not place bootstrap credentials/default passwords in source, migrations, logs, or test snapshots.

## Access-Control Contract

| Actor | Allowed Foundation Behavior |
| --- | --- |
| Customer | Own profile and future owned resources; no admin actions. |
| Admin | Only explicitly granted permissions; no automatic role assignment. |
| SuperAdmin | Role/permission administration subject to no-self-escalation and audit. |
| Anonymous | Public auth actions only. |

Role assignment requires target validation, actor permission, self-escalation check, protected-role rules, audit intent, and safe concurrency/idempotency.

## API And UI Contracts

No public endpoint. Packet 11 exposes user listing/role assignment/effective-permission APIs. Define policy names/requirements that Packet 11 can consume without direct storage access.

## Security, Logging, And Failure Rules

- Deny by default when permission data is unavailable/invalid.
- UI visibility is not authorization.
- Seed/bootstrap is idempotent and cannot silently reset passwords or privileges.
- Bootstrap inputs are required only for explicit bootstrap mode, validated without printing values, and disabled/removed after success.
- Assignment/removal and denied privileged attempts invoke audit contract.
- Avoid permission claims becoming stale indefinitely; define token/security-version invalidation strategy.

## Implementation Steps

1. Approve role/permission matrix, `P1-SEC-001`, and `P1-BOOT-001`.
2. Define stable framework-neutral permission constants and access results.
3. Implement effective-permission resolution and assignment/removal services.
4. Add protected-role, self-escalation, duplicate/idempotent, last-super-admin, and concurrency rules as approved.
5. Implement role/permission seed using stable IDs/codes and no credentials.
6. Implement explicit one-time local bootstrap reading secrets/env, creating/assigning safely, auditing, and disabling after success.
7. Register authorization policies/handlers needed for tests without adding business endpoints.
8. Define permission/token invalidation behavior after role changes.
9. Add unit/integration/security/config/redaction tests.
10. Run all checks, record evidence, and obtain security/product approval.

## Database Changes And Migrations

No schema change expected. Seed data may be migration-based or runtime initialization only after Packet 02/04 review. Bootstrap user credentials must never be migration data.

## API Contract

No endpoint. Service/policy contracts must support Packet 11 routes and return safe outcomes without exposing internal role tables.

## Test Matrix

| Test | Required Cases |
| --- | --- |
| Seed | First run; repeat run; missing permission; no duplicate roles/mappings. |
| Permission | Admin allowed/denied by permission; customer/anonymous denied; unavailable store denies. |
| Assignment | Authorized; unauthorized; duplicate; self-escalation; protected/last-super-admin; concurrency. |
| Bootstrap | Explicit mode success; missing secret fails safely; repeat safe; no credential logs; disabled after success. |
| Invalidation | Role/permission change affects future effective access/session as designed. |
| Architecture | Core has no Identity/HTTP/EF policy framework dependency. |

## Verification Commands

```powershell
dotnet build ECommerceSolution.slnx
dotnet test ECommerceSolution.slnx --no-build
rg -n "SuperAdmin|users.roles.assign|Password|Seed|Bootstrap|Secret" ECommerce.Core ECommerce.Infrastructure ECommerce.API
git diff --check
git diff --stat
git status --short
```

Inspect matches for hardcoded credentials or privilege bypass.

## Acceptance Criteria

- Roles/permissions seed idempotently with stable codes.
- Sensitive actions use permissions and deny by default.
- Self-escalation/protected-admin risks are tested.
- Bootstrap is explicit, secret-backed, idempotent, auditable, and credential-safe.
- Permission changes have session/claim invalidation behavior.
- No endpoints/UI/future roles/later features were added.
- Evidence and security/product approval exist.

## Security Stop Conditions

Stop on hardcoded/bootstrap credential exposure, role-only bypass, self-escalation, ability to remove all recovery admins without approved control, fail-open permission resolution, stale privilege without invalidation, or seed migration containing a password.

## Rollback And Handoff

Do not delete the only recovery admin during rollback. Revert code/metadata seeds carefully and preserve audit history. Hand policies, permissions, bootstrap evidence, and invalidation rules to Packets 08, 09, and 11.

## Copy-Ready Coding Prompt

```text
Execute Phase 1 Packet 07 from docs/implementation-guides/phase-1/07-rbac-permissions-and-bootstrap.md only after PHASE0-GATE, Packets 04-05, and decisions P1-SEC-001/P1-BOOT-001. Implement only framework-neutral RBAC/permissions, idempotent non-secret seeds, protected assignment rules, permission invalidation, and explicit secret-backed one-time super-admin bootstrap with tests. Add evidence/07-completion.md, run checks, and stop on credential, self-escalation, fail-open, recovery-admin, seed, or dependency concerns. Do not add admin endpoints.
```

