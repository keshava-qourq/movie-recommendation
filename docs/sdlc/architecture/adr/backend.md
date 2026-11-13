<!-- sdlc:auto:start adr -->
# ADR-0002: Use FastAPI 0.115 for Backend

Status: Accepted  
Date: 2026-11-13

## Context and Problem Statement

Which Backend technology does the project use? Decided in architecture v1.

## Considered Options

- FastAPI 0.115
- Django REST Framework
- Flask
- Node.js/Express

## Decision Outcome

Chosen option: FastAPI 0.115, because Keeping the API, the CSV importer and the TF-IDF/cosine recommender in one Python process is what lets the rebuild run inside the upload request now that the queue is out of scope: import_service calls vectorizer directly, with no serialisation boundary. FastAPI's dependency injection carries current_user and require_admin onto exactly the endpoints that need them, and its UploadFile streaming handles the multipart CSV.

## Consequences

A long rebuild occupies a worker thread in the same process that serves visitors, so the task is sized with headroom and the rebuild runs off the event loop in a thread pool; at the far end of the 50,000-film range the admin waits minutes (NFR-018, accepted). Scaling out later means ensuring only one task runs a rebuild at a time, enforced by a Postgres advisory lock.
<!-- sdlc:auto:end adr -->