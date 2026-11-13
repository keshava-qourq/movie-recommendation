# Sprint Plan

> Maintained by QuorQ AI SDLC. Text between `sdlc:auto` markers is regenerated; anything outside them is yours and is kept.

<!-- sdlc:auto:start sprints -->
## Sprint 1: Sprint 1 - Accounts and catalogue foundation

Users can sign up and sign in with hashed passwords and server-side sessions, a seeded admin exists, and an admin can upload a CSV that merges into the catalogue by film id without touching likes.

Capacity: 20 points · Committed: 20

| Key | Type | Title | Points | Role | Depends on |
|---|---|---|---|---|---|
| US-011 | Story | Sign up with email and password | 5 | backend-agent | — |
| US-012 | Story | Sign in and sign out securely | 5 | backend-agent | US-011 |
| US-014 | Story | Upload a catalogue CSV that merges by movie ID | 8 | backend-agent | US-012, US-017 |
| ↳ US-014-T1 | Task | Add films, catalog_imports and catalog_import_rows schema with like foreign keys | — | backend-agent | — |
| ↳ US-014-T2 | Task | Implement S3 upload and streaming CSV parse | — | backend-agent | — |
| ↳ US-014-T3 | Task | Implement transactional merge-by-id upsert | — | backend-agent | — |
| ↳ US-014-T4 | Task | Add admin-only upload endpoint and upload page | — | backend-agent | — |
| US-017 | Story | Sign in as the seeded first admin | 2 | devops-agent | US-011 |

## Sprint 2: Sprint 2 - Import reporting and similarity vectors

Uploads report per-row outcomes, and every upload rebuilds weighted film soups into capped TF-IDF vectors stored in pgvector without taking the site down.

Capacity: 20 points · Committed: 18

| Key | Type | Title | Points | Role | Depends on |
|---|---|---|---|---|---|
| US-005 | Story | Build the film text "soup" with weighted features | 5 | backend-agent | US-014 |
| US-015 | Story | See a per-row validation summary for an upload | 5 | backend-agent | US-014 |
| US-016 | Story | Rebuild similarity vectors after each upload | 8 | backend-agent | US-005, US-014 |
| ↳ US-016-T1 | Task | Add vector_builds and versioned film_vectors schema | — | backend-agent | — |
| ↳ US-016-T2 | Task | Implement the full-corpus rebuild routine | — | backend-agent | — |
| ↳ US-016-T3 | Task | Wire the rebuild into upload completion with atomic activation | — | backend-agent | — |
| ↳ US-016-T4 | Task | Add rebuild resilience test suite | — | qa-agent | — |

## Sprint 3: Sprint 3 - Public search, film pages and similar films

Any anonymous visitor can search with typo tolerance, see suggestions as they type, open a film detail page and get its 12 most similar films from stored vectors.

Capacity: 20 points · Committed: 19

| Key | Type | Title | Points | Role | Depends on |
|---|---|---|---|---|---|
| US-001 | Story | Search for a film by title | 5 | backend-agent | US-014 |
| US-002 | Story | See autocomplete suggestions while typing | 3 | frontend-agent | US-001 |
| US-003 | Story | View a film's detail page | 3 | frontend-agent | US-014 |
| US-004 | Story | See films similar to the one I am looking at | 5 | backend-agent | US-003, US-005, US-016 |
| US-006 | Story | Serve recommendations from precomputed vectors | 3 | backend-agent | US-005, US-016 |

## Sprint 4: Sprint 4 - Likes and the For you list

Signed-in viewers can like films anywhere they appear, manage their liked list, and get a personalised top-20 list, with clear sign-in prompts for anonymous visitors.

Capacity: 20 points · Committed: 16

| Key | Type | Title | Points | Role | Depends on |
|---|---|---|---|---|---|
| US-007 | Story | Like a film | 5 | frontend-agent | US-012, US-003, US-001 |
| US-008 | Story | See and unlike my liked films | 3 | frontend-agent | US-007 |
| US-009 | Story | Get a personalised "For you" list | 5 | backend-agent | US-007, US-006 |
| US-010 | Story | Be prompted to like films when I have none | 1 | frontend-agent | US-009 |
| US-020 | Story | Browse without an account and without paying | 2 | frontend-agent | US-004, US-007 |

## Sprint 5: Sprint 5 - Privacy, responsiveness and documentation

Users can delete their account and all likes, the whole app works on phones and desktops, and admins/operators have the CSV format guide and setup notes.

Capacity: 20 points · Committed: 8

| Key | Type | Title | Points | Role | Depends on |
|---|---|---|---|---|---|
| US-013 | Story | Delete my account and all my likes | 3 | backend-agent | US-012, US-008 |
| US-018 | Story | Follow the CSV format guide and setup notes | 2 | devops-agent | US-015, US-017 |
| US-019 | Story | Use WatchNext on a phone or a desktop | 3 | frontend-agent | US-003, US-008, US-009 |

## Unscheduled

_Nothing unscheduled._
<!-- sdlc:auto:end sprints -->
