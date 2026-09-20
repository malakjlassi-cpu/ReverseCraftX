# ReverseCraftX

**Turn an inspiration picture into a clear, honest blueprint of how the garment is made.**

ReverseCraftX is a visual reverse-engineering assistant for physical designs. Upload a picture of a garment and get a structured analysis (components, probable materials, structure, manufacturing steps) that clearly separates what is **visible** from what is **inferred**.

> **Status:** Early design phase. Documentation first, no runnable code yet.
>
> **V1 budget: 0 €.** V1 is built with free and open-source tools, uses only a free AI tier (or a local model), and runs locally. No payment card is needed.

![ReverseCraftX preview (mockup)](docs/image/preview-placeholder.svg.png)

*Mockup of the V1 interface. Replace it with a real screenshot once the app runs.*

## Table of contents

1. [Live demo](#live-demo)
2. [How it works](#how-it-works)
3. [V1 scope](#v1-scope)
4. [Technologies used](#technologies-used)
5. [Configuration and customization](#configuration-and-customization)
6. [Documentation](#documentation)
7. [Installation](#installation)
8. [Roadmap](#roadmap)
9. [License](#license)

## Live demo

Not planned for V1: V1 runs locally to stay 100 % free. A public demo may come later if a free hosting and AI option exists.

<!-- Live demo: https://your-demo-url.example (V1.1 or later) -->

## How it works

1. **Sign in** and accept the privacy notice (first analysis only). Only signed-in users can run an analysis; this protects the limited free AI capacity.
2. **Upload** an image of a garment (JPEG, PNG or WEBP, up to 10 MB), with an optional text description.
3. **The backend validates** the file (real type, size), stores it, and creates an analysis with the status `PENDING`. It answers immediately.
4. **The analysis runs in the background** (`PROCESSING`): an AI model looks at the image and returns structured data.
5. **The frontend polls** the analysis status every few seconds and shows "Analysis in progress...".
6. **The result is displayed** (`COMPLETED`), or a clear error with a retry option (`FAILED`).

```mermaid
sequenceDiagram
    participant U as User (browser)
    participant B as Backend (FastAPI)
    participant S as Image storage
    participant A as Analysis service
    participant M as AI model
    U->>B: Upload image (+ optional description)
    B->>S: Save validated image
    B-->>U: 202 Accepted (analysis id, PENDING)
    B->>A: Start analysis in background
    A->>M: Send image
    M-->>A: Structured result
    U->>B: GET analysis status (polling)
    B-->>U: PROCESSING
    A->>B: Save result (COMPLETED or FAILED)
    U->>B: GET analysis status
    B-->>U: COMPLETED + result
```

Status flow: `PENDING` -> `PROCESSING` -> `COMPLETED` or `FAILED`.

### Core principle: visible vs. inferred

The system never presents a guess as a fact. Each item of an analysis separates what is seen from what is assumed, with evidence and a confidence level. Indicative example (final schema defined in the AI decision record):

```json
{
  "materials": [
    {
      "observed": "glossy surface with soft folds",
      "hypothesis": "satin or polyester satin",
      "evidence": "light reflections, fluid drape",
      "confidence": "medium"
    }
  ]
}
```

## V1 scope

> *A signed-in user uploads an image of a garment and receives a structured analysis.*

**Included in V1**
- Sign up and sign in (JWT)
- Image upload with validation (JPEG, PNG, WEBP, max 10 MB) and optional description
- Asynchronous analysis with visible status (`PENDING`, `PROCESSING`, `COMPLETED`, `FAILED`)
- Structured result: components, probable materials, structure, hypotheses with uncertainty, high-level manufacturing steps
- Error handling with retry (unreadable image, non-garment image, timeout)
- Consultation of my own analyses
- Privacy notice before the first analysis (the image is sent to a third-party AI service)
- 100 % free stack: no paid service, no payment card, runs locally

**Not in V1** (planned later)
- Publishing designs, search, comments, saved articles, reporting, profile editing
- Finding or buying the original product, price or supplier comparison
- Exact measurements from a single image
- Ready-to-use sewing patterns
- Domains other than clothing
- Paid services, public hosting and public demo (V1 is 100 % free and local)

Full details: [`docs/PROJECT.md`](docs/PROJECT.md).

## Technologies used

| Layer | Technology |
|---|---|
| Frontend | HTML5, CSS3, vanilla JavaScript (Fetch API) |
| Backend | Python, FastAPI (async), Pydantic |
| Database | MySQL, SQLAlchemy (ORM) |
| Image storage | Local file system (`storage/images/`) in V1 |
| AI / analysis | Multimodal AI model called through a **free tier** (or a local model), structured JSON output, billing never activated *(final choice recorded in ADR-001)* |
| Authentication | JSON Web Tokens (JWT) |
| Background jobs | FastAPI `BackgroundTasks` in V1 |
| Testing | `pytest`, API tests |
| Quality and CI | `ruff`, GitHub Actions |
| Tooling | Git and GitHub, Python virtual environment (`venv`) |

Rationale for each choice: [`docs/tech_stack.md`](docs/tech_stack.md).

## Configuration and customization

Settings live in a `.env` file (copy `.env.example`, never commit the real `.env`). Names below are indicative and will be finalized with the code.

| Variable | Purpose | Example |
|---|---|---|
| `DATABASE_URL` | MySQL connection | `mysql+pymysql://user:pass@localhost/reversecraftx` |
| `JWT_SECRET` | Token signing key | *(long random string)* |
| `JWT_EXPIRE_MINUTES` | Token lifetime | `60` |
| `AI_API_KEY` | Key of the AI provider | *(secret)* |
| `AI_MODEL` | Model used for analysis | *(defined in ADR-001)* |
| `ANALYSIS_TIMEOUT_SECONDS` | Max time for one analysis | `60` |
| `MAX_UPLOAD_MB` | Max image size | `10` |
| `ANALYSIS_QUOTA_PER_USER` | Analyses allowed per user per day | `10` |
| `ANALYSIS_GLOBAL_DAILY_CAP` | AI calls allowed per day for the whole app, kept below the provider's free limit (required) | *(set from the provider's free limit)* |
| `STORAGE_PATH` | Where images are saved | `storage/images/` |

What you can customize:
- **AI model and provider:** the analysis service sits behind an interface, so the model can be replaced without touching the rest of the backend.
- **Limits:** upload size, timeout, per-user quota and global daily cap (to stay within the AI free tier).
- **Storage:** local disk in V1, designed to be swapped for a cloud service later.
- **Language:** English in V1. French and Arabic are candidates for later.

## Documentation

| File | Content | Status |
|---|---|---|
| [`docs/PROJECT.md`](docs/PROJECT.md) | Vision, V1 scope, success criteria, risks | Done |
| [`docs/use_case.md`](docs/use_case.md) | Use cases | Done |
| [`docs/REQUIREMENTS.md`](docs/REQUIREMENTS.md) | Functional and non-functional requirements | Done |
| [`docs/adr/001-ia.md`](docs/adr/001-ia.md) | Decision record for the AI approach and output schema | In progress (draft) |
| [`docs/tech_stack.md`](docs/tech_stack.md) | Technology choices | Being revised |
| [`docs/architecture.md`](docs/architecture.md) | Components and analysis flow | Being revised |
| [`docs/DATA_MODEL.md`](docs/DATA_MODEL.md) | Database schema | Being revised |
| `docs/API.md` | Endpoints, errors, status transitions | To write |

## Installation

Coming soon. This section will cover prerequisites (Python, MySQL), virtual environment, dependencies, database migrations, configuration and running the tests (`pytest`).

## Roadmap

- **V1:** authentication, image analysis, consult my analyses (100 % free, runs locally)
- **V1.1:** publish a design, search, view an article
- **V1.2:** comments, saved articles, profile, reporting
- **V2+:** other domains (accessories, furniture), more languages

## License

To be defined.
