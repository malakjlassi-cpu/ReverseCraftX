# Use Cases — ReverseCraftX

**V1 in one sentence:** A signed-in user uploads an image of a garment
and receives a structured analysis that separates what is visible from
what is inferred.

Only V1 use cases are detailed below. Later use cases are listed at
the end and will be detailed when their version starts.

## 1. Actors

| Actor | Can run an analysis? |
|---|---|
| Visitor (not signed in) | No — can only reach sign-up / sign-in |
| Authenticated User | Yes, within the daily quota |
| AI Model / Provider (external) | — never interacts with users directly |

Visitors cannot run analyses: every analysis consumes limited free AI
capacity and involves an uploaded file, so requiring an account
enables quotas, rate limiting, and abuse control.

## 2. Summary

| ID | Use case | Version |
|---|---|---|
| UC-03 | Sign Up / Sign In | **V1** |
| UC-05 | Analyze an Image | **V1** |
| UC-02 | View My Analysis Result | **V1** |
| UC-01 | Search Designs | V1.1 |
| UC-04 | Publish a Design | V1.1 |
| UC-11 | View Public Article | V1.1 |
| UC-06 | Save an Article | V1.2 |
| UC-07 | View Saved Articles | V1.2 |
| UC-08 | Add a Comment | V1.2 |
| UC-09 | Update User Profile | V1.2 |
| UC-10 | Report an Article | V1.2 |

**V1 flow:** UC-03 → UC-05 → UC-02

## 3. Common Definitions

**Analysis status**

```
PENDING ──> PROCESSING ──> COMPLETED
   │              │
   └──────────────┴──────> FAILED
```

`PENDING → FAILED` covers the case where processing never starts (e.g.
the server restarts before the job begins).

**Retry rules (fixed, not proposed):**

| Retry type | Used when | Consumes quota? |
|---|---|---|
| Same image | Temporary failure: timeout, provider error/capacity, server restart | No |
| Another image | Image-caused failure: not a garment, unusable quality | Yes (an AI call was made) |

Maximum **3 retries** with the same image. A failed analysis is never
modified — retrying always creates a **new** `Analysis` row, for
traceability.

**Rejected at upload vs. FAILED**

| Kind | Detected | Result |
|---|---|---|
| Rejected at upload | During the upload request (type, size, corrupted file, quota, auth) | Request refused with an HTTP error. Nothing stored, no analysis created. |
| FAILED analysis | During background processing (non-garment, unusable image, timeout, provider/output error) | An `Analysis` row exists with status FAILED and an `error_message`. |

## 4. V1 Use Cases

### UC-03 — Sign Up / Sign In

**Actor:** Visitor (becomes Authenticated User).
**Goal:** Create an account and sign in so analyses can be run and linked to a user.

**Sign up:** first name, last name, email, password (min. 8 characters).
**Sign in:** email, password.

**Main scenario (sign up):** fill form → validate fields → check email
not already used → store account with hashed password → confirm and
redirect to sign in.

**Main scenario (sign in):** fill form → verify credentials → issue an
access token (60 min lifetime) → redirect to the analysis page (UC-05).

**Errors:**

| Case | Behavior |
|---|---|
| Email already registered | Refused with a clear message |
| Invalid email / empty field | Field-level validation message |
| Password too short | Refused, rule explained |
| Wrong email or password | Generic "Invalid email or password" (doesn't reveal which) |
| Too many failed sign-ins | Rate-limited: 5 failures / 15 min per email |
| Token expired/invalid | Redirected to sign in with an explanatory message |

---

### UC-05 — Analyze an Image

**Primary actor:** Authenticated User. **Secondary actor:** AI Provider.
**Goal:** Get a structured analysis (components, materials, structure,
hypotheses, manufacturing steps) with observed/inferred clearly separated.

**Preconditions:** authenticated, daily quota not exceeded, global
daily capacity not exhausted, privacy notice accepted.

**Input:** image (JPEG/PNG/WEBP, ≤10 MB), optional description (≤500 chars).

**Main scenario:**
1. User selects an image, optionally adds a description, accepts the privacy notice (first time only).
2. Server validates: real file type, size, decodable image, quota, global cap.
3. Image stored under a random filename, EXIF stripped.
4. `Analysis` created as `PENDING`, identifier returned immediately.
5. Interface shows "Analysis in progress..." and polls every ~3 s.
6. Background task sets status to `PROCESSING`, calls the AI provider.
7. Result validated against the schema (ADR-001, §6).
8. Valid → stored, status `COMPLETED`. Invalid/error → status `FAILED` with `error_message`.
9. Interface detects the final status and shows the result (UC-02).

**Errors — rejected at upload (nothing stored):**

| Case | Message |
|---|---|
| Unsupported format | "Unsupported format. Use JPEG, PNG or WEBP." |
| File > 10 MB | "Image too large (max 10 MB)." |
| Unreadable/corrupted file | "This file could not be read as an image." |
| Description too long | Validation message with the limit |
| Daily quota exceeded | Message with reset time |
| Privacy notice not accepted | Message asking to accept it |
| Global daily capacity reached | "The service reached its daily capacity. Please try again tomorrow." |
| Not authenticated | Redirect to sign in |

**Errors — FAILED analysis (detected during processing):**

| Case | Message | Retry |
|---|---|---|
| Non-garment image | "No garment was detected in this image." | Another image |
| Unusable image (dark/blurry/cropped) | "The garment could not be analyzed reliably. Try a clearer image." | Another image |
| AI timeout (>60 s) | "The analysis took too long. Please try again." | Same image |
| Provider error / free-tier limit reached | "The analysis service is busy or unavailable. Please try again later." | Same image |
| Invalid AI output | "The analysis could not be produced correctly. Please try again." | Same image |
| Server restarted mid-analysis | "The analysis was interrupted. Please try again." | Same image |

**Other:** the user closing the page doesn't stop the analysis (it
finishes in the background, visible later in "My analyses"). A 4th
retry on the same image is refused, asking for another image.

---

### UC-02 — View My Analysis Result

**Actor:** Authenticated User (owner).
**Goal:** Read one of my analyses, or browse my history.

**Main scenario:** open "My analyses" → list shown (thumbnail, date,
status, newest first) → select one (or arrive directly from UC-05) →
page shows image, description, status, and — if `COMPLETED` — all
result sections with observed/inferred visually distinct, each
inference showing its evidence and confidence.

**Errors:**

| Case | Behavior |
|---|---|
| PENDING / PROCESSING | "Analysis in progress...", keeps polling |
| FAILED | Error message + applicable retry option |
| No analyses yet | Empty-state message with a link to start one |
| Not owner / doesn't exist | "Analysis not found" (same message both cases) |
| Image fails to load | Fallback text; analysis text still readable |
| Loading error | Error message with a retry action |
| Token expired | Redirect to sign in |

Not part of V1: author info, comments, saving, reporting, profile.

## 5. Later Use Cases (not detailed yet)

- **UC-01 — Search Designs** (V1.1): keyword search over public articles; "no results" message when nothing matches. Depends on UC-04.
- **UC-04 — Publish a Design** (V1.1): make an analysis visible as a public article. Blocked on a copyright decision for internet-sourced images (see PROJECT.md risks).
- **UC-11 — View Public Article** (V1.1): view a published article's image, description, analysis, and author.
- **UC-06 — Save an Article** (V1.2): bookmark an article. Depends on UC-11.
- **UC-07 — View Saved Articles** (V1.2): list of bookmarked articles. Depends on UC-06.
- **UC-08 — Add a Comment** (V1.2): comment on an article, displayed safely (no raw HTML).
- **UC-09 — Update User Profile** (V1.2): edit name/email/password.
- **UC-10 — Report an Article** (V1.2): flag an article; requires a moderation role and a `Report` entity.
