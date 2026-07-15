---
name: ui-styles-catalog
description: Catalog of ~85 named UI visual styles (glassmorphism, neumorphism, brutalism, claymorphism, etc.) with when-to-use guidance. Pair with 05-product-type-reasoning.md to pick a style for a given product type, and data/styles.csv for the raw lookup table.
---

# UI Styles Catalog

Full raw data (85 rows: style name, description, best-for, colors, effects, anti-patterns) lives in `../../data/styles.csv` — grep it for exact detail. This file gives the curated, load-bearing summary so you can pick a style without opening the CSV every time.

## How to use this file
1. Identify the product's emotional register (trust/premium/playful/technical/creative — see `00-priority-checklist.md` P3).
2. Pick **one** primary style family below. Optionally borrow one secondary accent technique (e.g. "Flat Design base + Glassmorphism for the nav bar only") — never stack three or more competing visual languages.
3. Cross-reference `05-product-type-reasoning.md` for the exact style pairing already validated for your product category.

## Core style families

**Minimalism** — Generous whitespace, restrained palette (1-2 accent colors), strong typographic hierarchy carries the design instead of decoration. Default-safe for B2B, trust-critical, and documentation products. Anti-pattern: using it as an excuse for weak hierarchy — minimalism still needs a clear focal point per screen.

**Flat Design** — No shadows/gradients/skeuomorphism; color and shape alone define hierarchy. Fast to build, ages well, pairs with any brand palette. Anti-pattern: flat interfaces with no depth cues at all can make interactive vs static elements ambiguous — always keep a hover/press state even without shadow.

**Glassmorphism** — Frosted-glass panels: `backdrop-filter: blur(10-20px)`, semi-transparent surface (`rgba` white/black at 10-20% opacity), thin 1px light border. Best for dashboards, overlays, premium consumer apps layered over colorful/photographic backgrounds. Anti-pattern: using it on a plain white background (there's nothing to blur, the effect disappears); overusing it drops contrast and hurts a11y — never use it for body text containers.

**Neumorphism (Soft UI)** — Dual soft shadows (light top-left, dark bottom-right) on a background matching the element color, producing an embossed/pressed look. Good for calm, low-stimulation contexts (wellness, meditation, healthcare). Anti-pattern: notoriously poor contrast — never use for primary CTAs or critical text; always pair with a WCAG-safe accessible variant.

**Claymorphism** — Soft, puffy, rounded 3D shapes with multi-layer shadows suggesting clay/plastic material. Friendly, playful register — great for edtech, kids' apps, habit trackers. Anti-pattern: too heavy for dense data screens; reserve for consumer-facing marketing/onboarding surfaces.

**Brutalism** — Raw, unpolished, high-contrast type-forward layouts; deliberately "undesigned" aesthetic, thick borders, harsh color blocks. Best for creative agencies, portfolios, boutique brands wanting to signal confidence/difference. Anti-pattern: never use for conversion-critical checkout/onboarding flows — the deliberate friction reads as broken to non-design audiences.

**Vibrant & Block-based** — Bold saturated colors in geometric blocks/cards, high energy. E-commerce, social, creator platforms. Anti-pattern: fatigues quickly on data-dense or long-session apps; reserve high saturation for accents, not full-screen fields.

**Liquid Glass** — Apple-style fluid, chromatic-aberration glass with more organic morphing than flat glassmorphism. Premium/luxury commerce, high-end hero moments. Anti-pattern: expensive to render well; don't apply broadly across a whole dense app — reserve for hero/flagship moments.

**Dark Mode (OLED)** — True near-black (`#000`-`#121212`) backgrounds, high-contrast accent colors, ideal for financial dashboards, dev tools, media apps, anything viewed in low-light or for long sessions. Anti-pattern: don't just invert a light palette — desaturate surface colors and boost accent saturation slightly to compensate for perceived-contrast shift in dark environments.

**3D & Hyperrealism** — Rendered 3D objects/scenes, realistic materials and lighting. Gaming, automotive, product configurators. Anti-pattern: heavy performance cost — always have a static-image or reduced-motion fallback for low-end devices.

**Motion-Driven** — Design where scroll-triggered/parallax animation is the primary storytelling device rather than decoration. Portfolios, agencies, storytelling landing pages. Anti-pattern: without a `prefers-reduced-motion` fallback this actively harms vestibular-sensitive users — always provide the static equivalent.

**Bento Box Grid** — Apple-style modular grid of variously-sized cards showing feature highlights at a glance. Great for feature showcases and product pages. Anti-pattern: don't force uneven content into equal-sized cells — let card size follow content weight.

**Swiss Modernism 2.0** — Grid-strict editorial layout, strong type hierarchy, restrained color, generous margins. Editorial/media/magazine products. Anti-pattern: needs genuinely excellent typography to work — a weak type system makes this look merely empty.

**Accessible & Ethical** — Style whose *primary* constraint is WCAG AAA compliance: high contrast, large type, no motion-dependent information, clear focus states. Government, healthcare, senior-focused, legal products should treat this as non-negotiable, not optional flavor.

**Cyberpunk UI / HUD-Sci-Fi FUI** — Neon accents on near-black, angular geometric framing, glitch/scan-line accents. Gaming, Web3, security/ops dashboards wanting a "high-tech" register. Anti-pattern: readability suffers fast — keep body text on a neutral high-contrast color even inside a neon shell.

**Organic Biophilic** — Natural textures, earth tones, rounded organic shapes, nature imagery. Sustainability, agriculture, wellness brands. Anti-pattern: avoid combining with harsh neon accents — keep the palette coherent with the natural theme throughout.

**Aurora UI** — Soft multi-color gradient meshes (aurora-borealis-like) as backgrounds behind minimal foreground UI. Travel, AI products, weather apps. Anti-pattern: gradients need to stay behind content, not under text — verify contrast at every gradient stop, not just the average.

**AI-Native UI** — Neutral base with a signature accent (commonly indigo/purple, `#6366F1`-family) for AI-specific moments: streaming text, "thinking" states, generative previews. Conversational and generative-AI products. Anti-pattern: avoid clichéd purple/pink gradient overload — many product-type rows in `data/ui-reasoning.csv` explicitly flag "AI purple/pink gradients" as an anti-pattern *outside* AI-native products; don't apply the AI aesthetic to non-AI products just because it looks modern.

**Spatial UI (visionOS-style)** — Frosted glass depth layers, gaze/hover affordances, environment-aware translucency for spatial computing/AR contexts.

**Holographic/HUD** — Deep-space/telemetry aesthetic: thin glowing line work, monospace data readouts, dark backgrounds. Aerospace, drone, quantum-computing dashboards.

**Gen Z Chaos / Expressive Anti-Design** — Deliberately chaotic type mixing, off-grid placement, maximalist color for youth-culture/creative platforms. High risk — validate against the actual target audience before using; reads as broken to a mainstream/enterprise audience.

## Quick anti-pattern list (cross-style)

- Never mix neumorphism with high-saturation vibrant colors — the low-contrast shadow language conflicts with vibrant hue signaling.
- Never apply heavy skeuomorphism as a "trendy" throwback without a specific brand reason — it consistently tests poorly for usability.
- Never default to dark mode for content-heavy reading apps without offering light mode — reading comfort varies by ambient light and user preference.
- Never let a decorative style choice (glass blur, 3D render, motion) reduce text contrast below WCAG AA — decoration never outranks legibility.
