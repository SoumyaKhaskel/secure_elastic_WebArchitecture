# Secure Elastic Web Architecture

> Designing a secure and scalable AWS architecture for public-facing applications experiencing sudden traffic spikes.

---

## Overview

Many small web applications work perfectly under normal traffic but struggle when traffic suddenly increases.

The problem often begins with a simple architecture designed around average traffic:

- A single compute instance
- Application and database hosted together
- Static files served from the application server
- Fixed infrastructure capacity
- Limited traffic protection
- Limited monitoring

This design may be sufficient for a small workload, but a sudden traffic spike can turn the same architecture into a bottleneck.

This project explores how a small or growing public-facing application can evolve into a more elastic, secure, and observable AWS architecture.

> **Note:** This is a conceptual architecture and system-design exercise for learning, experimentation, and portfolio demonstration. It does not represent the private production architecture of Amazon, Flipkart, or any other organization.

---

# The Problem

Consider a small public-facing application that normally handles relatively low traffic.

For most of the year, everything works normally.

Then a flash sale, product launch, viral campaign, registration event, or other sudden demand increase brings a large number of users to the application.

The workload changes quickly, but the infrastructure does not.

### Typical failure pattern

```text
Traffic Spike
     ↓
More Concurrent Requests
     ↓
CPU / Memory / Connection Pressure
     ↓
Higher Latency
     ↓
Timeouts and Failed Requests
     ↓
Application Degradation
     ↓
Possible Service Unavailability
```

![Poor Architecture - Scaling Failure Infographic](diagrams/phase-1/Poor%20Architecture_%20Scaling%20Failure%20Infographic.png)

---

# Proposed AWS Architecture

The proposed solution separates the application's major responsibilities into dedicated layers so that traffic, static content, application processing, and data storage are no longer dependent on a single server.

## Architecture Overview

```text
Users
  ↓
Amazon Route 53
  ↓
Amazon CloudFront
  ├── Static Content → Amazon S3
  │
  └── Dynamic Requests → AWS WAF
                           ↓
                       API Gateway
                           ↓
                         Lambda
                           ↓
                       DynamoDB
```

![Scalable Secure AWS Architecture](diagrams/phase-1/Scalable%20Secure%20AWS%20Architecture.png)

## Core Components

### 1. Route 53 — DNS Layer

Amazon Route 53 provides DNS resolution for the application's public domain and directs users toward the application delivery layer.

### 2. CloudFront — Edge and Content Delivery

Amazon CloudFront acts as the edge delivery layer.

Static content can be cached at CloudFront edge locations, reducing repeated requests reaching the backend infrastructure and improving response latency for users.

### 3. S3 — Static Content Storage

Amazon S3 stores static application assets such as:

- HTML
- CSS
- JavaScript
- Images
- Other static files

This separates static content delivery from application processing.

### 4. AWS WAF — Web Application Protection

AWS WAF provides a security control point for incoming application traffic.

It can be used for:

- Web request filtering
- Managed security rules
- Rate-based controls
- Application-layer protection

### 5. API Gateway — Dynamic Request Entry Point

Amazon API Gateway handles dynamic API requests before they reach the application logic.

It provides a controlled API entry point and can apply request-management controls such as throttling and validation.

### 6. AWS Lambda — Elastic Compute

AWS Lambda executes the application's backend logic on demand.

Instead of maintaining one permanently sized application server, compute capacity can respond to changing request volume.

This is particularly suitable for the bursty workload considered in this architecture.

### 7. DynamoDB — Managed Data Layer

Amazon DynamoDB provides the application's managed data store.

The database is separated from the compute layer rather than running on the same machine as the application.

The application accesses DynamoDB through controlled IAM permissions.

### 8. IAM — Access Control

AWS IAM controls service-to-service access using roles and policies.

The intended model follows least privilege.

For example:

```text
AWS Lambda
     ↓
IAM Role
     ↓
Required DynamoDB permissions only
```

The database is not directly exposed to users.

### 9. CloudWatch — Monitoring and Observability

Amazon CloudWatch provides centralized visibility into the architecture through:

- Application logs
- Service metrics
- Request counts
- Error rates
- Latency
- Alarms
- Dashboards

---

# Request and Data Flow

The following diagram shows how requests move through the proposed serverless AWS architecture.

![Serverless AWS Request Flow Architecture](diagrams/phase-1/Serverless%20AWS%20Request%20Flow%20Architecture.png)

## Static Content Request

When a user requests static content such as an image, CSS file, or JavaScript file:

```text
User
  ↓
Route 53
  ↓
CloudFront
  ↓
CloudFront Cache
  ↓
User
```

When the object is not available in the CloudFront cache:

```text
User
  ↓
Route 53
  ↓
CloudFront
  ↓
S3
  ↓
CloudFront
  ↓
User
```

This keeps static-content traffic away from the application compute layer.

---

## Dynamic Request

For a request that requires application logic:

```text
User
  ↓
Route 53
  ↓
CloudFront
  ↓
AWS WAF
  ↓
API Gateway
  ↓
AWS Lambda
  ↓
DynamoDB
  ↓
AWS Lambda
  ↓
API Gateway
  ↓
CloudFront
  ↓
User
```

The application logic executes inside Lambda and retrieves or updates the required data in DynamoDB.

---

# How This Addresses the Scaling Problem

The original architecture concentrated multiple workloads on one fixed-capacity machine.

The proposed architecture separates those workloads:

```text
                    Public Traffic
                         ↓
                    CloudFront
                    ↙         ↘
          Static Content     Dynamic Requests
               ↓                  ↓
               S3                WAF
                                  ↓
                             API Gateway
                                  ↓
                               Lambda
                                  ↓
                             DynamoDB
```

This allows the architecture to handle traffic more efficiently because:

- Static content can be served from the edge.
- Dynamic requests are separated from static delivery.
- Application compute is event-driven.
- Data storage is managed independently from compute.
- Public traffic passes through a dedicated security layer.
- Service access is controlled using IAM.
- System activity can be observed through CloudWatch.

---

# What Changes During a Traffic Spike?

The key architectural change is that the application is no longer dependent on one permanently sized server.

Instead:

```text
Normal Traffic
     ↓
Lower Request Volume
     ↓
CloudFront serves cached content
     ↓
Lambda handles required application requests
     ↓
DynamoDB handles application data
```

During a burst:

```text
Traffic Spike
     ↓
More Requests
     ↓
CloudFront absorbs cacheable traffic
     ↓
WAF filters incoming requests
     ↓
API Gateway handles dynamic requests
     ↓
Lambda scales with workload
     ↓
DynamoDB handles the data workload
```

The objective is to distribute the workload across purpose-built managed services rather than allowing one server to become the primary bottleneck.

---

# Initial Security Model

Security is introduced as part of the architecture rather than treated as a separate afterthought.

### Public Layer

```text
Internet
   ↓
CloudFront
   ↓
AWS WAF
```

### Application Access

```text
API Gateway
     ↓
Lambda
     ↓
IAM Role
     ↓
DynamoDB
```

The database should not be directly accessible from the public internet.

### Monitoring

```text
AWS Services
     ↓
CloudWatch
     ├── Logs
     ├── Metrics
     ├── Alarms
     └── Dashboard
```

---

# Phase 1 Scope

This first phase focuses on the fundamental scaling architecture:

- Identifying the single-server bottleneck
- Separating static and dynamic workloads
- Introducing edge delivery
- Introducing protected API access
- Moving application execution to serverless compute
- Separating compute from data storage
- Introducing IAM-based service access
- Adding centralized monitoring

Phase 2 will extend this architecture with deeper work around **security hardening, resilience, failure handling, traffic protection, vulnerability management, secure CI/CD, backup and recovery, and cost optimization**.

---

## Important Note

This architecture is a conceptual design for a small or growing public-facing application experiencing bursty traffic.

It is not intended to reproduce the private architecture of Amazon, Flipkart, or any other large-scale organization.

The purpose is to demonstrate how system-design decisions can be used to address scalability while incorporating security and operational considerations from the beginning.
