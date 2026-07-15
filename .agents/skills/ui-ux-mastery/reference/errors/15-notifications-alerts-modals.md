---
name: notifications-alerts-modals
description: Toasts, banners, dialogs, modal-vs-sheet-vs-inline choice, and notification-fatigue errors and fixes for web UI.
---

# Notifications, Alerts & Modals

## Anti-pattern: Modal used for everything, regardless of severity
**What's wrong:** Every confirmation, tip, and minor status update opens a blocking modal dialog.
**Why it matters:** Modals interrupt the task flow and demand a decision before anything else can happen — using them for low-stakes situations trains users to reflexively dismiss dialogs without reading, which then makes genuinely important modals (data loss, payment) less effective too.
**Fix:** Match presentation to actual severity/blocking-ness:

| Situation | Right pattern |
| --- | --- |
| Minor status update, no action needed | Toast (auto-dismissing) |
| Persistent but non-blocking condition (e.g. "You're offline") | Banner (dismissible or persistent until resolved) |
| Needs a decision but isn't destructive | Inline confirmation or a lightweight popover |
| Destructive/irreversible action | Modal confirmation |
| Requires full attention before proceeding (payment failed, session expired mid-task) | Modal or full-page interstitial |

## Anti-pattern: Toast notifications used for information the user needs to act on later
**What's wrong:** An important message (e.g. "Your export is ready to download") shown only as a toast that disappears after a few seconds, with no persistent record.
**Why it matters:** If the user isn't looking at the screen at that exact moment, or needs to act minutes later, the information is simply gone.
**Fix:** Reserve toasts for transient acknowledgments of actions the user just took; anything the user might need to act on later belongs in a persistent notification center, inbox, or a status area that stays visible/re-visitable.

## Anti-pattern: Too many simultaneous or too-frequent notifications
**What's wrong:** Multiple toasts stacking on screen at once, or a stream of low-value notifications (every minor background sync, every collaborator's cursor move) interrupting the user repeatedly.
**Why it matters:** Notification fatigue causes users to tune out or dismiss all notifications indiscriminately, including ones that actually matter.
**Fix:** Batch/aggregate low-value events ("3 changes synced" instead of 3 separate toasts), rate-limit non-critical notifications, and let users control notification granularity in settings for anything beyond core transactional alerts.

## Anti-pattern: Modal that can't be dismissed by clicking outside, pressing Escape, or a visible close control
**What's wrong:** A modal with no `X` button, no outside-click-to-close, no Escape handling — the only way out is completing the form inside it.
**Why it matters:** Removes user control and freedom (Nielsen heuristic #3); if the user opened it by mistake or changed their mind, they're stuck.
**Fix:** Every modal (except ones deliberately gating a mandatory, unavoidable action like a required terms-of-service acceptance) should support closing via a visible `X`, Escape key, and clicking the backdrop — and even mandatory modals should be exceedingly rare and clearly justified.

## Anti-pattern: Clicking outside a modal with unsaved changes silently discards them
**What's wrong:** A user editing a form inside a modal accidentally clicks the backdrop and the modal closes, losing everything typed, with no warning.
**Why it matters:** An accidental dismiss shouldn't be able to destroy work with zero recourse.
**Fix:** If a modal contains unsaved input, intercept the close attempt (backdrop click, Escape, `X`) with a lightweight "Discard unsaved changes?" confirmation, or better, autosave/persist the draft so no dismissal path can lose it at all.

## Anti-pattern: Nested modals (a modal opening another modal on top)
**What's wrong:** Clicking an action inside a modal opens a second modal stacked above it.
**Why it matters:** Stacked modals compound focus-management complexity, are disorienting (which "layer" am I in, and what happens if I close this one?), and are a common source of genuine accessibility bugs (focus trap conflicts between the two).
**Fix:** Avoid nested modals; if a sub-flow is genuinely needed from within a modal, replace the modal's content in place (a "step 2" view within the same dialog) rather than stacking a second dialog on top, or close the first modal before opening the second.

## Anti-pattern: Alert/confirm/prompt (native browser dialogs) used for a polished product
**What's wrong:** Using `window.confirm()`/`window.alert()`/`window.prompt()` for in-app confirmations.
**Why it matters:** Native browser dialogs are unstyleable, block the entire page (including any custom animation), look jarringly inconsistent with the rest of the product, and provide a poor experience on modern browsers that increasingly restrict or badge them.
**Fix:** Build a proper custom, in-brand dialog component for all confirmations; never rely on native `alert`/`confirm`/`prompt` in production UI.

## Anti-pattern: Banner/alert that can never be dismissed, permanently consuming screen space
**What's wrong:** A persistent top-of-page banner (announcement, promo, warning) with no close control, remaining forever even after the user has read/acknowledged it.
**Why it matters:** Permanently reduces usable screen space and becomes background noise the user has learned to ignore, defeating its own purpose.
**Fix:** Give every non-critical banner a dismiss control that persists the dismissal (don't show it again after being closed, at least for that specific message); reserve permanently non-dismissible banners strictly for currently-ongoing critical states (e.g. "You are offline" — legitimately can't be dismissed because the condition is still true).

## Anti-pattern: Snackbar/toast position or timing inconsistent across the app
**What's wrong:** Toasts appear top-right in one part of the app and bottom-center in another; durations vary unpredictably.
**Why it matters:** Inconsistency (Nielsen heuristic #4) makes each new context feel like relearning where to look.
**Fix:** Standardize toast position, animation, duration rules, and stacking behavior into a single shared component used everywhere.

## Anti-pattern: Critical errors communicated only via a toast that can be missed
**What's wrong:** A payment failure, data-loss risk, or security-relevant event surfaced only as a brief auto-dismissing toast.
**Why it matters:** If the severity of the underlying event outpaces the attention-grabbing power of the notification style used, the user may never register it happened.
**Fix:** Match the notification's persistence/intrusiveness to the real cost of the user missing it — a payment failure warrants a modal or a persistent banner plus (if applicable) an email, not just a toast that vanishes in 5 seconds.

## Quick Checklist
- [ ] Notification pattern (toast/banner/inline/modal) matches actual severity and blocking-ness
- [ ] Toasts reserved for transient acknowledgments; anything actionable later lives in a persistent surface
- [ ] Low-value notifications are batched/rate-limited, not fired individually and constantly
- [ ] Every dismissible modal supports close via visible control, Escape, and backdrop click
- [ ] Unsaved changes inside a modal are protected from silent loss on accidental dismissal
- [ ] No nested/stacked modals — sub-flows replace content in place instead
- [ ] No native `alert()`/`confirm()`/`prompt()` used in production UI
- [ ] Dismissible banners persist their dismissed state; only truly ongoing conditions stay non-dismissible
- [ ] Toast position, duration, and behavior are standardized app-wide
- [ ] Critical/high-cost events use a notification style persistent/intrusive enough to guarantee visibility
