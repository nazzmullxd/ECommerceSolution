# Packet 10: Current User Profile API Vertical Slices

## Packet Metadata

| Field | Value |
| --- | --- |
| Status | Blocked - PHASE0-GATE |
| Depends On | Packets 05, 07, 08, And 09 Approved |
| Layer Ownership | Core profile service/rules; Infrastructure persistence; API mapping/ownership |
| Produces | Current/update profile APIs and tests, `evidence/10-completion.md` |
| Human Review | API owner, security/privacy reviewer |

## Objective

Implement authenticated self-profile read/update vertical slices with strict ownership, field allowlists, validation, concurrency/failure behavior, minimal responses, and redacted audit/logging.

## Read First And Prerequisites

- Approved Phase 0 API/data/identity/privacy contracts.
- Packet 04 CustomerProfile mapping, Packet 05 account service, Packet 08 audit, Packet 09 authentication/current-user mechanism.
- Phase 1 roadmap sections 7, 14-19.

## In Scope

- `GET /api/v1/users/me`
- `PUT /api/v1/users/me`
- Framework-neutral profile query/update application service if not already present.
- Current-user ID resolution, ownership by construction, allowlisted fields, validation, safe DTOs, audit of changed field names, concurrency choice, tests/OpenAPI.

## Out Of Scope

- Addresses, avatar/image upload, preferences, public profiles, admin editing, arbitrary user ID routes, password/email/security changes, order data, or Phase 3 customer experience.

## Likely File Areas To Inspect

- Core profile commands/queries/results/contracts.
- Infrastructure profile persistence/repository if needed.
- API users/me controller/endpoints/mappers/OpenAPI.
- Current-user abstraction and profile/API/security tests.

## Architecture And Prohibited Dependencies

- Core profile rules cannot depend on HTTP claims, EF, Identity, or API DTOs.
- API resolves current authenticated user and passes stable user ID; client cannot submit owner ID.
- Infrastructure updates only allowlisted profile columns; no mass assignment of identity/admin fields.
- Controller does not query DbContext directly.

## API Contract And UI Boundary

| Method/Route | Auth | Contract |
| --- | --- | --- |
| `GET /api/v1/users/me` | Authenticated Customer/Admin/SuperAdmin | Minimal safe profile; no hashes, roles storage, audit, lockout, token/session metadata. |
| `PUT /api/v1/users/me` | Authenticated same owner | Allowed profile fields only; validated; returns safe updated profile or approved no-content. |

Decide whether email/phone changes are profile-only or security workflows. Default: email/password/roles/account state are excluded from this update. No UI implementation is required; Web can consume services later.

## Database Changes And Migrations

None expected. Stop on schema need and return to Packet 04 review.

## Security, Logging, And Failure Rules

- Ownership uses authenticated subject, never request `userId`.
- Do not expose whether another user/profile exists; no arbitrary-ID endpoint.
- Prevent overposting/mass assignment with explicit request DTO and mapping.
- Logs contain user ID/result/changed field names only; no full old/new values, phone, email, or body.
- Missing own profile follows approved repair/error behavior without leaking internals.
- Concurrency conflict returns canonical 409 if versioning is used.

## Implementation Steps

1. Confirm allowed fields, email/phone boundary, response fields, and concurrency policy.
2. Define/complete framework-neutral profile query/update service and validation.
3. Implement persistence with explicit field mapping and ownership key.
4. Add authenticated current-user resolution abstraction if not already approved.
5. Implement GET `/users/me` thin endpoint and tests.
6. Implement PUT `/users/me` thin endpoint, audit changed field names, and tests.
7. Add Problem Details/correlation/OpenAPI and negative/mass-assignment tests.
8. Inspect logs/responses, run all checks, create evidence, request privacy/security review.

## Test Matrix

| Test | Required Cases |
| --- | --- |
| Read | Authenticated own profile; anonymous 401; missing profile safe failure. |
| Update | Valid allowed fields; invalid lengths/formats; unknown fields ignored/rejected by serializer contract. |
| Ownership | No client owner ID; cannot access/update another user. |
| Overposting | Cannot change email/password/roles/permissions/active/security fields. |
| Audit/logs | Changed field names only; no full values/body/PII. |
| Concurrency | Approved conflict behavior. |

## Verification Commands

```powershell
dotnet build ECommerceSolution.slnx
dotnet test ECommerceSolution.slnx --no-build
rg -n "PasswordHash|SecurityStamp|TokenHash|Roles|Permissions|Request.Body|PhoneNumber|Email" ECommerce.API ECommerce.Core ECommerce.Infrastructure
git diff --check
git diff --stat
git status --short
```

Inspect matches for intended validation versus leakage/overposting.

## Acceptance Criteria

- Authenticated users can read/update only their own allowlisted profile fields.
- Request/response DTOs expose no identity security/admin internals.
- Mass assignment and cross-user access are impossible and tested.
- Audit/logs are privacy-safe.
- No address/admin/password/email-security/later feature or migration was added.
- Evidence and security/privacy/API approval exist.

## Security Stop Conditions

Stop on client-supplied owner ID, generic entity binding, email/password/role/account mutation, PII body/value logging, arbitrary user route, direct DbContext controller access, or schema change.

## Rollback And Handoff

Revert only Packet 10 profile/API/tests; preserve identity/schema/audit history. Handoff safe profile/current-user patterns to later customer-owned modules.

## Copy-Ready Coding Prompt

```text
Execute Phase 1 Packet 10 from docs/implementation-guides/phase-1/10-profile-api-vertical-slices.md only after PHASE0-GATE and Packets 05,07-09 approval. Implement only GET/PUT /api/v1/users/me with framework-neutral profile services, authenticated-subject ownership, explicit field mapping, validation, privacy-safe audit/logging, Problem Details/OpenAPI, and tests. Add evidence/10-completion.md, run checks, and stop on overposting, client owner IDs, PII logging, security-field mutation, direct EF controller access, or schema concerns.
```
