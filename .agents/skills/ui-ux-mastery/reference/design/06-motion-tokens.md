---
name: motion-tokens
description: Motion design tokens — durations, easings, and copy-paste GSAP snippets for hover, scroll reveal, stagger, page transitions, parallax, and loading states, at Subtle/Standard/Complex intensity tiers. Raw table in data/motion.csv.
---

# Motion Tokens

Full raw data (16 rows: category, intensity tier, trigger, duration, easing, GSAP snippet, framework notes, do/don't, performance notes) lives in `../../data/motion.csv`. This file summarizes the categories and the cross-cutting rules; grep the CSV for the exact snippet when implementing.

## Intensity tiers

Every motion category comes in three tiers — pick the tier that matches the product's overall energy (see `00-priority-checklist.md` P3 style selection), not per-component:

- **Subtle** — trust-critical, dense, professional products (fintech, B2B, dashboards). Small displacement (≤4px), fast (150-350ms), `power1`/`power2` easing.
- **Standard** — general consumer SaaS/marketing default. Moderate displacement, 200-600ms, `power2`/`back.out` easing.
- **Complex** — flagship/portfolio/marketing moments only, never in dense repeated UI. Larger displacement, 300-800ms, elastic/expo easing, may include pinning or shared-element transitions.

## Categories

1. **Hover Micro-interaction** — feedback on pointer hover for buttons/cards. Subtle = 1-2px lift/opacity shift; Standard = scale+shadow lift; Complex = magnetic cursor-follow (max 1-2 focal elements per screen).
2. **Scroll Reveal** — content fades/slides in as it enters viewport via ScrollTrigger. Subtle = small fade+8-16px shift; Standard = staggered section reveal; Complex = pinned scrub-driven scrollytelling (max 1-2 pinned sections per page — excessive pinning fights native scroll feel, especially on mobile).
3. **Stagger List** — sequential entrance of list/grid items. Per-item delay should shrink as list length grows (0.02-0.04s for lists >10 items) so total reveal time stays under ~1s. Never use `back.out` overshoot on dense data tables.
4. **Page Transition** — route-change animation. Exit must always resolve faster than entrance (asymmetric timing) so back/forward navigation feels responsive. Never block navigation on the animation finishing.
5. **Parallax Scroll** — background/decorative depth layering tied to scroll position. Apply to decorative/background layers only — never to body text or interactive controls (reading comfort + motion-sickness risk).
6. **Loading/Skeleton** — shimmer/pulse for async waits. Kill the loop the instant real content mounts; never run elaborate loaders for sub-300ms waits (they flash and feel worse than no indicator at all).

## Universal rules (apply regardless of tier)

- Animate `transform`/`opacity` only — anything touching layout (`width`, `height`, `top`, `left`, `margin`) forces reflow and causes jank; use `will-change: transform` sparingly and remove it once the animation settles.
- Every looping animation (`repeat: -1`) must be explicitly killed on unmount/route change or it leaks in SPA navigation.
- Respect `prefers-reduced-motion`: pause non-essential motion (parallax, autoplay, decorative loops); keep only state-communicating motion (loading spinners, focus rings) and cut its amplitude.
- Register GSAP plugins (ScrollTrigger, Flip, SplitText) once at app bootstrap, not per-component — and confirm plugin licensing before using paid-tier plugins like SplitText/MorphSVG.
- In React, prefer `useGSAP()` (from `@gsap/react`) scoped to a container ref for automatic cleanup on unmount rather than manual `useEffect` + manual `.kill()` bookkeeping.
- Cap total perceivable "wait" for any coordinated stagger/reveal sequence around ~1-1.5s; beyond that the UI feels slow even though each individual tween is fast.
