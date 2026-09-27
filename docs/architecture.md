# Architecture — ReverseCraftX (V1)

## 1. Architecture Overview

ReverseCraftX V1 is organized around a **single FastAPI backend** that
orchestrates everything: HTTP requests, database access, image storage,
and the analysis pipeline. There is **no separate analysis server** and
**no message queue** in V1 — asynchronous processing is done with
**FastAPI `BackgroundTasks`**, run inside the same process as the API.

```
                     USER
                       │
                       │ interaction
                       ▼
                ┌──────────────┐
                │   FRONTEND   │
                │  Interface   │
                └──────┬───────┘
                       │
                 HTTP / API
                       │
                       ▼
                ┌────────────────────────────┐
                │      FASTAPI BACKEND       │
                │                            │
                │  ┌──────────┐              │
                │  │ Routers  │              │
                │  └────┬─────┘              │
                │       │                    │
                │  ┌────▼─────┐              │
                │  │ Services │              │
                │  └────┬─────┘              │
                │       │                    │
                │  ┌────▼──────────┐         │
                │  │ Repositories  │         │
                │  └────┬──────────┘         │
                └───────┼────────────┬───────┘
                         │            │
              ┌──────────┼────────────┼──────────────┐
              │          │            │              │
              ▼          ▼            ▼              ▼
        ┌───────────┐ ┌───────────┐ ┌─────────────────────┐
        │  MySQL    │ │  Image    │ │  FastAPI             │
        │ Database  │ │  Storage  │ │  BackgroundTasks     │
        └───────────┘ │(local FS) │ │        │             │
                       └───────────┘ │        ▼             │
                                     │  AnalysisService     │
                                     │        │             │
                                     │        ▼             │
                                     │   AI Provider Client │
                                     └───────────────────────┘
```

There is no second server, no worker pool, and no Redis/Celery in V1.
Everything above the dashed line runs inside one FastAPI process.

## 2. System Components (V1)

### 2.1 User
A person using ReverseCraftX. In V1 there are two actor states:
- **Visitor**: can only reach sign-up / sign-in.
- **Authenticated User**: can upload an image, run an analysis, and
  view their own analysis history.

### 2.2 Frontend
Vanilla HTML/CSS/JS. For V1, the Frontend allows the user to:

- Sign up
- Sign in
- Upload a garment image (with an optional text description)
- View analysis status ("Analysis in progress...", polling)
- View analysis results (components, materials, structure,
  hypotheses, manufacturing steps, limitations)
- View previous analyses ("My analyses")
- Retry a failed analysis (same image or another image)

> **Future versions (not V1):** search designs, publish/view public
> articles, add comments, save articles, update profile, report
> content. These belong to V1.1 / V1.2 (see PROJECT.md, roadmap) and
> are intentionally excluded from the V1 architecture above.

The Frontend never talks to the database, image storage, or the AI
provider directly — everything goes through the Backend's HTTP API.

### 2.3 FastAPI Backend
The single application process. Internally organized in layers:

- **API / Routers** — receive HTTP requests, validate input shape
  (Pydantic schemas), return HTTP responses.
- **Services** — business logic (quota checks, privacy-notice check,
  retry rules, orchestrating upload + analysis).
- **Repositories** — the only layer that talks to MySQL (via the ORM).
- **Core** — configuration, security (JWT), database connection setup.

Example call path for `POST /analyses`:

```
POST /analyses
      │
      ▼
AnalysisRouter
      │
      ▼
AnalysisService ── ImageService ── ImageRepository ── MySQL
      │                                  │
      │                                  ▼
      │                           Image Storage (local FS)
      │
      ▼
AnalysisRepository ── MySQL
      │
      ▼
BackgroundTasks.add_task(run_analysis, analysis_id)
      │
      ▼
   (returns 202 immediately, analysis_id, status=PENDING)
```

The background task then does:

```
BackgroundTask
      │
      ▼
AnalysisService.process(analysis_id)
      │
      ▼
AIProviderClient.analyze(image, description)
      │
      ▼
validate result against schema (ADR-001, section 6)
      │
      ├── valid  → AnalysisRepository.mark_completed(result)
      └── invalid/error → AnalysisRepository.mark_failed(error_message)
```

### 2.4 Database (MySQL)
Stores structured V1 data: `User`, `Image`, `Analysis`.
See DATA_MODEL.md for the exact V1 schema (kept intentionally small —
`Article`, `Comment`, `SavedArticle`, `Report` are V1.1/V1.2 only).

### 2.5 Image Storage
Local filesystem (`storage/images/`) for raw image files, referenced
by path/filename from the `Image` table. Separated from the database
because images are binary files, not relational metadata.

### 2.6 Analysis pipeline (BackgroundTasks, not a separate server)
Manages the state machine:

```
PENDING ──> PROCESSING ──> COMPLETED
   │              │
   └──────────────┴──────> FAILED
```

Runs as a Python function scheduled via `BackgroundTasks` inside the
same FastAPI process — not a separate service, not a network hop.

### 2.7 AI Provider Client
A thin client (`AIProviderClient` / `AnalysisClient`) wrapping calls to
the external multimodal AI API chosen in ADR-001. Kept behind an
interface so:
- it can be mocked in all automated tests (no network, no key needed),
- the provider can be swapped later without touching business logic.

## 3. Main Communication Flow

```
User → Frontend → HTTP/API → FastAPI Backend
                                  ├── MySQL (User, Image, Analysis)
                                  ├── Image Storage (local FS)
                                  └── BackgroundTasks
                                          └── AnalysisService
                                                  └── AI Provider Client
```

## 4. Key Engineering Principles

- **Separation of concerns**: routers handle HTTP, services handle
  business rules, repositories handle persistence, the AI client
  handles the external call — nothing else touches MySQL or the AI
  provider directly.
- **No premature infrastructure**: no separate server, no message
  broker, no worker pool in V1. `BackgroundTasks` is sufficient at this
  scale and is free, since it needs no extra service to run or host.
- **Everything free by construction**: MySQL Community, local file
  storage, and FastAPI's built-in `BackgroundTasks` all run on the same
  machine with no paid service, matching the 0 € constraint in
  PROJECT.md.
