# ADR-001: Multi-Region Disaster Recovery Strategy

## 1. Decision Title

Selection of Active-Passive Warm Standby over Active-Active architecture.

## 2. Status

Proposed – Pending architecture validation.

## 3. Context

PaySecure Gateway processes 3.2 million transactions daily and supports 45,000 merchants.

The existing infrastructure operates in AWS Mumbai (ap-south-1).

The business requires:
- 99.99% availability.
- RPO below 1 minute.
- RTO below 5 minutes.
- Payment transaction integrity.
- Regional disaster recovery.
- Indian data localisation.

Two architectural approaches were evaluated.

## 4. Options Evaluated

### Option A: Active-Active

Both AWS regions simultaneously serve application traffic.

Advantages:
- Both regions process live requests.
- Traffic can be distributed across regions.
- Reduced application recovery dependency on standby activation.

Disadvantages:
- Complex cross-region write consistency.
- Conflict resolution requirements.
- Risk of duplicate payment processing.
- More complicated transaction ordering.
- Increased operational complexity.

### Option B: Active-Passive Warm Standby

Mumbai operates as the primary production region.

Hyderabad maintains replicated data and pre-provisioned application infrastructure.

During a confirmed regional disaster:
1. Detect the regional failure.
2. Fence the failed primary to prevent conflicting writes.
3. Verify replication status and recovery-point integrity.
4. Promote the secondary database.
5. Restore application dependencies.
6. Validate payment processing.
7. Redirect production traffic.

Advantages:
- Clear primary write ownership.
- Easier transaction consistency management.
- Controlled regional promotion.
- Simpler operational recovery procedures.

Disadvantages:
- Requires a controlled failover process.
- Replication lag can cause data loss.
- Standby capacity must be maintained.
- Failback requires careful reconciliation.

## 5. Decision

Select Active-Passive Warm Standby as the proposed DR architecture.

Primary Region:
AWS Mumbai – ap-south-1

Secondary Region:
AWS Hyderabad – ap-south-2

The application will use a single-writer strategy for payment transaction processing.

The architecture will target:
- RPO below 60 seconds.
- RTO below 300 seconds.
- Availability of 99.99%.

These targets are design objectives, not proven service guarantees.

## 6. Data Replication Strategy

### Aurora PostgreSQL
Use Aurora Global Database with cross-region replication.

Promote the secondary cluster during disaster recovery.

### DynamoDB
Evaluate Global Tables with multi-region consistency settings and application-level conflict controls.

### ElastiCache Redis
Use cross-region replication where supported.

Treat Redis as recoverable cache state, not the authoritative payment ledger.

### Apache Kafka
Use cross-region replication with independent regional clusters.

Use event IDs, idempotent consumers and offset reconciliation.

## 7. Payment Safety Controls

- Idempotency keys for payment requests.
- Unique transaction identifiers.
- Single-writer ownership.
- Primary-region fencing.
- Transaction status reconciliation.
- Duplicate event detection.
- Controlled failback.
- Auditable recovery procedures.

## 8. Risks and Mitigations

| Risk | Mitigation |
|---|---|
| Replication lag | Continuous lag monitoring |
| Split-brain | Fencing and single-writer control |
| Duplicate transactions | Idempotency and deduplication |
| Kafka event loss | Replication and reconciliation |
| Cache inconsistency | Cache rebuild and invalidation |
| Failed promotion | Recovery checkpoints and rollback plan |

## 9. Consequences

The selected architecture prioritises transaction integrity and operational simplicity.

The design still requires workload testing, capacity validation, failure simulation and regulatory review before production approval.

## 10. Validation Requirements

- Verify replication lag under peak load.
- Simulate primary-region failure.
- Verify database promotion.
- Validate application health.
- Test duplicate payment prevention.
- Measure actual RPO and RTO.
- Test controlled failback.

## 11. Approval

Proposed by: DevOps & Cloud Engineering

Project: PaySecure Gateway Multi-Region DR

Decision: Active-Passive Warm Standby

Status: Pending Business Continuity Review Board approval.

## 12. Architecture Evaluation Matrix

The following qualitative comparison evaluates both deployment models
against PaySecure's payment-processing requirements.

| Evaluation Criteria | Active-Active | Active-Passive Warm Standby |
|---|---|---|
| Traffic handling | Both regions serve traffic | Primary serves normal traffic |
| Write ownership | Requires multi-region coordination | Single authorised writer |
| Transaction consistency | More complex | Easier to coordinate |
| Conflict resolution | Required for concurrent writes | Reduced through single-writer design |
| Failover process | Traffic and write coordination | Controlled promotion and traffic switch |
| Operational complexity | High | Moderate |
| Infrastructure cost | Generally higher | Depends on standby capacity |
| Failback | Requires coordinated recovery | Requires controlled reconciliation |
| Payment safety | Requires strong distributed controls | Supports centralised write ownership |

## 13. Decision Rationale

Active-Passive Warm Standby is selected as the proposed architecture
because PaySecure prioritises payment transaction integrity,
controlled recovery and clear write ownership.

The selection does not imply that Active-Passive automatically
guarantees the required RPO, RTO or availability.

The design must be validated through:
- Peak-load testing.
- Replication-lag measurement.
- Regional failure simulation.
- Database promotion testing.
- Split-brain prevention testing.
- Payment reconciliation.
- Controlled failback testing.

## 14. Decision Review Conditions

This architecture decision must be reviewed if:

1. Business requirements change.
2. Active-Active becomes necessary for workload availability.
3. Regional service capabilities change.
4. Replication performance cannot support the required RPO.
5. Recovery testing cannot demonstrate the required RTO.
6. Regulatory or data-localisation requirements change.

Decision Status: Proposed – Pending Technical and Business Approval.

