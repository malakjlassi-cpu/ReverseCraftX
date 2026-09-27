# Data Model — ReverseCraftX (V1)

This document defines the relational database schema **actually needed
for V1**: authentication, image upload, and analysis. Entities needed
only for later versions (`Article`, `Comment`, `SavedArticle`, `Report`,
`Search`) are listed separately in section 3 so they are not built
before their use case exists (see PROJECT.md, "What V1 Will NOT Do").

## 1. V1 Entities

```
User
 │
 ├──< Image
 │       │
 │       └──< Analysis
 │
 └──< Analysis   (direct link, see note below)
```

### 1.1 User
Represents a signed-in person.

- Attributes:
  - `id` (PK, UUID / auto-increment)
  - `first_name` (String)
  - `last_name` (String)
  - `email` (String, unique, case-insensitive comparison — FR-UC03-03)
  - `password_hash` (String — never the raw password, NFR-SEC-01)
  - `privacy_notice_accepted_at` (Timestamp, nullable — FR-UC05-22)
  - `created_at` (Timestamp)
- Relations:
  - Owns multiple `Image`s (1:N)
  - Owns multiple `Analysis`es (1:N)

### 1.2 Image
Represents an uploaded garment photo. **Belongs directly to a `User`
in V1** — there is no `Article` yet, so an image cannot and does not
need to be attached to one.

- Attributes:
  - `id` (PK)
  - `user_id` (FK -> User) — owner / uploader
  - `original_filename` (String) — as received, never used to build a path (NFR-SEC-03)
  - `stored_filename` (String) — random name, e.g. UUID (NFR-SEC-03)
  - `mime_type` (String) — detected from content, not extension (NFR-SEC-02)
  - `size_bytes` (Integer)
  - `path` (String) — path under `storage/images/`
  - `created_at` (Timestamp)
- Relations:
  - Belongs to one `User` (N:1)
  - Can have multiple `Analysis` records (1:N) — one per attempt/retry
    (FR-UC05-19: a retry creates a new `Analysis` for the same stored image)

### 1.3 Analysis
Represents one AI analysis attempt for an image (UC-05).

- Attributes:
  - `id` (PK)
  - `user_id` (FK -> User) — owner (redundant with `image.user_id` but
    kept directly here so ownership checks, FR-UC02-07, never need a
    join through `Image`)
  - `image_id` (FK -> Image)
  - `description` (Text, nullable, max 500 chars — FR-UC05-08, FR-UC05-21)
  - `status` (Enum: `PENDING`, `PROCESSING`, `COMPLETED`, `FAILED`)
  - `components` (JSON, nullable — populated only when COMPLETED)
  - `probable_materials` (JSON, nullable)
  - `structure` (JSON, nullable)
  - `hypotheses` (JSON, nullable)
  - `manufacturing_steps` (JSON, nullable)
  - `limitations` (JSON, nullable)
  - `error_message` (Text, nullable — populated only when FAILED)
  - `retry_of_analysis_id` (FK -> Analysis, nullable — links a retry to
    the analysis it retries, for the 3-retries-per-image rule, FR-UC05-19)
  - `ai_model_version` (String, nullable — NFR-LOG-02)
  - `processing_duration_ms` (Integer, nullable — NFR-LOG-02)
  - `created_at` (Timestamp)
  - `started_at` (Timestamp, nullable — when it left PENDING)
  - `completed_at` (Timestamp, nullable — when it reached COMPLETED/FAILED)
- Relations:
  - Belongs to one `User` (N:1)
  - Belongs to one `Image` (N:1)
  - May reference a previous `Analysis` via `retry_of_analysis_id` (self N:1)

> **Why `Analysis.user_id` instead of relying on `Image → Article →
> User`:** in the previous draft, `Image` and `Analysis` were only
> reachable through `Article`, which doesn't exist until V1.1. That
> made it impossible to create an image or an analysis at all in V1.
> V1 has no `Article`, so ownership must be direct.

## 2. Notes on Constraints Enforced at the Application Level

These are business rules from REQUIREMENTS.md, not columns:

- Daily quota (FR-UC05-09): count of `Analysis` rows created by a
  `user_id` today, following the counting rules of FR-UC05-10 — not a
  stored counter, to avoid a second source of truth.
- Global daily cap (FR-UC05-23): count of AI provider calls made today
  across all users — can be derived from `Analysis` rows that reached
  PROCESSING today, or tracked in a small separate counter table if
  querying `Analysis` becomes inconvenient.
- Max 3 retries per image (FR-UC05-19): count of `Analysis` rows
  sharing the same `image_id` (or walking `retry_of_analysis_id`
  chains).

## 3. Entities for Later Versions (NOT built in V1)

Kept here only as a placeholder so the eventual schema migration is
easy to anticipate. Do not create these tables/models until the
corresponding use case is actually being implemented.

### 3.1 Article — V1.1 (UC-04, UC-11)
- `id`, `user_id` (FK -> User, author), `title`, `description`,
  `created_at`
- Will need a way to attach one or more existing `Image`/`Analysis`
  records to an article at publish time (exact mechanism to be decided
  in V1.1 — e.g. `Article.image_id` and/or `Article.analysis_id`).

### 3.2 Comment — V1.2 (UC-08)
- `id`, `article_id` (FK -> Article), `user_id` (FK -> User),
  `content`, `created_at`, `updated_at` (nullable)

### 3.3 SavedArticle — V1.2 (UC-06, UC-07)
- `user_id` (PK/FK -> User), `article_id` (PK/FK -> Article),
  `saved_at`

### 3.4 Report — V1.2 (UC-10)
- `id`, `article_id` (FK -> Article), `user_id` (FK -> User, reporter),
  `reason`, `status` (e.g. `open`, `reviewed`, `dismissed`), `created_at`

### 3.5 Search — V1.1 (UC-01), optional audit log
- `id`, `user_id` (FK -> User, nullable if guest search is allowed),
  `keyword`, `created_at`
