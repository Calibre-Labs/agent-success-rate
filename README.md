# agent-success-rate

A [Claude Code skill](https://code.claude.com/docs/en/skills) that helps a PM define a **success-rate metric** for a product agent and the **analytics instrumentation** to track it.

It reviews the agent's state machine (from code or a description), then guides you to a defensible metric built from three pillars:

- **User Actions** — did the user act on the output? (revealed preference)
- **User Sentiment** — how did the user feel? (explicit or inferred)
- **Evals** — was the output actually good, independent of the user?

The skill is **opinionated on the what** (all three pillars are considered) and **flexible on the how** (you choose how they combine — mean, weighted, conjunctive, disjunctive, or hard-gate). When agent code is available it inspects the FSM and **leads with a concrete recommendation** you accept or override.

Output is two documents — a **metric spec** and a **tool-agnostic instrumentation plan**. It never edits your application code.

## Install

Clone into your personal skills directory:

```bash
git clone https://github.com/<owner>/agent-success-rate.git ~/.claude/skills/agent-success-rate
```

Restart Claude Code (a newly created top-level skills directory needs a restart to be watched). Then either let Claude trigger it, or run it directly:

```text
/agent-success-rate
```

You can pass a path or a description:

```text
/agent-success-rate ./path/to/agent
```

## What's inside

| File | Purpose |
| --- | --- |
| `SKILL.md` | Entry point: the workflow Claude follows |
| `references/methodology.md` | The three pillars, gate design, and combination strategies |
| `references/fsm-review-checklist.md` | Extracting the FSM from code or via PM interview |
| `references/instrumentation-patterns.md` | Tool-agnostic event taxonomy + the session-end keystone |
| `templates/metric-spec.md` | Output: the metric definition |
| `templates/instrumentation-plan.md` | Output: events/properties + gap analysis |
| `examples/mapper3-walkthrough.md` | A full worked example |

## License

MIT — see [LICENSE](LICENSE).
