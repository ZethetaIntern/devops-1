# Disaster recovery runbooks

## Runbook 1: Complete Mumbai region failure

**Purpose:** Restore payment operations in Hyderabad while protecting ledger integrity.

**Trigger:** Loss of control plane, network path, power, or complete service degradation in Mumbai for more than the activation threshold.

**Owner:** Incident Commander, Database Lead, Network Lead.

**Actions:**

1. Declare a regional incident and open an incident record.
2. Confirm Hyderabad is healthy and its dependencies are ready.
3. Freeze write access to nonessential Mumbai workloads.
4. Confirm Aurora replication is valid and no new payments are being accepted in Mumbai.
5. Promote Hyderabad Aurora to primary after review.
6. Update production API routing to Hyderabad.
7. Promote Redis and Kafka consumers.
8. Validate merchant configuration, payment routing, and settlement services.
9. Re-enable payments in controlled batches.
10. Monitor reconciliation, authorization success, and queue lag.

**Exit criteria:**

- 99.99% regional services available.
- Successful transaction and settlement reconciliation.
- No unexplained ledger difference.
- Incident record and post-incident review complete.

**Rollback:** Reduce traffic back to Mumbai only after the region is independently validated and a controlled return plan is approved.

---

## Runbook 2: Hyderabad region failure

**Purpose:** Continue production processing in Mumbai without changing the financial ledger.

**Trigger:** Loss of availability or integrity in Hyderabad that affects API, Kafka, Redis, or Aurora replication.

**Actions:**

1. Detect and isolate Hyderabad dependency failure.
2. Route payment traffic to Mumbai.
3. Validate payment services and database limits in Mumbai.
4. Pause use of Hyderabad-only configuration.
5. Resume normal processing with no database promotion.
6. Restore the Hyderabad environment after inspection.

**Exit criteria:** Mumbai is operating normally and Hyderabad has no active impact on transactions.

---

## Runbook 3: Aurora PostgreSQL writer failure

**Purpose:** Promote the regional reader and restore transaction processing.

**Trigger:** Writer process failure, storage degradation, or loss of control-plane access.

**Actions:**

1. Confirm the issue is not an application-level connection problem.
2. Freeze all nonessential writes.
3. Validate that the failover candidate has the latest common transaction state.
4. Promote the candidate to the primary role.
5. Update all service endpoints and database endpoints.
6. Validate application transactions and schema compatibility.
7. Resume payment processing in batches.

**Exit criteria:** All services access the promoted writer and database reconciliation is complete.

**Safety:** The application must never write directly to the old primary or a replica during promotion.

---

## Runbook 4: Database split-brain

**Purpose:** Prevent two Aurora clusters from accepting conflicting financial transactions.

**Trigger:** Evidence that both regions can accept writes or the system has inconsistent transaction sequences.

**Actions:**

1. Declare a critical data-integrity incident.
2. Stop all application writes in both regions.
3. Remove write permissions from the recovery and primary database roles.
4. Compare the most recent transaction IDs and ledger totals.
5. Identify the last valid common transaction state.
6. Choose one authoritative cluster.
7. Restore the non-authoritative cluster from a validated snapshot or point-in-time rollback.
8. Reconstruct the missing transaction set with transaction IDs and event logs.
9. Validate ledger totals and merchant balances.
10. Re-enable writes only after dual approval.

**Exit criteria:** A single authoritative ledger exists and all financial transactions are reconciled.

**Critical rule:** Never merge two independent ledger states without an approved reconciliation strategy.

---

## Runbook 5: Kafka region failure

**Purpose:** Continue event processing from the surviving cluster.

**Trigger:** Loss of Kafka leadership, broker availability, or partition availability in one region.

**Actions:**

1. Determine whether the failing cluster is a complete outage or a partition.
2. Pause producers in the failing region.
3. Redirect consumers to the peer region.
4. Confirm MSK Replicator is healthy and event ordering is intact.
5. Rebuild missing consumer-group offsets from the last verified offset.
6. Resume consumers in batches.
7. Validate downstream events and reconciliation state.

**Exit criteria:** All critical consumers have recovered and no uncompensated events remain.

---

## Runbook 6: Kafka partition loss

**Purpose:** Recover a partition without duplicate settlement events.

**Trigger:** Partition unavailable, leader loss, or message loss warning.

**Actions:**

1. Suspend producers for the affected topic.
2. Identify the partition and key ranges.
3. Check broker health, leader election, and replication status.
4. If safe, trigger controlled leader recovery.
5. If not safe, create a replacement partition or replay from a peer cluster.
6. Reconcile event offsets with the last safely processed payment ID.
7. Resume processing with idempotency checks.
8. Verify each financial transaction has exactly one outcome.

**Exit criteria:** No partition remains without a valid leader or replicated records.

---

## Runbook 7: Kafka consumer group lag

**Purpose:** Restore processing throughput and control backlog.

**Trigger:** Consumer lag exceeds the defined threshold for more than 5 minutes.

**Actions:**

1. Check worker capacity and queue depth.
2. Pause noncritical consumers.
3. Scale the consumer group in the surviving region.
4. Validate consumer offset commits.
5. Check for duplicate processing or dead-letter events.
6. Rebalance partitions and resume consumers.
7. Validate settlements and notifications.

**Outcome:** Event backlog is controlled and transaction outcomes remain idempotent.

---

## Runbook 8: Redis primary failure

**Purpose:** Promote the Redis replica and regenerate stale session state.

**Trigger:** Redis primary unavailable or data corruption detected.

**Actions:**

1. Confirm the primary is not recoverable.
2. Promote the replica.
3. Update service configuration and connection endpoints.
4. Invalidate volatile cache entries.
5. Force services to re-register sessions.
6. Validate authentication and rate-limit state.
7. Reconcile any temporary cache-dependent workflows.

**Exit criteria:** Redis is operating with valid data and authenticated sessions are accepted.

---

## Runbook 9: DynamoDB replication lag

**Purpose:** Prevent stale merchant or routing configuration from affecting transactions.

**Trigger:** DynamoDB replication lag exceeds the policy threshold.

**Actions:**

1. Pause use of the affected regional configuration table.
2. Read from the primary region for transaction decisions.
3. Confirm the configuration version and checksum.
4. Check for incorrect sequence numbers or expired metadata.
5. Restore or re-publish the known-good configuration.
6. Resume traffic using the approved version.
7. Validate merchant and payment routing.

**Exit criteria:** Global table data is consistent and no transaction is routed using stale configuration.

---

## Runbook 10: Peak-load failover

**Purpose:** Maintain payment processing during a load surge and region failover.

**Trigger:** User traffic exceeds the design capacity or fails over while the system is operating above 80% CPU or memory.

**Actions:**

1. Trigger regional capacity protection.
2. Stop expensive nonessential jobs and scheduled tasks.
3. Increase API and worker capacity.
4. Shift traffic toward the healthy region.
5. Adjust Kafka and consumer scaling.
6. Rebalance Redis and database connections.
7. Temporarily reduce merchant-side retry load.
8. Monitor authorization success, queue backlog, and latency.
9. Re-enable normal operations after stability is confirmed.

**Exit criteria:** The system remains within capacity and no payment is failed because of service saturation.

---

## Runbook 11: DNS and network routing failure

**Purpose:** Restore user access without exposing the application to unsafe routing.

**Trigger:** DNS outage, certificate failure, or network path degradation.

**Actions:**

1. Validate the DNS provider and certificate status.
2. Use a validated alternate regional endpoint.
3. Update traffic routing to the healthy region.
4. Check regional network ACLs and security groups.
5. Reissue certificates only through approved certificate automation.
6. Validate API responses and merchant traffic.
7. Restore original routing after all health checks pass.

**Exit criteria:** API traffic is healthy, certificates are valid, and no unauthorized endpoint is active.

---

## Runbook 12: Security compromise and data exposure

**Purpose:** Contain a compromise and preserve evidence while continuing approved payment services.

**Trigger:** Security alert, unauthorized credential use, unusual API traffic, or suspected compromise.

**Actions:**

1. Freeze affected access and rotate the impacted credentials.
2. Isolate the compromised workload or security group.
3. Preserve logs, replicas, audit records, and API traces.
4. Notify the incident commander and compliance owner.
5. Assess whether data was accessed or modified.
6. Restrict cross-region replication while preserving forensic evidence.
7. Restore clean infrastructure from immutable images.
8. Re-enable services only after security review.
9. Notify regulators, payment partners, and affected parties where required.
10. Complete a post-incident root-cause analysis.

**Exit criteria:** The compromised path is closed, evidence is preserved, and the system is cleared by security and compliance owners.

## Runbook quality requirements

Each runbook must be tested:

- At least once per quarter.
- At least once per year under simulated production volume.
- During a change-window exercise.
- With a documented success condition and rollback path.
- With captured evidence for audit and operating review.
