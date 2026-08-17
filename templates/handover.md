# Handover: <branch-name>

_Started: <date>_

> State, not narrative. A session-start hook injects this file whole, so keep it lean — move finished
> narrative to `archive/` instead of letting it accrete. Append to it **immediately** after each finding
> or dead end; anything not written down before a context compaction is gone.

## Context
<!-- 2-4 sentences: what this branch is for, where it sits relative to other branches, what state it's
     in. The one paragraph you'd want if you opened this cold. -->

## Findings
<!-- Verified facts discovered this session. One bullet each. Prefer numbers over adjectives. Mark how
     it was verified (HTTP capture / set-diff / SQL / code-read / live run) so a future reader knows the
     confidence. Example:

     - Endpoint B is NOT a subset of endpoint A: set-diff on one account — 190 items (19.6%) appear
       only in B. The sort parameter changes WHAT enters the result window, not just the order. -->

## Dead ends
<!-- Approaches tried and rejected, with the reason. This is the highest-value section — it stops the
     next session re-walking the same path. Example:

     - Seeding discovery from the third-party dataset — rejected, it re-creates the dependency we are
       trying to remove. -->

## Decisions
<!-- Choices made and the rationale, especially "we picked X over Y because Z". Link the decision record
     if one was written. Example:

     - Windowed collection with a watermark table (decision 030). Granularity (source, entity, period).
       Distinguishes "empty" from "not collected". -->

## Open questions
<!-- Unresolved, with enough detail to resume. Flag blockers explicitly. Example:

     - P0: the fetch path treats an error object inside an HTTP 200 as end-of-data. Until fixed, every
       "zero results confirmed" verdict is unreliable — DO NOT run the backfill. -->

## In-flight work
<!-- What's running right now / last prompt dispatched / status unknown. So a resumed session knows what
     to check first. Example:

     - P0 fix prompt sent to the executor (last action before the session ended) — status unknown. -->

## Key files touched
<!-- path:line — one-line note on what's there and why it matters. Example:

     - src/collector/client.py:102 — MAX_PAGES=200 vs MAX_ALT_PAGES=100 (undocumented asymmetry) -->
