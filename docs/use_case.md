# Use Cases — ReverseCraftX

> **Status:** Draft v2 — aligned with the V1 scope in [`PROJECT.md`](PROJECT.md).
> **V1 in one sentence:** *A signed-in user uploads an image of a garment and receives a structured analysis that separates what is visible from what is inferred.*

This document defines the Use Cases (UC) of ReverseCraftX: actors, preconditions, inputs, main scenarios, error cases and postconditions. Only the **V1** use cases are fully detailed. Later use cases are listed with their target version and will be detailed before they are built.

Values marked *(proposed)* are starting points to be confirmed in `REQUIREMENTS.md` and the ADR on the AI approach.

---

## 1. Actors

| Actor | Type | Description | Can run an analysis? |
|---|---|---|---|
| **Visitor** | Primary | A person who is not signed in. | **No.** In V1 a visitor can only reach the sign-up and sign-in pages. |
| **Authenticated User** | Primary | A person signed in with an account. | **Yes**, within the daily quota. |
| **AI Model / Provider** | Secondary (external system) | The service that performs the visual analysis. It never interacts with users directly. | — |

**Decision — visitors cannot run analyses.** Every analysis has a real cost (AI call) and involves an uploaded file. Requiring an account allows quotas, rate limiting, traceability and abuse control.

---

## 2. Summary of Use Cases

| ID | Use case | Actor | Version |
|---|---|---|---|
| **UC-03** | Sign Up / Sign In | Visitor, Authenticated User | **V1** |
| **UC-05** | Analyze an Image | Authenticated User | **V1** |
| **UC-02** | View My Analysis Result | Authenticated User | **V1** |
| UC-01 | Search Designs | Visitor, Authenticated User | V1.1 |
| UC-04 | Publish a Design | Authenticated User | V1.1 |
| UC-11 | View Public Article | Visitor, Authenticated User | V1.1 |
| UC-06 | Save an Article | Authenticated User | V1.2 |
| UC-07 | View Saved Articles | Authenticated User | V1.2 |
| UC-08 | Add a Comment | Authenticated User | V1.2 |
| UC-09 | Update User Profile | Authenticated User | V1.2 |
| UC-10 | Report an Article | Authenticated User | V1.2 |

> **Changes from draft v1:**
> * UC-02 "View Article" is reduced to **View My Analysis Result** for V1. The public article page becomes **UC-11** (V1.1).
> * "Report an Article" was hidden inside UC-02; it is now its own use case, **UC-10** (V1.2).
> * Version targets follow the roadmap in `PROJECT.md`, section 9.

**V1 flow:**

```text
UC-03 Sign Up / Sign In ──> UC-05 Analyze an Image ──> UC-02 View My Analysis Result
```

---

## 3. Common Definitions

### Analysis status

| Status | Meaning | Final? |
|---|---|---|
| `PENDING` | The analysis is created and waiting to be processed. | No |
| `PROCESSING` | The AI analysis is running. | No |
| `COMPLETED` | A valid structured result is stored. | Yes |
| `FAILED` | The analysis could not be completed. An `error_message` explains why. | Yes |

Allowed transitions:

```text
PENDING ──> PROCESSING ──> COMPLETED
   │              │
   └──────────────┴──────> FAILED
```

`PENDING → FAILED` covers cases where processing never starts (for example, the server restarts before the job begins).

### What "retry" means

A final `FAILED` analysis is **never modified or overwritten**. It stays in the user's history for traceability. Retrying always creates a **new analysis**, in one of two ways:

| Retry type | Used when | What happens |
|---|---|---|
| **Retry with the same image** | The failure is temporary: timeout, provider error, server restart. | A new analysis is created for the **already stored image** and description. No new upload. |
| **Try another image** | The failure is caused by the image: not a garment, unusable quality. | The user returns to the upload form and submits a new image. |

Rules *(proposed)*:
* A failed analysis caused by a **system error** (timeout, provider error, restart) does **not** consume the user's daily quota.
* A failed analysis caused by the **image itself** does consume quota, because the AI call was made.
* Maximum **3 retries with the same image**, to avoid endless loops and cost.

### Synchronous rejection vs. `FAILED`

Two kinds of errors exist and are handled differently:

| Kind | When detected | Result |
|---|---|---|
| **Rejected at upload** | During the upload request (type, size, corrupted file, quota, authentication). | The request is refused with an error. **No image is stored and no analysis is created.** |
| **`FAILED` analysis** | During background processing (non-garment, unusable image, timeout, provider or output error). | An analysis exists with status `FAILED` and an `error_message`. |

---

## 4. V1 Use Cases

### UC-03 — Sign Up / Sign In

* **Actor:** Visitor (becomes an Authenticated User).
* **Goal:** Create an account and sign in, so that analyses can be run and are linked to a user.
* **Preconditions:**
  * *Sign up:* the email address is not already registered.
  * *Sign in:* an account exists for the email address.
* **Inputs:**
  * *Sign up:* first name, last name, email, password (minimum 8 characters *(proposed)*).
  * *Sign in:* email, password.

**Main Scenario A — Sign Up**
1. The visitor opens the sign-up page.
2. The visitor enters first name, last name, email and password.
3. The system validates the format of each field.
4. The system checks that the email is not already used.
5. The system stores the account with the password **hashed** (never stored in clear).
6. The system confirms the account creation and directs the user to sign in (or signs them in directly).

**Main Scenario B — Sign In**
1. The visitor opens the sign-in page.
2. The visitor enters email and password.
3. The system verifies the credentials.
4. The system issues an access token with a limited lifetime.
5. The user is redirected to the analysis page (UC-05).

* **Postconditions:** The user holds a valid token and can access UC-05 and UC-02. An account exists in the database.

**Error and Edge Cases**

| # | Case | System behavior |
|---|---|---|
| E1 | Email already registered (sign up) | Sign-up refused with a clear message; no account created. |
| E2 | Invalid email format or empty field | Field-level validation message; nothing submitted. |
| E3 | Password too short or too weak | Refused with the password rule explained. |
| E4 | Wrong email or password (sign in) | **Generic** message ("Invalid email or password"), without revealing which one is wrong. |
| E5 | Too many failed sign-in attempts | Temporary block or slow-down (rate limiting) with a message to try later. |
| E6 | Token expired or invalid while using the app | The user is redirected to sign in; a message explains the session ended. |
| E7 | Server or network error | Error message with a retry option; no partial account is left. |

---

### UC-05 — Analyze an Image

* **Primary actor:** Authenticated User.
* **Secondary actor:** AI Model / Provider.
* **Goal:** Obtain a structured analysis of a garment shown in an image: components, probable materials, structure, hypotheses with uncertainty, and high-level manufacturing steps.
* **Preconditions:**
  * The user is authenticated (valid token).
  * The user has not exceeded the daily analysis quota *(proposed: 10 per day)*.
  * The user has an image file on their device.
* **Inputs:**
  * **Image:** JPEG, PNG or WEBP, maximum 10 MB.
  * **Description** (optional): free text giving context or preferences, maximum 500 characters *(proposed)*.

**Main Scenario**
1. The user opens the analysis page.
2. The user selects an image and optionally types a description.
3. The interface performs a quick check of format and size and gives immediate feedback.
4. The user submits the form.
5. The system authenticates the user and checks the quota.
6. The system validates the file on the server: real file type (not just the extension), size, and that the image can be decoded.
7. The system stores the image under a random file name and removes embedded metadata (such as GPS data).
8. The system creates an analysis with status `PENDING` and immediately returns its identifier.
9. The interface displays "Analysis in progress..." and regularly asks the system for the status *(proposed: every 2–3 seconds)*.
10. The system starts the analysis in the background and sets the status to `PROCESSING`.
11. The system sends the image (and description) to the AI model.
12. The AI model returns a structured result.
13. The system validates the result against the expected structure: every element separates what is **observed** from what is **inferred**, with evidence and a confidence level.
14. The system stores the result and sets the status to `COMPLETED`.
15. The interface detects `COMPLETED` and displays the result (see UC-02).

* **Output:** An analysis containing components, probable materials, structure, hypotheses and uncertainty, and manufacturing steps, with observed and inferred content clearly separated.
* **Postconditions:**
  * *Success:* the image and a `COMPLETED` analysis are stored and linked to the user. The result is visible only to its owner.
  * *Failure:* a `FAILED` analysis with an `error_message` is stored (for errors detected during processing), and the user is offered a retry (see section 3).

**Error and Edge Cases**

*A. Rejected at upload (no image stored, no analysis created)*

| # | Case | System behavior | User next step |
|---|---|---|---|
| E1 | Unsupported format (not JPEG, PNG, WEBP) | Upload refused: "Unsupported format. Use JPEG, PNG or WEBP." | Choose another file. |
| E2 | File larger than 10 MB | Upload refused: "Image too large (max 10 MB)." | Choose a smaller file. |
| E3 | **Unreadable / corrupted file** (cannot be decoded, or extension does not match content) | Upload refused: "This file could not be read as an image." | Choose another file. |
| E4 | Description too long | Validation message showing the limit. | Shorten the text. |
| E5 | Daily quota exceeded | Upload refused with the quota message and when it resets. | Wait for reset. |
| E6 | Not authenticated or token expired | Redirect to sign in (UC-03). | Sign in and resubmit. |

*B. `FAILED` analysis (detected during processing)*

| # | Case | Status | Message to the user | Retry type |
|---|---|---|---|---|
| E7 | **Non-garment image** (no clothing detected) | `FAILED` | "No garment was detected in this image." | **Try another image** |
| E8 | **Unusable image** (too dark, blurry, garment mostly hidden or cropped) | `FAILED` | "The garment could not be analyzed reliably. Try a clearer image." | **Try another image** |
| E9 | **AI timeout** (no answer within the time limit *(proposed: 60 s)*) | `FAILED` | "The analysis took too long. Please try again." | **Same image** |
| E10 | AI provider error or unavailable | `FAILED` | "The analysis service is temporarily unavailable." | **Same image** |
| E11 | AI result does not match the expected structure (invalid output) | `FAILED` | "The analysis could not be produced correctly. Please try again." | **Same image** |
| E12 | Server restarted while the analysis was `PENDING` or `PROCESSING` | `FAILED` (set at startup) | "The analysis was interrupted. Please try again." | **Same image** |

*C. Other situations*

| # | Case | System behavior |
|---|---|---|
| E13 | The user closes the page during the analysis | The analysis continues in the background. The result is available later in "My analyses" (UC-02). |
| E14 | The user reaches the retry limit for one image (3) | Retry with the same image is disabled; the user is asked to try another image. |
| E15 | The user submits again while an analysis of the same image is running | The system returns the existing analysis instead of creating a duplicate *(to confirm)*. |

**Business Rules**
* Only signed-in users can run analyses (see section 1).
* An analysis and its image are **private to their owner** in V1.
* The system must never present an uncertain interpretation as a fact: every inferred item carries its evidence and a confidence level.
* Information that cannot be seen (back of the garment, lining, seams) is never reported as observed.

---

### UC-02 — View My Analysis Result

> Reduced V1 version of the former "View Article". The public article page is now UC-11 (V1.1).

* **Actor:** Authenticated User (owner of the analysis).
* **Goal:** Read the result of one of my analyses, and browse my past analyses.
* **Preconditions:**
  * The user is authenticated.
  * The analysis exists and belongs to the user.
* **Input:** An analysis reference, coming either from the end of UC-05 or from the "My analyses" list.

**Main Scenario**
1. The user opens "My analyses".
2. The system lists the user's analyses, newest first: thumbnail, date and status.
3. The user selects an analysis (or arrives directly from UC-05 when it completes).
4. The system displays the analysis page: the image, the optional description and the status.
5. For a `COMPLETED` analysis, the system displays: **components**, **probable materials**, **structure**, **hypotheses and uncertainty**, and **manufacturing steps**. Observed content and inferred content are visually distinct, and each inference shows its evidence and confidence.
6. The user reviews the content.

* **Output:** The detailed analysis, or its current status.
* **Postconditions:** None (read-only).

**Error and Edge Cases**

| # | Case | System behavior |
|---|---|---|
| E1 | The analysis is `PENDING` or `PROCESSING` | Shows "Analysis in progress..." and keeps checking the status until it is final. |
| E2 | The analysis is `FAILED` | Shows the error message and the retry option that applies (same image or another image, see UC-05). |
| E3 | The user has no analyses yet | Shows an empty-state message with a link to start an analysis (UC-05). |
| E4 | The analysis does not exist, or belongs to another user | Shows "Analysis not found". The same message is used in both cases so the existence of other users' analyses is not revealed. |
| E5 | The image fails to load | Shows fallback text; the analysis text remains fully readable. |
| E6 | Loading error (network or server) | Shows an error message with a retry option. |
| E7 | Token expired or invalid | Redirect to sign in (UC-03). |

**Not part of V1** (moved to later use cases): author information, comments (UC-08), saving to favorites (UC-06), reporting (UC-10), profile (UC-09).

---

## 5. Later Use Cases

These use cases are **not part of V1**. They keep their intent but are detailed only when their version starts. Each one is subject to the "V1 Will NOT Do" list in `PROJECT.md`.

### UC-01 — Search Designs *(V1.1)*

* **Actor:** Visitor / Authenticated User
* **Goal:** Quickly find published designs within ReverseCraftX.
* **Precondition:** None (no authentication required).
* **Input:** Search keywords (e.g., *"long dress"*, *"satin dress"*, *"wedding dress"*).
* **Main Scenario:**
  1. The user opens the search interface.
  2. The user enters keywords.
  3. The system queries public articles matching the input.
  4. The system displays the matching results.
  5. The user selects an article (UC-11).
* **Output:** A list of matching article previews.
* **Alternative / Edge Case:**
  * *No results:* The system informs the user that no matching designs were found.
* **Note:** Depends on UC-04 (there must be published designs). Search should cover analysis content such as materials, not only titles.

### UC-04 — Publish a Design *(V1.1)*

* **Actor:** Authenticated User
* **Goal:** Make one of my analyses visible as a public article.
* **Note:** To detail before V1.1. **Open question:** copyright of images taken from the internet must be decided before any public publishing feature (see risks in `PROJECT.md`). In V1, analyses stay private.

### UC-11 — View Public Article *(V1.1)*

* **Actor:** Visitor / Authenticated User
* **Goal:** View detailed information about a published article.
* **Preconditions:** The article exists and is public.
* **Main Scenario:**
  1. The user selects an article from the search results (UC-01).
  2. The system opens the article page.
  3. The system displays the main image, description, analysis, and author information.
  4. The user reviews the content. A visitor has read-only access.
* **Edge Cases:**
  * *Article not found / deleted:* The system displays a message indicating the article is no longer available.
  * *Image load error:* The system displays fallback text while keeping other textual information accessible.
  * *Loading error:* The system displays an error message with a retry option.
* **Note:** Comments, saving and reporting are separate use cases (UC-08, UC-06, UC-10).

### UC-06 — Save an Article *(V1.2)*

* **Actor:** Authenticated User
* **Goal:** Bookmark a public article to find it again later.
* **Note:** To detail before V1.2. Depends on UC-11.

### UC-07 — View Saved Articles *(V1.2)*

* **Actor:** Authenticated User
* **Goal:** See the list of articles I saved.
* **Note:** To detail before V1.2. Depends on UC-06.

### UC-08 — Add a Comment *(V1.2)*

* **Actor:** Authenticated User
* **Goal:** Comment on a public article.
* **Note:** To detail before V1.2. Comment content must be displayed safely (no raw HTML injection).

### UC-09 — Update User Profile *(V1.2)*

* **Actor:** Authenticated User
* **Goal:** Edit my personal information (name, email, password).
* **Note:** To detail before V1.2.

### UC-10 — Report an Article *(V1.2)*

* **Actor:** Authenticated User
* **Goal:** Report a public article that is inappropriate or infringes rights.
* **Note:** To detail before V1.2. Requires a moderation role and a report entity in the data model.
