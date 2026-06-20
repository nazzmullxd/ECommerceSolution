# Packet 09: Payment Webhook Validation And Settlement

## Packet Metadata

| Field | Value |
| --- | --- |
| Status | Blocked - PHASE0/PHASE1-GATE |
| Depends On | Packets 05, 07, 08 Approved; decision `P2-SETTLE-001` |
| Layer Ownership | Payments verify callbacks; application settlement orchestrates Payment/Order/Inventory; Infrastructure crypto/provider parsing |
| Risk Review | **Pre-code and post-test payment-security, commerce, database, and operations approval required** |
| Produces | Signed replay-safe webhook/settlement lifecycle, `evidence/09-completion.md` |

## Objective

Implement a bounded raw-body webhook endpoint that validates signature and replay controls before trusting payload, deduplicates provider events, verifies amount/currency/references/state, and settles Payment/Order/Inventory exactly once or records safe review state.

## Read First And Prerequisites

- Approved Phase 0/1 callback/security/audit/config contracts.
- Packet 01 states/idempotency, Packet 05 reservations, Packet 07 Orders, Packet 08 Payment/mock provider.
- Phase 2 roadmap sections 6-13 and 15-18.
- Approved single-database/transaction boundary and provider mock signature contract.

## In Scope

- PaymentWebhookEvent entity/status/mapping/migration.
- `POST /api/v1/payments/webhooks/{provider}` with provider allowlist, request-size/content limits, raw bytes capture, signature/timestamp/replay validation, safe parsing.
- Unique Provider+ProviderEventId, payload/signature hashes/safe metadata.
- Amount/currency/provider-payment/order-reference/state validation.
- Idempotent success/failure settlement orchestration across Payment, Order status/history, StockReservation confirm/release policy, audit, and domain/outbox intent.
- Duplicate, delayed, unknown, expired-reservation, transient DB, and requires-review behavior.

## Out Of Scope

- Real provider integration/secret, card data, refunds/chargebacks, shipment, external queue, manual review UI, automatic financial correction, or trusting browser return URLs as payment confirmation.

## Likely File Areas To Inspect

- Core Payments webhook/settlement commands/results/status policies and module interfaces.
- Infrastructure mock signature verifier/parser, PaymentWebhookEvent mapping/repository/transaction/DI/migration.
- API raw-body webhook endpoint/provider routing/options.
- Payment/Order/Inventory/audit integration and security/replay/concurrency/failure tests.

## Architecture And Prohibited Dependencies

- Provider cryptography/parsing belongs Infrastructure; Core receives a verified normalized event, never raw HTTP/provider SDK types.
- API must not deserialize/trust event fields before signature verification except minimal bounded provider routing metadata.
- One approved application settlement orchestrator owns cross-module transaction; Payment does not directly bypass Order/Inventory rules.
- Browser success/redirect is informational and cannot mark Paid.

## Validation And Settlement Order

1. Allowlist provider route and load secret reference safely.
2. Enforce content type/body size and capture exact raw bytes transiently.
3. Verify signature constant-time and timestamp/replay window before trusting payload.
4. Parse normalized allowlisted schema and validate required fields.
5. Start approved transaction; insert/claim unique provider event or return duplicate success.
6. Load Payment/Order/Reservation; validate provider payment ID, order reference, amount, currency, compatible states.
7. Apply exactly-once transition: Payment Succeeded/Failed, Order Paid/PaymentFailed or RequiresReview, reservation Confirm/Release only when safe, status history/audit/event intent.
8. Commit and return provider-compatible response. Transient local failure returns retryable response without partial effects.

## API Contract

`POST /api/v1/payments/webhooks/{provider}`

- Auth: verified provider signature, not customer bearer auth.
- Idempotency: Provider+ProviderEventId; valid duplicate returns provider-compatible success with no side effects.
- Invalid signature: reject without business mutation; persist/log only approved safe hash/metadata if abuse controls permit.
- Transient DB failure: retryable provider response.
- Valid but incompatible/unknown/expired: durable `RequiresReview`/safe event status per approved contract, no fulfillment claim.

## Database Changes And Migrations

Create only PaymentWebhookEvent fields/unique indexes/safe hashes/status/failure metadata/correlation and relationships. No raw body, card fields, provider secret, or unbounded failure payload.

## Security, Logging, And Failure Rules

- Never log/store raw callback body, signature header, secret, card data, auth headers/cookies, full order/customer data.
- Bound body, provider name, event ID, timestamps, failure reason.
- Invalid signature cannot create/update Payment/Order/Inventory.
- Duplicate/concurrent callbacks produce one settlement.
- Delayed success after reservation expiry becomes RequiresReview unless an approved safe fulfillment policy exists; never pretend payment did not occur.
- If stock confirmation fails after verified payment, retain payment evidence/review state and prohibit double charge/retry side effects.

## Implementation Steps

1. Obtain pre-code approval for mock signature format, timestamp window, response codes, event storage, delayed/expired/review behavior, and `P2-SETTLE-001` transaction owner.
2. Define normalized verified-event and settlement contracts/status rules in Core.
3. Add PaymentWebhookEvent mapping/constraints/indexes and reviewed migration.
4. Implement bounded raw-body capture and Infrastructure signature/timestamp/provider verification.
5. Implement unique event claim/deduplication before settlement.
6. Implement cross-module settlement transaction with amount/currency/reference/state checks and exact transitions.
7. Add safe invalid/duplicate/delayed/unknown/transient/requires-review responses/audit/logging.
8. Add domain/outbox event intents; Packet 10 makes publication durable before final Phase 2 approval.
9. Add unit, raw-signature, replay, duplicate/concurrency, tampering, delayed, expired, transaction rollback, redaction tests.
10. Run checks/failure injection, create evidence, obtain post-test approval.

## Test Matrix

| Test | Required Cases |
| --- | --- |
| Signature | Valid raw bytes; body changed; wrong/missing signature; wrong secret; constant-time path as practical. |
| Replay | Fresh/stale timestamp; duplicate event ID; concurrent duplicate one effect. |
| Validation | Amount/currency/payment/order/provider mismatch; unknown refs; malformed/oversized body. |
| Success | Compatible payment/order/reservation transitions/history/audit/event intent once. |
| Failure | Provider failed event releases per policy; duplicate safe. |
| Delayed/expired | RequiresReview/no unsafe stock confirmation or false fulfillment. |
| Transaction | Inject failure at each step; no partial local side effects; retry converges. |
| Redaction | No raw body/signature/secret/card/full customer/order data in DB/logs/evidence. |

## Verification Commands

```powershell
dotnet build ECommerceSolution.slnx
dotnet test ECommerceSolution.slnx --no-build
rg -n "Request.Body|Signature|Webhook|ProviderEventId|Payload|RequiresReview|Amount|Currency" ECommerce.API ECommerce.Core ECommerce.Infrastructure ECommerce.Tests
git diff --check
git diff --stat
git status --short
```

Inspect matches for verification-before-parse and sensitive persistence/logging.

## Acceptance Criteria

- Signature/timestamp/provider verification precedes trust/business mutation.
- Provider event uniqueness and concurrency produce exactly one settlement.
- Amount/currency/references/state are verified server-side.
- Settlement transaction/retry/review behavior prevents partial/double effects.
- Delayed/expired/stock-failure cases preserve verified payment evidence safely.
- No raw callback/card/secret storage/logging, real provider, refund, or later feature.
- Pre-code and post-test payment/security/commerce/database/operations approvals/evidence exist.

## Security Stop Conditions

Stop on deserialization/trust before signature, raw payload/signature/secret storage/logging, missing body/replay bounds, duplicate side effects, browser confirmation, unverified amount/currency/reference, partial settlement, expired-reservation auto-fulfillment, lost verified payment, real provider, or unreviewed migration.

## Rollback / Forward-Fix And Handoff

Preserve webhook/payment/order/inventory history after test/shared use; do not delete events to rerun. Before isolated use, reviewed migration rollback is possible. Prefer forward-fix/reconciliation for settlement defects. Handoff settlement events/transaction hooks to Outbox Packet 10 and recovery Packet 11.

## Copy-Ready Coding Prompt

```text
Execute Phase 2 Packet 09 from docs/implementation-guides/phase-2/09-payment-webhook-validation-and-settlement.md only after PHASE0/PHASE1 gates, Packets 05/07/08, P2-SETTLE-001, and pre-code payment/security/commerce/database/operations approval. Implement only bounded raw-body mock webhook verification-before-parse, PaymentWebhookEvent migration, replay/deduplication, amount/currency/reference checks, idempotent cross-module settlement/review behavior and tests. Add evidence/09-completion.md, run failure/replay checks, then obtain post-test approval. Stop on raw secrets/payloads, trust-before-signature, duplicate/partial effects, unsafe expired handling, or migration concerns.
```

