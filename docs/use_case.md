# Use Cases — ReverseCraftX (V1)

This document defines the core Use Cases (UC) for ReverseCraftX, detailing the actors, preconditions, inputs, main flows, and edge cases.

## Summary of Use Cases
* **UC-01:** Search Designs
* **UC-02:** View Article
* **UC-03:** Sign Up / Authenticate
* **UC-04:** Publish a Design
* **UC-05:** Analyze an Image
* **UC-06:** Save an Article
* **UC-07:** View Saved Articles
* **UC-08:** Add a Comment
* **UC-09:** Update User Profile


## Detailed Use Cases

### UC-01 — Search Designs
* **Actor:** Visitor / Authenticated User
* **Goal:** Quickly find published designs within ReverseCraftX.
* **Precondition:** None (No authentication required).
* **Input:** Search keywords (e.g., *"long dress"*, *"satin dress"*, *"wedding dress"*).
* **Main Scenario:**
  1. The user opens the search interface.
  2. The user enters keywords.
  3. The system queries public articles matching the input.
  4. The system displays the matching results.
  5. The user selects an article.
* **Output:** A list of matching article previews.
* **Alternative / Edge Case:** 
  * *No results:* The system informs the user that no matching designs were found.



### UC-02 — View Article
* **Actor:** Visitor / Authenticated User
* **Goal:** View detailed information about a selected article.
* **Preconditions:** 
  * The article exists and is public.
  * The user has accessed it via search or direct navigation.
* **Input:** Selected article reference.
* **Main Scenario:**
  1. The user selects an article from the search results.
  2. The system opens the article detail page.
  3. The system renders available details: main image, description, ReverseCraft AI analysis, author info, and comments.
  4. The user reviews the content.
* **Post-Actions by Role:**
  * *Visitor:* Read-only access.
  * *Authenticated User:* Can add a comment, save the article to favorites, or report the article.
* **Edge Cases:**
  * *Article not found / deleted:* The system displays a message indicating the article is no longer available.
  * *Image load error:* The system displays fallback text while keeping other textual information accessible.
  * *Loading error:* The system displays an error message with a retry option.


