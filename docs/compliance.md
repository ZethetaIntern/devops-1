# Compliance and regulatory mapping

## 1. Compliance position

This mapping is a design-level control alignment. It is not a legal determination or a substitute for a regulator review. Final design, controls, and evidence must be validated by a qualified legal, audit, security, and payment-network advisor before production certification.

## 2. RBI Master Direction on Payment Systems

| Design area | RBI-related requirement alignment | Architectural control |
|---|---|---|
| Operational resilience | Payment systems should maintain continuity and recover from disruptions | Two-region platform, automated failover, tested recovery runbooks |
| Security | Strong authentication, access controls, and protection of payment data | IAM, MFA, KMS, WAF, Secrets Manager, least privilege |
| Risk management | Identification, mitigation, and monitoring of operational risk | Risk register, monitoring, incident response, testing |
| Data confidentiality | Protection of payment data and related records | Encryption, segmentation, access logging |
| System availability | Continuity planning and service restoration | Active-active APIs, active-passive ledger, health checks |
| Audit trail | Evidence of transactions, actions, and security events | Immutable log stores, transaction IDs, blockchain-like audit semantics |
| Incident response | Detection, escalation, learning, and remediation | Incident command, defined runbooks, post-incident reports |
| Outsourcing and third parties | Appropriate oversight of cloud and support providers | Contract review, control mapping, and service provider obligations |
| Information security | Protection against unauthorized access, misuse, or data loss | Security monitoring, key management, network separation |

### RBI-specific obligations for the design

- Maintain transaction and settlement records required by law and banking rules.
- Preserve evidence of merchant onboarding, authorization, reconciliation, and dispute resolution.
- Maintain the role of authorized personnel and separate operations responsibilities.
- Demonstrate that the recovery procedure does not differ from the approved risk posture.
- Ensure that data residency and data movement are documented and approved.
- Retain the ability to produce a complete transaction timeline for investigation.

## 3. PCI-DSS v4.0 alignment

| PCI-DSS v4.0 domain | Requirement alignment | Control |
|---|---|---|
| Protect system components and cardholder data | Restrict access and protect stored data | Network segmentation, encryption, KMS, IAM |
| Strong cryptography | Encrypt data in transit and at rest | TLS, AWS KMS, customer-managed keys |
| Secure access | Strong authentication and least privilege | MFA, IAM identity center, service roles |
| Monitor and test networks | Detect and respond to security events | GuardDuty, Security Hub, SIEM, audit logs |
| Maintain service and security documentation | Evidence of design and control operation | Architecture records, runbooks, test evidence |
| Incident response | Contain and investigate incidents | Incident command, evidence preservation, formal review |
| Change control | Validate changes before production | IaC, pull requests, approval gates, rollback |
| Vulnerability management | Identify and mitigate weaknesses | Patch management, vulnerability scans, security testing |

### Payment-card data handling note

The design assumes that card data is tokenized or vaulted in a PCI-compliant service. Full PAN data is not retained in the application or analytics stores unless required and approved by the compliance function. The architecture must document the exact cardholder-data flow and network boundaries before launch.

## 4. NPCI UPI Technical Standards

| Design area | Control implication |
|---|---|
| UPI transaction integrity | Unique transaction and idempotency keys | Global transaction identifiers and replay-safe event processing |
| Availability | Reliable transactions even under regional disruption | Active-active API layer and regional failover |
| Observability | Detect delays, rejections, and duplicate messages | Structured logging, event metrics, reconciliation dashboards |
| Security | API and message authentication | Mutual TLS, signed tokens, independent security monitoring |
| Data consistency | Correct payment outcome and settlement state | Aurora ledger, Kafka event ordering, reconciliation controls |
| Network and system suitability | Regional routing and service continuity | Global traffic routing and cross-region readiness |

## 5. India data localisation requirements

India’s payment and data rules require careful review because requirements can depend on the exact payment service, data type, and applicable regulator. The design documents the following:

- Data intended for cross-region processing is retained within the geographic jurisdiction approved by the regulator.
- Cross-region replication is limited to explicitly approved data classes.
- Merchant, payment, and user data are separated by account and data classification labels.
- Production data is replicated only where the necessary contractual and regulatory approvals exist.
- Backup, archival, and analytics data are retained under approved location and access rules.
- A data-residency register includes the system owner, data class, location, retention period, and legal basis.

## 6. Control evidence required for audit

The following evidence must be maintained:

- Architecture review records and approvals.
- Network and data-flow diagrams.
- IAM and key-management policies.
- Encryption and certificate inventory.
- Infrastructure-as-code version history.
- Change approvals and rollback records.
- Recovery test plans and results.
- Kafka, Aurora, Redis, and DynamoDB health records.
- Incident records and post-incident actions.
- Vendor and payment-network contractual reviews.
- Data-retention and deletion records.

## 7. Risk and control gaps

| Gap | Risk | Required action |
|---|---|---|
| Aurora active-active write model not implemented | Data conflict or split-brain | Use controlled primary failover and idempotency |
| Kafka replay not strictly verified | Duplicate settlement or missing processing | Use transaction IDs and reconciliation offsets |
| Cross-region legal approval missing | Regulatory noncompliance | Complete legal and regulatory review before go-live |
| Card data not tokenized | PCI-DSS exposure | Store only tokenized card data and protect identifiers |
| Merchant network contracts not reviewed | Data-transfer or service restrictions | Review both upstream and downstream partner terms |
| No measured failover evidence | RTO/RPO not demonstrated | Run quarterly production-scale tests |

## 8. Readiness gate before production

The project cannot move to production without:

1. Legal confirmation of data localisation and payment-network constraints.
2. Approved data classification and retention schedule.
3. Complete PCI-DSS scope and card-data flow analysis.
4. A signed attack-surface and security architecture review.
5. Demonstrated RTO and RPO in a production-like environment.
6. Successful recovery runs for at least all 12 runbooks.
7. Independent audit or QSA review of critical systems.
8. A documented control ownership matrix.
