---
name: design-systems-consistency
description: Design tokens, component reuse, and cross-page consistency errors and fixes for web UI.
---

# Design Systems & Consistency

Consistency (Nielsen heuristic #4) is a force multiplier — every inconsistency forces the user to relearn something they thought they already knew, and every consistent pattern makes everything else in the product easier to learn by analogy.

## Anti-pattern: No shared design tokens — colors, spacing, and type sizes hardcoded ad hoc per component
**What's wrong:** Each component defines its own one-off hex color, pixel spacing value, or font size rather than referencing a shared set of tokens.
**Why it matters:** Hardcoded values drift apart over time (a "primary blue" that's subtly different in five different places), and a later brand/theme change requires hunting down every hardcoded instance instead of updating one source of truth.
**Fix:** Define a single set of design tokens (color, spacing, type scale, radius, shadow, z-index — see the relevant reference files for each) as CSS custom properties or a theme object, and require every component to reference tokens rather than hardcoded values.

```css
:root {
  --color-primary: #2563eb;
  --space-4: 16px;
  --radius-md: 8px;
}
.card { border-radius: var(--radius-md); padding: var(--space-4); }
```

## Anti-pattern: The same UI concept implemented multiple different ways across the app
**What's wrong:** Three different "card" components with slightly different padding/shadow/radius, built independently by not knowing (or not reusing) an existing one; two different date pickers; two different confirmation-dialog patterns.
**Why it matters:** Beyond the maintenance cost, subtly different implementations of the same concept register to users as unpolished inconsistency even if they can't articulate exactly why.
**Fix:** Before building a new instance of a common pattern (card, modal, dropdown, table, empty state), check whether an existing shared component already covers it; consolidate duplicates into one canonical, reusable implementation.

## Anti-pattern: Interaction patterns behave differently in different parts of the same app
**What's wrong:** Pressing Escape closes a modal in one part of the app but does nothing in another; a "save" auto-triggers on blur in one form but requires an explicit button in a near-identical one elsewhere.
**Why it matters:** Once a user learns a behavior in one context, they reasonably expect it everywhere else in the same product — violated expectations feel like bugs even when each individual behavior is "correct" in isolation.
**Fix:** Standardize core interaction behaviors (dismiss keys, save patterns, keyboard shortcuts, hover/focus behavior) as a documented convention and apply them uniformly, rather than deciding per-feature.

## Anti-pattern: Terminology and iconography inconsistent for the same concept
**What's wrong:** Covered from the writing angle in `14-microcopy-content-strategy.md`; the design-system angle is using different icons for the same action across different screens (a trash icon here, an "X" there, both meaning delete).
**Why it matters:** Icon meaning is learned per-product, not universal — using two different icons for the same action forces the user to relearn it in each context.
**Fix:** Maintain an icon usage guide mapping specific icons to specific meanings, one-to-one, and enforce it across the codebase (a shared icon component library helps here structurally).

## Anti-pattern: Component variants proliferate without a system
**What's wrong:** A button component grows ad hoc `size="sm2"`, `variant="blueish"`, `type="alt-primary"` props added one-off by different contributors solving slightly different problems each time.
**Why it matters:** Uncontrolled variant growth defeats the purpose of having a shared component at all — eventually it's easier to write a new one-off component than decode the existing variant matrix, restarting the whole problem.
**Fix:** Define a small, deliberate set of variants/sizes up front (e.g. primary/secondary/tertiary/destructive × sm/md/lg) and require new visual needs to be evaluated against that existing matrix before adding a new variant — most "new" needs fit an existing combination.

## Anti-pattern: Different pages/features have visibly different visual "eras"
**What's wrong:** Older parts of the product retain a prior visual style (colors, spacing, corner radius, typography) that was never updated when the design language evolved elsewhere.
**Why it matters:** A product that looks like it was built by several different teams with no coordination undermines trust in its overall quality and care, even if each individual screen is internally fine.
**Fix:** When a design system update happens, track and prioritize migrating high-traffic legacy screens rather than letting old and new styles coexist indefinitely; where a full migration isn't immediately feasible, at minimum ensure token-level values (colors, spacing) stay shared so a global update propagates everywhere automatically.

## Anti-pattern: Spacing and alignment drift between visually adjacent components
**What's wrong:** Two cards sitting side by side use slightly different internal padding, or a form's label-to-input gap differs from a visually similar form elsewhere on the same page.
**Why it matters:** Small misalignments are individually barely perceptible but collectively register as "something feels off" without an obvious cause — undermining polish.
**Fix:** Rely on the shared spacing scale (see `04-layout-spacing-grid.md`) consistently, and specifically review adjacent/comparable components side-by-side during design review for drift.

## Quick Checklist
- [ ] Colors, spacing, type scale, radius, and shadows are defined as shared tokens, never hardcoded per component
- [ ] Common UI concepts (cards, modals, dropdowns, empty states) have one canonical, reused implementation, not multiple divergent ones
- [ ] Core interaction behaviors (Escape, save patterns, keyboard shortcuts) are standardized app-wide
- [ ] Icons map one-to-one to meanings, consistently across the whole product
- [ ] Component variants are a small, deliberate, documented set — not organically sprawling props
- [ ] Design-system updates prioritize migrating high-traffic legacy screens rather than leaving mismatched visual eras indefinitely
- [ ] Spacing/alignment is checked for drift between visually adjacent or comparable components
