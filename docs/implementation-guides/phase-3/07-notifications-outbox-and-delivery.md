# Packet 07: Notifications, Outbox, And Delivery

## Packet Metadata

| Field | Value |
| --- | --- |
| Status | Blocked - PHASE0/PHASE1/PHASE2-GATE |
| Depends On | Packet 01 Approved; Phase 2 Outbox Approved; Packets 05-06 event contracts as applicable; `P3-NOTIFY-001` |
| Layer Ownership | Notifications owns recipient records and delivery attempts; consumes approved events |
| Risk Review | Pre-code and post-test privacy, event, operations, template, and UX review |
| Produces | In-app notifications and mock-email boundary, `evidence/07-completion.md` |

## Objective

Add reliable in-app customer/staff notifications and a mock email delivery boundary without coupling business transactions to delivery success or duplicating Phase 2 Outbox behavior.

## Read First And Prerequisites

- Approved Phase 1 identity/privacy and Phase 2 Outbox/retry/deduplication/dead-letter contracts.
- Packet 01 and approved producer contracts from Reviews/Support/Orders where implemented.
- Resolved `P3-NOTIFY-001` type/template/deduplication/retention/retry decisions.

## In Scope

- Notification recipient ownership, type, bounded title/body or template data, source reference, read/unread state, delivery state, timestamps, expiry/retention, and deduplication.
- Event consumers for explicitly approved Order/support/review outcomes; in-app delivery and deterministic local mock-email adapter.
- Caller-owned list/read-state APIs and Web notification experience.
- Retry and terminal-failure evidence integrated with the existing Outbox worker boundary.

## Out Of Scope

- Real SMTP/provider, push/SMS, marketing subscriptions, guest notifications, price-drop/restock automation, free-form administrator broadcast, AWS SQS/EventBridge/SES provisioning, or business-state mutation on delivery failure.

## Ownership And Read-Only Contracts

- **Owns:** Notification, recipient, rendered safe content/template version, source type/ID, read state, delivery channel/status/attempt metadata, dedupe key, retention.
- **Reads/consumes:** versioned approved event facts and recipient-safe identity/contact projection when a channel needs it.
- **Must not own/mutate:** Order, Review, SupportTicket, OutboxMessage, User, Payment, or the producer's transaction state.

## API Contract

| Route | Auth | Behavior |
| --- | --- | --- |
| `GET /api/v1/me/notifications` | Authenticated recipient | Paginated caller-owned notifications; allowlisted unread/type/date filters. |
| `PUT /api/v1/me/notifications/{notificationId}/read-state` | Recipient owner | Sets explicit read/unread state idempotently. |
| `POST /api/v1/me/notifications/read-all` | Recipient owner | Bounded/idempotent mark-read operation if approved. |

Delivery-worker and mock-email interfaces are internal, not public admin endpoints. Use source IDs safe for recipient navigation; never reveal internal aggregate data in URLs/content.

## UI Acceptance States

- Header indicator/list: loading, empty, unread count, paginated success, mark-read pending/success/conflict, session expiry, concealed cross-owner ID, and dependency failure.
- Notification links navigate only to resources the recipient can independently authorize; stale/deleted targets show a neutral unavailable state.
- Mock email is observable to developers through test evidence/local sink, not rendered as production success to users.

## Likely File Areas To Inspect

- Core Notification entities/templates/consumer/delivery interfaces and existing Outbox contracts.
- Infrastructure mapping, event handlers, mock adapter, retry/deduplication, migration.
- API owner-scoped endpoints and Web layout/list components/views.
- Unit, integration, event replay, API/BOLA, UI/accessibility, privacy, and failure tests.

## Architecture And Prohibited Dependencies

- Producers commit business state plus their Outbox event; Notification consumes later. No producer calls email or rolls back because delivery fails.
- Reuse Phase 2 dispatcher/retry/dead-letter policy; do not create a second event bus/outbox.
- Templates receive minimal typed data and produce encoded content; no raw entity serialization or sensitive message/order body.

## Database Changes And Migrations

Add/reconcile Notification with recipient, type, safe source reference, template/version/payload or rendered fields, channel, read/delivery state, dedupe key, attempts/next attempt/last error code, created/read/expiry timestamps, and version. Index recipient/unread/date, delivery state/next attempt, expiry, and uniquely enforce dedupe key per recipient/type/source.

## Security, Logging, And Failure Rules

- Recipient comes from approved event/current user and is enforced in queries. Navigation performs fresh authorization.
- Allow only registered templates/types; encode content and minimize data. Internal support notes and sensitive Order/address/payment details are prohibited.
- Log event/notification/recipient IDs, template/type, attempt/status/error code, and correlation/causation IDs; never contact details, rendered bodies, tokens, cookies, or private source payloads.
- Duplicate events create one notification. Exhausted delivery becomes terminal/inspectable without changing producer truth; in-app creation failure is retried from Outbox.

## Implementation Steps

1. Approve types/producers, typed template fields, dedupe keys, retention, read-all bound, retry/dead-letter, mock behavior, exact routes/UI.
2. Define Notification lifecycle, templates, consumer/delivery interfaces, and event compatibility tests.
3. Add reviewed mapping/indexes/uniqueness and migration.
4. Implement idempotent consumers and reuse the Phase 2 Outbox worker/retry boundary.
5. Implement mock email adapter, owner APIs, and accessible in-app UI.
6. Add replay, failure, privacy, ownership, template, UI, accessibility, and migration tests.
7. Run checks, record evidence, and obtain post-test review.

## Test Matrix

| Test | Required Cases |
| --- | --- |
| Events | One notification per dedupe key; replay/duplicate/out-of-order supported facts; unknown version fails safely. |
| Ownership | Recipient-only list/read; guessed ID concealed; target reauthorization. |
| Delivery | Mock success/failure; retry/backoff/exhaustion; producer state unaffected. |
| Privacy | Approved typed fields only; internal note/order/payment/private data absent from content/logs. |
| UI/API | Empty/unread/pagination/read/conflict/session/stale-target/error and accessibility states. |
| Migration | Fresh apply; dedupe/index/expiry constraints. |

## Verification Commands

```powershell
dotnet build ECommerceSolution.slnx
dotnet test ECommerceSolution.slnx --no-build
rg -n "Notification|OutboxMessage|Dedupe|Template|DeliveryStatus|read-state" ECommerce.Core ECommerce.Infrastructure ECommerce.API ECommerce.Web ECommerce.Tests
git diff --check
git status --short
```

## Acceptance Criteria

- Notifications are recipient-owned, idempotent, template-controlled, privacy-safe, and delivered asynchronously through the existing Outbox boundary.
- Delivery failure/replay never changes Order/Review/Support truth or creates duplicates.
- Exact owner API/UI states, target reauthorization, mock delivery, retries, logs, and tests pass.
- No real provider/AWS resource, marketing, guest tracking, or future wishlist automation is added.
- Completion evidence and post-test review exist.

## Security Stop Conditions

Stop for recipient from client input, cross-owner query, raw entity/event serialization, free-form template HTML, internal note/private Order/payment data, navigation without fresh authorization, duplicate outbox, synchronous producer delivery, sensitive logs, or unbounded retry/read-all operation.

## Rollback / Forward-Fix And Handoff

Pause consumers and UI routes; retain Outbox and Notification records for replay/diagnosis. Forward-fix templates/consumers and replay by dedupe key rather than deleting business events. Hand notification states and approved recipient events to Packet 11 and production observability planning.

## Copy-Ready Coding Prompt

```text
Execute Phase 3 Packet 07 only after all prerequisite gates, Packet 01, approved Phase 2 Outbox, relevant producer event contracts, P3-NOTIFY-001, and pre-code review. Implement only typed/idempotent Notification consumers, recipient-owned in-app APIs/UI, mock email boundary, existing-outbox retries, migration, privacy/security/failure tests. Never add a second outbox, synchronous producer delivery, client recipient IDs, raw entity payloads, sensitive content, or real cloud/email services. Add evidence/07-completion.md, run checks, and obtain post-test review.
```
