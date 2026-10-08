# Phase 1 — Problem and Architecture

## Overview

This project explores how a small public-facing web application can be redesigned to handle sudden and unpredictable traffic spikes.

The problem is common in applications that operate normally throughout most of the year but experience short periods of unusually high demand, such as flash sales, viral campaigns, product launches, registrations, ticket releases, or promotional events.

Large-scale platforms are designed for this type of workload from the beginning. Smaller applications often start with a much simpler architecture and only discover its limitations when traffic increases unexpectedly.

This project uses that scenario as the problem to solve.

> **Note:** The architecture presented in this project is a conceptual AWS design intended for learning, experimentation, and portfolio demonstration. It is not a representation of any private Amazon, Flipkart, or other production architecture.

---

## The Problem

Consider a small web application hosted on a single compute instance.

Under normal traffic, the application may work without any noticeable problems.

A sudden increase in visitors changes the workload dramatically.

For example:

```text
Normal traffic
     ↓
Low and predictable workload
     ↓
Single server performs normally

Traffic spike
     ↓
Large number of simultaneous requests
     ↓
CPU / memory / connection pressure
     ↓
Higher response latency
     ↓
Timeouts and failed requests
     ↓
Application becomes unstable or unavailable
```

The problem is not necessarily the application code itself.

The architecture is designed around average demand rather than burst demand.

---

## Example of the Initial Architecture

The initial design assumes that the application, database, and static content are hosted together on one server.

![Poor Architecture](../diagrams/phase-1/01-poor-architecture.png)

This architecture can be valid for a small application with predictable traffic.

Its weakness appears when demand increases significantly.

### Main limitations

- Single point of failure
- Fixed compute capacity
- Application and database compete for resources
- Static content consumes application-server resources
- Limited ability to absorb traffic spikes
- Limited fault tolerance
- Limited traffic protection
- Monitoring and response capabilities may be insufficient

---

## Design Goal

The objective is not to recreate the infrastructure of a large e-commerce platform.

The objective is to design a practical architecture that allows a smaller application to:

- Handle sudden traffic increases
- Scale compute capacity automatically
- Reduce unnecessary load on the application layer
- Deliver static content efficiently
- Protect public-facing endpoints
- Separate application processing from data storage
- Monitor system health and performance
- Keep infrastructure costs aligned with actual usage

---

## Proposed AWS Architecture

The proposed design separates the major responsibilities of the application into dedicated layers.

![Proposed Architecture](../diagrams/phase-1/02-proposed-architecture.png)

### High-level flow

```text
User
  ↓
Route 53
  ↓
CloudFront
  ├── Static content → S3
  │
  └── Dynamic requests → WAF → API Gateway
                                  ↓
                                Lambda
                                  ↓
                              DynamoDB
```

Supporting services provide identity, monitoring, logging, and access control.

---

## Layered Architecture

### 1. DNS and Edge Layer

**Amazon Route 53**

Provides DNS resolution for the public application domain.

**Amazon CloudFront**

Provides edge delivery and caching for content that can be served without reaching the application logic.

This reduces unnecessary traffic reaching the backend.

---

### 2. Static Content Layer

**Amazon S3**

Stores static assets such as:

- HTML
- CSS
- JavaScript
- Images
- Other static files

CloudFront can deliver these assets from its cache or retrieve them from S3 when required.

This prevents the application compute layer from serving every static object directly.

---

### 3. Security Layer

**AWS WAF**

Protects the public application path with controls such as:

- Managed web security rules
- Request filtering
- Rate-based rules
- Application-layer traffic controls

The WAF is positioned before the dynamic application processing layer.

---

### 4. API and Compute Layer

**Amazon API Gateway**

Handles dynamic API requests and provides a controlled entry point for backend operations.

**AWS Lambda**

Executes application logic on demand.

The serverless model removes the need to maintain a permanently sized application server for the expected peak workload.

---

### 5. Data Layer

**Amazon DynamoDB**

Provides managed, highly available data storage suitable for workloads designed around key-value or document access patterns.

The application interacts with the database through controlled service permissions rather than exposing the database directly to users.

---

### 6. Identity and Monitoring

**AWS IAM**

Controls which AWS services and identities can perform specific actions.

The design follows least-privilege principles wherever practical.

**Amazon CloudWatch**

Collects logs and metrics and can generate alarms for conditions such as:

- Increased error rates
- High latency
- Application failures
- Resource or request anomalies

---

## Request and Data Flow

The request flow is separated into static and dynamic paths.

![Data Flow Diagram](../diagrams/phase-1/03-data-flow-diagram.png)

### Static request

```text
User
  ↓
Route 53
  ↓
CloudFront
  ↓
CloudFront Cache
  ↓
Response to User
```

If the requested object is not cached, CloudFront can retrieve it from S3.

### Dynamic request

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
Lambda
  ↓
DynamoDB
  ↓
Lambda
  ↓
API Gateway
  ↓
CloudFront
  ↓
User
```

This separation allows static delivery and dynamic application processing to scale independently.

---

## Why This Architecture Addresses the Original Problem

The original architecture concentrates multiple responsibilities on one machine.

The proposed design separates those responsibilities.

Instead of:

```text
One server
├── Application
├── Database
└── Static content
```

the architecture becomes:

```text
Edge delivery
     ↓
Security filtering
     ↓
API processing
     ↓
Serverless compute
     ↓
Managed data storage
```

This removes the dependency on a single permanently sized application server and provides a foundation for handling bursty demand more effectively.

---

## Scope of Phase 1

Phase 1 focuses on the architecture and the scaling problem.

It establishes:

- The problem
- The initial failure architecture
- The proposed AWS architecture
- The request and data flow
- The major architectural components
- The initial security boundaries

Advanced security hardening, detailed failure handling, cost analysis, CI/CD security, vulnerability management, backup strategy, and additional resilience mechanisms will be addressed in Phase 2.

---

## Implementation Status

This repository currently documents a **proposed architecture and design exercise**.

The diagrams and architecture decisions should not be interpreted as evidence of a production deployment.

The goal is to demonstrate system-design reasoning, cloud architecture understanding, security considerations, and engineering trade-off analysis.
