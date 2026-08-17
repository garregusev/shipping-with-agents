# Design & review rubric

Distilled from a stabilization audit of a stateful data pipeline and the process failures that let those
defects reach production. The audit's root theme: **ambiguous success/failure semantics + desynchronized
coupled state.** Apply this rubric to any non-trivial control-plane, pipeline, or stateful change — and
to every review of one.

Each rule below is followed by the shape of the real defect that motivated it.

---

## A. Architecture / design invariants

Design *against* these failure generators:

1. **Typed results, not sentinels.** A function whose caller must tell success from failure returns an
   explicit typed result (enum/dataclass), never an ambiguous `(list, cursor, bool)` / `None` / empty
   where "no data" is indistinguishable from "blocked / malformed / schema-changed / rate-limited". A
   terminal verdict (`END_OF_DATA`, `complete`) must be **earned** — every success precondition
   explicitly checked — not the default fall-through.
   *Defect: a malformed HTTP 200 fell through to `(items=[], cursor=None, rate_limited=False)` = "honest
   end of data", silently closing the collection window.*

2. **Commit the claim in the same transaction as the effect.** A watermark / `complete` / progress
   marker must be written atomically with the data it asserts landed. Never "mark done, then hope a
   later or separate step persists it."
   *Defect: a window was watermarked complete before the separate load step ran; `complete` ≠
   data-in-table.*

3. **Couple state that must change together into one atomic swap.** Where several objects together form
   one logical identity (a network route + a client profile + a session + its tokens), rotate them as a
   single object, or they desync and you send fresh tokens over a stale route.
   *Defect: three rotation sites each updated one field of the identity and left the shared sessions
   stale.*

4. **Never destroy the only copy before the durable copy is confirmed.** Raw files/artifacts are data
   too. Use immutable paths + a manifest recording commit status; delete only after a confirmed commit
   and a retention period.
   *Defect: the loader deleted raw files even when the DB insert rolled back (`inserted=0`); repeating
   filenames also let a retry overwrite an un-loaded attempt.*

5. **One implementation of a process.** Duplicated paths (legacy / new / special-case) mean a fix in one
   silently misses the others. Prefer a single shared implementation; when logic must be duplicated,
   treat every fix as "find and patch all N copies."
   *Defect: a fix landed in the new path but not the legacy one; a separate bug lived in all three
   rotation sites and the first review pass fixed only two.*

6. **One durable source of truth for state.** Splitting state across an in-memory queue + progress files
   + a watermark table + raw JSON + the DB guarantees divergence.
   *Defect: a monthly progress file became a second, coarser watermark that froze unfinished
   sliding-window work.*

7. **Comments explain WHY, not WHAT/behavior; behavioral claims go in tests.** A comment asserting
   runtime behavior ("the session is recreated fresh next call") drifts from the code and becomes a
   lie — worse than no comment, because readers trust it. Prefer comments that capture intent /
   rationale / tradeoff (durable). Put any behavioral or invariant claim in a **test** — an executable
   comment that *fails* when the behavior changes — not prose. When a comment must encode a non-obvious
   **load-bearing assumption** you could not verify, tag it `# ASSUMPTION(verify): …` so the reviewer
   targets and **clears** it.
   *Defect: a bug hid behind a comment claiming a session was "recreated fresh on the next call" — true
   for one path, false for the other; the false claim misled the reviewer.*

---

## B. Review-after-change checklist

Run **every time**, especially on bug fixes, before accepting a change:

1. **Enumerate all instances of the pattern.** A fix to a repeated pattern (a rotation site, a
   mark-complete branch, a delete-after-write, a per-path duplicate) is incomplete until you `grep` the
   pattern, count the sites, and verify the fix hits **every** one.
   *Worked: a review grepped the assignment, found 3 rotation sites, caught that the executor fixed only
   2, and sent it back for the third.*

2. **Check whether existing tests encode the OLD (buggy) behavior.** A green suite can be green because
   a test asserted the bug. When fixing, search for tests asserting the pre-fix behavior and correct
   them.
   *Found: two rate-limit tests asserted `rate_limited=False` for garbage 200s — literally the bug.*

3. **Trace data flow; don't trust comments/docstrings.** Verify against the code, not its narration.

4. **Add an invariant test for cross-cutting state.** Mocked tests pass on logic while missing real
   coupling. After a state change (rotation, refresh), assert **every** dependent object reflects it.

5. **Run a systematic defect-hunt, not only hypothesis-driven reads.** Reading code to answer one
   question misses defects outside that path (and invites confirmation bias — a fresh single-session
   probe can pass *because* it bypasses the bug). Run an automated code review on every non-trivial PR
   before acceptance; run a deep one on the module periodically.

6. **Distinguish a tactical patch from the architectural fix.** When you patch a defect that a deeper
   redesign will replace, say so explicitly — don't let the band-aid masquerade as the cure.

7. **Audits of anything expensive include a cost-loop pass.** Auditing for data loss alone is not a
   complete audit. For every expensive operation verify (a) it is guarded by an "is there pending work?"
   check, and (b) its repetition is bounded when there is no work. Then validate code-derived
   conclusions against live log frequencies — an order-of-magnitude mismatch is a bug even when zero
   data is lost.

---

## C. When two agents/models touch the same area

Parallel work on the same control-plane files = conflicts + duplicated effort. Before starting: confirm
who owns which files, rebase onto the merged baseline, and drop already-landed items from scope.
Serialize edits to a shared file.
