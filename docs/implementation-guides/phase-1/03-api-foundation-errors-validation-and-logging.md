# Packet 03: API Foundation, Errors, Validation, And Logging

## Packet Metadata

| Field | Value |
| --- | --- |
| Status | Blocked - PHASE0-GATE |
| Depends On | Packet 01 Approved; `API-001` Phase 0 contract |
| Layer Ownership | Core error/result contracts; API middleware/filters; Infrastructure only for sinks/adapters |
| Produces | Canonical request pipeline, tests, `evidence/03-completion.md` |
| Human Review | API owner, solution architect, security reviewer |

## Objective

Implement the cross-cutting API foundation: framework-neutral results/exceptions, Problem Details mapping, validation strategy, correlation IDs, privacy-safe structured request logging, and safe pipeline ordering before auth endpoints are exposed.

## Read First And Prerequisites

- Approved Phase 0 API/security/operations contracts.
- Packet 01 evidence/conventions.
- Phase 1 roadmap sections 7, 14-18.
- Existing API `Program.cs`, middleware, sample controllers, appsettings, and tests.

## In Scope

- Application result/error taxonomy.
- Central exception handling and Problem Details responses.
- Request/business/database validation responsibilities.
- Correlation ID validation/generation/response/log propagation.
- Structured request completion logs and redaction rules.
- API pipeline order and test host support needed for this behavior.

## Out Of Scope

- Authentication/authorization, identity entities, token service, business endpoints, global production rate limiting, tracing backend, cloud logging, or later-phase modules.

## Likely File Areas To Inspect

- Core common results/errors/exceptions/validation abstractions.
- `ECommerce.API/Middleware/`, filters, `Program.cs`, API configuration.
- API/integration tests and test host fixtures.
- Existing placeholder `ExceptionHandlingMiddleware.cs`.

## Architecture And Prohibited Dependencies

- Core errors/results cannot depend on ASP.NET Core `ProblemDetails`, HTTP status codes, logging providers, or middleware.
- API maps Core/application outcomes to HTTP.
- Controllers remain thin; validation/business behavior belongs in reusable application services/pipeline.
- Infrastructure does not own HTTP error mapping.
- Do not add a validation package unless built-in/current patterns are insufficient and version/security review is approved.

## Data, API, And UI Contracts

| Contract | Requirement |
| --- | --- |
| Base/errors | `/api/v1`, camelCase, canonical Problem Details fields and safe details. |
| Validation | Field errors for request shape; business errors have stable codes; DB constraints remain defense in depth. |
| Correlation | Accept only bounded/safe `X-Correlation-Id`, generate if absent/invalid, return and log it. |
| Status mapping | 400 validation, 401 auth, 403 forbidden, 404 privacy-safe not found, 409 duplicate/concurrency, 422 business, 429 rate, 500 generic. |
| Logging | Route template, method, status, duration, correlation, authenticated user ID later; no sensitive body/header capture. |

No UI change. Web may later reuse safe error concepts without API-specific dependencies in Core.

## Security, Logging, And Failure Rules

- Never expose stack traces, SQL, connection strings, config values, package paths, secrets, tokens, cookies, authorization headers, or full personal data.
- Sanitize/length-limit client correlation IDs; never treat them as authentication or idempotency.
- Unexpected exceptions return generic 500 with trace/correlation identifier and are logged once at the boundary.
- Logging failure must not expose data or recursively crash error handling.
- Validation must reject overlong/bounded inputs before expensive work.

## Implementation Steps

1. Verify `API-001` and inspect current middleware/pipeline/tests.
2. Define framework-neutral result/error categories and stable error codes in Core only if approved contracts require them.
3. Define validation conventions and how request/business/database layers cooperate.
4. Replace/complete placeholder exception handling with central Problem Details mapping.
5. Add correlation middleware/handler with safe validation, generation, response header, and logging scope.
6. Add structured request completion logging with route template and redaction; do not log bodies by default.
7. Order pipeline correctly: error/correlation/logging before endpoint execution; authentication/authorization slots documented for later packets.
8. Add a deterministic test-only endpoint or test hook only if needed to verify mappings; do not expose it outside test/development behavior.
9. Add unit/API tests for every mapping, correlation case, and redaction rule.
10. Run build/tests/scope review and write evidence.

## Database Changes And Migrations

None.

## API Contract

This packet establishes cross-cutting behavior, not business routes. Test these representative outcomes: validation, conflict, business rule, not found, unexpected error, valid/invalid/missing correlation ID.

## Test Matrix

| Test | Required Cases |
| --- | --- |
| Problem Details | Each known category maps correctly; unknown is safe 500. |
| Validation | Multiple field errors; bounded input; no sensitive value echoed. |
| Correlation | Valid supplied; missing generated; invalid/overlong replaced; response/log match. |
| Logging | Route/status/duration present; body/auth/cookie/token absent. |
| Pipeline | Exception handled once; response content type/status stable. |
| Architecture | Core has no HTTP/ProblemDetails/middleware dependency. |

## Verification Commands

```powershell
dotnet build ECommerceSolution.slnx
dotnet test ECommerceSolution.slnx --no-build
rg -n "Request.Body|Authorization|Cookie|accessToken|refreshToken|password" ECommerce.API ECommerce.Core ECommerce.Infrastructure
git diff --check
git diff --stat
git status --short
```

Inspect search matches in context; names may appear in explicit redaction code/tests.

## Acceptance Criteria

- Canonical Problem Details behavior is tested.
- Correlation IDs are safe, consistent, and non-authoritative.
- Structured logs exclude prohibited data.
- Core remains HTTP/framework-neutral.
- No endpoint/auth/database/later-phase behavior was introduced.
- Build/tests pass and evidence/human approval exist.

## Security Stop Conditions

Stop if error responses leak internals, logging captures bodies/headers/secrets, Core needs ASP.NET dependencies, package compatibility is unclear, or pipeline order could bypass later auth/authorization.

## Rollback And Handoff

Revert only Packet 03 changes. Hand stable errors, validation, correlation, logging, and pipeline slots to Packets 04-11.

## Copy-Ready Coding Prompt

```text
Execute Phase 1 Packet 03 from docs/implementation-guides/phase-1/03-api-foundation-errors-validation-and-logging.md only after PHASE0-GATE and Packet 01 approval. Inspect existing middleware and tests. Implement only framework-neutral Core outcomes plus API Problem Details, validation conventions, safe correlation IDs, structured redacted logging, and tests. Add evidence/03-completion.md, run all checks, and stop on data leakage, HTTP-in-Core, package, or pipeline-order concerns. Do not add auth endpoints or database features.
```

