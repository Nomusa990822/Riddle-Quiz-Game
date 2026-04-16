<div align="center">

<img src="https://readme-typing-svg.demolab.com?font=Poppins&weight=700&size=40&duration=2500&pause=1000&color=F0A500&center=true&vCenter=true&width=600&lines=Riddle+Quiz+Game" />

<p align="center">
  <strong>Where classic brain teasers meet intelligent gameplay.</strong>
</p>

<div align="center">

<!-- Tech Stack -->
<img src="https://img.shields.io/badge/Python-3.12+-1e293b?style=for-the-badge&logo=python&logoColor=E6B17E" />
<img src="https://img.shields.io/badge/FastAPI-Backend-1e293b?style=for-the-badge&logo=fastapi&logoColor=E6B17E" />
<img src="https://img.shields.io/badge/JavaScript-Frontend-1e293b?style=for-the-badge&logo=javascript&logoColor=F59E0B" />
<img src="https://img.shields.io/badge/HTML5-Structure-1e293b?style=for-the-badge&logo=html5&logoColor=F59E0B" />
<img src="https://img.shields.io/badge/CSS3-Styling-1e293b?style=for-the-badge&logo=css3&logoColor=F59E0B" />
<img src="https://img.shields.io/badge/JWT-Authentication-1e293b?style=for-the-badge&logo=jsonwebtokens&logoColor=E6B17E" />

<br/><br/>

<!-- Features -->
<img src="https://img.shields.io/badge/AI-Riddle_Generation-0f172a?style=for-the-badge&color=E6B17E" />
<img src="https://img.shields.io/badge/Semantic-Answer_Scoring-0f172a?style=for-the-badge&color=F59E0B" />
<img src="https://img.shields.io/badge/Responsive-Tablet_Optimized-0f172a?style=for-the-badge&color=E6B17E" />
<img src="https://img.shields.io/badge/UI-Premium_Gradient_Theme-0f172a?style=for-the-badge&color=F59E0B" />
<img src="https://img.shields.io/badge/Portfolio-Standout_Project-0f172a?style=for-the-badge&color=E6B17E" />

</div>

</div>

---

## Overview

**Riddle Quiz Game** is a full-stack interactive quiz platform designed to go far beyond a basic riddle app.  
It combines:

- classic dataset-driven gameplay
- AI-assisted riddle generation
- semantic answer matching
- user authentication with JWT
- leaderboard, history, and global ranking systems
- a polished responsive interface optimized for tablet portrait and landscape layouts

This project was built to demonstrate strong skills across:

- backend engineering
- API design
- AI integration
- frontend interaction design
- responsive UI architecture
- product thinking

---

## Core Features

### Gameplay
- Classic riddle rounds
- AI-generated riddle rounds
- Multiple game modes
- Difficulty selection: Easy, Medium, Hard
- Category selection: Logic, Math, Science
- Timer-based question flow
- Hint support
- Skip question option
- Animated score display

### Intelligence Layer
- Retrieval-guided AI riddle generation
- Semantic answer scoring
- Similarity-aware evaluation beyond exact match
- Cached and reusable AI-assisted flow
- Riddle dataset conversion and deduplication

### User System
- Account creation
- Login and logout
- JWT-based authentication
- Avatar selection
- Personalized gameplay identity

### Progress and Rankings
- Leaderboard
- Recent game history
- Global ranking
- Streak tracking
- Accuracy tracking
- Score tracking by round

### UI / UX
- Premium gradient-based theme
- Animated hero title
- Slide-in account sidebar
- Responsive orientation-aware layout
- Tablet portrait and landscape optimization
- Smooth transitions and micro-interactions

---

## Why This Project Stands Out

This is not just a quiz app.

It is a **multi-layered software project** that combines:

- structured backend logic
- intelligent content generation
- semantic evaluation
- account systems
- persistent storage
- responsive interface design

It shows the ability to build a product that feels closer to a real platform than a classroom exercise.

---

## Tech Stack

### Backend
- Python
- FastAPI
- Uvicorn

### Frontend
- HTML
- CSS
- JavaScript

### Authentication
- JWT
- `python-jose`
- `passlib`

### AI / NLP
- Transformers
- Sentence Transformers
- Semantic similarity scoring

### Data Storage
- CSV-based persistence for:
  - users
  - leaderboard
  - history
  - used questions
  - riddles dataset

---

## Project Structure

```text
Riddle-AI/
│
├── app/
│   ├── main.py
│   ├── routes/
│   │   ├── auth.py
│   │   ├── game.py
│   │   └── leaderboard.py
│   │
│   ├── services/
│   │   ├── auth_service.py
│   │   ├── dataset.py
│   │   ├── generator.py
│   │   ├── retrieval.py
│   │   ├── answer_matcher.py
│   │   └── storage.py
│   │
│   ├── static/
│   │   ├── style.css
│   │   └── app.js
│   │
│   └── templates/
│       └── index.html
│
├── data/
│   ├── riddles.csv
│   ├── leaderboard.csv
│   ├── history.csv
│   ├── used_questions.csv
│   └── users.csv
│
├── riddles.txt
├── convert_riddles.py
├── requirements.txt
├── LICENSE
└── README.md
```
---

## System Architecture
```mermaid
flowchart TD
    A[User] --> B[Responsive Frontend<br/>HTML + CSS + JavaScript]
    B --> C[FastAPI Application]

    C --> D[Auth Routes]
    C --> E[Game Routes]
    C --> F[Leaderboard Routes]

    D --> G[JWT Auth Service]
    D --> H[User Storage]

    E --> I[Dataset Service]
    E --> J[AI Generator Service]
    E --> K[Semantic Answer Matcher]
    E --> L[Game Storage]

    F --> M[Leaderboard Storage]
    F --> N[History Storage]
    F --> O[Global Ranking Aggregation]

    I --> P[Riddles CSV Dataset]
    J --> P
    L --> Q[Used Questions CSV]
    H --> R[Users CSV]
    M --> S[Leaderboard CSV]
    N --> T[History CSV]
```
---

## Request Flow
```mermaid
sequenceDiagram
    participant U as User
    participant F as Frontend
    participant API as FastAPI
    participant AUTH as Auth Service
    participant GAME as Game Engine
    participant AI as AI Generator
    participant SCORE as Semantic Matcher
    participant DB as CSV Storage

    U->>F: Open app
    F->>API: Load options / rankings
    API->>DB: Read data
    DB-->>API: Return stored values
    API-->>F: Render homepage

    U->>F: Register / Login
    F->>API: Auth request
    API->>AUTH: Validate credentials
    AUTH-->>API: JWT token
    API-->>F: Auth success

    U->>F: Start game
    F->>API: /api/start
    API->>GAME: Build round

    alt Classic mode
        GAME->>DB: Load riddles from dataset
    else AI mode
        GAME->>AI: Generate retrieval-guided riddles
        AI->>DB: Use riddles dataset as grounding
    end

    API-->>F: Return round questions

    U->>F: Submit answers
    F->>API: /api/submit
    API->>SCORE: Evaluate semantic similarity
    SCORE-->>API: Match results + scoring
    API->>DB: Save history, leaderboard, used questions
    API-->>F: Return round results
```
---

## Game Modes

**1. Classic**
- Uses the structured local riddle dataset.

**2. AI**
- Uses retrieval-guided generation built from the local riddle dataset.

**3. Timed**
- Adds stronger time pressure to each question.

**4. Sudden Death**
- A mistake can end the round early.

**5. Endless**
- Extends the challenge format for repeated play.

---

## Dataset Pipeline

This project supports dataset growth through a text-to-CSV conversion flow.

**Input:**

```difficulty|question|answer```

**Conversion**
- reads from ```riddles.txt```
- removes duplicates
- infers category
- writes cleaned rows to ```data/riddles.csv```

**Categories**
- Logic
- Math
- Science

This gives the project a maintainable content pipeline instead of hardcoded questions.

---

## Authentication Architecture

The authentication system uses:
- registration
- login
- password hashing
- JWT token generation
- protected API endpoints

Protected endpoints require a valid token before a user can:
- start a game
- submit results
- access personalized identity state

---

## Semantic Scoring

Instead of relying only on strict string equality, the project supports semantic-aware answer evaluation.

This makes the game more intelligent by allowing:

- better handling of close answers
- more flexible evaluation logic
- stronger NLP integration

That moves the project beyond a simple quiz checker into a more modern intelligent system.

---

## Responsive Design Strategy

The UI was designed with orientation-aware behavior in mind.
