# Business Continuity Review Board Presentation

## Board members

1. **Chief Executive Officer** — business continuity commitment.
2. **Chief Financial Officer** — transaction and financial-risk exposure.
3. **Chief Technology Officer** — technical architecture and operational readiness.
4. **Chief Risk and Compliance Officer** — regulatory and control obligations.
5. **Chief Operations Officer** — customer-impact and service restoration.

## 1. Board decision requested

Approve the two-region resilience program and authorize the Q3 2026 implementation plan, subject to successful completion of the pre-production readiness gate.

## 2. Executive scenario

PaySecure processes 3.2 million daily transactions worth 500 crore INR across 45,000 merchants. A single Mumbai-region outage would produce severe merchant and payment-network disruption, financial reconciliation risk, and regulatory exposure. The proposed architecture reduces dependence on one region while keeping a controlled financial ledger model.

## 3. Proposed design

- **Mumbai:** primary region and active payment request writer.
- **Hyderabad:** active recovery region and second active production endpoint.
- **Applications:** active-active across the two regions.
- **Aurora:** primary/standby with controlled failover.
- **DynamoDB:** global table for configuration and non-ledger state.
- **Redis:** active regional cache with failover.
- **Kafka:** multi-region replicated event stream with replay guards.

## 4. Business impact assessment

| Scenario | Financial exposure | Operational exposure | Recovery action |
|---|---|---|---|
| Complete Mumbai outage | High merchant and settlement disruption | 5-minute RTO target for key services | Fail over to Hyderabad |
| Aurora writer outage | High financial integrity risk | Transaction acceptance may pause | Promote consistent standby |
| Kafka region outage | Medium-to-high event backlog | Delayed notification and settlement | Route to surviving region |
| Payment surge | High service degradation | Merchant impact and queue growth | Scale capacity and throttle noncritical work |
| Security compromise | High legal, financial, and reputational exposure | Service isolation and forensic review | Lockdown and restore clean service |

## 5. RTO and RPO commitments

| Service | RPO | RTO |
|---|---:|---:|
| Payment authorization | < 1 minute | < 5 minutes |
| Merchant configuration | < 1 minute | < 5 minutes |
| Settlement processing | < 1 minute | < 30 minutes |
| Kafka processing | < 1 minute | < 15 minutes |
| Notification and webhooks | < 1 minute | < 30 minutes |

## 6. Operational readiness outcome

The target outcome is a measured control environment where:

- The payment API remains available during a regional outage.
- Database failover uses a validated source-of-truth process.
- Merchant and financial states can be reconciled after failover.
- No payment outcome is created twice.
- Recovery is observed, measured, and independently reviewed.

## 7. Regulatory response

The design is mapped to:

- RBI Master Direction on Payment Systems.
- PCI-DSS v4.0.
- NPCI UPI Technical Standards.
- Applicable data-localisation and retention rules.

The implementation must still receive a final legal and compliance review and a production certification decision.

## 8. Risks and treatment

| Risk | Rating | Treatment |
|---|---|---|
| Aurora primary unavailable | Critical | Secondary promotion and validated recovery |
| Ledger split-brain | Critical | Application-level write freeze and reconciliation |
| Kafka replay duplication | High | Idempotency keys and offset reconciliation |
| Regulatory data-location gap | High | Legal and compliance review before deployment |
| Cyberattack on production | High | Isolation, evidence preservation, and clean restore |
| Capacity shortfall | Medium | Baseline, peak, and failover testing |

## 9. Decision points

### Decision A — Approve architecture

Approve the two-region design with the active-passive ledger model and active-active service tier.

### Decision B — Approve implementation funding

Allocate budget for the recovery environment, monitoring, cross-region replication, and runbook testing.

### Decision C — Approve readiness approval gates

Do not declare production readiness until all required tests, compliance gaps, and data-locality decisions are complete.

## 10. Board talking points

- “The architecture protects customer access and payment continuity without allowing two financial ledgers to operate independently.”
- “The proposed design is not simply a backup; it is a tested operational recovery capability.”
- “The RPO and RTO will be measured, not assumed.”
- “Final production certification requires complete compliance evidence and legal review.”
- “The board can approve the program while retaining a strict evidence-based go-live gate.”

## 11. Proposed approval motion

> Approve the two-region resilience program for PaySecure Gateway, authorize the implementation plan, and require evidence of the predefined recovery tests and compliance review before production go-live.
