# Packet 02: Health, Readiness, Liveness, And Degradation

## Packet Metadata

| Field | Value |
| --- | --- |
| Status | Blocked - PHASE0/PHASE1/PHASE2/PHASE3/PHASE4-GATE |
| Depends On | Packet 01 Approved; `P5-HEALTH-001` |
| Ownership | Each workload owns health semantics; operations owns routing contract |
| Manual Review | Architecture, SRE/operations, security, database, payment, Outbox, media, and AI owner review |
| Produces | Health/degradation contract and `evidence/02-completion.md` |

## Objective

Define and later verify startup, liveness, readiness, and diagnostic health behavior for Web, API, and workers so orchestrators can act safely without leaking internals, causing restart storms, or marking degraded critical commerce as healthy.

## Read First And Prerequisites

- Packet 01 workload/dependency/critical-journey inventory and owner matrix.
- Approved module failure/timeout/retry/idempotency contracts and Phase 4 disable/fallback behavior.
- Future ALB/ECS health requirements as design references only.

## In Scope

- Health endpoint contract, startup/config/migration compatibility check, liveness process/deadlock indicator, readiness critical dependency checks, worker lease/backlog state, and dependency degradation matrix.
- Separate public-safe aggregate status from restricted diagnostics; timeout/cache/parallelism/rate and no-side-effect rules.
- Traffic/drain/maintenance behavior, startup grace, readiness transitions, dependency recovery, local tests and staging routing test plan.

## Out Of Scope

- AWS ALB/ECS configuration, deployment code, auto-healing policy, production endpoint exposure, deep destructive probe, real payment call, business write, full provider diagnostics, or SLO/alert thresholds owned by Packets 03/10.

## Ownership And Operational Contracts

- **Owns:** per-workload startup/live/ready semantics, dependency criticality, aggregate status, degradation/recovery behavior.
- **Reads:** typed configuration and shallow dependency probes through approved adapters.
- **Must not own/mutate:** dependency state, business data, migration execution, traffic infrastructure, or incident decision.

Proposed routes, subject to approved conventions:

- `/health/live`: process responsive only; no database/provider dependency.
- `/health/ready`: required config plus critical local dependencies; no expensive/external side effects.
- `/health/startup` or startup state: initialization complete and schema compatibility checked without auto-migration.
- Restricted detailed diagnostics only for authorized/internal operational access; public response is minimal.

## Evidence Required By Tier

| Tier | Required Evidence |
| --- | --- |
| Design Evidence | Dependency criticality/degradation matrix, route/status contract, security and routing review. |
| Local Verification Evidence | Automated endpoint/startup/dependency failure/recovery tests and actual timings/log scan. |
| Staging Evidence | Orchestrator/load-balancer route/drain/restart/dependency outage simulation against release artifact. |
| Production Approval Evidence | Health paths/access/timeouts/grace/routing accepted for exact topology with monitoring owner. |

## API And UI Contract

- Public health response exposes status and optional generic component categories only, never connection strings, hostnames, versions exploitable by attackers, stack traces, queue/source/customer details, or secrets.
- Use consistent status mapping: liveness unhealthy only when process cannot serve; readiness unhealthy when critical traffic must stop; degraded/noncritical state follows approved status contract.
- Customer UI uses feature-specific safe degradation, not raw health output.

## Likely File Areas To Inspect

- API/Web Program pipelines, typed options/startup validation, Infrastructure DB/storage/outbox/provider adapters, worker services, Tests integration fixtures.
- No deployment/orchestrator files are created in this packet.

## Architecture And Prohibited Dependencies

- Health checks are read-only, bounded, cancellation-aware, and cannot execute migrations, publish messages, initiate payment, reserve stock, write files, or call AI generation.
- Liveness excludes downstream dependencies to avoid restart storms.
- Readiness includes only dependencies whose absence makes serving that workload unsafe; optional features degrade behind flags/fallback.
- One failing worker must not automatically make Web/API unready unless shared correctness requires it and owner approves.

## Database Changes And Migrations

No migration. Startup may compare expected schema/migration compatibility without applying changes. Any automatic migration attempt is a stop condition.

## Security, Logging, Audit, And Failure Rules

- Health endpoints have explicit network/auth/rate/cache policy and safe response. Detailed endpoint access is logged/audited when sensitive.
- Log state transition/component category/duration/correlation where meaningful, not every successful probe or sensitive diagnostic.
- Probe timeout/failure returns known status and does not throw detailed errors to client.
- Payment, external email, AI, and future AWS providers use configuration/circuit state or shallow safe checks, not live billable/business calls.

## Implementation Steps

1. Approve criticality and degradation matrix for every workload/dependency.
2. Define startup/live/ready/detailed route, status, timeout, caching, access and logging contracts.
3. Define no-side-effect shallow probes and typed failure categories.
4. Plan local implementation/automated tests for missing config, DB unavailable/slow, storage, outbox worker, optional providers, recovery and cancellation.
5. Define staging traffic/drain/restart/startup-grace/dependency simulation and evidence template.
6. Obtain design review; later implement only after packet becomes executable and record each evidence tier separately.

## Test And Review Matrix

| Test | Required Cases |
| --- | --- |
| Liveness | Healthy with downstream outage; unhealthy on process/deadlock simulation only. |
| Readiness | Config/schema/DB critical failures; optional media/email/AI degradation; recovery transition. |
| Workers | Outbox/index/cleanup lease/backlog/fatal loop behavior and workload-specific readiness. |
| Safety | No writes/migrations/provider charges; timeout/cancellation; bounded parallel probes. |
| Security | Public response redacted; detailed endpoint restricted; no secret/internal topology. |
| Staging plan | Drain/routing/startup grace/restart-storm/old-new-version compatibility scenarios. |

## Verification Commands

```powershell
dotnet build ECommerceSolution.slnx
dotnet test ECommerceSolution.slnx --no-build
rg -n "AddHealthChecks|MapHealthChecks|IHealthCheck|BackgroundService|Migrate\(|EnsureCreated|ConnectionString" ECommerce.API ECommerce.Web ECommerce.Infrastructure ECommerce.Tests
git diff --check
git status --short
```

## Acceptance Criteria

- Every workload/dependency has approved startup/live/ready/degraded semantics and owner.
- Probes are bounded/read-only/no-side-effect, liveness avoids restart storms, and readiness fails traffic safely.
- Public responses/logs expose no internals/secrets and optional failures use approved degradation.
- Design/local evidence pass; staging/production remain pending until executed and signed.

## Manual Approval Gates

Manual review is required before health code, detailed endpoint exposure, readiness dependency changes, orchestrator routing, or production threshold/grace configuration.

## Stop Conditions

Stop for missing owner/criticality decision, liveness tied to downstream, probe business write/migration/provider call, secret/internal response, unbounded probe, readiness masking unsafe checkout/payment/database state, absent recovery test, or staging/production marked complete from design.

## Rollback / Forward-Fix And Handoff

Future implementation rollback disables new probe exposure/routing and restores prior known-safe health behavior. Never disable correctness dependencies merely to make readiness green; forward-fix probes/config. Hand health signals to Packets 03, 05-06, 08-10.

## Copy-Ready Future Execution Prompt

```text
Execute Phase 5 Packet 02 only after prior gates, Packet 01, P5-HEALTH-001, and design approval. Inspect actual workloads, then implement or document only bounded read-only startup/liveness/readiness/diagnostic health contracts and local tests in approved app files; create no deployment/orchestrator/AWS code. Liveness must not depend on downstream systems, probes must not write/migrate/call billable providers, and public output must reveal no internals. Record design/local evidence separately, leave staging/production pending, stop on unsafe degradation ambiguity, and add evidence/02-completion.md.
```
