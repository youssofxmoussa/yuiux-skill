---
name: shadcn-tailwind-quickref
description: shadcn/ui + Tailwind CSS component, theming, and accessibility quick reference — component/theming patterns, CVA variants, dark mode via CSS variables, and common pitfalls. Use for any React + Tailwind + shadcn build.
---

# shadcn/ui + Tailwind Quick Reference

## Core principle

shadcn/ui is not a component *library* you install as a dependency — it's a set of copy-into-your-repo component source files built on Radix primitives + Tailwind. You own and can edit every component directly; there is no black-box package to fight. Prefer editing the copied component source over wrapping it in extra layers when you need custom behavior.

## Theming

- Define the semantic color tokens (see `02-color-palettes.md`'s token model: `background`, `foreground`, `primary`, `secondary`, `muted`, `accent`, `destructive`, `border`, `ring`, `card`) as CSS custom properties in `:root` and `.dark`, then reference them in `tailwind.config` via `hsl(var(--primary))` etc. — never hardcode hex values directly in component `className` strings.
- Dark mode should flip the CSS variable values, not add a parallel set of Tailwind `dark:` utility overrides scattered through every component — centralizing theme values in variables keeps light/dark visually and structurally consistent.
- Verify contrast for every token pairing in **both** themes independently — a token set that passes AA in light mode does not automatically pass in dark mode.

## Component variant patterns (CVA — class-variance-authority)

- Define variants (size, intent/color, state) via `cva()` rather than conditional className strings scattered through JSX — this keeps variant logic declarative and co-located with the component.
- Compose new component "flavors" by wrapping the base shadcn component and passing through variant props, rather than duplicating the entire component file for small visual differences.

## Accessibility (inherited from Radix, don't undo it)

- shadcn components built on Radix primitives (`Dialog`, `DropdownMenu`, `Popover`, `Select`, `Tabs`) already handle focus trapping, `Escape`-to-close, and correct ARIA roles — don't strip these out when customizing styles; only override visual (className) props, not the underlying primitive behavior.
- When composing a custom trigger for a Radix primitive (e.g. wrapping `DialogTrigger` around a custom icon button), make sure the custom trigger still forwards `asChild` correctly so Radix can attach its own ARIA attributes — a common bug is wrapping the trigger in an extra unnecessary `<div>` that breaks this chain.
- Icon-only buttons built with shadcn's `Button` component still need an explicit `aria-label` — the component doesn't add one automatically just because it's shadcn.

## Layout & spacing conventions

- Use Tailwind's spacing scale (`gap-2`, `gap-4`, `p-6`, etc.) consistently rather than arbitrary values (`gap-[13px]`) — arbitrary values are a signal the spacing scale needs a token, not an escape hatch to reach for by default.
- Compose layouts with Tailwind's `flex`/`grid` utilities directly in JSX rather than writing custom CSS files — this keeps layout logic co-located with the component markup, which is the framework's core ergonomic advantage.

## Common pitfalls

- Forgetting to wrap the app in the shadcn `<Toaster />` / theme provider at the root, then components silently fail to pick up theme context.
- Overriding a Radix primitive's default z-index without checking the app's z-index scale (see `04-ux-guidelines-reference.md`) — dialogs/popovers rendered via portal need to sit above all app content including other fixed elements.
- Applying `focus:outline-none` to a shadcn component without shadcn's paired `focus-visible:ring-*` utility already present — always verify the focus ring survives any style override (P0 rule in `00-priority-checklist.md`).
- Installing every available shadcn component "just in case" — only add components you're using; unused copied source files add maintenance surface with no benefit.

## When to reach for shadcn vs. hand-rolled components

Use shadcn for: dialogs/modals, dropdowns, selects, tabs, tooltips, popovers, forms with validation (paired with `react-hook-form` + `zod`), data tables (paired with `@tanstack/react-table`) — anywhere correct keyboard/focus/ARIA behavior is non-trivial to hand-roll correctly. Hand-roll simple presentational components (badges, simple cards, static layout wrappers) where there's no complex interaction behavior to inherit.
