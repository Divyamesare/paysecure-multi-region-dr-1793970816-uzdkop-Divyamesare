# PaySecure Multi-Region Disaster Recovery Architecture

## 1. Project Overview

PaySecure Gateway Private Limited is a fictional payment technology company
that requires a resilient multi-region disaster recovery architecture.

The project proposes a secure and recoverable payment platform across two
AWS regions located in India.

## 2. Business Context

| Business Metric | Value |
|---|---|
| Daily transactions | 3.2 million |
| Daily transaction value | INR 500 crore |
| Merchants | 45,000 |
| Current architecture | Single AWS region |
| Current availability | 99.92% |
| Target availability | 99.99% |

## 3. Project Objectives

The main objectives are:

1. Design a multi-region disaster recovery architecture.
2. Compare Active-Active and Active-Passive deployment models.
3. Select an appropriate architecture for payment transaction integrity.
4. Define database and messaging replication strategies.
5. Design automated and controlled recovery procedures.
6. Protect payment data and prevent duplicate transactions.
7. Create 12 disaster recovery runbooks.
8. Map architecture controls to applicable regulatory requirements.
9. Develop a lightweight local failure simulation.
10. Prepare technical documentation and a review-board presentation.

## 4. Recovery Objectives

| Metric | Required Target |
|---|---|
| Availability | 99.99% |
| Recovery Point Objective (RPO) | Less than 1 minute |
| Recovery Time Objective (RTO) | Less than 5 minutes |

RPO represents the maximum acceptable data loss measured in time.

RTO represents the maximum acceptable time to restore the service.

These are proposed business targets. Actual achievement must be demonstrated
through appropriate testing and operational evidence.


## 5. Regional Architecture Design

### 5.1 Primary Region – Mumbai

AWS Region: ap-south-1

The Mumbai region is the primary production environment.

Responsibilities:
- Receive normal merchant payment traffic.
- Run the active payment application.
- Process authorised payment writes.
- Maintain primary database and messaging services.
- Publish replication data to the DR region.
- Generate monitoring alerts and operational logs.

### 5.2 Disaster Recovery Region – Hyderabad

AWS Region: ap-south-2

The Hyderabad region is the warm standby environment.

Responsibilities:
- Maintain a ready-to-activate application environment.
- Receive replicated database and messaging data.
- Monitor replication health and regional availability.
- Activate payment processing only after authorised failover.
- Support controlled recovery and failback operations.

### 5.3 Proposed Technology Components

| Layer | Proposed Technology |
|---|---|
| Traffic management | Route 53 / Global Accelerator evaluation |
| Application | Containerised payment application |
| Compute | Amazon ECS evaluation |
| Database | Amazon Aurora PostgreSQL |
| NoSQL database | Amazon DynamoDB |
| Cache | Amazon ElastiCache for Redis |
| Messaging | Amazon MSK / Kafka |
| Infrastructure as Code | Terraform |
| Monitoring | Amazon CloudWatch |
| Security | IAM, KMS, encryption and audit logging |

Service availability, regional support, quotas and costs must be validated
before selecting the final implementation.

### 5.4 High-Level Request Flow

1. Merchant sends a payment request.
2. Global traffic management routes the request to Mumbai.
3. The payment application validates the request.
4. The application checks the idempotency key.
5. The authorised primary database records the transaction.
6. Payment events are published to Kafka.
7. Replication processes transfer supported data to Hyderabad.
8. Monitoring checks application health and replication lag.

### 5.5 Regional Failover Flow

1. Detect a suspected primary-region outage.
2. Verify the outage and assess the impact.
3. Prevent the old primary region from accepting new writes.
4. Check replicated data and recovery readiness.
5. Obtain authorised failover approval.
6. Promote the DR database and activate the standby application.
7. Switch traffic to Hyderabad.
8. Verify payment processing, event handling and reconciliation.

### 5.6 Architecture Safety Principles

- Never allow two independent payment writers during failover.
- Use idempotency keys to protect against duplicate payments.
- Treat the durable payment ledger as the source of truth.
- Monitor replication lag continuously.
- Record every failover and recovery action.
- Perform reconciliation after recovery.

