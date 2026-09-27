# Project Definition — ReverseCraftX

**V1 in one sentence:** A signed-in user uploads an image of a garment
and receives a structured analysis that separates what is visible from
what is inferred.

## 1. Problem

Someone finds a clothing design they love online (Pinterest, etc.) but
can't find it in stores, and struggles to describe it precisely enough
for a tailor to reproduce it — proportions, construction, and fabric
are hard to communicate from a photo alone.

**Core challenge:** help someone go from a visual inspiration to a
concrete understanding of how that design is built.

## 2. Goal & Vision

ReverseCraftX is a visual reverse-engineering assistant for physical
designs: it turns an image into structured manufacturing insight.

- **V1 domain:** clothing (dresses, tailoring, etc.).
- **Long-term:** accessories, furniture, other physical products.

## 3. Target User

V1 is designed for the **Beginner**: someone with an inspiration image
who wants to understand a garment well enough to brief a tailor.
(Artisans, designers, and curious users can use the same output later,
but V1 makes no dedicated effort for them — no advanced vocabulary
modes, no professional workflows.)

## 4. V1 Scope

**Actors**
- Signed-in user: the only one who can run an analysis.
- Visitor: can only reach sign-up / sign-in.

**Input:** one image (JPEG/PNG/WEBP, max 10 MB) of one garment, plus an
optional text description. Single view only — hidden parts (back,
lining, seams) are never treated as observed.

**Output:** a structured analysis with:
- **Components** — visible parts (bodice, sleeves, collar, skirt...)
- **Probable materials** — fabric hypotheses
- **Structure** — how the parts relate
- **Hypotheses & uncertainty** — every inference has evidence + a confidence level
- **Manufacturing steps** — high-level production guide

**Language:** English only in V1.

**Budget: 0 €.** Only free/open-source tools (Python, FastAPI, MySQL
Community, Git/GitHub, pytest). The AI is reached through a free tier
or a local model; billing is never activated. V1 runs locally — no
paid hosting, no domain, no public demo. Because free tiers have
limits and may reuse submitted content, the system enforces a
per-user quota and a global daily cap, and shows a privacy notice
before the first analysis (see ADR-001).

## 5. Core Principle: Managing Uncertainty

Every item in an analysis separates:
- what is **observed** in the image,
- what is **inferred** (the hypothesis),
- the **evidence** supporting the inference,
- a **confidence level** (low / medium / high).

The system never presents a guess as a fact.

## 6. What V1 Will NOT Do

- Find the exact original product, or handle purchases.
- Guarantee exact measurements from a single image.
- Generate ready-to-use sewing patterns.
- Publishing, search, comments, saving, reporting, or a public profile
  (all deferred — see Roadmap).
- Any domain other than clothing.
- Any paid service or public deployment.

## 7. Success Criteria for V1

Evaluated on a reference set of ~20 garment images (varied garments +
a few non-garment images):

| # | Criterion | Target |
|---|---|---|
| 1 | Valid, schema-checked result with observed/inferred separation | 100 % of successful analyses |
| 2 | Time from upload to result | ≤ 30 s for 90 % (experimental target, see NFR-PERF-01) |
| 3 | Analyses judged useful enough to brief a tailor | ≥ 80 % |
| 4 | Non-garment / unreadable images end in a clear FAILED state | 100 % |
| 5 | Inferences shown without confidence or evidence | 0 |
| 6 | Total cost to build, test, and run V1 | 0 € |
| 7 | AI usage stays within the provider's free limits | global daily cap below the documented free limit |

## 8. Main Risks

| Risk | Mitigation |
|---|---|
| Free-tier AI limits reached or withdrawn | Global daily cap, per-user quota, no auto-retry, AI behind an interface, Plan B in ADR-001 |
| Free tier may reuse submitted images | Privacy notice, no identifiable people, EXIF stripped |
| Hallucinated / wrong analyses | Observed/inferred separation, confidence levels, reference set |
| Non-deterministic AI output | AI mocked in tests, output validated against a schema |
| Scope creep toward a social platform | Section 6 above is binding; new ideas go to the V1.1/V1.2 backlog |

## 9. Roadmap

- **V1:** authentication + image analysis + "my analyses" (100 % free, local).
- **V1.1:** publish a design as an article, search, view article.
- **V1.2:** comments, saved articles, profile, reporting.
- **V2+:** other domains, additional languages.

## 10. Engineering Objectives

V1 also serves as a sandbox to practice: requirements definition, Git
workflow, layered/clean architecture, API + AI integration,
asynchronous processing, testing, and documentation.
