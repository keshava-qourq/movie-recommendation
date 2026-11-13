# Traceability Matrix

> Maintained by QuorQ AI SDLC. Text between `sdlc:auto` markers is regenerated; anything outside them is yours and is kept.

<!-- sdlc:auto:start legend -->
Built from recorded links, not written by a model: requirement key → plan item (`source_stable_key`) → development ticket (`stable_key`) → the ticket's test steps → the last live deployment that started after the work finished. **AC tests** counts the story's acceptance criteria that have a test case, and those whose test cases all passed in the latest finished E2E run.
<!-- sdlc:auto:end legend -->

<!-- sdlc:auto:start matrix -->
| Requirement | Epic | Title | Plan items | Sprint | Design | Code | Commits | Ticket | Tests | AC tests | Deployed |
|---|---|---|---|---|---|---|---|---|---|---|---|
| US-001 | EPIC-001 | Search for a film by title | US-001 | 3 | GET /films/search, api, db, films, search_index, web_app | — | — | not started | not run | 0/4 tested · 0/4 passing | — |
| US-002 | EPIC-001 | See autocomplete suggestions while typing | US-002 | 3 | GET /films/suggest, api, films, search_index, web_app | — | — | not started | not run | 0/3 tested · 0/3 passing | — |
| US-003 | EPIC-001 | View a film's detail page | US-003 | 3 | GET /films/{film_id}, api, db, films, web_app | — | — | not started | not run | 0/5 tested · 0/5 passing | — |
| US-004 | EPIC-002 | See films similar to the one I am looking at | US-004 | 3 | GET /films/{film_id}/similar, api, db, film_vectors, films, recommender, web_app | — | — | not started | not run | 0/5 tested · 0/5 passing | — |
| US-005 | EPIC-002 | Build the film text "soup" with weighted features | US-005 | 2 | db, film_vectors, films, recommender | — | — | not started | not run | 0/3 tested · 0/3 passing | — |
| US-006 | EPIC-002 | Serve recommendations from precomputed vectors | US-006 | 3 | GET /films/{film_id}/similar, GET /me/recommendations, api, db, film_vectors, recommender | — | — | not started | not run | 0/3 tested · 0/3 passing | — |
| US-007 | EPIC-003 | Like a film | US-007 | 4 | PUT /me/likes/{film_id}, api, db, films, likes, users, web_app | — | — | not started | not run | 0/5 tested · 0/5 passing | — |
| US-008 | EPIC-003 | See and unlike my liked films | US-008 | 4 | DELETE /me/likes/{film_id}, GET /me/likes, api, db, films, likes, web_app | — | — | not started | not run | 0/4 tested · 0/4 passing | — |
| US-009 | EPIC-003 | Get a personalised "For you" list | US-009 | 4 | GET /me/recommendations, api, db, film_vectors, films, likes, recommender, web_app | — | — | not started | not run | 0/5 tested · 0/5 passing | — |
| US-010 | EPIC-003 | Be prompted to like films when I have none | US-010 | 4 | GET /me/recommendations, likes, web_app | — | — | not started | not run | 0/2 tested · 0/2 passing | — |
| US-011 | EPIC-004 | Sign up with email and password | US-011 | 1 | POST /auth/signup, api, db, sessions, users, web_app | — | — | not started | not run | 0/5 tested · 0/5 passing | — |
| US-012 | EPIC-004 | Sign in and sign out securely | US-012 | 1 | GET /me, POST /auth/login, POST /auth/logout, api, db, sessions, users, web_app | — | — | not started | not run | 0/5 tested · 0/5 passing | — |
| US-013 | EPIC-004 | Delete my account and all my likes | US-013 | 5 | DELETE /me, api, db, likes, sessions, users, web_app | — | — | not started | not run | 0/5 tested · 0/5 passing | — |
| US-014 | EPIC-005 | Upload a catalogue CSV that merges by movie ID | US-014, US-014-T1, US-014-T2, US-014-T3, US-014-T4 | 1 | GET /admin/catalog/uploads/{import_id}, POST /admin/catalog/uploads, api, catalog_import_rows, catalog_importer, catalog_imports, db, films, likes, object_store, web_app | — | — | not started | not run | 0/5 tested · 0/5 passing | — |
| US-015 | EPIC-005 | See a per-row validation summary for an upload | US-015 | 2 | GET /admin/catalog/uploads, GET /admin/catalog/uploads/{import_id}, api, catalog_import_rows, catalog_importer, catalog_imports, db, web_app | — | — | not started | not run | 0/5 tested · 0/5 passing | — |
| US-016 | EPIC-005 | Rebuild similarity vectors after each upload | US-016, US-016-T1, US-016-T2, US-016-T3, US-016-T4 | 2 | GET /admin/catalog/uploads/{import_id}, api, catalog_importer, db, film_vectors, films, recommender, vector_builds | — | — | not started | not run | 0/5 tested · 0/5 passing | — |
| US-017 | EPIC-005 | Sign in as the seeded first admin | US-017 | 1 | api, db, users | — | — | not started | not run | 0/4 tested · 0/4 passing | — |
| US-018 | EPIC-005 | Follow the CSV format guide and setup notes | US-018 | 5 | catalog_importer, films | — | — | not started | not run | 0/4 tested · 0/4 passing | — |
| US-019 | EPIC-006 | Use WatchNext on a phone or a desktop | US-019 | 5 | web_app | — | — | not started | not run | 0/3 tested · 0/3 passing | — |
| US-020 | EPIC-006 | Browse without an account and without paying | US-020 | 4 | GET /films/search, GET /films/{film_id}, GET /films/{film_id}/similar, api, sessions, web_app | — | — | not started | not run | 0/3 tested · 0/3 passing | — |
<!-- sdlc:auto:end matrix -->
