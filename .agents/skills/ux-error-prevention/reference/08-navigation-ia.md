---
name: navigation-ia
description: Navigation patterns, breadcrumbs, tabs, sidebars, deep linking, and back-button correctness errors and fixes for web UI.
---

# Navigation & Information Architecture

## Anti-pattern: No indication of current location
**What's wrong:** A nav bar/sidebar where the currently-active page/tab looks identical to every inactive one.
**Why it matters:** Users orient themselves largely through "where am I relative to everything else" — without an active-state cue, every navigation action feels like jumping into the unknown rather than moving within a known structure.
**Fix:** Give the active nav item a persistent, unambiguous visual difference (not just on hover) — background fill, bold weight, underline, or accent-colored indicator — and keep the browser tab title and any breadcrumb in sync with it.

```jsx
<NavLink className={({ isActive }) => isActive ? "nav-item active" : "nav-item"}>
  Dashboard
</NavLink>
```

## Anti-pattern: Icon-only navigation with no labels
**What's wrong:** A sidebar of unlabeled icons (home, folder, gear, bell) relying purely on shape recognition.
**Why it matters:** Icon meaning is far less universal than designers assume — a "folder" icon might mean projects, files, or teams depending on the app, and there's no way for a new user to guess correctly consistently.
**Fix:** Pair icons with visible text labels wherever there's room (sidebars, bottom tab bars); if space is truly constrained, provide a labeled tooltip on hover/focus and always include `aria-label` for screen readers regardless.

## Anti-pattern: Hamburger menu used on desktop where there's no space constraint
**What's wrong:** Collapsing primary navigation behind a hamburger icon on a wide desktop viewport that has ample room to show it directly.
**Why it matters:** Hidden navigation measurably reduces discovery and use of the items inside it compared to visible navigation — hiding it on desktop trades discoverability for no real space benefit.
**Fix:** Show primary navigation directly (top bar or persistent sidebar) on desktop/tablet widths; reserve the hamburger pattern for genuinely narrow (mobile) viewports where space is the actual constraint.

## Anti-pattern: Deep or inconsistent menu structures
**What's wrong:** Reaching a common task requires navigating through 4+ nested menu levels, or the same feature is reachable via two structurally different paths that behave slightly differently.
**Why it matters:** Every additional navigation level compounds the chance of the user taking a wrong turn, and inconsistent paths break the mental model users build of "how this app is organized."
**Fix:** Keep primary/common tasks reachable within 2–3 clicks from anywhere in the app; if a feature is accessible from multiple paths, make sure every path leads to the exact same state, not near-duplicates.

## Anti-pattern: No breadcrumbs on deep hierarchical content
**What's wrong:** A nested content structure (categories → subcategories → items, or folders → subfolders → files) with no way to see or jump to a parent level short of repeatedly clicking "back."
**Why it matters:** Users lose track of their place in the hierarchy and can't jump directly to a known ancestor level.
**Fix:** Add a breadcrumb trail for any content nested 2+ levels deep, with every ancestor level clickable:

```html
<nav aria-label="Breadcrumb">
  <ol>
    <li><a href="/projects">Projects</a></li>
    <li><a href="/projects/acme">Acme</a></li>
    <li aria-current="page">Invoices</li>
  </ol>
</nav>
```

## Anti-pattern: Tabs used for content that isn't actually parallel/mutually exclusive
**What's wrong:** Using a tab pattern for what's actually a sequential wizard, or for content the user commonly needs to compare side-by-side.
**Why it matters:** Tabs imply "pick one view of the same subject" — using them for sequential steps hides the fact that order/progress matters, and using them where comparison is common forces constant re-switching and loses context.
**Fix:** Reserve tabs for genuinely parallel, independent views of the same entity (e.g. "Overview / Activity / Settings" for one project). Use a stepper/wizard UI for sequential flows, and a side-by-side layout (or an explicit "compare" feature) when comparison is a common task.

## Anti-pattern: Browser back button doesn't do what the user expects
**What's wrong:** In a single-page app, pressing browser back exits the whole app, skips over an in-app modal/step the user just navigated through, or lands on a blank/broken state.
**Why it matters:** Back-button behavior is one of the most deeply ingrained expectations on the web — breaking it feels like the app is fighting the browser, and can lose user context entirely.
**Fix:** Use real URL-backed routing (not just in-memory state) for every navigable view, including modal/drawer opens where feasible (`?modal=edit-item`), so back/forward and deep links/refreshes all behave correctly and predictably.

## Anti-pattern: Refreshing the page loses the user's place
**What's wrong:** A filtered/sorted list, an open detail panel, or a specific tab resets to the default state on refresh because that state lives only in memory, not the URL.
**Why it matters:** Users refresh pages constantly (intentionally or by accident) and expect to land back where they were, especially for anything they might want to bookmark or share.
**Fix:** Reflect meaningful UI state (filters, sort, active tab, open item, page number) in the URL query string, and read it back on load.

```js
// ?status=active&sort=recent&page=2
const [params, setParams] = useSearchParams();
```

## Anti-pattern: No way to reach a "home" or safe default state
**What's wrong:** No persistent logo/home link, or clicking it does nothing / goes somewhere unexpected.
**Why it matters:** Users treat the logo-as-home-link as a near-universal escape hatch; its absence or misbehavior removes a safety net they're relying on.
**Fix:** Make the logo/brand mark in the header a link back to the app's home/dashboard, consistently, on every page.

## Anti-pattern: Search buried or missing when the content volume warrants it
**What's wrong:** A content-heavy app (many items, documents, records) with no visible search, forcing users to browse/filter manually.
**Why it matters:** Once content volume crosses a threshold, browsing stops scaling and search becomes the primary navigation method for returning users who know roughly what they want.
**Fix:** Surface a persistent, easily-reachable search entry point (not buried in a menu) once there are more than a screen or two's worth of items to navigate.

## Anti-pattern: Dead-end states with no forward action
**What's wrong:** A 404 page, an empty search result, or an error state that offers no link/action to move forward — just an explanation.
**Why it matters:** Every screen a user can land on should have at least one path forward; leaving them stuck is a guaranteed abandonment point.
**Fix:** Every dead-end screen gets at least one clear next action: "Go to homepage," "Clear filters," "Try a different search," "Contact support," etc.

## Anti-pattern: Inconsistent scroll position handling on navigation
**What's wrong:** Navigating to a new page/route keeps the previous page's scroll position instead of resetting to top, or vice versa when returning to a list should restore the previous scroll position.
**Why it matters:** Landing mid-page on new content is disorienting; losing scroll position when returning to a previously-scrolled list forces re-finding your place.
**Fix:** Reset scroll to top on navigating to a genuinely new page/content; restore prior scroll position specifically when returning to a list/feed the user was previously scrolled through (most routers provide scroll-restoration utilities for this distinction).

## Quick Checklist
- [ ] Active nav/tab state is persistently and unambiguously visible, not just on hover
- [ ] Navigation icons are paired with visible labels wherever space allows
- [ ] Hamburger menus reserved for genuinely narrow viewports, not hidden on desktop for no reason
- [ ] Common tasks reachable within 2–3 clicks from anywhere; duplicate paths lead to identical results
- [ ] Breadcrumbs present for hierarchies 2+ levels deep
- [ ] Tabs used only for parallel/independent views, not sequential flows or frequently-compared content
- [ ] Browser back/forward and refresh behave correctly via real URL-backed routing
- [ ] Meaningful UI state (filters/sort/tab/page) is reflected in the URL
- [ ] Logo/brand mark reliably links home on every page
- [ ] Search is visible and reachable once content volume warrants it
- [ ] Every dead-end screen (404, empty, error) offers at least one forward action
- [ ] Scroll position resets on new pages, restores when returning to a previously-scrolled list
