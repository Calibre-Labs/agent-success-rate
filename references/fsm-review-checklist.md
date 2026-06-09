# FSM review checklist

Goal: a confirmed, shared picture of the agent before any metric design. Works two ways — extract from code, or interview the PM when there's no code (or no access).

## What you must come away with

1. **States** — the named stages the agent moves through (e.g. NEW → PLANNING → ACTING → REFINING → COMPLETE).
2. **Transitions** — what events move the agent between states; which are user-driven vs. agent-driven.
3. **Terminal / success state(s)** — where a run legitimately *ends*, and whether "ended" means "succeeded."
4. **Tools / actions** — what the agent can call or produce; which produce a user-actionable artifact.
5. **Unit of analysis** — is success measured per **turn**, per **session**, or per **task**? Pick one and define its boundaries.
6. **Run boundaries** — when does a run start, and crucially **how does it end** (explicit "done", new run opened, idle timeout, tab close)? The end signal is the keystone for instrumentation.

## Mode A — extract from code

Search for, then confirm with the PM:
- State machine: an enum/union of states, a transition map or `nextState`-style function, guards/validators.
- Terminal state: a `COMPLETE`/`DONE`/`FINISHED` value and where it's set; idle-sweep or timeout logic.
- Session vs. turn: distinct IDs (session ID vs. message/turn ID); where each is minted.
- Tools: the registered tool/function list and which emit artifacts (files, exports, structured outputs).
- Existing instrumentation: analytics/tracing calls, especially any "session end" / lifecycle event (note where it fires and whether it's reliable — e.g. only on a poll, only on explicit completion).

Produce a compact FSM summary (states, transitions, terminal, unit) and a one-line note on how runs end today. Confirm before moving on.

## Mode B — interview the PM (no code)

Ask, one at a time:
- "Walk me through what your agent does from the user's first message to the point you'd call it 'done'."
- "What are the distinct stages it goes through? Does it ever loop back?"
- "When is a run *over*? How would your system know — does the user click something, or does it just go idle?"
- "What can the agent actually produce or do — anything the user downloads, sends, or acts on?"
- "Are you measuring one message, one back-and-forth session, or a whole task that might span sessions?"
- "Where does it most often go wrong today?" (seeds the Evals / Issues thinking)

## Red flags to surface early
- **No clear end signal.** If runs never explicitly end, success can't be scored per run reliably — note this; it usually becomes the first instrumentation ask.
- **Unit ambiguity.** Mixing turn-level and session-level signals into one rate produces a number nobody can interpret. Force the choice.
- **Terminal ≠ success.** Reaching COMPLETE because the user gave up is not success — make sure the gates can tell those apart.
