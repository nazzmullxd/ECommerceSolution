# Packet 05: Identity Registration, Login, And Account Safety

## Packet Metadata

| Field | Value |
| --- | --- |
| Status | Blocked - PHASE0-GATE |
| Depends On | Packet 04 Approved; decision `P1-AUTH-002` |
| Layer Ownership | Core application contracts/rules; Infrastructure ASP.NET Identity adapter |
| Produces | Tested registration/login/account service, `evidence/05-completion.md` |
| Human Review | Security reviewer, solution architect, product owner |

## Objective

Implement safe application services for registration, login, account state, password/lockout behavior, and user-enumeration resistance before exposing HTTP endpoints.

## Read First And Prerequisites

- Approved Phase 0 identity/security contracts.
- Packet 03 error/validation/logging contract and Packet 04 model.
- Phase 1 roadmap sections 8, 10, 14, 16-19, and 24.
- Current ASP.NET Identity configuration and tests.

## In Scope

- Identity adapter/service contracts.
- Password policy and Identity hashing configuration.
- Registration transaction/workflow: user, profile, default Customer role, required audit contract call.
- Login validation, lockout, inactive/unconfirmed-account policy, generic failures, and safe result model.
- Account security-stamp/session-revocation hooks needed by Packet 06.

## Out Of Scope

- JWT/refresh token issuing, HTTP endpoints, real email sending, password-reset endpoint, MFA implementation, admin roles/bootstrap, customer addresses, or later-phase modules.

## Likely File Areas To Inspect

- Core identity/profile contracts, commands/results/validation.
- Infrastructure Identity types/configuration/services and DI.
- Tests for Identity persistence/application services.
- Audit contract from Packet 04; implementation arrives in Packet 08.

## Architecture And Prohibited Dependencies

- Core defines use cases and safe results; no `UserManager`, `SignInManager`, Identity result types, hashing implementation, or HTTP context in Core.
- Infrastructure translates Identity framework outcomes into Core/application outcomes.
- Registration orchestration must have an explicit consistency strategy across user/profile/role/audit.
- Do not generate tokens or return framework entities.

## Data, API, And UI Contracts

- Inputs: normalized/validated email, password supplied transiently, allowed profile fields only.
- Outputs: safe user/account summary or generic failure category; never hashes/stamps/internal lockout details.
- No API route in this packet. Packet 09 maps services to HTTP.
- Decide and document duplicate-registration response privacy separately from internal unique enforcement.

## Security, Logging, And Failure Rules

- ASP.NET Identity performs password hashing; plaintext password lifetime is limited to request/service call and never logged/persisted elsewhere.
- Generic login failure must not distinguish unknown email from wrong password.
- Lockout increments and thresholds follow approved policy; avoid permanent denial through unbounded attacker lockout.
- Inactive/unconfirmed behavior follows `P1-AUTH-002` without leaking account status publicly.
- Registration partial failure must not leave an active account missing required profile/role/audit consistency without recovery.

## Implementation Steps

1. Confirm `P1-AUTH-002` and approved password/lockout defaults.
2. Define Core commands/results/interfaces and validation without framework types.
3. Configure ASP.NET Identity password, lockout, email uniqueness/normalization, and account requirements in Infrastructure.
4. Implement registration orchestration for user/profile/default role with explicit transaction/compensation and audit contract invocation.
5. Implement login verification with lockout/inactive/unconfirmed checks and generic external failure.
6. Add security-stamp/account-state hooks needed for later session invalidation.
7. Register services in Infrastructure/composition root without exposing endpoints.
8. Add unit/integration/security tests, including race/duplicate registration and user enumeration.
9. Search logs/errors for prohibited values, run all checks, write evidence, request security review.

## Database Changes And Migrations

No expected schema change. If Identity configuration reveals a required schema change, stop and return to Packet 04 migration review.

## API Contract

No endpoint. Preserve future contracts for `POST /api/v1/auth/register` and `POST /api/v1/auth/login`; HTTP status/DTO mapping belongs to Packet 09.

## Test Matrix

| Test | Required Cases |
| --- | --- |
| Registration | Valid; invalid email; weak password; duplicate/race; profile/default role; partial failure. |
| Login | Valid; wrong password; unknown email same public result; locked; inactive; unconfirmed per decision. |
| Password | Identity hash only; policy enforced; plaintext absent from persistence/logs. |
| Enumeration | Timing/message/status category sufficiently generic at service contract. |
| Audit contract | Required registration/login security events invoked with redacted metadata. |
| Architecture | Core has no Identity framework types. |

## Verification Commands

```powershell
dotnet build ECommerceSolution.slnx
dotnet test ECommerceSolution.slnx --no-build
rg -n "password|PasswordHash|SecurityStamp|AccessFailed|Lockout" ECommerce.Core ECommerce.Infrastructure ECommerce.API
git diff --check
git diff --stat
git status --short
```

Review matches for leakage and dependency boundaries.

## Acceptance Criteria

- Registration/login services are framework-neutral at Core boundary and tested.
- Password hashing/policy/lockout use approved Identity configuration.
- Public failures resist user enumeration.
- Registration consistency and audit invocation are explicit.
- No token issuing, HTTP endpoint, real email, bootstrap, or later feature was added.
- Evidence and security/architecture approval exist.

## Security Stop Conditions

Stop on password logging/storage, account-enumerating failures, partial registration without safe recovery, Core Identity dependency, unapproved email-confirmation policy, or schema change outside Packet 04.

## Rollback And Handoff

Revert only Packet 05 behavior/config/tests. Preserve approved schema. Hand safe registration/login/account contracts to Packets 06, 08, and 09.

## Copy-Ready Coding Prompt

```text
Execute Phase 1 Packet 05 from docs/implementation-guides/phase-1/05-identity-registration-login-and-account-safety.md only after PHASE0-GATE, Packet 04 approval, and decision P1-AUTH-002. Implement only framework-neutral registration/login/account services backed by ASP.NET Identity, with password/lockout policy, generic failures, registration consistency, audit contract calls, and tests. Add evidence/05-completion.md, run all checks, and stop on password leakage, enumeration, partial-state, schema, or dependency concerns. Do not add tokens or endpoints.
```
