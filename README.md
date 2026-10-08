# Check the current files
find diagrams/phase-1 -maxdepth 1 -type f -print

# Rename the diagrams to clean filenames
mv "diagrams/phase-1/Poor Architecture_ Scaling Failure Infographic.png" \
   "diagrams/phase-1/01-poor-architecture.png"

# Replace the next two source names below with the exact filenames shown by the find command
# Example:
mv "diagrams/phase-1/<YOUR-PROPOSED-ARCHITECTURE-FILENAME>.png" \
   "diagrams/phase-1/02-proposed-architecture.png"

mv "diagrams/phase-1/<YOUR-DATA-FLOW-FILENAME>.png" \
   "diagrams/phase-1/03-data-flow-diagram.png"

# Replace README with the corrected version
cat > README.md <<'EOF'
# Secure Elastic Web Architecture

> Designing a secure and scalable AWS architecture for public-facing applications experiencing sudden traffic spikes.

## Overview

Many small web applications work well under normal traffic but struggle when traffic suddenly increases because their architecture was designed around average demand rather than burst demand.

This project explores how a small application can evolve from a single-server architecture into a more elastic and security-focused AWS architecture.

---

## The Problem

A typical small application may begin with:

- A single compute instance
- Application and database on the same machine
- Static files served directly by the application
- Limited traffic protection
- Limited monitoring
- Fixed infrastructure capacity

This architecture can work for normal traffic but becomes a bottleneck during sudden traffic spikes.

### Example failure scenario

![Poor Architecture](diagrams/phase-1/01-poor-architecture.png)

---

## Proposed Architecture

The proposed architecture separates DNS, edge delivery, security filtering, application processing, and data storage.

![Proposed AWS Architecture](diagrams/phase-1/02-proposed-architecture.png)

---

## Request and Data Flow

The following diagram shows how a user request moves through the proposed architecture.

![Data Flow Diagram](diagrams/phase-1/03-data-flow-diagram.png)

---

## Core Architecture

```text
User
  ↓
Route 53
  ↓
CloudFront
  ├── Static Content → S3
  │
  └── Dynamic Request → WAF
                         ↓
                     API Gateway
                         ↓
                       Lambda
                         ↓
                     DynamoDB
