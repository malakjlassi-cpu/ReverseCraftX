# Technical Stack & Design Choices — ReverseCraftX (V1)

This document outlines the technology stack and architectural decisions chosen for the first version (V1) of ReverseCraftX, balancing technical rigor, pedagogical value, and development efficiency.

---

## 1. Summary of V1 Technology Stack

| Component | V1 Chosen Technology | Rationale |
| :--- | :--- | :--- |
| **Frontend** | HTML5, CSS3, Vanilla JavaScript (Fetch API) | Perfect for mastering end-to-end HTTP requests, DOM manipulation, and avoiding heavy framework abstraction layers in V1. |
| **Backend** | Python + FastAPI | High performance, native async support, and seamless integration with the Python AI / Computer Vision ecosystem. |
| **Database** | MySQL | Robust, reliable relational database, well-suited for structured data and straightforward SQL queries. |
| **Image Storage** | Local File System (`storage/images/`) | Simple file management for V1 before transitioning to cloud solutions (e.g., AWS S3, Cloudinary). |
| **AI / Analysis** | Python (Computer Vision models) | Direct integration with FastAPI for processing image analysis and handling asynchronous status pipelines. |
| **Authentication** | JSON Web Tokens (JWT) | Stateless, secure authentication standard for REST APIs. |
| **Version Control** | Git & GitHub | Source code management and remote repository hosting. |
| **Testing** | `pytest` & API testing | Automated verification of backend logic and endpoints. |
| **Environment** | Python Virtual Environment (`venv`) | Isolated dependency management without containerization overhead in early stages. |

---

## 2. Detailed Architecture & Design Decisions

### 2.1 Frontend Architecture
* **Vanilla Approach:** Using plain HTML, modular CSS, and vanilla JS (`fetch`) allows a deep understanding of the request-response lifecycle between the client and the FastAPI backend.
* **Structure:**
  ```text
  frontend/
  ├── index.html
  ├── css/
  └── js/
      ├── api.js
      ├── search.js
      ├── article.js
      ├── auth.js
      └── analysis.js
2.2 Backend Architecture (FastAPI)
Modular Pattern: Organized to separate routing, business logic, schemas, and data access, preventing monolithic code files.

Structure:

Plaintext
backend/
├── app/
│   ├── api/          # Routers and endpoints
│   ├── models/       # Database ORM models (SQLAlchemy)
│   ├── schemas/      # Pydantic validation schemas
│   ├── services/     # Business logic layer
│   ├── repositories/ # Data access layer
│   ├── core/         # Security, configuration, database connection
│   └── main.py       # Application entry point
└── tests/            # Pytest suite
2.3 Asynchronous Analysis Pipeline
To handle AI processing latency gracefully, the analysis workflow (UC-05) follows an explicit state machine:

Plaintext
User Upload ──> FastAPI ──> PENDING ──> PROCESSING ──> COMPLETED / FAILED
States: PENDING, PROCESSING, COMPLETED, FAILED.

Error Handling: If an error occurs (timeout, invalid image format, or non-garment detection), the system catches the exception, updates the status to FAILED, stores the error_message, and allows the user to retry.

2.4 API Design & HTTP Status Codes
The backend exposes RESTful endpoints adhering to standard HTTP status codes:

200 OK / 201 Created: Successful requests/creations.

400 Bad Request: Validation errors or malformed payloads.

401 Unauthorized: Missing or invalid authentication token.

403 Forbidden: Insufficient permissions for the requested action.

404 Not Found: Target resource does not exist.

500 Internal Server Error: Unexpected server or AI pipeline failure
