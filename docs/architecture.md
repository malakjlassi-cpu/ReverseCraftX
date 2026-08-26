# Architecture — ReverseCraftX

## 1. Architecture Overview
ReverseCraftX is organized around several components, each with distinct responsibilities.
The user interacts with the application through the user interface (Frontend).

The Frontend communicates with the Backend via an API using the HTTP protocol.
The Backend contains the core application logic and communicates with:
* The database for structured data;
* Image storage for image files;
* The analysis server for processing images;
* The AI model for visual analysis.

### General Architecture
                         USER
                           │
                           │ interaction
                           ▼
                    ┌──────────────┐
                    │   FRONTEND   │
                    │  Interface   │
                    └──────┬───────┘
                           │
                     HTTP / API
                           │
                           ▼
                    ┌──────────────┐
                    │    BACKEND   │
                    │  Application │
                    │    Logic     │
                    └──────┬───────┘
                           │
          ┌────────────────┼─────────────────┐
          │                │                 │
          ▼                ▼                 ▼
    ┌───────────┐   ┌──────────────┐  ┌──────────────┐
    │ DATABASE  │   │IMAGE STORAGE │  │   ANALYSIS   │
    │           │   │              │  │   SERVER     │
    └───────────┘   └──────────────┘  └───────┬──────┘
                                              │
                                              ▼
                                       ┌─────────────┐
                                       │  AI MODEL   │
                                       └─────────────┘



## 2. System Components

### 2.1 User
The user is any person utilizing ReverseCraftX. They interact with the application through interface elements such as:
* Buttons;
* Search bars;
* Forms;
* Clickable images;
* Detail view pages.

Users do not communicate directly with the database or the AI model.

### 2.2 Frontend
The Frontend is the visible interface of ReverseCraftX. It allows the user to:
* Search for designs;
* View articles;
* Publish designs;
* Upload images;
* Consult analysis results;
* Add comments;
* Save articles;
* Update their profile.

The Frontend sends requests to the Backend through the API.
