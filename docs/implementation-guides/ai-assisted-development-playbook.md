# AI-Assisted Development Playbook

## Purpose

This playbook explains how to use a basic AI coding tool safely after the final design is approved.

## Golden Rule

Give the AI one small vertical slice at a time. Never ask it to implement a whole phase in one prompt.

## Safe Prompt Shape

Use this structure:

```text
Implement module: <module name>

Read first:
- <roadmap phase document>
- <feature template or API contract>

Scope:
- <exact included work>

Out of scope:
- <exact excluded work>

Architecture rules:
- Target .NET 10, ASP.NET Core on .NET 10, EF Core compatible with .NET 10, C#, Onion Architecture, and the modular monolith.
- Keep the task local/free-first unless a human-approved document explicitly says otherwise.
- Keep Core independent from Infrastructure.
- Put provider/database code in Infrastructure.
- Use the approved `/api/v1` route style, Problem Details errors, correlation IDs, and idempotency rules where required.
- Add validation, authorization, ownership checks, logging/audit behavior, and tests.
- Do not change unrelated files.
- Do not provision cloud services, add deployment code, or introduce paid providers unless the task explicitly requests it.

Acceptance criteria:
- <pass/fail criteria>
```

## Review Checklist For AI Output

| Check | Question |
| --- | --- |
| Architecture | Did Core stay free of Infrastructure/API/Web dependencies? |
| Scope | Did the AI change unrelated files? |
| Security | Are authorization and ownership checks present? |
| Validation | Are invalid inputs rejected? |
| Idempotency | Is duplicate-submit behavior safe where needed? |
| Tests | Are unit/integration/security tests added or updated? |
| Data | Are migrations intentional and reviewed? |
| Logs | Are secrets, tokens, payment data, and unnecessary PII excluded? |

## High-Risk Modules Requiring Manual Review

- Authentication and authorization.
- Admin permissions.
- Inventory reservation.
- Checkout.
- Payment callback/webhook handling.
- Refunds.
- File uploads.
- AI/RAG retrieval and answer generation.
- Migrations touching order, payment, or inventory tables.

## Good AI Tasks

- Add a DTO and validation tests.
- Implement a simple read-only API.
- Add a page using an existing service.
- Add unit tests for a clear business rule.
- Update documentation after a human-approved decision.

## Bad AI Tasks

- "Build the whole checkout system."
- "Make the app production ready."
- "Add AI everywhere."
- "Refactor the architecture."
- "Fix all security issues."

## Stop Conditions

Stop and ask for human review if:

- The AI wants to change project references.
- The AI wants to store payment card data.
- The AI removes authorization checks.
- The AI changes the approved API error format away from Problem Details without an architecture decision.
- The AI introduces microservices, cloud providers, or paid services as requirements without approval.
- The AI changes inventory and payment in the same patch.
- The AI creates a migration with unexpected destructive changes.
- The AI adds cloud credentials, API keys, or secrets to source files.
