# Catudoku — cat-themed Sudoku for Android

Flutter, live on Google Play as
[Catudoku: Cat Sudoku Puzzle](https://play.google.com/store/apps/details?id=com.catudoku.app).

---

## Decision 1 — the eCPM "bug" that wasn't

**The symptom.** eCPM fell by roughly 70% month over month and fill rate collapsed from ~99% to the
30–45% range. It looked exactly like a broken ad configuration, and the obvious move was to go hunting
through the mediation setup for what broke.

**The finding.** Nothing broke. It was a **geo-mix shift**. ASO work had grown installs — but the growth
came disproportionately from low-eCPM countries, which diluted the average while the high-value US
segment's own eCPM stayed healthy and kept producing the majority of revenue. **Total revenue kept
growing the entire time the headline metric was falling.**

**Why it matters.** eCPM is a ratio, and a ratio moves when either side moves. Chasing it as a bug would
have meant "fixing" a mix that was the *result of the growth we wanted*. The rule that came out of this:
track **total revenue and fill rate**; treat eCPM as diagnostic, never as a target.

**But two real levers did surface** while investigating — an entire mediation group had been silently
disabled around the time of the drop (which independently explained one network serving zero
impressions), and two more networks were configured but serving nothing. Both were genuine, actionable
monetization work that the "it's all fine, it's just mix" conclusion would have buried if the
investigation had stopped at the diagnosis.

**The transferable part:** *"the metric moved for a boring reason"* and *"there is real work here"* are
not mutually exclusive, and stopping at the first one is the more comfortable mistake.

---

## Decision 2 — free hints in the first game

New players lose their first Sudoku and leave. Hints are the obvious remedy and also the obvious thing
to monetize, which is the trap: charging for the mechanic that prevents first-session churn optimises
the wrong end of the funnel. Hints are free in the first game, priced after — the user has to have a
good first session before there is anything worth monetizing.

## Decision 3 — the rating prompt fires on the first win

Ratings are asked for at the single highest point of the session — the first completed puzzle — rather
than on a timer or on launch count. Store rating is a growth input, not a vanity metric: it feeds
conversion on the listing that ASO work is driving traffic to. Asking at a neutral moment wastes the
one prompt you get.

## Decision 4 — release pipeline before release features

CI handling lint, R8 and signing was built before the store launch rather than after the first painful
manual release. For a solo operator directing agents, the release path is the step with the least
tolerance for improvisation and the one where a mistake is most visible.

---

## What the process caught

The eCPM investigation is the clearest example in either project of **"logs first, theories second"**
paying off. The plausible theory — broken ad config — was actionable, specific, and wrong. The
per-country breakdown was five minutes of looking. Every hour that would have gone into "fixing" the
mediation setup would have been spent making a healthy system worse.

It is also the clearest example of why decision records are worth the ten minutes: the conclusion
*"don't chase eCPM as a bug"* is exactly the kind of finding that gets re-litigated three months later
by whoever sees the graph next — including me.

---

*Live on Google Play. Source is private.*
