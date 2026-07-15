---
name: color-contrast-theming
description: Palette construction, WCAG contrast math, semantic color roles, and dark mode errors and fixes for web UI.
---

# Color, Contrast & Theming

## Anti-pattern: No semantic color system, only raw hex values scattered in code
**What's wrong:** `color: #3b82f6` hardcoded in dozens of components instead of a named token like `--color-primary`.
**Why it matters:** A single brand-color tweak requires a find-and-replace across the codebase, and it's easy to introduce inconsistent near-duplicate blues.
**Fix:** Define a token layer once, reference tokens everywhere:

```css
:root {
  --color-primary: #2563eb;
  --color-primary-hover: #1d4ed8;
  --color-success: #16a34a;
  --color-warning: #d97706;
  --color-danger: #dc2626;
  --color-bg: #ffffff;
  --color-surface: #f8fafc;
  --color-border: #e2e8f0;
  --color-text-primary: #0f172a;
  --color-text-secondary: #475569;
  --color-text-tertiary: #94a3b8;
}
```

## Anti-pattern: Color as the only signal for meaning
**What's wrong:** Error fields shown only by a red border, success only by a green checkmark color with no icon, chart series distinguished only by hue.
**Why it matters:** ~1 in 12 men and ~1 in 200 women have some form of color vision deficiency; on top of that, color alone fails under grayscale printing, low-quality screens, and glare.
**Fix:** Always pair color with a second channel — icon, shape, pattern, text label, or position:

```html
<!-- Bad -->
<input class="border-red-500" />

<!-- Good -->
<input class="border-red-500" aria-invalid="true" aria-describedby="err-1" />
<p id="err-1" class="text-red-600 flex items-center gap-1">
  <ErrorIcon /> Email is required
</p>
```
For charts, use distinct shapes/patterns/line-styles in addition to color, and label series directly rather than relying purely on a color-coded legend.

## Anti-pattern: Contrast ratio not actually checked
**What's wrong:** A palette chosen purely by eye ("looks fine to me") without measuring contrast against WCAG.
**Why it matters:** "Looks fine" on a calibrated design monitor in a dim room is not the same as "readable" on a budget phone screen in sunlight.
**Fix:** Every text/background pairing must be checked against WCAG 2.x thresholds:

| Content type | Minimum ratio (AA) | Enhanced (AAA) |
| --- | --- | --- |
| Normal text (<24px, or <19px bold) | 4.5:1 | 7:1 |
| Large text (≥24px, or ≥19px bold) | 3:1 | 4.5:1 |
| UI components / graphical objects (icons, input borders, focus indicators) | 3:1 | — |

The ratio formula: relative luminance `L = 0.2126*R + 0.7152*G + 0.0722*B` (on linearized sRGB channels), then `contrast = (L1 + 0.05) / (L2 + 0.05)` for the lighter over darker luminance. In practice, use a tool (browser devtools contrast checker, or any WCAG contrast calculator) rather than computing by hand — but know the thresholds above by heart and check every new color pairing against them before shipping.

## Anti-pattern: Placeholder text used as if it were body/label text, at low contrast
**What's wrong:** Input placeholder styled at 2.5:1 contrast because "placeholders are supposed to be subtle."
**Why it matters:** If the placeholder is the only field guidance (see forms), it needs to be legible; subtlety should never cross below the 4.5:1 floor for anything conveying real information.
**Fix:** Style placeholders no lighter than your secondary text color, and never rely on placeholder-only guidance in the first place (see `06-forms-inputs.md`).

## Anti-pattern: Dark mode is "invert everything"
**What's wrong:** Naive dark mode implemented by CSS filter inversion, or by simply swapping `#fff`→`#000` and `#000`→`#fff` without rethinking every semantic color.
**Why it matters:** Pure black backgrounds cause halation/glow for many users and make elevation impossible to express with shadows (shadows are invisible on black); inverted brand colors often clash or become illegible; inverted photos/illustrations look wrong.
**Fix:** Build dark mode as its own considered palette, not a mechanical inversion:
- Use dark gray (`#121212`–`#1a1a1a`), not pure black, as the base surface — this leaves room for lighter "elevated" surfaces above it.
- Elevation in dark mode is expressed by *lighter* surfaces stacked on top of the base, not by shadows (which barely render on dark backgrounds). Each elevated layer gets a slightly lighter background than what's beneath it.
- Desaturate/lighten saturated brand colors slightly for dark backgrounds — a fully saturated brand blue that works on white can vibrate/glow uncomfortably on near-black.
- Never invert photographic content; exclude `<img>`/`<video>` from any CSS filter-based inversion approach entirely (and prefer a real token-based approach over filter inversion regardless).

```css
[data-theme="dark"] {
  --color-bg: #121212;
  --color-surface: #1e1e1e;
  --color-surface-elevated: #262626;
  --color-text-primary: #f1f5f9;
  --color-text-secondary: #94a3b8;
  --color-primary: #60a5fa; /* lightened from the light-mode blue */
}
```

## Anti-pattern: No respect for `prefers-color-scheme`
**What's wrong:** Dark mode exists only as a manual in-app toggle; the app ignores the OS-level preference on first load.
**Why it matters:** Users who've already set a system-wide preference (often for accessibility/eye-strain reasons) get the wrong theme by default and have to know your app has its own separate toggle.
**Fix:** Default to `prefers-color-scheme`, let an explicit in-app choice override and persist:

```css
@media (prefers-color-scheme: dark) {
  :root { /* dark tokens */ }
}
```
```js
const stored = localStorage.getItem("theme");
const theme = stored ?? (matchMedia("(prefers-color-scheme: dark)").matches ? "dark" : "light");
```

## Anti-pattern: Every state uses the same "brand blue," including destructive actions
**What's wrong:** Delete/remove/cancel-subscription buttons styled in the same primary blue as "Save" and "Continue."
**Why it matters:** Users pattern-match color to risk level across the whole web; a destructive action in the "safe" color removes the visual friction that should exist before an irreversible action.
**Fix:** Reserve a distinct danger color (typically red-family) exclusively for destructive actions, and never use it for anything else — its rarity is what makes it a reliable warning signal.

## Anti-pattern: Overusing saturated color across the whole UI
**What's wrong:** Every card, badge, button, and heading uses a different fully-saturated color "for personality."
**Why it matters:** When everything is loud, nothing stands out (Von Restorff effect in reverse), and the interface reads as visually exhausting rather than lively.
**Fix:** Reserve high-saturation color for the few elements that need to command attention (primary CTA, active state, key data point); use neutral/desaturated tones for structural chrome (borders, backgrounds, secondary text).

## Anti-pattern: Focus/hover/active/selected states not visually distinguished from each other
**What's wrong:** A nav item's "hover" and "selected/current" states look identical, or focus and hover share the exact same style.
**Why it matters:** Users lose track of what's currently active vs. what they're merely pointing at.
**Fix:** Design each interaction state as visually distinct, ideally layering (e.g. selected = background fill + bold text; hover = subtle background only; focus = ring/outline in addition to whatever else is present).

## Quick Checklist
- [ ] All colors reference named semantic tokens, no raw hex scattered in components
- [ ] Every color-conveyed meaning has a second channel (icon/label/shape)
- [ ] Every text/background pairing measured against WCAG AA (4.5:1 normal, 3:1 large text, 3:1 UI components)
- [ ] Dark mode is a deliberate palette (dark gray base, lighter elevation layers), not a filter inversion
- [ ] `prefers-color-scheme` respected as the default before any manual override
- [ ] Danger color reserved exclusively for destructive actions
- [ ] High saturation reserved for the few elements that must draw the eye
- [ ] Hover, focus, active, and selected states are all visually distinct from each other
