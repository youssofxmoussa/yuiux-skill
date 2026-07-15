---
name: typography-pairings
description: 57 vetted heading/body Google Fonts pairings by mood and use case (Classic Elegant, Modern Professional, Tech Startup, Developer Mono, etc.), with ready CSS/Tailwind config. Raw table in data/typography.csv; full Google Fonts metadata in data/google-fonts.csv.
---

# Typography Pairings

Full raw data (57 rows: pairing name, category, heading font, body font, mood keywords, best-for, Google Fonts URL, CSS `@import`, Tailwind `fontFamily` config) lives in `../../data/typography.csv`. A separate `../../data/google-fonts.csv` lists the full Google Fonts catalog with classification/keywords/popularity if you need to look up or substitute an individual typeface. Grep both by mood keyword (e.g. `grep -i "playful"`) to find a match fast.

## How to pick a pairing

1. Start from the product's emotional register (same register used for style selection in `00-priority-checklist.md` P3 and colors in `02-color-palettes.md` — keep type, color, and style consistent with one register, not three different ones).
2. Pick a **named pairing** wholesale from the CSV rather than combining fonts ad hoc — each row is a pre-validated heading/body combination with contrast in weight/character already balanced.
3. Copy the row's Tailwind config or CSS import directly; don't hand-roll `@import` URLs (weight subsets matter for load performance — the data file already selects the minimal weight set needed).

## Representative pairings by register

| Register | Pairing | Heading / Body |
|---|---|---|
| Luxury / editorial / high-end e-commerce | Classic Elegant | Playfair Display / Inter |
| SaaS / corporate / professional services | Modern Professional | Poppins / Open Sans |
| Tech startup / AI / dev tools | Tech Startup | Space Grotesk / DM Sans |
| Publishing / literary / news | Editorial Classic | Cormorant Garamond / Libre Baskerville |
| Dashboards / admin / enterprise apps | Minimal Swiss | Inter / Inter (single family, weight variation only) |
| Kids / edtech / gaming / entertainment | Playful Creative | Fredoka / Nunito |
| Marketing / agency / event / sports | Bold Statement | Bebas Neue (display only, headlines) / Source Sans 3 |
| Wellness / health / meditation / organic | Wellness Calm | Lora / Raleway |
| Developer tools / CLI / documentation | Developer Mono | JetBrains Mono / IBM Plex Sans |

(37 more rows in the CSV cover additional registers — brutalist/experimental, luxury fintech, biotech/scientific, gaming/esports, children's/kawaii, government/civic, and more. Grep for the specific mood you need.)

## Universal typography rules (apply regardless of chosen pairing)

- Max 2 typefaces per product — one heading/display family, one body family (or a single variable family used across weights, as in Minimal Swiss).
- Display-only fonts (e.g. Bebas Neue, all-caps display faces) are for headlines only — never set body copy or UI labels in them; they fail readability at small sizes.
- Body text minimum 16px on web, 16-17pt on mobile — never go smaller for primary reading content, even to fit a design mock.
- Use a consistent modular type scale (ratio 1.2-1.333 for UI-dense products, larger ratios like 1.414-1.618 for editorial/marketing) rather than picking sizes per screen ad hoc.
- Load fonts with `font-display: swap` (already baked into the CSS imports in the data file) to avoid invisible-text-until-loaded (FOIT).
- Limit weight variants actually loaded to what's used (the data file's Google Fonts URLs already scope to the needed weights) — loading all 9 weights of a variable font when only 3 are used is a common unnecessary performance cost.
- Line height: 1.5-1.6 for body copy, 1.1-1.3 for large display headings — tighter headings, looser body text.
- Line length: keep body text columns to ~60-75 characters (`max-width: 65ch` in CSS / Tailwind `max-w-prose`) for reading comfort.
