# Packet 11: Single-Region HA, Active-Passive DR, And Residency

## Packet Metadata

| Field | Value |
| --- | --- |
| Status | Blocked - PHASE0/PHASE1/PHASE2/PHASE3/PHASE4/PHASE5-GATE |
| Depends On | Packets 01-02 Approved; Phase 5 RTO/RPO/restore/topology; `P6-DR-001` |
| Decision Type | Planning only; single-region/Multi-AZ target first; no multi-region writes |
| Manual Review | Business continuity, architecture, SRE/operations, DB/payment/inventory, security/privacy/residency, backup, network, finance/cost and incident review |
| Produces | HA/active-passive/residency decision model and `evidence/11-completion.md` |

## Objective

Plan single-region high availability first, then replicas/CDN/backup-region and active-passive disaster recovery only when approved RTO/RPO, residency and business impact justify cost; explicitly reject premature active-active commerce writes.

## Read First And Prerequisites

- Approved Phase 5 topology/health/SLO/backup/restore/incident/load/cost and actual production availability/failure evidence.
- Order/payment/inventory/loyalty consistency and provider/webhook/DNS/data-protection dependencies.
- Applicable data residency/transfer/legal requirements from named counsel/privacy owner.

## In Scope

- Failure-domain inventory: process/task/AZ/DB/storage/queue/provider/Region/control plane/human/deployment/data corruption.
- Single-region multiple app replicas, Multi-AZ relational DB target, statelessness/shared keys/state, health/drain/rolling deployment, capacity and AZ failure behavior.
- Read replicas only for approved stale/read workloads, CloudFront/static-media caching, backup/PITR/media replication and recovery-region copies.
- Active-passive pilot-light/warm-standby criteria, data/config/secret/artifact/infra/runbook replication, DNS/traffic failover, recovery order, payment/webhook/provider coordination, RTO/RPO tests and failback.
- Data residency/sovereignty/classification/transfer/encryption/key/log/backup constraints.
- Active-active rejection criteria and exceptional ADR requirements.

## Out Of Scope

- AWS resource/IaC/account/region/network, actual failover, active-active writes/global database, conflict-free commerce, multi-master inventory/order/payment/loyalty, paid DR environment, production DNS change or legal advice.

## Ownership And Boundaries

- **Owns:** HA/DR stage/trigger/failure/recovery/failover/failback/residency design and evidence requirements.
- **Reads:** workload/data/dependency/RTO/RPO/cost/security/provider contracts.
- **Must not own/mutate:** business conflict semantics, production topology, AWS resources, legal requirements or go-live authority.

## Evidence Required

| Evidence | Required Proof |
| --- | --- |
| Entry Evidence | Phase 5 restore/incident/health/on-call/SLO stable in single-region design. |
| Measured Need Evidence | Outage/revenue/RTO/RPO/residency and regional-failure business impact/cost model. |
| Design Evidence | Failure domains, HA/DR stages, data/secret/provider recovery, drills/failback/residency/cost/rollback. |
| Monolith-First Evidence | Single-region replica/Multi-AZ/backup/restore/load/failure drills before regional expansion. |
| Extraction/Cloud Evidence | Separate AWS service/account/region/cost/IAM/DR approval; no active-active without exceptional ADR. |

## Data And Contract Design

Classify each store/copy as authoritative, replicated, rebuildable or backup-only, with consistency/RPO, region/residency, encryption/key owner, replication/failover/failback, deletion/legal hold and integrity verification. Read replicas cannot receive writes or serve transaction/auth truth where lag is unsafe.

Define recovery dependency order for secrets/keys/config/network/database/media/app/workers/outbox/search/read models/webhooks, and which derived data rebuilds rather than replicates.

## API And UI Contract

- Public API/webhook DNS and idempotency remain stable through failover; clients may retry safely.
- Maintenance/read-only/degraded behavior is explicit. Do not accept checkout/payment/stock writes when authoritative state/region ownership is uncertain.
- UI shows safe unavailable/retry/order-review state without claiming payment/order outcome from stale replica.

## Likely File Areas To Inspect

- Phase 5 topology/health/backup/runbooks/cost and module provider/idempotency/feature-flag contracts.
- Planning docs only; no cloud/deployment files.

## Architecture And Prohibited Dependencies

- Single writer/region for transactional commerce. Active-passive promotes one writer under controlled fencing.
- No active-active order/payment/inventory/promotion/loyalty writes, last-write-wins or timestamp conflict resolution.
- Read replicas/CDN/cache serve only approved stale/public reads with freshness/invalidation.
- Failover without tested failback/data reconciliation is incomplete.

## Database Changes And Migrations

None. HA/replica/global DB features cannot drive provider-specific schema without Packet 13/Phase 5 migration approval. Schema versions must be compatible across recovery artifacts.

## Security, Logging, Audit, And Failure Rules

- Region/account/replica/backup access, encryption keys, secrets, audit/log retention and data transfer obey residency/least privilege.
- Log failover decision IDs/stages/versions/status/timing/correlation and safe aggregate data, not credentials/private endpoints/customer data.
- Split-brain/uncertain writer/DNS partial/provider callback to old region/backlog/replay/lag/key unavailable/restore corrupt/failback conflict fails closed and invokes incident command.
- Any residency uncertainty blocks replication/region choice.

## Planning Or Implementation Steps

1. Approve RTO/RPO/residency/business impact and failure-domain/owner inventory.
2. Prove single-region replicas/Multi-AZ target, statelessness, capacity, rolling failure and restore maturity first.
3. Classify authoritative/replicated/rebuildable/backup data and dependency recovery order.
4. Evaluate replicas/CDN/backup region then active-passive options with cost/quotas/security/operations.
5. Define fencing, traffic/webhook failover, reconciliation, RTO/RPO drill, failback and abort.
6. Document active-active rejection and exceptional ADR prerequisites/conflict model/test burden.
7. Obtain planning review; no cloud or failover implementation.

## Test And Review Matrix

| Review | Required Cases |
| --- | --- |
| Single-region | Process/task/AZ/DB failover, capacity, health/drain, deployments and backup restore. |
| Replicas/CDN | Lag/stale/read-only/failure/invalidation/privacy and no transaction truth. |
| Active-passive | Region outage, writer fencing, DNS/webhook, RPO gap, backlog/idempotency, recovery order. |
| Failback | Data divergence/reconciliation, old writer isolation, schema/config/key compatibility. |
| Residency/security | Data/log/audit/backup/key transfer, access, deletion/legal hold and evidence. |
| Active-active rejection | Payment/order/inventory/loyalty conflict cases and unproven operational/cost maturity. |

## Verification Commands

```powershell
rg -n "RTO|RPO|Multi-AZ|Read Replica|CloudFront|Backup|Active-Passive|Active-Active|Residency|Failover|Failback" docs/roadmap docs/implementation-guides/phase-5 docs/implementation-guides/phase-6
git diff --check
git status --short
```

## Acceptance Criteria

- Single-region/Multi-AZ plus tested backup/restore remains the first production resilience target.
- Replica/CDN/active-passive introductions have measured triggers, data consistency/residency, cost, security, drills and failback.
- Transactional writer fencing and fail-closed uncertainty preserve money/stock/order/loyalty integrity.
- Active-active writes remain rejected absent exceptional separate ADR; no AWS implementation occurs.

## Manual Approval Gates

Manual approval is mandatory for RTO/RPO/residency, authoritative/replicated data, Multi-AZ/replica/CDN/backup region, encryption/key/IAM, active-passive cost/topology/failover/failback/drills and any active-active ADR consideration.

## Stop Conditions

Stop for unstable Phase 5 restore/incident controls, unknown residency, no writer fencing/failback, read replica as transaction truth, active-active/last-write-wins commerce, untested RTO/RPO, provider callback ambiguity, missing cost/on-call, AWS resource/deployment or legal assumption.

## Rollback / Forward-Fix And Handoff

Planning-only. Future DR rollout leaves primary single-region path intact until standby drills pass. Abort failover on fencing/data/key uncertainty; preserve single writer and recover through incident plan. Failed secondary can be removed without source mutation. Hand service triggers to Packet 13 and risks/ARB to 14.

## Copy-Ready Constrained Prompt

```text
Execute Phase 6 Packet 11 as planning only after prior gates, Packets 01-02 and P6-DR-001. Update only single-region/Multi-AZ/read-replica/CDN/backup-region/active-passive/residency/failover/failback/cost/security/drill design and evidence. Do not provision AWS, write IaC/deployment, introduce active-active/multi-region writes, use replicas as transaction truth, invent residency law, or approve failover without fencing/reconciliation. Explicitly reject premature active-active and add evidence/11-completion.md with continuity/security/cost review.
```
