# Methodology: the three-pillar agent success rate

A product agent succeeds when the user got what they came for. That is hard to read from a single signal, so this method triangulates three pillars, turns each into one or more **pass/fail gates**, and combines the gates into a per-run score. The **success rate** is the aggregate of those per-run scores over a population (mean score, or % of runs above a threshold).

Keep one rule in mind throughout: **score only the gates you have evidence for.** "No signal" is not "fail." How missing gates are handled is the crux of the combination step.

---

## The three pillars (required)

### 1. User Actions — revealed preference
Did the user *do* something that only makes sense if the output was useful? The hardest signal to fake.

Signal menu (pick what exists / is worth instrumenting):
- **Adopt**: export, download, copy, save, "accept"/"apply", commit, send.
- **Engage**: continued turns in the same task, returns within N days, follow-up refinement.
- **Convert**: a downstream business action (purchase, publish, ticket created, deal advanced).
- **Explicit positive**: share to a colleague, bookmark/pin.

Binarize: define the action(s) that count as a pass for one run. Example gate — "user exported OR shared the result." Absent when the run produced no artifact the user *could* act on.

### 2. User Sentiment — how it felt
Explicit or inferred. Catches "technically fine but frustrating."

Signal menu:
- **Explicit**: thumbs up/down, star rating, CSAT, survey.
- **Inferred**: LLM-judge over the user's own messages (frustration, gratitude, repetition, "no, I meant…").
- **Behavioral proxy**: rage-repeat of the same request, abandonment mid-task.

Binarize: pass = net-positive (e.g. thumbs-up with no thumbs-down; inferred sentiment ≥ neutral). Absent when the user left no explicit feedback and there is too little text to infer.

### 3. Evals — was it actually good
A judgment of the output independent of the user, so silent-but-wrong runs still fail.

Signal menu:
- **LLM-as-judge**: rubric scoring of the final answer (relevance, completeness, correctness).
- **Rule checks**: schema/format validity, required-field presence, no forbidden content.
- **Groundedness**: claims supported by retrieved/searched sources; citations resolve.
- **Tool correctness**: required tools were called and succeeded; outputs parse.

Binarize: pass = meets the rubric / all hard checks pass. Always present if you run an eval; treat as absent only when no eval was run for that run.

> A metric that omits a pillar is allowed only if you say so out loud and explain why (e.g. "no eval harness yet — Evals gate deferred, success rate currently reads Actions + Sentiment only"). Default is to instrument the gap.

---

## Combination strategies (choose one)

How the per-pillar gates roll into a single per-run score. Each handles the "present gates only" rule.

| Strategy | Formula (over *present* gates) | Use when | Watch out |
| --- | --- | --- | --- |
| **Mean of present gates** *(default)* | average of present gate values ∈ {0…1} | general case, sparse signals | a strong pillar can mask a weak one |
| **Weighted average** | Σ(wᵢ·gateᵢ)/Σ(wᵢ) over present | one pillar is the truer north star | weights are a judgment call; document them |
| **Conjunctive (all pass)** | 1 iff every present gate passes, else 0 | high-stakes, any bad dimension = failure | depresses & destabilizes the rate at low volume |
| **Disjunctive (any pass)** | 1 iff any present gate passes | early-stage, low bar on purpose | flatters the agent; easy to plateau |
| **Hard gate + score** | one pillar is pass/fail prerequisite; others form the graded score | correctness is a floor (Evals) | needs that pillar always instrumented |

**Reporting the rate:** either the mean per-run score, or the share of runs scoring ≥ a threshold. State the unit (per session / per task / per turn) and the denominator (which runs are eligible — usually those with ≥1 user turn).

---

## Detect-and-recommend (when code is available)

Inspect the codebase and lead with a recommendation instead of a blank menu. Grep heuristics per pillar:

- **FSM / unit of analysis**: state enums, transition tables/functions, a terminal/"complete" state, session vs. turn IDs, idle-timeout or session-end logic.
- **User Actions**: handlers or events named `export`, `download`, `share`, `copy`, `save`, `accept`, `apply`, `publish`, `convert`.
- **User Sentiment**: `thumbs`, `rating`, `feedback`, `csat`, `sentiment`, or any LLM-judge over user text.
- **Evals**: `eval`, `scorer`, `judge`, `groundedness`, `schema`/`validate`, tool-result parsing, test gates.

Turn findings into a recommendation:
1. **Pillars present** → propose those signals as the starting gates.
2. **Pillars partial/absent** → flag them and recommend the minimal instrumentation to light them up before the rate is trustworthy.
3. **Combination**:
   - Default to **mean of present gates**.
   - If the agent looks correctness-critical (financial, legal, medical, code-execution, irreversible side effects) → recommend **hard gate on Evals + score on the rest**.
   - If only one pillar is instrumented → recommend instrumenting the other two *before* trusting the rate; until then report the single-pillar number with a loud caveat.
4. Present the recommendation with a one-line rationale, then offer the alternatives and let the PM override any gate, weight, or strategy.

---

## Validation & rollout (after data flows)
- **Volume**: a rate is noise until enough eligible runs carry the gates. State a minimum-N before reporting.
- **Gate-presence distribution**: check how often each gate is present; a gate that's almost always absent isn't doing work.
- **Spot-check**: read a sample of passes and fails to confirm the gates match human judgment; tune thresholds.
- **Drift**: re-check when the agent or its tools change — a new tool or state can silently break a gate.
