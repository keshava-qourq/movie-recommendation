# Planning

> Maintained by QuorQ AI SDLC. Text between `sdlc:auto` markers is regenerated; anything outside them is yours and is kept.

<!-- sdlc:auto:start narrative -->
### EPIC-001 · approved · source: planning:EPIC-001

Movie search and film details. Any visitor, signed in or not, can find a film in the catalogue by typing its title and open its detail page. Covers three stories worth 11 points, split between backend (search) and frontend (suggestions, detail page).

Goal: a visitor — signed in or not — can locate a film by title and open its detail page.

Assumptions carried forward from discovery (NOT yet validated):
- Search is title-based keyword/substring matching with typo tolerance and autocomplete suggestions.
- No browse-by-genre or filter pages in this version.
- Film metadata comes solely from the admin-uploaded CSV; no live integration with an external movie database, so posters appear only where the CSV supplied a poster_url.

Plan allocation for this epic (from plan v1 items): US-001 5 pts / backend-agent; US-002 3 pts / frontend-agent; US-003 3 pts / frontend-agent. Epic item itself carries no points and no assigned role.

Story and acceptance-criterion text is attached to the plan document verbatim and is not restated here.

**Related:** US-001, US-002, US-003

### EPIC-002 · approved · source: planning:EPIC-002

Similar-movie recommendations. Each film's detail page shows a ranked list of the films most like it. Three backend stories worth 13 points cover the ranked list, the weighted text soup and serving from precomputed vectors.

Goal: every film detail page carries a ranked list of the most similar films.

Assumptions carried forward from discovery (NOT yet validated):
- Similarity is content-based: each film becomes one combined text "soup" of plot + genres + director + cast, with genres, director and cast weighted above plot.
- The soup is vectorised with TF-IDF; films are compared by cosine similarity.
- Vectors are precomputed and stored in pgvector after each catalogue upload, never computed per request.
- No custom ML model and no accuracy target — quality judged subjectively in this version; no A/B testing and no analytics instrumentation.

Plan allocation (plan v1): US-004 5 pts, US-005 5 pts, US-006 3 pts, all role backend-agent. Epic item carries no points and no assigned role.

Note: plan task US-016-T2/T3 (under EPIC-005) describe the rebuild routine that produces these vectors, so this epic's delivery depends on EPIC-005 work. Plan item wording also refers to "capped" TF-IDF vectors, a detail not present in the epic assumptions.

Story and acceptance-criterion text is attached verbatim to the plan document and is not restated here.

**Related:** US-004, US-005, US-006

### EPIC-003 · approved · source: planning:EPIC-003

Likes and the "For you" list. Signed-in viewers like films and receive a personalised ranked list built from everything they have liked. Four stories worth 14 points, mixed frontend and backend.

Goal: signed-in viewers mark the films they enjoyed and get a personalised ranked list derived from all their likes.

Assumptions carried forward from discovery (NOT yet validated):
- "For you" is the mean of the vectors of all the user's liked films, ranked by cosine similarity, excluding films already liked, returning a default top 20.
- Liking is available from search results and similar-film lists as well as from a film's detail page.
- No dislike / not-interested marking and no watchlist in this version.

Plan allocation (plan v1): US-007 5 pts / frontend-agent; US-008 3 pts / frontend-agent; US-009 5 pts / backend-agent; US-010 1 pt / frontend-agent. Epic item carries no points and no assigned role.

Story and acceptance-criterion text is attached verbatim to the plan document and is not restated here.

**Related:** US-007, US-008, US-009, US-010

### EPIC-004 · approved · source: planning:EPIC-004

Accounts, sign-in and personal data. Open self-serve email/password signup so likes attach to an account, plus deletion of the account and all its data. Three backend stories worth 13 points. Carries an unresolved password-reset conflict.

Goal: open self-serve signup with email and password so likes are saved to an account, with the ability to delete the account and all its data.

Assumptions carried forward from discovery (NOT yet validated):
- No social login and no email-verification gate on first use.
- Personal data limited to email, password hash and like history.
- No advertising, no profiling for third parties, no data sharing; generic GDPR-style handling because no region was named.

OPEN CONFLICT to resolve (recorded as stated in the input, both sides kept):
- The confirmed understanding assumed password reset by emailed link.
- The chosen delivery scope removes the email provider from the stack.
- Resolution drafted: no self-serve password reset in this version; a user who loses their password cannot recover it unaided. The input states this gap should be closed before or shortly after launch. No story in this plan covers it.

Plan allocation (plan v1): US-011 5 pts, US-012 5 pts, US-013 3 pts, all role backend-agent. Epic item carries no points and no assigned role.

Story and acceptance-criterion text is attached verbatim to the plan document and is not restated here.

**Related:** US-011, US-012, US-013

### EPIC-005 · approved · source: planning:EPIC-005

Catalogue administration via CSV. The admin-uploaded CSV is the single source of truth for film data; uploads merge by movie ID, never destroy user likes, and trigger a rebuild of the similarity vectors. Largest epic at 25 points across five stories plus eight tasks.

Goal: admin CSV upload is the single source of truth for film data; uploads merge by movie ID, preserve user likes, and trigger a similarity-vector rebuild.

Assumptions carried forward from discovery (NOT yet validated):
- Expected CSV columns: id, title, year, plot, genres, director, cast, and optional poster_url.
- Import is error-tolerant: bad rows are rejected and reported rather than aborting the file.

OPEN CONFLICT to resolve (both sides recorded as stated):
- The confirmed understanding assumed the vector rebuild would run asynchronously on a queue.
- The chosen delivery scope removes the queue from the stack.
- Resolution drafted: the rebuild runs as part of the upload operation, with progress visible to the admin. Plan task US-016-T3 reflects this ("wire the rebuild into upload completion with atomic activation").

Plan allocation (plan v1), 25 story points total:
- US-014 8 pts / backend-agent, broken into tasks US-014-T1 (films, catalog_imports and catalog_import_rows schema with like foreign keys), US-014-T2 (S3 upload and streaming CSV parse), US-014-T3 (transactional merge-by-id upsert), US-014-T4 (admin-only upload endpoint and upload page) — all backend-agent, no points.
- US-015 5 pts / backend-agent.
- US-016 8 pts / backend-agent, broken into tasks US-016-T1 (vector_builds and versioned film_vectors schema), US-016-T2 (full-corpus rebuild routine), US-016-T3 (wire rebuild into upload completion with atomic activation) — backend-agent — and US-016-T4 (rebuild resilience test suite) — qa-agent; no points on tasks.
- US-017 2 pts / devops-agent.
- US-018 2 pts / devops-agent.
Epic item carries no points and no assigned role.

Note: task detail introduces implementation specifics not present in the epic assumptions — S3 for upload storage, streaming parse, versioned film_vectors with atomic activation.

Story and acceptance-criterion text is attached verbatim to the plan document and is not restated here.

**Related:** US-014, US-015, US-016, US-017, US-018

### EPIC-006 · approved · source: planning:EPIC-006

Public, free access on any device. WatchNext is open to anyone on the web, works in mobile and desktop browsers, and is entirely free. Two frontend stories worth 5 points.

Goal: the app is publicly accessible, responsive on mobile and desktop, and free of charge.

Assumptions carried forward from discovery (NOT yet validated):
- Responsive web app hosted in a single cloud region; no native iOS or Android apps.
- No payments, subscriptions, plans or billing of any kind, so no billing capability is drafted anywhere in the document.

Plan allocation (plan v1): US-019 3 pts / frontend-agent; US-020 2 pts / frontend-agent. Epic item carries no points and no assigned role.

Story and acceptance-criterion text is attached verbatim to the plan document and is not restated here.

**Related:** US-019, US-020

### SPRINT-1 · draft · source: plan:SPRINT-1

Sprint 1 — Accounts and catalogue foundation. Capacity 20 points. Delivers signup/sign-in with hashed passwords and server-side sessions, a seeded admin account, and an admin CSV upload that merges into the catalogue by film id without touching likes.

Sprint name: "Sprint 1 - Accounts and catalogue foundation". Capacity: 20.

Goal as stated in plan v1: "Users can sign up and sign in with hashed passwords and server-side sessions, a seeded admin exists, and an admin can upload a CSV that merges into the catalogue by film id without touching likes."

Scope linkage: the plan input lists backlog items as a flat list with no explicit sprint assignment. The related_ids above are inferred from the sprint goal wording (accounts -> US-011/US-012, seeded admin -> US-017, merge-by-id upload -> US-014 and its tasks US-014-T1..T4) and should be confirmed against the plan owner before being treated as a committed sprint backlog.

Plan status at time of documentation: generated, version 1 — not recorded as approved.

**Related:** EPIC-004, EPIC-005, US-011, US-012, US-014, US-017

### SPRINT-2 · draft · source: plan:SPRINT-2

Sprint 2 — Import reporting and similarity vectors. Capacity 20 points. Uploads report per-row outcomes, and every upload rebuilds weighted film soups into capped TF-IDF vectors stored in pgvector without taking the site down.

Sprint name: "Sprint 2 - Import reporting and similarity vectors". Capacity: 20.

Goal as stated in plan v1: "Uploads report per-row outcomes, and every upload rebuilds weighted film soups into capped TF-IDF vectors stored in pgvector without taking the site down."

Scope linkage: inferred from goal wording — per-row outcome reporting -> US-015; rebuild on every upload with no downtime -> US-016 and its tasks US-016-T1..T4; weighted film soups -> US-005 (listed under EPIC-002). No explicit sprint assignment exists in the plan input; confirm with the plan owner.

Dependency noted in the goal: the rebuild runs inline with the upload (see the EPIC-005 queue conflict), and the site must stay usable while it runs.

Plan status at time of documentation: generated, version 1 — not recorded as approved.

**Related:** EPIC-002, EPIC-005, US-005, US-015, US-016

### SPRINT-3 · draft · source: plan:SPRINT-3

Sprint 3 — Public search, film pages and similar films. Capacity 20 points. Anonymous visitors can search with typo tolerance, see suggestions as they type, open a film detail page and get its 12 most similar films from stored vectors.

Sprint name: "Sprint 3 - Public search, film pages and similar films". Capacity: 20.

Goal as stated in plan v1: "Any anonymous visitor can search with typo tolerance, see suggestions as they type, open a film detail page and get its 12 most similar films from stored vectors."

Scope linkage: inferred from goal wording — typo-tolerant search -> US-001; type-ahead suggestions -> US-002; film detail page -> US-003; 12 similar films -> US-004; served from stored vectors -> US-006. No explicit sprint assignment exists in the plan input; confirm with the plan owner.

Dependency: relies on vectors produced by the Sprint 2 rebuild work.

Plan status at time of documentation: generated, version 1 — not recorded as approved.

**Related:** EPIC-001, EPIC-002, US-001, US-002, US-003, US-004, US-006

### SPRINT-4 · draft · source: plan:SPRINT-4

Sprint 4 — Likes and the For you list. Capacity 20 points. Signed-in viewers can like films anywhere they appear, manage their liked list, and get a personalised top-20 list, with clear sign-in prompts for anonymous visitors.

Sprint name: "Sprint 4 - Likes and the For you list". Capacity: 20.

Goal as stated in plan v1: "Signed-in viewers can like films anywhere they appear, manage their liked list, and get a personalised top-20 list, with clear sign-in prompts for anonymous visitors."

Scope linkage: inferred from goal wording — liking from any surface -> US-007; managing the liked list -> US-008; personalised top 20 -> US-009; empty-state prompt -> US-010; sign-in prompts for anonymous visitors -> US-020 (listed under EPIC-006). No explicit sprint assignment exists in the plan input; confirm with the plan owner.

Dependency: requires accounts/sessions from Sprint 1 and stored vectors from Sprint 2.

Plan status at time of documentation: generated, version 1 — not recorded as approved.

**Related:** EPIC-003, EPIC-006, US-007, US-008, US-009, US-010, US-020

### SPRINT-5 · draft · source: plan:SPRINT-5

Sprint 5 — Privacy, responsiveness and documentation. Capacity 20 points. Users can delete their account and all likes, the app works on phones and desktops, and admins/operators have the CSV format guide and setup notes.

Sprint name: "Sprint 5 - Privacy, responsiveness and documentation". Capacity: 20.

Goal as stated in plan v1: "Users can delete their account and all likes, the whole app works on phones and desktops, and admins/operators have the CSV format guide and setup notes."

Scope linkage: inferred from goal wording — account and like deletion -> US-013; mobile and desktop usability -> US-019; CSV format guide and setup notes -> US-018. No explicit sprint assignment exists in the plan input; confirm with the plan owner.

Open item for this sprint window: the EPIC-004 conflict leaves no self-serve password reset in scope anywhere in plan v1, and the input states the gap should be closed before or shortly after launch.

Plan status at time of documentation: generated, version 1 — not recorded as approved. Total committed story points across the backlog are 81 against 100 points of sprint capacity.

**Related:** EPIC-004, EPIC-005, EPIC-006, US-013, US-018, US-019
<!-- sdlc:auto:end narrative -->
