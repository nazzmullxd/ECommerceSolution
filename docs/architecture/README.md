# Architecture Docs

Use this folder for system design materials before implementation begins.

## Planned Documents

| Document | Purpose |
| --- | --- |
| `system-design.md` | C4-style system context, container, and component design. |
| `domain-boundaries.md` | Bounded contexts, aggregate ownership, and dependency rules. |
| `api-contracts.md` | API route inventory, DTO shapes, auth rules, and error model. |
| `data-model.md` | Entity relationships, migration strategy, seed data, and audit model. |
| `deployment-architecture.md` | AWS runtime design, environments, networking, and operations. |
| `security-scalability-review.md` | Review findings, risks, mitigations, and approval gates. |

## Architecture Defaults

- Keep `ECommerce.Core` independent from infrastructure and presentation concerns.
- Put external integrations behind Core interfaces.
- Prefer vertical feature slices with clear tests and demos.
- Defer physical `src/` and `tests/` project moves until the system design is approved.
- Design with the AWS Well-Architected pillars: operational excellence, security, reliability, performance efficiency, cost optimization, and sustainability.
