# Packet 01: Repository And Product Discovery

## Packet Metadata

| Field | Value |
| --- | --- |
| Status | Not Started |
| Depends On | None |
| Produces | `artifacts/product-context.md`, `artifacts/glossary.md` |
| Human Reviewers | Product owner, solution architect, development lead |

## Objective

Establish a verified view of the repository, stakeholders, product goals, constraints, assumptions, vocabulary, and measurable success criteria. Later packets must not guess business context that this packet should resolve.

## Read First

- `ECommerce Platform Requirements Roadmap.md`
- `docs/roadmap/phase-0.md`
- `docs/architecture/system-design.md`
- `docs/architecture/security-scalability-review.md`
- Repository solution/project files and existing documentation indexes

## In Scope

- Read-only repository inventory and baseline health evidence.
- Stakeholder/user map, product vision, MVP/production boundaries, constraints, assumptions, glossary, and candidate success metrics.
- Decision ownership and unresolved discovery questions.

## Out Of Scope

- Application, migration, package, project-reference, deployment, or cloud changes.
- Final domain/API/data/security design owned by later packets.
- Inventing market, currency, tax, shipping, compliance, or revenue requirements.

## Required Artifacts

### `artifacts/product-context.md`

Include:

1. Verified repository snapshot: solution file, projects, dependency direction observed, target frameworks, test projects, documentation structure, and dirty-worktree warning.
2. Stakeholder matrix: Customer, Store Admin, Support Agent, Business Owner, Developer, Operator, Security Reviewer, and decision owner.
3. Business goals and explicitly separated non-goals.
4. User personas with jobs, pain points, critical journeys, and prohibited access.
5. MVP versus production-ready versus enterprise scope.
6. Functional, technical, security, privacy, cost, schedule, and local/free-first constraints.
7. Assumption register with status, owner, impact, and validation method.
8. Proposed measurable success metrics; mark unapproved numbers as `Proposed`.
9. Open questions and decision deadlines.

### `artifacts/glossary.md`

Define canonical terms including user/account/profile, role/permission, product/variant/SKU, stock/reservation, cart/checkout session, order/payment, webhook, outbox/domain/integration event, audit/log, RAG/embedding/vector/search document, source of truth, idempotency, correlation ID, PII, RTO, and RPO.

## Questions To Answer

- Which target market, jurisdiction, currencies, languages, tax assumptions, and payment regions are intended?
- What is the MVP launch/demo outcome and what is explicitly excluded?
- Who approves product, security, architecture, and operational decisions?
- Which user journeys are revenue-, security-, or trust-critical?
- What numeric targets are useful now, and which belong to Phase 5?
- What repository state is observed rather than assumed?

## Contract And Safety Rules

| Concern | Rule |
| --- | --- |
| Architecture | Record the current structure; do not move projects or redefine Onion Architecture. |
| Data/API/UI | Capture known scope only; later packets own detailed contracts. |
| Security | Classify personas by allowed/prohibited access and flag unknown privacy jurisdiction. |
| Logging/Audit | Do not copy secrets, credentials, tokens, personal paths, or sensitive local configuration into artifacts. |
| Failure | If a business-critical answer is unavailable, record owner/default/impact and mark dependent acceptance blocked. |

## Required Diagrams

Add a Mermaid stakeholder/product ecosystem diagram showing users, decision owners, the platform, and external provider categories. Do not show AWS resources as provisioned.

## Step-By-Step Instructions

1. Run `git status --short`, `rg --files`, and inspect solution/project files without editing them.
2. Record target frameworks and project references as observed facts.
3. Compare current repository shape with the roadmap's current/future structure.
4. Extract product goals, users, constraints, MVP scope, production scope, and enterprise scope from approved docs.
5. Build the stakeholder and decision-ownership matrix.
6. Write personas and critical journeys, including forbidden actions and sensitive data touched.
7. Create the assumption register; never turn an assumption into a fact through wording.
8. Propose measurable metrics and label unapproved values.
9. Create the glossary and resolve duplicate terms.
10. Run review and verification commands, then request human approval.

## Expected File Areas

- Create/update only `docs/implementation-guides/phase-0/artifacts/product-context.md` and `artifacts/glossary.md`.
- Update packet status in `README.md` only after review.

## Verification And Evidence

| Check | Evidence |
| --- | --- |
| Repository inventory | Commands and summarized output with no secrets. |
| Scope separation | MVP/production/enterprise table reviewed by product owner. |
| Assumptions | Every assumption has status, owner, impact, and validation path. |
| Glossary | Canonical term table has no contradictory definitions. |
| Build/test | Record current baseline result if run; otherwise `not rerun - documentation-only scope`. |
| Scope | `git diff --name-only` contains only allowed Phase 0 documentation. |

Suggested commands:

```powershell
git status --short
rg --files
rg -n "<TargetFramework|<ProjectReference" -g "*.csproj"
git diff --check
git diff --name-only
```

## Review Checklist

- Observed repository facts are separated from target architecture.
- Every stakeholder and decision owner is named by role.
- Personas include ownership and forbidden-access expectations.
- Success metrics are measurable or clearly proposed.
- Market/compliance unknowns are visible, not guessed.
- No source code or unrelated documentation changed.

## Acceptance Criteria

- Both required artifacts exist and are readable by a beginner.
- Product owner accepts goals, non-goals, personas, and phase boundaries.
- Architect accepts repository facts and constraints.
- Every blocker has an owner and required-by phase.
- Evidence is attached and scope verification passes.

## Manual Review And Stop Conditions

Stop if repository evidence contradicts the documented .NET 10/Onion baseline, secrets appear in tracked files, or the requested business scope materially changes the roadmap. Do not repair these issues in this packet; report them for human decision.

## Rollback

Revert only this packet's two artifact files and status entry. Do not alter source files or discard unrelated user changes.

## Handoff

Packet 02 receives the approved glossary, personas, critical journeys, phase boundaries, constraints, assumptions, and decision owners.

## Copy-Ready Execution Prompt

```text
Execute Phase 0 Packet 01 from docs/implementation-guides/phase-0/01-repository-and-product-discovery.md. Read every listed source first. Create only the two named Phase 0 artifacts. Inspect and report repository state without modifying application files. Separate facts, defaults, proposals, and blockers. Run the packet verification commands, show scope evidence, and stop for human approval instead of self-approving.
```

