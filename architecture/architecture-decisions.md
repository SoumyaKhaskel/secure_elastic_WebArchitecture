# Architecture Decisions

## 1. Route 53 for DNS

### Decision

Use Amazon Route 53 as the application's DNS layer.

### Reason

The architecture requires a managed DNS entry point that directs users toward the public application delivery layer.

---

## 2. CloudFront for Edge Delivery

### Decision

Use Amazon CloudFront for content delivery.

### Reason

Static content should not require the backend application to process every request.

CloudFront provides caching and edge delivery, reducing repeated requests reaching the origin.

---

## 3. S3 for Static Content

### Decision

Store static application assets in Amazon S3.

### Reason

Static assets such as images, CSS, JavaScript, and other files do not require application-server processing.

Separating these assets from compute reduces unnecessary backend workload.

---

## 4. AWS WAF for Public Application Protection

### Decision

Place AWS WAF in the public application request path.

### Reason

Public-facing APIs require traffic inspection and request controls before reaching application logic.

The design allows rules such as managed protections and rate-based controls to be introduced at the edge/application entry point.

---

## 5. API Gateway for Dynamic Requests

### Decision

Use Amazon API Gateway as the controlled entry point for dynamic APIs.

### Reason

The architecture separates API request handling from application execution.

API Gateway can provide request-management capabilities before invoking backend compute.

---

## 6. Lambda for Application Compute

### Decision

Use AWS Lambda for request-driven application processing.

### Reason

The workload is assumed to be bursty rather than continuously high.

A serverless compute model allows the architecture to respond to changing request volume without manually maintaining a permanently sized application server.

### Trade-off

Lambda is not automatically the correct choice for every application.

Applications requiring long-running processes, specialized runtime environments, persistent local state, or workloads poorly suited to event-driven execution may require a different compute model.

---

## 7. DynamoDB for Application Data

### Decision

Use Amazon DynamoDB for the conceptual data layer.

### Reason

The architecture is intended to demonstrate managed, highly available data storage that can support variable request volumes.

### Important consideration

DynamoDB design depends heavily on access patterns, partition-key selection, item size, consistency requirements, and workload characteristics.

The database design would therefore need to be validated against the actual application before production use.

---

## 8. IAM for Service Access

### Decision

Use AWS IAM roles and policies for service-to-service authorization.

### Reason

The application should not rely on unrestricted credentials.

Access should be granted according to the minimum permissions required by each component.

For example:

```text
Lambda
  ↓
IAM Role
  ↓
Only required DynamoDB actions
```

---

## 9. CloudWatch for Monitoring

### Decision

Use Amazon CloudWatch for logs, metrics, and alarms.

### Reason

A scalable architecture also needs visibility into application health.

Monitoring should cover indicators such as request volume, errors, latency, and relevant service metrics.

---

## 10. Separation of Responsibilities

### Decision

Separate:

```text
DNS
Edge delivery
Security filtering
API handling
Compute
Data storage
Monitoring
```

### Reason

Each layer should have a defined responsibility rather than forcing one server or service to perform unrelated functions.

This improves scalability, maintainability, and the ability to reason about individual failure points.

---

## 11. Keep the Architecture Proportional to the Problem

### Decision

Do not introduce additional AWS services simply to increase architectural complexity.

### Reason

The target environment is a small or growing application with bursty demand and limited engineering resources.

The architecture should solve the scaling problem without creating unnecessary operational complexity.

Additional resilience, security, automation, and disaster-recovery controls can be introduced when justified by the application's requirements.
