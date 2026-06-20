# Packet 07: Threat Model And Security Controls

## Packet Metadata

| Field | Value |
| --- | --- |
| Status | Not Started |
| Depends On | Packets 03-06 Approved |
| Produces | `artifacts/threat-model.md` |
| Human Reviewers | Security reviewer, solution architect, module owners |

## Objective

Create an OWASP-informed threat model and verification catalog for authentication, admin, file upload, checkout, payment, webhooks, AI/RAG, APIs, data, dependencies, logging, and operations.

## Read First

- Approved Packets 01-06 artifacts
- `docs/roadmap/phase-0.md`, section 9
- `docs/architecture/security-scalability-review.md`
- `docs/ai-rag/rag-architecture.md`
- Security sections in Phase 1-5 roadmap documents

## In Scope

- Assets, actors, trust boundaries, abuse cases, OWASP Top 10/API/ASVS/LLM categories, likelihood/impact, controls, residual risk, owner, phase, verification, and launch gates.

## Out Of Scope

- Penetration testing, code scans, WAF/IAM rules, production configuration, paid tools, provider credentials, or claiming compliance certification.
- Accepting high/critical residual risk without named human approval.

## Required Artifact

Create `artifacts/threat-model.md` containing:

1. Method, scope, assumptions, scoring scale, risk-acceptance rule, and review cadence.
2. Protected assets and data classifications.
3. Threat actors and abuse personas.
4. Trust-boundary/data-flow diagrams linked to Packet 03.
5. Threat register fields: ID, area, scenario, asset, boundary, OWASP mapping, likelihood, impact, inherent risk, preventive/detective/recovery controls, verification, residual risk, owner, target phase, and status.
6. Required scenarios for auth/session, broken access/BOLA, admin escalation, injection/mass assignment, unrestricted resources/rate abuse, upload/path/content abuse, checkout tampering/duplicate submit, stock overselling, webhook forgery/replay, payment-data exposure, outbox duplicate/loss, secret/config leak, dependency risk, PII logging, audit tampering/failure, prompt injection, retrieval poisoning, unauthorized RAG, hallucinated policy, conversation leakage, and AI cost abuse.
7. Security acceptance tests by phase and manual-review gates.
8. Incident signals and required audit/monitoring evidence.
9. Residual-risk acceptance table with named owner.
10. Deferred production controls clearly mapped to Phase 5.

## Questions To Answer

- What scoring method and threshold require mitigation versus acceptance?
- Which controls must exist in Phase 1/2/3/4 versus Phase 5?
- Which sensitive actions must fail closed if audit or authorization infrastructure fails?
- What anonymous endpoints require abuse controls before production?
- Which AI sources/data are categorically excluded?
- Who can accept residual risk and for how long?

## Contract And Safety Rules

| Concern | Rule |
| --- | --- |
| Architecture | Controls follow module/trust boundaries; no security bypass through Admin or adapters. |
| Data | Minimize PII; no card data; restricted data never enters public AI index. |
| API/UI | Server-side validation/auth/ownership; bounded resources; safe errors; upload allowlists. |
| Security | Defense in depth; negative/adversarial tests; explicit residual-risk ownership. |
| Logging/Audit | Security signals use IDs/redacted metadata; never log secrets, tokens, full private prompts, addresses, payment payloads, or support messages. |
| Failure | Fail closed for auth, privilege, payment verification, and required sensitive audit; define safe degradation elsewhere. |

## Required Diagrams

- Mermaid threat/data-flow diagram with trust boundaries.
- Mermaid checkout/payment/webhook attack path and controls.
- Mermaid AI/RAG prompt/retrieval/source threat path and controls.
- Mermaid security-review/mitigate/verify/accept/escalate decision flow.

## Step-By-Step Instructions

1. Define assets, actors, boundaries, scoring, and acceptance authority.
2. Walk each required user/system flow and enumerate abuse cases.
3. Map scenarios to OWASP categories without treating mapping as mitigation.
4. Score inherent risk and define preventive, detective, and recovery controls.
5. Map every control to owner, phase, and objective verification evidence.
6. Add negative/adversarial tests and operational detection signals.
7. Calculate/document residual risk and identify blockers.
8. Separate local/MVP controls from deferred production defenses.
9. Draw required diagrams and cross-check against API/data/identity contracts.
10. Obtain explicit security approval; AI cannot accept residual risk.

## Expected File Areas

- Create/update only `artifacts/threat-model.md` and Packet 07 status.

## Verification And Evidence

| Check | Evidence |
| --- | --- |
| Scenario coverage | Every required area has at least one threat. |
| Control quality | Preventive, detective, recovery, and verification fields complete. |
| Ownership | Every high/critical risk has owner and target phase. |
| AI safety | Injection, poisoning, authorization, hallucination, leakage, and cost included. |
| Logging privacy | Prohibited data list and redaction verification exist. |
| Scope | No security/deployment code or services changed. |

```powershell
rg -n "Auth|Admin|Upload|Checkout|Payment|Webhook|Outbox|AI/RAG|Logging|Dependency" docs/implementation-guides/phase-0/artifacts/threat-model.md
rg -n "Likelihood|Impact|Residual|Owner|Verification|Target Phase" docs/implementation-guides/phase-0/artifacts/threat-model.md
git diff --check
git diff --name-only
```

Build/test evidence may reuse Packet 01. Threat scenarios and verification mappings are the design-test evidence.

## Review Checklist

- Required threat areas are complete.
- High/critical risks have controls, tests, owners, and phases.
- Payment/webhook controls include signatures, replay, amount/currency, and idempotency.
- Upload controls include type/content/size/name/path/ownership.
- AI controls include pre-retrieval authorization and approved sources.
- Sensitive logging and audit-failure risks are explicit.

## Acceptance Criteria

- Artifact, register, tests, and four diagrams exist.
- Security reviewer approves scoring and control mappings.
- No unowned high/critical residual risk remains.
- Phase 1 entry security controls are clearly identified.
- No compliance or production-security claim is made without evidence.

## Manual Review And Stop Conditions

Stop on unowned high/critical risk, request to weaken payment/authorization/AI controls, missing data classification, or request to log sensitive payloads. Escalate rather than documenting an unsafe default.

## Rollback

Revert only the threat-model artifact and Packet 07 status. Preserve earlier approved contracts.

## Handoff

Packets 08-09 receive operational detection, recovery, deferred-control, security-test, and residual-risk requirements.

## Copy-Ready Execution Prompt

```text
Execute Phase 0 Packet 07 from docs/implementation-guides/phase-0/07-threat-model-and-security-controls.md after Packets 03-06 are approved. Create only artifacts/threat-model.md and the allowed status update. Build the complete risk/control/verification register and four diagrams, cover every required commerce and AI threat, assign owners/phases, avoid sensitive logging, and stop for human security review on any high/critical residual risk.
```

