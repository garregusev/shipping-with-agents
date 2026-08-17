# Engineering standards

The rules that make changes safe to make twice. Read before any non-trivial feature, fix, or
investigation; the one-line triggers live in `AGENTS.md`.

## Logs first, theories second

When debugging ANY failure where logs exist (server logs, request logs, wrapper logs, preserved test
tmpdirs) — START by reading them; establish what actually happened before proposing causes or fixes.

*Lesson: two connector fixes were built on plausible auth theories while the server logs showed the
requests NEVER REACHED the server. One log check would have replaced both iterations.*

## New features and non-trivial changes: risks → tests → code

Before implementing: (1) enumerate concrete risks and failure modes, (2) write failing tests that cover
those risks, (3) only then write the implementation. Skip only for truly trivial changes (typos, config
values, one-liners). When in doubt, write the tests first.

## Design rules for control-plane / stateful changes

Full checklist: [`../review-rubric.md`](../review-rubric.md).

1. **Typed results, not sentinels** — a terminal verdict like `complete`/`END_OF_DATA` is *earned* by
   checking every success precondition, never a default fall-through where "no data" looks identical to
   "blocked / malformed / schema-changed".
2. **Commit the claim with the effect** — a watermark/`complete` is written in the SAME transaction as
   the data it asserts landed.
3. **Couple state that must change together into ONE atomic swap.**
4. **Never delete the only copy before the durable copy is confirmed.**
5. **One implementation per process** — duplicated paths mean a fix in one silently misses the others.

## Review every non-trivial change with the fixed checklist

1. **Enumerate ALL instances of any pattern you touched** — grep it, count the sites, verify the fix
   hits every one.
2. **Check whether existing tests encode the OLD buggy behavior** — a green suite can be green because a
   test asserted the bug.
3. **Trace data flow, don't trust comments.**
4. **Add an invariant test for cross-cutting state.**
5. **Run an automated code review on every non-trivial PR** before accepting it.

## No magic numbers

Every behavioural constant (timeout, threshold, cap, window, retry count, cooldown, backoff) is either
derived from a measurement or an *explicit guess* — env-configurable, with a TODO to calibrate **and a
log line emitting the actual value at which it mattered**, so it can be calibrated later. Never present
an air-picked number as justified; don't re-tune by guessing — instrument the real value.

## Sunk cost is not a reason to keep an approach

Decide continue-vs-switch on the approach's merits *now*. Keeping it is justified only by evidence it
works, or a cheap **time-boxed test with an explicit exit criterion** ("if X by day N, switch") — never
by "we already paid".

## Config files are ASCII-only

Comments and values in `.env`, service units, shell scripts and reverse-proxy configs stay in English
ASCII. Non-ASCII comments in config files have caused services to fail to start with encoding errors
that look like nothing else.

## Where records live

- **Non-trivial decisions** → `docs/decisions/` — trigger: *"would a future developer ask why this was
  done this way?"* Template: [`../templates/decision.md`](../templates/decision.md).
- **Open engineering items** → `docs/issues/backlog.md` — every new idea/hypothesis/deferred fix gets a
  row IN THE SAME TURN it appears; blocked rows name a wake-condition; finished rows move to Done with a
  date.
- **Executor prompts** → `docs/prompts/NNN-name.md`, one decision-level change per prompt; the reviewing
  session writes the `## Outcome` block at acceptance (never the executor).
- **Session continuity** → `docs/handover/main.md` — STATE, not narrative; update at milestones.
  Narrative older than the current phase goes to `docs/handover/archive/`. Template:
  [`../templates/handover.md`](../templates/handover.md).
- **Recurring failures** → `docs/troubleshooting.md`, one entry per problem in a fixed shape:
  **Symptom / Root cause / Fix / Prevention**. Add the entry the turn you solve the problem, not later.
