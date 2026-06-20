# Packet 04: Identity/Access Entities And EF Core Mappings

## Packet Metadata

| Field | Value |
| --- | --- |
| Status | Blocked - PHASE0-GATE |
| Depends On | Packets 02-03 Approved; Phase 0 identity/data contracts |
| Layer Ownership | Core framework-neutral entities/contracts; Infrastructure Identity/EF types and mappings |
| Produces | Foundation model, migration, tests, `evidence/04-completion.md` |
| Human Review | Solution architect, database reviewer, security reviewer |

## Objective

Implement the Phase 1 identity/access/audit persistence model and constraints without coupling Core to ASP.NET Identity or EF Core and without adding account/token/API behavior.

## Read First And Prerequisites

- Approved Phase 0 data and identity/access/audit contracts/ADRs.
- Packet 02 provider/context/migration decisions.
- Packet 03 result/error/logging contracts.
- Phase 1 roadmap sections 4-5 and 8-9.
- Existing `ApplicationUser`, DbContexts, Core user skeletons, Infrastructure repository/service skeletons, and migrations.

## In Scope

- `ApplicationUser`, `CustomerProfile`, `Role`, `Permission`, `UserRole`, `RolePermission`, `RefreshToken`, and `AuditLog` model.
- Framework boundary decision for Identity-derived persistence types.
- EF mappings, relationships, constraints, indexes, classifications, concurrency/audit timestamps, and migration.
- Repository/application contracts only where later services require them.

## Out Of Scope

- Registration/login, password policy, token generation/validation, seeding/bootstrap execution, endpoints, audit service behavior, customer addresses, product/cart/order/payment/inventory/AI entities.

## Likely File Areas To Inspect

- Core `Entities/`, identity/access/audit contracts/value objects.
- Infrastructure `Identity/`, `Data/`, EF configurations/migrations.
- Existing `UserInfo`, `ApplicationUser`, duplicate `UserService`, repositories, and DbContexts.
- Mapping/constraint/migration tests.

## Architecture And Prohibited Dependencies

- ASP.NET Identity-derived `ApplicationUser`/role persistence types belong in Infrastructure unless an accepted ADR explicitly says otherwise.
- Core owns framework-neutral customer profile, access/audit/token business concepts and contracts.
- Do not reference Identity/EF packages from Core.
- Do not duplicate framework identity tables and custom role/user mappings accidentally. Approve how custom assignment metadata (`AssignedBy`, timestamps) is represented.
- Controllers/API do not access DbContexts directly.

## Entity And Database Contract

| Entity | Required Integrity |
| --- | --- |
| ApplicationUser | Unique normalized email, active/account state, Identity security/lockout fields, timestamps. |
| CustomerProfile | One-to-one user link, allowed profile fields, no auth secret duplication. |
| Role | Unique normalized name, system-role marker. |
| Permission | Unique stable code and category. |
| UserRole | Unique user/role; assignment actor/time if approved. |
| RolePermission | Unique role/permission; grant actor/time if approved. |
| RefreshToken | Unique token hash, user/family indexes, expiry/revocation/replacement/reuse metadata; never raw token. |
| AuditLog | Actor/action/target/result/reason/correlation/timestamp/redacted metadata indexes; append-oriented. |

Define delete/restrict behavior explicitly. Identity/account deletion must not cascade-delete protected audit history silently.

## API And UI Contracts

None. Entities are not response DTOs. Never expose password hash, security stamp, token hash, audit metadata, or internal normalized fields automatically.

## Security, Logging, And Failure Rules

- Never create fields for plaintext password/token, signing secret, cookie, authorization header, or card data.
- Bound metadata/strings and avoid arbitrary sensitive JSON dumping into AuditLog.
- Duplicate email/role/permission/mapping and concurrency failures must be detectable and mapped later.
- Migration must be reviewed for accidental tables/columns/cascades and no unrelated destructive operations.

## Implementation Steps

1. Verify Phase 0 contracts and Packet 02 context/provider decisions.
2. Inventory existing skeleton types and choose replace/rename/remove actions without preserving duplicate concepts.
3. Record the framework boundary: Identity-derived persistence types in Infrastructure; framework-neutral concepts in Core.
4. Model entities/value objects with minimal required fields and classifications.
5. Define relationships, delete behavior, unique constraints, indexes, lengths, timestamps, and concurrency where approved.
6. Configure mappings in Infrastructure and register them with the approved DbContext boundary.
7. Add repository/contracts only for concrete Packet 05-08 needs; avoid generic repository abstraction by habit.
8. Generate the Phase 1 foundation migration using the approved startup/context project.
9. Review migration operations and generated SQL for the chosen provider; back up any existing local DB before apply.
10. Add mapping/constraint/index/migration tests, including raw-token absence.
11. Apply/recreate local test DB, run all tests, record evidence, and request review.

## Database Changes And Migrations

This packet owns the initial Phase 1 foundation migration. It must contain only approved identity/access/profile/refresh/audit schema. No later-phase table is allowed. Never auto-apply to production.

## API Contract

None.

## Test Matrix

| Test | Required Cases |
| --- | --- |
| Mapping | Every entity mapped to approved context/schema/table. |
| Unique | Email, role, permission, user-role, role-permission, token hash. |
| Relationships | Profile/user; tokens/user; mappings; audit actor optional/restricted. |
| Delete behavior | Protected audit/history not silently cascaded. |
| Token safety | Schema stores hash only; raw token property absent. |
| Migration | Fresh create succeeds; expected indexes/constraints exist; no later tables. |
| Architecture | Core has no EF/Identity dependency. |

## Verification Commands

Use the approved migration commands from Packet 02, plus:

```powershell
dotnet build ECommerceSolution.slnx
dotnet test ECommerceSolution.slnx --no-build
rg -n "Password|Token|Secret|Cookie|Authorization" ECommerce.Core ECommerce.Infrastructure
git diff --check
git diff --stat
git status --short
```

Inspect matches for prohibited storage versus legitimate hash/config names.

## Acceptance Criteria

- All eight required entity concepts and integrity rules are implemented.
- Core remains EF/Identity-free.
- Raw refresh tokens and secrets cannot be persisted.
- Migration is reviewed, reproducible, scoped, and tested on a fresh local DB.
- No account behavior/API/seeding/later-phase table was added.
- Evidence and architecture/database/security approvals exist.

## Security Stop Conditions

Stop on plaintext token/password storage, duplicate Identity/custom tables, unsafe cascade, arbitrary sensitive audit payload, unreviewed destructive migration, provider mismatch, or framework dependency in Core.

## Rollback And Handoff

Back up before apply. Prefer forward correction after shared use; during isolated local development, rollback only with database reviewer approval. Hand entity/mapping/constraint contracts to Packets 05-08.

## Copy-Ready Coding Prompt

```text
Execute Phase 1 Packet 04 from docs/implementation-guides/phase-1/04-identity-access-entities-and-mappings.md only after PHASE0-GATE and Packets 02-03 approval. Inspect and reconcile existing skeletons. Implement only the eight approved identity/access/profile/refresh/audit concepts with framework-neutral Core and Identity/EF Infrastructure mappings, generate/review the scoped migration, add constraint/migration/security tests and evidence/04-completion.md, and stop on raw-token storage, duplicate mapping, cascade, migration, or dependency concerns.
```

