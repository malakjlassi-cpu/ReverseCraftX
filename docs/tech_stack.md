# Technical Stack — ReverseCraftX (V1)

All choices are free/open-source, matching the 0 € budget in PROJECT.md.

## 1. V1 Stack

| Component | V1 choice | Why |
|---|---|---|
| Frontend | HTML5, CSS3, vanilla JavaScript (Fetch API) | Learn the request/response cycle directly, no framework overhead |
| Backend | Python + FastAPI | Async support, easy AI-provider integration, good for a beginner |
| Database | MySQL (Community) | Free, reliable, simple relational queries |
| Image storage | Local filesystem (`storage/images/`) | No setup cost; cloud storage can come later |
| Analysis processing | FastAPI `BackgroundTasks` (in-process) — **not** a separate server, **not** Celery/Redis in V1 | Enough for V1's scale; adding a worker/queue is a later optimization, not a requirement now |
| AI analysis | External multimodal (vision + language) API, free tier only — see ADR-001 | Chosen in ADR-001 over classical computer vision, which can't reason about materials or explain evidence in words |
| Authentication | JSON Web Tokens (JWT) | Stateless, standard for a REST API |
| Version control | Git & GitHub | Free hosting, standard workflow |
| Testing | pytest + FastAPI `TestClient` | AI client mocked — no network or key needed to run tests (NFR-TEST-01) |
| Environment | Python `venv` | No containerization overhead needed yet |

## 2. Backend Structure

```
backend/
├── app/
│   ├── api/            # Routers / endpoints
│   ├── core/            # Config, security (JWT), DB connection
│   ├── models/          # SQLAlchemy ORM models
│   ├── schemas/          # Pydantic request/response schemas
│   ├── services/         # Business logic (quota, retry rules, orchestration)
│   ├── repositories/      # Data access layer (only layer touching MySQL)
│   └── main.py           # App entry point
└── tests/                # pytest suite
```

Call path for a typical request:

```
Router (HTTP in/out)
   → Service (business logic)
       → Repository (MySQL) and/or AI client
```

Routes never contain business logic directly, and only repositories
talk to the database — this keeps the app testable without a real
database or a real AI key.

## 3. Frontend Structure

```
frontend/
├── index.html
├── css/
└── js/
    ├── api.js       # fetch wrapper, base URL, auth header
    ├── auth.js      # sign up / sign in
    └── analysis.js  # upload, polling, result display
```

(`search.js` / `article.js` are not part of V1 — they belong to V1.1
once UC-01/UC-04/UC-11 exist.)

## 4. Asynchronous Analysis Pipeline

```
User Upload → FastAPI → PENDING → (BackgroundTasks) → PROCESSING → COMPLETED / FAILED
```

No separate analysis server: the background task runs inside the same
FastAPI process (see architecture.md). On any error (timeout, invalid
image, non-garment, invalid AI output), the task catches it, sets
`FAILED`, and stores `error_message` — the server keeps running for
other requests (NFR-REL-01).

## 5. API Design Conventions

RESTful endpoints; contracts are fixed in `API.md`. Standard status
codes used across the API:

| Code | Meaning |
|---|---|
| 200 / 201 / 202 | Success / created / accepted for background processing |
| 400 | Validation error / malformed payload |
| 401 | Missing or invalid token |
| 403 | Forbidden (e.g. privacy notice not accepted) |
| 404 | Resource not found (or not owned — same response either way) |
| 409 | Conflict (e.g. duplicate email, max retries reached) |
| 413 / 415 | File too large / unsupported file type |
| 429 | Rate-limited (login attempts, daily quota) |
| 503 | Global daily AI capacity reached |

Every error body follows `{"error": "<code>", "message": "<text>"}`
(NFR-API-01).

## 6. What This Stack Deliberately Excludes (for now)

- No message broker / task queue (Celery, Redis) — add only if
  `BackgroundTasks` becomes a real bottleneck.
- No containerization (Docker) — not needed to run one FastAPI
  process plus MySQL locally.
- No frontend framework — plain JS is enough for V1's few pages.
- No cloud storage / paid hosting — local filesystem, local run only.
