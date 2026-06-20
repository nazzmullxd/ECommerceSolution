# Packet 02: Configuration, Database, And Migrations

## Packet Metadata

| Field | Value |
| --- | --- |
| Status | Blocked - PHASE0-GATE |
| Depends On | Packet 01 Approved; decisions `P1-DB-001`, `P1-DB-002` |
| Layer Ownership | Core abstractions only if needed; Infrastructure persistence/config; API/Web composition |
| Produces | Local database/config baseline, migration workflow, `evidence/02-completion.md` |
| Human Review | Solution architect, database reviewer, security reviewer |

## Objective

Choose and configure one primary local database path, approve the DbContext boundary, establish typed configuration/startup validation, safe user-secrets, migration workflow, and non-secret seed strategy before entity/auth implementation.

## Read First And Prerequisites

- Approved Phase 0 data/identity/operations contracts and relevant ADRs.
- Packet 01 evidence and conventions.
- Phase 1 roadmap sections 3-5, 9, 20-21, and 24.
- Existing Infrastructure DbContexts, project packages, appsettings files, Program entry points, `.gitignore`, and `NuGet.Config`.

## In Scope

- Decision/evidence for local provider and DbContext boundary.
- Connection configuration by environment without credentials.
- Typed options and startup validation scaffolding for database plus future auth/audit groups.
- Design-time DbContext/migration command strategy.
- Migration naming/review/apply/rollback-forward-fix rules.
- System role/permission seed-data rules and super-admin bootstrap input contract only.

## Out Of Scope

- Identity entities/mappings, auth/token services, real super-admin creation, production database, cloud secrets, automatic production migrations, or multiple local providers working simultaneously without a requirement.

## Likely File Areas To Inspect

- `ECommerce.Infrastructure/Data/`, `ECommerce.Infrastructure/Identity/`, Infrastructure DI/configuration.
- API/Web `appsettings*.json`, `Program.cs`, project files.
- Migration placeholder and future design-time factory location.
- Tests for configuration/database/migrations.

## Architecture And Prohibited Dependencies

- EF Core/provider/design-time code stays in Infrastructure.
- Core must not reference DbContext, EF configuration, connection strings, or provider packages.
- API/Web supply composition/configuration but do not contain mappings or repository logic.
- One physical local database is the MVP default; context split is an internal design decision, not a microservice/database split.

## Data, Configuration, And Seed Contracts

| Area | Required Contract |
| --- | --- |
| Provider | Exactly one approved primary local provider with documented limitations. |
| Context | One context or two contexts sharing one DB; ownership/migration/transaction consequences documented. |
| Connection | Config key only in tracked JSON; actual secret/connection supplied via user-secrets/environment. |
| Options | Typed groups validate required values at startup without printing secret values. |
| Migrations | Generated from Infrastructure, reviewed, deterministic, never auto-applied in production. |
| Seeds | Stable system role/permission identifiers; idempotent; no passwords/tokens. |
| Bootstrap | Inputs from secrets/env; implementation deferred to Packet 07. |

## Security, Logging, And Failure Rules

- Never commit connection credentials, signing keys, bootstrap password, or secret-bearing launch settings.
- Missing required config fails startup with safe key names only.
- Migration failure stops startup/deployment path; no partially initialized app.
- Seed logs may record IDs/result, never credentials or full personal data.
- Do not silently fall back to an unintended provider/database.

## Implementation Steps

1. Verify Phase 0 and Packet 01 gates.
2. Compare SQLite versus SQL Server Developer/LocalDB against team OS, production direction, concurrency, migration fidelity, and tests.
3. Record/approve `P1-DB-001` and select one primary local provider.
4. Review the two existing empty DbContexts and approve `P1-DB-002` with transaction/migration consequences.
5. Remove only redundant provider/package/config paths after package compatibility/security review.
6. Configure DbContext registration and safe connection lookup in Infrastructure/composition root.
7. Define typed options and startup validation without real secret examples.
8. Establish design-time migration commands/factory only if required by the chosen hosting/context design.
9. Define migration review/apply/recovery procedure and seed-data contract.
10. Add configuration/database smoke tests and missing-config negative tests.
11. Generate no Phase 1 entity migration yet unless this packet's approved minimal foundation requires it; prefer Packet 04 for entity migration.
12. Run verification and create evidence.

## Database Changes And Migrations

- A migration is not required merely to prove connectivity.
- If a minimal migration is generated, inspect every operation, ensure no unrelated/destructive changes, and include generated SQL/review evidence without secrets.
- Never hand-edit generated migration without documented reason and reviewer approval.

## API Contract

No endpoint. Startup/configuration failure returns no sensitive API details.

## Test Matrix

| Test | Required Cases |
| --- | --- |
| Configuration | Required key present; missing key fails safely; secret value absent from message/log. |
| Database | Context resolves; local connection succeeds; provider is expected provider. |
| Context boundary | Migrations/transactions follow approved one/two-context decision. |
| Seeds | Idempotent role/permission metadata; no credentials. |
| Migration | Create/apply/recreate workflow documented and tested if migration exists. |

## Verification Commands

Use approved provider/context-specific commands discovered during execution, plus:

```powershell
dotnet build ECommerceSolution.slnx
dotnet test ECommerceSolution.slnx --no-build
dotnet user-secrets list --project ECommerce.API
git diff --check
git diff --stat
git status --short
```

Do not paste secret-list output into evidence.

## Acceptance Criteria

- `P1-DB-001` and `P1-DB-002` are human-approved.
- One primary local provider and context boundary are implemented/tested.
- Secrets are external to tracked configuration.
- Missing configuration fails safely.
- Migration/seed workflow is reproducible and reviewed.
- No identity behavior, credentials, production/cloud work, or later-phase feature was added.
- Evidence and human approval exist.

## Security Stop Conditions

Stop on real credentials in files/output, ambiguous provider fallback, destructive migration, unresolved context transaction boundary, package compatibility uncertainty, or bootstrap design requiring hardcoded credentials.

## Rollback And Handoff

Back up any local development database before risky migration testing. Revert only Packet 02 changes/migration after reviewer direction. Hand the approved provider, context, options, migration, and seed contracts to Packets 04 and 07.

## Copy-Ready Coding Prompt

```text
Execute Phase 1 Packet 02 from docs/implementation-guides/phase-1/02-configuration-database-and-migrations.md only after PHASE0-GATE and Packet 01 approval. Inspect current DbContexts/providers/configuration, obtain human decisions P1-DB-001 and P1-DB-002, implement one safe local database/config/migration path in the correct layers, keep secrets outside tracked files, add tests and evidence/02-completion.md, run all checks, and stop on destructive migration, secret, provider, context-boundary, or package concerns. Do not implement identity behavior or later-phase features.
```
