# API Contract — ReverseCraftX (V1)

Status: Draft v1 — fixes the endpoint contracts referenced by
REQUIREMENTS.md ("Final endpoint contracts are fixed in API.md").

All endpoints are served by the single FastAPI backend. Base path:
`/api` (adjust if your `main.py` uses a different prefix — keep it
consistent everywhere).

Every error response uses the uniform body defined in NFR-API-01:

```json
{ "error": "<code>", "message": "<human-readable text>" }
```

## 1. Authentication

### POST /api/auth/register
Creates an account. See FR-UC03-01, FR-UC03-02, FR-UC03-03.

**Auth:** none

**Request** (`application/json`):
```json
{
  "first_name": "Amina",
  "last_name": "Ben Ali",
  "email": "amina@example.com",
  "password": "at-least-8-characters"
}
```

**Response — 201 Created**:
```json
{
  "id": 1,
  "first_name": "Amina",
  "last_name": "Ben Ali",
  "email": "amina@example.com",
  "created_at": "2026-09-27T10:00:00Z"
}
```
(No `password` or `password_hash` field is ever returned.)

**Errors:**
| Status | error code | When |
|---|---|---|
| 400 | `validation_error` | invalid email, short password, empty name |
| 409 | `email_already_registered` | email already used (case-insensitive) |

---

### POST /api/auth/login
See FR-UC03-04, FR-UC03-05, FR-UC03-06.

**Auth:** none

**Request**:
```json
{ "email": "amina@example.com", "password": "at-least-8-characters" }
```

**Response — 200 OK**:
```json
{
  "access_token": "eyJhbGciOi...",
  "token_type": "bearer",
  "expires_in": 3600
}
```

**Errors:**
| Status | error code | When |
|---|---|---|
| 401 | `invalid_credentials` | wrong password OR unknown email — same message both cases (FR-UC03-05) |
| 429 | `too_many_attempts` | 5 failed attempts for this email within 15 minutes (FR-UC03-06) |

---

## 2. Analyses

All endpoints below require `Authorization: Bearer <token>`.
Missing/expired/invalid token → **401** `unauthorized`.

### POST /api/analyses
Uploads an image and starts an analysis. See FR-UC05-01 through
FR-UC05-24, use_case.md UC-05.

**Auth:** required. Must also have accepted the privacy notice
(FR-UC05-22), otherwise **403** `privacy_notice_not_accepted`.

**Request** (`multipart/form-data`):
| field | type | required | notes |
|---|---|---|---|
| `image` | file | yes | JPEG/PNG/WEBP, max 10 MB |
| `description` | string | no | max 500 characters |

**Response — 202 Accepted**:
```json
{
  "id": 42,
  "status": "PENDING",
  "created_at": "2026-09-27T10:05:00Z"
}
```
Returned in under 2 seconds (NFR-PERF-02), before the AI has answered.

**Errors (rejected at upload — nothing stored, no analysis created):**
| Status | error code | Requirement |
|---|---|---|
| 415 | `unsupported_format` | FR-UC05-05 |
| 413 | `file_too_large` | FR-UC05-06 |
| 400 | `unreadable_image` | FR-UC05-07 |
| 400 | `description_too_long` | FR-UC05-08 |
| 429 | `daily_quota_exceeded` | FR-UC05-09 (message states reset time) |
| 403 | `privacy_notice_not_accepted` | FR-UC05-22 |
| 503 | `daily_capacity_reached` | FR-UC05-23 (global cap) |

---

### GET /api/analyses
Lists the current user's analyses, newest first. See FR-UC02-01.

**Auth:** required.

**Query params:** none in V1 (pagination may be added later; not required for ~20 reference images / small personal usage).

**Response — 200 OK**:
```json
{
  "items": [
    {
      "id": 42,
      "status": "COMPLETED",
      "thumbnail_url": "/api/images/17/thumbnail",
      "created_at": "2026-09-27T10:05:00Z"
    }
  ]
}
```
Never includes another user's analyses (enforced server-side by
filtering on `user_id`, not by trusting a query param).

---

### GET /api/analyses/{id}
Reads one analysis in full. See FR-UC02-02, FR-UC02-03, FR-UC02-04, FR-UC02-05, FR-UC02-07.

**Auth:** required. Ownership enforced.

**Response — 200 OK (COMPLETED)**:
```json
{
  "id": 42,
  "status": "COMPLETED",
  "description": "Summer dress, linen if possible",
  "image_url": "/api/images/17",
  "components": [
    {
      "name": "bodice",
      "observed": "fitted bodice with a V-neckline",
      "hypothesis": "darts or princess seams shape the waist",
      "evidence": "smooth fit at bust and waist, no visible gathering",
      "confidence": "medium"
    }
  ],
  "probable_materials": [ /* same item shape, + "component" field */ ],
  "structure": [ /* same item shape, + "parent" field */ ],
  "hypotheses": [
    { "hypothesis": "...", "evidence": "...", "confidence": "low" }
  ],
  "manufacturing_steps": [
    { "order": 1, "title": "Cut the panels", "description": "..." }
  ],
  "limitations": [
    "The analysis is based on a single view; back and lining are not visible."
  ],
  "created_at": "2026-09-27T10:05:00Z",
  "completed_at": "2026-09-27T10:05:22Z"
}
```
The full shape of `components` / `probable_materials` / `structure` /
`hypotheses` / `manufacturing_steps` / `limitations` follows the AI
output schema fixed in **ADR-001, section 6** — this file does not
redefine it, only references it, so the two never drift apart.

**Response — 200 OK (PENDING / PROCESSING)**:
```json
{ "id": 42, "status": "PROCESSING", "description": "...", "image_url": "/api/images/17" }
```
(no analysis fields yet — frontend keeps polling, FR-UC05-02)

**Response — 200 OK (FAILED)**:
```json
{
  "id": 42,
  "status": "FAILED",
  "error_message": "The analysis took too long. Please try again.",
  "retry_type": "same_image",
  "image_url": "/api/images/17"
}
```
`retry_type` is `"same_image"` or `"another_image"` (see use_case.md,
section 3, "What retry means").

**Errors:**
| Status | error code | When |
|---|---|---|
| 404 | `analysis_not_found` | not owned by the user, or does not exist — same response both cases (FR-UC02-07) |

---

### POST /api/analyses/{id}/retry
Retries a FAILED analysis with the same stored image. See FR-UC05-19.

**Auth:** required. Ownership enforced. Only valid when `retry_type`
of the target analysis is `"same_image"`.

**Response — 202 Accepted**:
```json
{ "id": 43, "status": "PENDING", "retry_of_analysis_id": 42 }
```

**Errors:**
| Status | error code | When |
|---|---|---|
| 404 | `analysis_not_found` | not owned / does not exist |
| 409 | `retry_not_applicable` | target analysis failed for an image reason, not a system reason |
| 409 | `max_retries_reached` | already retried 3 times for this image (FR-UC05-19) |

---

### GET /api/images/{id}
Serves the raw image file (used as `image_url` above). See NFR-SEC-06.

**Auth:** required. Ownership enforced — no public image URL in V1.

**Response — 200 OK**: binary image data with correct `Content-Type`.

**Errors:**
| Status | error code | When |
|---|---|---|
| 404 | `image_not_found` | not owned / does not exist |

---

## 3. Status Code Summary

| Status | Meaning in this API |
|---|---|
| 200 | Successful read |
| 201 | Account created |
| 202 | Accepted for background processing (analysis submitted or retried) |
| 400 | Validation error (body) |
| 401 | Missing/invalid/expired token, or wrong login |
| 403 | Privacy notice not accepted |
| 404 | Resource not found or not owned (never distinguishes the two) |
| 409 | Conflict (email taken, retry not applicable, max retries) |
| 413 | File too large |
| 415 | Unsupported file type |
| 429 | Rate-limited (login attempts or daily quota) |
| 503 | Global daily AI capacity reached |

## 4. Notes

- No pagination, filtering, or sorting params in V1 beyond
  "newest first" — matches the small reference-image-scale usage
  described in PROJECT.md and keeps the contract simple to implement
  and test with `pytest` + FastAPI's `TestClient` (all free, no
  external service needed to run the test suite, per NFR-TEST-01).
- CORS: only the configured frontend origin is allowed (NFR-SEC-07) —
  set via an environment variable, not hardcoded, so it costs nothing
  to change between local dev and any future environment.
