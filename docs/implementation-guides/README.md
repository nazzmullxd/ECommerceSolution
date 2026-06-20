# Implementation Guides

Use this folder for beginner-friendly build instructions.

## Planned Documents

| Document | Purpose |
| --- | --- |
| `feature-module-template.md` | Standard template for implementing any feature module. |
| `ai-assisted-development-playbook.md` | How to safely hand one small module at a time to an AI coding tool. |
| `phase-planning-prompts.md` | Seven copy-ready prompts for generating modular Phase 0-6 instruction packages. |
| `testing-guide.md` | Unit, integration, and demo test expectations. |
| `coding-conventions.md` | Naming, folders, dependency rules, DTO conventions, and error handling. |
| `local-development-guide.md` | Local setup, database setup, migrations, seed data, and troubleshooting. |

## Implementation Rule

Every feature should be delivered as a vertical slice:

1. Domain model or value object.
2. Core interface and application logic.
3. Infrastructure implementation.
4. API endpoint or Web UI screen.
5. Unit and integration tests.
6. Demo script.
