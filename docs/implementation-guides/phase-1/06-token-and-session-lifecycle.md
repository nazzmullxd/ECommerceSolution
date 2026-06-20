# Packet 06: Access Token And Session Lifecycle

## Packet Metadata

| Field | Value |
| --- | --- |
| Status | Blocked - PHASE0-GATE |
| Depends On | Packets 04-05 Approved; decisions `P1-AUTH-001`, `P1-AUTH-003` |
| Layer Ownership | Core token/session contracts and decisions; Infrastructure crypto/JWT/persistence |
| Produces | Token/session service with rotation/reuse/revocation tests, `evidence/06-completion.md` |
| Human Review | Security reviewer and solution architect |

## Objective

Implement short-lived access-token issuing/validation contracts and opaque refresh-token session lifecycle with hashing, atomic rotation, reuse detection, revocation, logout, and safe failure behavior.

## Read First And Prerequisites

- Approved Phase 0 identity/API/security contracts.
- Packet 02 secret/config rules, Packet 04 refresh model, Packet 05 account-state hooks.
- Phase 1 roadmap sections 7-8, 11, 16-19, and 20.
- Current JWT package/configuration and official compatibility/security guidance at execution time.

## In Scope

- Typed token options and startup validation.
- Core token/session abstractions and safe result types.
- JWT access token creation with minimal claims.
- Cryptographically random opaque refresh tokens returned only when created/rotated; hash persistence only.
- Token family, rotation, replay/reuse detection, revocation, logout, password/security-stamp invalidation, expiry cleanup contract, and concurrency safety.

## Out Of Scope

- HTTP endpoints/cookies, external identity/OAuth, MFA, real key-management service, device-management UI, payment tokens, or production key rotation automation.

## Likely File Areas To Inspect

- Core identity/token contracts/value objects.
- Infrastructure Identity/token services, refresh persistence, options/DI.
- API auth configuration only if required for validation registration; endpoint mapping remains Packet 09.
- Token/security/integration tests.

## Architecture And Prohibited Dependencies

- Core must not reference JWT libraries, signing credentials, Identity framework, EF Core, `HttpContext`, cookies, or bearer middleware.
- Infrastructure owns randomness, hashing, JWT signing, token persistence, clocks/adapters.
- API composition may configure bearer validation after transport decision; HTTP extraction belongs Packet 09.
- Use an injectable clock/random abstraction where needed for deterministic tests; do not weaken production randomness.

## Token And Session Contract

| Area | Required Rule |
| --- | --- |
| Access token | Short lived (roadmap proposed 15 minutes), issuer/audience/signature/expiry validated, minimal claims. |
| Signing key | Secret/user-secrets/env only; minimum strength validated; never logged. |
| Refresh token | Opaque CSPRNG value; store cryptographic hash only; return plaintext only at issue/rotation boundary. |
| Family | Stable family ID; one-time token use; replacement link; atomic rotation. |
| Reuse | Reuse of rotated/revoked token revokes family/session and emits high-risk audit event. |
| Logout | Current session revoked; repeat logout safe/idempotent. |
| Account change | Password/security-stamp/disable revokes or invalidates active sessions. |
| Claims | Subject, minimal email if approved, roles/permission-version marker, issued/expiry/issuer/audience; no profile/address/secrets. |

## API And UI Contracts

No endpoint in this packet. Packet 09 owns `/auth/refresh` and `/auth/logout` transport. `P1-AUTH-003` must decide browser/API refresh-token transport/storage; do not default to browser local storage.

## Security, Logging, And Failure Rules

- Never log/serialize plaintext token outside intended auth response; never log token hash, signing key, cookie, or authorization header.
- Compare hashes safely and make rotate/reuse decisions transactionally with concurrency protection.
- Expired/revoked/unknown/reused refresh tokens return generic authentication failure externally.
- Missing/weak issuer/audience/signing configuration fails startup without echoing values.
- Database failure during rotation issues no usable unpersisted replacement session.

## Implementation Steps

1. Approve `P1-AUTH-001` and `P1-AUTH-003`, lifetimes, claims, issuer/audience, and admin-session policy.
2. Define framework-neutral Core token/session commands/results/interfaces.
3. Implement typed options/startup validation using secret references, never tracked key values.
4. Implement access token generation and validation configuration with minimal claims.
5. Implement opaque refresh generation and one-way hash persistence.
6. Implement atomic rotation and family/replacement linkage.
7. Implement expired/revoked/reused handling; reuse revokes family and records audit contract event.
8. Implement logout/session revocation and account security-stamp invalidation behavior.
9. Add cleanup query/worker contract only; do not build unrelated background infrastructure.
10. Add unit, integration, concurrency, replay, config, and redaction tests.
11. Run vulnerability/build/test/scope checks, record evidence, obtain security approval.

## Database Changes And Migrations

No expected schema change beyond Packet 04. Stop and return to Packet 04 review if additional token integrity fields/indexes are required.

## API Contract

Service outputs must support future login/refresh responses without exposing internal hash/family/security metadata. HTTP details remain Packet 09.

## Test Matrix

| Test | Required Cases |
| --- | --- |
| Access | Valid claims/signature/issuer/audience/expiry; expired/invalid rejected. |
| Refresh | Random/unique; hash-only persistence; valid rotation; old token invalid. |
| Reuse | Rotated/revoked token reuse revokes family and emits audit event. |
| Concurrency | Two refresh attempts cannot both succeed. |
| Logout | Current session revoked; repeated logout safe. |
| Account change | Password/disable/security stamp invalidates sessions. |
| Configuration | Missing/weak secret metadata fails safely. |
| Redaction | Tokens/hashes/keys absent from logs and evidence. |

## Verification Commands

```powershell
dotnet build ECommerceSolution.slnx
dotnet test ECommerceSolution.slnx --no-build
dotnet list ECommerceSolution.slnx package --vulnerable --include-transitive
rg -n "accessToken|refreshToken|TokenHash|SigningKey|Authorization|Cookie" ECommerce.API ECommerce.Core ECommerce.Infrastructure
git diff --check
git diff --stat
git status --short
```

Inspect all matches; never paste token values into evidence.

## Acceptance Criteria

- Access tokens are short-lived, minimally claimed, and fully validated.
- Refresh tokens are opaque, hash-only, rotated atomically, revocable, and replay-detected.
- Concurrent refresh cannot issue two valid replacements.
- Missing/weak secret configuration fails safely.
- Core has no JWT/Identity/HTTP/EF dependency.
- No endpoints, cookies, MFA, cloud keys, or later-phase features were added.
- Evidence and security approval exist.

## Security Stop Conditions

Stop on plaintext token persistence/logging, localStorage assumption, weak/hardcoded signing key, non-atomic rotation, replay without family revocation, overbroad claims, schema drift, or JWT dependencies in Core.

## Rollback And Handoff

Revert only Packet 06 code/config/tests. Revoke any test sessions if needed. Hand approved token/session contracts and transport decision to Packets 08-09.

## Copy-Ready Coding Prompt

```text
Execute Phase 1 Packet 06 from docs/implementation-guides/phase-1/06-token-and-session-lifecycle.md only after PHASE0-GATE, Packets 04-05, and decisions P1-AUTH-001/P1-AUTH-003. Implement only framework-neutral token/session contracts plus Infrastructure JWT and opaque hash-only refresh lifecycle with atomic rotation, reuse-family revocation, logout, config validation, and tests. Add evidence/06-completion.md, run all checks, and stop on plaintext tokens, weak secrets, non-atomic replay handling, schema, claims, transport, or dependency concerns. Do not add endpoints.
```

