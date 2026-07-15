---
name: ux-guidelines-reference
description: 99-rule cross-cutting UX guideline table (navigation, animation, layout, touch, interaction) with do/don't code examples and severity ratings. Raw table in data/ux-guidelines.csv — overlaps and complements reference/errors/ but with copy-paste-ready code snippets.
---

# UX Guidelines Reference

Full raw data (99 rows: category, issue, platform, description, do, don't, good code example, bad code example, severity) lives in `../../data/ux-guidelines.csv`. This file summarizes by category with the highest-severity rules called out; the CSV has an exact copy-paste code snippet for every rule. This complements — rather than duplicates — the deeper narrative coverage in `../errors/`; use the CSV when you want an instant code-level fix, use `../errors/` when you want the fuller "why" and edge cases.

## Navigation (High/Medium/Low)
- Smooth-scroll anchor links (`scroll-behavior: smooth`), and compensate sticky-nav height with matching body/section padding so the nav never overlaps content.
- Always visually mark the active nav item (color/underline) — never leave every link styled identically regardless of current location.
- Preserve back-button history correctly (`history.pushState`, not `location.replace`) so back behaves predictably.
- Update the URL to reflect view/state changes so pages are shareable and bookmarkable; reserve breadcrumbs for sites 3+ levels deep (skip them on flat single-level sites — they add clutter with no wayfinding benefit there).

## Animation (High/Medium/Low)
- Animate 1-2 key elements per view maximum — animating everything that moves reads as chaotic, not lively.
- Keep micro-interaction duration to 150-300ms; anything over ~500ms on core UI feels sluggish, not smooth.
- Always honor `prefers-reduced-motion` — this is flagged High severity, not a nice-to-have.
- Any async wait needs a skeleton/spinner; a frozen UI with zero feedback is a High-severity failure, not a minor gap.
- Hover-only interactions exclude touch users — always provide a tap/click equivalent for anything functionally important, not just a hover state.
- Reserve infinite/looping animation for loading indicators only; looping decorative elements (bouncing icons, etc.) is distracting.
- Animate `transform`/`opacity`, never `width`/`height`/`top`/`left` — those properties trigger expensive layout recalculation.
- Use `ease-out` for entering elements and `ease-in` for exiting ones; linear easing on UI transitions reads as robotic.

## Layout (High/Medium)
- Maintain a defined z-index scale (e.g. 10/20/30/50) rather than reaching for arbitrary large values (`z-[9999]`) that make future stacking-context conflicts unpredictable.
- Test that content actually fits before applying `overflow-hidden` — it's a common source of silently clipped content.
- Reserve layout space for async content (fixed dimensions or `aspect-ratio`) to prevent layout shift when images/data load — an unreserved layout shift is High severity, not cosmetic.
- Use `dvh`/`min-h-dvh` instead of `100vh` on mobile web — `100vh` is unreliable once mobile browser chrome (address bar) is factored in.
- Cap text-content width to ~65-75 characters (`max-w-prose`) — full-viewport-width paragraphs are measurably harder to read.
- Understand what creates a new CSS stacking context (a positioned element with a set z-index, `transform`, `filter`, etc.) before debugging "z-index not working" — it's almost always a stacking-context issue, not a wrong z-index value.

## Touch (High/Medium/Low — mobile web & native)
- Minimum 44×44px touch targets; minimum 8px gap between adjacent targets.
- Avoid overriding system horizontal-swipe-back with a custom gesture on the main content area — reserve horizontal swipe for an explicit carousel.
- Use `touch-action: manipulation` to remove the legacy 300ms tap delay on mobile web.
- Disable pull-to-refresh (`overscroll-behavior: contain`) where it isn't a meaningful action, to prevent accidental refresh.
- Use haptic feedback sparingly — for confirmations and important actions only, never on every tap (fatigues and desensitizes the signal).

## Interaction / Focus (High)
- Every interactive element needs a visible focus ring (`focus:ring-2` or equivalent) — removing `outline` without a replacement is a High-severity accessibility failure, not a stylistic preference.

## How to apply this file
When implementing any interactive component, cross-check it against the relevant category above before considering it done — most of these are one-line fixes (a CSS property, a media query, an ARIA attribute) that are cheap to apply proactively but expensive to retrofit once a pattern is copied across a codebase.
