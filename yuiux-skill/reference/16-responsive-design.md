# 16 — Responsive Design

> Responsive design is not about making things "fit" on mobile. It's about designing the optimal experience for each context.

---

## Mobile-First Methodology

Write base styles for the smallest screen, then progressively enhance for larger screens:

```css
/* ✅ Mobile-first: base = mobile, then enhance */
.component {
  display: block;         /* single column on mobile */
  padding: 16px;
}

@media (min-width: 768px) {
  .component {
    display: flex;        /* row layout on tablet */
    padding: 24px;
  }
}

@media (min-width: 1024px) {
  .component {
    padding: 32px;
    gap: 48px;
  }
}

/* ❌ Desktop-first: harder to reason about, leads to regressions */
.component {
  display: flex;
  padding: 32px;
}
@media (max-width: 768px) {
  .component { display: block; padding: 16px; }
}
```

---

## Breakpoint System

```css
/* Tailwind-aligned breakpoints */
/* xs: 480px — large phones */
/* sm: 640px — small tablets */
/* md: 768px — tablets */
/* lg: 1024px — small desktops */
/* xl: 1280px — desktops */
/* 2xl: 1536px — large screens */

/* In JavaScript */
const breakpoints = {
  xs:  480,
  sm:  640,
  md:  768,
  lg:  1024,
  xl:  1280,
  '2xl': 1536,
} as const;

/* CSS custom media queries (not yet standard, but useful for reference) */
:root {
  --bp-xs:  480px;
  --bp-sm:  640px;
  --bp-md:  768px;
  --bp-lg:  1024px;
  --bp-xl:  1280px;
  --bp-2xl: 1536px;
}
```

### React Breakpoint Hook

```tsx
function useBreakpoint() {
  const [bp, setBp] = useState<'xs' | 'sm' | 'md' | 'lg' | 'xl' | '2xl'>('md');
  
  useEffect(() => {
    const queries = [
      { bp: '2xl' as const, mq: window.matchMedia('(min-width: 1536px)') },
      { bp: 'xl'  as const, mq: window.matchMedia('(min-width: 1280px)') },
      { bp: 'lg'  as const, mq: window.matchMedia('(min-width: 1024px)') },
      { bp: 'md'  as const, mq: window.matchMedia('(min-width: 768px)')  },
      { bp: 'sm'  as const, mq: window.matchMedia('(min-width: 640px)')  },
      { bp: 'xs'  as const, mq: window.matchMedia('(min-width: 480px)')  },
    ];
    
    function update() {
      const match = queries.find(({ mq }) => mq.matches);
      setBp(match?.bp ?? 'xs');
    }
    
    update();
    queries.forEach(({ mq }) => mq.addEventListener('change', update));
    return () => queries.forEach(({ mq }) => mq.removeEventListener('change', update));
  }, []);
  
  return {
    bp,
    isMobile:  ['xs'].includes(bp),
    isTablet:  ['sm', 'md'].includes(bp),
    isDesktop: ['lg', 'xl', '2xl'].includes(bp),
    gte: (target: keyof typeof breakpoints) => {
      const order = ['xs', 'sm', 'md', 'lg', 'xl', '2xl'];
      return order.indexOf(bp) >= order.indexOf(target);
    },
  };
}
```

---

## Fluid Typography with clamp()

```css
/* clamp(minimum, preferred, maximum) */

/* Heading — 24px on mobile, scales to 48px on wide screens */
h1 { font-size: clamp(24px, 3.5vw + 16px, 48px); }
h2 { font-size: clamp(20px, 2.5vw + 14px, 36px); }
h3 { font-size: clamp(18px, 1.8vw + 12px, 28px); }
h4 { font-size: clamp(16px, 1.2vw + 12px, 22px); }

/* Body — stays stable */
body { font-size: clamp(15px, 1vw + 12px, 17px); }

/* Hero display */
.hero-title { font-size: clamp(32px, 6vw + 12px, 80px); }
```

### CSS clamp() Calculator Formula

```
clamp(min, preferred, max)

preferred formula: val-at-sm + (val-at-lg - val-at-sm) / (bp-lg - bp-sm) * (100vw - bp-sm)

For font-size 16px at 320px → 20px at 1280px:
preferred = 16 + (20-16) / (1280-320) * (100vw - 320px)
          = 16 + 4/960 * (100vw - 320px)
          = 16 + 0.42vw + (-1.33px)
          ≈ 0.42vw + 14.67px

Result: clamp(16px, 0.42vw + 14.67px, 20px)
```

---

## Fluid Spacing

```css
/* Section padding that scales with viewport */
.section {
  padding-block: clamp(48px, 8vw, 96px);
  padding-inline: clamp(16px, 4vw, 48px);
}

/* Container with fluid horizontal padding */
.container {
  width: 100%;
  max-width: 1280px;
  margin-inline: auto;
  padding-inline: clamp(16px, 4vw, 32px);
}

/* Gap that scales */
.card-grid {
  gap: clamp(16px, 2vw, 32px);
}
```

---

## Image Responsiveness

```html
<!-- Responsive image with srcset -->
<img
  src="image-800.webp"
  srcset="
    image-400.webp  400w,
    image-800.webp  800w,
    image-1200.webp 1200w
  "
  sizes="
    (max-width: 640px) 100vw,
    (max-width: 1024px) 50vw,
    33vw
  "
  alt="Description of image"
  loading="lazy"
  decoding="async"
/>

<!-- Art direction with picture element -->
<picture>
  <!-- Mobile: square crop -->
  <source
    media="(max-width: 640px)"
    srcset="image-mobile-400.webp 400w, image-mobile-800.webp 800w"
    sizes="100vw"
  />
  <!-- Desktop: wide crop -->
  <source
    srcset="image-wide-800.webp 800w, image-wide-1600.webp 1600w"
    sizes="(max-width: 1280px) 100vw, 1280px"
  />
  <img src="image-wide-800.webp" alt="Description" loading="lazy" />
</picture>
```

```css
/* Always set object-fit on images in containers */
.card-image img {
  width: 100%;
  height: 100%;
  object-fit: cover;   /* crop to fill */
  object-position: center;
}

/* Aspect ratio containers */
.image-4-3  { aspect-ratio: 4/3; }
.image-16-9 { aspect-ratio: 16/9; }
.image-1-1  { aspect-ratio: 1; }
.image-3-4  { aspect-ratio: 3/4; }
```

---

## Layout Patterns for Responsiveness

### Holy Grail (Header, Sidebar, Main, Footer)

```css
.app {
  display: grid;
  grid-template-rows: auto 1fr auto;
  grid-template-columns: 1fr;
  min-height: 100dvh;
}

@media (min-width: 1024px) {
  .app {
    grid-template-columns: 240px 1fr;
    grid-template-rows: auto 1fr auto;
    grid-template-areas:
      "header header"
      "sidebar main"
      "footer footer";
  }
  .app-header  { grid-area: header; }
  .app-sidebar { grid-area: sidebar; display: block; } /* un-hide */
  .app-main    { grid-area: main; }
  .app-footer  { grid-area: footer; }
}

/* Hide sidebar on mobile (show via hamburger) */
.app-sidebar {
  display: none;
  position: fixed;
  inset: 0;
  z-index: var(--z-fixed);
  background: hsl(var(--background));
  width: 280px;
  overflow-y: auto;
}
.app-sidebar.open { display: block; }
```

### Flex-based Responsive Row→Column

```css
.feature-row {
  display: flex;
  flex-direction: column;
  gap: 32px;
}

@media (min-width: 768px) {
  .feature-row {
    flex-direction: row;
    align-items: center;
    gap: 48px;
  }
  
  .feature-row--reverse { flex-direction: row-reverse; }
  
  .feature-image { flex: 1; }
  .feature-content { flex: 1; }
}
```

---

## Viewport Units

```css
/* Use dvh instead of vh to account for mobile browser chrome */
.hero { min-height: 100dvh; }          /* dynamic viewport height */
.modal { max-height: calc(100dvh - 48px); }

/* svh = smallest viewport height (when browser chrome is visible) */
/* lvh = largest viewport height (when browser chrome is hidden) */
/* dvh = dynamic — changes as browser chrome shows/hides */

/* Use dvw sparingly (can cause horizontal layout issues) */
```

---

## Touch-Friendly Responsive Adjustments

```css
/* Larger touch targets on mobile */
@media (pointer: coarse) {
  /* User is on a touch device */
  .nav-link { 
    padding: 12px 16px; /* larger on touch */
    min-height: 44px;
  }
  
  /* Don't show hover effects on touch */
  .card--interactive:hover {
    transform: none;
    box-shadow: var(--shadow-sm);
  }
}

@media (pointer: fine) {
  /* User has a mouse — can show hover effects */
  .card--interactive:hover {
    transform: translateY(-2px);
    box-shadow: var(--shadow-md);
  }
}

/* Hover effects only when device supports hover */
@media (hover: hover) {
  .card--interactive:hover {
    /* hover effects here */
  }
}
```

---

## Print Styles

```css
@media print {
  /* Hide navigation, actions, ads */
  nav, aside, .ad, .btn, .skip-link, .toast-container {
    display: none !important;
  }
  
  /* Ensure text is black on white */
  body { color: black; background: white; }
  
  /* Expand links to show URLs */
  a[href]:after { content: " (" attr(href) ")"; font-size: 0.8em; }
  
  /* Avoid breaking inside cards */
  .card { break-inside: avoid; }
  
  /* Show borders instead of shadows (shadows don't print well) */
  .card { box-shadow: none; border: 1px solid #ccc; }
  
  /* Expand all collapsed sections for print */
  details, [hidden] { display: block !important; }
}
```

---

## Responsive Checklist

- [ ] Mobile-first CSS (base styles = mobile, @media min-width for larger)
- [ ] Tested at 320px, 375px, 768px, 1024px, 1280px widths
- [ ] No horizontal scroll at any tested width
- [ ] Touch targets ≥ 44×44px on mobile
- [ ] Images use `srcset` and `sizes` for responsive loading
- [ ] Images use `aspect-ratio` to prevent layout shift while loading
- [ ] Fluid typography with `clamp()` for headings
- [ ] `dvh` used instead of `vh` for full-height elements
- [ ] Hover effects gated behind `@media (hover: hover)`
- [ ] Container has appropriate `max-width` and fluid `padding-inline`
- [ ] Sidebar collapses on mobile with hamburger menu
- [ ] Print styles defined for content-heavy pages
- [ ] No `position: fixed` elements overlap content without scroll workaround
