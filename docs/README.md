# E-Commerce Solution Design Docs

This folder contains the planning and architecture materials for the e-commerce platform. The goal is to make the product design clear enough that a novice developer or AI coding agent can implement each module without guessing.

## Document Map

| Folder | Purpose |
| --- | --- |
| `architecture/` | System design, domain boundaries, API contracts, data model, deployment decisions. |
| `roadmap/` | MVP and production module plans, sprint breakdowns, delivery sequencing. |
| `implementation-guides/` | Step-by-step feature templates and coding checklists. |
| `ai-rag/` | RAG design, Vector DB strategy, embeddings, retrieval flows, evaluation, guardrails. |

## Current Authoritative Roadmap

The main roadmap currently lives at:

```text
../ECommerce Platform Requirements Roadmap.md
```

Use that file as the source of truth for the overall direction. Use the phase documents below for detailed system design and implementation guidance.

## Phase Design Documents

| Phase | Document |
| --- | --- |
| Phase 0: Discovery and System Design | `roadmap/phase-0.md` |
| Phase 1: MVP Foundation | `roadmap/phase-1-mvp-foundation.md` |
| Phase 2: MVP Commerce Flow | `roadmap/phase-2-mvp-commerce-flow.md` |
| Phase 3: MVP Experience Layer | `roadmap/phase-3-mvp-experience-layer.md` |
| Phase 4: AI-Enhanced MVP | `roadmap/phase-4-ai-enhanced-mvp.md` |
| Phase 5: Production Readiness | `roadmap/phase-5-production-readiness.md` |
| Phase 6: Scale and Enterprise | `roadmap/phase-6-scale-enterprise.md` |

## Design Order

1. Finalize system design and domain boundaries.
2. Define MVP modules and API contracts.
3. Define data model and migration strategy.
4. Define AI/RAG architecture.
5. Write novice-friendly implementation guides.
6. Start application implementation module by module.

## Design Rules

- Use free/public resources and local tools during design.
- Treat AWS managed services as production targets until cost controls are approved.
- Keep diagrams in Mermaid where possible so they are readable in plain Markdown.
- Put every major decision in a document before code implementation starts.
