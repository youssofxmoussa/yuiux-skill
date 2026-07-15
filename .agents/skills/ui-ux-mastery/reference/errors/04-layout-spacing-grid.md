---
name: layout-spacing-grid
description: Spacing scales, grid systems, alignment, visual rhythm, and responsive breakpoint errors and fixes for web UI.
---

# Layout, Spacing & Grid

## Anti-pattern: No spacing scale, arbitrary pixel values everywhere
**What's wrong:** `margin: 13px`, `padding: 18px`, `gap: 22px` scattered ad hoc through the stylesheet.
**Why it matters:** Inconsistent spacing is one of the fastest "tells" of an unpolished UI — even when individual gaps look fine in isolation, the lack of rhythm across the page reads as sloppy.
**Fix:** Define a spacing scale (4px or 8px base unit is standard) and use only scale values everywhere:

```css
:root {
  --space-1: 4px;
  --space-2: 8px;
  --space-3: 12px;
  --space-4: 16px;
  --space-5: 24px;
  --space-6: 32px;
  --space-7: 48px;
  --space-8: 64px;
}
```
Every `margin`, `padding`, and `gap` in the app should resolve to one of these tokens (via CSS variables, Tailwind's default scale, or your framework's spacing utilities). If a spot genuinely needs an in-between value, that's a sign the scale itself needs a step added — not a one-off exception.

## Anti-pattern: Inconsistent spacing between similar elements
**What's wrong:** Card padding is 16px on one page and 20px on another; the gap between form fields is 12px in one form and 16px in the next.
**Why it matters:** Small inconsistencies are individually invisible but collectively create a subtle sense that the app wasn't built with care.
**Fix:** Componentize spacing decisions once (a `Card`, `Stack`, or `FormField` component that owns its internal spacing) rather than re-deciding padding/gap at every call site.

## Anti-pattern: Elements not aligned to a shared grid/baseline
**What's wrong:** A page's left edges (labels, headings, card content) don't line up vertically because each section picked its own padding.
**Why it matters:** Misaligned edges create invisible visual "static" — the eye has to work harder to parse structure that should be free.
**Fix:** Establish one outer container width/gutter and align all sections' left/right edges to it. Use a real grid (`display: grid` with defined columns, or a 12-column container) for multi-column layouts rather than eyeballing widths.

```css
.page-container {
  max-width: 1200px;
  margin-inline: auto;
  padding-inline: var(--space-5);
}
```

## Anti-pattern: No clear visual grouping — flat wall of fields/cards
**What's wrong:** A settings page with 20 fields in one continuous list, no section headers, no grouping whitespace.
**Why it matters:** Per Miller's Law, users process a handful of chunks at a time, not 20 flat items — an ungrouped list is cognitively exhausting to scan.
**Fix:** Group related fields under section headings, with more space between groups than within a group ("proximity" from Gestalt principles: things that are related should be visually closer than things that aren't).

```
[Profile]
  Name        [_______]
  Email       [_______]
        <-- larger gap here than between fields above -->
[Notifications]
  Email alerts [toggle]
  SMS alerts   [toggle]
```

## Anti-pattern: Center-aligned body text/paragraphs
**What's wrong:** `text-align: center` applied to multi-line paragraphs, form labels, or long descriptions "because it looks balanced."
**Why it matters:** Centered text has a ragged left edge, which the eye needs to track back to on every new line — this materially slows reading for anything longer than a single short line.
**Fix:** Reserve center alignment for short, single-line elements (a hero headline, a button label, an empty-state title). Left-align (or right-align for RTL) anything users need to actually read.

## Anti-pattern: Cramming too much into the viewport with no breathing room
**What's wrong:** Every available pixel filled with content, zero whitespace margins, elements touching the viewport edges.
**Why it matters:** Whitespace is not empty/wasted space — it's what lets the eye separate one group from another and rest between tasks. Its absence causes visual fatigue and makes the busiest UI elements compete for attention equally.
**Fix:** Enforce a minimum outer margin/padding on every screen (typically ≥16px on mobile, ≥24–48px on desktop) and be deliberate about whitespace between sections, not just around the outer edge.

## Anti-pattern: Layout breaks or overlaps at non-standard viewport widths
**What's wrong:** Layout tested only at exactly 1440px and 375px; anything in between (common laptop widths like 1024–1280px, foldables, browser windows not maximized) shows overlapping elements or broken wrapping.
**Why it matters:** Real users resize windows, use split-screen, and own a huge range of device widths — a layout that only works at two exact breakpoints is fragile in practice.
**Fix:** Use fluid, `flex`/`grid`-based layouts with `min()`/`max()`/`clamp()` rather than fixed pixel widths wherever possible, and actually drag-resize the browser window across the full range while testing, not just check three fixed device presets.

```css
.hero-title {
  font-size: clamp(1.75rem, 4vw + 1rem, 3rem);
}
```

## Anti-pattern: Fixed-height containers that clip dynamic content
**What's wrong:** A card given a hardcoded `height: 200px` that clips or overflows once real (longer) content is dropped in, versus the placeholder text used while designing.
**Why it matters:** Content length is rarely uniform in production — user names, item titles, and descriptions vary widely in length, and a layout that only survives the demo data is not survivable.
**Fix:** Use `min-height` instead of fixed `height` for any container holding variable-length content, and explicitly design/test the overflow behavior (wrap, truncate + tooltip, or scroll) rather than letting it clip silently.

## Anti-pattern: Z-index chaos
**What's wrong:** Random z-index values (`z-index: 9999`, `z-index: 100000`) sprinkled wherever something didn't stack correctly, with no system.
**Why it matters:** Eventually two "highest priority" elements collide and one wins arbitrarily and unpredictably (e.g. a tooltip trapped behind a modal, or a toast notification hidden behind a dropdown).
**Fix:** Define a fixed z-index scale mapped to semantic layers, and never use an ad hoc value outside it:

```css
:root {
  --z-dropdown: 10;
  --z-sticky-header: 20;
  --z-overlay: 30;
  --z-modal: 40;
  --z-toast: 50;
  --z-tooltip: 60;
}
```

## Anti-pattern: Layout shift as content loads (CLS)
**What's wrong:** Images without reserved dimensions, ads/embeds that inject after initial paint, web fonts that render at a different metric than the fallback — all causing content to jump around after the page appears loaded.
**Why it matters:** Beyond the raw Core Web Vitals score hit, layout shift is one of the most viscerally annoying experiences on the web — users lose their place, misclick a button that moved, or have their reading position jump.
**Fix:** Always reserve space before content loads: explicit `width`/`height` (or `aspect-ratio`) on images/embeds, skeleton placeholders sized to match the eventual content, and `font-display: swap` with a metrically-similar fallback font to minimize the reflow when the real font swaps in.

```html
<img src="hero.jpg" width="1200" height="600" alt="..." />
```
```css
.video-embed { aspect-ratio: 16 / 9; }
```

## Quick Checklist
- [ ] All spacing values come from a fixed scale, no ad-hoc pixel values
- [ ] Spacing is componentized (cards/forms own their internal rhythm), not redecided at every call site
- [ ] Content aligns to a shared grid/container edge across the page
- [ ] Related fields/items are grouped with proximity; groups have more space between them than within them
- [ ] Multi-line text is left-aligned (or right-aligned for RTL), never centered
- [ ] Every screen has a minimum outer margin; whitespace is deliberate, not leftover
- [ ] Layout uses fluid flex/grid with `clamp()`/`min()`/`max()`, tested across the full resize range, not just fixed breakpoints
- [ ] Containers with variable content use `min-height`, not fixed `height`
- [ ] Z-index values come from a fixed semantic scale
- [ ] Image/embed dimensions are reserved up front to prevent layout shift
