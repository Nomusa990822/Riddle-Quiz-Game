# System Architecture

Riddle Quiz Game is a full-stack generative AI reasoning platform built around secure authentication, structured gameplay modes, strict answer validation, adaptive scoring, leaderboard progression, badge-based achievements, and an AI-assisted riddle generation pipeline.

The system is designed as a modular architecture where the frontend, API routes, authentication layer, gameplay engine, AI intelligence layer, and persistence layer are separated for maintainability, scalability, and deployment readiness.

The application supports local development using SQLite and production deployment using PostgreSQL.

```mermaid
flowchart TD

%% =====================================================
%% CLIENT / PRESENTATION LAYER
%% =====================================================
subgraph CLIENT["Frontend / Presentation Layer"]
    LANDING["landing.html<br/>Public landing page · product entry"]
    UI["index.html<br/>Gameplay dashboard · results · stats"]
    CSS["style.css<br/>Glass UI · responsive design · light/dark themes"]
    JS["app.js<br/>Client state · API requests · dynamic rendering"]
end

LANDING --> CSS
LANDING --> JS
UI --> CSS
UI --> JS

%% =====================================================
%% FASTAPI APPLICATION LAYER
%% =====================================================
subgraph API["FastAPI Application Layer"]
    MAIN["main.py<br/>App startup · router registration · static files"]

    subgraph ROUTES["Route Layer"]
        AUTH_R["auth.py<br/>Register · login · refresh · logout · current user"]
        GAME_R["game.py<br/>Options · start game · submit answers · hints · reset"]
        LEAD_R["leaderboard.py<br/>Leaderboard · history · global rankings"]
    end
end

JS -->|"Authentication requests"| AUTH_R
JS -->|"Gameplay requests"| GAME_R
JS -->|"Ranking and analytics requests"| LEAD_R

MAIN --> AUTH_R
MAIN --> GAME_R
MAIN --> LEAD_R

%% =====================================================
%% AUTHENTICATION AND SECURITY
%% =====================================================
subgraph AUTH["Authentication and Security Layer"]
    AUTH_S["auth_service.py<br/>Password hashing · JWT access tokens · refresh tokens"]
    TOKEN["Token Lifecycle<br/>access expiry · refresh rotation · logout invalidation"]
end

AUTH_R --> AUTH_S
AUTH_S --> TOKEN

%% =====================================================
%% GAME ENGINE
%% =====================================================
subgraph ENGINE["Gameplay Engine Layer"]
    ORCH["game_orchestrator.py<br/>Mode routing · riddle selection · anti-repetition"]
    DATASET["dataset.py<br/>CSV loading · riddle normalization"]
    MATCH["answer_matcher.py<br/>Strict matching · normalization · similarity scoring"]
    SCORE["scoring.py<br/>Base points · streak bonus · speed bonus · hint penalty"]
    HINTS["hints.py<br/>Smart hints · hint limits"]
    BADGE_ENGINE["badge_engine.py<br/>Round badges · vault progress · unlock logic"]
    DBSTORE["db_storage.py<br/>Persistence abstraction · database operations"]
end

GAME_R --> ORCH
GAME_R --> HINTS
GAME_R --> MATCH
GAME_R --> SCORE
GAME_R --> BADGE_ENGINE
GAME_R --> DBSTORE

ORCH --> DATASET
ORCH --> DBSTORE
ORCH --> MATCH
MATCH --> SCORE
SCORE --> BADGE_ENGINE
LEAD_R --> DBSTORE
AUTH_S --> DBSTORE

%% =====================================================
%% GENERATIVE AI / INTELLIGENCE LAYER
%% =====================================================
subgraph AI["Generative AI Intelligence Layer"]
    GEN["generator.py<br/>Groq-powered riddle generation"]
    REFINE["riddle_refiner.py<br/>LLM refinement · wording improvement"]
    VALIDATOR["riddle_validator.py<br/>Structure checks · answer leakage checks"]
    QUALITY["riddle_quality.py<br/>Clarity · difficulty · ambiguity evaluation"]
    SEMANTIC["semantic_uniqueness.py<br/>Semantic duplicate filtering"]
    EXPLAIN["explanations.py<br/>Adaptive explanations · scenario reasoning"]
end

ORCH -->|"AI mode request"| GEN
GEN --> REFINE
REFINE --> VALIDATOR
VALIDATOR --> QUALITY
QUALITY --> SEMANTIC
SEMANTIC --> ORCH

GAME_R --> EXPLAIN
HINTS --> EXPLAIN
ORCH --> EXPLAIN

%% =====================================================
%% GAME MODES
%% =====================================================
subgraph MODES["Game Mode System"]
    CLASSIC["Classic<br/>standard difficulty-based rounds"]
    AI_MODE["AI Mode<br/>generated · refined · validated riddles"]
    TIMED["Timed<br/>shorter time pressure"]
    SUDDEN["Sudden Death<br/>first wrong answer ends round"]
    ENDLESS["Endless<br/>continuous progression loop"]
    SCENARIOS["Scenarios<br/>super-hard reasoning mode"]
end

ORCH --> CLASSIC
ORCH --> AI_MODE
ORCH --> TIMED
ORCH --> SUDDEN
ORCH --> ENDLESS
ORCH --> SCENARIOS

AI_MODE --> GEN
SCENARIOS --> EXPLAIN

%% =====================================================
%% DATA SOURCES
%% =====================================================
subgraph SOURCES["Static Data Sources"]
    RIDDLES[("riddles.csv<br/>Curated riddle dataset")]
    EXPLANATIONS[("explanations.json<br/>Explanation templates")]
end

DATASET --> RIDDLES
ORCH --> RIDDLES
EXPLAIN --> EXPLANATIONS
EXPLANATIONS --> HINTS

%% =====================================================
%% PERSISTENCE LAYER
%% =====================================================
subgraph DATA["Persistent Data Layer"]
    DB_ABS["Database Abstraction<br/>dev and production persistence"]

    SQLITE[("SQLite<br/>Local development database")]
    POSTGRES[("PostgreSQL<br/>Production database")]

    USERS[("users<br/>accounts · avatars · auth identity")]
    TOKENS[("refresh_tokens<br/>token state · revocation")]
    SESSIONS[("game_sessions<br/>active rounds · answers · hint state")]
    HISTORY[("game_history<br/>scores · accuracy · badges · streaks")]
    USED[("used_questions<br/>anti-repetition memory")]
    ANALYTICS[("riddle_analytics<br/>similarity · timing · hints")]
end

DBSTORE --> DB_ABS
DB_ABS --> SQLITE
DB_ABS --> POSTGRES

SQLITE --> USERS
SQLITE --> TOKENS
SQLITE --> SESSIONS
SQLITE --> HISTORY
SQLITE --> USED
SQLITE --> ANALYTICS

POSTGRES --> USERS
POSTGRES --> TOKENS
POSTGRES --> SESSIONS
POSTGRES --> HISTORY
POSTGRES --> USED
POSTGRES --> ANALYTICS

AUTH_S --> USERS
TOKEN --> TOKENS
GAME_R --> SESSIONS
ORCH --> USED
MATCH --> ANALYTICS
SCORE --> HISTORY
BADGE_ENGINE --> HISTORY
LEAD_R --> HISTORY

%% =====================================================
%% USER OUTPUT / PRODUCT LAYER
%% =====================================================
subgraph OUTPUT["User-Facing Output Layer"]
    RESULTS["Round Results<br/>score · accuracy · streak · feedback"]
    BADGES["Badge Vault<br/>progress · unlocks · earned achievements"]
    BOARD["Leaderboard<br/>round-based ranking"]
    GLOBAL["Global Rankings<br/>aggregated player performance"]
    HISTORY_UI["Recent History<br/>latest completed rounds"]
    STATS["Player Stats<br/>accuracy · consistency · streaks · badges"]
end

HISTORY --> RESULTS
HISTORY --> BADGES
HISTORY --> BOARD
HISTORY --> GLOBAL
HISTORY --> HISTORY_UI
HISTORY --> STATS

RESULTS --> JS
BADGES --> JS
BOARD --> JS
GLOBAL --> JS
HISTORY_UI --> JS
STATS --> JS
