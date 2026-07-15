# 19 — Design Tokens

> Design tokens are the atoms of a design system. They capture every design decision — color, spacing, typography, motion — as named, reusable values that can be shared across platforms.

---

## What Are Design Tokens?

A design token is a **named design decision**: instead of hardcoding `#2563eb` in 50 places, you name it `--color-primary` and use the name everywhere. When the brand color changes, you change one value.

```
Raw value:  #2563eb
Without tokens: background-color: #2563eb;  (scattered everywhere)
With tokens:    background-color: var(--color-primary);  (one source of truth)
```

---

## Token Hierarchy

```
Tier 1: Primitives (raw values)
  --color-blue-600: #2563eb;
  --space-4: 16px;
  --font-size-base: 16px;

Tier 2: Semantic (purpose-based)
  --color-primary: var(--color-blue-600);
  --space-element-gap: var(--space-4);
  --font-size-body: var(--font-size-base);

Tier 3: Component (component-specific)
  --button-bg: var(--color-primary);
  --button-padding-y: var(--space-2);
  --button-font-size: var(--font-size-body);
```

**Rules:**
- Components reference Tier 2 (semantic) tokens, never Tier 1 (primitives)
- Tier 2 references Tier 1 only
- Never skip tiers: `--button-bg: #2563eb` (skips semantic) — WRONG

---

## Complete Token Set

### Color Tokens

```css
/* ===== TIER 1: PRIMITIVES ===== */
:root {
  /* Blue scale */
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
  
  /* Neutral scale */
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
  
  /* Green scale */
  --color-green-50:  #f0fdf4;
  --color-green-500: #22c55e;
  --color-green-600: #16a34a;
  --color-green-700: #15803d;
  
  /* Red scale */
  --color-red-50:  #fef2f2;
  --color-red-400: #f87171;
  --color-red-500: #ef4444;
  --color-red-600: #dc2626;
  --color-red-700: #b91c1c;
  
  /* Amber scale */
  --color-amber-50:  #fffbeb;
  --color-amber-400: #fbbf24;
  --color-amber-500: #f59e0b;
  --color-amber-600: #d97706;
  
  /* Pure */
  --color-white: #ffffff;
  --color-black: #000000;
}

/* ===== TIER 2: SEMANTIC ===== */
:root {
  /* Background */
  --color-bg:           var(--color-white);
  --color-bg-subtle:    var(--color-slate-50);
  --color-bg-muted:     var(--color-slate-100);
  
  /* Surface (elevated over background) */
  --color-surface:        var(--color-white);
  --color-surface-raised: var(--color-white);
  --color-surface-sunken: var(--color-slate-50);
  
  /* Text */
  --color-text:           var(--color-slate-900);
  --color-text-muted:     var(--color-slate-500);
  --color-text-subtle:    var(--color-slate-400);
  --color-text-on-primary:var(--color-white);
  --color-text-on-dark:   var(--color-white);
  --color-text-disabled:  var(--color-slate-300);
  
  /* Brand / Primary */
  --color-primary:        var(--color-blue-600);
  --color-primary-hover:  var(--color-blue-700);
  --color-primary-active: var(--color-blue-800);
  --color-primary-subtle: var(--color-blue-50);
  
  /* Secondary */
  --color-secondary:        var(--color-slate-100);
  --color-secondary-hover:  var(--color-slate-200);
  --color-secondary-active: var(--color-slate-300);
  
  /* Status */
  --color-success:        var(--color-green-600);
  --color-success-subtle: var(--color-green-50);
  --color-warning:        var(--color-amber-600);
  --color-warning-subtle: var(--color-amber-50);
  --color-danger:         var(--color-red-600);
  --color-danger-hover:   var(--color-red-700);
  --color-danger-subtle:  var(--color-red-50);
  --color-info:           var(--color-blue-600);
  --color-info-subtle:    var(--color-blue-50);
  
  /* Border */
  --color-border:         var(--color-slate-200);
  --color-border-strong:  var(--color-slate-300);
  --color-border-subtle:  var(--color-slate-100);
  
  /* Interactive */
  --color-focus-ring:     var(--color-blue-600);
  --color-focus-ring-bg:  var(--color-blue-100);
}

/* ===== DARK MODE: TIER 2 OVERRIDES ===== */
@media (prefers-color-scheme: dark) {
  :root {
    --color-bg:           var(--color-slate-950);
    --color-bg-subtle:    var(--color-slate-900);
    --color-bg-muted:     var(--color-slate-800);
    --color-surface:        var(--color-slate-900);
    --color-surface-raised: var(--color-slate-800);
    --color-surface-sunken: var(--color-slate-950);
    --color-text:           var(--color-slate-50);
    --color-text-muted:     var(--color-slate-400);
    --color-text-subtle:    var(--color-slate-500);
    --color-primary:        var(--color-blue-500);
    --color-primary-hover:  var(--color-blue-400);
    --color-border:         var(--color-slate-800);
    --color-border-strong:  var(--color-slate-700);
    --color-border-subtle:  var(--color-slate-900);
    --color-success:        var(--color-green-500);
    --color-danger:         var(--color-red-500);
  }
}
```

### Spacing Tokens

```css
:root {
  /* ===== TIER 1: PRIMITIVES ===== */
  --space-0:   0;
  --space-px:  1px;
  --space-0-5: 2px;
  --space-1:   4px;
  --space-1-5: 6px;
  --space-2:   8px;
  --space-2-5: 10px;
  --space-3:   12px;
  --space-3-5: 14px;
  --space-4:   16px;
  --space-5:   20px;
  --space-6:   24px;
  --space-7:   28px;
  --space-8:   32px;
  --space-9:   36px;
  --space-10:  40px;
  --space-11:  44px;
  --space-12:  48px;
  --space-14:  56px;
  --space-16:  64px;
  --space-20:  80px;
  --space-24:  96px;
  --space-32:  128px;
  
  /* ===== TIER 2: SEMANTIC ===== */
  --space-component-xs:  var(--space-1);   /* 4px  — icon gap */
  --space-component-sm:  var(--space-2);   /* 8px  — tight elements */
  --space-component:     var(--space-4);   /* 16px — standard padding */
  --space-component-lg:  var(--space-6);   /* 24px — card padding */
  --space-component-xl:  var(--space-8);   /* 32px — large component */
  
  --space-layout-sm:     var(--space-6);   /* 24px — between related sections */
  --space-layout:        var(--space-12);  /* 48px — section spacing */
  --space-layout-lg:     var(--space-16);  /* 64px — major section */
  --space-layout-xl:     var(--space-24);  /* 96px — page-level spacing */
}
```

### Typography Tokens

```css
:root {
  /* ===== TIER 1: PRIMITIVES ===== */
  
  /* Scale */
  --font-size-xs:   12px;
  --font-size-sm:   14px;
  --font-size-base: 16px;
  --font-size-lg:   18px;
  --font-size-xl:   20px;
  --font-size-2xl:  24px;
  --font-size-3xl:  30px;
  --font-size-4xl:  36px;
  --font-size-5xl:  48px;
  --font-size-6xl:  60px;
  --font-size-7xl:  72px;
  
  /* Weights */
  --font-weight-light:    300;
  --font-weight-regular:  400;
  --font-weight-medium:   500;
  --font-weight-semibold: 600;
  --font-weight-bold:     700;
  --font-weight-extrabold:800;
  
  /* Line heights */
  --leading-none:     1;
  --leading-tight:    1.25;
  --leading-snug:     1.375;
  --leading-normal:   1.5;
  --leading-relaxed:  1.625;
  --leading-loose:    2;
  
  /* Letter spacing */
  --tracking-tighter: -0.05em;
  --tracking-tight:   -0.025em;
  --tracking-normal:  0;
  --tracking-wide:    0.025em;
  --tracking-wider:   0.05em;
  --tracking-widest:  0.1em;
  
  /* Families */
  --font-sans:  'Inter', ui-sans-serif, system-ui, sans-serif;
  --font-mono:  'JetBrains Mono', ui-monospace, 'Cascadia Code', monospace;
  --font-serif: 'Georgia', ui-serif, serif;
  
  /* ===== TIER 2: SEMANTIC ===== */
  --text-display-size:    var(--font-size-5xl);
  --text-display-weight:  var(--font-weight-bold);
  --text-display-leading: var(--leading-tight);
  
  --text-h1-size:    var(--font-size-4xl);
  --text-h1-weight:  var(--font-weight-bold);
  --text-h1-leading: var(--leading-tight);
  
  --text-h2-size:    var(--font-size-3xl);
  --text-h2-weight:  var(--font-weight-semibold);
  --text-h2-leading: var(--leading-snug);
  
  --text-h3-size:    var(--font-size-2xl);
  --text-h3-weight:  var(--font-weight-semibold);
  --text-h3-leading: var(--leading-snug);
  
  --text-h4-size:    var(--font-size-xl);
  --text-h4-weight:  var(--font-weight-medium);
  --text-h4-leading: var(--leading-snug);
  
  --text-body-size:    var(--font-size-base);
  --text-body-weight:  var(--font-weight-regular);
  --text-body-leading: var(--leading-relaxed);
  
  --text-small-size:    var(--font-size-sm);
  --text-small-leading: var(--leading-normal);
  
  --text-caption-size:    var(--font-size-xs);
  --text-caption-leading: var(--leading-normal);
  
  --text-label-size:    var(--font-size-sm);
  --text-label-weight:  var(--font-weight-medium);
  --text-label-leading: var(--leading-none);
  
  --text-code-size:   var(--font-size-sm);
  --text-code-family: var(--font-mono);
}
```

### Border & Radius Tokens

```css
:root {
  /* ===== TIER 1 ===== */
  --radius-none: 0;
  --radius-sm:   4px;
  --radius-md:   6px;
  --radius-lg:   8px;
  --radius-xl:   12px;
  --radius-2xl:  16px;
  --radius-3xl:  24px;
  --radius-full: 9999px; /* pill */
  
  --border-width-thin:   1px;
  --border-width-medium: 2px;
  --border-width-thick:  4px;
  
  /* ===== TIER 2 ===== */
  --radius-component:     var(--radius-md);  /* inputs, buttons */
  --radius-card:          var(--radius-lg);  /* cards, modals */
  --radius-popover:       var(--radius-lg);  /* dropdowns, popovers */
  --radius-badge:         var(--radius-full);/* badges, chips */
  --radius-avatar:        var(--radius-full);/* avatars */
}
```

### Shadow Tokens

```css
:root {
  /* ===== TIER 1 ===== */
  --shadow-xs: 0 1px 2px 0 rgb(0 0 0 / 0.05);
  --shadow-sm: 0 1px 3px 0 rgb(0 0 0 / 0.1), 0 1px 2px -1px rgb(0 0 0 / 0.1);
  --shadow-md: 0 4px 6px -1px rgb(0 0 0 / 0.1), 0 2px 4px -2px rgb(0 0 0 / 0.1);
  --shadow-lg: 0 10px 15px -3px rgb(0 0 0 / 0.1), 0 4px 6px -4px rgb(0 0 0 / 0.1);
  --shadow-xl: 0 20px 25px -5px rgb(0 0 0 / 0.1), 0 8px 10px -6px rgb(0 0 0 / 0.1);
  --shadow-2xl:0 25px 50px -12px rgb(0 0 0 / 0.25);
  --shadow-inner: inset 0 2px 4px 0 rgb(0 0 0 / 0.05);
  --shadow-none: 0 0 #0000;
  
  /* ===== TIER 2 ===== */
  --shadow-card:    var(--shadow-sm);
  --shadow-popover: var(--shadow-xl);
  --shadow-modal:   var(--shadow-2xl);
  --shadow-dropdown:var(--shadow-md);
  --shadow-button:  var(--shadow-xs);
}
```

### Motion Tokens

```css
:root {
  /* ===== TIER 1: PRIMITIVES ===== */
  --duration-0:    0ms;
  --duration-75:   75ms;
  --duration-100:  100ms;
  --duration-150:  150ms;
  --duration-200:  200ms;
  --duration-300:  300ms;
  --duration-500:  500ms;
  --duration-700:  700ms;
  --duration-1000: 1000ms;
  
  --ease-linear:   linear;
  --ease-in:       cubic-bezier(0.4, 0, 1, 1);
  --ease-out:      cubic-bezier(0, 0, 0.2, 1);
  --ease-in-out:   cubic-bezier(0.4, 0, 0.2, 1);
  --ease-spring:   cubic-bezier(0.34, 1.56, 0.64, 1);
  
  /* ===== TIER 2: SEMANTIC ===== */
  --duration-micro:      var(--duration-75);   /* button press */
  --duration-fast:       var(--duration-150);  /* hover, simple fade */
  --duration-base:       var(--duration-200);  /* standard transition */
  --duration-slow:       var(--duration-300);  /* dropdown, modal open */
  --duration-page:       var(--duration-300);  /* page transitions */
  
  --ease-enter:   var(--ease-out);   /* elements entering */
  --ease-exit:    var(--ease-in);    /* elements leaving */
  --ease-move:    var(--ease-in-out);/* elements repositioning */
  --ease-bounce:  var(--ease-spring);/* playful interactions */
}
```

### Z-Index Tokens

```css
:root {
  --z-below:    -1;
  --z-base:      0;
  --z-raised:    1;
  --z-sticky:    20;
  --z-fixed:     30;
  --z-overlay:   40;
  --z-modal:     50;
  --z-popover:   60;
  --z-toast:     70;
  --z-tooltip:   80;
  --z-maximum:   9999;
}
```

---

## Token Naming Conventions

```
Pattern: --[category]-[property]-[variant]-[state]

Examples:
  --color-text-muted                    ✅
  --color-border-strong                 ✅
  --space-component-lg                  ✅
  --duration-base                       ✅
  --radius-card                         ✅

Bad examples:
  --blueButton                          ❌ (not semantic, camelCase)
  --color-blue-primary-hover-disabled   ❌ (too specific, too long)
  --p-24                                ❌ (not descriptive, abbreviation)
  --bigRadius                           ❌ (subjective, not systematic)
```

---

## Tokens in Tailwind Config

```javascript
// tailwind.config.ts
import type { Config } from 'tailwindcss';

const config: Config = {
  content: ['./src/**/*.{ts,tsx}'],
  darkMode: 'class',
  theme: {
    extend: {
      colors: {
        // Reference CSS custom properties
        background:  'hsl(var(--background))',
        foreground:  'hsl(var(--foreground))',
        primary: {
          DEFAULT:    'hsl(var(--primary))',
          foreground: 'hsl(var(--primary-foreground))',
        },
        secondary: {
          DEFAULT:    'hsl(var(--secondary))',
          foreground: 'hsl(var(--secondary-foreground))',
        },
        destructive: {
          DEFAULT:    'hsl(var(--destructive))',
          foreground: 'hsl(var(--destructive-foreground))',
        },
        muted: {
          DEFAULT:    'hsl(var(--muted))',
          foreground: 'hsl(var(--muted-foreground))',
        },
        accent: {
          DEFAULT:    'hsl(var(--accent))',
          foreground: 'hsl(var(--accent-foreground))',
        },
        border:  'hsl(var(--border))',
        input:   'hsl(var(--input))',
        ring:    'hsl(var(--ring))',
        card: {
          DEFAULT:    'hsl(var(--card))',
          foreground: 'hsl(var(--card-foreground))',
        },
      },
      borderRadius: {
        lg: 'var(--radius)',
        md: 'calc(var(--radius) - 2px)',
        sm: 'calc(var(--radius) - 4px)',
      },
      fontFamily: {
        sans: ['Inter', 'ui-sans-serif', 'system-ui', 'sans-serif'],
        mono: ['JetBrains Mono', 'ui-monospace', 'monospace'],
      },
      animation: {
        'enter':    'enter 200ms ease-out',
        'exit':     'exit 150ms ease-in',
        'shimmer':  'shimmer 1.5s linear infinite',
        'spin-slow':'spin 2s linear infinite',
      },
      keyframes: {
        enter: {
          from: { opacity: '0', transform: 'translateY(8px)' },
          to:   { opacity: '1', transform: 'translateY(0)' },
        },
        exit: {
          from: { opacity: '1', transform: 'translateY(0)' },
          to:   { opacity: '0', transform: 'translateY(8px)' },
        },
        shimmer: {
          from: { backgroundPosition: '200% center' },
          to:   { backgroundPosition: '-200% center' },
        },
      },
    },
  },
  plugins: [],
};

export default config;
```

---

## Token Checklist

- [ ] All raw values defined at Tier 1 (primitives)
- [ ] All component styles reference Tier 2 (semantic) tokens
- [ ] Dark mode overrides only override Tier 2 tokens (not Tier 1)
- [ ] No hardcoded hex colors in component styles
- [ ] No arbitrary spacing values — all on token scale
- [ ] Tailwind config references CSS custom properties (not hardcoded values)
- [ ] Token names are semantic (describe purpose, not appearance)
- [ ] All motion durations are tokenized
- [ ] Z-index values use token scale (no magic numbers)
