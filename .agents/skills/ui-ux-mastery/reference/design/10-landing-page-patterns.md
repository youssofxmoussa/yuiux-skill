---
name: landing-page-patterns
description: 34 named landing-page/section-order patterns (hero+features, funnel, pricing, waitlist, bento showcase, etc.) with CTA placement, color strategy, and conversion notes. Raw table in data/landing.csv.
---

# Landing Page Patterns

Full raw data (34 rows: pattern name, section order, CTA placement, color strategy, recommended effects, conversion optimization notes) lives in `../../data/landing.csv`. This file gives the selection heuristic; grep the CSV for exact section-order and effect specs per pattern.

## Choosing a pattern

Match the pattern to the primary conversion goal, not to what looks impressive:

- **Direct-response / single CTA** → Minimal Single Column or Hero-Centric Design. Large single CTA, minimal chrome, mobile-first — best for early-stage products with one clear action.
- **Feature-heavy SaaS** → Hero + Features + CTA, Feature-Rich Showcase, or Bento Grid Showcase (Apple-style modular cards) for scannable high information density without clutter.
- **Trust-building / high consideration purchase** → Hero + Testimonials + CTA, or Trust & Authority + Conversion for enterprise — social proof or credibility signals *before* the ask, never after.
- **Pricing-led** → Pricing Page + CTA or Pricing-Focused Landing: highlight the recommended tier visually, always show the annual discount, and use FAQ to pre-empt objections rather than leaving them for support.
- **Pre-launch** → Waitlist/Coming Soon: countdown + email capture above the fold, show live waitlist count for social proof, and offer a referral incentive.
- **Narrative/portfolio/agency** → Scroll-Triggered Storytelling or Horizontal Scroll Journey — use sparingly, simplify heavily on mobile, and always keep primary navigation visible even during scroll-jacking sequences.
- **Comparison-driven** → Comparison Table + CTA: highlight your product's row/column, keep the comparison factual, and put pricing in the table if it's favorable.
- **Product demo-led** → Product Demo + Features or Interactive 3D Configurator: autoplay the demo muted, provide captions, and make sure the CTA is reachable without requiring the demo to finish.

## CTA placement rules (apply across all patterns)

- One primary CTA style per page — repeating the *same* CTA at hero, mid-page, and footer beats introducing several different competing CTAs.
- Contrast ratio between CTA and its background should be ≥7:1 where possible — this is called out repeatedly across pattern rows as a conversion lever, not just an accessibility nicety.
- Sticky nav CTA for any page longer than ~2 viewport heights, so the action is always one tap away regardless of scroll position.

## Section-order defaults

Most converting patterns share a spine: **Hero (value prop) → Proof (features/testimonials/logos) → Objection handling (FAQ/comparison/pricing) → CTA**. Deviate from this only when the pattern specifically calls for it (e.g. Funnel patterns intentionally break proof into progressive steps; Lead Magnet patterns move the form up before deep feature explanation).

## Common pitfalls across all patterns

- Video-first heroes need muted autoplay, captions, and heavy compression — uncompressed hero video is one of the most common landing-page performance killers.
- Countdown timers and urgency indicators must reflect *real* scarcity — fabricated urgency erodes trust the moment a user reloads the page and sees the same countdown.
- Testimonial/social-proof sections need a photo + name + role at minimum; anonymous or generic quotes carry little persuasive weight.
- Multi-step funnels must show progress (step indicator) and keep each step's cognitive load low — cramming multiple decisions into one step defeats the purpose of funneling.
