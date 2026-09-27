# Requirements — ReverseCraftX

**V1 in one sentence:** A signed-in user uploads an image of a garment
and receives a structured analysis that separates what is visible from
what is inferred.

This document lists the functional (FR) and non-functional (NFR)
requirements of V1. Every V1 requirement has at least one acceptance
criterion written as Given/When/Then, so it can become a test.

## 0. How to Read This Document

- `[V1]` = must be delivered now, has acceptance criteria. `[V1.1]` /
  `[V1.2]` = planned later, criteria written when that version starts.
- `FR-UCxx-nn` = requirement `nn` of use case `UCxx` (see use_case.md).
  `NFR-CATEGORY-nn` = non-functional requirement.
- Test tags: `[Unit]` (no network/DB), `[API]` (backend endpoints, AI
  mocked), `[E2E]` (through the browser), `[Manual]` (checklist),
  `[Benchmark]` (measured on the ~20-image reference set).
- Test naming includes the requirement ID, e.g.
  `test_fr_uc05_06_rejects_oversize_file`.
- The AI is always mocked in automated tests: no network, no cost,
  deterministic results.
- Values below (token lifetime, quotas, timeouts...) are **fixed
  decisions** for V1, not proposals — change them here first if you
  change your mind, then update the code and tests together.

## 1. Functional Requirements — V1

### UC-03 — Sign Up / Sign In

- **FR-UC03-01** Allow a visitor to create an account with first name, last name, email, password.
  - AC: given no account for `amina@example.com`, submitting valid sign-up fields creates the account, returns 201, and the response contains no password or hash. `[API]`
- **FR-UC03-02** Validate sign-up fields: email format, non-empty names, password ≥ 8 characters.
  - AC1: a 7-character password is refused with a field-level error. `[API]`
  - AC2: `not-an-email` is refused with a field-level error. `[API]`
  - AC3: an empty first name is refused. `[API]`
- **FR-UC03-03** Refuse sign-up when the email is already registered (case-insensitive).
  - AC: given `amina@example.com` exists, signing up with `AMINA@example.com` returns 409 and creates no second account. `[API]`
- **FR-UC03-04** Allow sign-in with email + password; return an access token valid for **60 minutes**.
  - AC: a correct sign-in returns 200 with a token expiring 60 minutes after issue. `[API]`
- **FR-UC03-05** On sign-in failure, return the same generic message whether the email is unknown or the password is wrong ("Invalid email or password", 401).
  - AC1/AC2: wrong password and unknown email both return the same status and message. `[API]`
- **FR-UC03-06** Limit failed sign-in attempts: **5 failures per email per 15 minutes**.
  - AC: a 6th attempt within 15 minutes returns 429 even with the correct password; after 15 minutes, a correct sign-in succeeds. `[API]` (controllable clock)
- **FR-UC03-07** Every analysis-related endpoint requires a valid token.
  - AC1–3: no token, expired token, or tampered token all return 401. `[API]`
- **FR-UC03-08** On session expiry (401), redirect to sign-in with an explanatory message; a visitor opening the analysis page is redirected too.
  - AC1/AC2: expired-token submission and visitor page access both redirect appropriately. `[E2E]`

### UC-05 — Analyze an Image

**Upload and processing**

- **FR-UC05-01** Accept an authenticated submission (image + optional description), create an `Analysis` with status `PENDING`.
  - AC1: valid 2 MB JPEG + description → 202, identifier, `PENDING`, records created. `[API]`
  - AC2: same without description → accepted the same way. `[API]`
- **FR-UC05-02** Run the analysis asynchronously: answer immediately, show "Analysis in progress...", poll every **3 seconds** until `COMPLETED`/`FAILED`, then stop.
  - AC1: submission answers in <2 s even with a slow AI mock. `[API]`
  - AC2–4: polling behavior (interval, stop on final status, stop on leaving page). `[E2E]` (fake timers)
- **FR-UC05-03** On success: store the result, set `COMPLETED`, make it readable with observed/inferred separated.
  - AC: with a valid AI mock result, status becomes `COMPLETED` and reading the analysis returns that result. `[API]`
- **FR-UC05-04** On failure during processing (non-garment, unusable, timeout, provider error, invalid output, interruption): set `FAILED`, store `error_message`, offer the applicable retry; server keeps running.
  - AC1–3: non-garment, unusable, and provider-error cases each produce the right message and retry option. `[API]` `[E2E]`
- **FR-UC05-16** AI timeout at **60 seconds** → `FAILED` with a timeout message.
  - AC: with a 1-second test timeout and a never-answering mock, status becomes `FAILED` within 3 s. `[API]`
- **FR-UC05-17** On app startup, any analysis left `PENDING`/`PROCESSING` becomes `FAILED` ("interrupted").
  - AC: given one PENDING, one PROCESSING, one COMPLETED at startup, only the first two become FAILED. `[API]`
- **FR-UC05-18** Analysis continues if the user leaves the page; result stays available in "My analyses".
  - AC: stop polling, reopen the list later → analysis shows COMPLETED and readable. `[API]`
- **FR-UC05-21** The optional description is stored and passed to the AI as context.
  - AC: description is stored and received by the mocked AI client. `[Unit]`

**Input validation (rejected at upload — nothing stored)**

- **FR-UC05-05** Refuse files whose real type isn't JPEG/PNG/WEBP (415, "Unsupported format. Use JPEG, PNG or WEBP."). Includes a mismatched extension. `[API]`
- **FR-UC05-06** Refuse files > **10 MB** (413, "Image too large (max 10 MB)."). `[API]`
- **FR-UC05-07** Refuse undecodable / extension-mismatched files (400, "This file could not be read as an image."). `[API]`
- **FR-UC05-08** Refuse a description > **500 characters** (validation error); exactly 500 is accepted. `[API]`

**Quota**

- **FR-UC05-09** Max **10 analyses per user per UTC calendar day**. An 11th submission returns 429 with the reset time; other users unaffected. `[API]` (controllable clock)
- **FR-UC05-10** Counting rules: a `FAILED`-by-system-error analysis (timeout, provider error/capacity, interruption) does **not** count. A `COMPLETED` analysis, or `FAILED`-by-image (non-garment, unusable), **does** count. An upload rejected before creation never counts. `[API]`

**Free-tier constraints**

- **FR-UC05-22** Show a privacy notice before the first analysis (image sent to a third party, may be reused on free plans, no identifiable people). Must be accepted to continue; acceptance stored with the account.
  - AC1–3: notice shown/hidden appropriately; direct submission without acceptance is refused (403). `[E2E]` `[API]`
- **FR-UC05-23** Enforce a **global daily cap** on AI calls across all users, set below the provider's free daily limit (target: at most 80 % of it). Every call counts, including retries and failed ones; rejected uploads don't.
  - AC1/AC2: cap reached → 503 with a clear message, nothing created; retries count toward the cap. `[API]` (controllable clock)
- **FR-UC05-24** On a provider rate-limit/quota error: `FAILED` with "The analysis service is busy or unavailable. Please try again later.", retry with the same image offered, user quota **not** consumed, **no automatic retry**.
  - AC1/AC2: message and unconsumed quota confirmed; mock called exactly once. `[API]` `[Unit]`

**Output format and observed/inferred separation**

- **FR-UC05-11** A `COMPLETED` result has: `components`, `probable_materials`, `structure`, `hypotheses`, `manufacturing_steps`, `limitations`. `components` and `manufacturing_steps` are non-empty. `[Unit]`
- **FR-UC05-12** Every item in `components`/`probable_materials`/`structure` has a non-empty `observed` and a `confidence` in {low, medium, high}; any `hypothesis` requires a non-empty `evidence`. Exact schema fixed in ADR-001. `[Unit]` `[Benchmark]` `[Manual]`
- **FR-UC05-13** A schema-invalid result is never stored as `COMPLETED` — the analysis becomes `FAILED` with "The analysis could not be produced correctly. Please try again." `[Unit]` `[API]`
- **FR-UC05-14** `limitations` is non-empty and states at least: single-view only; hidden parts (back, lining, inner seams) not observed unless visible. `[Unit]` `[Manual]`
- **FR-UC05-15** No exact measurements as facts (qualitative words only, e.g. "knee-length"). `[Benchmark]`

**Retry**

- **FR-UC05-19** For a temporary failure, allow a retry with the **same** stored image/description (new `Analysis`, original stays `FAILED`). Max **3 retries** per image.
  - AC1/AC2: retry creates a new PENDING analysis; a 4th retry on the same image is refused (409). `[API]`
- **FR-UC05-20** For an image-caused failure, offer a clear path to submit **another** image. `[E2E]`

### UC-02 — View My Analysis Result

- **FR-UC02-01** List the user's own analyses, newest first, with thumbnail/date/status. Never another user's. `[API]`
- **FR-UC02-02** Show image, description, and status on the analysis page. `[E2E]`
- **FR-UC02-03** For `COMPLETED`: show all result sections; observed/inferred visually distinct; each inference shows evidence + confidence. `[E2E]`
- **FR-UC02-04** For `PENDING`/`PROCESSING`: show "Analysis in progress..." and keep polling (FR-UC05-02). `[E2E]`
- **FR-UC02-05** For `FAILED`: show the error message and the applicable retry option. `[E2E]`
- **FR-UC02-06** Empty state with a link to start an analysis when the user has none. `[E2E]`
- **FR-UC02-07** Only the owner can access an analysis; another user's analysis and a non-existent one return the same 404. `[API]`
- **FR-UC02-08** Fallback text if the image fails to load; analysis text stays readable. `[E2E]`
- **FR-UC02-09** Error message + retry action if loading the page data fails. `[E2E]`

## 2. Non-Functional Requirements — V1

**Performance**
- **NFR-PERF-01** ≤ 30 s for 90 % of analyses (experimental target); hard limit 60 s (FR-UC05-16). `[Benchmark]`
- **NFR-PERF-02** Submission endpoint answers in < 2 s server time for a 10 MB image. `[API]`
- **NFR-PERF-03** Status endpoint answers in < 500 ms for 95 % of requests. `[API]`

**Files**
- **NFR-FILE-01** Max image size **10 485 760 bytes**, enforced server-side without loading the whole file into memory. `[API]`
- **NFR-FILE-02** Accepted types: JPEG, PNG, WEBP only (see FR-UC05-05, NFR-SEC-02).

**Security**
- **NFR-SEC-01** Passwords stored only as a salted hash (argon2id or bcrypt); never logged or returned in clear. `[Unit]`
- **NFR-SEC-02** File type determined from content, never from extension/declared type alone. `[API]`
- **NFR-SEC-03** Stored images use random filenames; user-supplied names never build a path. `[API]`
- **NFR-SEC-04** Embedded metadata (EXIF/GPS) stripped from stored images. `[Unit]`
- **NFR-SEC-05** Secrets (JWT key, AI key, DB password) from environment config only; app refuses to start without the JWT key. `[Unit]` `[Manual]`
- **NFR-SEC-06** Analyses and images reachable only by their owner (no public URL). `[API]`
- **NFR-SEC-07** CORS allows only the configured frontend origin. `[API]`
- **NFR-SEC-08** All user/AI text displayed as plain text, never interpreted as HTML. `[E2E]`
- **NFR-SEC-09** Parameterized queries only (via ORM). `[API]`
- **NFR-SEC-10** `[V1.1, once deployed]` HTTPS only in any deployed environment (V1 runs locally, not deployed).

**Compatibility**
- **NFR-COMP-01** Works on the latest two stable versions of Chrome, Firefox, Edge, Safari (desktop). `[Manual]`
- **NFR-COMP-02** Layout adapts from 360 px to 1920 px, no horizontal scroll; mobile browsers best-effort. `[Manual]`

**Reliability**
- **NFR-REL-01** A failure in one analysis never stops the server or affects other requests. `[API]`

**API**
- **NFR-API-01** Every error response uses `{"error": "<code>", "message": "<text>"}`. `[API]`

**Language**
- **NFR-LANG-01** Interface and analysis content in English. `[Benchmark]` `[Manual]`

**Testability**
- **NFR-TEST-01** AI client hidden behind an interface; the whole automated suite runs without network or AI key. `[Unit]` `[API]`
- **NFR-TEST-02** A ~20-image reference set exists, each with a recorded expected outcome. `[Manual]`

**Cost**
- **NFR-COST-01** V1 buildable/testable/runnable at 0 €, no payment card required at any step. `[Manual]`
- **NFR-COST-02** The global daily cap (FR-UC05-23) is a required setting; app refuses to start without it. `[Unit]`

**Observability**
- **NFR-LOG-01** Every status change logged with the analysis ID and duration; logs never contain passwords/tokens/keys. `[API]`
- **NFR-LOG-02** Each analysis records the AI model version and processing duration (and usage figures if provided), to track free-tier consumption. `[API]`

## 3. Requirements for Later Versions (not detailed yet)

- **UC-01 — Search Designs** [V1.1]: keyword search, result cards, click-through, "no results" message.
- **UC-04 — Publish a Design** [V1.1]: to be written once the image-copyright question is settled.
- **UC-11 — View Public Article** [V1.1]: view image, description, AI analysis, author on a public article.
- **UC-08 — Add a Comment** [V1.2]
- **UC-10 — Report an Article** [V1.2]
- **UC-06, UC-07, UC-09** [V1.2]: save article, view saved, update profile.

## 4. Traceability

Every `[V1]` requirement has an acceptance criterion, and automated
test names include the requirement ID. Build/test order:

```
FR-UC03 (auth)
   → FR-UC05-01/05/06/07/08/22/23 (upload)
   → FR-UC05-02/03/04/16/17/24 (async analysis)
   → FR-UC05-11…15 (result format)
   → FR-UC02 (display)
   → FR-UC05-19/20 (retry)
```

## 5. Fixed Values (V1)

| Item | Value |
|---|---|
| Token lifetime | 60 minutes |
| Failed sign-in limit | 5 per email per 15 minutes |
| Minimum password length | 8 characters |
| Status polling interval | 3 seconds |
| AI time limit | 60 seconds |
| Description maximum | 500 characters |
| Daily quota | 10 analyses per user, per UTC calendar day |
| Maximum retries per image | 3 |
| Target time | 30 s for 90 % of analyses (experimental) |
| Extension mismatch | Refused (strict) |
| Global daily cap | At most 80 % of the provider's free daily limit (exact value set after ADR-001) |
| Cap-reached response | 503, "daily capacity" message |
| Provider capacity error | FAILED, no auto-retry, user quota not consumed |
| Privacy notice | Shown before first analysis, acceptance stored with the account |
