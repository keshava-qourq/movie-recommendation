<!-- sdlc:auto:start adr -->
# ADR-0001: Use React 19 for Frontend

Status: Accepted  
Date: 2026-11-13

## Context and Problem Statement

Which Frontend technology does the project use? Decided in architecture v1.

## Considered Options

- React 19
- Vue 3
- SvelteKit
- Server-rendered Jinja templates

## Decision Outcome

Chosen option: React 19, because The design reuses one LikeButton component in search results, similar-film grids and the film detail page, each toggling optimistically against PUT/DELETE /me/likes/{film_id}; React's component model and local state make that a single implementation rather than three. The SPA is a static bundle on CloudFront, reads the API host from VITE_API_BASE_URL and holds the session token in memory with a localStorage fallback, attaching it as Authorization: Bearer.

## Consequences

A client-rendered SPA means film pages are not server-rendered, so there is no SEO or first-paint benefit for anonymous visitors — acceptable because nothing in scope depends on search-engine indexing. Token-in-localStorage is the only option across two origins, so XSS discipline (no dangerouslySetInnerHTML on catalogue text) carries the weight a cookie's HttpOnly flag would otherwise carry.
<!-- sdlc:auto:end adr -->