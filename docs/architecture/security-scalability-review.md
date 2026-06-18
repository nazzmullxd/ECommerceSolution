# Security And Scalability Review

## Purpose

This document records the senior architecture review of the roadmap before implementation. It explains the risks that must be fixed in the design so the team does not build fragile commerce behavior into the foundation.

## Review Lens

The review uses these public references:

- AWS Well-Architected Framework: operational excellence, security, reliability, performance efficiency, cost optimization, and sustainability.
- AWS IAM best practices: least privilege, short-lived credentials, MFA, and secure access patterns.
- OWASP Top 10 and OWASP ASVS: web application security risks and verification controls.
- OWASP Top 10 for LLM Applications: prompt injection, sensitive information disclosure, excessive agency, and vector/embedding weaknesses.
- AWS retry/backoff and idempotency guidance: safe retries for distributed systems.

## Critical Risks And Required Design Fixes

| Area | Problem | Required Fix |
| --- | --- | --- |
| Authentication | Login alone does not prove authorization. | Use role and policy checks for every admin/customer action. Add tests for forbidden access. |
| Authorization | Admin features can accidentally expose customer PII. | Add RBAC, audit logging, and data minimization to admin endpoints. |
| Checkout | Duplicate submit can create duplicate orders or charges. | Require idempotency keys for checkout and payment commands. |
| Payment | Card handling can create PCI scope. | Use hosted/sandbox gateway flow; store only gateway references and status. |
| Payment callbacks | Fake callbacks can mark unpaid orders as paid. | Validate callback signature, source, replay window, and idempotency. |
| Inventory | Concurrent checkout can oversell. | Use stock reservation with expiry and database concurrency checks. |
| Email/jobs | Background work can be lost after DB commit. | Use transactional outbox table before adding SQS/EventBridge. |
| AI/RAG data leakage | Model may leak private customer, order, payment, or internal data. | Do not index private/payment data in Phase 4. Use approved sources, visibility classification, authorization filters, retrieval logs, confidence threshold, and fallback to ticket. |
| AI/RAG prompt injection | User prompts or retrieved documents may try to override system rules. | Treat all retrieved content as untrusted data, reject jailbreak instructions, require source-grounded answers, and test prompt-injection cases. |
| AI/RAG retrieval abuse | Users may retrieve sources outside their role or overuse public AI endpoints. | Authorize before retrieval, filter in the backing query, audit admin usage, add endpoint rate limits, and avoid logging sensitive full prompts. |
| File upload | Images can carry malicious content or huge files. | Validate extension, content type, size, ownership, and storage path. |
| Caching | Stale inventory or prices can corrupt checkout. | Cache catalog reads only. Recalculate checkout totals from database. |
| Secrets | Keys can leak through source control or logs. | Use local user-secrets/environment variables now; Secrets Manager later. |
| Production deployment | A bad release can break checkout, payment, or auth. | Require CI/CD gates, staging smoke tests, migration review, health checks, and rollback/forward-fix plan. |
| Backups | Untested backups may fail during a real incident. | Define RTO/RPO, backup frequency, retention, local restore simulation, and future RDS/S3 restore drills. |
| Observability | Operators may not detect checkout/payment/security failures quickly. | Define logs, metrics, traces, dashboards, alert rules, runbooks, and incident owners before launch. |
| Premature microservices | Service extraction can create distributed transactions, network failures, and operational overhead before value. | Scale the modular monolith first and require extraction criteria, ownership, APIs/events, migration plan, and rollback path. |
| Enterprise promotions | Coupons, stacking, and loyalty can be abused or corrupt totals. | Keep discount calculation server-side, add usage limits, audit records, idempotency, and refund/points reversal rules. |
| Multi-warehouse inventory | Stock can be oversold across warehouses. | Use warehouse-level stock rows, reservations, concurrency tokens, transfer audit, and clear allocation rules. |
| Reporting scale | Analytics queries can overload transactional tables or leak PII. | Use read models/projections, permissioned exports, privacy-safe aggregates, and retention rules. |
| Enterprise permissions | Permission sprawl can grant staff too much power. | Use least privilege, separation of duties, permission ownership, regular review, and sensitive action approval. |
| Multi-region complexity | Active-active writes can corrupt orders, inventory, payments, or loyalty. | Start single-region/Multi-AZ, use backups/read replicas/active-passive only when RTO/RPO justify it. |
| Cost | Managed AWS services can create early spend. | Use local/free tools until design approval; estimate production with Pricing Calculator. |

## Scalability Decisions

| Decision | Reason |
| --- | --- |
| Start with modular monolith. | Lower operational complexity and easier learning path. |
| Use interfaces for infrastructure adapters. | Allows local mock, free tool, and AWS provider swaps. |
| Use outbox before distributed messaging. | Prevents lost integration events without needing cloud services. |
| Add queues in production readiness. | Email, indexing, notifications, and analytics can scale independently. |
| Avoid microservices in MVP. | Checkout, inventory, payment, and order consistency are easier inside one deployable. |
| Cache only safe reads. | Prevents stale price/inventory from affecting payment decisions. |
| Add environment separation before production. | Local, development, staging, and production must use separate configuration, databases, secrets, and approval rules. |
| Add health checks before traffic routing. | Deployments and future load balancers need a safe way to detect readiness and liveness. |
| Scale monolith before extraction. | Indexing, read models, background jobs, safe caching, and horizontal scaling are cheaper and safer than early microservices. |
| Treat reporting/search as derived data. | Read models and indexes can be rebuilt and must not become checkout, payment, or inventory truth. |
| Delay active-active multi-region. | The consistency and operational cost is not justified until business requirements prove it. |

## Security Acceptance Criteria

- Every endpoint has an explicit authorization rule.
- Every write operation has validation and audit behavior.
- Every external callback has signature validation.
- Every sensitive operation has a negative test for unauthorized access.
- Logs exclude passwords, tokens, payment data, and unnecessary PII.
- All secrets are provided through environment/user-secrets locally and a secrets service in production.
- Production deployments require reviewed migrations, smoke tests, rollback/forward-fix plan, and monitoring owner.
- Backup restore has been tested before production launch.
- Enterprise promotions, loyalty, multi-warehouse inventory, and reporting exports have explicit authorization and audit rules.
- Any service extraction has a documented owner, data boundary, API/event contract, observability plan, cost review, and fallback strategy.

## Production Approval Gates

Before provisioning production AWS services:

1. Complete threat model for checkout, payment, admin, file upload, and AI/RAG.
2. Create AWS cost estimate.
3. Define budget and anomaly alerts.
4. Define backup and restore process.
5. Define deployment rollback process.
6. Run security review using OWASP ASVS-inspired checklist.
7. Run a Well-Architected review for the workload.
8. Confirm observability dashboards, alert rules, and runbooks exist.
9. Confirm production access control and incident response ownership.
10. Complete enterprise scaling review before adding microservices, multi-region writes, or paid enterprise data services.
