---
name: data-tables-dashboards
description: Dense-UI, table, chart, pagination-vs-infinite-scroll, and real-time-data errors and fixes for web UI.
---

# Data Tables & Dashboards

Dense, data-heavy interfaces have their own UX physics — the priorities from marketing/consumer UI (whitespace, minimalism above all) partially invert here: information density and scanability are the actual goals, but density is not an excuse to skip clarity, hierarchy, or feedback.

## Anti-pattern: Table with no fixed header when scrolling
**What's wrong:** A long table's column headers scroll out of view, leaving the user staring at rows of numbers with no idea which column is which.
**Why it matters:** Forces constant scroll-up-to-check-header-then-scroll-back-down, which is exhausting for any real analysis task.
**Fix:** Use a sticky header (`position: sticky; top: 0`) so column labels stay visible while scrolling the body.

## Anti-pattern: Numeric columns not right-aligned
**What's wrong:** Numbers left-aligned like text, making it hard to compare magnitudes at a glance down a column.
**Why it matters:** Right-alignment lets the eye compare digit-by-digit and immediately spot which values are larger; left-aligned numbers require reading each value individually.
**Fix:** Right-align numeric columns, left-align text columns, and use tabular figures (see `02-typography-readability.md`) so digits line up vertically too.

## Anti-pattern: No empty/zero-state distinct from a loading state in a table
**What's wrong:** A table shows the exact same visual (blank body) whether data is still loading or has genuinely returned zero rows.
**Why it matters:** The user can't tell if they should keep waiting or if there's really nothing there.
**Fix:** Distinguish explicitly: skeleton rows while loading, a clear "no results" row/message once loading completes with zero rows (see `09-feedback-loading-empty-error-states.md`).

## Anti-pattern: Pagination controls with no indication of total size
**What's wrong:** "Next / Previous" buttons with no "Page 3 of 47" or "Showing 21–40 of 932" context.
**Why it matters:** Users can't gauge how much more data exists or whether it's worth continuing to page through.
**Fix:** Always show total count and current position: "Showing 21–40 of 932 results."

## Anti-pattern: Infinite scroll used where users need to reach a specific, findable position (e.g. "the bottom," or a specific row to bookmark/share)
**What's wrong:** An admin table or search-result list uses infinite scroll, making it impossible to jump to "page 12" directly, share a specific position via URL, or reliably reach the end.
**Why it matters:** Infinite scroll is well-suited to casual, linear-consumption feeds (social media) but actively hurts task-oriented, comparison-heavy, or reference-lookup use cases where users need addressable, revisitable positions.
**Fix:** Use real pagination (with page numbers reflected in the URL) for task-oriented tables/lists; reserve infinite scroll for casual browsing/feed contexts, and even then consider a "load more" button over fully automatic infinite scroll to keep the footer reachable.

## Anti-pattern: No way to reorder, resize, hide, or persist table column configuration
**What's wrong:** A wide data table with a fixed column set and order that can't be customized, forcing every user into the same layout regardless of what they actually care about.
**Why it matters:** Power users of dense tables (the primary audience for dashboards/admin tools) frequently have different priority columns; a rigid table forces constant horizontal scrolling to reach what matters to them specifically.
**Fix:** Where the table is a core, frequently-used tool (not a one-off list), support column show/hide and reordering, and persist the user's configuration (per-user, not just per-session).

## Anti-pattern: Charts with no axis labels, units, or legend
**What's wrong:** A bar/line chart showing bars with numbers but no indication of what unit ($ ? % ? count?) or what time range is plotted.
**Why it matters:** A chart without units/context is decoration, not information — the viewer has to guess at the meaning of what they're looking at.
**Fix:** Every chart needs: axis labels with units, a legend (or direct labeling) for multiple series, and a visible time range/date context if temporal.

## Anti-pattern: Chart type mismatched to the data's actual relationship
**What's wrong:** A pie chart used for a time series, or a line chart used for unrelated discrete categories.
**Why it matters:** Pie charts communicate part-to-whole relationships; line charts communicate continuous trends over an ordered axis (usually time) — using the wrong chart type for the underlying data relationship actively misleads or confuses rather than clarifying.
**Fix:** Match chart type to the relationship being shown: line/area for trends over time, bar for comparing discrete categories, pie/donut sparingly and only for true part-of-a-whole with few (≤5–6) categories, scatter for correlation between two variables.

## Anti-pattern: Real-time/live-updating dashboard data shifts under the user's cursor
**What's wrong:** A table auto-refreshes and re-sorts every few seconds, causing rows to jump position exactly as the user is about to click one.
**Why it matters:** Causes misclicks on the wrong row and makes the interface feel unstable/untrustworthy for any careful review task.
**Fix:** Pause automatic re-sorting/reordering while the user's cursor is hovering/interacting with the table, or clearly flag new/changed data (e.g. a highlight pulse on updated rows) without silently reordering rows already visible; offer an explicit "refresh now" instead of forcing continuous reflow during active use.

## Anti-pattern: Dense dashboards with no visual hierarchy — every metric weighted equally
**What's wrong:** A dashboard with 20 KPI cards, all the same size, color, and prominence, with no indication of which 2–3 actually matter most right now.
**Why it matters:** Equal visual weight for unequal-importance information defeats the purpose of a dashboard, which should let users triage at a glance.
**Fix:** Establish a clear visual hierarchy — the most important 1–3 metrics larger/more prominent (top-left, largest type), secondary metrics smaller/grouped, with color reserved for genuinely notable states (e.g. a metric currently outside its healthy range), not applied uniformly.

## Anti-pattern: No way to export/copy table data
**What's wrong:** A data table with valuable information but no export (CSV/copy) option, forcing manual retyping if the user needs the data elsewhere.
**Why it matters:** Dashboards/tables are frequently a stepping stone to another tool (spreadsheet, presentation, report) — blocking that transition is a real functional gap, not just a nice-to-have.
**Fix:** Provide a copy-to-clipboard or CSV export action on any data table holding information users are likely to need elsewhere.

## Quick Checklist
- [ ] Table headers stay fixed/sticky while scrolling the body
- [ ] Numeric columns right-aligned with tabular figures; text columns left-aligned
- [ ] Loading state and genuine zero-result state are visually distinct
- [ ] Pagination shows total count and current position, not just Next/Previous
- [ ] Infinite scroll reserved for casual feeds; task-oriented tables use real, URL-addressable pagination
- [ ] Frequently-used tables support column customization, persisted per user
- [ ] Every chart has axis labels, units, and a legend/direct labeling
- [ ] Chart type matches the actual data relationship (trend/comparison/part-whole/correlation)
- [ ] Live-updating tables don't reorder/reflow under an actively-interacting cursor
- [ ] Dashboard metrics have a clear visual hierarchy, not uniform weighting
- [ ] Tables holding valuable data offer copy/export
