# Packet 01: Solution Baseline And Conventions

## Packet Metadata

| Field | Value |
| --- | --- |
| Status | Blocked - PHASE0-GATE |
| Depends On | Approved Phase 0 acceptance/contracts |
| Layer Ownership | Solution/project configuration and documentation; no business feature owner |
| Produces | Verified baseline, architecture tests/conventions, `evidence/01-completion.md` |
| Human Review | Solution architect and development lead |

## Objective

Verify .NET 10 compatibility and current repository health, clarify project responsibilities/dependencies, remove or quarantine template ambiguity safely, establish conventions, and add dependency-rule tests without implementing identity behavior.

## Read First And Prerequisites

- Approved Phase 0 approval record, ADRs, and cross-phase contracts.
- `docs/roadmap/phase-1-mvp-foundation.md`, sections 3-6 and 22-24.
- This package README.
- All `.slnx`, `.csproj`, entry-point, existing skeleton, test, `NuGet.Config`, `.gitignore`, and config files.

Prerequisite: `PHASE0-GATE` passed and a clean understanding of existing user changes. A dirty worktree is allowed, but unrelated changes must be identified and preserved.

## In Scope

- SDK/workload/restore/build/test/package-vulnerability baseline.
- Project reference graph and automated dependency-rule checks.
- Project/layer responsibility and folder/naming/nullable/async conventions.
- Inventory and disposition plan for sample/placeholder/duplicate skeletons.
- Local setup prerequisites and safe command guide.

## Out Of Scope

- Identity, EF mappings/migrations, auth middleware, tokens, endpoints, database choice, project moves, broad namespace refactor, or adding business modules.

## Likely File Areas To Inspect

- `ECommerceSolution.slnx`, all `*.csproj`, `NuGet.Config`.
- `ECommerce.API/Program.cs`, `ECommerce.Web/Program.cs`.
- Existing Core/Infrastructure user/identity/service skeletons.
- `ECommerce.Tests/*` and documentation.

Exact edits must follow inspection. Do not delete placeholders merely because they are empty; record whether to replace, retain, or remove in a scoped change.

## Architecture And Prohibited Dependencies

| Project | Allowed | Prohibited |
| --- | --- | --- |
| Core | BCL/framework-neutral domain/application code | Infrastructure, API, Web, EF Core, Identity, JWT, HTTP/provider SDKs |
| Infrastructure | Core plus approved persistence/Identity adapters | API/Web presentation dependencies |
| API | Core and Infrastructure composition root | Business rules or direct ad hoc EF/provider access in controllers |
| Web | Core and approved composition wiring | Separate authorization rules or direct persistence in controllers/views |
| Tests | Projects under test | Production secrets and order-dependent shared test state |

## Data, API, And UI Contracts

- No database, API, or UI behavior changes in this packet.
- Preserve `/api/v1` and Problem Details contracts for later packets.
- Sample controllers/pages must be inventoried; removal requires explicit scope and test evidence.

## Security, Logging, And Failure Rules

- Do not print environment variables, user-secrets, NuGet credentials, or config values.
- Vulnerability output may record package IDs/versions/advisories but no private source credentials.
- Restore/build failure is evidence; do not bypass it by weakening package sources or target framework.
- Project-reference violation blocks later packets.

## Implementation Steps

1. Confirm Phase 0 gate files and accepted contract IDs exist.
2. Run and record `dotnet --info`; verify a compatible .NET 10 SDK.
3. Inventory solution projects, target frameworks, references, packages, entry points, skeletons, and tests.
4. Run restore/build/test baseline before edits.
5. Run package vulnerability/deprecation review using supported CLI commands.
6. Write or refine coding/folder/dependency conventions in Phase 1 evidence or an approved existing guide.
7. Add focused architecture/dependency tests using existing test tooling when possible; do not introduce a package without compatibility/security review.
8. Decide the scoped disposition of template/sample/duplicate skeleton files; apply only changes necessary for a clean foundation baseline.
9. Run restore/build/test and compare with baseline.
10. Record evidence and request review.

## Database Changes And Migrations

None. Any generated migration or database configuration is a scope violation.

## API Contract

None. Existing sample endpoints may be removed only if explicitly approved as template cleanup; do not add Phase 1 endpoints.

## Test Matrix

| Test | Required Result |
| --- | --- |
| Core dependency | Fails if Core references Infrastructure/API/Web or forbidden packages. |
| Infrastructure dependency | Fails on API/Web reference. |
| Solution baseline | Restore/build/test commands have recorded results. |
| Nullable/compiler | Existing warning policy is documented; do not hide warnings indiscriminately. |
| Scope | No auth/database/business behavior introduced. |

## Verification Commands

Use the README shared commands plus repository-appropriate reference inspection:

```powershell
rg -n "<TargetFramework|<ProjectReference|<PackageReference" -g "*.csproj"
git diff --check
git diff --stat
git status --short
```

## Acceptance Criteria

- .NET 10 SDK/restore/build/test status is recorded accurately.
- Project responsibilities and forbidden dependencies are enforceable and documented.
- Existing skeleton/template disposition is explicit and scoped.
- No package version was changed without compatibility/vulnerability evidence.
- No database, auth, endpoint, or later-phase behavior was added.
- `evidence/01-completion.md` and human approval exist.

## Security Stop Conditions

Stop for review if Phase 0 gate is missing, SDK/package compatibility is uncertain, a vulnerability has no disposition, project references must change, secrets appear, or cleanup would delete user work.

## Rollback And Handoff

Revert only Packet 01 changes, preserving pre-existing work. Handoff the verified project graph, commands, conventions, skeleton disposition, and baseline results to Packets 02-03.

## Copy-Ready Coding Prompt

```text
Execute Phase 1 Packet 01 from docs/implementation-guides/phase-1/01-solution-baseline-and-conventions.md only after PHASE0-GATE passes. Inspect the repository first. Verify .NET 10, restore/build/test, project references, package vulnerabilities, and existing skeletons. Implement only approved baseline/convention/dependency-test cleanup, preserve unrelated work, add evidence/01-completion.md, run all checks, and stop for human review on package, project-reference, secret, or destructive-cleanup concerns. Do not implement identity, database, API, or later-phase features.
```

