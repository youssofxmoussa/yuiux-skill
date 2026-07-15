---
name: color-palettes
description: Semantic color-token system and 190+ product-type color palettes (primary/secondary/accent/background/card/muted/border/destructive/ring, all WCAG-checked). Raw table in data/colors.csv.
---

# Color Palettes

Full raw data (193 rows, one per product type: `Primary, On Primary, Secondary, On Secondary, Accent, On Accent, Background, Foreground, Card, Card Foreground, Muted, Muted Foreground, Border, Destructive, On Destructive, Ring` — plus a `Notes` column showing any hue adjusted for WCAG contrast) lives in `../../data/colors.csv`. Grep it by product type keyword (e.g. `grep -i "fintech" colors.csv`) to get a ready-to-use token set.

## The token model (use this structure regardless of the specific hues chosen)

Every palette in the data file follows the same semantic-token shape — adopt this as your project's design-token schema even when you invent a new palette from scratch:

| Token | Purpose |
|---|---|
| `primary` / `on-primary` | Brand color and the text/icon color guaranteed readable on top of it |
| `secondary` / `on-secondary` | Supporting brand color for secondary actions |
| `accent` / `on-accent` | High-attention color for a single CTA/highlight role — not a second brand color |
| `background` / `foreground` | Page-level base and default text color |
| `card` / `card-foreground` | Surface color for elevated content and its text |
| `muted` / `muted-foreground` | De-emphasized backgrounds/text (secondary info, disabled states) |
| `border` | Dividers, input outlines, card edges |
| `destructive` / `on-destructive` | Delete/error actions and their contrasting text |
| `ring` | Focus-ring color (usually equals `primary`) |

Never invent one-off colors outside this token set per screen — every color used in the product should map to one of these roles so theming (including dark mode) stays consistent.

## Selection heuristic

1. Look up the product type (or nearest analog) in `data/colors.csv`.
2. If no close match exists, start from the semantic register in `05-product-type-reasoning.md` (e.g. "Trust & Authority" → navy/blue primary, minimal accent) and build the token set by hand, verifying contrast with the rules below.
3. Note any `[Accent adjusted from #X for WCAG 3:1]` annotations in the data — these are colors the source data already corrected for non-text contrast (icons, borders, focus rings). Preserve that reasoning if you further adjust a palette.

## Recurring semantic-color conventions across the dataset

- **Trust-critical products** (fintech, banking, legal, government, insurance) converge on navy/blue primaries with minimal, restrained accent colors — never playful or high-saturation.
- **Destructive** is almost universally a red in the `#DC2626`-`#EF4444` range regardless of overall brand palette — don't reskin destructive actions to match brand color, keep the universal red-for-danger convention.
- **Dark-mode-first products** (financial/analytics dashboards, gaming, dev tools, media/OTT) use near-black backgrounds (`#0F172A`-`#0F0F23`) with desaturated surface tones and a single saturated accent (commonly green for "positive"/status, or the brand hue) — never just invert a light palette's lightness values.
- **Playful/consumer** products (education, pet tech, creator platforms) use higher-saturation primaries paired with warm accent colors and light, airy backgrounds.
- **AI-native products** commonly use purple/indigo (`#7C3AED`, `#6366F1`) as primary — but per `data/ui-reasoning.csv`, this same purple/pink gradient treatment is explicitly flagged as an **anti-pattern** when applied to non-AI product types (fintech, government, healthcare, legal, logistics, construction, senior care) — don't borrow the "AI aesthetic" outside genuinely AI-forward products.

## Contrast rules (apply to every custom palette)

- Body text on `background`/`card`: ≥4.5:1 contrast.
- Large text (≥24px or ≥18.5px bold) and UI icons/borders: ≥3:1.
- Verify `on-*` pairings against the *exact* hex they'll render on, not a close approximation — small hex shifts can silently drop a pairing below AA.
- When a brand hue itself fails contrast against white/black text, don't force it — adjust the hue's lightness/saturation slightly (as the dataset does, noted via `[Accent adjusted from ... ]`) rather than accepting a failing pair.

## Dark mode as a first-class palette, not an inversion

- Build dark-mode `background`/`card`/`muted` values independently — desaturate slightly and avoid true `#000` for card surfaces (use `#0E1223`-`#1E293B` range) so elevation is still perceivable.
- Accent/status colors (success green, danger red) typically need a slight saturation *increase* in dark mode to read at the same perceptual intensity against a dark background.
