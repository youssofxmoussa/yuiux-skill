---
name: performance-perceived-performance
description: Core Web Vitals, perceived-speed techniques, lazy loading, and debouncing errors and fixes for web UI.
---

# Performance & Perceived Performance

Speed is a UX feature, not a separate engineering concern — every millisecond of unexplained delay reads to the user as friction, and every un-signposted wait reads as brokenness.

## Anti-pattern: No feedback within the first ~100ms of any user-initiated action
**What's wrong:** A click/tap produces zero visible response until the full underlying operation (network request, computation) resolves.
**Why it matters:** Human perception treats delays under ~100ms as instantaneous, but anything longer without feedback starts registering as lag, and by ~1 second the user's attention has already started to drift.
**Fix:** Separate "acknowledge the action" from "complete the action." Always render an immediate state change (pressed style, disabled state, spinner) synchronously on interaction, even if the real result is still pending.

## Anti-pattern: Large, unoptimized images shipped at full resolution
**What's wrong:** A 4000×3000px photograph displayed in a 400px-wide card, served at full original file size.
**Why it matters:** This is one of the largest and most avoidable contributors to slow page loads and poor Largest Contentful Paint (LCP) scores — the single Core Web Vital most correlated with perceived load speed.
**Fix:** Serve images sized and compressed for their actual display size, in modern formats, with responsive `srcset`:

```html
<img
  src="hero-800.webp"
  srcset="hero-400.webp 400w, hero-800.webp 800w, hero-1600.webp 1600w"
  sizes="(max-width: 600px) 100vw, 800px"
  width="800" height="450"
  alt="..."
  loading="lazy"
/>
```

## Anti-pattern: Everything loaded eagerly on initial page load
**What's wrong:** All images, all below-the-fold content, and all route bundles loaded up front regardless of whether the user will ever scroll to or navigate to them.
**Why it matters:** This delays the content the user actually needs first (above the fold, the current route) behind everything they might never need.
**Fix:** Lazy-load below-the-fold images/media (`loading="lazy"`), code-split by route so users only download the JS for the page they're on, and defer non-critical third-party scripts (analytics, chat widgets) until after the main content is interactive.

```js
const SettingsPage = lazy(() => import("./SettingsPage"));
```

## Anti-pattern: No debouncing/throttling on expensive, high-frequency handlers
**What's wrong:** A search-as-you-type field fires a network request on every single keystroke; a scroll or resize listener runs an expensive layout calculation on every pixel of movement.
**Why it matters:** This wastes bandwidth/compute and frequently causes visible jank or race-condition bugs (an older, slower request's response overwriting a newer one).
**Fix:** Debounce input-driven requests (wait for a pause in typing, typically 200–400ms) and throttle continuous events like scroll/resize (cap to animation-frame rate):

```js
const debouncedSearch = useMemo(() => debounce(runSearch, 300), []);
```
For search specifically, also cancel/ignore stale in-flight responses when a newer query has already been issued.

## Anti-pattern: Blocking the main thread with synchronous heavy computation
**What's wrong:** A large client-side data transformation, sort, or parse runs synchronously in response to a user action, freezing the UI (no scroll, no click response) until it completes.
**Why it matters:** A frozen UI is indistinguishable from a crashed one to the user in the moment it's happening.
**Fix:** Break heavy synchronous work into chunks (yield back to the browser between chunks), or move it off the main thread into a Web Worker for genuinely expensive operations.

## Anti-pattern: Layout shift after initial paint (poor CLS)
**What's wrong:** Ads, embeds, images without dimensions, or late-loading fonts cause visible content to jump after the page appears to have finished loading.
**Why it matters:** Beyond the Cumulative Layout Shift score, this is one of the most viscerally frustrating experiences on the web — misclicks on a button that moved, losing your reading position.
**Fix:** See `04-layout-spacing-grid.md`'s CLS section — reserve space for every async-loading element up front.

## Anti-pattern: Slow Time to Interactive despite fast visual paint
**What's wrong:** The page appears fully rendered quickly, but buttons/inputs don't actually respond for several more seconds while JavaScript finishes hydrating/executing.
**Why it matters:** This is arguably worse than a slow visual load — the user sees a "ready" page, tries to interact, and gets nothing, which reads as broken rather than slow.
**Fix:** Minimize the JS needed before interactivity (code-split aggressively, avoid large synchronous bundles), and where using server-rendering/hydration frameworks, prioritize hydrating above-the-fold interactive elements first.

## Anti-pattern: No pagination/virtualization on large lists
**What's wrong:** Rendering thousands of DOM nodes for a long list/table all at once.
**Why it matters:** Large unvirtualized DOM trees cause slow initial render, laggy scrolling, and high memory use — directly degrading the perceived quality of an otherwise well-designed list.
**Fix:** Paginate, use infinite-scroll with incremental loading, or virtualize (render only the visible window of rows, e.g. via a windowing library) for any list that can realistically grow into the hundreds or more.

## Anti-pattern: Network requests issued in a slow waterfall instead of in parallel
**What's wrong:** A page's data-fetching code awaits request A, then only starts request B after A resolves, when B doesn't actually depend on A's result.
**Why it matters:** This needlessly compounds latency — three independent 300ms requests run sequentially cost ~900ms; run in parallel they cost ~300ms.
**Fix:** Issue independent requests concurrently:

```js
// Bad
const user = await fetchUser();
const posts = await fetchPosts();

// Good
const [user, posts] = await Promise.all([fetchUser(), fetchPosts()]);
```

## Anti-pattern: Caching absent, so every navigation re-fetches everything
**What's wrong:** Returning to a previously-visited page/tab re-fetches and re-renders from a blank/loading state every time, even though nothing has changed.
**Why it matters:** Forces the user to sit through a redundant loading state for data they already saw seconds ago.
**Fix:** Cache fetched data (e.g. via a query library's built-in cache) keyed by request parameters, show cached data instantly on revisit while optionally revalidating in the background ("stale-while-revalidate"), rather than blanking and refetching from scratch.

## Anti-pattern: No perceived-speed techniques used at all for genuinely slow operations
**What's wrong:** An operation that will legitimately take several seconds (report generation, large export, AI processing) shows only a static spinner with no further context for its entire duration.
**Why it matters:** Unexplained multi-second waits feel much longer than they are, and users start to doubt whether anything is happening at all.
**Fix:** For genuinely long operations, use progressive disclosure of status ("Fetching data… / Processing… / Almost done…"), a real progress percentage if determinable, or at minimum a reassuring message that sets an expectation ("This usually takes about 20 seconds").

## Quick Checklist
- [ ] Every user-initiated action gets visible feedback within ~100ms, independent of the real operation's duration
- [ ] Images are served pre-sized/compressed for their actual display size, with `loading="lazy"` below the fold
- [ ] Routes/components are code-split; non-critical third-party scripts load deferred
- [ ] High-frequency handlers (search input, scroll, resize) are debounced/throttled
- [ ] Heavy synchronous computation is chunked or moved to a worker, never freezing the main thread
- [ ] Image/embed dimensions are reserved to prevent layout shift
- [ ] Interactivity is prioritized to match visual readiness (no "looks ready but doesn't respond" gap)
- [ ] Large lists are paginated, infinite-scrolled, or virtualized
- [ ] Independent network requests run in parallel, not a sequential waterfall
- [ ] Previously-fetched data is cached and shown instantly on revisit
- [ ] Genuinely long operations show progressive status, not just an indefinite spinner
