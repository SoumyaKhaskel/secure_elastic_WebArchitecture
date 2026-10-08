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
