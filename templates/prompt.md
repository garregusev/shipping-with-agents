# NNN — <short task name>

_Branch: `[[prefix]]/<desc>` · Executor: <model> · Status: dispatched | accepted | rejected_

> One decision = one prompt. A fresh executor with no chat history must be able to run this file
> top-to-bottom. If it carries more than one concern, the acceptance checklist below is mandatory.

## Why

<!-- One paragraph. What problem this solves and what happens if we don't. The executor makes better
     judgment calls when it knows the intent, not just the instruction. -->

## Task

<!-- Concrete steps with `file:line` targets wherever possible. Name the functions, the tables, the
     screens. Vague scope is where drift comes from. -->

1.
2.

## Risks → tests

<!-- Enumerate the concrete failure modes FIRST, then the test that covers each one — including
     regression guards for the neighbours this change could break ("the other path still works",
     "a healthy retry must NOT abort"). -->

| Risk | Test that covers it |
|---|---|
| | |

## Out of scope

<!-- Explicit. This is what stops an eager executor from "helpfully" refactoring three adjacent modules.
     List the things it might reasonably think are included and are not. -->

-

## Gated steps

<!-- Anything production / irreversible / cost-incurring. State it here and require explicit
     confirmation before running. Never bundle these with the safe parts. -->

- [ ] <none> | <step requiring confirmation>

## Commit & push

- Commit message shape: `<type>: <what changed>`
- Push to `[[prefix]]/<desc>`, then STOP. Do not open the PR, do not merge, do not deploy.

---

## Outcome

> **Written by the reviewing design session, not the executor.** This is acceptance, not self-report.
> Even if the executor pre-fills it, the reviewer re-validates before signing off.

- **What landed vs the prompt:**
- **Deviations / surprises:**
- **Tests:** <names + pass counts, or the failure>
- **Commits:** <links>
- **Prompt quality retro (1–5):** <score> — <what made it good, or what made the executor drift>
