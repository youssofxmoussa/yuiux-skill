---
name: search-filtering
description: Search UX, filters, facets, sort, and zero-result-state errors and fixes for web UI.
---

# Search & Filtering

## Anti-pattern: Search box with no visible affordance that it's a search box
**What's wrong:** A plain text input with no magnifying-glass icon, no placeholder hinting at what can be searched, sitting among other fields with nothing marking it as search.
**Why it matters:** Users scan for the familiar search icon/pattern; an unlabeled input gets skipped entirely.
**Fix:** Use `type="search"`, a search icon, and placeholder text describing scope: `placeholder="Search projects, tasks, and files"`.

## Anti-pattern: Search requires an exact match, case, or spelling
**What's wrong:** Searching "cofee" returns nothing because the stored term is "coffee"; searching "Acme" and "acme" return different results.
**Why it matters:** Real user input is messy — typos, casing, partial words are the norm, not the exception; a search that demands precision fails at its core job.
**Fix:** Search should be case-insensitive, substring/partial-match tolerant at minimum, and ideally fuzzy-matched (tolerant of small typos) and/or stemmed (matching "run"/"running"/"runs").

## Anti-pattern: No debounce, firing a request on every keystroke
**What's wrong:** Every character typed fires a new search request immediately.
**Why it matters:** Wastes requests, and out-of-order responses (a slow early-keystroke response arriving after a fast later one) can display stale/wrong results — see `11-performance-perceived-performance.md`.
**Fix:** Debounce (~200–300ms pause) and cancel/ignore stale in-flight requests when a newer query supersedes them.

## Anti-pattern: Zero-result state with no guidance
**What's wrong:** "No results" shown with nothing else — no suggestion to check spelling, broaden the search, or clear filters.
**Why it matters:** A dead end at exactly the moment the user is trying to find something specific.
**Fix:** Offer a specific next step: "No results for “invocie”. Did you mean “invoice”?" or "Try removing some filters," and if filters are active, show a one-click way to clear them alongside the message.

## Anti-pattern: Filters that silently produce zero results with no feedback on which filter caused it
**What's wrong:** A user has 4 filters active, results drop to zero, and there's no indication of which filter(s) are responsible.
**Why it matters:** Forces a slow trial-and-error process of toggling filters one at a time to diagnose the problem.
**Fix:** Show the active filter count/chips prominently with individual remove controls on each, so the user can selectively relax constraints rather than guessing or clearing everything.

```
Active filters: [Status: Active ×] [Owner: Me ×] [Date: Last 7 days ×]
0 results — try removing a filter above.
```

## Anti-pattern: Filter state lost on navigation or refresh
**What's wrong:** Applying filters, then navigating away and back (or refreshing) resets to the unfiltered default.
**Why it matters:** See `08-navigation-ia.md` — meaningful state should live in the URL, and filtered views are exactly the kind of state users want to bookmark/share/return to.
**Fix:** Reflect active filters in the URL query string; restore on load.

## Anti-pattern: No way to see how many results a filter combination will return before applying
**What's wrong:** Filter checkboxes with no counts shown, so applying a filter is a gamble on whether it'll return anything.
**Why it matters:** Users can't make informed decisions about which filters are worth trying.
**Fix:** Where feasible, show a result count next to each filter option based on the currently-applied other filters (standard faceted-search pattern): "Status: Active (14) · Archived (3)".

## Anti-pattern: Sort order changes silently reset scroll position or selection
**What's wrong:** Changing the sort dropdown on a list jumps the scroll back to top and deselects any in-progress multi-select, with no warning.
**Why it matters:** Loses the user's place and any in-progress work for what should be a minor reordering action.
**Fix:** Preserve scroll position and selection state across a re-sort where reasonably possible; if a selection genuinely can't survive (e.g. selection was position-based), warn before applying.

## Anti-pattern: Autocomplete/typeahead suggestions that don't match final search behavior
**What's wrong:** Typeahead suggests "Acme Corp — Invoices," but selecting it (or pressing Enter without selecting) produces different results than clicking the suggestion would have.
**Why it matters:** Breaks the user's trust that the suggestions reflect what will actually happen.
**Fix:** Ensure pressing Enter on the raw typed query and clicking a specific suggestion produce predictable, consistent, explainable results — and make clear which one the user is about to trigger (e.g. highlight the currently-focused suggestion distinctly).

## Anti-pattern: Search scope is unclear
**What's wrong:** A global search box with no indication of what it actually searches (just this page's list? the whole app? just titles, or also content?).
**Why it matters:** Users under- or over-trust the search depending on wrong assumptions about its scope, leading to false "no results" conclusions when the real content exists just outside the searched scope.
**Fix:** State scope explicitly (placeholder text, a scope selector, or grouped result sections labeled by source: "Projects (3) / Files (12) / People (1)").

## Quick Checklist
- [ ] Search input is visually unambiguous (icon, placeholder describing scope) and uses `type="search"`
- [ ] Matching is case-insensitive and tolerant of partial/typo'd input
- [ ] Input is debounced; stale responses are discarded
- [ ] Zero-result states suggest a specific next step (spelling check, broaden search, clear filters)
- [ ] Active filters are visible as removable chips, individually clearable
- [ ] Filter/search/sort state is reflected in the URL and survives navigation/refresh
- [ ] Filter options show result counts where feasible
- [ ] Scroll position and selection survive a sort-order change where possible
- [ ] Typeahead suggestions and actual search results behave consistently
- [ ] Search scope is explicit, not left to the user's assumption
