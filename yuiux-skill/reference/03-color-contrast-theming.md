# 03 — Color, Contrast & Theming

> Color is functional before it is aesthetic. Every color decision in a UI must serve communication, then hierarchy, then brand — in that order.

---

## The Color Hierarchy

```
1. Accessibility (contrast ratios — non-negotiable)
2. Communication (semantic meaning: red=error, green=success)
3. Hierarchy (primary > secondary > tertiary actions)
4. Brand (personality and recognition)
5. Aesthetics (preference — lowest priority)
```

Never let #4 or #5 violate #1 or #2.

---

## WCAG Contrast Requirements

### The Math

```
Relative Luminance:
  L = 0.2126 × R + 0.7152 × G + 0.0722 × B

where each channel is linearized:
  channel = val/255
  linearized = channel ≤ 0.03928 
               ? channel / 12.92 
               : ((channel + 0.055) / 1.055) ^ 2.4

Contrast Ratio:
  CR = (L_lighter + 0.05) / (L_darker + 0.05)
```

### Pass/Fail Thresholds

| Text Type | AA | AAA |
|-----------|-----|-----|
| Normal text (< 18px normal, < 14px bold) | 4.5:1 | 7:1 |
| Large text (≥ 18px normal OR ≥ 14px bold) | 3:1 | 4.5:1 |
| UI components (inputs, buttons, icons) | 3:1 | — |
| Decorative / incidental | exempt | exempt |
| Logotypes | exempt | exempt |

### Common Failures with Fixes

```css
/* ❌ FAIL: gray-400 on white = 2.85:1 */
.helper { color: #9ca3af; background: #ffffff; }

/* ✅ PASS: gray-500 on white = 4.63:1 (barely passes AA) */
.helper { color: #6b7280; background: #ffffff; }

/* ✅ STRONG: gray-600 on white = 7.0:1 (passes AAA) */
.helper { color: #4b5563; background: #ffffff; }

/* ❌ FAIL: white text on yellow = ~1.07:1 */
.badge-warning { color: #ffffff; background: #fbbf24; }

/* ✅ PASS: dark text on yellow = ~7.0:1 */
.badge-warning { color: #1c1917; background: #fbbf24; }

/* ❌ FAIL: light blue on white = 2.05:1 */
.link { color: #60a5fa; } /* blue-400 */

/* ✅ PASS: medium blue on white = 5.71:1 */
.link { color: #2563eb; } /* blue-600 */
```

### Dark Mode Contrast Trap

```css
/* ❌ Dark mode afterthought — fails contrast */
@media (prefers-color-scheme: dark) {
  body { background: #1a1a1a; color: #888888; } /* 3.0:1 ❌ */
}

/* ✅ Dark mode planned from start */
@media (prefers-color-scheme: dark) {
  body { 
    background: #0f172a; /* slate-900 */
    color: #e2e8f0;      /* slate-200 — 14.7:1 ✅ */
  }
  .muted { 
    color: #94a3b8;      /* slate-400 — 4.48:1 ✅ (barely) */
  }
  .helper {
    color: #cbd5e1;      /* slate-300 — 8.9:1 ✅ strong */
  }
}
```

---

## Color System Architecture

### Three-Layer Token System

```css
/* ===== LAYER 1: PRIMITIVES ===== */
/* Raw values — never reference directly in components */
:root {
  /* Blues */
  --color-blue-50:  #eff6ff;
  --color-blue-100: #dbeafe;
  --color-blue-200: #bfdbfe;
  --color-blue-300: #93c5fd;
  --color-blue-400: #60a5fa;
  --color-blue-500: #3b82f6;
  --color-blue-600: #2563eb;
  --color-blue-700: #1d4ed8;
  --color-blue-800: #1e40af;
  --color-blue-900: #1e3a8a;
  --color-blue-950: #172554;
  
  /* Grays (Slate) */
  --color-slate-50:  #f8fafc;
  --color-slate-100: #f1f5f9;
  --color-slate-200: #e2e8f0;
  --color-slate-300: #cbd5e1;
  --color-slate-400: #94a3b8;
  --color-slate-500: #64748b;
  --color-slate-600: #475569;
  --color-slate-700: #334155;
  --color-slate-800: #1e293b;
  --color-slate-900: #0f172a;
  --color-slate-950: #020617;
  
  /* Semantic status colors */
  --color-green-500: #22c55e;
  --color-green-600: #16a34a;
  --color-green-700: #15803d;
  
  --color-red-500: #ef4444;
  --color-red-600: #dc2626;
  --color-red-700: #b91c1c;
  
  --color-yellow-400: #facc15;
  --color-yellow-500: #eab308;
  --color-yellow-600: #ca8a04;
  
  /* Whites */
  --color-white: #ffffff;
}

/* ===== LAYER 2: SEMANTIC ===== */
/* Purpose-based aliases — use in base styles and layouts */
:root {
  /* Backgrounds */
  --background:        var(--color-white);
  --surface:           var(--color-slate-50);
  --surface-raised:    var(--color-white);
  
  /* Text */
  --foreground:        var(--color-slate-900);
  --foreground-muted:  var(--color-slate-500);
  --foreground-subtle: var(--color-slate-400);
  
  /* Brand */
  --primary:           var(--color-blue-600);
  --primary-hover:     var(--color-blue-700);
  --primary-active:    var(--color-blue-800);
  --primary-subtle:    var(--color-blue-50);
  --primary-foreground:var(--color-white);
  
  /* Secondary */
  --secondary:          var(--color-slate-100);
  --secondary-hover:    var(--color-slate-200);
  --secondary-foreground:var(--color-slate-900);
  
  /* Status */
  --success:           var(--color-green-600);
  --success-subtle:    #f0fdf4; /* green-50 */
  --success-foreground:var(--color-white);
  
  --warning:           var(--color-yellow-600);
  --warning-subtle:    #fefce8; /* yellow-50 */
  --warning-foreground:var(--color-slate-900);
  
  --destructive:       var(--color-red-600);
  --destructive-hover: var(--color-red-700);
  --destructive-subtle:#fef2f2; /* red-50 */
  --destructive-foreground: var(--color-white);
  
  /* Borders */
  --border:            var(--color-slate-200);
  --border-strong:     var(--color-slate-300);
  --input:             var(--color-slate-300);
  
  /* Focus */
  --ring:              var(--color-blue-600);
  
  /* Shadows */
  --shadow-sm:  0 1px 2px 0 rgb(0 0 0 / 0.05);
  --shadow:     0 1px 3px 0 rgb(0 0 0 / 0.1), 0 1px 2px -1px rgb(0 0 0 / 0.1);
  --shadow-md:  0 4px 6px -1px rgb(0 0 0 / 0.1), 0 2px 4px -2px rgb(0 0 0 / 0.1);
  --shadow-lg:  0 10px 15px -3px rgb(0 0 0 / 0.1), 0 4px 6px -4px rgb(0 0 0 / 0.1);
  --shadow-xl:  0 20px 25px -5px rgb(0 0 0 / 0.1), 0 8px 10px -6px rgb(0 0 0 / 0.1);
}

/* ===== DARK MODE ===== */
@media (prefers-color-scheme: dark) {
  :root {
    --background:        var(--color-slate-950);
    --surface:           var(--color-slate-900);
    --surface-raised:    var(--color-slate-800);
    
    --foreground:        var(--color-slate-50);
    --foreground-muted:  var(--color-slate-400);
    --foreground-subtle: var(--color-slate-500);
    
    --primary:           var(--color-blue-500);
    --primary-hover:     var(--color-blue-400);
    --primary-active:    var(--color-blue-300);
    --primary-subtle:    rgb(37 99 235 / 0.15);
    
    --secondary:          var(--color-slate-800);
    --secondary-hover:    var(--color-slate-700);
    --secondary-foreground: var(--color-slate-50);
    
    --success:           var(--color-green-500);
    --success-subtle:    rgb(34 197 94 / 0.15);
    
    --warning:           var(--color-yellow-400);
    --warning-subtle:    rgb(234 179 8 / 0.15);
    --warning-foreground:var(--color-slate-900);
    
    --destructive:       var(--color-red-500);
    --destructive-hover: var(--color-red-400);
    --destructive-subtle:rgb(239 68 68 / 0.15);
    
    --border:            var(--color-slate-800);
    --border-strong:     var(--color-slate-700);
    --input:             var(--color-slate-700);
    
    --shadow-sm:  0 1px 2px 0 rgb(0 0 0 / 0.3);
    --shadow:     0 1px 3px 0 rgb(0 0 0 / 0.4), 0 1px 2px -1px rgb(0 0 0 / 0.4);
    --shadow-md:  0 4px 6px -1px rgb(0 0 0 / 0.4);
    --shadow-lg:  0 10px 15px -3px rgb(0 0 0 / 0.4);
    --shadow-xl:  0 20px 25px -5px rgb(0 0 0 / 0.4);
  }
}

/* Class-based dark mode (for Lovable/shadcn/ui) */
.dark {
  /* Same as @media above, but class-toggled */
  --background: #020617;
  --foreground: #f8fafc;
  /* ... etc */
}
```

---

## Color Psychology

### What Colors Communicate

| Color | Associations | Use in UI | Avoid |
|-------|-------------|-----------|-------|
| **Blue** | Trust, calm, professional, technology | Primary actions, links, info states | Error states |
| **Green** | Success, growth, nature, money | Success states, progress, positive metrics | Error states |
| **Red** | Danger, urgency, error, passion | Error states, destructive actions | General decoration |
| **Yellow/Amber** | Warning, caution, energy, attention | Warning states, highlights | Success states |
| **Orange** | Enthusiasm, creativity, warmth | CTAs in consumer apps, highlights | Error states |
| **Purple** | Luxury, creativity, mystery | Premium features, creative tools | Status indicators |
| **Gray** | Neutral, disabled, subtle | Disabled states, subtle backgrounds | Branded actions |
| **Black** | Authority, sophistication, contrast | Text, dark backgrounds | Status indicators |
| **White** | Clean, minimal, space | Backgrounds, space | Text on light backgrounds |

### Status Color Rules (Always)

```
✅ Success   → Green (#16a34a light, #22c55e dark)
⚠️ Warning   → Amber/Yellow (#ca8a04 light, #facc15 dark)  
❌ Error     → Red (#dc2626 light, #ef4444 dark)
ℹ️ Info      → Blue (#2563eb light, #3b82f6 dark)
```

**Never use:**
- Red for success
- Green for error
- Yellow for info (reserved for warning)
- Any status color for branding (color meaning gets confused)

### Color + Icon Rule
```
ALWAYS pair color with an icon or text label for status.
Color alone fails WCAG 1.4.1 (Use of Color).

❌ Red border around field (only visual cue is color)
✅ Red border + error icon + error message text
```

---

## Palette Construction

### 5-Step Process

**Step 1: Choose a brand hue**
Select one primary hue (e.g., 220° — blue). The brand color should be on the 500–700 step of the scale (strong enough to be primary action color with sufficient contrast on white).

**Step 2: Build the tint/shade scale**
Generate a 10-step scale (50, 100, 200, 300, 400, 500, 600, 700, 800, 900).
- Steps 50–200: backgrounds, subtle highlights
- Steps 300–400: disabled states, borders in dark mode
- Steps 500–600: MAIN BRAND COLOR (check contrast!)
- Steps 700–800: hover/active states, dark mode primary
- Steps 900: text on light backgrounds

**Step 3: Choose a neutral (gray) family**
Options: Slate (blue-tinted), Gray (neutral), Stone (warm), Zinc (cool-neutral). Match the warmth/coolness of your brand color.

**Step 4: Define semantic colors**
Map success/warning/error/info from standard palettes. Adjust shades for dark mode.

**Step 5: Verify all combinations**
Check every text/background pair for WCAG AA compliance before finalizing.

### Generating Palette Colors (HSL Formula)

```javascript
// Generate a 10-step color scale from a base HSL
function generateScale(h, s, baseL) {
  const steps = [95, 90, 80, 68, 55, 43, 35, 27, 20, 12];
  return steps.map((l, i) => ({
    step: [50, 100, 200, 300, 400, 500, 600, 700, 800, 900][i],
    value: `hsl(${h}, ${s}%, ${l}%)`
  }));
}

// Example: Blue brand color
const bluePalette = generateScale(217, 91, 43);
// → blue-600: hsl(217, 91%, 43%) ≈ #2563eb
```

---

## Dark Mode Implementation

### Strategy: CSS Custom Properties + Class Toggle

```tsx
// React dark mode with system preference + user override
import { useEffect, useState } from 'react';

type Theme = 'light' | 'dark' | 'system';

function useTheme() {
  const [theme, setTheme] = useState<Theme>(() => {
    if (typeof window === 'undefined') return 'system';
    return (localStorage.getItem('theme') as Theme) ?? 'system';
  });
  
  useEffect(() => {
    const root = document.documentElement;
    const mq = window.matchMedia('(prefers-color-scheme: dark)');
    
    function apply(t: Theme) {
      const isDark = t === 'dark' || (t === 'system' && mq.matches);
      root.classList.toggle('dark', isDark);
    }
    
    apply(theme);
    localStorage.setItem('theme', theme);
    
    // Update when system preference changes
    if (theme === 'system') {
      mq.addEventListener('change', () => apply('system'));
      return () => mq.removeEventListener('change', () => apply('system'));
    }
  }, [theme]);
  
  return { theme, setTheme };
}
```

### Dark Mode Color Rules

```
LIGHT → DARK mappings:
  Background:    white → slate-950
  Surface:       slate-50 → slate-900
  Border:        slate-200 → slate-800
  Text primary:  slate-900 → slate-50
  Text muted:    slate-500 → slate-400 (check contrast!)
  
  Primary:       blue-600 → blue-500 (lighter in dark = still readable)
  Success:       green-600 → green-500
  Error:         red-600 → red-500
  Warning:       yellow-600 → yellow-400 (yellow text needs to be MUCH lighter in dark)
  
  Shadows:       rgba(0,0,0,0.1) → rgba(0,0,0,0.4)
```

### Images in Dark Mode

```css
/* Reduce image brightness in dark mode for comfort */
@media (prefers-color-scheme: dark) {
  img {
    filter: brightness(0.9);
  }
}

/* Or class-based */
.dark img {
  filter: brightness(0.9);
}

/* Invert dark-mode-unfriendly SVGs */
.dark .svg-icon-dark {
  filter: invert(1);
}
```

---

## Semantic Color Usage Rules

### Status Colors Must Include Text/Icon

```tsx
// ❌ Color as sole indicator
<div style={{ border: '1px solid red' }}>
  Username is taken
</div>

// ✅ Color + icon + text
<div className="flex items-center gap-2 text-destructive">
  <AlertCircleIcon className="w-4 h-4" aria-hidden />
  <span>Username is already taken</span>
</div>
```

### Disabled States

```css
/* Disabled elements should be visually distinct but not invisible */
[disabled], [aria-disabled="true"] {
  opacity: 0.5;
  cursor: not-allowed;
  pointer-events: none;
  
  /* Never go below 2:1 contrast for disabled text */
  /* (disabled elements are exempt from WCAG contrast but should still be visible) */
}
```

### Focus Rings

```css
/* Universal focus visible rule */
:focus-visible {
  outline: 2px solid var(--ring);
  outline-offset: 2px;
}

/* Remove old :focus outline — only use :focus-visible */
:focus:not(:focus-visible) {
  outline: none;
}

/* High contrast mode support */
@media (forced-colors: active) {
  :focus-visible {
    outline: 2px solid ButtonText;
  }
}
```

---

## Brand Color Application

### The 60/30/10 Rule

```
60% — Primary neutral (background, surface)
30% — Secondary (secondary surfaces, borders)
10% — Accent (CTAs, links, highlights — brand color)
```

```
Example: A blue-branded SaaS app

60%: White backgrounds, light gray surfaces
30%: Slate borders, gray text, subtle backgrounds
10%: Blue primary buttons, blue links, blue active states
```

### When to Use Brand Color

```
✅ Primary CTA buttons
✅ Links (in body text)
✅ Active/selected navigation items
✅ Form focus rings
✅ Progress indicators
✅ Checkboxes and radio buttons (checked state)
✅ Toggle switches (on state)

❌ Error messages (use red — brand color creates confusion)
❌ Warning messages (use yellow)
❌ Large background areas (too overwhelming)
❌ Body text (decreases readability for colored text)
❌ Every element on screen (reduces emphasis)
```

---

## Color Contrast Audit Process

### Automated

```bash
# Using axe-cli
npx axe-cli https://your-app.com --save report.json

# Using lighthouse
lighthouse https://your-app.com --only-categories=accessibility --output=json

# Using pa11y
npx pa11y https://your-app.com --standard WCAG2AA
```

### Manual Spot Checks

Top 10 text/background pairs to always check:
1. Body text on page background
2. Helper/muted text on page background
3. White text on primary button
4. White text on destructive button
5. Text on input placeholder
6. Text on card surface
7. Header text on brand-colored header
8. Error message text on error background
9. Dark mode — body text
10. Dark mode — muted text

### Tools
- [WebAIM Contrast Checker](https://webaim.org/resources/contrastchecker/)
- [Polypane Contrast Grid](https://contrast.polypane.app/)
- Browser DevTools → Accessibility → Contrast
- VS Code extension: Color Highlight + Accessibility Linter

---

## Color Anti-Patterns

| Anti-Pattern | Problem | Fix |
|-------------|---------|-----|
| Placeholder-only contrast check | Placeholder fades; real text might fail | Check with real entered text (same color) |
| Gray text on gray background | Common muted+surface fail | Check both light and dark mode |
| Saturated color on saturated color | Vibration / visual noise | Use neutral for one side |
| Gradient text | Contrast varies across gradient | Test darkest+lightest regions separately |
| White icon on pale background | Fails contrast | Use a border or adjust bg |
| Status color for decoration | Pollutes meaning | Reserve status colors for status only |
| Brand color for error state | Users learn red=error; blue=error breaks model | Always use red for errors |
| Too many accent colors | Visual chaos | Max 2–3 colors in active use |
| No color in dark mode (all gray) | "Gray mode" — loses identity | Adjust saturation, not just lightness |

---

## Complete shadcn/ui Compatible Token Set

```css
/* Drop into globals.css — compatible with shadcn/ui and Lovable */

@layer base {
  :root {
    --background: 0 0% 100%;
    --foreground: 222.2 84% 4.9%;
    --card: 0 0% 100%;
    --card-foreground: 222.2 84% 4.9%;
    --popover: 0 0% 100%;
    --popover-foreground: 222.2 84% 4.9%;
    --primary: 221.2 83.2% 53.3%;
    --primary-foreground: 210 40% 98%;
    --secondary: 210 40% 96.1%;
    --secondary-foreground: 222.2 47.4% 11.2%;
    --muted: 210 40% 96.1%;
    --muted-foreground: 215.4 16.3% 46.9%;
    --accent: 210 40% 96.1%;
    --accent-foreground: 222.2 47.4% 11.2%;
    --destructive: 0 84.2% 60.2%;
    --destructive-foreground: 210 40% 98%;
    --border: 214.3 31.8% 91.4%;
    --input: 214.3 31.8% 91.4%;
    --ring: 221.2 83.2% 53.3%;
    --radius: 0.5rem;
  }

  .dark {
    --background: 222.2 84% 4.9%;
    --foreground: 210 40% 98%;
    --card: 222.2 84% 4.9%;
    --card-foreground: 210 40% 98%;
    --popover: 222.2 84% 4.9%;
    --popover-foreground: 210 40% 98%;
    --primary: 217.2 91.2% 59.8%;
    --primary-foreground: 222.2 47.4% 11.2%;
    --secondary: 217.2 32.6% 17.5%;
    --secondary-foreground: 210 40% 98%;
    --muted: 217.2 32.6% 17.5%;
    --muted-foreground: 215 20.2% 65.1%;
    --accent: 217.2 32.6% 17.5%;
    --accent-foreground: 210 40% 98%;
    --destructive: 0 62.8% 30.6%;
    --destructive-foreground: 210 40% 98%;
    --border: 217.2 32.6% 17.5%;
    --input: 217.2 32.6% 17.5%;
    --ring: 224.3 76.3% 48%;
  }
}
```
