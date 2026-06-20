# Packet 04: Guest Cart Security, Expiry, And Login Merge

## Packet Metadata

| Field | Value |
| --- | --- |
| Status | Blocked - PHASE0/PHASE1-GATE |
| Depends On | Packet 03 Approved; decision `P2-GUEST-001` |
| Layer Ownership | Cart owns guest identity/merge; Infrastructure crypto/cookie persistence; API transport |
| Risk Review | **Pre-code and post-test security plus commerce approval required** |
| Produces | Guest cart key/cookie/expiry/merge behavior and tests, `evidence/04-completion.md` |

## Objective

Implement a server-issued guest-cart bearer identity, abuse limits, expiry, anonymous cart operations, and an atomic/idempotent post-login merge that cannot expose or take over another cart.

## Read First And Prerequisites

- Approved Phase 0/1 cookie/CSRF/auth/ownership/idempotency contracts.
- Packet 01 idempotency and Packet 03 Cart aggregate/API.
- Phase 2 roadmap sections 3-4, 7, 12-15, and 18.

## In Scope

- High-entropy server-issued guest cart key; hash-only database lookup/storage.
- Approved cookie/header transport, Secure/HttpOnly/SameSite/path/domain/expiry and CSRF decision.
- Anonymous cart get/add/update/remove using possession of valid key plus bounded abuse controls.
- Guest cart expiry/abandonment cleanup contract.
- `POST /api/v1/cart/merge` after login with explicit source credential, target current-user cart, idempotency, conflict/unavailable-item behavior, key revocation, and audit.

## Out Of Scope

- Guest checkout, guest order, cross-device guest synchronization, persistent marketing identity, wishlist, CAPTCHA service, production WAF, or storing raw guest keys.

## Likely File Areas To Inspect

- Core Cart guest-owner/merge commands/results/policies.
- Infrastructure key generation/hash/lookup, Cart persistence, expiry job/clock, DI.
- API cart credential/cookie middleware/helpers/endpoints and auth transition integration.
- Security/concurrency/API/cookie/merge tests.

## Architecture And Prohibited Dependencies

- Core receives an opaque guest-cart identity abstraction/value, not `HttpContext`/cookie/crypto implementation.
- Infrastructure uses cryptographically secure random values and one-way hashes.
- API owns cookie/header extraction/set/clear; never accepts arbitrary cart ID as authority.
- Merge calls Cart service; login endpoint must not contain merge business logic.

## Guest Identity And Merge Contract

| Area | Rule |
| --- | --- |
| Key | High entropy, server-issued, unguessable bearer secret; plaintext only in protected client transport. |
| Storage | Hash only; index hash; never log/plaintext persist. |
| Cookie | Approved Secure/HttpOnly/SameSite/path/domain/lifetime; production requires HTTPS. |
| Limits | Max carts per client/rate class, max items/quantity, expiry, request/content bounds. |
| Merge target | Current authenticated customer's single active cart. |
| Merge lines | Same variant combines quantities up to limit; inactive/missing handled deterministically; no price copied as truth. |
| Completion | Source becomes Merged, key invalidated, target updated atomically, cookie cleared/rotated. |
| Replay | Same merge key/request returns original result; different payload/key conflict policy. |

## API Contract

- Packet 03 cart routes support Anonymous/Customer by resolving either valid guest credential or authenticated owner.
- `POST /api/v1/cart/merge` requires authenticated customer plus valid guest credential and `Idempotency-Key`.
- Responses disclose no guest key hash, other cart ID, owner identity, or internal unavailable-product details.

## Database Changes And Migrations

Use Packet 03 guest fields if already approved. Any missing guest-hash/merged/expiry/index/idempotency field requires a separately reviewed scoped migration. No raw key column.

## Security, Logging, And Failure Rules

- Possession of opaque key authorizes only that guest cart; authenticated ownership always wins after merge.
- Prevent session fixation by rotating/invalidating guest credential at merge/login boundary.
- Cookie-based anonymous writes require approved CSRF/SameSite analysis.
- Merge is one transaction with concurrency protection; partial source/target state is forbidden.
- Logs/audit use source/target IDs and result, never key/hash/cookie/full items/PII.
- Invalid/expired key returns safe empty/new/unauthorized behavior per approved UX without cart enumeration.

## Implementation Steps

1. Obtain pre-code approval for `P2-GUEST-001`, CSRF, expiry, limits, merge/replay/conflict rules.
2. Define Core guest identity/merge policies without HTTP/crypto dependencies.
3. Implement secure key generation, hash lookup, expiry, and invalidation in Infrastructure.
4. Implement API credential/cookie lifecycle and anonymous ownership resolution.
5. Enable anonymous Cart operations through the same Cart service with limits.
6. Implement atomic merge algorithm, target get/create, item combine, unavailable-item result, source state/key invalidation, and audit.
7. Integrate merge trigger explicitly after login or as client-called endpoint per approved contract; avoid hidden duplicate invocation.
8. Add unit/integration/API/security/concurrency/replay/cookie tests.
9. Search logs/DB/test snapshots for raw keys, run checks, create evidence, obtain post-test approval.

## Test Matrix

| Test | Required Cases |
| --- | --- |
| Key | Unique/high-entropy behavior; hash-only DB; invalid/tampered/expired rejected safely. |
| Cookie | Approved attributes; set/renew/clear; no script access; CSRF behavior tested. |
| Anonymous cart | Create/read/add/update/remove own; random cart ID cannot access another. |
| Limits | Item/quantity/cart/request/rate class bounds. |
| Merge | Empty/nonempty; same variant combine; quantity limit; inactive item; existing/no target. |
| Idempotency | Repeat same merge stable; different fingerprint conflict; concurrent merge one effect. |
| Fixation | Old key unusable after merge; source Merged; cookie cleared/rotated. |
| Logging | No raw key/hash/cookie/full cart payload. |

## Verification Commands

```powershell
dotnet build ECommerceSolution.slnx
dotnet test ECommerceSolution.slnx --no-build
rg -n "GuestCart|GuestCartKey|Set-Cookie|SameSite|HttpOnly|Secure|Request.Cookies|Response.Cookies" ECommerce.API ECommerce.Core ECommerce.Infrastructure ECommerce.Tests
git diff --check
git diff --stat
git status --short
```

Review matches for raw secret persistence/logging and HTTP leakage into Core.

## Acceptance Criteria

- Guest identity is unguessable, hash-only, bounded, expiry-controlled, and not logged.
- Anonymous operations cannot access another cart.
- Merge is atomic, concurrency/idempotency-safe, invalidates source credential, and audits safely.
- Cookie/CSRF behavior has explicit security approval/tests.
- No guest checkout/order/marketing identity or later feature was added.
- Pre-code and post-test human approvals/evidence exist.

## Security Stop Conditions

Stop on arbitrary cart ID authority, raw key/hash logging/storage, weak randomness, missing Secure/HttpOnly/SameSite/CSRF decision, session fixation, non-atomic merge, cross-user takeover, unbounded abuse, or unreviewed migration.

## Rollback / Forward-Fix And Handoff

Invalidate issued test guest keys/cookies when rolling back. Before shared data, revert scoped migration only with review; otherwise forward-fix. Hand guest owner/merge/expiry/idempotency behavior to Checkout Packet 06.

## Copy-Ready Coding Prompt

```text
Execute Phase 2 Packet 04 from docs/implementation-guides/phase-2/04-guest-cart-security-and-merge.md only after PHASE0/PHASE1 gates, Packet 03, P2-GUEST-001, and pre-code security/commerce approval. Implement only server-issued hash-only guest cart identity/cookie, bounded anonymous cart ownership, expiry, and atomic idempotent login merge with credential invalidation and tests. Add evidence/04-completion.md, run checks, then obtain post-test approval. Stop on raw keys, cookie/CSRF ambiguity, fixation, takeover, non-atomic merge, abuse, or migration concerns.
```
