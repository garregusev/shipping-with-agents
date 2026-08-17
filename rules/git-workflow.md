# Git workflow

## The default for any code change

Feature branch → push → PR → merge. Executor sessions push their own `[[branch-prefix]]/<description>`
branch and **must NOT push feature code directly to the deployed branch**. Direct pushes are for trivial
docs/iteration commits **explicitly agreed as such** — never feature, pipeline, or API code.

The `gh` CLI is available and authenticated: create the PR with `gh pr create` after pushing.

## Merge-and-roll-out flow

When a PR needs nothing from the user on GitHub beyond the merge button, ask ONE confirmation in chat;
an affirmative reply authorizes `gh pr merge` AND the follow-up rollout of that change in the same flow.
Prod-touching rollout steps are still named with their concrete risks before running, but need no
separate re-confirmation. Delete the branch on merge.

If the PR needs anything else from the user on GitHub (design disagreement, branch-protection change,
competing PRs), fall back to user-merges-via-UI.

## CI is the gate — look at it BEFORE merging, as a separate step

Wait for CI to be green before merging, and **never chain `gh pr merge` after the checks command in one
shell line.** *Why: that exact chaining once merged a PR with a red check that had caught a real
migration-number collision.* Look first, then merge.

Verify and say so before merging: the PR is mergeable, no unresolved change requests, every required
check passed.

If CI is ever paused, the substitute is mandatory and explicit: run the affected suites locally and
state in the PR which ran and their counts — "I could not run them" is not a pass, it is a reason to
stop and say so.

## Pre-push hook

Activate once per session (idempotent):

```bash
git config core.hooksPath .githooks
```

`.githooks/pre-push` runs the suites affected by the push. Scoping is fail-safe: docs-only pushes run
nothing; anything unrecognised runs every suite. Steps that can't find their dependencies print a loud
WARNING and skip — so the hook is a first line of defence, not evidence on its own.

**⚠️ NEVER use `git push --no-verify`.** If the conversation ever heads toward pushing without tests —
stop and warn the user explicitly before proceeding.

## Deploy verification

After deploying, verify **by content on the target** — `git log --oneline -1` plus a `grep` for a line
from the new code — never by the output of a chained local-pull-then-remote command. *Why: three PRs
"deployed" successfully in the transcript while the remote merge silently failed on an untracked file;
the reassuring output came from the LOCAL pull.* If a file was ever copied to the server by hand, remove
it before merging.

## Commit messages / PR bodies

Merging without a human reviewer means the reviewer of record is the executor. Write the PR body so a
person can reconstruct the decision later: what changed, what was verified and how, what was
deliberately left out. An acceptance checklist is the cheapest form of this.

## Related

- [Engineering standards](engineering-standards.md) — risks→tests→code, review checklist
- Work is lost through direct push + branch delete; before debugging a "new" bug, check `git fsck` for
  whether it was already fixed once.
