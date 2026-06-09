# Agent Success Rate — Metric Spec

> Output artifact. Fill every section. Keep it short enough that an eng lead and a PM can both read it in one sitting.

## Agent
- **Name / surface:**
- **What it does (1–2 lines):**
- **FSM summary:** states → transitions → terminal state(s)
- **Unit of analysis:** turn | session | task — _and its boundaries_
- **How a run ends:** explicit done | new run | idle timeout (which exist?)

## Success definition
One sentence: a run succeeds when …

## Pillars & gates

### User Actions
- **Signals used:**
- **Gate (pass condition):**
- **Present when:** / **Absent when:**

### User Sentiment
- **Signals used:** explicit | inferred
- **Gate (pass condition):**
- **Present when:** / **Absent when:**

### Evals
- **Signals used:**
- **Gate (pass condition):**
- **Present when:** / **Absent when:**

> If any pillar is intentionally omitted, state which and why here.

## Combination
- **Strategy:** mean of present gates | weighted | conjunctive | disjunctive | hard-gate + score
- **Formula:**
- **Weights / hard gate (if any):**
- **Reported as:** mean per-run score | % runs ≥ threshold (state threshold)
- **Denominator (eligible runs):**

## Caveats
- **Minimum N before reporting:**
- **Known blind spots:**
- **Recompute / revisit when:**
