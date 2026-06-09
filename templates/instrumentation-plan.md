# Agent Success Rate — Instrumentation Plan

> Output artifact. Tool-agnostic. Lists the events/properties needed to compute the metric in the spec, what exists today, and the gaps to close.

## Unit lifecycle
| Event | Fires when | Key properties | Exists today? |
| --- | --- | --- | --- |
| `run_started` | run begins | `run_id`, `user_id`, agent version, context | |
| `run_ended` | every end path | `run_id`, `end_reason`, `success_score`, gate values, `gates_present` | |

> Confirm `run_ended` fires on **all** end paths (explicit / new-run / timeout) and is idempotent.

## Per-gate signals
| Pillar | Signal / event | Fires when | Properties | Exists today? | Gap / action |
| --- | --- | --- | --- | --- | --- |
| Actions | | | | | |
| Sentiment | | | | | |
| Evals | | | | | |

## Score emission
- **Where computed:** at run end, reading accumulated gates
- **Property:** dedicated `success_score` (+ individual gate values, `gates_present`)
- **Flush:** explicit flush on run end

## Gap summary (prioritized)
1.
2.
3.

## Tool mapping (fill in for your stack)
- **Analytics tool:**
- **How session-level scoring/classification is triggered here:**
- **Notes / known gotchas (short IDs, sampling, trace shape):**

## Validation checklist
- [ ] `run_ended` fires once per run on every end path
- [ ] Each gate's signal is queryable and present at expected frequency
- [ ] `success_score` reads on a dedicated property
- [ ] Minimum-N defined before the rate is reported
- [ ] Sample of passes/fails spot-checked against human judgment
