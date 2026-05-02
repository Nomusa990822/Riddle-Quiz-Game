# System Flow

This diagram shows the complete runtime path from landing page entry, authentication, dashboard navigation, game option loading, riddle selection or AI generation, answer validation, scoring, badge progression, persistence, leaderboard updates, and frontend refresh.

The system supports both local development and production deployment:

- **SQLite** is used for local development and lightweight testing.
- **PostgreSQL** is used for production persistence.
- The database access layer abstracts the active environment so gameplay, authentication, scoring, history, and ranking logic can use the same persistence interface.

```mermaid
flowchart TD

%% =====================================================
%% ENTRY / LANDING PAGE
%% =====================================================
START["User opens Riddle Quiz Game"] --> LANDING["Landing Page<br/>landing.html · favicon · product entry"]

LANDING --> LANDING_AUTH{"Stored auth token?"}

LANDING_AUTH -- "No" --> LOGIN_CTA["Login / Register CTA"]
LOGIN_CTA --> APP_AUTH["Navigate to /app?auth=login"]

LANDING_AUTH -- "Yes" --> DASH_CTA["Dashboard CTA"]
DASH_CTA --> APP["Gameplay Dashboard<br/>index.html · style.css · app.js"]

APP_AUTH --> APP

%% =====================================================
%% FRONTEND APPLICATION BOOT
%% =====================================================
APP --> INIT["Frontend App Boot<br/>load theme · restore user · attach listeners"]
INIT --> AUTH_CHECK{"Authenticated?"}

AUTH_CHECK -- "No" --> AUTH_MODAL["Auth Modal<br/>login · register · reset password"]
AUTH_CHECK -- "Yes" --> LOAD_OPTIONS["Load available game options"]

%% =====================================================
%% AUTHENTICATION FLOW
%% =====================================================
AUTH_MODAL --> AUTH_ACTION{"Auth action"}

AUTH_ACTION -- "Register" --> REGISTER_API["POST /api/auth/register"]
AUTH_ACTION -- "Login" --> LOGIN_API["POST /api/auth/login"]
AUTH_ACTION -- "Refresh" --> REFRESH_API["POST /api/auth/refresh"]
AUTH_ACTION -- "Logout" --> LOGOUT_API["POST /api/auth/logout"]

REGISTER_API --> AUTH_ROUTE["auth.py"]
LOGIN_API --> AUTH_ROUTE
REFRESH_API --> AUTH_ROUTE
LOGOUT_API --> AUTH_ROUTE

AUTH_ROUTE --> AUTH_SERVICE["auth_service.py<br/>password hashing · JWT creation · refresh rotation · token revocation"]

AUTH_SERVICE --> DBSTORE_AUTH["db_storage.py<br/>auth persistence operations"]

%% =====================================================
%% DATABASE ENVIRONMENT
%% =====================================================
subgraph DATA["Persistent Data Layer"]
    DB_ABS["Database Abstraction<br/>environment-aware storage"]

    SQLITE[("SQLite<br/>local development")]
    POSTGRES[("PostgreSQL<br/>production database")]

    USERS[("users<br/>accounts · avatars · profile data")]
    TOKENS[("refresh_tokens<br/>refresh state · revocation")]
    SESSIONS[("game_sessions<br/>active rounds · hidden answers")]
    USED[("used_questions<br/>anti-repetition memory")]
    HISTORY[("game_history<br/>scores · accuracy · badges · streaks")]
    ANALYTICS[("riddle_analytics<br/>similarity · timing · hint usage")]
end

DBSTORE_AUTH --> DB_ABS
DB_ABS --> SQLITE
DB_ABS --> POSTGRES

SQLITE --> USERS
SQLITE --> TOKENS
SQLITE --> SESSIONS
SQLITE --> USED
SQLITE --> HISTORY
SQLITE --> ANALYTICS

POSTGRES --> USERS
POSTGRES --> TOKENS
POSTGRES --> SESSIONS
POSTGRES --> USED
POSTGRES --> HISTORY
POSTGRES --> ANALYTICS

AUTH_SERVICE --> USERS
AUTH_SERVICE --> TOKENS

AUTH_SERVICE --> AUTH_SUCCESS["Return access token · refresh token · user profile"]
AUTH_SUCCESS --> INIT
AUTH_SUCCESS --> LOAD_OPTIONS

%% =====================================================
%% OPTIONS FLOW
%% =====================================================
LOAD_OPTIONS --> OPTIONS_API["GET /api/options"]
OPTIONS_API --> GAME_ROUTE_OPTIONS["game.py<br/>options endpoint"]

GAME_ROUTE_OPTIONS --> DATASET["dataset.py<br/>load and normalize curated riddles"]
DATASET --> RIDDLES_CSV[("riddles.csv<br/>curated riddle dataset")]

RIDDLES_CSV --> OPTIONS_MAP["Build options map<br/>mode → difficulty → category"]
OPTIONS_MAP --> FORM["Populate selectors<br/>mode · difficulty · category"]

%% =====================================================
%% START ROUND
%% =====================================================
FORM --> START_CLICK["User clicks Start Game"]
START_CLICK --> START_API["POST /api/start"]

START_API --> GAME_ROUTE["game.py<br/>validate request · user · mode · difficulty · category"]
GAME_ROUTE --> DBSTORE_GAME["db_storage.py<br/>session · history · used-question operations"]

DBSTORE_GAME --> DB_ABS

GAME_ROUTE --> ACTIVE_STATE["Read active sessions and recently used questions"]
ACTIVE_STATE --> SESSIONS
ACTIVE_STATE --> USED

ACTIVE_STATE --> ORCH["game_orchestrator.py<br/>prepare_riddles() · mode routing · selection policy"]

%% =====================================================
%% MODE ROUTING
%% =====================================================
ORCH --> MODE_CHECK{"Selected game mode"}

MODE_CHECK -- "Classic" --> CLASSIC["Classic Mode<br/>standard difficulty-based round"]
MODE_CHECK -- "Timed" --> TIMED["Timed Mode<br/>shorter limits · speed pressure"]
MODE_CHECK -- "Sudden Death" --> SUDDEN["Sudden Death<br/>first wrong answer ends run"]
MODE_CHECK -- "Endless" --> ENDLESS["Endless Mode<br/>progressive round flow"]
MODE_CHECK -- "Scenarios" --> SCENARIOS["Scenario Mode<br/>super-hard reasoning questions"]
MODE_CHECK -- "AI" --> AI_PIPE["Generative AI Pipeline"]

%% =====================================================
%% GENERATIVE AI PIPELINE
%% =====================================================
AI_PIPE --> AI_CONTEXT["Build AI generation context<br/>difficulty · category · banned questions · answer diversity"]
AI_CONTEXT --> AI_PROMPT["Prompt Engineering Layer<br/>rules · output format · constraints"]
AI_PROMPT --> GENERATOR["generator.py<br/>Groq LLM generation"]

GENERATOR --> RAW_AI["Raw AI candidates"]
RAW_AI --> REFINER["riddle_refiner.py<br/>rewrite · normalize · improve clarity"]
REFINER --> VALIDATOR["riddle_validator.py<br/>structure · leakage · answer validity"]
VALIDATOR --> QUALITY["riddle_quality.py<br/>clarity · ambiguity · difficulty fit"]
QUALITY --> SEMANTIC["semantic_uniqueness.py<br/>semantic duplicate filtering"]

SEMANTIC --> AI_READY{"Enough valid AI riddles?"}

AI_READY -- "No" --> DATASET_FALLBACK["Fallback to curated dataset<br/>controlled support only"]
DATASET_FALLBACK --> RIDDLES_CSV

AI_READY -- "Yes" --> AI_SELECT["AI candidate pool"]
RIDDLES_CSV --> MERGED_POOL["Merged candidate pool<br/>AI + curated fallback"]
AI_SELECT --> MERGED_POOL

%% =====================================================
%% NON-AI SELECTION PIPELINE
%% =====================================================
CLASSIC --> DATASET_POOL["Dataset candidate pool"]
TIMED --> DATASET_POOL
SUDDEN --> DATASET_POOL
ENDLESS --> DATASET_POOL
SCENARIOS --> SCENARIO_POOL["Scenario candidate pool<br/>super-hard category"]

DATASET_POOL --> RIDDLES_CSV
SCENARIO_POOL --> RIDDLES_CSV

RIDDLES_CSV --> SELECT_PIPE["Selection Pipeline<br/>dedupe · answer diversity · fatigue ranking"]
MERGED_POOL --> SELECT_PIPE

SELECT_PIPE --> RECENT_CHECK{"Recently used or reserved?"}
USED --> RECENT_CHECK
SESSIONS --> RECENT_CHECK

RECENT_CHECK -- "Yes" --> SELECT_PIPE
RECENT_CHECK -- "No" --> FINAL_SET["Final round question set"]

%% =====================================================
%% SESSION CREATION
%% =====================================================
FINAL_SET --> PAYLOAD_BUILD["Build frontend payload<br/>questions only · answers hidden"]
FINAL_SET --> SESSION_CREATE["Create secure session<br/>hidden answers · timing · hints · metadata"]

SESSION_CREATE --> SESSIONS
FINAL_SET --> USED_WRITE["Store selected questions as used/reserved"]
USED_WRITE --> USED

PAYLOAD_BUILD --> FRONTEND_GAME["Render active round in frontend"]

%% =====================================================
%% PLAY LOOP
%% =====================================================
FRONTEND_GAME --> USER_ACTION{"User action"}

USER_ACTION -- "Request hint" --> HINT_API["POST /api/hint"]
HINT_API --> GAME_ROUTE
GAME_ROUTE --> HINTS["hints.py<br/>hint limits · context-aware hint generation"]
HINTS --> EXPLANATIONS[("explanations.json<br/>hint anchors · feedback templates")]
EXPLANATIONS --> HINT_RESPONSE["Return hint · update hint count"]
HINT_RESPONSE --> FRONTEND_GAME

USER_ACTION -- "Answer question" --> ANSWER_BUFFER["Store answer locally<br/>answer text · time taken"]
ANSWER_BUFFER --> ROUND_COMPLETE{"Round complete?"}

USER_ACTION -- "Sudden Death answer" --> CHECK_API["POST /api/check-answer"]
CHECK_API --> GAME_ROUTE
GAME_ROUTE --> LIVE_SESSION_READ["Read hidden answer for current question"]
LIVE_SESSION_READ --> SESSIONS

LIVE_SESSION_READ --> LIVE_MATCH["answer_matcher.py<br/>strict live validation"]
LIVE_MATCH --> LIVE_SCORE["scoring.py<br/>live score · streak state"]
LIVE_SCORE --> SUDDEN_WRONG{"Wrong answer?"}

SUDDEN_WRONG -- "Yes" --> SUBMIT_API["POST /api/submit"]
SUDDEN_WRONG -- "No" --> FRONTEND_GAME

ROUND_COMPLETE -- "No" --> FRONTEND_GAME
ROUND_COMPLETE -- "Yes" --> SUBMIT_API

%% =====================================================
%% SUBMISSION AND EVALUATION
%% =====================================================
SUBMIT_API --> GAME_ROUTE
GAME_ROUTE --> SESSION_READ["Load completed session<br/>hidden answers · metadata"]
SESSION_READ --> SESSIONS

SESSION_READ --> MATCH["answer_matcher.py<br/>strict normalization · exact match · similarity"]
MATCH --> SCORE["scoring.py<br/>base points · streak bonus · speed bonus · hint penalty"]
SCORE --> BADGE_ENGINE["Badge Engine<br/>round badges · vault progress · unlock events"]

BADGE_ENGINE --> EXPLAIN_ENGINE["explanations.py<br/>adaptive feedback · scenario explanations"]
MATCH --> EXPLAIN_ENGINE

EXPLAIN_ENGINE --> RESULT_PAYLOAD["Build result payload<br/>score · accuracy · streak · feedback · badges · explanations"]

%% =====================================================
%% PERSISTENCE AFTER ROUND
%% =====================================================
RESULT_PAYLOAD --> SAVE_HISTORY["Persist completed round"]
SAVE_HISTORY --> HISTORY

MATCH --> SAVE_ANALYTICS["Persist answer analytics<br/>similarity · timing · hints"]
SAVE_ANALYTICS --> ANALYTICS

RESULT_PAYLOAD --> DELETE_SESSION["Delete completed session"]
DELETE_SESSION --> SESSIONS

%% =====================================================
%% RANKING AND DASHBOARD REFRESH
%% =====================================================
HISTORY --> LEADERBOARD["Compute leaderboard<br/>eligible scores · rank positions"]
HISTORY --> GLOBAL["Compute global rankings<br/>aggregate performance"]
HISTORY --> HISTORY_UI["Recent history<br/>latest completed rounds"]
HISTORY --> STATS["Player stats<br/>accuracy · consistency · streaks · badges"]
HISTORY --> VAULT["Badge vault<br/>progress · unlock state · next unlock"]

LEADERBOARD --> FRONTEND_REFRESH["Frontend refresh"]
GLOBAL --> FRONTEND_REFRESH
HISTORY_UI --> FRONTEND_REFRESH
STATS --> FRONTEND_REFRESH
VAULT --> FRONTEND_REFRESH
RESULT_PAYLOAD --> FRONTEND_REFRESH

FRONTEND_REFRESH --> APP
