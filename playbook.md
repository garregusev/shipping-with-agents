# Agentic Workflow Playbook

> **Portable.** Copy this into any project to get the working agreement without re-deriving it. It is
> project-agnostic; anything in `[[double brackets]]` is a per-project blank to fill in. Pair it with a
> project `AGENTS.md` that holds the project-specific facts; this file holds the *process*.
>
> Origin: distilled from a multi-day pipeline-hardening effort (9 design→execute→accept cycles) and its
> retro. The rules that earned their place are marked with the failure that motivated them.

---

## 0. The one-paragraph model

A **design session** (expensive/rate-limited model) enumerates risks, picks an approach, and writes a
self-contained prompt. An **executor session** (cheaper model) writes code + tests, runs them, commits,
pushes — and stops. The design session then **reviews the result and writes the acceptance record**.
Design is cheap to redo and expensive to get wrong; execution is the opposite. Keep them separate.

---

## 1. Prompts (the unit of work)

- **One decision = one prompt.** A prompt carries exactly one coherent change. *Why: bundling four
  concerns into one prompt once silently dropped the third — nobody noticed until review.* If a prompt
  truly must carry more than one, give it an **explicit acceptance checklist** the reviewer ticks off
  item by item.
- **Numbered, self-contained files**: `[[docs/prompts]]/NNN-short-name.md`. A fresh executor with no
  chat history must be able to execute it. Include: *why* (one paragraph), *task* with `file:line`
  targets, *risks→tests*, *commit+push* instructions, and explicit **out of scope**.
- **Gate the dangerous parts inside the prompt.** Any production / irreversible / cost-incurring step
  says so and requires explicit confirmation before running — never bundled with the safe parts.
- **State the branch and the push target.** Don't make the executor guess.

## 2. Acceptance & the outcome loop

- **The `## Outcome` block is written by the reviewing design session, not the executor.** It is
  *acceptance*, not self-report. *Why: a self-rating calls a dropped fix "done"; a reviewer catches it
  — and catches the reviewer's own earlier errors too.* Even if an executor pre-fills it, the reviewer
  re-validates.
- **Outcome = what landed vs the prompt, deviations/surprises, tests, commit links, and a 1–5 retro on
  prompt quality.** Keep it ~half a page.
- **Two-tier conclusions.** A clear lesson (from comparing prompt ↔ result ↔ docs) goes straight into
  canon (`AGENTS.md` / a decision doc / module README). Everything else accrues in the Outcome block.
- **Index + periodic retro.** Keep a one-row-per-prompt index. Every N prompts, do a batch retro
  (`RETRO-NNN.md`): read the intent+outcome pairs, find what made prompts succeed/drift, port general
  lessons here. *Store intent+outcome only — never execution artifacts (diffs/logs live in commits),
  or the review stops being cheap.*

## 3. Handover discipline (survives context loss)

- Maintain a **per-workstream handover doc** (`[[docs/handover]]/<branch>.md`). After every non-trivial
  finding / dead end / decision / open question, append a bullet **immediately** — do not batch it.
  *Why: context gets compacted mid-session; anything not written down is gone.*
- Sections: Context / Findings / Dead ends / Decisions / Open questions / In-flight / Key files.
  **Dead ends are the highest-value section** — they stop the next session re-walking a failed path.
- Prefer numbers over adjectives; note how each fact was verified (test / set-diff / live run / billing).

Template: [`templates/handover.md`](templates/handover.md).

## 4. Decision records

When a non-obvious choice is made (architecture, a trade-off, a workaround for a subtle constraint),
write a short `[[docs/decisions]]/NNN-name.md`: the problem, **why X and not Y**, and the verification.
Trigger: *"would a future developer ask why this was done this way?"*

Template: [`templates/decision.md`](templates/decision.md).

## 5. risks → tests → code

For any non-trivial change: (1) enumerate concrete risks/failure modes, (2) write tests that cover them
— **including regression guards for the neighbours the change could break**, (3) then implement. *Why:
the named guards ("the other path still works", "a healthy retry must NOT abort") are exactly what kept
fixes from breaking adjacent behaviour.* Skip only for truly trivial changes.

## 6. Production protocol

- **Production actions require separate, explicit confirmation** — never bundled into another task's
  approval. Approval for one action does not extend to the next. *(Define what counts as production for
  this project — e.g. shared DBs, live containers, cron, deploys, store releases. [[project-specific]])*
- **Audit/observe first, change second.** On a risky change, run the read-only diagnosis (or an
  observation window) *before* mutating anything. *Why: this ordering twice caught a wrong premise
  before any change was made.*
- **Deploying code ≠ activating it.** Merging/pushing to the deployed branch is housekeeping; the
  actual prod effect (restart, pointing traffic at it, promoting a release track) is a separate,
  separately-confirmed step.
- **Prefer a structurally read-only path over a disciplined one.** Where the platform allows it, give the
  agent a genuinely read-only account (unprivileged shell user, SELECT-only DB role) and escalate
  deliberately, rather than relying on the agent to restrain itself. A rule you can't violate beats a
  rule you agree to follow.
- **Stop on the first access error, don't retry.** Servers with brute-force protection ban on repeated
  failures. Report the exact error and wait.

## 7. Irreversible operations (no exceptions)

Before the first command of anything hard/impossible to reverse (dropping data, migrating stores,
deleting files, schema changes on prod): **(1)** write the full plan incl. a rollback per step and show
it first; **(2)** scan the project rules for warnings relevant to this op and turn each into an explicit
step; **(3)** verify the backup is actually restorable (not just that it exists); **(4)** choose the
simple, proven path over the clever fast one; **(5)** state out loud "this cannot be undone, backup
verified, proceeding." The forcing function is the point.

Full protocol: [`rules/irreversible-operations.md`](rules/irreversible-operations.md).

## 8. Verify claims against ground truth

- **Cost / scale numbers: trust the external source of truth (billing, dashboard), never a
  self-reported counter, when a decision rides on them.** *Why: a self-reported traffic counter was
  ~30–45× off for weeks and skewed every cost decision.*
- **Security severity: prove the exploit path empirically before rating it.** Network topology, port
  bindings, group membership, or auth config may already mitigate. *Why: a "critical, injection could
  DROP" finding turned out unreachable because of how the port was published.*
- **Validation gates: gate on the most direct mechanism signal, not a noisy aggregate.** *Why: a
  ✓/✗-ratio gate needed a big sample to mean anything; "0 of the bad event vs 62 before" proved the fix
  in minutes.*
- **Empirical hypotheses: verify from real captured data, aggregated, not from 2–3 samples.**
- **No magic numbers.** Every constant that governs behaviour (timeout, threshold, cap, window size,
  retry count, cooldown, backoff) is either (a) derived from a measurement, or (b) an *admitted* guess:
  env-configurable, with a TODO to calibrate **and a log line emitting the actual value at which it
  mattered** so it can be calibrated from real data later. Never present an air-picked number as if it
  were justified, and don't re-tune it by guessing — instrument the real value and calibrate. *Why: a
  set of caps, windows and thresholds were all invented then re-tuned blindly, hiding that we never
  measured the thing the number was supposed to represent.*
- **Sunk cost is not a reason.** Decide continue-vs-switch on the approach's merits *now*, not on what's
  already spent (prepaid capacity, time, code written). Keeping an approach is justified only by evidence
  it works — or by a cheap, **time-boxed test with an explicit exit criterion** ("if X hasn't happened by
  day N, switch"). If the real reason you're keeping it is "we already paid / already built it," that's a
  red flag — name it and re-decide on the merits. *Why: an external service was kept past its usefulness
  because the capacity was prepaid; the honest framing is a dated exit criterion, not "it's already paid
  for."*

## 9. Enforcement hooks (make the rules mechanical)

Rules that depend on memory rot after a context compaction. Wire them as hooks so the harness enforces
them:
- **SessionStart** → load the current workstream's handover doc into context.
- **Stop** → block finishing if the handover wasn't updated this session; optionally auto-commit work so
  nothing is left only in the sandbox.
- **pre-push** (git hook, `core.hooksPath`) → run the test suites before every push; **never bypass with
  `--no-verify`.**

A standing preference that must survive compaction (e.g. response language, a default) belongs in the
always-loaded rules file, not just in chat. *Why: a chat-only preference silently reverted after a
summary.*

## 10. One rules file, many vendors

Keep the project's agent instructions in a single `AGENTS.md` and make `CLAUDE.md` / `GEMINI.md`
symlinks to it. Parallel hand-maintained copies drift, and the drift is invisible until an agent acts on
the stale one.

Keep that file **thin**: always-on rules inline, everything else a trigger → link table
("before you touch X, read Y"). A rules file long enough to be skimmed is a rules file that gets skimmed.

## 11. Git workflow

- Feature work on a `[[branch-prefix]]/<desc>` branch; quick contained fixes can go straight to the
  deployed branch. The deployed branch reflects deployed code.
- `git push -u origin <branch>`; on network failure, retry with backoff. Prefer fetching/pulling
  specific branches. Resolve handover-doc merge conflicts by keeping both sides.
- Don't push to the deployed branch, open PRs, or do irreversible git ops (`reset --hard`, force-push,
  branch delete) without explicit permission.

Full rules: [`rules/git-workflow.md`](rules/git-workflow.md).

## 12. Communication [[project-specific]]

- Reply to the user in **[[their language]]**; keep code, commits, and executor prompts in their natural
  language. Put this in the always-loaded rules file so it survives compaction.
- No blocking popups in remote/headless sessions — ask in plain text. Give a recommendation, not an
  exhaustive menu. Report outcomes faithfully (failing tests stay failing in the report).
- Plain language, no internal jargon. Lead with the result and the next step; metric codenames and
  invented shorthand belong in docs, not in replies.

---

## Setup checklist for a new project

1. Drop this file in; create `[[docs/prompts]]/`, `[[docs/decisions]]/`, `[[docs/handover]]/` with a
   one-line README each, plus a prompt `_TEMPLATE.md` whose `## Outcome` is marked reviewer-only.
2. Create `AGENTS.md`; symlink `CLAUDE.md` and `GEMINI.md` to it.
3. Add the three hooks (SessionStart load, Stop handover-guard, pre-push tests) and set
   `core.hooksPath`.
4. In the project rules file, fill the `[[blanks]]`: production definition, branch prefix, language,
   test commands.
5. Start the first prompt at `001`. Run the first retro after ~8–10 prompts.
