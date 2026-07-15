---
name: design-system-generation-workflow
description: Step-by-step methodology for deriving a coherent, product-specific design system (style, color, type, motion, components) from a one-line product brief, using this skill's reference files — no CLI/script dependency, works in any agent environment.
---

# Design System Generation Workflow

This is the practical "how to actually use this skill end-to-end" file. It replaces the original pro-max skill's CLI-driven workflow (`scripts/search.py --design-system` etc.) with a pure-reasoning process any agent can follow directly from these markdown files and CSV lookups — no code execution required, though grepping the CSVs with a shell tool is a helpful shortcut when available.

## Step 1 — Classify the product

From the user's brief, identify: product type/vertical, target audience, and primary emotional register (trust / premium / playful / technical / creative — see `00-priority-checklist.md` P3). If the brief maps closely to an existing category, look it up directly in `05-product-type-reasoning.md` (and its backing `data/ui-reasoning.csv` / `data/products.csv`) for a validated starting point covering style, color mood, typography mood, and motion together as a set.

## Step 2 — Lock the style direction

Pick **one** primary style family from `01-ui-styles-catalog.md`, optionally one secondary accent technique. Do not proceed to color/type selection until this is locked — style choice constrains what colors and motion will actually look coherent (e.g. neumorphism forecloses high-saturation vibrant color; brutalism forecloses soft pastel palettes).

## Step 3 — Derive the color token set

Use `02-color-palettes.md` and `data/colors.csv` to either look up a matching product-type palette or construct one following the semantic token model (`primary/secondary/accent/background/card/muted/border/destructive/ring`). Verify every `on-*` pairing against contrast rules before finalizing. If dark mode is needed, derive it as an independent palette per the dark-mode section, not an inversion.

## Step 4 — Pick a typography pairing

Use `03-typography-pairings.md` to select one named pairing matching the same emotional register as the style/color choices. Confirm the pairing's "Best For" list plausibly includes the product's vertical; if not, pick the nearest mood-match rather than forcing a mismatched pairing.

## Step 5 — Set motion tokens

Use `06-motion-tokens.md` to pick an intensity tier (Subtle/Standard/Complex) matching the register — trust-critical and data-dense products default to Subtle, consumer/marketing to Standard, and flagship/portfolio moments only to Complex. Apply the same tier consistently across hover/scroll/stagger/transition categories rather than mixing tiers within one product.

## Step 6 — Select supporting patterns

- Landing/marketing pages: pick a section-order pattern from `10-landing-page-patterns.md` matching the primary conversion goal.
- Dashboards/data-heavy screens: pick chart types per data shape from `07-charts-dataviz.md`, respecting accessibility grade minimums.
- Native/mobile builds: apply `09-app-interface-patterns.md` on top of the above (it's additive, not a replacement).
- Icons: pick the Phosphor style-config preset in `08-icons-guide.md` matching the chosen style family.

## Step 7 — Apply the priority checklist as a build-order gate

Build in the P0→P10 order from `00-priority-checklist.md` — accessibility and touch targets are structural (P0-P1) and expensive to retrofit; style/color/motion polish (P3-P6) should be layered on top of a functionally sound, accessible base, not the other way around.

## Step 8 — Pre-ship review

Before presenting the result, cross-check against:
- `../errors/26-master-anti-pattern-catalog.md` for the master anti-pattern list.
- The specific anti-patterns flagged for this product's category in `data/ui-reasoning.csv` (every category row lists explicit anti-patterns — e.g. "AI purple/pink gradients" for trust-critical verticals, "static design" for gaming/creative).
- `15-dev-pipeline-ci-automation.md` if the project has (or should have) automated quality gates.

## Worked example (condensed)

**Brief: "build a habit-tracking app for young professionals."**
1. Classify → Habit Tracker in `data/ui-reasoning.csv` → Social Proof-Focused pattern, Claymorphism + Vibrant & Block-based style, streak-warm color mood, playful/rounded typography, spring-bounce soft-press motion.
2. Style → Claymorphism primary, no secondary needed (avoid mixing with glass/neumorphism per the anti-pattern rules in `01-ui-styles-catalog.md`).
3. Color → look up "Habit Tracker" in `data/colors.csv` for the exact token set (streak-warm amber/orange primary, progress-green accent).
4. Typography → Playful Creative pairing (Fredoka/Nunito) from `03-typography-pairings.md`.
5. Motion → Standard tier: spring/bounce entrances, 200ms soft-press feedback on habit-completion taps.
6. Supporting patterns → progress-ring chart pattern (`07-charts-dataviz.md` gauge/bullet family) for daily/weekly completion; onboarding pattern from `../errors/13-onboarding-first-run.md`.
7. Priority gate → confirm 44px tap targets on the daily-check-in buttons (P1), confirm color isn't the sole signal for streak-broken state (P0), then layer in the claymorphism shadows and spring motion (P5-P6).

## Why this replaces the CLI-driven approach

The original tool's `scripts/search.py --design-system` workflow required a Python runtime and BM25 search index to look up this same information. That's Claude-Code/CLI-specific tooling that isn't guaranteed to exist in every environment (in particular, many Lovable-style agents can't execute arbitrary shell scripts). Every lookup above is achievable by reading a markdown file or grepping a CSV directly — no script execution dependency — so this workflow runs identically in Replit, Lovable, Claude, or any other agent environment with basic file-read/search capability.
