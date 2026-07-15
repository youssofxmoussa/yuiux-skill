---
name: microcopy-content-strategy
description: Error tone, button labels, confirmation copy, and placeholder-text errors and fixes for web UI writing.
---

# Microcopy & Content Strategy

Every word in a UI is a design decision. Bad microcopy makes a well-built interface feel untrustworthy; good microcopy can rescue an imperfect one.

## Anti-pattern: Blaming or accusatory tone in errors
**What's wrong:** "You entered an invalid value." "You must fix the errors below." "Incorrect password."
**Why it matters:** Framing the system's inability to process input as the user's fault reads as hostile, even when technically accurate — it violates Nielsen's "help users recover from errors" heuristic, which explicitly calls for constructive, non-blaming phrasing.
**Fix:** Describe the situation and the fix without assigning blame:

```
Bad:  "You entered an invalid value."
Good: "That doesn't look like a valid phone number. Try including the area code."

Bad:  "Incorrect password."
Good: "That password doesn't match this account. Try again or reset your password."
```

## Anti-pattern: Vague, non-actionable error copy
**What's wrong:** "Something went wrong." "An error occurred." "Oops!"
**Why it matters:** Tells the user nothing about what happened or what to do next — see `09-feedback-loading-empty-error-states.md` for the full pattern. Listed here specifically as a *writing* discipline: never let a placeholder-quality error string ship to production.
**Fix:** Name the failure and the recovery action, every time, no exceptions: "Couldn't save your changes — check your connection and try again."

## Anti-pattern: Overly technical or internal jargon exposed to end users
**What's wrong:** "Payload validation failed," "Null reference in handler," "The tenant record could not be resolved," shown verbatim to a non-technical end user.
**Why it matters:** Internal engineering vocabulary means nothing to the audience and actively signals the product wasn't finished — it exposes implementation details that should be an internal concern.
**Fix:** Translate every user-facing string to plain language a non-technical person in the target audience would use, and log/report the technical detail separately (error tracking) rather than surfacing it in the UI.

## Anti-pattern: Inconsistent terminology for the same concept
**What's wrong:** The same object is called "item" on the list page, "entry" on the detail page, and "record" in an email notification.
**Why it matters:** Inconsistent naming forces the user to work out, each time, whether these are actually the same thing — a small but real tax on comprehension repeated across the whole product.
**Fix:** Maintain a small glossary of core nouns/verbs for the product (even just a short list in project documentation) and use them identically everywhere: UI copy, emails, error messages, empty states, help docs.

## Anti-pattern: Button/action labels that don't match the resulting screen's heading
**What's wrong:** A button says "Get started" but leads to a page titled "Account setup," or a nav item says "Team" but the destination page is titled "Members."
**Why it matters:** Mismatched labels break the scent trail users follow to navigate confidently — a small mismatch causes a moment of doubt ("did I click the right thing?") on every use.
**Fix:** Keep the label a user clicks and the heading of where it leads either identical or obviously synonymous — check this explicitly for every nav item, button, and link during review.

## Anti-pattern: Confirmation dialogs with unclear or symmetric-looking Yes/No framing
**What's wrong:** A destructive confirmation dialog with generic "OK"/"Cancel" buttons that don't restate what's being confirmed.
**Why it matters:** A user skimming quickly (which is the norm, not the exception) can't tell from "OK" alone what they're actually agreeing to, especially if they've seen many similar-looking dialogs before.
**Fix:** Restate the specific action in both the dialog body and the confirming button's label:

```
Delete "Q3 Budget Report"?
This can't be undone.
[ Delete report ]   Cancel
```
Never use bare "Yes"/"No" for anything consequential — restate the action in the button itself so the user isn't relying on having read the header carefully.

## Anti-pattern: Overpromising in marketing copy that the product then doesn't deliver
**What's wrong:** Landing-page copy or in-app messaging that describes a capability more powerfully than the actual feature ("AI-powered insights" for a feature that's just a static average calculation).
**Why it matters:** The gap between promise and delivery is exactly where trust erodes — users forgive limitations they were told about upfront far more than limitations they discover after being oversold.
**Fix:** Write copy that's accurate to what's actually shipped; if a feature is genuinely limited (beta, partial coverage, approximate), say so plainly rather than let the user discover the limitation on their own.

## Anti-pattern: Placeholder/lorem-ipsum text or default framework strings left in shipped UI
**What's wrong:** "Lorem ipsum," "Your headline here," "Item name," or a framework's default scaffold text visible in a production interface.
**Why it matters:** Signals the product is unfinished, immediately undermining confidence regardless of how polished everything else is.
**Fix:** Treat every visible string as a review item before shipping — grep for common placeholder patterns ("lorem", "TODO", "placeholder", "Item 1", "test test") as a final pass.

## Anti-pattern: Humor or cuteness misapplied to serious/high-stakes contexts
**What's wrong:** A playful, joke-y tone applied to an error that just lost the user's data, a billing failure, or a security warning ("Uh oh, spaghetti-o! Something broke!").
**Why it matters:** Tone mismatch between the message and the actual stakes reads as tone-deaf and can feel dismissive of a genuinely frustrating moment.
**Fix:** Calibrate tone to stakes — light, human warmth is fine for empty states and onboarding; be direct, clear, and appropriately serious for errors involving money, data loss, or security. When unsure, default to plain and respectful over clever.

## Anti-pattern: Truncated or cryptic labels to save space, with no full-text fallback
**What's wrong:** A button labeled just "Del" or "Cfg" to fit a tight layout, or a column header abbreviated with no explanation.
**Why it matters:** Saves visual space at the cost of comprehension — users shouldn't have to decode abbreviations to use core functionality.
**Fix:** Prefer redesigning the layout to fit full words over abbreviating; if abbreviation is unavoidable (e.g. a very narrow table column), pair it with a tooltip or `title` attribute giving the full term.

## Anti-pattern: Time-sensitive or relative dates presented ambiguously
**What's wrong:** "2 days ago" shown with no way to see the exact date/time, or a raw ISO timestamp ("2026-07-14T03:22:10Z") shown to a non-technical user with no localization.
**Why it matters:** Relative time is friendlier for a glance but loses precision when it matters (was that just now or a week ago exactly?); raw ISO timestamps are unreadable to most users and ignore timezone.
**Fix:** Show relative time for recent events with the exact localized timestamp available on hover/tap ("2 days ago" with a tooltip showing "July 12, 2026, 3:22 PM"), and always localize to the user's timezone and date-format convention.

## Quick Checklist
- [ ] No error message blames or accuses the user
- [ ] Every error names the problem and a specific recovery action
- [ ] No internal/technical jargon exposed in user-facing copy
- [ ] Core nouns/verbs for the product are used consistently everywhere (UI, emails, docs)
- [ ] Button/link labels match the heading of the screen they lead to
- [ ] Confirmation dialogs restate the specific action in the body and the confirming button, never bare "Yes/OK"
- [ ] Marketing/feature copy is accurate to what's actually shipped, not aspirational
- [ ] No leftover placeholder/lorem-ipsum/default scaffold text in shipped UI
- [ ] Tone is calibrated to stakes — plain and clear for money/data/security, warmer elsewhere
- [ ] No cryptic abbreviations without a full-text fallback
- [ ] Relative dates pair with an exact, localized timestamp on hover/tap
