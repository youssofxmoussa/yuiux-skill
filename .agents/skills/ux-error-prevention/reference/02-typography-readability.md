---
name: typography-readability
description: Type scale, hierarchy, line length, legibility, and font pairing errors and fixes for web UI.
---

# Typography & Readability

Typography is the single highest-leverage lever for making a UI feel premium versus amateur — it costs nothing to build correctly and is one of the most common places "cheap-looking" software goes wrong.

## Anti-pattern: No type scale, ad-hoc font sizes everywhere
**What's wrong:** Every heading, label, and body text uses a hand-picked `font-size` (14px here, 15px there, 13px somewhere else) with no relationship between them.
**Why it matters:** Inconsistent sizing reads as visual noise even if no single size is "wrong." Users unconsciously use size differences to infer hierarchy; random sizes give false signals.
**Fix:** Define a fixed type scale (e.g. a modular scale with ratio ~1.25–1.333) and use only those steps everywhere:

```css
:root {
  --text-xs: 0.75rem;   /* 12px */
  --text-sm: 0.875rem;  /* 14px */
  --text-base: 1rem;    /* 16px */
  --text-lg: 1.25rem;   /* 20px */
  --text-xl: 1.5rem;    /* 24px */
  --text-2xl: 2rem;     /* 32px */
  --text-3xl: 2.5rem;   /* 40px */
}
```
Every text element in the app maps to exactly one of these tokens. If a designer/agent wants a size not on the scale, that's a signal to reconsider the hierarchy, not to add a one-off value.

## Anti-pattern: Body text below 16px on the web
**What's wrong:** Using 12–13px as the default body/paragraph text size to fit more content.
**Why it matters:** Below ~16px, reading speed and comprehension drop measurably, and mobile browsers historically auto-zoom form inputs under 16px, causing a jarring zoom-in on focus.
**Fix:** 16px (`1rem`) is the floor for primary reading text on desktop and mobile. Use smaller sizes (`--text-sm`, `--text-xs`) only for secondary metadata (timestamps, captions, helper text), never for primary content or form input text.

## Anti-pattern: Lines too long or too short to read comfortably
**What's wrong:** Paragraph text that stretches the full width of a wide viewport (100+ characters per line), or is squeezed into a narrow column that wraps every 3 words.
**Why it matters:** Optimal line length for sustained reading is roughly 45–75 characters (~66 is the oft-cited ideal). Longer lines make it hard for the eye to track back to the start of the next line; shorter lines break reading rhythm with excessive wraps.
**Fix:** Constrain text blocks with `max-width` in `ch` units, which scale with the font:

```css
.prose {
  max-width: 65ch;
}
```

## Anti-pattern: Insufficient line-height (leading)
**What's wrong:** `line-height: 1` or unset (browser default ~1.2) on body paragraphs, making lines feel cramped and hard to scan.
**Why it matters:** Tight leading increases the chance the eye jumps to the wrong line, especially in longer paragraphs.
**Fix:** Body text: `line-height: 1.5–1.6`. Headings can be tighter (`1.1–1.3`) since they're short and the relationship between wrapped lines matters less. Never go below `1.4` for anything meant to be read continuously.

## Anti-pattern: Justified text on the web
**What's wrong:** `text-align: justify` on paragraphs.
**Why it matters:** Without proper hyphenation (which browsers do inconsistently), justification creates uneven word spacing and "rivers" of white space that hurt readability.
**Fix:** Use left-aligned (or right-aligned for RTL) text. Never justify body copy on the web.

## Anti-pattern: All-caps for anything but short labels
**What's wrong:** Long headings, buttons with multi-word phrases, or body text rendered in `text-transform: uppercase`.
**Why it matters:** Uppercase text removes the ascender/descender shape cues (the silhouette of "Playground" vs "PLAYGROUND") that speed up word recognition, slowing reading measurably as length increases.
**Fix:** Reserve uppercase for very short labels (tab names, section eyebrows, badges — 1–2 words) and always pair with slightly increased `letter-spacing` (0.02–0.08em) to compensate for the tighter apparent spacing uppercase creates.

## Anti-pattern: No clear hierarchy between heading levels
**What's wrong:** `<h1>`, `<h2>`, and `<h3>` all rendered at visually similar sizes/weights, or hierarchy communicated only by color with no size/weight difference.
**Why it matters:** Users scan pages by hierarchy before they read; flat hierarchy forces them to read everything to find what matters.
**Fix:** Combine at least two of {size, weight, color, spacing} to differentiate each heading level clearly, and keep the semantic HTML heading order intact (`h1` → `h2` → `h3`, never skipping levels) for both visual and accessibility reasons.

## Anti-pattern: Font weight used as the only differentiator, with too many weights
**What's wrong:** Mixing 5+ font weights (300, 400, 500, 600, 700, 800) across a single page with no system to it.
**Why it matters:** Excess weight variety looks unintentional and increases page weight (more font files to load) for no visual gain.
**Fix:** Pick 2–3 weights total for the whole app (e.g. Regular 400, Medium 500, Semibold 600) and assign each a fixed role (body = 400, emphasis/labels = 500, headings = 600). Never use Bold (700+) and Semibold interchangeably for the same role.

## Anti-pattern: More than two typefaces
**What's wrong:** A display font for headings, a different font for body, a third "friendly" font for buttons or numbers.
**Why it matters:** Each additional typeface is another visual system the user has to unconsciously reconcile, and adds load-time cost. Most polished products use exactly one typeface (varying weight/size for hierarchy) or at most two (a display face for large headings + a workhorse face for everything else).
**Fix:** Default to a single, high-quality variable font family covering the whole app. Only add a second face if there's a specific brand reason, and never use a third.

## Anti-pattern: System font fallback stack missing or wrong
**What's wrong:** `font-family: "CustomFont"` with no fallback, so if the web font fails to load (slow network, blocked request) text is invisible or falls back to Times New Roman.
**Why it matters:** Users see either blank text (`font-display: block` default in older setups) or a jarring flash of unstyled text.
**Fix:** Always declare a realistic fallback stack and set `font-display: swap` (or `optional` if layout stability matters more than seeing the custom font at all):

```css
@font-face {
  font-family: "Inter";
  src: url("/fonts/inter.woff2") format("woff2");
  font-display: swap;
}
body {
  font-family: "Inter", -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif;
}
```

## Anti-pattern: Numeric misalignment in tables/dashboards
**What's wrong:** Numbers in a table column don't align on the decimal/ones place because the font uses proportional (variable-width) digits by default.
**Why it matters:** Misaligned numbers are much harder to scan and compare at a glance — exactly the task tables/dashboards exist for.
**Fix:** Use tabular figures for any column of numbers:

```css
.numeric-cell {
  font-variant-numeric: tabular-nums;
}
```

## Anti-pattern: Low text contrast used for "elegance"
**What's wrong:** Light gray text (#999 or lighter) on a white background used for body copy because it "looks cleaner."
**Why it matters:** This routinely fails WCAG AA contrast (see `03-color-contrast-theming.md`) and is genuinely harder to read for a large share of users, not just those with diagnosed vision impairment — screen glare, aging eyes, and small displays all compound the problem.
**Fix:** Reserve very light gray for true tertiary metadata in small sizes; primary and secondary text should hit at least a 4.5:1 contrast ratio against its background.

## Anti-pattern: Truncating text without any way to see the full content
**What's wrong:** `text-overflow: ellipsis` on a table cell, tag, or title with no tooltip, expand affordance, or accessible full-text alternative.
**Why it matters:** The user knows something is being hidden but has no way to retrieve it — worse than not showing hierarchy at all.
**Fix:** Pair truncation with a native `title` attribute (or a proper tooltip component) so hover/focus reveals the full text, and make sure the full text is still available to screen readers (don't truncate the accessible name, only the visual rendering).

## Quick Checklist
- [ ] All text sizes come from a fixed scale, no ad-hoc `font-size` values
- [ ] Body text ≥16px; line-height 1.5–1.6
- [ ] Paragraph max-width ~65ch
- [ ] No justified text
- [ ] Uppercase reserved for short labels with added letter-spacing
- [ ] Heading hierarchy uses ≥2 combined cues (size+weight+color), semantic order preserved
- [ ] ≤3 font weights, ≤2 typefaces total
- [ ] Fallback font stack + `font-display: swap` declared
- [ ] Tabular numerals in any numeric table column
- [ ] Truncated text has an accessible/hoverable full-text fallback
