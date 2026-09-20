# Project Definition — ReverseCraftX

> **Status:** Draft v3 — V1 scope clarified, **100 % free constraint** added.
> **V1 in one sentence:** *A signed-in user uploads an image of a garment and receives a structured analysis that separates what is visible from what is inferred.*

## 1. Problem Statement
The genesis of ReverseCraftX stems from a common frustration: falling in love with a clothing design online (e.g., on Pinterest), only to face failure when trying to find it in physical stores or explaining it to local artisans.

Even when working with a tailor, miscommunications regarding proportions, construction, or fabric quality often lead to disappointing results. Furthermore, the desired design might not even exist as a ready-made product.

**The core challenge:** How can we help someone bridge the gap between a visual inspiration and a concrete understanding of how that design is constructed and can be reproduced?

## 2. Project Goal & Vision
ReverseCraftX is a **visual reverse-engineering assistant for physical designs**.

The system transforms visual references into structured manufacturing insights:

* **Initial Domain:** Clothing designs (dresses, tailoring, etc.).
* **Long-Term Vision:** Expand progressively to accessories, furniture, decorative items, and other physical products.

## 3. Target Audience
ReverseCraftX is built for a diverse range of users, adapting its complexity to their needs:

* **Beginners:** Individuals with no manufacturing background looking to understand a design to communicate it clearly to an artisan.
* **Artisans & Tailors:** Professionals using the analysis as a baseline for bespoke creations.
* **Designers & Creators:** Creators studying existing designs to decompose them into components and materials.
* **Curious Users:** Anyone interested in how everyday objects are constructed.

### Primary persona for V1
V1 is designed and validated for the **Beginner**: someone with an inspiration image who wants to understand the garment well enough to explain it to a tailor. The other audiences benefit from the same analysis, but V1 makes no dedicated effort to serve them (no advanced technical vocabulary modes, no professional workflows).

## 4. First Version (V1) Scope
V1 focuses exclusively on establishing a **complete analysis pipeline for clothing designs**: upload → analysis → consultable result.

### Actors
* **Signed-in user:** the only actor allowed to run an analysis (limits abuse and protects the limited free AI capacity).
* **Visitor (not signed in):** can only reach the sign-up / sign-in pages in V1.

### Inputs
* An image of a clothing design (JPEG, PNG, WEBP; maximum 10 MB).
* An optional text description providing context or user preferences.

### Assumptions on the input
* One image contains one main garment, reasonably visible and in focus.
* The system works from a single view; hidden parts (back, lining, seams) are never treated as observed.

### Outputs
The system returns a structured analysis containing:

1. **Components:** Visible parts (bodice, sleeves, collar, skirt, etc.).
2. **Probable Materials:** Hypotheses on fabric types (cotton, satin, linen, velvet, etc.).
3. **Structure:** Hierarchical organization of the design.
4. **Hypotheses & Uncertainty:** Clear separation between strict visual observations and AI-driven assumptions, each with a confidence level.
5. **Manufacturing Steps:** High-level guide on how a similar garment could be produced.

### Language
* **V1 interface and analysis language: English.**
* French and Arabic are candidates for a later version (Arabic requires right-to-left layout support).

### Budget constraint
**V1 must be 100 % free (budget: 0 €).** It must be built, tested and run without any paid service and without a payment card:

* Only free and open-source tools are used (Python, FastAPI, MySQL Community, Git, GitHub, pytest, ruff).
* The AI is reached through a **free tier** of a provider, or through a **local model**. Billing is never activated.
* V1 runs **locally**: no paid hosting, no domain name, no public demo.
* Free tiers have limits, can change without notice, and may allow the provider to use submitted images. The system therefore limits its own usage (per-user quota and global daily cap), shows a privacy notice before the first analysis, and keeps the AI behind an interface so the provider can be replaced (see [ADR-001](adr/001-ia.md)).

## 5. Core Principle: Managing Uncertainty
ReverseCraftX follows a strict philosophical rule: **Distinguish between what is visible and what is inferred.**
The system must never present uncertain visual interpretations as absolute facts. Every analysis communicates probabilities rather than absolute truths.

In practice, every item in an analysis carries:
* what is **observed** in the image;
* what is **inferred** (the hypothesis);
* the **evidence** supporting the inference;
* a **confidence** level (`low`, `medium`, `high`).

## 6. What V1 Will NOT Do
To maintain a realistic execution scope, V1 deliberately excludes:

### Product limits
* Finding exact original products or managing purchases.
* Guaranteeing exact physical measurements from a single image.
* Generating ready-to-use sewing patterns.
* Comparing suppliers, stores, or fabric prices.

### Platform features (deferred to V1.1 / V2)
* Publishing designs as public articles.
* Searching published designs.
* Comments on articles.
* Saving / bookmarking articles.
* Reporting content and moderation tools.
* Editable user profile.
* Search history.

### Domains
* Anything other than clothing (accessories, furniture, decorative items).

### Paid services and public hosting
* Any paid service (paid AI plan, paid hosting, domain name) and any public deployment or demo.

## 7. Success Criteria for V1
V1 is considered successful when, on a **reference set of about 20 garment images** built by the author (varied garments, plus a few non-garment images):

| # | Criterion | Target |
|---|-----------|--------|
| 1 | Analysis returns a valid structured result (schema-validated) with observed / inferred separation on every item | 100 % of successful analyses |
| 2 | Time from upload to result | ≤ 30 s for 90 % of analyses *(to be confirmed after ADR-001)* |
| 3 | Analyses judged **useful** by the author (a beginner could use them to brief a tailor) | ≥ 80 % |
| 4 | Non-garment or unreadable images end in `FAILED` with a clear message and a retry option | 100 % |
| 5 | Inferences presented without a confidence level or evidence | 0 |
| 6 | Total cost of building, testing and running V1 | **0 €** (no paid service, no payment card required) |
| 7 | AI usage stays within the provider's free limits | Global daily cap configured below the documented free daily limit |

## 8. Risks

| Risk | Impact | Mitigation |
|------|--------|------------|
| **Free-tier limits** of the AI provider (quota reached, reduced or withdrawn) | Analyses fail, or V1 cannot run at 0 € | Global daily cap below the free limit, per-user quota, no automatic retries, AI behind an interface, Plan B in ADR-001 (other free provider, small local model, replay of recorded results); billing is never activated |
| **Free-tier data terms** (submitted images may be used by the provider to improve its products) | Privacy concern | Privacy notice accepted before the first analysis, test only with non-sensitive garment photos, no identifiable people, EXIF removed |
| **Poor or hallucinated analyses** (materials and construction are hard to judge from a photo) | Loss of user trust | Strict observed / inferred separation, confidence levels, reference image set, output validation |
| **Non-deterministic AI output** | Hard to test | Mock the AI client in unit tests, validate against a schema |
| **Dependency on an external AI provider** (latency, outages, price changes) | Failures, delays | Timeout handling, `FAILED` state with retry, AI client behind an interface |
| **Copyright** of images uploaded from the internet | Legal exposure | Images kept private to their owner in V1; revisit before any public publishing feature |
| **Privacy and upload security** (EXIF/GPS data, malicious files) | Data leak, attacks | Strip metadata, validate real file type, enforce size limits, random file names |
| **Scope creep** toward a social platform | V1 never ships | "What V1 Will NOT Do" list is binding; new ideas go to the V1.1 backlog |

## 9. Roadmap (indicative)
* **V1:** Authentication + image analysis + consult my analyses (100 % free, runs locally).
* **V1.1:** Publish a design as an article, search, view article; public demo only if a free hosting and AI option exists.
* **V1.2:** Comments, saved articles, profile, reporting.
* **V2+:** Other domains (accessories, furniture), additional languages.

## 10. Engineering Objectives (Learning Goals)
Beyond the product itself, V1 serves as a practical sandbox to master software engineering fundamentals:

* Requirements definition and project structuring.
* Git workflow and version control.
* Clean architecture design and modular development.
* API design and AI model integration.
* Asynchronous task handling, testing, and documentation.
