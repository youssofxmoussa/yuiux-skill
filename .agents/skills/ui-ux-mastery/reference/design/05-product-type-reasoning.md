---
name: product-type-reasoning
description: Decision table mapping 101+ product types (SaaS, fintech, healthcare, gaming, marketplace, etc.) to recommended UI pattern, style pairing, color mood, typography mood, key motion effects, conditional decision rules, and anti-patterns to avoid. Raw table in data/ui-reasoning.csv and data/products.csv.
---

# Product-Type Design Reasoning

Full raw data lives in two complementary files:
- `../../data/ui-reasoning.csv` (101 rows: UI category, recommended pattern, style priority, color mood, typography mood, key effects, conditional decision rules as JSON, anti-patterns, severity)
- `../../data/products.csv` (193 rows: product type, keywords, primary/secondary style recommendation, landing page pattern, dashboard style, color palette focus, key considerations)

Grep either by product-type keyword to get the full validated recommendation before designing from scratch. This is the single most efficient way to skip weeks of trial-and-error style exploration — treat it as the starting hypothesis, then adjust only where the specific brand has a stated reason to diverge.

## How to use this table

For any new product, look up the closest matching product type and read off five decisions simultaneously (pattern, style, color mood, typography mood, motion) — these were validated as a *coherent set*, not independently, so don't cherry-pick just the color from one row and the style from another.

## Cross-cutting patterns worth internalizing (apply even without an exact product-type match)

**Trust-critical verticals converge on the same anti-pattern list.** Government, fintech/banking, legal, insurance, senior care, healthcare, logistics, construction, and agriculture rows *all* independently flag "AI purple/pink gradients" and "playful design" as anti-patterns, and converge on Minimalism + Accessible & Ethical + Trust & Authority as the style family, professional blue/navy as color mood, and high-contrast readable typography. If a product sits in this cluster, resist the urge to make it "more modern" with trendy gradients — it actively works against the product's credibility signal.

**Data-dense/real-time products want dark mode + status color, not light mode.** Financial dashboards, analytics dashboards, smart-home/IoT, cybersecurity, and drone-fleet-manager rows all specify Dark Mode (OLED) as primary or secondary style, paired with real-time number/pulse animations and strict red/green (or tactical green/alert red) semantic coloring. Light-mode-by-default is explicitly listed as the anti-pattern in every one of these rows.

**Playful/consumer-facing categories want soft, tactile styles with spring motion.** Educational apps, habit trackers, recipe apps, children's/childcare products, and pet tech converge on Claymorphism or Vibrant & Block-based with soft multi-layer shadows and spring/bounce easing (200-250ms soft press) — flat/minimal/dark treatments are flagged as anti-patterns here because they read as cold for this audience.

**Premium/luxury categories want slow, deliberate motion — never fast or bouncy.** Luxury e-commerce, luxury/premium brand, hospitality, and photography studio rows specify Liquid Glass/Glassmorphism with 400-600ms transitions and chromatic-aberration/fluid effects; "vibrant & block-based" and fast animations are explicitly anti-patterns for this cluster — speed reads as cheap in a luxury context.

**AI-native products get their own aesthetic — but it's specific to them.** AI/Chatbot Platform, biohacking/longevity, and AI-driven dynamic landing rows use AI-Native UI or neutral-plus-purple-accent (`#6366F1`-family) with streaming-text/typing-indicator motion. This aesthetic should stay scoped to genuinely AI-forward product moments (chat interfaces, generative previews) — see `02-color-palettes.md` for the explicit warning against exporting this look to non-AI verticals.

**Accessibility-first is a P0 style constraint, not decoration, for an identifiable cluster.** Government/public service, senior care/elderly, and to a lesser extent healthcare/medical clinic rows specify WCAG AAA and large type (16-18px+) as a `must_have` in their decision rules, not merely a suggestion — treat these as non-negotiable requirements when the product matches this cluster, on par with the P0 accessibility rules in `00-priority-checklist.md`.

## Reading the "Decision_Rules" JSON column

Rows in `ui-reasoning.csv` include a small JSON object of conditional rules, e.g. `{"if_luxury": "switch-to-liquid-glass", "if_conversion_focused": "add-urgency-colors"}`. Treat these as branch points: apply the base recommendation, then check whether any condition matches your specific brief (e.g. "is this the luxury tier of an e-commerce brand?") and apply the override if so.

## Worked example

**Task: build a B2B fintech invoicing dashboard.**
1. Look up "Fintech/Crypto" and "Financial Dashboard" in `ui-reasoning.csv` → both recommend Minimalism/Trust & Authority style family, navy/trust-blue + gold color mood, professional typography, smooth number-count animations, and explicitly flag "playful design," "unclear fees," and "AI purple/pink gradients" as anti-patterns.
2. Cross-reference `data/colors.csv` for "Fintech/Crypto" → gold-forward primary (`#F59E0B`) on dark navy background, or "Financial Dashboard" → dark `#0F172A` background with green/red semantic alerts — pick whichever matches whether this is a marketing-facing or operational-dashboard surface.
3. Cross-reference `03-typography-pairings.md` for a professional/trustworthy register (Modern Professional or Minimal Swiss pairing).
4. Apply `06-motion-tokens.md` Subtle tier throughout — this cluster wants restrained, fast (150-300ms) motion, never Complex-tier elastic/bounce effects.
