<!-- sdlc:auto:start adr -->
# ADR-0003: Use PostgreSQL 16 for Database

Status: Accepted  
Date: 2026-11-13

## Context and Problem Statement

Which Database technology does the project use? Decided in architecture v1.

## Considered Options

- PostgreSQL 16
- MySQL 8
- SQLite
- MongoDB

## Decision Outcome

Chosen option: PostgreSQL 16, because One database holds users, sessions, films, likes and the 1024-dimension vectors through pgvector, so recommendations need no second store to keep in sync. The merge rule is a single INSERT ... ON CONFLICT (id) DO UPDATE inside a transaction, and ON DELETE RESTRICT on likes.film_id makes 'never destroy likes' a constraint rather than a convention. Build-versioned film_vectors plus a unique partial index on is_active make the rebuild swap atomic.

## Consequences

Cosine ranking over 50,000 vectors is an HNSW index scan in the same instance that serves page reads, so a heavy rebuild and heavy browsing compete for the same IO — fine at this size, and the fixed 1024-feature cap keeps the index small. Storing two builds briefly roughly doubles vector storage during a rebuild, which is tens of megabytes.
<!-- sdlc:auto:end adr -->