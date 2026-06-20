# Packet 04: Security Hardening, Scanning, And Approval Gates

## Packet Metadata

| Field | Value |
| --- | --- |
| Status | Blocked - PHASE0/PHASE1/PHASE2/PHASE3/PHASE4-GATE |
| Depends On | Packet 01 Approved; Phase 1-4 security evidence; `P5-SEC-001`, `P5-RATE-001` |
| Ownership | Security owns control baseline; module owners own behavior/remediation |
| Manual Review | Mandatory pre-code and post-test security, Identity, Payment, Upload, AI, privacy, operations, and UX review; manual approval for rate limits and payment behavior |
| Produces | Production hardening/security gate package and `evidence/04-completion.md` |

## Objective

Review and later verify HTTPS, browser/API controls, authentication/session/admin/MFA, authorization, rate limits, upload safety, dependencies/static analysis, input/error safety, secret scanning, and OWASP-style approval gates without weakening existing money, identity, or AI rules.

## Read First And Prerequisites

- Packet 01 surface/data/owner inventory and approved Phase 1-4 threat/security/test evidence.
- Actual auth transport, trusted proxy, hosting/browser topology, upload decoder/storage, webhook provider, and AI enablement state.
- Public OWASP Top 10/API/ASVS/LLM guidance referenced by roadmap; use as review framework, not certification claim.

## In Scope

- TLS/HTTPS redirect/HSTS/proxy-forwarded headers, secure headers/CSP/referrer/content-type/frame/permissions policy, CORS, CSRF/anti-forgery, secure cookies/token transport, session timeout/revocation, lockout, admin MFA or signed compensation.
- Endpoint authorization/ownership inventory and negative tests; admin/network/access restrictions design.
- Rate/concurrency/body/page/filter limits for auth, registration/refresh/reset, checkout/payment initiation/webhook, upload, search/AI, admin/report/indexing; trusted proxy/IP/key strategy and safe `429`.
- Upload validation/decoder/storage/scan/quarantine strategy, dependency/vulnerability/secret/static/license scan design, production error/config validation, OWASP risk/gate evidence.

## Out Of Scope

- WAF/IAM/cloud rule, certificate/domain, real MFA provider, payment semantic change, new upload type, paid scanner, production pentest claim, compliance certification, CI workflow code, or accepting unresolved critical vulnerability.

## Ownership And Operational Contracts

- **Owns:** cross-cutting security baseline, gate checklist, scan policy/severity/SLA, rate-limit policy contract, browser/security header profile, production auth/admin requirements.
- **Reads:** module auth/ownership/validation/payment/upload/AI behavior and deployment topology.
- **Must not own/mutate:** payment settlement/webhook semantics, Identity source, upload business lifecycle, AI safety thresholds, IAM, or risk acceptance.

## Evidence Required By Tier

| Tier | Required Evidence |
| --- | --- |
| Design Evidence | Threat/control matrix, endpoint authorization/rate inventory, headers/CORS/CSRF/session/MFA/upload/scan policies and named approvals. |
| Local Verification Evidence | Automated header/CORS/CSRF/auth/BOLA/rate/upload/error tests and dependency/static/secret scan reports using safe tools. |
| Staging Evidence | TLS/proxy/cookie/CSP/CORS/CSRF/MFA/limits/upload/scan/error behavior and security regression against release artifact. |
| Production Approval Evidence | Security signs unresolved-risk disposition, exact rate/payment/admin/MFA/upload controls and go-live gate. |

## API And UI Contract

- Every route has explicit anonymous/auth/role/permission/ownership/CSRF/rate/body/content-type behavior and safe Problem Details.
- Browser state-changing cookie requests require anti-forgery; API bearer flows follow approved CORS/token transport. Never wildcard credentialed origins.
- Security headers are environment/topology aware and tested for Web/API/media/payment redirect compatibility.
- `429` includes safe bounded retry guidance; rate-limit identifiers never expose raw IP/user/token and do not replace lockout/idempotency/provider webhook validation.

## Likely File Areas To Inspect

- API/Web Program/middleware/auth/CORS/antiforgery/headers/rate configuration, controllers/endpoints, Identity/token services, upload/payment/AI adapters, package manifests, Tests security suites.
- No workflow, WAF, IAM, certificate, DNS, or cloud file is created.

## Architecture And Prohibited Dependencies

- Security checks are server-side; UI hiding/WAF/rate limiting are defense in depth, not authorization.
- Trusted proxy/forwarded-header configuration must be exact before trusting scheme/IP; otherwise deny/avoid IP security decisions.
- Payment callbacks retain signature/replay/idempotency/amount/currency validation; rate limits cannot drop valid retries without provider-approved behavior.
- Feature flags cannot bypass authorization/validation/audit.

## Database Changes And Migrations

None expected. MFA/session/rate/audit persistence change requires owning Phase 1/2 schema and migration/manual review. Do not add ad hoc security tables from this packet.

## Security, Logging, Audit, And Failure Rules

- Define vulnerability severities, exploitability review, exception owner/expiry/compensation, and block criteria; critical/high exploitable findings block release.
- Never log headers/cookies/tokens/passwords/secrets/payment payloads/full PII/upload bytes/AI prompts. Rate/scan logs use hashed/bucketed safe identifiers.
- Missing invalid security config fails startup or feature closed. CSP/reporting/scan/rate backend failure has explicit safe degradation.
- Audit admin auth/MFA changes, role/permission, security config/exception, upload rejection/quarantine, payment validation anomalies, AI disable/safety actions.

## Implementation Steps

1. Reconcile exact hosting/auth/browser/payment/upload/AI topology and endpoint inventory.
2. Obtain pre-code approval for headers/CSP/CORS/CSRF/session/admin MFA/limits/trusted proxy/upload scan/scan severities.
3. Define per-endpoint control matrix and negative/abuse test plan.
4. Plan/implement approved local app hardening and tests only after gate; do not change payment semantics.
5. Run dependency vulnerability, secret, static and license checks with versions/results/exceptions.
6. Execute local OWASP-inspired review and define staging TLS/proxy/browser/security test plan.
7. Obtain post-test security/module reviews; record unresolved findings with severity/expiry/owner.

## Test And Review Matrix

| Test | Required Cases |
| --- | --- |
| Browser/API | HTTPS/HSTS/proxy, header/CSP compatibility, CORS preflight/credentials, CSRF success/failure, secure cookies. |
| Identity/admin | Login/refresh/logout/revocation/lockout/session timeout, MFA/compensation, self-escalation/BOLA negatives. |
| Rates/resources | Every endpoint group, burst/concurrency/body/page/filter, proxy spoof, `429`, lockout/idempotency interaction. |
| Payment | Signed/invalid/duplicate/delayed webhook under limits; no semantic/availability regression. |
| Upload/AI | Content decoder/size/path/quarantine/scan failure; injection/private retrieval/disable/rate regressions. |
| Scans/errors | Vulnerable/transitive/secret/static/license reports; safe config/startup/Problem Details/no stack/internal details. |

## Verification Commands

```powershell
dotnet build ECommerceSolution.slnx
dotnet test ECommerceSolution.slnx --no-build
dotnet list ECommerceSolution.slnx package --vulnerable --include-transitive
rg -n "UseHsts|UseHttpsRedirection|Cors|Antiforgery|RateLimit|ForwardedHeaders|SameSite|SecurePolicy|Content-Security-Policy|AllowAnyOrigin" ECommerce.API ECommerce.Web ECommerce.Infrastructure ECommerce.Tests
git diff --check
git status --short
```

## Acceptance Criteria

- Every endpoint and browser flow has explicit tested auth/ownership/CSRF/CORS/rate/body/header/error behavior.
- Admin MFA or approved time-bound compensation, payment/webhook safety, upload hardening, and AI controls pass owner review.
- Dependency/static/secret/license scan policy and local reports have no unaccepted blocking finding.
- Design/local evidence pass; staging/production remain pending and require exact topology/release verification.

## Manual Approval Gates

Manual approval is mandatory for auth/session/admin/MFA, CORS/CSRF/CSP, trusted proxy, rate limits, payment/webhook behavior, upload scan/quarantine, vulnerability exceptions, secrets/IAM implications, staging security result, and production security gate.

## Stop Conditions

Stop for missing topology/owner, wildcard credentialed CORS, CSRF gap, insecure cookie/token storage, trusted proxy ambiguity, auth/ownership bypass, payment semantic change, unsafe upload/scan failure-open, client-controlled limits, secret/sensitive logging, critical/high unaccepted vulnerability, missing manual rate/payment approval, or documentation marked as staging/production proof.

## Rollback / Forward-Fix And Handoff

Future hardening rollout must have per-control compatibility rollback without reopening insecure access. Disable affected feature, restore last secure config, and forward-fix policy/test; never remove authorization to recover availability. Hand approved gates to Packets 05-06, 08-12.

## Copy-Ready Future Execution Prompt

```text
Execute Phase 5 Packet 04 only after prior gates, Packet 01, P5-SEC-001/P5-RATE-001, and mandatory pre-code security/module approval. Inspect actual auth/browser/proxy/payment/upload/AI topology, then implement or document only approved app hardening and local security tests/scans. Add no workflow/WAF/IAM/cloud/certificate/secret, do not change payment settlement semantics, and never weaken authorization/CSRF/upload/AI controls. Record rate/payment approvals and all four evidence tiers separately, leave staging/production pending, stop on blocking vulnerability or ambiguity, and add evidence/04-completion.md.
```
