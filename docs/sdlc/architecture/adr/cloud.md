<!-- sdlc:auto:start adr -->
# ADR-0004: Use AWS for Cloud

Status: Accepted  
Date: 2026-11-13

## Context and Problem Statement

Which Cloud technology does the project use? Decided in architecture v1.

## Considered Options

- AWS
- Google Cloud
- Azure
- Fly.io

## Decision Outcome

Chosen option: AWS, because One account carries the whole topology: RDS PostgreSQL 16 with pgvector and automated backups, one Fargate task for the API and its in-process rebuild, a second small Fargate task for Meilisearch, S3 for the admin's CSVs behind the approved AWS keys, and CloudFront for the SPA origin. The ALB and CloudFront hosts are what make the two-origin, bearer-token design concrete.

## Consequences

Single region with no failover (NFR-019, accepted) — a regional outage is a full outage. Fargate plus RDS plus a second always-on container is more baseline cost than a single VM, bought in exchange for managed backups and no server patching on a free product.
<!-- sdlc:auto:end adr -->