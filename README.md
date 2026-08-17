# Shipping with agents

How I run software projects where I write the specs and the review gates, and AI agents write the code.

I'm a product manager. I don't hand-write the implementation — I decide what gets built, why, and what
"done" means, then direct coding agents through it and accept or reject what comes back. This repository
is the operating system I ended up with after doing that across two shipped Android apps. It is
process, not application code.

If you're here from my LinkedIn: this is the honest version of "how I build things." The code in my
apps was written by agents. The specs, the decision records, the review rubric, and the call on what
ships — those are mine, and they're what this repo contains.

---

## The one-paragraph model

A **design session** (expensive model, my attention) enumerates risks, picks an approach, and writes a
self-contained prompt. An **executor session** (cheaper model) writes code and tests, runs them, commits,
pushes — and stops. The design session then **reviews the result and writes the acceptance record**.
Design is cheap to redo and expensive to get wrong; execution is the opposite. Keeping them separate is
the whole trick.

```
spec ──► prompt ──► executor ──► review gate ──► acceptance record ──► release
 │                                    │                  │
 └── one decision per prompt          └── I reject here  └── lessons go back into the rules
```

## What's here

| File | What it is |
|---|---|
| [`playbook.md`](playbook.md) | The working agreement: prompts, acceptance, handover, production protocol. Portable — `[[blanks]]` are per-project. |
| [`review-rubric.md`](review-rubric.md) | What I check before accepting a non-trivial change. Each rule maps to a defect that actually reached production. |
| [`rules/`](rules/) | Always-loaded project rules: engineering standards, git workflow, irreversible operations. |
| [`templates/`](templates/) | Handover doc, decision record (ADR), and feature spec formats. |
| [`case-studies/`](case-studies/) | Two shipped apps, the product decisions behind them, and what the process caught. |

## Why any of this exists

Every rule here was bought with a failure. A prompt that bundled four concerns silently dropped the
third. A self-rated "done" hid a fix that never landed. A rotation bug lived in three call sites and the
first review pass fixed two. A volume migration that optimised for downtime wiped a database and cost
two hours of recovery.

Agents are fast and confident, which means the expensive failure mode isn't bad code — it's plausible
code that nobody checked against the thing it was supposed to do. The rules that survive here are the
ones that turn "someone should check that" into a step that can't be skipped.

## The parts that generalise

Three ideas do most of the work, and none of them are about prompting:

1. **Separate who proposes from who accepts.** The executor never writes its own `## Outcome` block. A
   self-rating calls a dropped fix "done"; a reviewer catches it — and catches the reviewer's own earlier
   errors too.
2. **Write down the dead ends.** Context gets compacted, sessions end, and the single highest-value
   section of any handover doc is the list of approaches that failed and why. It stops the next session
   re-walking a path you already paid for.
3. **Make the rules mechanical.** A rule that depends on remembering it rots after the first context
   compaction. Session-start hooks that load state, stop-hooks that block on an un-updated handover, and
   a pre-push hook that runs the tests are worth more than any amount of good intention.

## Using it

Start with [`playbook.md`](playbook.md) — it's written to be copied into a new project and filled in.
The setup checklist at the bottom takes about an hour and is the part I'd actually recommend doing.

---

*Process distilled from work on [Catudoku](https://play.google.com/store/apps/details?id=com.catudoku.app)
and SnipShot. Application code and business specifics are not included here by design.*
