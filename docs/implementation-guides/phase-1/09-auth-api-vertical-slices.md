# Packet 09: Authentication API Vertical Slices

## Packet Metadata

| Field | Value |
| --- | --- |
| Status | Blocked - PHASE0-GATE |
| Depends On | Packets 03 and 05-08 Approved; `P1-AUTH-001`, `P1-AUTH-003` |
| Layer Ownership | API transport/mapping; Core application services; Infrastructure adapters already implemented |
| Produces | Register/login/refresh/logout APIs and tests, `evidence/09-completion.md` |
| Human Review | API owner and security reviewer |

## Objective

Expose the approved registration, login, refresh, and logout services through thin `/api/v1/auth` endpoints with canonical validation/errors, safe token transport, correlation, audit, and abuse controls.

## Read First And Prerequisites

- Approved Phase 0 API/identity/security contracts.
- Packets 03, 05, 06, 07, and 08 evidence/contracts.
- Phase 1 roadmap sections 7, 10-11, 14-19.
- Current API Program/controllers/http file/test host.

## In Scope

- `POST /api/v1/auth/register`
- `POST /api/v1/auth/login`
- `POST /api/v1/auth/refresh`
- `POST /api/v1/auth/logout`
- Request/response DTO mapping, endpoint auth metadata, approved bearer/refresh transport, Problem Details, correlation, content type, safe local abuse controls/lockout behavior, OpenAPI descriptions, and API/security tests.

## Out Of Scope

- Profile/admin endpoints, password reset/change, email delivery, MFA, social login, Web login UI/cookie flow unless `P1-AUTH-001` explicitly includes it, production WAF/rate infrastructure, or later-phase APIs.

## Likely File Areas To Inspect

- API auth controllers/endpoints, request/response models/mappers, `Program.cs`, auth middleware/policies, Swagger/OpenAPI, `.http` examples.
- Core/Infrastructure services from earlier packets only for defects found through contract tests; do not refactor them casually.
- API/integration/security tests.

## Architecture And Prohibited Dependencies

- Endpoints validate/map/call application services; no `DbContext`, `UserManager`, token hashing, role-table query, or business rule in controllers.
- Core has no HTTP/cookie/ProblemDetails dependency.
- Infrastructure details never appear in public DTOs/errors.
- Authentication middleware must run before authorization.

## API Contract

| Method/Route | Auth | Success | Key Failures |
| --- | --- | --- | --- |
| `POST /api/v1/auth/register` | Anonymous | `201` safe user/session result per confirmation decision | 400 invalid, safe 409/validation duplicate policy |
| `POST /api/v1/auth/login` | Anonymous | `200` safe user/access/refresh transport | Generic 401 invalid/locked/inactive/unconfirmed as approved |
| `POST /api/v1/auth/refresh` | Approved refresh credential transport | `200` rotated access/refresh result | Generic 401 expired/revoked/reused; family revoked on reuse |
| `POST /api/v1/auth/logout` | Authenticated/current session | `204` or approved no-content result; repeat safe | 401 missing/invalid auth; no token disclosure |

Requests/responses use camelCase and canonical Problem Details. Never return token hash, family ID, security stamp, lockout internals, permission storage metadata, or audit details.

## Database Changes And Migrations

None. Stop and return to owning packet if API tests reveal schema changes.

## Security, Logging, And Failure Rules

- Apply approved transport: do not improvise localStorage, cookie, SameSite, CSRF, or dual-mode behavior.
- Never log auth request bodies, passwords, tokens, cookies, authorization headers, or response credentials.
- Generic login/refresh failure resists enumeration/token-state disclosure.
- Require JSON/content limits and bounded fields.
- Identity lockout is mandatory; endpoint rate limiting follows approved Phase 1 scope and must not create account-lockout abuse without review.
- Refresh response is sent only after successful atomic persistence.

## Implementation Steps

1. Confirm prerequisites and approved transport/status/error contracts.
2. Configure authentication middleware/validation and pipeline order.
3. Define API request/response DTOs separately from entities/framework types.
4. Implement register endpoint and API tests.
5. Implement login endpoint with generic failures and redaction tests.
6. Implement refresh endpoint with approved credential extraction/rotation and replay tests.
7. Implement logout endpoint with current-session revocation and idempotent repeat behavior.
8. Add endpoint metadata/OpenAPI examples using fake values only.
9. Add auth/validation/error/content/abuse/security tests and inspect logs.
10. Run all checks, create evidence, obtain security/API approval.

## Test Matrix

| Endpoint | Required Cases |
| --- | --- |
| Register | Valid; validation; weak password; duplicate safe result; default role/profile/audit. |
| Login | Valid; wrong/unknown same result; locked; inactive; unconfirmed; no credential logs. |
| Refresh | Valid rotation; expired; revoked; reused family revocation; concurrent double refresh. |
| Logout | Valid; repeated; missing/invalid access; session revoked. |
| Pipeline | Authentication before authorization; Problem Details/correlation/content type. |
| Security | No secret DTO fields/logs/OpenAPI real values; bounded requests. |

## Verification Commands

```powershell
dotnet build ECommerceSolution.slnx
dotnet test ECommerceSolution.slnx --no-build
rg -n "password|accessToken|refreshToken|Authorization|Cookie|SecurityStamp|TokenHash" ECommerce.API
git diff --check
git diff --stat
git status --short
```

Review all matches; request/response property names are expected, values/logging are not.

## Acceptance Criteria

- Four auth endpoints match approved contracts and use application services.
- Login/refresh failures are generic and replay-safe.
- Token transport follows approved decision and no credentials are logged.
- Pipeline order, Problem Details, correlation, validation, OpenAPI, and tests pass.
- No profile/admin/reset/MFA/UI/later features or migration was added.
- Evidence and security/API approval exist.

## Security Stop Conditions

Stop on token/password logging, unapproved transport, localStorage guidance, enumeration, refresh-before-persist, missing auth pipeline, controller persistence/business logic, real secrets in examples, or schema change.

## Rollback And Handoff

Revoke test sessions if needed. Revert only Packet 09 API/config/tests; preserve underlying approved services/schema. Handoff authenticated-current-user behavior to Packets 10-11.

## Copy-Ready Coding Prompt

```text
Execute Phase 1 Packet 09 from docs/implementation-guides/phase-1/09-auth-api-vertical-slices.md only after PHASE0-GATE and Packets 03,05-08 plus transport decisions are approved. Implement only register/login/refresh/logout API slices as thin mappings over approved services, configure correct auth pipeline and safe transport, add Problem Details/correlation/OpenAPI/API/security tests and evidence/09-completion.md, run all checks, and stop on credential leakage, enumeration, replay, transport, controller-logic, or schema concerns.
```

