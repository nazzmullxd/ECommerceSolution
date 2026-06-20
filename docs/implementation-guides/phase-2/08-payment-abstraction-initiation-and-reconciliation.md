# Packet 08: Payment Abstraction, Initiation, And Reconciliation

## Packet Metadata

| Field | Value |
| --- | --- |
| Status | Blocked - PHASE0/PHASE1-GATE |
| Depends On | Packets 01 and 07 Approved; decision `P2-PAY-001` |
| Layer Ownership | Payments own Payment lifecycle; Core gateway contract; Infrastructure deterministic mock adapter |
| Risk Review | **Pre-code and post-test payment-security, commerce, and database approval required** |
| Produces | Payment model/mock initiation/reconciliation/API/tests, `evidence/08-completion.md` |

## Objective

Implement a provider-neutral Payment lifecycle and deterministic local hosted-payment mock for idempotent initiation and status reconciliation, with order-owned trusted amount/currency and zero card-data handling.

## Read First And Prerequisites

- Approved Phase 0/1 secrets/API/audit/security contracts.
- Packet 01 payment status/idempotency rules and Packet 07 pending Order/Money/ownership.
- Phase 2 roadmap sections 3-5, 6-7, 9-10, 12-15, and 18.
- Official provider guidance only when a future real provider is approved; no real integration now.

## In Scope

- Payment entity/status/version/idempotency/provider reference/mappings/migration.
- Core `IPaymentGateway`-style contract and payment application service.
- Deterministic Infrastructure mock/sandbox hosted-session adapter with fake redirect/reference/status query.
- `POST /api/v1/orders/{orderId}/payments` for own pending order.
- Server-derived amount/currency/order reference, initiation idempotency, retry/attempt policy, safe redirect host/expiry.
- Internal/manual-local reconciliation query for `Unknown`/`RequiresReview` without automatic refund/cancel.

## Out Of Scope

- Real provider SDK/credentials/network call, card form/data/token, webhook settlement (Packet 09), refunds, captures/voids beyond mock status, subscriptions, split payments, saved methods, production reconciliation job.

## Likely File Areas To Inspect

- Core Payments entities/value objects/gateway/service commands/results.
- Infrastructure Payments mappings/repository/mock adapter/options/DI/migration.
- API payment initiation endpoint/DTO/OpenAPI.
- Unit/integration/API/idempotency/security/provider-contract tests.

## Architecture And Prohibited Dependencies

- Core gateway contract has no provider SDK/HTTP-client concrete types.
- Infrastructure adapter translates provider/mock formats and validates allowed redirect base/config.
- Payment reads trusted Order amount/currency/status through application contract; client sends no amount/currency/payment status.
- API does not invoke provider adapter directly or access DbContext.
- Provider reference is not authorization.

## Payment And Idempotency Contract

| Area | Rule |
| --- | --- |
| Eligibility | Own Order in approved PendingPayment-compatible state. |
| Amount | Exact trusted Order total/currency; one currency; no client override. |
| Key | Required `Idempotency-Key`, scoped to customer/order/initiation; request fingerprint excludes secrets. |
| Replay | Same key/same request returns same Payment/hosted session if valid; different fingerprint 409. |
| Attempts | Retry/second attempt policy approved; never create duplicate simultaneous charge intent silently. |
| Provider | Fake deterministic ID/status/redirect/expiry; no card field. |
| Reconciliation | Query by safe provider reference; compare amount/currency/status; unresolved mismatch -> RequiresReview. |

## API Contract

`POST /api/v1/orders/{orderId}/payments`

- Auth: Customer owner; future admin/system reconciliation uses separate internal permission.
- Header: required bounded `Idempotency-Key`.
- Request: no amount, currency, card, provider status, success URL, or arbitrary redirect host unless approved allowlisted option.
- Success: Payment ID, Order ID, provider name, allowed hosted redirect URL, expiry, status.
- Errors: 401/ownership-safe 404, 409 key/state/concurrency, 422 incompatible order, 503 safe mock/provider unavailable category.

## Database Changes And Migrations

Create only Payment schema/relationships/unique provider reference when present/idempotency/order-status indexes/version/timestamps/failure code bounds. No card/payment-method columns and no raw provider response payload.

## Security, Logging, And Failure Rules

- Never store/log PAN, CVV, expiry, payment credential/token, provider secret, raw request/response, authorization/cookie, or full customer/order details.
- Validate/allowlist redirect origin/path; do not create open redirect/SSRF input.
- Provider timeout/ambiguous result becomes `Unknown`/`RequiresReview`; do not retry charge blindly.
- Persistence failure after mock session creation uses deterministic/idempotent provider lookup on retry; no duplicate intent.
- Reconciliation is evidence/repair support, not automatic refund/cancel/fulfillment.

## Implementation Steps

1. Obtain pre-code approval for `P2-PAY-001`, attempt policy, status mapping, redirect, timeout/retry/idempotency/reconciliation behavior.
2. Define framework-neutral Payment/gateway/service contracts and invariants.
3. Add Payment mapping/constraints/indexes and reviewed migration.
4. Implement deterministic mock hosted adapter and contract tests without card inputs.
5. Implement own-order eligibility, trusted amount/currency, durable idempotent initiation, and persistence/provider ambiguity handling.
6. Implement thin initiation endpoint/DTO/Problem Details/OpenAPI/audit/logging.
7. Implement internal/manual-local reconciliation query/command for Unknown/RequiresReview with no refund/cancel side effect.
8. Add unit, migration, API, ownership, idempotency, duplicate/concurrency, redirect, provider-failure, redaction tests.
9. Run checks, create evidence, obtain post-test payment-security approval.

## Test Matrix

| Test | Required Cases |
| --- | --- |
| Eligibility | Own pending order; wrong owner/status; missing order. |
| Amount | Exact order amount/currency; client cannot override; mismatch requires review. |
| Idempotency | Same/same replay; same/different conflict; concurrent one provider intent/local record. |
| Adapter | Deterministic success/failure/timeout/unknown/query; allowed redirect only; no card fields. |
| Attempts | Approved retry policy; no accidental duplicate active payment. |
| Reconcile | Matching, mismatch, unknown reference/status; no automatic financial side effect. |
| Migration/logging | Constraints/indexes; no sensitive fields/payload/log values. |

## Verification Commands

```powershell
dotnet build ECommerceSolution.slnx
dotnet test ECommerceSolution.slnx --no-build
rg -n "Card|PAN|CVV|Expiry|PaymentToken|ProviderSecret|Redirect|Amount|Currency|Idempotency" ECommerce.API ECommerce.Core ECommerce.Infrastructure ECommerce.Tests
git diff --check
git diff --stat
git status --short
```

Inspect matches; card terms should appear only in prohibitions/tests, never model/API.

## Acceptance Criteria

- Payment initiation derives trusted Order amount/currency and enforces ownership/state/idempotency.
- Deterministic mock hosted adapter is provider-neutral at Core boundary and stores no card data.
- Duplicate/concurrent/ambiguous provider behavior cannot create duplicate intent silently.
- Redirect/config/logging are safe; migration is scoped/reviewed.
- No real provider, webhook settlement, refund/capture, saved method, or later feature was added.
- Pre-code and post-test payment/security/commerce/database approvals/evidence exist.

## Security Stop Conditions

Stop on any card/payment credential field, real provider secret/network integration, client amount/currency/status/redirect, open redirect, duplicate active charge intent, blind retry after ambiguity, provider SDK in Core, raw payload/logging, BOLA, or unreviewed migration.

## Rollback / Forward-Fix And Handoff

Use only fake mock sessions; invalidate/clear them on rollback. Before shared data, reviewed migration rollback may be possible; after payment records exist, preserve history and forward-fix. Hand provider/payment/idempotency/status/reconciliation contracts to Webhook Packet 09.

## Copy-Ready Coding Prompt

```text
Execute Phase 2 Packet 08 from docs/implementation-guides/phase-2/08-payment-abstraction-initiation-and-reconciliation.md only after PHASE0/PHASE1 gates, Packets 01/07, P2-PAY-001, and pre-code payment/security/commerce/database approval. Implement only provider-neutral Payment, scoped migration, deterministic no-card hosted mock, owned server-amount idempotent initiation API, safe reconciliation, and tests. Add evidence/08-completion.md, run checks, then obtain post-test approval. Stop on card data, real provider/secret, client amount/redirect/status, duplicate/blind retry, raw payload, BOLA, SDK-in-Core, or migration concerns.
```

