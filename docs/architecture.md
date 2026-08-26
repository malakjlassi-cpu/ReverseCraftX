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
User
↓
Frontend
↓
API / HTTP
↓
Backend


### 2.3 Backend
The Backend is the core logic engine of the application. It is responsible for:
* Receiving requests from the Frontend;
* Validating incoming data;
* Managing authentication;
* Applying business rules;
* Interacting with the database;
* Managing image storage;
* Triggering and tracking analyses;
* Returning results to the Frontend.

The Backend acts as the primary intermediary between the Frontend and internal services.

##2.4 API
The API allows the Frontend to request operations from the Backend. Communication occurs over HTTP.

Concept:
Frontend
│
│ HTTP Request
▼
API
│
▼
Backend

### 2.5 Database
The Database stores structured application data, including:
* User
* Article
* Image metadata
* Analysis
* Comment
* SavedArticle
* Report

Data is managed exclusively by the Backend. 

Frontend ──> Backend ──> Database


### 2.6 Image Storage
Image Storage is dedicated to storing raw image files. It is separated from the Database because images are physical files, whereas the Database primarily stores metadata and references.


The Backend handles the communication between the application logic and the storage layer.

### 2.7 Analysis Server
The Analysis Server manages the asynchronous analysis workflow. It receives analysis requests from the Backend and coordinates with the AI model.

Backend ──> Analysis Server ──> AI Model


It manages the different states of an analysis pipeline:
PENDING ──> PROCESSING ──> COMPLETED

or:
PROCESSING ──> FAILED


### 2.8 AI Model
The AI Model performs the visual analysis. It receives an image and outputs structured data used to construct the ReverseCraftX analysis report:

Image ──> AI Model ──> [Components, Materials, Structure, Hypotheses, Manufacturing Steps]


The AI Model does not handle user management or the user interface.


## 3. Main Communication Flow

The general communication principle follows a strict hierarchical pipeline:

User
↓
Frontend
↓
HTTP / API
↓
Backend
↓
├── Database
├── Image Storage
└── Analysis Server
↓
AI Model


### Key Engineering Principles
* **Separation of Concerns:** Each component has a precise responsibility.
* **Frontend:** Focuses on user interaction and rendering.
* **Backend:** Handles application logic, routing, and orchestration.
* **Database:** Persists structured relational data.
* **Image Storage:** Persists raw media files.
* **Analysis Server:** Manages background processing jobs and AI orchestration.
* **AI Model:** Executes visual inference.
