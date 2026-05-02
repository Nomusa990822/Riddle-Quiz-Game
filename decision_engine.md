# Decision Engine  

The Decision Engine is the core evaluation system responsible for validating answers, computing scores, assigning badges, updating analytics, and persisting results.

It transforms raw user input into structured, deterministic outcomes that drive player progression and system intelligence.

---

## System Properties  

The engine is designed to be:

- **Deterministic** → same input always produces the same output  
- **Layered** → normalization → validation → scoring → aggregation → rewards  
- **Strict-first** → correctness prioritizes precision over approximation  
- **State-aware** → uses session data (hints, timing, streaks)  
- **Analytics-driven** → every decision feeds future optimization  

---

## Decision Pipeline (Full Flow)  

```mermaid
flowchart TD

%% =====================================================
%% USER SUBMISSION
%% =====================================================
START["User submits answers"] --> SUBMIT["POST /api/submit"]

SUBMIT --> ROUTE["game.py<br/>submit_game()"]
ROUTE --> SESSION_LOAD["Load active session"]
SESSION_LOAD --> SESSIONS_DB[("game_sessions<br/>active rounds · hidden answers")]

SESSION_LOAD --> LOOP["Iterate over questions"]

%% =====================================================
%% NORMALIZATION LAYER
%% =====================================================
LOOP --> USER_RAW["User answer"]
LOOP --> TRUE_RAW["Correct answer"]

USER_RAW --> USER_NORM["Normalize user answer<br/>lowercase · trim · punctuation removal"]
TRUE_RAW --> TRUE_NORM["Normalize correct answer"]

%% =====================================================
%% VALIDATION LAYER (STRICT-FIRST)
%% =====================================================
USER_NORM --> EMPTY_CHECK{"Empty answer?"}

EMPTY_CHECK -- "Yes" --> WRONG["Mark incorrect<br/>score = 0"]

EMPTY_CHECK -- "No" --> EXACT_CHECK{"Exact match?"}
TRUE_NORM --> EXACT_CHECK

EXACT_CHECK -- "Yes" --> CORRECT["Mark correct"]

EXACT_CHECK -- "No" --> STRICT_MODE{"Strict mode enabled?"}

STRICT_MODE -- "Yes" --> WRONG

STRICT_MODE -- "No" --> FUZZY["Fuzzy similarity"]
FUZZY --> SIM_SCORE["Compute similarity score"]

SIM_SCORE --> THRESHOLD{"Above threshold?"}
THRESHOLD -- "Yes" --> CORRECT

THRESHOLD -- "No" --> SEMANTIC_CHECK{"Semantic equivalence?"}
SEMANTIC_CHECK -- "Yes" --> CORRECT
SEMANTIC_CHECK -- "No" --> WRONG

%% =====================================================
%% SCORING LAYER
%% =====================================================
CORRECT --> BASE["Base points<br/>Easy=2 · Medium=3 · Hard=5"]

BASE --> STREAK["Apply streak bonus"]
STREAK --> SPEED["Apply speed bonus"]
SPEED --> HINT["Apply hint penalty"]

HINT --> QUESTION_SCORE["Final question score"]

WRONG --> ZERO_SCORE["Final question score = 0"]

%% =====================================================
%% ROUND AGGREGATION
%% =====================================================
QUESTION_SCORE --> ROUND_ACCUM["Accumulate total score"]
ZERO_SCORE --> ROUND_ACCUM

ROUND_ACCUM --> METRICS["Compute metrics<br/>accuracy · max streak · avg response time"]

%% =====================================================
%% BADGE ENGINE
%% =====================================================
METRICS --> BADGE_ENGINE["badge_engine.py<br/>build_round_badges()"]

BADGE_ENGINE --> PERFECT{"Perfect Round"}
BADGE_ENGINE --> ELITE{"Elite Score"}
BADGE_ENGINE --> STREAK_BADGE{"High streak"}
BADGE_ENGINE --> HINT_BADGE{"No hints used"}
BADGE_ENGINE --> SPEED_BADGE{"Fast performance"}

PERFECT --> BADGE_LIST
ELITE --> BADGE_LIST
STREAK_BADGE --> BADGE_LIST
HINT_BADGE --> BADGE_LIST
SPEED_BADGE --> BADGE_LIST

BADGE_LIST["Assemble awarded badges"]

%% =====================================================
%% PERSISTENCE LAYER
%% =====================================================
subgraph DATA["Persistent Data Layer"]
    DB_ABS["Database Abstraction"]

    SQLITE[("SQLite<br/>local development")]
    POSTGRES[("PostgreSQL<br/>production database")]

    HISTORY_DB[("game_history<br/>scores · accuracy · badges · streaks")]
    ANALYTICS_DB[("riddle_analytics<br/>similarity · timing · hints")]
    LEADERBOARD_DB[("leaderboard<br/>ranked results")]
end

BADGE_LIST --> HISTORY_WRITE["Persist round results"]
BADGE_LIST --> ANALYTICS_WRITE["Persist per-question analytics"]

HISTORY_WRITE --> DB_ABS
ANALYTICS_WRITE --> DB_ABS

DB_ABS --> SQLITE
DB_ABS --> POSTGRES

SQLITE --> HISTORY_DB
SQLITE --> ANALYTICS_DB
SQLITE --> LEADERBOARD_DB

POSTGRES --> HISTORY_DB
POSTGRES --> ANALYTICS_DB
POSTGRES --> LEADERBOARD_DB

%% =====================================================
%% LEADERBOARD DECISION
%% =====================================================
HISTORY_WRITE --> LB_CHECK{"Eligible for leaderboard?"}

LB_CHECK -- "Yes" --> LB_WRITE["Insert leaderboard entry"]
LB_CHECK -- "No" --> RESULTS_BUILD

LB_WRITE --> LEADERBOARD_DB
LEADERBOARD_DB --> RANKING["Compute ranking position"]

%% =====================================================
%% FINAL RESPONSE
%% =====================================================
RESULTS_BUILD["Build response payload<br/>score · breakdown · badges · feedback"]

RANKING --> RESULTS_BUILD

RESULTS_BUILD --> DELETE_SESSION["Delete session"]
DELETE_SESSION --> SESSIONS_DB

%% =====================================================
%% FRONTEND UPDATE
%% =====================================================
RESULTS_BUILD --> UI["Frontend renders results"]

UI --> SCORE_VIEW["Score + breakdown"]
UI --> BADGE_VIEW["Badge display"]
UI --> TABLE_VIEW["Per-question results"]
UI --> EXPLAIN_VIEW["Explanation panel"]

%% =====================================================
%% DASHBOARD REFRESH
%% =====================================================
UI --> REFRESH["Refresh dashboard"]

REFRESH --> LOAD_HISTORY["Reload history"]
REFRESH --> LOAD_LB["Reload leaderboard"]
REFRESH --> LOAD_STATS["Reload player stats"]
REFRESH --> LOAD_GLOBAL["Reload global rankings"]

LOAD_HISTORY --> HISTORY_DB
LOAD_LB --> LEADERBOARD_DB

LOAD_HISTORY --> END["Decision cycle complete"]
LOAD_LB --> END
LOAD_STATS --> END
LOAD_GLOBAL --> END
