# Packet 01: Commerce Contracts, States, And Idempotency

## Packet Metadata

| Field | Value |
| --- | --- |
| Status | Blocked - PHASE0/PHASE1-GATE |
| Depends On | Approved Phase 0/1 contracts and evidence |
| Layer Ownership | Core framework-neutral commerce primitives/contracts/tests |
| Risk Review | Commerce architect before coding; security reviewer for idempotency/errors |
| Produces | Money/status/error/event/idempotency baseline, `evidence/01-completion.md` |

## Objective

Define and test the framework-neutral commerce language used by every later packet: money/quantity/time rules, status machines, error/results, commands/events, transaction boundaries, and idempotency ownership.

## Read First And Prerequisites

- Approved Phase 0 contracts/ADRs and Phase 1 acceptance/evidence.
- Phase 2 roadmap sections 2-5, 7-13, and 18.
- Phase 1 Core conventions, Problem Details mapping, clock/current-user/audit contracts.
- This package README and current Core/tests.

## In Scope

- Money/currency/rounding/precision invariants.
- Positive quantity and UTC/time/expiry concepts.
- Cart, checkout, reservation, order, payment, webhook, and outbox statuses plus legal transitions.
- Stable domain/application error codes and result categories compatible with Phase 1.
- Command/query/event catalog and versioned integration-event naming.
- Idempotency key/request fingerprint/replay/conflict/retention contracts and owner matrix.
- Transaction/consistency boundary decision records used by later packets.

## Out Of Scope

- EF entities/mappings/migrations, repositories, APIs, workers, provider adapters, pricing implementation, or state-changing services.

## Likely File Areas To Inspect

- Core common value objects/results/errors/events and Phase 1 conventions.
- New commerce module folders/namespaces only after approved module layout inspection.
- Unit/architecture tests.

## Architecture And Prohibited Dependencies

- Core contracts use no EF, HTTP, JSON provider, payment SDK, background-service, or Infrastructure types.
- Do not create one giant commerce service or shared mutable base entity.
- Module status types are owned by their module; cross-module translation is explicit.
- Integration events are facts, not commands, and contain minimal classified data.

## Data And API Contracts

| Concept | Required Rule |
| --- | --- |
| Money | Decimal amount plus ISO currency; same currency for arithmetic; explicit rounding at business boundary. |
| Quantity | Positive integer for cart/reservation/order line; bounded per approved policy. |
| Time | UTC via testable clock; expiry comparisons deterministic. |
| Errors | Stable codes map through Phase 1 Problem Details without HTTP in Core. |
| Idempotency | Bounded key, owner/scope, request hash, retained result reference, same/different fingerprint behavior. |
| Events | `<Domain>.<Entity>.<PastTenseAction>.v<Version>` for outbox/integration events. |

No endpoint in this packet; define contracts future endpoints consume.

## State And Transaction Contracts

- Document every legal/illegal transition for Cart, CheckoutSession, StockReservation, Order, Payment, PaymentWebhookEvent, and OutboxMessage.
- Identify strong transaction boundaries: inventory reserve/release/confirm; checkout+pending order where approved; payment settlement+order+inventory+outbox within one database boundary where possible.
- Identify eventual behavior: email/notification delivery and future external publishing.
- Define which retries are safe and which require reconciliation/manual review.

## Security, Logging, And Failure Rules

- Money/status/idempotency values from clients are untrusted.
- Fingerprints exclude secrets and store hashes, not full sensitive requests.
- Errors/events/log metadata use IDs and safe categories, not address/payment payloads.
- Illegal transitions fail without mutation.
- Duplicate/retry behavior is deterministic and tested.

## Implementation Steps

1. Verify foundation gates and read accepted contracts.
2. Approve `P2-MONEY-001` and `P2-IDEMP-001`.
3. Define Money, Currency, Quantity, clock/expiry semantics and unit tests.
4. Define module status values and transition policies with exhaustive tests.
5. Define commerce error codes/results compatible with Phase 1 mapping.
6. Define command/query/event catalog and event metadata/versioning.
7. Define idempotency contract and ownership/replay/fingerprint/retention matrix.
8. Define transaction/eventual-consistency boundaries and retry classifications.
9. Add architecture tests preventing outer dependencies in Core contracts.
10. Run build/tests/scope checks, create evidence, request review.

## Database Changes And Migrations

None.

## API Contract

No route. Produce error/idempotency/status contracts for Packets 02-11.

## Test Matrix

| Test | Required Cases |
| --- | --- |
| Money | Same currency operations; mismatch rejected; rounding/zero/negative rules. |
| Quantity | Valid bounds; zero/negative/overflow rejected. |
| States | Every legal transition succeeds; every illegal transition fails without mutation. |
| Idempotency | Same key/same hash replay; same key/different hash conflict; scope isolation; expiry. |
| Events | Name/version/required metadata; sensitive fields absent. |
| Architecture | Core contracts contain no forbidden framework/provider dependency. |

## Verification Commands

```powershell
dotnet build ECommerceSolution.slnx
dotnet test ECommerceSolution.slnx --no-build
rg -n "Microsoft.EntityFrameworkCore|Microsoft.AspNetCore|HttpContext|Payment.*Sdk" ECommerce.Core
git diff --check
git diff --stat
git status --short
```

## Acceptance Criteria

- Money, quantity, status, error, event, idempotency, and transaction contracts are approved/tested.
- Illegal transitions and duplicate/different-payload behavior are deterministic.
- Core remains framework/provider neutral.
- No persistence/API/provider/worker/later-phase behavior was added.
- Evidence and pre/post commerce/security approval exist.

## Security Stop Conditions

Stop on missing foundation gate, client-controlled trusted amount/status, ambiguous idempotency owner, sensitive fingerprint/event payload, currency ambiguity, HTTP/EF/provider dependency in Core, or unapproved transaction boundary.

## Rollback / Forward-Fix And Handoff

Before downstream use, revert this packet if rejected. After dependent packets consume contracts, prefer a versioned forward fix and update cross-phase contracts rather than silently changing statuses/events. Handoff approved primitives to Packets 02-11.

## Copy-Ready Coding Prompt

```text
Execute Phase 2 Packet 01 from docs/implementation-guides/phase-2/01-commerce-contracts-states-and-idempotency.md only after PHASE0/PHASE1 gates and decisions P2-MONEY-001/P2-IDEMP-001. Implement only framework-neutral Core money/quantity/status/error/event/idempotency/transaction contracts and exhaustive unit/architecture tests. Add evidence/01-completion.md, run checks, and stop on client-trusted values, ambiguous ownership/currency/transactions, sensitive payloads, or outer dependencies. Do not add persistence, APIs, workers, or providers.
```

