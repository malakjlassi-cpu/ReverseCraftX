# Functional Requirements — ReverseCraftX

This document lists the functional requirements (FR) for ReverseCraftX V1, mapped to each Use Case (UC).

## UC-01 — Search Designs
* **FR-UC01-01:** The system must allow any user (authenticated or unauthenticated) to search for designs.
* **FR-UC01-02:** The system must allow the user to enter one or multiple keywords to execute a search.
* **FR-UC01-03:** The system must display designs matching the keywords entered by the user.
* **FR-UC01-04:** Each search result must be presented as a visual card containing at least one preview image.
* **FR-UC01-05:** Each search result card must be clickable.
* **FR-UC01-06:** Selecting a search result must open the corresponding article page.
* **FR-UC01-07:** If no design matches the search keywords, the system must inform the user that no results were found.

---

## UC-02 — View Article & Interactions
* **FR-UC02-01:** The system must allow any visitor or user to view a public article.
* **FR-UC02-02:** The system must display the main article image.
* **FR-UC02-03:** The system must display the article description.
* **FR-UC02-04:** The system must display the associated AI analysis details.
* **FR-UC02-05:** The system must display the author of the article.
* **FR-UC02-06:** An authenticated user must be able to add a comment to the article.
* **FR-UC02-07:** An authenticated user must be able to report an article.

---

## UC-05 — Image Analysis
* **FR-UC05-01:** The system must allow the user to upload an image (accepted formats: JPEG, PNG, WEBP; maximum size: 10 MB).
* **FR-UC05-02:** The system must trigger the analysis asynchronously and display a loading indicator ("Analysis in progress...") to the user.
* **FR-UC05-03:** Upon successful analysis, the system must store and display the results while clearly separating factual observations from AI hypotheses.
* **FR-UC05-04:** If the analysis fails (API timeout, unreadable image, or non-garment image), the system must catch the error, update the analysis status to `FAILED`, and display a clear error message allowing the user to retry with another image.
