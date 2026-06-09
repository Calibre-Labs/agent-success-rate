# Worked example: a market-mapping agent

A run of the skill against a real agent — a market-mapping assistant that clarifies scope, then generates outputs (market map, feature matrix, SWOT, funding timeline, news digest) and refines them.

## 1. FSM (extracted from code)
- **States:** NEW → PLANNING → MAPPING → REFINING → COMPLETE (COMPLETE can re-enter PLANNING).
- **Unit of analysis:** **session** (a whole mapping task; may span several turns).
- **How runs end:** explicit (user opens a new chat → old session marked COMPLETE) and an **idle-timeout sweep** (>5 min). Both emit a session-end event.
- **Tools:** a search tool + five output generators. Output generators produce user-actionable artifacts.

## 2. Signals detected
- **User Actions:** present — share links and a markdown export path; thumbs feedback endpoint.
- **User Sentiment:** present — explicit thumbs up/down; an inferred-sentiment pass over user messages.
- **Evals:** partial — groundedness can be derived from whether output generators were backed by search calls, plus an "output tool emitted when required" check. No standalone LLM-judge.

## 3. Recommended metric (lead with this)
Three pass/fail gates, **mean of present gates** (sparse signals; not correctness-critical enough to hard-gate):

- **G1 — User Action:** pass if the session had a thumbs-up **or** a share. Absent if neither possible.
- **G2 — Sentiment:** pass if net-positive (explicit thumbs, else inferred ≥ neutral). Absent if no feedback and too little text to infer.
- **G3 — Clean run (Evals):** pass if no tool failures **and** output-tool-emit compliance **and** generated outputs were grounded in search. Always present.

`success_score = mean(present gates)` → ∈ {0, 0.33, 0.5, 0.67, 1.0}. **Reported as** mean score per session over sessions with ≥1 user turn.

_PM override taken:_ none — accepted as recommended. (Alternatives offered: weight G3 higher, or hard-gate on G3.)

## 4. Instrumentation plan (highlights)
- Emit `session_ended` on both end paths (explicit + timeout), idempotent, carrying `success_score`, the three gate values, and `gates_present`.
- Put the score on a **dedicated** numeric property (a shared high-cardinality score property charts unreliably).
- Actions/Sentiment events already exist; the gap is the per-session **groundedness signal** for G3 — persist search-call counts per turn so the gate is computable.
- Flush analytics + tracing on session end; de-dupe so one session = one score.

## 5. Caveats surfaced
- Low volume: the per-session rate is noisy until enough sessions carry gates; classification/observability features that need a volume threshold won't light up early.
- Backfill of historical sessions skips groundedness (search counts weren't persisted before) — mark those rows so the two cohorts stay distinguishable.
