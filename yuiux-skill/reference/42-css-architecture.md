# 42 — CSS Architecture

> CSS at scale is a discipline. Without a system, it collapses into specificity wars, !important nightmares, and styles nobody dares to delete.

---

## CSS Architecture Options

```
BEM          — Block Element Modifier, class-based naming
Utility-first — Tailwind CSS, write styles in markup
CSS Modules  — scoped CSS, import in JS
CSS-in-JS    — styled-components, Emotion, Panda CSS
Layered CSS  — @layer cascade management
```

**Recommendation for new projects:**
- Lovable / Next.js / Vite + React: **Tailwind CSS** (utility-first)
- Complex custom components: **CSS Modules** alongside Tailwind
- Design systems: **CSS custom properties + design tokens**

---

## Tailwind CSS Architecture

```
src/
├── styles/
│   ├── globals.css     — @tailwind directives + CSS variables + base resets
│   ├── typography.css  — prose styles for content
│   └── animations.css  — custom keyframes
├── lib/
│   └── utils.ts        — cn() utility
└── tailwind.config.ts  — theme extension
```

### globals.css

```css
/* globals.css */
@tailwind base;
@tailwind components;
@tailwind utilities;

/* ===== BASE LAYER — Resets and base styles ===== */
@layer base {
  *, *::before, *::after {
    box-sizing: border-box;
    margin: 0;
    padding: 0;
  }

  html {
    scroll-behavior: smooth;
    -webkit-text-size-adjust: 100%;
    tab-size: 4;
  }

  @media (prefers-reduced-motion: reduce) {
    html { scroll-behavior: auto; }
    *, *::before, *::after {
      animation-duration: 0.01ms !important;
      animation-iteration-count: 1 !important;
      transition-duration: 0.01ms !important;
    }
  }

  body {
    font-family: var(--font-sans);
    font-size: var(--text-body-size, 16px);
    line-height: var(--text-body-leading, 1.625);
    color: hsl(var(--foreground));
    background: hsl(var(--background));
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
  }

  /* Typography base styles */
  h1, h2, h3, h4, h5, h6 {
    font-weight: 600;
    line-height: 1.25;
    text-wrap: balance;
  }

  p, li { text-wrap: pretty; }

  /* Accessible focus styles */
  :focus-visible {
    outline: 2px solid hsl(var(--ring));
    outline-offset: 2px;
    border-radius: 2px;
  }

  /* Remove default focus styles for mouse users */
  :focus:not(:focus-visible) { outline: none; }

  /* Images */
  img, video {
    max-width: 100%;
    height: auto;
    display: block;
  }

  /* Interactive elements */
  button, [role="button"] { cursor: pointer; }
  button:disabled, [aria-disabled="true"] { cursor: not-allowed; }

  /* Form elements */
  input, textarea, select {
    font: inherit;
    color: inherit;
  }

  /* Remove list styles */
  ul[role="list"], ol[role="list"] { list-style: none; }

  /* Selection */
  ::selection {
    background: hsl(var(--primary) / 0.2);
    color: hsl(var(--foreground));
  }

  /* Scrollbar styling (Webkit) */
  ::-webkit-scrollbar { width: 6px; height: 6px; }
  ::-webkit-scrollbar-track { background: transparent; }
  ::-webkit-scrollbar-thumb {
    background: hsl(var(--border));
    border-radius: 3px;
  }
  ::-webkit-scrollbar-thumb:hover {
    background: hsl(var(--muted-foreground));
  }
}

/* ===== CSS CUSTOM PROPERTIES (Design Tokens) ===== */
:root {
  /* Colors — HSL values for Tailwind opacity modifier support */
  --background:       0 0% 100%;
  --foreground:       222.2 84% 4.9%;
  --card:             0 0% 100%;
  --card-foreground:  222.2 84% 4.9%;
  --primary:          222.2 47.4% 11.2%;
  --primary-foreground: 210 40% 98%;
  --secondary:        210 40% 96.1%;
  --secondary-foreground: 222.2 47.4% 11.2%;
  --muted:            210 40% 96.1%;
  --muted-foreground: 215.4 16.3% 46.9%;
  --accent:           210 40% 96.1%;
  --accent-foreground: 222.2 47.4% 11.2%;
  --destructive:      0 84.2% 60.2%;
  --destructive-foreground: 210 40% 98%;
  --border:           214.3 31.8% 91.4%;
  --input:            214.3 31.8% 91.4%;
  --ring:             222.2 84% 4.9%;
  --radius:           0.5rem;

  /* Typography */
  --font-sans: 'Inter', ui-sans-serif, system-ui, sans-serif;
  --font-mono: 'JetBrains Mono', ui-monospace, monospace;

  /* Spacing scale */
  --space-1: 4px; --space-2: 8px;  --space-3: 12px;
  --space-4: 16px; --space-6: 24px; --space-8: 32px;
  --space-12: 48px; --space-16: 64px; --space-24: 96px;
}

.dark {
  --background:       222.2 84% 4.9%;
  --foreground:       210 40% 98%;
  --card:             222.2 84% 4.9%;
  --card-foreground:  210 40% 98%;
  --primary:          210 40% 98%;
  --primary-foreground: 222.2 47.4% 11.2%;
  --secondary:        217.2 32.6% 17.5%;
  --secondary-foreground: 210 40% 98%;
  --muted:            217.2 32.6% 17.5%;
  --muted-foreground: 215 20.2% 65.1%;
  --accent:           217.2 32.6% 17.5%;
  --accent-foreground: 210 40% 98%;
  --destructive:      0 62.8% 30.6%;
  --destructive-foreground: 210 40% 98%;
  --border:           217.2 32.6% 17.5%;
  --input:            217.2 32.6% 17.5%;
  --ring:             212.7 26.8% 83.9%;
}
```

---

## CSS Layers (@layer)

CSS layers give you explicit control over cascade order — solving specificity wars:

```css
/* Define layer order — lowest → highest priority */
@layer reset, tokens, base, components, utilities, overrides;

@layer reset {
  *, *::before, *::after { box-sizing: border-box; margin: 0; }
}

@layer tokens {
  :root {
    --color-primary: #2563eb;
    --space-4: 16px;
  }
}

@layer base {
  body { font-family: var(--font-sans); }
  h1 { font-size: 2rem; }
}

@layer components {
  .btn {
    display: inline-flex;
    align-items: center;
    padding: 8px 16px;
    border-radius: var(--radius);
  }
  
  .btn-primary {
    background: var(--color-primary);
    color: white;
  }
}

@layer utilities {
  .sr-only {
    position: absolute; width: 1px; height: 1px;
    padding: 0; margin: -1px; overflow: hidden;
    clip: rect(0, 0, 0, 0); white-space: nowrap; border: 0;
  }
  
  .not-sr-only {
    position: static; width: auto; height: auto;
    padding: 0; margin: 0; overflow: visible;
    clip: auto; white-space: normal;
  }
}

@layer overrides {
  /* Intentional overrides — site-specific exceptions */
  .landing-hero .btn { font-size: 1.125rem; padding: 12px 24px; }
}
```

---

## CSS Modules (with TypeScript)

```typescript
// Button.module.css
.root {
  display: inline-flex;
  align-items: center;
  gap: 8px;
  padding: 8px 16px;
  border-radius: 6px;
  font-size: 14px;
  font-weight: 500;
  cursor: pointer;
  transition: background 150ms ease, color 150ms ease, box-shadow 150ms ease;
}

.root:focus-visible {
  outline: 2px solid var(--color-primary);
  outline-offset: 2px;
}

.root:disabled {
  opacity: 0.5;
  pointer-events: none;
}

/* Variants via composes */
.primary {
  composes: root;
  background: var(--color-primary);
  color: white;
}

.primary:hover  { background: var(--color-primary-hover); }
.primary:active { background: var(--color-primary-active); }

.secondary {
  composes: root;
  background: var(--color-secondary);
  color: var(--color-text);
  border: 1px solid var(--color-border);
}

/* Sizes */
.sm { padding: 4px 12px; font-size: 13px; }
.lg { padding: 12px 24px; font-size: 16px; }
```

```tsx
// Button.tsx — importing CSS Module
import styles from './Button.module.css';
import clsx from 'clsx';

interface ButtonProps extends React.ButtonHTMLAttributes<HTMLButtonElement> {
  variant?: 'primary' | 'secondary';
  size?: 'sm' | 'md' | 'lg';
}

export function Button({ variant = 'primary', size = 'md', className, ...props }: ButtonProps) {
  return (
    <button
      {...props}
      className={clsx(
        styles[variant],
        size !== 'md' && styles[size],
        className
      )}
    />
  );
}
```

---

## Specificity Management

```css
/* Specificity scale */
:where(html) { /* 0-0-0 — no specificity, easily overridden */ }
element      { /* 0-0-1 — base styles */ }
.class       { /* 0-1-0 — utility and component styles */ }
#id          { /* 1-0-0 — avoid except for document structure */ }
!important   { /* nuclear — only for accessibility overrides */ }

/* ✅ Use :where() to apply resets without adding specificity */
:where(h1, h2, h3, h4, h5, h6) {
  font-weight: 600;
  /* This has 0 specificity — any .class rule overrides it */
}

/* ✅ Use :is() to group selectors without maximum specificity */
:is(ul, ol)[role="list"] {
  list-style: none;
  /* Specificity = highest selector in list (1 element) */
}

/* ❌ Deep nesting — creates high specificity */
.nav .nav-item .nav-link .nav-link-icon { color: blue; }
/* Specificity: 0-3-1 — hard to override */

/* ✅ Flat naming */
.nav-link-icon { color: blue; }
/* Specificity: 0-1-0 — easy to override */
```

---

## Container Queries

```css
/* Style based on parent container size, not viewport */
.card-container {
  container-type: inline-size;
  container-name: card;
}

@container card (min-width: 400px) {
  .card-layout {
    display: grid;
    grid-template-columns: auto 1fr;
    gap: 16px;
  }
}

@container card (max-width: 399px) {
  .card-layout {
    display: flex;
    flex-direction: column;
  }
}

/* Container queries for components — more flexible than media queries */
.sidebar-container {
  container-type: inline-size;
}

@container (min-width: 300px) {
  .widget-title { font-size: 1.25rem; }
  .widget-actions { display: flex; }
}

@container (max-width: 299px) {
  .widget-title { font-size: 1rem; }
  .widget-actions { display: none; }
}
```

---

## Modern CSS Features

```css
/* Subgrid — align nested items to parent grid */
.card-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(280px, 1fr));
  gap: 24px;
}

.card {
  display: grid;
  grid-template-rows: subgrid; /* inherit parent grid rows */
  grid-row: span 3;
}

/* :has() — parent selector */
/* Card with an image — style parent based on children */
.card:has(img) { padding: 0; }
.card:has(img) .card-content { padding: 16px; }

/* Form field that has an error */
.form-field:has(.field-error) label { color: hsl(var(--destructive)); }
.form-field:has(.field-error) input { border-color: hsl(var(--destructive)); }

/* Nav item that is current */
.nav-item:has([aria-current="page"]) { background: hsl(var(--accent)); }

/* :is() — group selectors */
:is(h1, h2, h3, h4, h5, h6) { line-height: 1.25; }
:is(button, [role="button"]):disabled { opacity: 0.5; }

/* Nesting (native CSS) */
.button {
  padding: 8px 16px;
  background: hsl(var(--primary));

  &:hover { background: hsl(var(--primary) / 0.9); }
  &:focus-visible { outline: 2px solid hsl(var(--ring)); }
  &:disabled { opacity: 0.5; }

  & .button-icon { width: 16px; height: 16px; }
  &.button--sm { padding: 4px 12px; font-size: 13px; }
  &.button--lg { padding: 12px 24px; font-size: 16px; }
}
```

---

## Performance: Critical CSS

```html
<!-- Critical CSS inline — above-fold styles -->
<head>
  <style id="critical-css">
    /* Minimal styles needed for first render */
    *, *::before, *::after { box-sizing: border-box; }
    body { margin: 0; font-family: system-ui, sans-serif; }
    .hero {
      min-height: 100dvh;
      display: flex;
      align-items: center;
      justify-content: center;
    }
    .hero-title { font-size: clamp(2rem, 5vw, 4rem); font-weight: 700; }
    /* ~1-3KB of critical styles */
  </style>

  <!-- Defer non-critical CSS -->
  <link
    rel="preload"
    href="/styles/main.css"
    as="style"
    onload="this.onload=null;this.rel='stylesheet'"
  />
  <noscript><link rel="stylesheet" href="/styles/main.css"></noscript>
</head>
```

```bash
# Extract critical CSS automatically
npx critical 'https://yoursite.com' \
  --inline \
  --width 1300 \
  --height 900 \
  --output dist/index.html
```

---

## CSS Architecture Checklist

- [ ] Design tokens defined as CSS custom properties in `:root`
- [ ] Dark mode tokens in `.dark` or `[data-theme="dark"]` override
- [ ] No `!important` except in accessibility utilities (`.sr-only`)
- [ ] No ID selectors for styling (only for JS targeting)
- [ ] Specificity kept low — class-level max
- [ ] Tailwind semantic classes used (not arbitrary values)
- [ ] CSS layers (`@layer`) defined for large codebases
- [ ] No hardcoded colors — all reference CSS variables
- [ ] `prefers-reduced-motion` respected in all animations
- [ ] Container queries used for component-level responsiveness
- [ ] `:focus-visible` used (not `:focus`) for focus styles
- [ ] `-webkit-font-smoothing: antialiased` on body
- [ ] Critical CSS inlined for initial render
- [ ] Non-critical CSS deferred with preload trick
- [ ] Scrollbar styled for consistency (optional but nice)
