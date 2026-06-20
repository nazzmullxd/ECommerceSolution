# Packet 03: Observability, SLOs, Dashboards, And Retention

## Packet Metadata

| Field | Value |
| --- | --- |
| Status | Blocked - PHASE0/PHASE1/PHASE2/PHASE3/PHASE4-GATE |
| Depends On | Packets 01-02 Approved; `P5-SLO-001`, `P5-LOG-001` |
| Ownership | Operations owns telemetry platform contract; module owners own signal meaning/runbook |
| Manual Review | SRE/operations, security/privacy, audit, database, commerce/payment, AI, product, and cost review |
| Produces | Telemetry contract, SLI/SLO/dashboard/retention design, `evidence/03-completion.md` |

## Objective

Define and later verify privacy-safe structured logs, correlation/causation propagation, metrics, traces, audit review, retention, dashboards, SLIs/SLOs, and error-budget behavior for every critical workload and customer journey.

## Read First And Prerequisites

- Packet 01 workloads/critical journeys/owners and Packet 02 health signals.
- Approved Phase 1-4 logging/audit/correlation/outbox/provider privacy contracts.
- Business support hours, candidate availability/latency objectives, privacy jurisdiction, and local telemetry tool constraints.

## In Scope

- Canonical log event schema/redaction, correlation/trace/causation propagation across API/Web/worker/outbox/payment/indexing, exception classification and sampling.
- Bounded metrics and traces for request, database, checkout/inventory/payment/webhook, outbox/worker, auth/admin/upload, support, and enabled AI.
- Audit-log protection/review cadence separate from diagnostics.
- Per-environment verbosity/sampling/retention/access, SLI definitions, candidate SLO/error budget, dashboard purpose/audience/panels/freshness, local evidence and future CloudWatch/OpenTelemetry mapping.

## Out Of Scope

- Paid telemetry backend, CloudWatch/X-Ray resource, production alert rules owned by Packet 10, application business feature, raw payload tracing, production SLO approval without measurements, or replacing mandatory audit with logs/traces.

## Ownership And Operational Contracts

- **Owns:** telemetry field/event/metric/trace naming, units/labels, redaction, sampling, retention, SLI formula, dashboard contract, review workflow.
- **Reads:** approved operational/business outcomes and health/dependency states.
- **Must not own/mutate:** business state, AuditLog source, authorization, alert response, or risk acceptance.

Required shared fields include UTC timestamp, severity, event ID/name/version, service/workload/module, environment, release version, route/operation, status/outcome/failure category, duration, correlation/trace/span/causation IDs, and pseudonymous actor/aggregate IDs only when justified. Metric labels must be bounded; never use user/order/correlation/query/URL values as labels.

## Evidence Required By Tier

| Tier | Required Evidence |
| --- | --- |
| Design Evidence | Telemetry dictionary, privacy/redaction matrix, propagation diagram, SLI/SLO/error-budget candidates, dashboard/retention/access reviews. |
| Local Verification Evidence | Automated propagation/redaction/metric/trace tests, sample synthetic logs/traces, cardinality and failure tests. |
| Staging Evidence | Release telemetry reaches approved sink; dashboards/SLIs match known load/failures; sampling/retention/access verified. |
| Production Approval Evidence | Final SLOs/retention/access/dashboards and monitoring owner approved for exact release/workload. |

## API And UI Contract

- API accepts bounded validated `X-Correlation-Id` only under approved trust rule, generates one otherwise, and returns safe trace/correlation ID in Problem Details.
- Trace context propagation follows approved standard; correlation ID is never auth/idempotency.
- Operational dashboards are restricted and aggregate; no full PII, support text, payment payload, upload body/path, AI prompt/source, token/header/cookie, SQL parameter, internal secret/URL.

## Likely File Areas To Inspect

- API/Web middleware/pipeline, Core operation/result/event contracts, Infrastructure DB/outbox/provider/worker logging, AuditLog services, Tests logging/telemetry fixtures.
- Future CloudWatch/OpenTelemetry mappings remain documentation until separately approved.

## Architecture And Prohibited Dependencies

- Use framework logging/Activity/Meter abstractions and approved adapters; Core must not depend on logging backend/CloudWatch SDK.
- Logs, metrics, traces, and audit have distinct purposes/access/retention; one cannot silently substitute another.
- Instrumentation must not change transaction outcome, leak private data, or block request indefinitely. Mandatory AuditLog failure semantics remain module-owned.

## Database Changes And Migrations

No telemetry business table by default. Audit schema changes require Phase 1 owner/migration/privacy review. Do not store high-volume traces/logs in the transactional database without an ADR/capacity plan.

## Security, Logging, Audit, And Failure Rules

- Ban passwords, hashes, tokens, cookies, authorization headers, secrets/connection strings, full PII/address/support/review text, payment/webhook payload, upload bytes/path, AI full prompt/answer/source/vector, raw SQL parameters.
- Redact at source plus sink defense; test nested exception/scopes/serialization. Hashing IDs does not automatically make data non-personal.
- Telemetry sink failure degrades without losing business correctness; bounded buffering/drop policy and self-observability must be explicit.
- Audit access/export/retention is least privilege and audited; tampering/deletion protection is reviewed in Packet 11.

## Implementation Steps

1. Approve telemetry dictionary, prohibited fields, redaction/sampling/access/retention and propagation boundaries.
2. Define SLIs for availability, latency, correctness/freshness and critical business failures with numerator/denominator/exclusions/source/owner.
3. Propose SLOs/error budgets from roadmap candidates and business needs; label unmeasured targets proposed.
4. Define local framework-neutral instrumentation and tests for HTTP, DB, workers, providers, events and health.
5. Define dashboard views for executive/operations/security/commerce/payment/AI with owner/runbook/freshness.
6. Define staging calibration and production approval evidence; map future CloudWatch/OpenTelemetry without resources.
7. Run local redaction/propagation/cardinality/failure tests when executable and obtain reviews.

## Test And Review Matrix

| Test | Required Cases |
| --- | --- |
| Propagation | API-Web-worker-outbox/payment/indexing IDs, malformed/missing client ID, async causation. |
| Redaction | Every banned field nested in request/exception/provider/audit/AI/upload context. |
| Metrics | Correct units/buckets/counts, bounded labels, no IDs/high-cardinality attacker input. |
| Traces | Expected spans/status without payload/secret; sampling and provider failure. |
| SLI/SLO | Known fixtures validate formulas/exclusions; candidate vs approved clearly labelled. |
| Failure/access | Sink unavailable/buffer full; dashboard/audit unauthorized; retention cleanup safe. |

## Verification Commands

```powershell
dotnet build ECommerceSolution.slnx
dotnet test ECommerceSolution.slnx --no-build
rg -n "ILogger|ActivitySource|Meter|Correlation|TraceId|SpanId|AuditLog|Password|Authorization|Cookie|Prompt|Payload" ECommerce.API ECommerce.Web ECommerce.Core ECommerce.Infrastructure ECommerce.Tests
git diff --check
git status --short
```

## Acceptance Criteria

- Telemetry/SLI/dashboard fields, formulas, owners, privacy/access/retention and failure behavior are explicit and versioned.
- Correlation/trace/causation propagates across critical sync/async paths without becoming security/idempotency input.
- Redaction/cardinality/sink-failure tests pass and AuditLog remains protected/separate.
- Design/local evidence are approved; SLOs/staging/production claims remain pending until measured and signed.

## Manual Approval Gates

Manual approval is mandatory for SLOs/error budgets, log/trace/audit retention/access, sampling, privacy redaction, dashboard exposure, telemetry vendor/CloudWatch mapping, and production monitoring ownership.

## Stop Conditions

Stop for unknown signal owner/source, sensitive/high-cardinality telemetry, raw payload/SQL/prompt logging, correlation as auth/idempotency, audit replaced by logs, unbounded sink buffer, invented measurement, SLO marked approved without data, or staging/production evidence inferred from local design.

## Rollback / Forward-Fix And Handoff

Future instrumentation rollback disables defective enrichment/export while retaining minimal safe operational logs/audit. Do not suppress a critical signal solely to reduce noise; forward-fix schema/redaction/cardinality and rerun evidence. Hand SLIs/signals to Packets 05, 08-10, 12.

## Copy-Ready Future Execution Prompt

```text
Execute Phase 5 Packet 03 only after prior gates, Packets 01-02, P5-SLO-001/P5-LOG-001, and manual design review. Inspect existing logging/audit first, then implement or document only approved structured telemetry, correlation/trace/causation, bounded metrics/traces, redaction tests, SLI formulas, dashboard/retention design, and local evidence. Add no paid backend, CloudWatch resource, sensitive payload, high-cardinality labels, business behavior, or invented SLO result. Separate all four evidence tiers, leave staging/production pending, and add evidence/03-completion.md.
```
