---
name: priority-checklist
description: Ranked (P0-P10) cross-platform UI/UX rule checklist — accessibility, touch, performance, style selection, layout, typography/color, animation, forms, navigation, charts. Read this first when starting any UI build or review.
---

# Priority Checklist (P0 = never skip → P10 = polish)

This is the single highest-leverage file in the skill. When time is short, apply these in order — each tier assumes the tiers above it are already satisfied. Every rule here is platform-agnostic: it applies whether you are shipping React+Tailwind on Replit, a Lovable-generated app, or a Claude Code project in any stack.

## P0 — Accessibility (never ship without this)

- Every interactive element has a visible focus indicator (`:focus-visible`, 2px min, 3:1 contrast against background) — never `outline: none` without a replacement.
- Every icon-only control has an accessible name (`aria-label`, `accessibilityLabel`, or equivalent). No silent icon buttons.
- Text contrast: body text ≥ 4.5:1, large text (≥18px bold or ≥24px regular) ≥ 3:1. Verify against the *actual* rendered background, not the design token in isolation.
- All functionality is reachable by keyboard alone: tab order follows visual order, no keyboard traps, `Escape` closes overlays.
- Form inputs have a programmatically associated `<label>` — placeholder text is never a substitute for a label.
- Respect `prefers-reduced-motion`: disable parallax, autoplay, and large-amplitude motion when set.
- Semantic HTML/roles first (`<button>`, `<nav>`, heading levels in order) — ARIA is a patch for what semantic markup can't express, not the default tool.
- Color is never the *only* signal for state (error/success/selected) — pair with icon, text, or pattern.

## P1 — Touch & Interaction

- Minimum touch target: 44×44pt (iOS) / 48×48dp (Android) / 44×44px (web) even if the visible icon is smaller — pad the hit area.
- Minimum 8px gap between adjacent tappable targets to prevent mis-taps.
- Never trap a native gesture (edge-swipe-back, vertical scroll) inside a custom gesture handler.
- Give every destructive action (delete, remove, cancel subscription) a confirmation step or an undo window — never a single silent tap.
- Loading/disabled states must visibly block double-submission on tap-happy users (disable button + show spinner on the same interaction, don't wait for a re-render).

## P2 — Performance & Perceived Performance

- First contentful paint budget: design for the content the user cares about to render before anything decorative.
- Any operation >300ms shows a loading indicator; anything >100ms should at minimum register a state change (pressed/disabled).
- Long lists (>50 items) are virtualized (`FlatList`/`react-window`/windowing), never rendered in full via `.map()` into an unconstrained scroll view.
- Images are served at the display resolution, lazy-loaded below the fold, and use modern formats (WebP/AVIF) with explicit width/height to avoid layout shift.
- Debounce/throttle high-frequency handlers: scroll, resize, search-as-you-type (250-400ms).
- Skeleton screens > spinners for content-shaped loading (cards, lists, avatars) — they reduce perceived wait time and prevent layout jump on load.

## P3 — Style Selection (pick ONE cohesive direction, don't blend five)

Match the UI style to the product's emotional register before touching color or type. See `01-ui-styles-catalog.md` and `05-product-type-reasoning.md` for the full decision table (100+ product types mapped to style/color/motion). Quick defaults:

| Product register | Default style pairing |
|---|---|
| Trust-critical (fintech, healthcare, legal, gov) | Minimalism + Accessible & Ethical, high contrast, no gimmicks |
| Consumer SaaS / dashboard | Flat Design or Glassmorphism + subtle motion |
| Premium/luxury commerce | Liquid Glass or Glassmorphism, slow (400-600ms) transitions |
| Playful/consumer/edtech | Claymorphism or Vibrant & Block-based, spring motion |
| Dev tools / technical | Dark Mode (OLED) + Minimalism, monospace accents |
| Creative/agency/portfolio | Brutalism or Motion-Driven, high artistic freedom |

Never mix more than two "load-bearing" style languages on one screen (e.g. glassmorphism cards inside a neumorphic shell reads as visually confused).

## P4 — Layout & Responsive

- Design mobile-first; verify the layout holds at 320px width before scaling up.
- Use a real spacing scale (4/8px base grid) — no ad hoc pixel values.
- Content max-width on large screens (~1200-1280px for text-heavy layouts) — never let body text stretch edge-to-edge on a 27" monitor.
- Every breakpoint transition must be tested, not assumed: tablet portrait (768px) and small laptop (1024px) are the most commonly broken.
- Sticky/fixed elements (headers, CTAs) must not cover content on short viewports (test at 600px height, not just narrow width).

## P5 — Typography & Color

- Max 2 typefaces per product (one display/heading, one body — or one family across weights). See `03-typography-pairings.md` for 57 vetted pairings.
- Type scale uses a consistent ratio (1.2-1.333 for UI, larger for editorial/marketing) — not arbitrary sizes per screen.
- Body text minimum 16px on web / 16-17pt on mobile to avoid forced zoom and reading fatigue.
- Every color in the palette has a defined *semantic* role (primary, success, warning, danger, neutral scale) before it's used — never pick colors ad hoc per screen. See `02-color-palettes.md`.
- Dark mode is a deliberate palette (adjusted lightness/saturation), not light-mode colors with values inverted.

## P6 — Animation & Motion

- Micro-interactions (hover, press): 150-250ms, `ease-out`.
- Entrances/reveals: 250-500ms, `ease-out`; exits are faster than entrances (exit ≤ 70% of entrance duration) so the UI never feels laggy leaving.
- Stagger lists in increments of 20-80ms per item, cap total stagger under ~600ms for the last item.
- Animate `transform`/`opacity` only where possible — never animate `width`/`height`/`top`/`left` (forces layout, causes jank).
- Full motion reference with exact durations, easings, and copy-paste GSAP snippets by category (hover, scroll reveal, stagger, page transition, parallax, loading) lives in `06-motion-tokens.md`.

## P7 — Forms & Feedback

- Validate on blur/submit, not on every keystroke (keystroke-level validation is only for real-time strength meters/availability checks).
- Every error message is specific and actionable ("Password needs a number" not "Invalid input") and appears next to the field, not only in a banner.
- Success is confirmed, not assumed silent — toast, inline check, or state change every time.
- Multi-step forms show progress and preserve entered data on back-navigation.
- Never disable the submit button pre-emptively to "prevent errors" — let the user submit and show validation; hard-disabling with no visible reason reads as broken.

## P8 — Navigation Patterns

- Primary navigation has 3-7 top-level items; more than that needs grouping or a "More" pattern.
- Current location is always visible (active nav state, breadcrumb, or page title matching the nav label clicked).
- Back behavior is predictable: browser/hardware back never exits the app unexpectedly or loses in-progress form state.
- Modals and sheets always have an explicit close affordance in addition to backdrop-click/swipe-down.

## P9 — Charts & Data Density

- Match chart type to data shape, not visual preference — see `07-charts-dataviz.md` for a full data-type → chart-type decision table with accessibility grades. Pie charts are graded C (colorblind-unsafe, imprecise past 5-6 slices); prefer bar/stacked-bar by default.
- Every chart needs a non-visual fallback (data table, text summary) for anything graded B or below on accessibility.
- Dense dashboards group related metrics, use consistent color-to-meaning mapping across all charts on the same screen, and never rely on a legend the user must cross-reference from memory across scrolling.

## P10 — Craft & Polish (the 20% that separates "functional" from "excellent")

- Empty states are designed, not blank — they explain what will appear and how to create the first item.
- Icons are paired with text labels except for a small set of universally understood glyphs (back arrow, close X, search).
- Consistent corner radius, shadow depth, and border treatment across all components of the same elevation tier.
- Copy tone is consistent (see `14-microcopy-content-strategy.md` in `reference/errors/`) — no mixing formal and casual voice across the same product.
- The interface still looks intentional with JavaScript-disabled/slow-network fallback states, not just in the happy path screenshot.
