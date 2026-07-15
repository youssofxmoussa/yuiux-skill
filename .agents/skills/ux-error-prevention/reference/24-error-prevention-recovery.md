---
name: error-prevention-recovery
description: Undo, confirmation dialogs, destructive-action safeguards, and graceful-degradation errors and fixes for web UI.
---

# Error Prevention & Recovery

Nielsen's heuristic is "error prevention" first, "help users recognize, diagnose, and recover from errors" second — this file covers both halves: stopping mistakes before they happen, and making them cheap to undo when they do.

## Anti-pattern: Destructive actions with no confirmation
**What's wrong:** "Delete," "Remove all," "Archive workspace" fire immediately on click with no intermediate step.
**Why it matters:** Misclicks happen — a confirmation step is the cheapest possible insurance against an action that can't be undone.
**Fix:** Gate every irreversible, high-cost action behind a confirmation that restates the specific consequence (see `14-microcopy-content-strategy.md` for copy pattern); scale friction to actual severity — a single confirm click for moderate risk, a typed confirmation phrase for catastrophic/bulk-irreversible actions (e.g. deleting a whole workspace).

```
Delete workspace "Acme Corp"?
This will permanently delete 340 items and cannot be undone.
Type the workspace name to confirm: [__________]
[ Delete workspace ]  Cancel
```

## Anti-pattern: Confirmation dialogs added indiscriminately to low-stakes actions
**What's wrong:** A confirmation dialog appears for reversible, low-cost actions (archiving a single item, closing a panel, logging out).
**Why it matters:** Overusing confirmation dilutes its power — users start reflexively clicking through every dialog without reading, which then defeats the purpose for the genuinely dangerous ones.
**Fix:** Reserve confirmation dialogs strictly for actions that are both (a) hard/impossible to reverse and (b) meaningfully costly if done by mistake. For everything else, prefer undo (below) over upfront confirmation.

## Anti-pattern: No undo for reversible actions
**What's wrong:** Archiving, deleting a single item, or moving something to a different folder has no undo path — the only recovery is manually redoing the inverse action (if the user even notices in time).
**Why it matters:** Undo is frequently a better UX pattern than upfront confirmation: it doesn't slow down the common case (correct actions), while still providing a safety net for mistakes.
**Fix:** For actions that are cheap to reverse, skip the confirmation dialog and instead perform the action immediately with a visible, time-limited undo option (a toast with an "Undo" button is the standard pattern):

```jsx
deleteItem(id);
showToast("Item deleted", { action: { label: "Undo", onClick: () => restoreItem(id) } });
```

## Anti-pattern: Irreversible actions framed as reversible, or vice versa
**What's wrong:** Language implies something can be undone ("Move to trash") when it's actually permanent, or implies permanence for something that's actually fully recoverable.
**Why it matters:** Users calibrate their carefulness based on perceived reversibility — a mismatch between the label's implication and the true behavior leads either to careless permanent mistakes or unnecessary anxiety over a safe action.
**Fix:** Be precise: "Move to trash (recoverable for 30 days)" versus "Delete permanently" are meaningfully different promises — make sure the label matches the actual system behavior exactly.

## Anti-pattern: Bulk/multi-select destructive actions with no per-item review
**What's wrong:** Selecting 50 items and clicking "Delete" removes all 50 immediately with only a generic "Delete 50 items?" confirmation, no way to review which 50.
**Why it matters:** Bulk selection is exactly where accidental over-selection (e.g. a "select all" that included more than intended) causes the largest-blast-radius mistakes.
**Fix:** Show the actual count and, where feasible, a way to review/verify the selected set before committing bulk destructive actions; combine with undo for the whole batch where the operation is reversible.

## Anti-pattern: No graceful degradation when a non-critical feature fails
**What's wrong:** A single failed API call for a secondary widget (e.g. a "related items" panel) causes a JS error that breaks the entire page rather than just that widget.
**Why it matters:** See `09-feedback-loading-empty-error-states.md`'s error-boundary pattern — a UX-level error-prevention principle is that failure should be contained, not systemic.
**Fix:** Scope error boundaries and try/catch around independent, non-critical sections so a single failure degrades gracefully (that one section shows an error, everything else keeps working) rather than taking down the whole experience.

## Anti-pattern: Constraints/validation only enforced after the fact, never prevented up front
**What's wrong:** A date-range picker lets the user pick an end date before the start date, only rejecting it on submit; a quantity field allows negative numbers before validation catches it after submission.
**Why it matters:** Where a constraint is knowable at input time, preventing the invalid state from being enterable at all is strictly better than letting the user create it and then telling them it was wrong.
**Fix:** Prevent invalid states structurally wherever feasible (disable/hide the invalid options, clamp numeric inputs to valid ranges, disallow selecting an end date before the chosen start date) rather than relying purely on after-the-fact validation.

```html
<input type="number" min="0" max="99" />
```

## Anti-pattern: Auto-save with no way to recover from a bad auto-saved state
**What's wrong:** An editor auto-saves continuously, but if the user makes a mistake (accidentally deletes a large section) that gets auto-saved too, there's no version history or way to revert.
**Why it matters:** Auto-save removes the natural checkpoint that manual-save habits used to provide — without a version history, auto-save can auto-save a mistake just as readily as an intentional edit.
**Fix:** Pair auto-save with some form of version history or periodic snapshots the user can revert to, especially for content-authoring tools where the cost of losing a good prior state is high.

## Anti-pattern: Warnings shown but easily dismissed without reading
**What's wrong:** A one-time warning dialog with a large, easy-to-hit "Continue anyway" button and no friction, for a genuinely risky action.
**Why it matters:** A warning that's as easy to click through as a routine confirmation provides essentially zero actual protection.
**Fix:** For genuinely high-risk warnings, add proportionate friction (a short delay before the confirming button becomes clickable, a checkbox acknowledging the specific risk, or requiring the user to type a confirmation phrase) rather than a same-weight, instantly-clickable button.

## Quick Checklist
- [ ] Every irreversible, high-cost action requires confirmation proportionate to its actual risk
- [ ] Confirmation isn't overused on low-stakes, reversible actions — undo is preferred there instead
- [ ] Reversible actions offer a visible, time-limited undo rather than an upfront confirmation
- [ ] Labels accurately reflect true reversibility ("trash" vs. "delete permanently")
- [ ] Bulk destructive actions show the real affected count and a way to review the selection
- [ ] Non-critical feature failures are contained (error boundaries) and don't break the whole page
- [ ] Invalid states are structurally prevented at input time wherever feasible, not just caught after submit
- [ ] Auto-save is paired with version history/snapshots so a bad auto-saved state can be recovered
- [ ] High-risk warnings carry proportionate friction, not a same-weight, instantly-dismissible button
