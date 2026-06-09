# Instrumentation patterns (tool-agnostic)

How to track the signals each gate needs, in a way that maps onto any analytics or observability tool. Stay neutral by default; the tool-specific notes at the end are examples, not requirements.

## The session-end keystone

Most per-run rollups and out-of-the-box classifiers fire off an **explicit end signal** for the unit of analysis. If a run never signals it ended, the metric either never computes or computes against a partial run.

- Emit an explicit **`<unit> ended`** event (e.g. `session_ended`) at every way a run can finish: explicit "done", a new run opening, and an idle timeout sweep.
- Carry the per-run score (and gate values) on or alongside that event so the rate reads from one row per run.
- Know how your tool decides to run its own scoring/classification — usually one of: on the end event, on span/trace completion, on a sampled background pass, or only after a volume threshold. Design the end signal so those triggers fire on a *complete* run, not an empty shell.
- Watch trace shape: if you open a parent/root span and close it immediately, root-scoped scoring can fire against an empty trace. Keep the run's parent open until the run truly ends, or scope scoring to the child spans that carry content.

## Event & property taxonomy

Keep names stable, namespaced, and unit-aware. A workable neutral schema:

- **Lifecycle**: `run_started`, `run_ended` (props: `end_reason` = explicit | new_run | timeout).
- **Per pillar**, one event or property each so gates are queryable:
  - Actions: `artifact_exported`, `artifact_shared`, `result_accepted` …
  - Sentiment: `feedback_submitted` (prop `sentiment` = positive | negative), `sentiment_inferred`.
  - Evals: `eval_scored` (props: `eval_name`, `passed`, `score`).
- **The metric itself**: a dedicated, single-purpose property for the score (e.g. `success_score`) plus the individual gate values and a `gates_present` count. Do not overload a high-cardinality shared property — a dedicated field charts reliably and indexes immediately.
- **Identity & segmentation**: stable `run_id` / `session_id`, `user_id`, agent version/build, and any context (use case, plan tier) you'll want to slice by.

## Firing points (map each gate to a moment)
- **Actions** fire from the UI/handler when the user takes the action — instrument the export/share/accept paths.
- **Sentiment (explicit)** fires from the feedback control; **(inferred)** is computed at run end over the user's messages.
- **Evals** fire when the eval runs — inline at output time, or async after the run; emit `eval_scored` either way and reconcile at run end.
- **Score** is computed and emitted **once, at run end**, reading the gates accumulated during the run.

## Gotchas
- **Flush at boundaries.** Batched SDKs can drop events if the process recycles before the run-end flush. Flush explicitly when a run ends.
- **Short IDs / validation.** Some tools silently reject events with IDs shorter than a default minimum — set the minimum low if you use short user/run IDs.
- **Idempotent end.** A run can "end" via several paths; de-duplicate so you emit one `run_ended` (and one score) per run.
- **Backfill carefully.** Recomputing historical scores is fine, but mark backfilled rows and keep a schema version so old and new cohorts stay distinguishable.

## Tool-specific notes (examples, optional)

These illustrate the neutral patterns above; nothing here is required.

- **Product analytics (e.g. Amplitude):** model `run_ended` as a first-class event and put `success_score` on a dedicated numeric event property. Use an agent/session SDK if available so lifecycle and enrichment properties attach automatically; otherwise raw `track()` works if you include the required identity props. Be aware session-level classification typically keys off the explicit session-end event.
- **Eval/trace observability (e.g. Braintrust):** online scoring runs on span completion (sampled); it isn't a session-end concept. Topics/facet classification reads whole traces in a background pass above a volume threshold. If your trace nests turns under a run-level span, keep that span open until the run ends, or scope scoring to the turn spans that carry content.
