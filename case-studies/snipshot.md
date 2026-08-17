# SnipShot — area & scrolling screenshots for Android

Native Kotlin, minSdk 30. Two decisions are the reason the app exists in the shape it does.

---

## Decision 1 — AccessibilityService capture instead of MediaProjection

**The constraint.** MediaProjection is *the* screen-capture API on Android, and it is the obvious choice.
It also requires a consent dialog for **every** projection session on Android 14+, plus a persistent
foreground-service notification. On some vendor skins that notification suppressed its own text and the
idle service got killed outright. And Samsung-style long screenshots need *injected scrolling*, which
MediaProjection cannot do at all.

**The decision.** `AccessibilityService.takeScreenshot()` for capture and `dispatchGesture()` for the
auto-scroll. A one-time enable in accessibility settings replaces the per-capture dialog entirely.

**What it cost, accepted knowingly.** A Play accessibility declaration and a disclosure video at review.
To keep that declaration defensible the service has to stay events-blind — no `eventTypes`, no window
content — which is a permanent design constraint, not a launch-time checkbox. Sideloaded installs also
hit Android 13+ "Restricted settings" friction (Play installs don't).

**Why it's the right trade.** The competing option was cheaper to build and worse to use, every single
time the user took a screenshot. Recurring friction beats one-time friction only when the one-time cost
is genuinely one-time — here it is.

---

## Decision 2 — Rewarded quota, fail-open

**The model.** Five free screenshots, then a rewarded ad grants five more; users can stockpile in
advance rather than being interrupted at the worst moment.

**Two safety rails, both deliberate:**

1. **Capture is never gated — only save and share.** The frozen frame waits through the ad, so the
   user's moment is never lost. Gating capture itself would trade reviews for the same revenue: a
   missed screenshot is a one-star review, and one-star reviews cost more than the impression earns.
2. **Fail-open.** The gate appears only when a rewarded ad is *actually loaded*. No fill, no key,
   offline → nothing is blocked. A paywall that can strand a user with no way forward is a one-star
   factory.

**Deferred on purpose.** Per-country grant sizing via mediation segments is post-launch — data first.
Guessing the grant size per geo before there is any geo data is exactly the "magic number" the
engineering rules forbid.

---

## What the process caught

The interesting failures on this project were product failures, not code failures, and they were caught
at the decision-record stage rather than in review — writing down *why X and not Y* forced the
accessibility trade-off to be priced (declaration, disclosure video, events-blind constraint) **before**
the implementation existed, instead of discovering it at Play review.

That is the argument for decision records in general: they are cheap when written, and the alternative
is finding out during store review that the shortcut you took is the thing being reviewed.

---

*Source is private.*
