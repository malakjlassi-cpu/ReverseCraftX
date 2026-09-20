# Requirements — ReverseCraftX

> **Status:** Draft v2 — aligned with [`PROJECT.md`](PROJECT.md) and [`use_case.md`](use_case.md).
> **V1 in one sentence:** *A signed-in user uploads an image of a garment and receives a structured analysis that separates what is visible from what is inferred.*

This document lists the functional requirements (FR) and non-functional requirements (NFR) of ReverseCraftX. Every **V1** requirement comes with at least one **acceptance criterion** written as *Given / When / Then*, so that it can become a test.

---

## 0. How to Read This Document

### Version labels

| Label | Meaning |
|---|---|
| **[V1]** | Must be delivered in the first version. Has acceptance criteria. |
| **[V1.1]** | Planned for the next version. Acceptance criteria are written when the version starts. |
| **[V1.2]** | Planned later. Same rule. |

### Identifiers

* **FR-UCxx-nn**: functional requirement number `nn` of use case `UCxx` (see `use_case.md`).
* **NFR-CATEGORY-nn**: non-functional requirement (`PERF`, `FILE`, `SEC`, `COMP`, `REL`, `API`, `LANG`, `TEST`, `LOG`).
* **AC**: acceptance criterion attached to a requirement.

### Wording

* **must** = mandatory for the version. **should** = recommended, may be dropped if time is short.

### Test level tags

| Tag | Meaning |
|---|---|
| `[Unit]` | Automated unit test (no network, no database if possible). |
| `[API]` | Automated test calling the backend endpoints, with the AI client mocked. |
| `[E2E]` | Test through the browser interface (automated or scripted). |
| `[Manual]` | Checked by a person, with a written checklist. |
| `[Benchmark]` | Measured on the reference image set (about 20 images). |

### Conventions

* **Test naming:** automated tests include the requirement ID in their name (for example `test_fr_uc05_06_rejects_oversize_file`), so every requirement can be traced to a test.
* **HTTP status codes** and **values marked *(proposed)*** are starting points. Final endpoint contracts are fixed in `API.md`; values are confirmed in section 5.
* **The AI is always mocked** in automated tests: no network, no cost, deterministic results.

### Changes from draft v1

| Draft v1 | Now | Reason |
|---|---|---|
| FR-UC01-01 … 07 | Same IDs, **[V1.1]** | Search is out of V1. |
| FR-UC02-01 … 05 (View Article) | **FR-UC11-01 … 05** **[V1.1]** | Public article page moved to UC-11. |
| FR-UC02-06 (comment) | **FR-UC08-01** **[V1.2]** | Comments are their own use case. |
| FR-UC02-07 (report) | **FR-UC10-01** **[V1.2]** | Reporting is its own use case. |
| *(new)* | **FR-UC02-01 … 09** **[V1]** | UC-02 is now "View My Analysis Result". |
| *(new)* | **FR-UC03-01 … 08** **[V1]** | Authentication was missing. |
| FR-UC05-01 … 04 | Kept, **FR-UC05-02 and 04 corrected**, plus FR-UC05-05 … 21 | Polling made explicit; unreadable images are rejected at upload, not `FAILED`. |

---

## 1. Functional Requirements — V1

### UC-03 — Sign Up / Sign In

**FR-UC03-01 [V1]** The system must allow a visitor to create an account with first name, last name, email and password.
* **AC:** **Given** no account exists for `amina@example.com`, **when** a visitor submits the sign-up form with a first name, a last name, that email and a valid password, **then** the account is created, the response indicates success (201), and the response contains no password and no password hash. `[API]`

**FR-UC03-02 [V1]** The system must validate sign-up fields: email format, non-empty first and last names, password of at least 8 characters *(proposed)*.
* **AC1:** **Given** a visitor, **when** they submit a password of 7 characters, **then** the request is refused with a validation error on the password field and no account is created. `[API]`
* **AC2:** **Given** a visitor, **when** they submit `not-an-email` as email, **then** the request is refused with a validation error on the email field. `[API]`
* **AC3:** **Given** a visitor, **when** they leave the first name empty, **then** the request is refused with a validation error on that field. `[API]`

**FR-UC03-03 [V1]** The system must refuse a sign-up when the email is already registered (comparison is not case-sensitive).
* **AC:** **Given** an account exists for `amina@example.com`, **when** someone signs up with `AMINA@example.com`, **then** the system answers with a conflict (409) and no second account exists. `[API]`

**FR-UC03-04 [V1]** The system must allow a registered user to sign in with email and password and must return an access token with a limited lifetime *(proposed: 60 minutes)*.
* **AC:** **Given** a registered account, **when** the user signs in with the correct email and password, **then** the response is successful (200) and contains an access token whose expiry is 60 minutes after its issue time. `[API]`

**FR-UC03-05 [V1]** When sign-in fails, the system must return the same generic message whether the email is unknown or the password is wrong.
* **AC1:** **Given** a registered account, **when** the user signs in with a wrong password, **then** the answer is 401 with the message "Invalid email or password". `[API]`
* **AC2:** **Given** no account for `ghost@example.com`, **when** someone signs in with that email, **then** the answer has the same status and the same message as AC1. `[API]`

**FR-UC03-06 [V1]** The system must limit failed sign-in attempts *(proposed: 5 failures per email within 15 minutes)*.
* **AC:** **Given** 5 failed sign-in attempts for the same email within 15 minutes, **when** a 6th attempt is made, even with the correct password, **then** the answer is 429 and no token is issued; **and when** 15 minutes have passed, **then** a correct sign-in succeeds. `[API]` *(with a controllable clock)*

**FR-UC03-07 [V1]** Every endpoint that runs or reads analyses must require a valid token.
* **AC1:** **Given** no token, **when** a request is sent to submit an analysis, **then** the answer is 401. `[API]`
* **AC2:** **Given** an expired token, **when** a request is sent to list analyses, **then** the answer is 401. `[API]`
* **AC3:** **Given** a token whose signature was altered, **when** a request is sent to read an analysis, **then** the answer is 401. `[API]`

**FR-UC03-08 [V1]** When the session ends (401), the interface must redirect the user to the sign-in page with an explanatory message. A visitor who opens the analysis page must also be redirected.
* **AC1:** **Given** a user whose token has expired, **when** they submit an analysis, **then** they land on the sign-in page with the message "Your session has expired. Please sign in again." `[E2E]`
* **AC2:** **Given** a visitor (no token), **when** they open the analysis page address, **then** they are redirected to the sign-in page. `[E2E]`

---

### UC-05 — Analyze an Image

#### Upload and processing

**FR-UC05-01 [V1]** The system must allow an authenticated user to submit an image (JPEG, PNG or WEBP, maximum 10 MB) with an optional text description, and must create an analysis with status `PENDING`.
* **AC1:** **Given** a signed-in user with quota left and a valid 2 MB JPEG, **when** they submit it with the description "Summer dress, linen if possible", **then** the system answers 202 with an analysis identifier and status `PENDING`, and an image record and an analysis record linked to that user exist. `[API]`
* **AC2:** **Given** the same user, **when** they submit a valid image without a description, **then** the submission is accepted the same way. `[API]`

**FR-UC05-02 [V1] *(corrected)*** The system must run the analysis **asynchronously**: it must answer the submission immediately, without waiting for the AI. The interface must display a loading indicator ("Analysis in progress...") and **must query the analysis status at a regular interval** *(proposed: every 3 seconds)* until the status is `COMPLETED` or `FAILED`, then stop querying.
* **AC1:** **Given** an AI mock that answers after 10 seconds, **when** the user submits an image, **then** the submission answer (202, `PENDING`) is returned in under 2 seconds, before the AI has answered. `[API]`
* **AC2:** **Given** a submitted analysis in `PENDING` or `PROCESSING`, **when** the analysis page is open, **then** the text "Analysis in progress..." is visible and the interface sends a status request every 3 seconds (tolerance ±1 second). `[E2E]` *(with fake timers)*
* **AC3:** **Given** polling is running, **when** the status becomes `COMPLETED` or `FAILED`, **then** no further status request is sent and the indicator is replaced by the result or the error message. `[E2E]`
* **AC4:** **Given** polling is running, **when** the user leaves the page, **then** no further status request is sent. `[E2E]`

**FR-UC05-03 [V1]** When the analysis succeeds, the system must store the result, set the status to `COMPLETED`, and make the result available for display with observations and hypotheses clearly separated (see FR-UC05-12 and FR-UC02-03).
* **AC:** **Given** an AI mock returning a valid result, **when** processing ends, **then** the analysis status is `COMPLETED`, the result is stored, and reading the analysis returns that result. `[API]`

**FR-UC05-04 [V1] *(corrected)*** If the analysis fails **during processing** (non-garment image, unusable image, AI timeout, AI provider error, invalid AI output, interruption), the system must catch the error, set the status to `FAILED`, store an `error_message`, and display a clear message with the applicable retry option (FR-UC05-19 and FR-UC05-20). The server must keep running and other analyses must be unaffected.
* **AC1 (non-garment):** **Given** an AI mock reporting that no garment is visible, **when** processing ends, **then** the status is `FAILED`, the message is "No garment was detected in this image.", no result is stored, and the interface offers to try another image. `[API]` `[E2E]`
* **AC2 (unusable image):** **Given** an AI mock reporting that the garment cannot be analyzed reliably, **when** processing ends, **then** the status is `FAILED`, the message asks for a clearer image, and the interface offers to try another image. `[API]`
* **AC3 (provider error):** **Given** an AI mock that raises an error, **when** processing runs, **then** the status is `FAILED`, the message says the service is temporarily unavailable, and the interface offers a retry with the same image. `[API]`

**FR-UC05-16 [V1]** If the AI does not answer within the time limit *(proposed: 60 seconds)*, the system must set the status to `FAILED` with a timeout message.
* **AC:** **Given** an AI mock that never answers and a time limit set to 1 second for the test, **when** the analysis runs, **then** the status becomes `FAILED` within 3 seconds with the message "The analysis took too long. Please try again." `[API]`

**FR-UC05-17 [V1]** When the application starts, every analysis left in `PENDING` or `PROCESSING` must be set to `FAILED` with an "interrupted" message.
* **AC:** **Given** the database contains one `PENDING`, one `PROCESSING` and one `COMPLETED` analysis, **when** the application starts, **then** the first two become `FAILED` with the message "The analysis was interrupted. Please try again." and the `COMPLETED` one is unchanged. `[API]`

**FR-UC05-18 [V1]** The analysis must continue if the user leaves the page, and its result must remain available in "My analyses".
* **AC:** **Given** a submitted analysis, **when** the client stops polling and later opens the list of analyses after the AI mock has answered, **then** the analysis appears with status `COMPLETED` and its result is readable. `[API]`

**FR-UC05-21 [V1]** The optional description must be stored with the analysis and transmitted to the AI as context.
* **AC:** **Given** a submission with the description "in linen", **when** the analysis is processed, **then** the description is stored with the analysis and the AI client mock received it. `[Unit]`

#### Input validation (rejected at upload: nothing stored, no analysis created)

**FR-UC05-05 [V1]** The system must refuse files whose real type is not JPEG, PNG or WEBP.
* **AC1:** **Given** a signed-in user, **when** they upload a GIF or a PDF, **then** the answer is 415 with the message "Unsupported format. Use JPEG, PNG or WEBP.", no file is stored and no analysis record is created. `[API]`
* **AC2:** **Given** a text file renamed `photo.jpg`, **when** it is uploaded, **then** it is refused in the same way. `[API]`

**FR-UC05-06 [V1]** The system must refuse files larger than 10 MB (limit defined in NFR-FILE-01).
* **AC:** **Given** a valid JPEG of 11 MB, **when** it is uploaded, **then** the answer is 413 with the message "Image too large (max 10 MB).", nothing is stored and no analysis is created. `[API]`

**FR-UC05-07 [V1]** The system must refuse files that cannot be decoded as an image, or whose extension does not match their real type.
* **AC1:** **Given** a truncated, corrupted `photo.jpg`, **when** it is uploaded, **then** the answer is 400 with the message "This file could not be read as an image.", nothing is stored and no analysis is created. `[API]`
* **AC2:** **Given** a valid PNG file named `photo.jpg`, **when** it is uploaded, **then** it is refused with the same message. `[API]`

**FR-UC05-08 [V1]** The system must refuse a description longer than 500 characters *(proposed)*.
* **AC1:** **Given** a description of 501 characters, **when** the form is submitted, **then** the answer is a validation error mentioning the limit and nothing is created. `[API]`
* **AC2:** **Given** a description of exactly 500 characters, **when** the form is submitted, **then** it is accepted. `[API]`

#### Quota

**FR-UC05-09 [V1]** The system must limit each user to a maximum number of analyses per day *(proposed: 10, per calendar day in UTC)*.
* **AC:** **Given** a user who already started 10 analyses today, **when** they submit an 11th, **then** the answer is 429 with a message stating when the quota resets, nothing is stored and no analysis is created; **and** another user is not affected. `[API]` *(with a controllable clock)*

**FR-UC05-10 [V1]** Quota counting must follow these rules: an analysis that ends `FAILED` because of a **system error** (timeout, provider error, interruption) does **not** count; an analysis that ends `COMPLETED`, or `FAILED` because of the **image** (non-garment, unusable), **does** count; a request rejected at upload does **not** count.
* **AC1:** **Given** a quota of 10 with 9 analyses counted, **when** the 10th ends `FAILED` by timeout, **then** the counted usage stays at 9. `[API]`
* **AC2:** **Given** the same situation, **when** the 10th ends `FAILED` as non-garment, **then** the counted usage becomes 10. `[API]`
* **AC3:** **Given** a quota of 10 with 9 counted, **when** an upload is rejected for its format, **then** the counted usage stays at 9. `[API]`

#### Output format and observed / inferred separation

**FR-UC05-11 [V1]** A `COMPLETED` result must contain these sections: `components`, `probable_materials`, `structure`, `hypotheses`, `manufacturing_steps`, `limitations`. `components` and `manufacturing_steps` must contain at least one item. `hypotheses` holds general assumptions not attached to a single component.
* **AC:** **Given** a stored `COMPLETED` result, **when** it is validated against the result schema, **then** all six sections are present and `components` and `manufacturing_steps` are not empty. `[Unit]`

**FR-UC05-12 [V1]** In `components`, `probable_materials` and `structure`, every item must separate what is **observed** from what is **inferred**: it must have a non-empty `observed` text and a `confidence` of `low`, `medium` or `high`; any item that contains a `hypothesis` must also contain a non-empty `evidence`. The exact schema is fixed in the AI decision record (ADR-001); the rules above are the invariant.
* **AC1:** **Given** any valid result, **when** it is validated, **then** every item has a non-empty `observed` and a `confidence` in `{low, medium, high}`. `[Unit]`
* **AC2:** **Given** any valid result, **when** it is validated, **then** every item with a `hypothesis` has a non-empty `evidence`. `[Unit]`
* **AC3:** **Given** the results for the reference images, **when** all `observed` texts are scanned for hedging words (*probably, likely, might, possibly, maybe, seems*), **then** none is found; **and** a reviewer confirms on a sample that no assumption is written in `observed`. `[Benchmark]` `[Manual]`

**FR-UC05-13 [V1]** A result that does not match the schema must never be stored as `COMPLETED`: the analysis must become `FAILED`.
* **AC:** **Given** AI mocks returning (a) a result missing `confidence`, (b) text that is not JSON, (c) a `confidence` value of `certain`, **when** each is processed, **then** each analysis ends `FAILED` with the message "The analysis could not be produced correctly. Please try again.", no result is stored, and the retry offered is with the same image. `[Unit]` `[API]`

**FR-UC05-14 [V1]** The result must contain a non-empty `limitations` list stating what cannot be determined from the image (at least: the analysis is based on a single view; hidden parts such as the back, lining or seams are not observed unless visible).
* **AC1:** **Given** any valid result, **when** it is validated, **then** `limitations` is present and not empty. `[Unit]`
* **AC2:** **Given** reference images that show only the front of a garment, **when** a reviewer reads the results, **then** no `observed` text describes the back, lining or inner seams. `[Manual]`

**FR-UC05-15 [V1]** The result must not state exact dimensions of the garment (lengths, widths, sizes) as facts. Proportions may be described qualitatively (for example "knee-length").
* **AC:** **Given** the results for the reference images, **when** `observed` and `hypothesis` texts are scanned for a number followed by a unit (*cm, mm, m, in, inch*), **then** no match is found. `[Benchmark]`

#### Retry

**FR-UC05-19 [V1]** For a temporary failure (timeout, provider error, interruption, invalid output), the system must allow a **retry with the same image**: it creates a **new analysis** for the already stored image and description, without a new upload; the failed analysis stays unchanged. A maximum of 3 retries per image applies *(proposed)*.
* **AC1:** **Given** an analysis `FAILED` by timeout, **when** the user requests a retry, **then** a new analysis in `PENDING` is created for the same stored image, and the original stays `FAILED` with its `error_message` unchanged. `[API]`
* **AC2:** **Given** an image already retried 3 times, **when** a 4th retry is requested, **then** the answer is 409 with a message asking to try another image. `[API]`
* **AC3:** **Given** an analysis `FAILED` as non-garment, **when** its details are read, **then** it indicates that the applicable retry is with another image, not with the same image. `[API]`

**FR-UC05-20 [V1]** For a failure caused by the image (non-garment, unusable), the interface must offer a clear path to submit **another image**.
* **AC:** **Given** an analysis `FAILED` as non-garment, **when** the error is displayed, **then** a "Try another image" action leads to the upload form. `[E2E]`

---

### UC-02 — View My Analysis Result

**FR-UC02-01 [V1]** The system must list the authenticated user's analyses, newest first, with a thumbnail, the date and the status. It must never list another user's analyses.
* **AC:** **Given** user A has 3 analyses and user B has 2, **when** A opens "My analyses", **then** exactly A's 3 analyses are listed, newest first, each with thumbnail, date and status. `[API]`

**FR-UC02-02 [V1]** The analysis page must display the image, the optional description and the status.
* **AC:** **Given** an analysis owned by the user, **when** its page is opened, **then** the image, the description (if any) and the status are visible. `[E2E]`

**FR-UC02-03 [V1]** For a `COMPLETED` analysis, the page must display components, probable materials, structure, hypotheses, manufacturing steps and limitations. Observed content and inferred content must be **visually distinct** (different label and style), and each inference must show its evidence and confidence.
* **AC1:** **Given** a `COMPLETED` analysis, **when** the page is displayed, **then** each item shows an element labeled "Observed" and, when it has a hypothesis, a separate element labeled "Inferred" with its evidence and confidence. `[E2E]`
* **AC2:** **Given** the same page, **when** its content is inspected, **then** no inferred text appears inside an "Observed" element. `[E2E]`

**FR-UC02-04 [V1]** For an analysis in `PENDING` or `PROCESSING`, the page must display "Analysis in progress..." and keep checking the status as specified in FR-UC05-02.
* **AC:** **Given** an analysis in `PROCESSING`, **when** its page is opened, **then** "Analysis in progress..." is shown, and when the status becomes `COMPLETED` the result appears without manual refresh. `[E2E]`

**FR-UC02-05 [V1]** For a `FAILED` analysis, the page must display the error message and the applicable retry option (same image or another image).
* **AC:** **Given** an analysis `FAILED` by timeout, **when** its page is opened, **then** the message and a "Retry with the same image" action are shown. `[E2E]`

**FR-UC02-06 [V1]** When the user has no analyses, the list must show an empty-state message with a link to start an analysis.
* **AC:** **Given** a user with no analyses, **when** they open "My analyses", **then** a message is shown with a link to the analysis page. `[E2E]`

**FR-UC02-07 [V1]** The system must give access to an analysis only to its owner. A request for another user's analysis and a request for a non-existent analysis must produce the **same** "not found" answer.
* **AC:** **Given** an analysis owned by user A, **when** user B requests it, **then** the answer is 404 with the same body as for a random non-existent identifier. `[API]`

**FR-UC02-08 [V1]** If the image fails to load, the page must display fallback text while keeping the analysis text readable.
* **AC:** **Given** an analysis whose image address returns an error, **when** its page is opened, **then** fallback text replaces the image and all analysis sections are still displayed. `[E2E]`

**FR-UC02-09 [V1]** If loading the page data fails, the page must display an error message with a retry action.
* **AC:** **Given** the backend answers with an error, **when** the page loads, **then** an error message and a "Retry" button are shown; **and when** the backend works again and the user clicks "Retry", **then** the page displays the analysis. `[E2E]`

---

## 2. Non-Functional Requirements — V1

### Performance

**NFR-PERF-01 [V1]** Time from submission to a final status must be **at most 30 seconds for 90 % of analyses**, and analyses that exceed the time limit *(proposed: 60 seconds)* end `FAILED` (FR-UC05-16). *(Target to be confirmed after ADR-001.)*
* **AC:** **Given** the reference image set (about 20 garment images) and the real AI, **when** all are analyzed one after another, **then** at least 90 % reach a final status within 30 seconds. `[Benchmark]`

**NFR-PERF-02 [V1]** The submission endpoint must answer in under 2 seconds of server time for a 10 MB image (network transfer excluded).
* **AC:** **Given** an AI mock and a valid 10 MB image, **when** it is submitted, **then** the server answers 202 in under 2 seconds. `[API]`

**NFR-PERF-03 [V1]** The status endpoint must answer in under 500 ms for 95 % of requests.
* **AC:** **Given** 100 status requests on an existing analysis, **when** response times are measured, **then** at least 95 of them are under 500 ms. `[API]`

### File constraints

**NFR-FILE-01 [V1]** The maximum image size is **10 MB (10 485 760 bytes)**. The limit must be enforced **by the server**, even if the interface check is bypassed, and an oversize upload must be stopped without loading the whole file into memory.
* **AC1:** **Given** a valid image of exactly 10 485 760 bytes, **when** it is uploaded, **then** it is accepted. `[API]`
* **AC2:** **Given** a valid image of 10 485 761 bytes sent directly to the endpoint (bypassing the interface), **when** it is uploaded, **then** the answer is 413. `[API]`

**NFR-FILE-02 [V1]** Accepted types are JPEG, PNG and WEBP only.
* **AC:** covered by FR-UC05-05 and NFR-SEC-02. `[API]`

### Security

**NFR-SEC-01 [V1]** Passwords must be stored only as a salted hash from a modern adaptive algorithm (argon2id or bcrypt). A password must never be stored, logged or returned in clear.
* **AC1:** **Given** an account created with password `P`, **when** the stored value is read from the database, **then** it is different from `P` and is verified successfully by the hashing library. `[Unit]`
* **AC2:** **Given** two accounts created with the same password, **when** their stored hashes are compared, **then** they are different (salt). `[Unit]`

**NFR-SEC-02 [V1]** The file type must be determined from the **file content** (decoding the image), never from the extension or the declared content type alone.
* **AC:** **Given** a file with content that is not an image but a `.jpg` name and an `image/jpeg` declared type, **when** it is uploaded, **then** it is refused (FR-UC05-05). `[API]`

**NFR-SEC-03 [V1]** Stored images must use random file names; the name supplied by the user must never be used to build a path.
* **AC:** **Given** an upload named `../../evil.jpg`, **when** it is stored, **then** the file is inside the storage folder under a random name (for example a UUID). `[API]`

**NFR-SEC-04 [V1]** Embedded metadata (EXIF, including GPS location) must be removed from stored images.
* **AC:** **Given** a JPEG containing GPS EXIF data, **when** it is uploaded and stored, **then** the stored file contains no GPS tags. `[Unit]`

**NFR-SEC-05 [V1]** Secrets (JWT signing key, AI key, database password) must come from environment configuration and never from the repository. The application must refuse to start if the JWT key is missing.
* **AC1:** **Given** the environment without the JWT key, **when** the application starts, **then** it stops with an explicit configuration error. `[Unit]`
* **AC2:** **Given** the repository, **when** a secret scanner is run, **then** it reports no secret. `[Manual]` *(tool, for example gitleaks)*

**NFR-SEC-06 [V1]** Analyses and their image files must be reachable only by their owner (no public image address).
* **AC:** **Given** an image belonging to user A, **when** user B, or a request without token, asks for its file, **then** the answer is 404 (or 401 without token) and no image data is returned. `[API]`

**NFR-SEC-07 [V1]** CORS must allow only the configured frontend origin.
* **AC:** **Given** a request with the header `Origin: https://evil.example`, **when** it reaches the backend, **then** the response does not allow that origin. `[API]`

**NFR-SEC-08 [V1]** All user-provided text (names, descriptions) and all AI-generated text must be displayed as **plain text**, never interpreted as HTML.
* **AC:** **Given** a description `<script>alert(1)</script><b>x</b>`, **when** it is displayed on the analysis page, **then** it appears literally, no script runs and no bold element is created. `[E2E]`

**NFR-SEC-09 [V1]** Database access must use parameterized queries (through the ORM).
* **AC:** **Given** a sign-in with the email `' OR 1=1 --`, **when** it is submitted, **then** it is treated as an invalid email or unknown account, never as a successful sign-in. `[API]`

**NFR-SEC-10 [V1, at deployment]** In any deployed environment, the application must be served over HTTPS only.
* **AC:** **Given** the deployed address, **when** it is requested over plain HTTP, **then** it is redirected to HTTPS. `[Manual]`

### Compatibility

**NFR-COMP-01 [V1]** The interface must work on the **latest two stable versions** of Chrome, Firefox, Edge and Safari (desktop).
* **AC:** **Given** each supported browser, **when** the tester runs the checklist (sign up, sign in, upload, view result, failure and retry), **then** every step works. `[Manual]`

**NFR-COMP-02 [V1]** The layout should adapt to screen widths from 360 px to 1920 px without horizontal scrolling; Chrome on Android and Safari on iOS are best-effort.
* **AC:** **Given** browser developer tools set to widths 360, 768 and 1920 px, **when** the analysis page and the result page are displayed, **then** no horizontal scrollbar appears and all content is readable. `[Manual]`

### Reliability

**NFR-REL-01 [V1]** A failure in one analysis must not stop the server or affect other users' requests.
* **AC:** **Given** an AI mock that raises an exception for one analysis, **when** other requests are sent during and after it, **then** they are answered normally. `[API]`

### API consistency

**NFR-API-01 [V1]** Every error answer must use one uniform body: `{"error": "<code>", "message": "<human-readable text>"}`, including validation errors.
* **AC:** **Given** an error of each kind (validation, 401, 404, 409, 413, 415, 429), **when** the body is read, **then** it contains exactly the fields `error` and `message`. `[API]`

### Language

**NFR-LANG-01 [V1]** Interface messages and analysis content are in **English** in V1.
* **AC:** **Given** the reference images, **when** the results are checked, **then** all analysis text is in English. `[Benchmark]` `[Manual]`

### Testability and quality

**NFR-TEST-01 [V1]** The AI client must be hidden behind an interface so that the whole automated test suite runs **without network access and without an AI key**.
* **AC:** **Given** an environment without network and without AI key, **when** the automated test suite is run, **then** it passes. `[Unit]` `[API]`

**NFR-TEST-02 [V1]** A reference set of about 20 images must exist, each with its expected outcome (garment analyzed, or non-garment/unusable), to evaluate quality.
* **AC:** **Given** the repository documentation, **when** the reference set is opened, **then** each image has a recorded expected outcome. `[Manual]`

### Observability and cost

**NFR-LOG-01 [V1]** Each analysis status change must be logged with the analysis identifier and duration. Logs must never contain passwords, tokens or API keys.
* **AC:** **Given** an analysis that goes `PENDING` → `PROCESSING` → `COMPLETED`, **when** the logs are read, **then** each change appears with the analysis identifier and duration, and a search for the test password and token finds nothing. `[API]`

**NFR-LOG-02 [V1]** For each analysis, the system must record the AI model version and the processing duration (and usage or cost figures when the provider gives them), to control costs.
* **AC:** **Given** a completed analysis, **when** its record is read, **then** it contains the model version and the processing duration. `[API]`

---

## 3. Requirements for Later Versions

Acceptance criteria are written when each version starts. Each is subject to the "V1 Will NOT Do" list in `PROJECT.md`.

### UC-01 — Search Designs **[V1.1]**

* **FR-UC01-01:** The system must allow any user (authenticated or unauthenticated) to search for designs.
* **FR-UC01-02:** The system must allow the user to enter one or multiple keywords to execute a search.
* **FR-UC01-03:** The system must display designs matching the keywords entered by the user.
* **FR-UC01-04:** Each search result must be presented as a visual card containing at least one preview image.
* **FR-UC01-05:** Each search result card must be clickable.
* **FR-UC01-06:** Selecting a search result must open the corresponding article page (UC-11).
* **FR-UC01-07:** If no design matches the search keywords, the system must inform the user that no results were found.

### UC-04 — Publish a Design **[V1.1]**

* *To be written.* Prerequisite: decision on image copyright (see risks in `PROJECT.md`).

### UC-11 — View Public Article **[V1.1]** *(formerly FR-UC02-01 … 05)*

* **FR-UC11-01:** The system must allow any visitor or user to view a public article.
* **FR-UC11-02:** The system must display the main article image.
* **FR-UC11-03:** The system must display the article description.
* **FR-UC11-04:** The system must display the associated AI analysis details.
* **FR-UC11-05:** The system must display the author of the article.

### UC-08 — Add a Comment **[V1.2]** *(formerly FR-UC02-06)*

* **FR-UC08-01:** An authenticated user must be able to add a comment to an article.

### UC-10 — Report an Article **[V1.2]** *(formerly FR-UC02-07)*

* **FR-UC10-01:** An authenticated user must be able to report an article.

### UC-06, UC-07, UC-09 **[V1.2]**

* *To be written* (save an article, view saved articles, update profile).

---

## 4. Traceability

Every **[V1]** requirement above has at least one acceptance criterion. Automated test names include the requirement ID. The recommended order to build and test is:

```text
FR-UC03 (auth)  ──>  FR-UC05-01/05/06/07/08 (upload)  ──>  FR-UC05-02/03/04/16/17 (async analysis)
                ──>  FR-UC05-11…15 (result format)     ──>  FR-UC02 (display)  ──>  FR-UC05-19/20 (retry)
```

---

## 5. Values to Confirm

| Item | Proposed value | Where used |
|---|---|---|
| Token lifetime | 60 minutes | FR-UC03-04 |
| Failed sign-in limit | 5 per email per 15 minutes | FR-UC03-06 |
| Minimum password length | 8 characters | FR-UC03-02 |
| Status polling interval | 3 seconds | FR-UC05-02 |
| AI time limit | 60 seconds | FR-UC05-16 |
| Description maximum | 500 characters | FR-UC05-08 |
| Daily quota | 10 analyses per user, per UTC calendar day | FR-UC05-09 |
| Maximum retries per image | 3 | FR-UC05-19 |
| Target time | 30 s for 90 % of analyses | NFR-PERF-01 |
| Extension mismatch | Refuse (strict) | FR-UC05-07 |
| Duplicate submission of an image being analyzed | Undecided (see `use_case.md`, E15) | — |
