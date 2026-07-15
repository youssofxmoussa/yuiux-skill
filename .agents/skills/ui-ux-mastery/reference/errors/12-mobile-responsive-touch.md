---
name: mobile-responsive-touch
description: Touch targets, gestures, viewport quirks, and responsive layout failure modes and fixes for web UI.
---

# Mobile, Responsive & Touch

## Anti-pattern: Designing desktop-first and shrinking down
**What's wrong:** Building the full desktop layout first, then squeezing it into mobile widths via media queries as an afterthought.
**Why it matters:** Desktop-first layouts routinely carry assumptions (hover-based reveal, wide multi-column grids, generous horizontal space) that simply don't translate — the mobile version ends up as a compromise rather than a considered experience, and for most consumer products mobile traffic is the majority, not the exception.
**Fix:** Design and build the smallest viewport first, then progressively enhance for more space (mobile-first CSS: base styles apply to mobile, `min-width` media queries add complexity for larger screens), so the default experience is never an afterthought.

```css
.card { padding: var(--space-3); }        /* mobile default */
@media (min-width: 768px) {
  .card { padding: var(--space-5); }      /* enhanced for larger screens */
}
```

## Anti-pattern: Fixed viewport meta tag or missing viewport tag
**What's wrong:** No `<meta name="viewport">` tag at all, or one that disables zoom (`user-scalable=no`, `maximum-scale=1`).
**Why it matters:** Without a proper viewport tag, mobile browsers render at a desktop-width virtual viewport and scale the whole page down, making everything tiny and forcing manual pinch-zoom just to read. Disabling zoom entirely removes a critical accessibility affordance for low-vision users who rely on pinch-zoom.
**Fix:**
```html
<meta name="viewport" content="width=device-width, initial-scale=1" />
```
Never add `maximum-scale=1` or `user-scalable=no`.

## Anti-pattern: Horizontal scroll caused by unconstrained content
**What's wrong:** A wide table, a long unbroken string, or an element with a fixed pixel width wider than the viewport causes the whole page to scroll sideways.
**Why it matters:** Horizontal scrolling on the page level (as opposed to a deliberately contained horizontal-scroll component) is almost always unintentional and immediately signals a broken layout.
**Fix:** Constrain content with `max-width: 100%`, use `overflow-x: auto` scoped to the specific wide element (e.g. a table wrapper) rather than letting it blow out the page, and use `word-break`/`overflow-wrap` for unpredictable long strings (URLs, IDs).

```css
.table-wrapper { overflow-x: auto; max-width: 100%; }
.long-text { overflow-wrap: break-word; }
```

## Anti-pattern: Hover-dependent interactions with no touch equivalent
**What's wrong:** A dropdown/tooltip/menu that only opens on `:hover`, or an action revealed only when hovering over a row.
**Why it matters:** Touch devices have no hover state — content or controls that exist only on hover are completely inaccessible on mobile/tablet.
**Fix:** Trigger on click/tap (with hover as a bonus enhancement on devices that support it, via `@media (hover: hover)`), not as the only path.

## Anti-pattern: Tap targets sized for a mouse cursor
**What's wrong:** Small icon buttons, checkboxes, or link text sized around 24–32px with tight spacing, fine for a precise mouse pointer but not a fingertip.
**Why it matters:** See Fitts's Law and the WCAG 2.5.5 target-size guidance — repeated in this file because it's the single most common mobile-specific usability failure. Average adult fingertip contact area is roughly 45–57px; anything meaningfully smaller invites mis-taps.
**Fix:** ≥44×44px tap area (padding, not just visual size) with ≥8px gaps between adjacent independent targets — see `07-buttons-controls.md`.

## Anti-pattern: Bottom-of-screen critical actions obstructed by OS chrome or the on-screen keyboard
**What's wrong:** A "Submit" or "Continue" button pinned near the bottom edge gets covered by the mobile browser's address bar/home indicator, or by the virtual keyboard when a text field above it is focused.
**Why it matters:** The user literally cannot see or reach the button they need to complete the task.
**Fix:** Account for safe-area insets on iOS (`env(safe-area-inset-bottom)`), and when a form field is focused with the keyboard open, either scroll the active field (and its submit action, if adjacent) into the visible viewport, or keep primary actions reachable above the fold rather than only at the very bottom.

```css
.bottom-bar {
  padding-bottom: max(var(--space-3), env(safe-area-inset-bottom));
}
```

## Anti-pattern: Native browser UI conflicts with custom gesture handling
**What's wrong:** Custom swipe-to-dismiss or horizontal-swipe interactions that conflict with the browser's native back-swipe gesture or pull-to-refresh, causing accidental navigation or refresh.
**Why it matters:** Users lose work or get bounced out of a flow because the app's custom gesture and the OS/browser's built-in gesture were fighting for the same swipe.
**Fix:** Scope custom swipe handling narrowly (e.g. only within a specific horizontal carousel), avoid claiming the full-width edge zones the browser uses for back-navigation, and disable pull-to-refresh (`overscroll-behavior: contain` / `overscroll-behavior-y: contain`) on views where an accidental refresh would be destructive (e.g. mid-form).

## Anti-pattern: Double-tap-to-zoom triggered accidentally by fast taps
**What's wrong:** Rapid legitimate taps (e.g. double-tapping a "like" button, a common social-app gesture) accidentally trigger the browser's native double-tap-zoom instead of the intended action.
**Why it matters:** Confuses an intended in-app interaction with an unrelated browser behavior.
**Fix:** Set `touch-action: manipulation` on interactive elements to suppress double-tap-zoom on that element while preserving normal scrolling elsewhere.

## Anti-pattern: Assuming one input type (mouse or touch), never both
**What's wrong:** Detecting "mobile" purely by screen width and assuming narrow-width devices are always touch and wide devices are always mouse — missing touch laptops, tablets in landscape with a connected mouse/trackpad, and foldables.
**Why it matters:** Width-based device assumptions increasingly don't match reality; a hybrid device can have a wide viewport and touch input, or vice versa.
**Fix:** Design controls to work with both input types regardless of viewport width (adequate tap targets *and* proper hover/focus states); use `@media (pointer: coarse)` / `(pointer: fine)` when input-type-specific behavior is genuinely needed, rather than inferring it from viewport width alone.

## Anti-pattern: Orientation change breaks the layout
**What's wrong:** Rotating a phone/tablet from portrait to landscape causes overlapping elements, clipped content, or a broken fixed-height container.
**Why it matters:** Users rotate devices constantly (especially for video, tables, or wide charts) and expect the layout to adapt gracefully, not break.
**Fix:** Avoid fixed viewport-height assumptions (`100vh` can behave inconsistently across orientation changes and mobile browser chrome show/hide); test explicitly in both orientations, not just portrait.

## Anti-pattern: Text input zooms the page on focus
**What's wrong:** An input with `font-size` below 16px causes iOS Safari to auto-zoom the whole page when the user taps into it.
**Why it matters:** The jarring zoom-in disorients the user and often requires them to manually zoom back out to see the rest of the form.
**Fix:** Keep form input font sizes at 16px or larger (see `02-typography-readability.md`).

## Anti-pattern: Native mobile controls unnecessarily reimplemented
**What's wrong:** A custom date picker, time picker, or dropdown built entirely from scratch instead of using native `<input type="date">`, `<input type="time">`, or `<select>` where the native control is genuinely sufficient.
**Why it matters:** Native mobile controls are already optimized for touch, already accessible, already familiar to the user from every other app on their device, and free — reimplementing them usually produces something worse on at least one of those axes.
**Fix:** Default to native form controls unless there's a specific, justified reason (e.g. a date-range picker with visual calendar context that native inputs genuinely can't provide) to build a custom one — and if you do build custom, make sure it matches or exceeds native accessibility/touch behavior, not just its visual polish.

## Quick Checklist
- [ ] Layouts are designed mobile-first, then progressively enhanced for larger viewports
- [ ] Viewport meta tag present and never disables user zoom
- [ ] No unintended horizontal page scroll; wide elements are locally scoped and scrollable
- [ ] No interaction depends on hover alone; tap/click always works as a full alternative
- [ ] All tap targets ≥44×44px with adequate spacing
- [ ] Bottom-anchored actions account for safe-area insets and on-screen keyboard overlap
- [ ] Custom swipe gestures don't fight the browser's native back-swipe/pull-to-refresh
- [ ] `touch-action: manipulation` set where rapid taps shouldn't trigger zoom
- [ ] Controls work for both touch and mouse/trackpad regardless of viewport width
- [ ] Layout tested and correct in both portrait and landscape
- [ ] Form input font size ≥16px to prevent iOS auto-zoom
- [ ] Native form controls used by default; custom controls only when justified and at least as accessible
