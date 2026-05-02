# Generative AI Pipeline

The Generative AI pipeline powers AI Mode by generating, refining, validating, filtering, and selecting riddles before they are shown to the player.

The pipeline is intentionally controlled: AI-generated content is not trusted blindly. Every generated riddle must pass structural validation, quality checks, semantic uniqueness filtering, answer-safety checks, and anti-repetition rules before it becomes part of an active round.

The system supports:
- Groq-powered LLM generation
- prompt-driven riddle creation
- refinement and validation loops
- semantic duplicate filtering
- dataset fallback when AI quality is insufficient
- SQLite for local development
- PostgreSQL for production persistence

```mermaid
flowchart TD

%% =====================================================
%% ENTRY POINT
%% =====================================================
START["User selects AI Mode"] --> START_REQ["POST /api/start"]
START_REQ --> GAME_ROUTE["game.py<br/>Start round endpoint"]
GAME_ROUTE --> ORCH["game_orchestrator.py<br/>AI mode orchestration"]

%% =====================================================
%% GENERATION CONTEXT
%% =====================================================
ORCH --> CONTEXT["Generation Context Builder<br/>difficulty · category · round size · used questions · banned answers"]
CONTEXT --> POLICY["Generation Policy<br/>strict answerability · no leakage · difficulty alignment"]
POLICY --> PROMPT["Prompt Engineering Layer<br/>structured templates · constraints · output format"]

%% =====================================================
%% LLM GENERATION
%% =====================================================
PROMPT --> GROQ["Groq LLM API<br/>batch riddle generation"]
GROQ --> RAW_POOL["Raw AI Candidate Pool<br/>question · answer · category · difficulty"]

%% =====================================================
%% REFINEMENT LOOP
%% =====================================================
RAW_POOL --> REFINE["riddle_refiner.py<br/>rewrite · normalize wording · improve clarity"]
REFINE --> STRUCT_VALIDATE["riddle_validator.py<br/>schema · missing fields · answer leakage · category fit"]

STRUCT_VALIDATE --> STRUCT_CHECK{"Structurally valid?"}

STRUCT_CHECK -- "No" --> REPAIR["Repair / regenerate candidate"]
REPAIR --> GROQ

STRUCT_CHECK -- "Yes" --> QUALITY["riddle_quality.py<br/>clarity · ambiguity · solvability · difficulty score"]

QUALITY --> QUALITY_CHECK{"Quality acceptable?"}

QUALITY_CHECK -- "No" --> REPAIR
QUALITY_CHECK -- "Yes" --> ANSWER_SAFE["Answer Safety Check<br/>single expected answer · no obvious leakage · normalized answer"]

ANSWER_SAFE --> ANSWER_CHECK{"Answer usable?"}

ANSWER_CHECK -- "No" --> REPAIR
ANSWER_CHECK -- "Yes" --> SEMANTIC["semantic_uniqueness.py<br/>semantic similarity · duplicate filtering"]

%% =====================================================
%% STATIC SOURCES
%% =====================================================
subgraph STATIC["Static Knowledge Sources"]
    RIDDLES_CSV[("riddles.csv<br/>curated fallback dataset")]
    EXPLANATIONS_JSON[("explanations.json<br/>reasoning · hints · feedback templates")]
end

%% =====================================================
%% PERSISTENCE LAYER
%% =====================================================
subgraph DATA["Persistent Data Layer"]
    DB_ABS["Database Abstraction<br/>environment-aware persistence"]

    SQLITE[("SQLite<br/>local development")]
    POSTGRES[("PostgreSQL<br/>production deployment")]

    USERS[("users<br/>player identity")]
    SESSIONS[("game_sessions<br/>active AI rounds · hidden answers")]
    USED[("used_questions<br/>anti-repetition memory")]
    HISTORY[("game_history<br/>scores · accuracy · badges")]
    ANALYTICS[("riddle_analytics<br/>similarity · timing · hint usage")]
end

DB_ABS --> SQLITE
DB_ABS --> POSTGRES

SQLITE --> USERS
SQLITE --> SESSIONS
SQLITE --> USED
SQLITE --> HISTORY
SQLITE --> ANALYTICS

POSTGRES --> USERS
POSTGRES --> SESSIONS
POSTGRES --> USED
POSTGRES --> HISTORY
POSTGRES --> ANALYTICS

ORCH --> DB_ABS
DB_ABS --> USED

%% =====================================================
%% DUPLICATE AND FATIGUE CONTROL
%% =====================================================
SEMANTIC --> DUP_CHECK{"Recently used, repeated, or too similar?"}
USED --> DUP_CHECK

DUP_CHECK -- "Yes" --> REPAIR
DUP_CHECK -- "No" --> AI_POOL["Validated AI Candidate Pool"]

%% =====================================================
%% FALLBACK CONTROL
%% =====================================================
AI_POOL --> SUFFICIENT{"Enough high-quality AI riddles?"}

SUFFICIENT -- "Yes" --> SELECTOR
SUFFICIENT -- "No" --> FALLBACK["Fallback Loader<br/>controlled curated dataset support"]

FALLBACK --> RIDDLES_CSV
RIDDLES_CSV --> MERGE["Merge AI + Curated Candidates"]
MERGE --> SELECTOR

%% =====================================================
%% FINAL SELECTION
%% =====================================================
SELECTOR["Selection Engine<br/>AI ratio · answer diversity · difficulty balance · category balance · fatigue ranking"] --> FINAL_SET["Final AI Round Set"]

FINAL_SET --> USED_WRITE["Store selected questions as used"]
USED_WRITE --> USED

%% =====================================================
%% EXPLANATION AND HINT METADATA
%% =====================================================
FINAL_SET --> EXPLAIN_META["Explanation Metadata Builder<br/>reasoning anchors · hint anchors · expected answer"]
EXPLAIN_META --> EXPLANATIONS_JSON

EXPLANATIONS_JSON --> HINT_ENGINE["hints.py<br/>context-aware hints"]
EXPLANATIONS_JSON --> EXPLAIN_ENGINE["explanations.py<br/>adaptive feedback · scenario reasoning"]

%% =====================================================
%% SESSION CREATION
%% =====================================================
EXPLAIN_META --> SESSION_CREATE["Create Secure Game Session<br/>questions exposed · answers hidden"]
SESSION_CREATE --> SESSIONS
SESSION_CREATE --> FRONTEND["Return playable riddles to frontend"]

%% =====================================================
%% GAMEPLAY LOOP
%% =====================================================
FRONTEND --> PLAY["User plays AI round"]

PLAY --> HINT_REQUEST{"Hint requested?"}

HINT_REQUEST -- "Yes" --> HINT_ENGINE
HINT_ENGINE --> PLAY

HINT_REQUEST -- "No" --> SUBMIT

PLAY --> SUBMIT["POST /api/submit"]

%% =====================================================
%% EVALUATION
%% =====================================================
SUBMIT --> SESSION_READ["Load session with hidden answers"]
SESSION_READ --> SESSIONS

SESSION_READ --> MATCH["answer_matcher.py<br/>strict normalization · exact match · similarity scoring"]
MATCH --> SCORE["scoring.py<br/>base points · streak bonus · speed bonus · hint penalty"]
SCORE --> BADGES["Badge Engine<br/>round badges · vault progress · unlock events"]
BADGES --> EXPLAIN_ENGINE

EXPLAIN_ENGINE --> RESULT_PAYLOAD["Result Payload<br/>score · accuracy · feedback · badges · explanations"]

%% =====================================================
%% SAVE RESULTS
%% =====================================================
RESULT_PAYLOAD --> HISTORY
RESULT_PAYLOAD --> ANALYTICS

RESULT_PAYLOAD --> SESSION_DELETE["Delete completed session"]
SESSION_DELETE --> SESSIONS

%% =====================================================
%% PLAYER FEEDBACK
%% =====================================================
HISTORY --> STATS["Update Player Stats<br/>accuracy · consistency · streaks"]
HISTORY --> BOARD["Update Leaderboard<br/>rank · score · difficulty"]
HISTORY --> GLOBAL["Update Global Ranking<br/>aggregate performance"]

STATS --> DASHBOARD["Frontend Refresh<br/>stats · leaderboard · badge vault"]
BOARD --> DASHBOARD
GLOBAL --> DASHBOARD
RESULT_PAYLOAD --> DASHBOARD
