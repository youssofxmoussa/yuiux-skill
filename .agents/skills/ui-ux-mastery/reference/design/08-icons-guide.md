---
name: icons-guide
description: Icon library conventions (Phosphor primary, Heroicons fallback), category reference, style-config presets per design language, and accessibility rules for icon usage. Raw table in data/icons.csv.
---

# Icons Guide

Full raw data (105 rows: category, icon name, keywords, library, import code, usage, style) lives in `../../data/icons.csv`. Grep it by keyword when you need a specific icon's exact import/usage snippet.

## Library strategy

- **Primary: Phosphor Icons** (`@phosphor-icons/react` for web, `phosphor-react-native` for Expo/RN) — large, consistent, multi-weight (thin/light/regular/bold/fill/duotone) icon set covering navigation, action, status, communication, user, media, commerce, data, files, layout, social, device, security, location, time, and development categories.
- **Fallback: Heroicons** (`@heroicons/react/24/outline` or `/solid`) when Phosphor lacks a semantically fitting icon. When falling back, match stroke weight and corner rounding to whatever Phosphor icons are already on-screen so the set doesn't visually clash.
- Never mix three or more icon libraries in one product — pick primary + one fallback, and standardize import patterns for the whole codebase.

## Accessibility rules

- Icon-only buttons **always** need an accessible name: `aria-label` (web) or `accessibilityLabel` (React Native). This is a P0 rule — see `00-priority-checklist.md`.
- Purely decorative icons (next to text that already conveys the same meaning) should be hidden from assistive tech: `aria-hidden="true"` on web, `accessible={false}` / `importantForAccessibility="no-hide-descendants"` on RN.
- Icon size for tap targets: render the icon at 20-24px but pad the touch area to the 44×44 minimum (see P1 touch rules) rather than enlarging the glyph itself.

## Style-config presets

The icon *treatment* should match the product's overall style language (see `01-ui-styles-catalog.md`), not just its category:

- **Bold Typography / editorial mobile** — `weight="regular"`, 20px for controls / 32px for feature anchors, always paired with a mono-label; standalone icons only for universal navigation glyphs (back arrow).
- **Cyberpunk/HUD** — `weight="regular"`, accent-colored glow (drop shadow at 0.6 opacity / 8px radius on the icon's wrapper), square (non-rounded) containers, paired with a mono-font data label.
- **Academia/scholarly** — `weight="thin"`, warm brass/muted accent color, circular 1px-bordered containers, book/scroll/key-style metaphors rather than sharp geometric icons.
- **Web3/DeFi/fintech** — `weight="regular"`, saturated brand accent (e.g. bitcoin orange), circular blurred-glass containers with a colored glow, finance-appropriate icon choices (trend, wallet, shield, layers).

## Category quick-reference

Navigation (menu, back/forward, caret, home, close, external-link) · Action (add, remove, delete, edit, save, upload/download, copy, share, search, filter, settings) · Status (success/error/warning/info/loading/pending) · Communication (email, chat, phone, send, notification) · User (profile, team, invite, sign-in/out) · Media (image, video, play/pause, volume, mic, camera) · Commerce (cart, bag, payment, price, discount) · Data (bar/pie chart, trend up/down, activity, database) · Files (document, folder, attachment, link, clipboard) · Layout (grid, list, columns, fullscreen, sidebar) · Social (like, star, thumbs, bookmark, flag) · Device (mobile, tablet, monitor, laptop, printer) · Security (lock, shield, key, show/hide password) · Location (pin, map, compass, globe) · Time (calendar, refresh, undo/redo) · Development (code, terminal, git, github).

## SVG hand-authoring rules (when generating a custom icon rather than using the library)

- ViewBox: `0 0 24 24` standard, `0 0 16 16` for compact UI contexts.
- Use `currentColor` for fill/stroke so the icon inherits CSS color — never hardcode a hex value inside the SVG.
- Include a `<title>` element for accessibility.
- `stroke-linecap="round"` and `stroke-linejoin="round"` for outlined styles to match most modern icon sets.
- Design at 24px and verify legibility at both 16px and 48px before shipping.
