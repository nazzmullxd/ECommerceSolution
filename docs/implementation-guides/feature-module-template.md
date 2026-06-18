# Feature Module Template

Use this template for every implementation module. A novice developer or AI coding tool should be able to follow it without making architecture decisions.

## Module Name

Example: `Cart - Add Item`

## Objective

State the business goal in one or two sentences.

Example: Allow a customer or guest session to add a product variant to a cart.

## User Story

As a `<user type>`, I want `<action>` so that `<business value>`.

## In Scope

- List exactly what will be built.
- Keep the list small.
- Include API, UI, data, tests, and docs if needed.

## Out Of Scope

- List tempting work that must not be included.
- This prevents AI tools from expanding the task.

## Entities And Data

| Entity | Change |
| --- | --- |
| EntityName | Add/update/read/delete details. |

## Interfaces

| Interface | Responsibility |
| --- | --- |
| `IExampleService` | Business operation exposed by Core. |
| `IExampleRepository` | Persistence contract owned by Core. |

## API Contract

| Method | Route | Auth | Request | Response |
| --- | --- | --- | --- | --- |
| POST | `/api/example` | Customer/Admin/Anonymous | `ExampleRequest` | `ExampleResponse` |

## UI Contract

| Page | Behavior |
| --- | --- |
| Page name | Main user actions, validation, empty state, success state, error state. |

## Security Rules

- Who is allowed to perform the action?
- Who is explicitly forbidden?
- What data must not be logged?
- What ownership check is required?
- What abuse case must be tested?

## Failure Rules

- What happens if validation fails?
- What happens if the same request is sent twice?
- What happens if the database operation fails?
- What happens if an external provider fails?

## Implementation Steps

1. Add or update Core entity/value object.
2. Add or update Core DTOs.
3. Add or update Core interface.
4. Add Infrastructure implementation.
5. Register service in dependency injection.
6. Add API endpoint or Web UI action.
7. Add validation and authorization.
8. Add logging/audit event if needed.
9. Add unit tests.
10. Add integration tests.
11. Add demo script.
12. Update documentation.

## Tests

| Test Type | Required Cases |
| --- | --- |
| Unit | Business rules and validation. |
| Integration | Database/API behavior. |
| Security | Unauthorized, forbidden, ownership violation. |
| Failure | Duplicate request, provider failure, invalid state. |

## Acceptance Criteria

- Clear pass/fail bullet.
- Clear pass/fail bullet.
- Clear pass/fail bullet.

## Demo Script

1. Start the app.
2. Perform user action.
3. Show expected result.
4. Show failed/forbidden case if applicable.

## AI Coding Prompt Guardrails

When using an AI coding tool, include this instruction:

```text
Implement only this module. Do not change unrelated files. Preserve Onion Architecture. Do not put Infrastructure dependencies in Core. Add or update tests for the behavior. Do not bypass authorization or validation. If a requirement is unclear, stop and ask.
```
