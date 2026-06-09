---
name: agent-success-rate
description: Helps a PM define an agent success-rate metric for a product agent and the analytics instrumentation to track it. Reviews the agent's FSM (from code or a description), recommends a metric built from three pillars — User Actions, User Sentiment, and Evals — lets the PM choose how they combine, then produces a metric spec and a tool-agnostic instrumentation plan. Use when asked to "define agent success", "measure if my agent is working", "agent success rate", "instrument my agent's FSM", or "what should I track for this agent".
argument-hint: "[path-to-agent-code or description]"
allowed-tools: Read, Grep, Glob
---

# Agent Success Rate

Define a defensible **success-rate metric** for a product agent and the **instrumentation** needed to track it. Output is two documents — a metric spec and an instrumentation plan — never code changes.

This skill is **opinionated on the what** (every metric reckons with three pillars) and **flexible on the how** (the PM chooses how the pillars combine). When agent code is available, it inspects the FSM and **leads with a concrete recommendation** the PM accepts or overrides.

## The three pillars (required)

Every success-rate metric this skill produces must account for all three. Each catches what the others miss — do not let a metric ship that silently ignores one (call out the gap explicitly instead).

| Pillar | Answers | Typical signals |
| --- | --- | --- |
| **User Actions** | Did the user *act* on the output? (revealed preference) | share, export, copy, accept/apply, save, continued use, downstream conversion |
| **User Sentiment** | How did the user *feel*? | explicit thumbs/rating; inferred tone (LLM-judge on user messages) |
| **Evals** | Was the output *actually good*, independent of the user? | LLM-as-judge, rule checks, groundedness, schema/format validity, tool-call success |

See `references/methodology.md` for per-pillar signal menus and how to turn each into a pass/fail gate (including the "no evidence" rule, which matters for combination).

## Workflow

Run these steps in order. Prefer asking the PM one focused question at a time over guessing.

### 1. Map the agent's FSM
- If given a path/repo, read it. Otherwise interview the PM.
- Follow `references/fsm-review-checklist.md` to extract: states, transitions, terminal/success state(s), available tools/actions, and the **unit of analysis** (turn vs. session vs. task) and where a "run" begins and ends.
- Produce a short FSM summary and confirm it with the PM before continuing.

### 2. Detect signals already present (code only)
- Scan for evidence of each pillar already in the code (see the detection heuristics in `references/methodology.md` → "Detect-and-recommend"). Examples: export/share/thumbs handlers (Actions), feedback endpoints or sentiment/LLM-judge code (Sentiment), groundedness/schema/eval checks (Evals).
- Note which pillars are **instrumented**, **partially instrumented**, or **absent**.

### 3. Recommend a metric (then let the PM steer)
- **Lead with a recommendation** derived from step 2: which signals to use per pillar, and a recommended combination strategy with a one-line rationale.
- Default combination is **mean of present gates** unless the FSM suggests otherwise (e.g. a correctness-critical agent → recommend Evals as a hard gate). See `references/methodology.md` → "Combination strategies".
- Present the alternatives as explicit choices and let the PM override any part. Confirm the final per-pillar gates and combination formula.

### 4. Back-map to instrumentation
- For every selected signal, specify the event/property required, when it fires, and whether it exists today (gap analysis).
- Apply `references/instrumentation-patterns.md`: the **session-end keystone** (most rollups/classifiers key off an explicit end signal), naming conventions, and trigger/lifecycle gotchas. Keep it tool-agnostic; include tool-specific notes only as examples.

### 5. Emit the two artifacts
- Fill `templates/metric-spec.md` and `templates/instrumentation-plan.md`.
- Write them to the working directory (or print them) — do not modify application code.

## Output rules
- Tool-agnostic by default: events, properties, and firing points the PM can map to any analytics tool.
- Be explicit about evidence gaps and low-volume caveats — a success rate is meaningless until enough sessions carry the required signals.
- A worked end-to-end example is in `examples/mapper3-walkthrough.md`.
