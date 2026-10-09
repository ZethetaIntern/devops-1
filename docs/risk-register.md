# Risk register and acceptance criteria

## 1. Key risk categories

| ID | Risk | Likelihood | Impact | Owner | Mitigation | Acceptance criteria |
|---|---|---:|---:|---|---|---|
| R-01 | Complete Mumbai regional outage | Medium | Critical | CTO | Active-active application tier and Aurora cross-region failover | Recovery test completed and API functionality restored within 5 minutes |
| R-02 | Aurora data divergence | Low | Critical | Database Lead | Prevent writes to standby, verify transaction sequence, reconcile ledger | No unexplained financial transaction difference after failover |
| R-03 | Split-brain database writes | Low | Critical | Database Lead | Lock write access and use a single authoritative ledger | Dual approval required before writes resume |
| R-04 | Kafka partition or leader failure | Medium | High | Messaging Lead | Replicator, broker redundancy, controlled replay | Consumer lag and transaction outcomes remain within target |
| R-05 | Kafka duplicate financial event | Medium | High | Integration Lead | Idempotency keys and deduplication store | Each financial event is processed exactly once |
| R-06 | Redis stale session or rate-limit state | Medium | Medium | Application Lead | Cache invalidation and session re-registration | No transaction accepted from stale session state |
| R-07 | DynamoDB local configuration drift | Medium | High | Platform Lead | Global table validation and versioned configuration | Transaction routing uses the auditable configuration version |
| R-08 | Traffic surge during failover | Medium | High | Operations Lead | Capacity guardrails, queue backpressure, and merchant backoff | Throughput stays within agreed capacity limits |
| R-09 | DNS or certificate failure | Medium | High | Network Lead | Global endpoint failover and certificate automation | API endpoints remain valid and reachable |
| R-10 | Cybersecurity compromise | Low | Critical | CISO | Segmentation, IAM, cryptography, evidence preservation | Incident contained without unauthorized payment data use |
| R-11 | Data-localisation review gap | Medium | Critical | Compliance Officer | Define geographic scope and data classification | All replicated data has documented legal approval |
| R-12 | Payment network contract restriction | Medium | High | Legal and Partnerships | Review partner terms before cross-region replication | No unapproved data transfer or unsupported processing |
| R-13 | Unproven RTO and RPO | Medium | High | Operations Lead | Formal test plan and quarterly validation | 3 successful measurement runs for each critical service |
| R-14 | Regulatory control evidence not complete | Medium | High | Compliance Officer | Maintain and review control evidence | Audit evidence available and independently reviewed |

## 2. Acceptance criteria

The architecture is accepted when:

1. The platform has an active-active application tier across Mumbai and Hyderabad.
2. Aurora has a documented single-writer failover process and a tested promotion path.
3. Payment transactions use durable, idempotent keys across all processing stages.
4. Kafka event replay is proven to produce one financial outcome per transaction key.
5. Replication lag is measured and maintained below the approved threshold.
6. RPO remains under one minute and RTO under five minutes for payment authorization.
7. No critical or high-severity security findings remain unresolved.
8. Required data-localisation and payment-network approvals are documented.
9. All 12 runbooks have been tested at least once and have documented evidence.
10. A board-approved operational readiness sign-off is retained.

## 3. Risk treatment priorities

1. Prevent an unsupported Aurora split-brain.
2. Protect transaction idempotency across Kafka replay and downstream processing.
3. Prove the recovery path under production-like load.
4. Complete legal and data-localisation review before data is replicated.
5. Preserve evidence and control ownership for RBI, PCI, NPCI, and auditor review.

## 4. Outstanding decisions

- Whether the environment will use an AWS-native failover model or an external managed-failover service for the database.
- Whether payment-network data can be replicated to Hyderabad under existing contracts.
- Whether an additional third-region recovery architecture is required for a wider catastrophe scenario.
- Whether monthly or quarterly failover exercises will be production-scale by default.
- What exact capacity and traffic-weighting policy is approved for Mumbai and Hyderabad.
