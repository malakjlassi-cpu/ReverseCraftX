# Project Definition — ReverseCraftX

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


## 4. First Version (V1) Scope
V1 focuses exclusively on establishing a complete analysis pipeline for clothing designs.

### Inputs
* An image of a clothing design (JPEG, PNG, WEBP).
* An optional text description providing context or user preferences.

### Outputs
The system returns a structured analysis containing:
1. **Components:** Visible parts (bodice, sleeves, collar, skirt, etc.).
2. **Probable Materials:** Hypotheses on fabric types (cotton, satin, linen, velvet, etc.).
3. **Structure:** Hierarchical organization of the design.
4. **Hypotheses & Uncertainty:** Clear separation between strict visual observations and AI-driven assumptions.
5. **Manufacturing Steps:** High-level guide on how a similar garment could be produced.



## 5. Core Principle: Managing Uncertainty
ReverseCraftX follows a strict philosophical rule: **Distinguish between what is visible and what is inferred.** 
The system must never present uncertain visual interpretations as absolute facts. Every analysis communicates probabilities rather than absolute truths.



## 6. What V1 Will NOT Do
To maintain a realistic execution scope, V1 deliberately excludes:
* Finding exact original products or managing purchases.
* Guaranteeing exact physical measurements from a single image.
* Generating ready-to-use sewing patterns.
* Comparing suppliers, stores, or fabric prices.



## 7. Engineering Objectives (Learning Goals)
Beyond the product itself, V1 serves as a practical sandbox to master software engineering fundamentals:
* Requirements definition and project structuring.
* Git workflow and version control.
* Clean architecture design and modular development.
* API design and AI model integration.
* Asynchronous task handling, testing, and documentation.
