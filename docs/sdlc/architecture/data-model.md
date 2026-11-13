# Data Model

<!-- sdlc:auto:start entities -->
### users

| Field | Type | PK | FK | Nullable |
|---|---|---|---|---|
| id | uuid | True | — | False |
| email | varchar(320) unique, stored lowercase | False | — | False |
| password_hash | text (bcrypt, one-way only — never logged, NFR-008) | False | — | False |
| role | varchar(16) check in ('viewer','admin') default 'viewer' | False | — | False |
| created_at | timestamptz default now() | False | — | False |

### sessions

| Field | Type | PK | FK | Nullable |
|---|---|---|---|---|
| id | uuid | True | — | False |
| user_id | uuid | False | users.id | False |
| token_hash | text unique (sha256 of the opaque bearer token — the token itself is never stored) | False | — | False |
| created_at | timestamptz default now() | False | — | False |
| last_seen_at | timestamptz | False | — | False |
| expires_at | timestamptz (SESSION_TTL_MINUTES of inactivity, NFR-009) | False | — | False |
| revoked_at | timestamptz | False | — | True |

### films

| Field | Type | PK | FK | Nullable |
|---|---|---|---|---|
| id | bigint (movie id from the CSV — the merge key) | True | — | False |
| title | text | False | — | False |
| year | integer | False | — | True |
| plot | text | False | — | True |
| genres | text[] | False | — | True |
| director | text | False | — | True |
| cast_members | text[] | False | — | True |
| poster_url | text | False | — | True |
| soup | text (weighted plot + genres + director + cast, rebuilt on import) | False | — | True |
| created_at | timestamptz default now() | False | — | False |
| updated_at | timestamptz default now() | False | — | False |

### likes

| Field | Type | PK | FK | Nullable |
|---|---|---|---|---|
| id | uuid | True | — | False |
| user_id | uuid (on delete cascade) | False | users.id | False |
| film_id | bigint (on delete restrict — an import never deletes films) | False | films.id | False |
| created_at | timestamptz default now() | False | — | False |

### vector_builds

| Field | Type | PK | FK | Nullable |
|---|---|---|---|---|
| id | uuid | True | — | False |
| import_id | uuid | False | catalog_imports.id | True |
| status | varchar(16) check in ('running','active','superseded','failed') | False | — | False |
| is_active | boolean default false (unique partial index where is_active) | False | — | False |
| feature_count | integer (TF-IDF max_features, 1024 — NFR-005) | False | — | False |
| film_count | integer | False | — | True |
| started_at | timestamptz default now() | False | — | False |
| finished_at | timestamptz | False | — | True |
| error | text | False | — | True |

### film_vectors

| Field | Type | PK | FK | Nullable |
|---|---|---|---|---|
| id | bigserial | True | — | False |
| film_id | bigint (on delete cascade) | False | films.id | False |
| build_id | uuid (on delete cascade, unique with film_id) | False | vector_builds.id | False |
| embedding | vector(1024) (pgvector, L2-normalised TF-IDF — cosine ranked, HNSW index per build) | False | — | False |

### catalog_imports

| Field | Type | PK | FK | Nullable |
|---|---|---|---|---|
| id | uuid | True | — | False |
| admin_user_id | uuid (on delete set null) | False | users.id | True |
| original_filename | text | False | — | False |
| s3_key | text (object in S3_BUCKET) | False | — | False |
| status | varchar(24) check in ('validating','merging','rebuilding','completed','refused','failed') | False | — | False |
| rows_added | integer default 0 | False | — | False |
| rows_updated | integer default 0 | False | — | False |
| rows_rejected | integer default 0 | False | — | False |
| message | text (refusal or failure reason shown on the report) | False | — | True |
| started_at | timestamptz default now() | False | — | False |
| finished_at | timestamptz | False | — | True |

### catalog_import_rows

| Field | Type | PK | FK | Nullable |
|---|---|---|---|---|
| id | bigserial | True | — | False |
| import_id | uuid (on delete cascade) | False | catalog_imports.id | False |
| row_number | integer (1-based line number in the uploaded file) | False | — | False |
| reason | text (e.g. missing id, missing title, non-numeric year) | False | — | False |
| raw_excerpt | text (truncated copy of the rejected row) | False | — | True |
<!-- sdlc:auto:end entities -->

<!-- sdlc:auto:start relationships -->
| Source | Target | Type |
|---|---|---|
| users | sessions | one-to-many |
| users | likes | one-to-many |
| films | likes | one-to-many |
| films | film_vectors | one-to-many |
| vector_builds | film_vectors | one-to-many |
| catalog_imports | catalog_import_rows | one-to-many |
| catalog_imports | vector_builds | one-to-many |
| users | catalog_imports | one-to-many |
<!-- sdlc:auto:end relationships -->

<!-- sdlc:auto:start er-diagram -->
```mermaid
erDiagram
  users {
    uuid id PK
    varchar_320__unique__stored_lowercase email
    text__bcrypt__one_way_only___never_logged__NFR_008_ password_hash
    varchar_16__check_in___viewer___admin___default__viewer_ role
    timestamptz_default_now__ created_at
  }
  sessions {
    uuid id PK
    uuid user_id FK
    text_unique__sha256_of_the_opaque_bearer_token___the_token_itself_is_never_stored_ token_hash
    timestamptz_default_now__ created_at
    timestamptz last_seen_at
    timestamptz__SESSION_TTL_MINUTES_of_inactivity__NFR_009_ expires_at
    timestamptz revoked_at
  }
  films {
    bigint__movie_id_from_the_CSV___the_merge_key_ id PK
    text title
    integer year
    text plot
    text__ genres
    text director
    text__ cast_members
    text poster_url
    text__weighted_plot___genres___director___cast__rebuilt_on_import_ soup
    timestamptz_default_now__ created_at
    timestamptz_default_now__ updated_at
  }
  likes {
    uuid id PK
    uuid__on_delete_cascade_ user_id FK
    bigint__on_delete_restrict___an_import_never_deletes_films_ film_id FK
    timestamptz_default_now__ created_at
  }
  vector_builds {
    uuid id PK
    uuid import_id FK
    varchar_16__check_in___running___active___superseded___failed__ status
    boolean_default_false__unique_partial_index_where_is_active_ is_active
    integer__TF_IDF_max_features__1024___NFR_005_ feature_count
    integer film_count
    timestamptz_default_now__ started_at
    timestamptz finished_at
    text error
  }
  film_vectors {
    bigserial id PK
    bigint__on_delete_cascade_ film_id FK
    uuid__on_delete_cascade__unique_with_film_id_ build_id FK
    vector_1024___pgvector__L2_normalised_TF_IDF___cosine_ranked__HNSW_index_per_build_ embedding
  }
  catalog_imports {
    uuid id PK
    uuid__on_delete_set_null_ admin_user_id FK
    text original_filename
    text__object_in_S3_BUCKET_ s3_key
    varchar_24__check_in___validating___merging___rebuilding___completed___refused___failed__ status
    integer_default_0 rows_added
    integer_default_0 rows_updated
    integer_default_0 rows_rejected
    text__refusal_or_failure_reason_shown_on_the_report_ message
    timestamptz_default_now__ started_at
    timestamptz finished_at
  }
  catalog_import_rows {
    bigserial id PK
    uuid__on_delete_cascade_ import_id FK
    integer__1_based_line_number_in_the_uploaded_file_ row_number
    text__e_g__missing_id__missing_title__non_numeric_year_ reason
    text__truncated_copy_of_the_rejected_row_ raw_excerpt
  }
  users ||--o{ sessions : "one-to-many"
  users ||--o{ likes : "one-to-many"
  films ||--o{ likes : "one-to-many"
  films ||--o{ film_vectors : "one-to-many"
  vector_builds ||--o{ film_vectors : "one-to-many"
  catalog_imports ||--o{ catalog_import_rows : "one-to-many"
  catalog_imports ||--o{ vector_builds : "one-to-many"
  users ||--o{ catalog_imports : "one-to-many"
```
<!-- sdlc:auto:end er-diagram -->
