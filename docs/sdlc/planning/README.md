# Planning

> Maintained by QuorQ AI SDLC. Text between `sdlc:auto` markers is regenerated; anything outside them is yours and is kept.

<!-- sdlc:auto:start narrative -->
### EPIC-001 · approved · source: plan:EPIC-001

Movie search and film details. Any visitor, signed in or not, can find a film in the catalogue by typing its title and open its detail page.

Goal: title-driven discovery for all visitors, with no sign-in required.

Assumption carried forward from discovery (not yet validated):
- Search is title-based keyword/substring matching with typo tolerance and autocomplete suggestions.
- No browse-by-genre or filter pages in this version.
- Film metadata comes solely from the admin-uploaded CSV; no live integration with an external movie database, so posters appear only where the CSV supplied a poster URL.

**Related:** US-001, US-002, US-003, REQ-FEAT-002, REQ-FEAT-003

### EPIC-002 · approved · source: plan:EPIC-002

Similar-movie recommendations. Each film's detail page shows a ranked list of the films most like it.

Goal: content-based 'more like this' on every film page.

Assumption carried forward from discovery (not yet validated):
- Each film becomes one combined text 'soup' of plot + genres + director + cast, with genres, director and cast weighted above plot.
- The soup is vectorised with TF-IDF; films compared by cosine similarity.
- Vectors are precomputed and stored in pgvector after each catalogue upload, never computed per request.
- No custom machine-learning model and no accuracy target; quality judged subjectively, with no A/B testing or analytics instrumentation.

**Related:** US-004, US-005, US-006, REQ-FEAT-003, NFR-001, NFR-003, NFR-020

### EPIC-003 · approved · source: plan:EPIC-003

Likes and the "For you" list. Signed-in viewers like the films they enjoyed and get a personalised ranked list built from everything they have liked.

Goal: personalisation driven entirely by a viewer's own likes.

Assumption carried forward from discovery (not yet validated):
- 'For you' is the mean of the vectors of all the user's liked films, ranked by cosine similarity, excluding films already liked, returning a default top 20.
- Liking is available from search results and similar-film lists as well as from a film's detail page.
- No dislike / not-interested marking and no watchlist in this version.

**Related:** US-007, US-008, US-009, US-010, REQ-FEAT-004

### EPIC-004 · approved · source: plan:EPIC-004

Accounts, sign-in and personal data. Open self-serve signup with email and password so likes are saved to an account, plus the ability to delete the account and all its data.

Goal: lightweight accounts with full self-serve deletion.

Assumptions carried forward from discovery (not yet validated):
- No social login and no email-verification gate on first use.
- Personal data limited to email, password hash and like history; no advertising, no profiling for third parties, no data sharing; generic GDPR-style handling since no region was named.

Conflict to resolve (recorded as stated, both sides):
- The confirmed understanding assumed password reset by emailed link.
- The chosen delivery scope removes the email provider from the stack.
- Resolution drafted: no self-serve password reset in this version; a user who loses their password cannot recover it unaided. Gap should be closed before or shortly after launch.

**Related:** US-011, US-012, US-013, REQ-FEAT-001, NFR-016

### EPIC-005 · approved · source: plan:EPIC-005

Catalogue administration via CSV. The admin-uploaded CSV is the single source of truth for film data; uploads merge by movie ID, never destroy user likes, and trigger a rebuild of the similarity vectors.

Goal: a safe, repeatable admin path for keeping the catalogue current.

Assumptions carried forward from discovery (not yet validated):
- Expected CSV columns: id, title, year, plot, genres, director, cast and optional poster_url.
- Import is error-tolerant: bad rows are rejected and reported rather than aborting the file.

Conflict to resolve (recorded as stated, both sides):
- The confirmed understanding assumed the vector rebuild would run asynchronously on a queue.
- The chosen delivery scope removes the queue from the stack.
- Resolution drafted: the rebuild runs as part of the upload operation, with progress visible to the admin.

**Related:** US-014, US-015, US-016, US-017, US-018, REQ-FEAT-005, REQ-FEAT-006, NFR-018

### EPIC-006 · approved · source: plan:EPIC-006

Public, free access on any device. WatchNext is open to anyone on the web, works in mobile and desktop browsers, and is entirely free.

Goal: immediate value for anonymous visitors on any screen size, at no cost.

Assumptions carried forward from discovery (not yet validated):
- A responsive web app hosted in a single cloud region, with no native iOS or Android apps.
- No payments, subscriptions, plans or billing of any kind, so no billing capability is drafted anywhere in the document.

**Related:** US-019, US-020, NFR-014, NFR-015, NFR-019
<!-- sdlc:auto:end narrative -->
