# Product Requirements

> Maintained by QuorQ AI SDLC. Text between `sdlc:auto` markers is regenerated; anything outside them is yours and is kept.

<!-- sdlc:auto:start narrative -->
## Summary

### REQ-STACK-001 · approved · source: requirements:technology-stack

Product owner approved a fixed four-part stack: FastAPI backend, React frontend, PostgreSQL 16 database, AWS single-region hosting. Substitutions are not permitted. Requirements version 1, delivery scope 'lightweight'.

- Backend: FastAPI 0.115 — recommendation maths (text 'soup' per film, TF-IDF vectorising of 5,000–50,000 films after each CSV upload, cosine ranking) is Python work; scikit-learn and pandas do it in a few lines. CSV importer, vectoriser and web API live in one Python codebase, avoiding a second language boundary.
- Frontend: React 19 — one responsive web app for anonymous browsers and signed-in viewers: search box with live autocomplete, film pages with ranked similar-film grids, like buttons that toggle instantly in search results, similar lists and detail pages. One reusable like-button component instead of three copies. Also the platform default.
- Database: PostgreSQL 16 — small, highly relational data (users, likes, films, one precomputed vector per film). Transactional upserts and foreign keys guarantee the merge rule 'update existing by ID, add new, never destroy likes', so a half-failed import cannot break the catalogue or likes. Also hosts similarity vectors via pgvector.
- Cloud: AWS — single-region deployment assumed; one account covers managed PostgreSQL with automated backups, a container for the API and vector-rebuild worker, S3 for admin CSV uploads, and a CDN for fast film pages. Scales from a handful to thousands of users without re-architecting; costs stay modest as the app is free and serves no video or images of its own.
- Design against exactly these; do not substitute.

**Related:** REQ-SVC-001, REQ-SCOPE-001

### REQ-SVC-001 · approved · source: requirements:services

Three runtime services were approved and must all appear in runtime needs: pgvector inside PostgreSQL, Meilisearch for title search, and Amazon S3 for uploaded CSV files. Four AWS API keys are required.

- Vector store: pgvector (in PostgreSQL), known — one precomputed vector per film plus cosine ranking for both 'similar movies' and 'For you' live inside the existing PostgreSQL; no extra database to pay for or keep in sync. TF-IDF vectors must be capped to a fixed width (~1,000–1,500 features) to stay indexable; normal practice and no quality loss at this catalogue size.
- Search: Meilisearch, container getmeili/meilisearch:v1.10.3 — typo-tolerant title search and type-ahead suggestions out of the box for a 50,000-film catalogue; plain database matching would miss 'inceptoin' and feel sluggish as live autocomplete.
- File storage: Amazon S3 (SaaS) — the admin's uploaded CSV is parked in storage for the background importer to read; the original file plus its import report stay available for checking or re-running an upload. Env: AWS_ACCESS_KEY_ID, AWS_SECRET_ACCESS_KEY, AWS_REGION, S3_BUCKET.
- API keys needed by the app: AWS_ACCESS_KEY_ID, AWS_REGION, AWS_SECRET_ACCESS_KEY, S3_BUCKET.

**Related:** REQ-STACK-001, NFR-002, NFR-005

### REQ-SCOPE-001 · approved · source: requirements:scope

Scope is fixed for version 1: search, film details, similar films, likes, a 'For you' list, accounts, and admin CSV catalogue upload. A sizeable exclusion list keeps external metadata, watchlists, native apps and several infrastructure services out.

IN SCOPE (capabilities):
- Movie title search; film detail page with plot, year, genres, director and cast.
- Similar-movies list ranked by cosine similarity; default 12 similar films, excluding the film itself.
- Like a film from its detail page; My liked films list with unlike.
- 'For you' personalised list from all liked films; default top 20, excluding films already liked; prompt to like some films when a viewer has no likes yet.
- Signup and sign-in with email and password; hashed passwords with server-side sessions; delete my account and all my likes; first admin account seeded at deployment.
- Catalogue CSV upload merging by movie ID; CSV columns id, title, year, plot, genres, director, cast, poster_url; posters shown only where the CSV supplies a poster URL.
- Similarity vector rebuild after each catalogue upload; film text 'soup' weighting genres, director and cast above plot; TF-IDF vectors precomputed in pgvector, not computed per request.
- Responsive layout for desktop and mobile browsers; CSV format guide and setup notes.
- Free to use — no payments, subscriptions or billing.

OUT OF SCOPE (do not design or build):
- Live integration with an external movie database (TMDb/IMDb) for posters and metadata.
- Watchlist / save-to-watch-later, browse-by-genre pages, dislike/not-interested marking.
- Native iOS/Android apps, user reviews/ratings/comments, social following or sharing feeds, streaming-availability ('where to watch') data, trailers or video playback, training a custom ML model.
- Queue (suggested Redis-as-a-queue removed from the stack).
- Cache (suggested Redis removed from the stack).
- Email (suggested SendGrid removed from the stack).

**Related:** REQ-STACK-001, REQ-SVC-001

### REQ-FEAT-001 · approved · source: requirements:functional:Sign-in & accounts

Self-serve email/password accounts with two roles. Anonymous visitors keep full browsing access; only liking, the personalised list and liked films require an account. Users can delete everything the service holds about them. No billing anywhere.

- Create an account: sign up with email plus a password meeting the stated minimum length. Refused with a helpful message if the email is already in use; field-level messages for malformed email or too-short password. New accounts work straight away, with no email verification step.
- Sign in and sign out: registered users sign in with email and password and stay signed in while using the app, with their likes available. Wrong password gives a generic failure message. Signing out ends the session immediately.
- Delete my account and likes: available from account settings after an explicit confirmation step; removes the account record, email address and every like. User returns to the public site as an anonymous visitor; the film catalogue is untouched; the same email can later create a brand-new account with no previous likes.
- See what data is held: account settings states plainly that only email address, password hash and like history are stored.
- Viewer and admin roles: every account created through the public signup form is a viewer. The admin role (which unlocks catalogue upload) can only be granted directly in the database; no self-serve request.
- First admin account at setup: deployment creates exactly one admin account from configured seed credentials so a catalogue can be uploaded before any other user exists. Re-running deployment against an environment that already has the seeded admin creates no duplicate and leaves the existing account intact.
- Browse without an account: anonymous visitors can search, open film detail pages and view similar-film lists with no sign-in prompt.
- Free with no billing: no payment, subscription, plan choice, price or billing detail is ever requested or displayed — including during signup, liking, the personalised list and account deletion.

**Related:** NFR-008, NFR-009, NFR-010, NFR-011, NFR-012, NFR-013, NFR-016, NFR-017

### REQ-FEAT-002 · approved · source: requirements:functional:Finding films

Any visitor can find a film by title. Matching is partial and typo-tolerant, and suggestions appear once at least two characters are typed.

- Search films by title: any visitor types a title into the search box and sees matching catalogue films with title and year. No matches produces a friendly empty-result message inviting another title, not an error.
- Forgiving title matching: matches on part of a title as well as the whole title ('godfath' finds 'The Godfather'); tolerates typing mistakes ('interstelar' still finds 'Interstellar').
- Suggestions while typing: after at least two characters, suggested film titles with their years appear beneath the search box. Choosing a suggestion opens that film's detail page directly. Fewer than two characters shows no suggestions.

**Related:** REQ-SVC-001, NFR-002

### REQ-FEAT-003 · approved · source: requirements:functional:Film page & similar films

The film detail page shows catalogue metadata and a ranked list of the 12 most similar films, served from precomputed similarity data rather than calculated per request. Missing posters and empty optional fields degrade gracefully.

- Film detail page: shows title, year, plot, genres, director and cast on one page, taken entirely from the admin-uploaded catalogue.
- Posters and missing details: poster image shown when the catalogue row supplied a poster address; otherwise a placeholder, never a broken image. Empty optional fields such as director or cast are omitted or marked unavailable, and the rest of the page still renders.
- Film not found page: requesting a film not in the catalogue shows a clear 'film not found' page with a link back to search.
- Similar films list: 12 films most like the current one, ordered most to least similar, never including the film itself. Fewer than 12 other prepared films lists all available ones without error. Selecting any listed film opens its own page with its own similar list. A film not yet prepared still shows its details, with a note that recommendations are not available for it yet.
- Weighted film text: each film's similarity text combines plot, genres, director and cast into one block, with genres, director and cast counting for more than plot wording. A film with an empty plot still gets a usable similarity text from its remaining fields and appears in similar-film results.
- Recommendations from stored results: similar-film lists and the personalised list are produced from similarity data prepared after the last catalogue upload, never recalculated across the catalogue while a visitor waits.

**Related:** REQ-SVC-001, NFR-001, NFR-003, NFR-005

### REQ-FEAT-004 · approved · source: requirements:functional:Likes & For you

Signed-in viewers like films from anywhere they appear, manage those likes in one list, and get a 20-film personalised list built from everything they liked. Anonymous visitors are invited to sign in instead.

- Like a film anywhere: a signed-in viewer can like a film from its detail page, from search results and from a similar-films list. The control switches to the liked state immediately and the like saves without leaving the page. Likes survive signing out and back in. Submitting a like twice for the same film still leaves exactly one like, with no error.
- Sign-in prompt for personal features: an anonymous visitor trying to like a film, open the personalised list or open their liked films is invited to sign in or sign up. No like is recorded and no personalised list is shown until they do.
- My liked films and unlike: a signed-in viewer sees every liked film with title and year and can remove any like from that list. An unliked film stops influencing the personalised list and becomes eligible to be recommended again. With no likes, an empty-state message invites them to search for films they enjoyed.
- Personalised For you list: a viewer with at least one like sees 20 films ranked by closeness to the combined profile of everything they liked, excluding films already liked. The list updates to reflect each new like. If fewer than 20 unliked prepared films exist, all available ones are listed without error.
- Prompt when there are no likes: a signed-in viewer with no likes sees a prompt explaining how to get recommendations, with a route into search, instead of an empty page. After liking a film, the prompt is replaced by a populated list.

**Related:** REQ-FEAT-001, REQ-FEAT-003, NFR-011, NFR-015

### REQ-FEAT-005 · approved · source: requirements:functional:Catalogue upload

Admins upload the film list as CSV. Imports merge by film identifier, reject bad rows individually with a per-row report, and trigger a similarity rebuild. The site stays usable throughout, and failures leave previous data in use.

- Upload a catalogue file: an admin uploads a CSV with columns id, title, year, plot, genres, director, cast and an optional poster address. The upload page shows progress and the outcome of the import.
- Merge by film identifier: new identifiers are added; rows matching an existing film update that film in place keeping the same identifier; films absent from the file are left untouched. Likes on updated films all still exist and still point at the same film.
- Admin-only upload access: only signed-in admins can reach or submit the catalogue upload. Viewers and anonymous visitors are refused and no catalogue change takes place.
- Per-row import summary: bad rows are rejected rather than aborting the file; valid rows still apply. On completion the admin sees counts of rows added, updated and rejected, with each rejected row identified by row number and reason. A file without the optional poster column is accepted and those films simply have no poster.
- Refuse unreadable files: a file that is not valid CSV or has no recognisable header row is refused with a clear message, and the catalogue is left exactly as it was.
- Rebuild similarity after upload: when an upload adds or updates films, similarity texts and comparison data are rebuilt, and the admin can see when the rebuild started and finished. After completion, newly added films can appear in similar-film lists and changed genres or cast are reflected.
- Keep working during a rebuild: search and film detail pages stay usable and continue serving previously stored similarity results. A rebuild that fails partway is reported on the upload result and the previous results remain in use so recommendations keep working.

**Related:** REQ-SVC-001, NFR-006, NFR-007, NFR-010, NFR-018

### REQ-FEAT-006 · approved · source: requirements:functional:Guides & setup notes

Two written documents ship with the product: a CSV format guide for admins preparing catalogue files, and setup notes for operators standing up an instance.

- CSV format guide: lists the columns id, title, year, plot, genres, director, cast and optional poster address; states which are required; shows how multi-value fields such as genres and cast are separated. Includes a small worked example file and explains merge-by-identifier behaviour and how rejected rows are reported, so a file built from the example uploads with zero rejections.
- Setup notes for operators: cover the configuration needed to run the service, including the database and the seeded first admin credentials, and are sufficient to get a running instance.

**Related:** REQ-FEAT-005, REQ-SVC-001

### NFR-001 · approved · source: requirements:NFR-001

Performance. Similar-film lists must return within a few seconds on a full catalogue.

- Category: performance.
- With a 50,000-film catalogue whose similarity data is already prepared, a visitor's similar-films list is displayed within a few seconds on a normal connection.
- Achieved by reading stored results rather than recalculating during the request.
- Target: 50,000-film catalogue; list returned and displayed within a few seconds on a normal connection.

**Related:** REQ-FEAT-003

### NFR-002 · approved · source: requirements:NFR-002

Performance. Title search and autocomplete must feel immediate as the visitor types.

- Category: performance.
- Applies across a catalogue of up to 50,000 films.
- Must hold even when the typed text contains spelling mistakes.

**Related:** REQ-FEAT-002, REQ-SVC-001

### NFR-003 · approved · source: requirements:NFR-003

Usability. Recommendations must be stable between catalogue uploads.

- Category: usability.
- Opening the same film's detail page twice with no catalogue upload in between produces identical similar-film lists.
- Target: two lists identical in content and order.

**Related:** REQ-FEAT-003

### NFR-004 · approved · source: requirements:NFR-004

Scalability. The app must support 5,000–50,000 films and thousands of signed-up viewers without re-architecting.

- Category: scalability.
- Growth must be absorbed from the single-region deployment with no re-architecting.
- Target: 5,000–50,000 films; thousands of users.

**Related:** REQ-STACK-001

### NFR-005 · approved · source: requirements:NFR-005

Scalability. Per-film similarity vectors are capped to a fixed width so they remain indexable in the database.

- Category: scalability.
- Vectors capped to roughly 1,000–1,500 features.
- Rationale: keeps them indexable in PostgreSQL/pgvector at this catalogue size.
- Target: around 1,000–1,500 features per film vector.

**Related:** REQ-SVC-001

### NFR-006 · approved · source: requirements:NFR-006

Availability. The site stays usable while the catalogue similarity data rebuilds.

- Category: availability.
- Search, film detail pages and recommendation lists remain available throughout a rebuild, serving previously stored results until the new ones are ready.
- A rebuild that fails partway leaves the previous results in use so recommendations keep working.

**Related:** REQ-FEAT-005

### NFR-007 · approved · source: requirements:NFR-007

Availability. A failed import must never leave broken data.

- Category: availability.
- An upload failing partway leaves the catalogue and every user's likes in a consistent state.
- No film is half-written; no film absent from the file is removed; no like is lost or repointed.

**Related:** REQ-FEAT-005, REQ-STACK-001

### NFR-008 · approved · source: requirements:NFR-008

Security. Passwords are stored only as a one-way hash, and failed sign-in reveals nothing about account existence.

- Category: security.
- The plain password cannot be recovered from storage or from any log.
- A failed sign-in returns a generic message that does not reveal whether an email address has an account.

**Related:** REQ-FEAT-001

### NFR-009 · approved · source: requirements:NFR-009

Security. Sessions expire on inactivity and can be ended explicitly.

- Category: security.
- A signed-in session expires after the configured period of inactivity; the user is asked to sign in again.
- Sign out ends the session immediately.
- Deleting an account ends that user's session at once; further attempts to reach the personalised list or liked films require a fresh sign-in.

**Related:** REQ-FEAT-001

### NFR-010 · approved · source: requirements:NFR-010

Security. Catalogue upload is restricted to admin accounts only.

- Category: security.
- Only accounts holding the admin role can reach or submit a catalogue upload.
- Viewers and anonymous visitors are refused, with no catalogue change.
- The admin role can only be granted directly in the database, never through a self-serve request.

**Related:** REQ-FEAT-005, REQ-FEAT-001

### NFR-011 · approved · source: requirements:NFR-011

Privacy. Likes and personalised recommendations are visible only to their owner.

- Category: privacy.
- A signed-in user sees only their own likes and their own personalised list, never another user's.
- No other user can view or change them.

**Related:** REQ-FEAT-004

### NFR-012 · approved · source: requirements:NFR-012

Privacy. Personal data is limited to email address, password hash and like history.

- Category: privacy.
- No personal data is stored beyond email address, password hash and like history.
- No advertising, no profiling for third parties, no sharing of personal data with anyone.

**Related:** REQ-FEAT-001

### NFR-013 · approved · source: requirements:NFR-013

Compliance. Generic GDPR-style handling: self-serve account deletion removes all personal data.

- Category: compliance.
- GDPR-style handling applies generically (no region was named).
- A user can delete their account unaided.
- Once deletion completes: account record, email address and all likes are gone; sign-in with those details fails; the film catalogue is unaffected.

**Related:** REQ-FEAT-001

### NFR-014 · approved · source: requirements:NFR-014

Compatibility. All main pages work in both mobile and desktop browsers.

- Category: compatibility.
- Search, film detail pages, similar-film lists, the personalised list and the liked-films list all reflow to a single-column layout at narrow mobile viewports.
- No horizontal scrolling and no overlapping content at narrow viewports.
- On desktop the wider viewport is used and all content remains readable.

**Related:** REQ-SCOPE-001

### NFR-015 · approved · source: requirements:NFR-015

Usability. Touch-friendly controls on mobile, with instant Like feedback.

- Category: usability.
- On a mobile browser the search box, autocomplete suggestions and the Like control are operable by touch, with targets large enough to tap accurately.
- The Like control switches to its liked state immediately wherever it appears, without leaving the current page.

**Related:** REQ-FEAT-002, REQ-FEAT-004

### NFR-016 · approved · source: requirements:NFR-016

Security — accepted risk. No self-serve password reset in this version, because the email provider was removed from the delivery scope.

- Category: security. Marked as an accepted risk.
- Consequence: a user who loses their password cannot recover it unaided.
- Cause: the email provider (SendGrid) was removed from the delivery scope.
- Accepted for this version; should be closed before or shortly after launch.

**Related:** REQ-FEAT-001, REQ-SCOPE-001

### NFR-017 · approved · source: requirements:NFR-017

Security — accepted risk. No email-verification gate on signup and no social login.

- Category: security. Marked as an accepted risk.
- Accounts are usable immediately on signup with no email-verification step.
- No social login.
- Consequence: an email address is not proven to belong to the person who registered it. Accepted for this version.

**Related:** REQ-FEAT-001

### NFR-018 · approved · source: requirements:NFR-018

Performance — accepted risk. The similarity rebuild runs inside the upload operation rather than in the background.

- Category: performance. Marked as an accepted risk.
- Cause: the queue (Redis) was removed from the delivery scope.
- Consequence: the admin waits for the rebuild to finish and sees its progress, rather than it running asynchronously. Accepted for this version.

**Related:** REQ-FEAT-005, REQ-SCOPE-001

### NFR-019 · approved · source: requirements:NFR-019

Availability — accepted risk. Single-region hosting with no failover.

- Category: availability. Marked as an accepted risk.
- The app is hosted in one cloud region only.
- A region-wide outage takes the whole service down.
- No cross-region redundancy or failover is provided in this version.

**Related:** REQ-STACK-001

### NFR-020 · approved · source: requirements:NFR-020

Maintainability — accepted risk. No accuracy target or analytics for recommendation quality.

- Category: maintainability. Marked as an accepted risk.
- Recommendation quality is judged subjectively in this version.
- No accuracy target, no A/B testing, no analytics instrumentation to measure whether suggestions are good.

**Related:** REQ-FEAT-003, REQ-FEAT-004
<!-- sdlc:auto:end narrative -->

<!-- sdlc:auto:start summary -->
**Requirements version:** v1  
**Delivery scope:** lightweight  
**Approved:** 2026-11-13

**Capabilities in scope:** Movie title search, Film detail page with plot, year, genres, director and cast, Similar-movies list ranked by cosine similarity, Like a film from its detail page, "For you" personalised list from all liked films, My liked films list with unlike, Signup and sign-in with email and password, Catalogue CSV upload merging by movie ID, Similarity vector rebuild after each catalogue upload, Delete my account and all my likes, Responsive layout for desktop and mobile browsers, CSV format guide and setup notes, Film text "soup" weighting genres, director and cast above plot, TF-IDF vectors precomputed in pgvector, not computed per request, Default 12 similar films, excluding the film itself, Default top 20 "For you" films, excluding films already liked, Prompt to like some films when a viewer has no likes yet, CSV columns: id, title, year, plot, genres, director, cast, poster_url, Posters shown only where the CSV supplies a poster URL, Hashed passwords with server-side sessions, First admin account seeded at deployment, Free to use — no payments, subscriptions or billing

**Approved technology stack:**

| Layer | Choice | Rationale |
|---|---|---|
| backend | FastAPI | The heart of WatchNext is the recommendation maths: building a text 'soup' per film, TF-IDF vectorising 5,000–50,000 films after each CSV upload, and cosine-ranking them. That is Python work — scikit-learn and pandas do it in a few lines and are the same tools your approach was described in. FastAPI lets the CSV importer, the vectoriser and the web API all live in one Python codebase, so there is no second language boundary between the app and the recommender. |
| cloud | AWS | A single-region deployment was assumed and AWS covers everything WatchNext needs in one account: a managed PostgreSQL with automated backups, a container for the API and the vector-rebuild worker, S3 for the admin's uploaded CSV files, and a CDN so anonymous visitors anywhere get fast film pages. It also scales from a handful of users to the thousands you expect without re-architecting, and costs stay modest because the app is free and serves no video or images of its own. |
| database | PostgreSQL | Exactly as assumed. Your data is small and highly relational — users, their likes, films, and one precomputed vector per film — and the CSV merge rule you specified ('update existing by ID, add new, never destroy likes') is precisely what PostgreSQL's transactional upserts and foreign keys guarantee: a half-failed import cannot leave the catalogue or anyone's likes in a broken state. It also hosts the similarity vectors through pgvector, so recommendations need no separate database. |
| frontend | React | WatchNext is one responsive web app used by anonymous browsers and signed-in viewers alike: a search box with live autocomplete, film pages with ranked similar-film grids, and like buttons that must toggle instantly wherever they appear — in search results, in similar lists, on the detail page. React handles that kind of constantly-updating interface well and keeps one like-button component reused in all three places rather than three copies. It is also the platform default here, and nothing you described argues for anything else. |
<!-- sdlc:auto:end summary -->

<!-- sdlc:auto:start epics -->
## EPIC-001: Movie search and film details

Any visitor, signed in or not, can find a film in the catalogue by typing its title and open its detail page.

Assumption carried forward from discovery (not yet validated): search is title-based keyword/substring matching with typo tolerance and autocomplete suggestions. No browse-by-genre or filter pages are in this version. Film metadata comes solely from the admin-uploaded CSV — there is no live integration with an external movie database, so posters appear only where the CSV supplied a poster URL.

### US-001: Search for a film by title

**Priority:** critical

> As a visitor (signed in or not), I want to type a movie title and see matching films from the catalogue, so that I can find the film I have just watched and start from there.

| ID | Given | When | Then |
|---|---|---|---|
| AC-001 | the catalogue contains a film titled "Blade Runner" | an anonymous visitor searches for "blade runner" | the film "Blade Runner" appears in the results with its title and year, without the visitor being asked to sign in |
| AC-002 | the catalogue contains a film titled "The Godfather" | a visitor searches for the partial term "godfath" | "The Godfather" is returned, because matching is keyword/substring based on title |
| AC-003 | the catalogue contains a film titled "Interstellar" | a visitor searches with the misspelling "interstelar" | "Interstellar" is still returned, because matching is typo tolerant (assumed behaviour, pending validation) |
| AC-004 | the catalogue contains no film matching the search term | a visitor searches for "zzzzqqq" | an empty-result message is shown inviting the visitor to try another title, and no error is displayed |

### US-002: See autocomplete suggestions while typing

**Priority:** medium

> As a visitor, I want title suggestions to appear as I type, so that I can pick the right film without typing the full title.

| ID | Given | When | Then |
|---|---|---|---|
| AC-005 | a visitor is on the search box | they have typed at least two characters of a title that exists in the catalogue | a list of suggested film titles with their years is shown beneath the box |
| AC-006 | a suggestion list is displayed | the visitor selects one of the suggestions | they are taken directly to that film's detail page |
| AC-007 | a visitor has typed fewer than two characters | the suggestion list would be evaluated | no suggestions are shown |

### US-003: View a film's detail page

**Priority:** critical

> As a visitor, I want to see a film's plot, year, genres, director and cast on one page, so that I can confirm it is the film I mean before looking at what is similar.

| ID | Given | When | Then |
|---|---|---|---|
| AC-008 | a film in the catalogue has plot, year, genres, director and cast values | a visitor opens that film's detail page | the title, year, plot, genres, director and cast are all displayed |
| AC-009 | a film's CSV row supplied a poster_url | a visitor opens its detail page | the poster image is displayed |
| AC-010 | a film's CSV row supplied no poster_url | a visitor opens its detail page | the page renders normally with a placeholder in place of the poster and no broken image |
| AC-011 | an optional field such as director or cast is empty for a film | a visitor opens its detail page | that field is omitted or shown as unavailable, and the rest of the page still renders |
| AC-012 | a visitor requests a film ID that is not in the catalogue | the page loads | a "film not found" page is shown with a link back to search |

## EPIC-002: Similar-movie recommendations

Each film's detail page shows a ranked list of the films most like it.

Assumption carried forward from discovery (not yet validated): similarity is content-based. Each film is turned into one combined text "soup" of plot + genres + director + cast, with genres, director and cast weighted above plot; the soup is vectorised with TF-IDF and films are compared by cosine similarity. Vectors are precomputed and stored (in pgvector) after each catalogue upload, never computed per request. There is no custom machine-learning model and no accuracy target — quality is judged subjectively in this version, with no A/B testing or analytics instrumentation.

### US-004: See films similar to the one I am looking at

**Priority:** critical

> As a visitor (signed in or not), I want a ranked list of films most like the one on screen, so that I can decide what to watch next.

| ID | Given | When | Then |
|---|---|---|---|
| AC-013 | a film has a precomputed similarity vector and the catalogue holds more than 12 other vectorised films | a visitor opens that film's detail page | exactly 12 similar films are listed, ordered from highest to lowest cosine similarity |
| AC-014 | a visitor is viewing the similar-films list for film X | the list is rendered | film X itself does not appear anywhere in the list |
| AC-015 | the catalogue holds fewer than 12 other vectorised films | a visitor opens a film's detail page | all available similar films are listed, in similarity order, without error |
| AC-016 | a similar-films list is displayed | the visitor selects one of the listed films | that film's own detail page opens, with its own similar-films list |
| AC-017 | a film has no precomputed vector yet (for example it was added by an upload whose rebuild has not finished) | a visitor opens its detail page | the metadata is still shown and the similar-films area explains that recommendations are not available yet |

### US-005: Build the film text "soup" with weighted features

**Priority:** high

> As a product owner, I want each film's similarity text to weight genres, director and cast above plot, so that recommendations are driven by the people and categories behind a film rather than plot wording alone.

| ID | Given | When | Then |
|---|---|---|---|
| AC-018 | a film has plot, genres, director and cast values | its similarity text is built | the text combines all four fields into a single soup, with genres, director and cast contributing more weight than plot (assumed weighting, pending validation) |
| AC-019 | two films share a director and two genres but have unrelated plots, and two other films share only plot wording | similarity scores are compared | the pair sharing director and genres scores higher than the pair sharing only plot wording |
| AC-020 | a film has an empty plot but populated genres, director and cast | its soup is built and vectorised | a vector is still produced from the available fields and the film appears in similarity results |

### US-006: Serve recommendations from precomputed vectors

**Priority:** high

> As a visitor, I want similar-film lists to appear without waiting for the maths to run, so that browsing stays responsive on a catalogue of up to 50,000 films.

| ID | Given | When | Then |
|---|---|---|---|
| AC-021 | the catalogue has been vectorised after an upload | a visitor opens any film detail page | the similar-films list is produced from stored vectors and no TF-IDF vectorisation of the catalogue is performed during the request |
| AC-022 | a catalogue of 50,000 films with stored vectors | a visitor requests a film's similar-films list | the list is returned and displayed within a few seconds on a normal connection |
| AC-023 | the same film detail page is opened twice with no catalogue upload in between | both lists are compared | the two lists are identical in content and order |

## EPIC-003: Likes and the "For you" list

Signed-in viewers like the films they enjoyed and get a personalised ranked list built from everything they have liked.

Assumption carried forward from discovery (not yet validated): "For you" is the mean of the vectors of all the user's liked films, ranked by cosine similarity, excluding films already liked, returning a default top 20. Liking is available from search results and similar-film lists as well as from a film's detail page. There is no dislike / not-interested marking and no watchlist in this version.

### US-007: Like a film

**Priority:** critical

> As a signed-in viewer, I want to like a film from its detail page, from search results or from a similar-films list, so that the app learns what I enjoy and saves it to my account.

| ID | Given | When | Then |
|---|---|---|---|
| AC-024 | a signed-in viewer is on a film's detail page and has not liked that film | they select Like | the control switches to a liked state immediately and the like is stored against their account |
| AC-025 | a signed-in viewer sees a film in search results or in a similar-films list | they select Like next to that film | the like is saved without leaving the current page |
| AC-026 | a viewer has liked a film and then signs out and signs back in | they revisit that film | the film is still shown as liked |
| AC-027 | an anonymous visitor is viewing a film | they select Like | they are prompted to sign in or sign up, and no like is recorded until they do |
| AC-028 | a viewer has already liked a film | the like action is submitted again for the same film | the account still holds exactly one like for that film and no duplicate or error is created |

### US-008: See and unlike my liked films

**Priority:** high

> As a signed-in viewer, I want a list of every film I have liked, with the ability to remove a like, so that I can correct mistakes and keep my recommendations accurate.

| ID | Given | When | Then |
|---|---|---|---|
| AC-029 | a viewer has liked five films | they open My liked films | all five films are listed with their titles and years |
| AC-030 | a viewer is on My liked films | they unlike one of the films | the film is removed from the list and is no longer recorded as liked on their account |
| AC-031 | a viewer has unliked a film | their "For you" list is next generated | that film no longer contributes to the list and becomes eligible to be recommended again |
| AC-032 | a viewer has liked no films | they open My liked films | an empty-state message invites them to search for films they enjoyed |

### US-009: Get a personalised "For you" list

**Priority:** critical

> As a signed-in viewer, I want a ranked list of films based on everything I have liked, so that I get suggestions without having to start from one particular film.

| ID | Given | When | Then |
|---|---|---|---|
| AC-033 | a viewer has liked at least one film and the catalogue holds more than 20 other vectorised films | they open "For you" | exactly 20 films are listed in descending order of cosine similarity to the mean vector of their liked films (assumed method, pending validation) |
| AC-034 | a viewer has liked films A, B and C | their "For you" list is generated | none of A, B or C appears in the list |
| AC-035 | a viewer likes an additional film | they reopen "For you" | the list reflects the new like, with the newly liked film excluded |
| AC-036 | the catalogue holds fewer than 20 unliked vectorised films | a viewer opens "For you" | all available films are listed in similarity order, without error |
| AC-037 | an anonymous visitor | they try to open "For you" | they are prompted to sign in or sign up and no personalised list is shown |

### US-010: Be prompted to like films when I have none

**Priority:** medium

> As a new signed-in viewer with no likes, I want a clear prompt explaining how to get recommendations, so that I know what to do instead of facing an empty page.

| ID | Given | When | Then |
|---|---|---|---|
| AC-038 | a signed-in viewer has liked no films | they open "For you" | a prompt is shown asking them to like some films, with a route into search, and no recommendation list is displayed |
| AC-039 | a viewer was shown the no-likes prompt and then likes a film | they return to "For you" | the prompt is replaced by a populated recommendation list |

## EPIC-004: Accounts, sign-in and personal data

Open self-serve signup with email and password, so likes are saved to an account, plus the ability to delete the account and all its data.

Assumptions carried forward from discovery (not yet validated): no social login and no email-verification gate on first use. Personal data is limited to email, password hash and like history; there is no advertising, no profiling for third parties and no data sharing, with generic GDPR-style handling since no region was named.

Conflict to resolve: the confirmed understanding assumed password reset by emailed link, but the chosen delivery scope removes the email provider from the stack. No self-serve password reset is drafted for this version; a user who loses their password cannot recover it unaided. This gap should be closed before or shortly after launch.

### US-011: Sign up with email and password

**Priority:** critical

> As a visitor, I want to create an account with my email address and a password, so that my likes and recommendations are saved to me.

| ID | Given | When | Then |
|---|---|---|---|
| AC-040 | a visitor is on the signup form | they submit an unused email address and a password meeting the stated minimum length | the account is created, they are signed in, and they are returned to where they were |
| AC-041 | an account already exists for an email address | a visitor tries to sign up with that same address | signup is refused with a message telling them to sign in instead, and no second account is created |
| AC-042 | a visitor submits a malformed email address or a password shorter than the stated minimum | they submit the form | a field-level validation message is shown and no account is created |
| AC-043 | an account has been created | the stored user record is inspected | the password is held only as a one-way hash and the plain password is not recoverable from storage or from any log |
| AC-044 | a newly created account | the user immediately searches, likes a film and opens "For you" | all of these work without any email verification step |

### US-012: Sign in and sign out securely

**Priority:** critical

> As a registered user, I want to sign in, stay signed in while I use the app, and sign out when I am done, so that only I can see and change my likes.

| ID | Given | When | Then |
|---|---|---|---|
| AC-045 | a registered user on the sign-in form | they submit their correct email and password | they are signed in and their liked films are available to them |
| AC-046 | a registered user | they submit a wrong password | sign-in is refused with a generic message that does not reveal whether the email address exists |
| AC-047 | a signed-in user | they choose Sign out | their session ends immediately and any further attempt to reach "For you" or My liked films with that ended session requires signing in again |
| AC-048 | a signed-in user who has been inactive beyond the configured session lifetime | they next request a signed-in page | the session has expired and they are asked to sign in again |
| AC-049 | two different registered users | user A is signed in | user A sees only their own likes and "For you" list, never user B's |

### US-013: Delete my account and all my likes

**Priority:** high

> As a registered user, I want to delete my account together with my like history, so that I can remove my personal data from the service when I choose.

| ID | Given | When | Then |
|---|---|---|---|
| AC-050 | a signed-in user on their account settings | they request deletion and confirm it on an explicit confirmation step | their account record, email address and all their likes are deleted |
| AC-051 | a user who has requested deletion | the deletion completes | their session ends and they are returned to the public site as an anonymous visitor |
| AC-052 | an account has been deleted | someone attempts to sign in with that email and password | sign-in fails, and the same email address may be used to create a brand-new account with no previous likes |
| AC-053 | a user deletes their account | the catalogue is inspected afterwards | no film records have been removed or altered by the deletion |
| AC-054 | a user is on their account settings | they review what the service holds about them | the page states that only their email address, password hash and like history are stored |

## EPIC-005: Catalogue administration via CSV

The admin-uploaded CSV is the single source of truth for film data. Uploads merge by movie ID, never destroy user likes, and trigger a rebuild of the similarity vectors.

Assumptions carried forward from discovery (not yet validated): expected CSV columns are id, title, year, plot, genres, director, cast and optional poster_url; import is error-tolerant, so bad rows are rejected and reported rather than aborting the file.

Conflict to resolve: the confirmed understanding assumed the vector rebuild would run asynchronously on a queue, but the chosen delivery scope removes the queue from the stack. The rebuild is therefore drafted as part of the upload operation, with progress visible to the admin.

### US-014: Upload a catalogue CSV that merges by movie ID

**Priority:** critical

> As a admin, I want to upload the film list as a CSV that merges into the existing catalogue by movie ID, so that I can keep the catalogue current without losing data or users' likes.

| ID | Given | When | Then |
|---|---|---|---|
| AC-055 | a signed-in admin on the catalogue upload page | they upload a CSV containing a row whose id is not yet in the catalogue | that film is added to the catalogue |
| AC-056 | a film with id 101 already exists in the catalogue | an upload contains a row with id 101 and changed field values | the existing film record is updated in place and keeps the same id |
| AC-057 | a film with id 202 exists in the catalogue | an upload is processed that contains no row with id 202 | film 202 remains in the catalogue unchanged and is not deleted |
| AC-058 | users have liked films that are updated by an upload | the upload completes | every one of those likes still exists and still points at the same film |
| AC-059 | a signed-in viewer who is not an admin, or an anonymous visitor | they attempt to reach the catalogue upload page or submit an upload | access is refused and no catalogue change occurs |

### US-015: See a per-row validation summary for an upload

**Priority:** high

> As a admin, I want bad rows reported rather than the whole file rejected, so that a few malformed rows do not block an otherwise good catalogue update.

| ID | Given | When | Then |
|---|---|---|---|
| AC-060 | a CSV in which some rows are missing an id or a title | the admin uploads the file | the valid rows are still applied, the invalid rows are rejected, and processing does not abort |
| AC-061 | an upload has finished processing | the admin views the result | a summary shows counts of rows added, rows updated and rows rejected |
| AC-062 | rows were rejected | the admin views the summary | each rejected row is identified by its row number with the reason for rejection |
| AC-063 | a file that is not valid CSV or has no recognisable header row | the admin uploads it | the upload is refused with a clear message and the catalogue is left unchanged |
| AC-064 | a CSV without the optional poster_url column | it is uploaded | the rows are accepted and the affected films simply have no poster |

### US-016: Rebuild similarity vectors after each upload

**Priority:** critical

> As a admin, I want the similarity vectors recomputed whenever the catalogue changes, so that recommendations reflect the films I have just added or updated.

| ID | Given | When | Then |
|---|---|---|---|
| AC-065 | an upload has added or updated films | the upload completes | a rebuild of the film soups and TF-IDF vectors runs and the admin can see that it has started and when it has finished |
| AC-066 | a rebuild has completed after a film was newly added | a visitor opens a related film's detail page | the newly added film is eligible to appear in the similar-films list |
| AC-067 | a rebuild has completed after a film's genres or cast were changed | that film's similar-films list is viewed | the list reflects the updated values rather than the previous ones |
| AC-068 | a rebuild is in progress | visitors use search and film detail pages | the site remains usable, serving the previously stored vectors until the rebuild finishes |
| AC-069 | a rebuild fails partway through | the admin views the upload result | the failure is reported and the previously stored vectors remain in use so recommendations keep working |

### US-017: Sign in as the seeded first admin

**Priority:** high

> As a operator deploying WatchNext, I want a first admin account created at deployment, so that someone can upload the initial catalogue before any other user exists.

| ID | Given | When | Then |
|---|---|---|---|
| AC-070 | a freshly deployed environment | deployment completes | exactly one admin account exists, created from the configured seed credentials |
| AC-071 | the seeded admin account | the operator signs in with it | they can reach the catalogue upload page |
| AC-072 | a user who signed up through the public signup form | their role is inspected | they hold the viewer role, and the admin role can only be granted directly in the database — there is no self-serve request for it |
| AC-073 | a deployment is run a second time against an environment that already has the seeded admin | seeding runs | no duplicate admin account is created and the existing account is left intact |

### US-018: Follow the CSV format guide and setup notes

**Priority:** medium

> As a admin or operator, I want written guidance on the CSV format and on setting the app up, so that I can prepare a valid catalogue file and run the service without guessing.

| ID | Given | When | Then |
|---|---|---|---|
| AC-074 | an admin preparing a catalogue file | they open the CSV format guide | it lists the columns id, title, year, plot, genres, director, cast and optional poster_url, states which are required, and shows how multi-value fields such as genres and cast are separated |
| AC-075 | the CSV format guide | an admin reads it | it includes a small worked example file and explains the merge-by-id behaviour and how rejected rows are reported |
| AC-076 | an operator setting up a new environment | they follow the setup notes | the notes cover the required configuration, including the database and the seeded first admin credentials, and are sufficient to get a running instance |
| AC-077 | an admin follows the example in the guide to build a file | they upload it | it is accepted with zero rejected rows |

## EPIC-006: Public, free access on any device

WatchNext is open to anyone on the web, works in mobile and desktop browsers, and is entirely free.

Assumptions carried forward from discovery (not yet validated): a responsive web app hosted in a single cloud region, with no native iOS or Android apps. There are no payments, subscriptions, plans or billing of any kind, so no billing capability is drafted anywhere in this document.

### US-019: Use WatchNext on a phone or a desktop

**Priority:** high

> As a visitor, I want the site to be usable in my mobile browser as well as on desktop, so that I can look up what to watch next from the sofa.

| ID | Given | When | Then |
|---|---|---|---|
| AC-078 | a visitor on a mobile browser at a narrow viewport | they open search, a film detail page, the similar-films list, "For you" and My liked films | each page reflows to a single-column layout with no horizontal scrolling and no overlapping content |
| AC-079 | a visitor on a mobile browser | they use the search box, autocomplete suggestions and the Like control | each is operable by touch with targets large enough to tap accurately |
| AC-080 | a visitor on a desktop browser | they open the same pages | the layout makes use of the wider viewport and all content remains readable |

### US-020: Browse without an account and without paying

**Priority:** high

> As a anonymous visitor, I want to search films and see similar titles without signing up or paying, so that I can get value immediately and only create an account if I want personalisation.

| ID | Given | When | Then |
|---|---|---|---|
| AC-081 | an anonymous visitor arriving at the site | they search for a film and open its detail page | search results, film metadata and the similar-films list are all available with no sign-in prompt |
| AC-082 | an anonymous visitor | they attempt to like a film or open "For you" or My liked films | they are invited to sign in or sign up, and those features remain unavailable until they do |
| AC-083 | any visitor or signed-in user | they use any part of the app, including signup, liking, "For you" and account deletion | no payment, subscription, plan selection, price or billing detail is ever requested or displayed |
<!-- sdlc:auto:end epics -->
