# 02 — Typography & Readability

> Type is the interface. 95% of web design is typography. Every rule here serves one goal: text the user can read without effort.

---

## The One Absolute Rule

**Body text must be readable without effort.** If a user has to slow down, squint, or zoom to read content, the design has failed. Every rule below enforces this.

---

## Type Scale

### Recommended Scale: Major Third (1.25×)

```
12px  — caption, badge, micro-label
14px  — helper text, secondary body, table cell
16px  — BASE body text (minimum for main content)
20px  — lead paragraph, intro text
25px  — h4, card title
31px  — h3
39px  — h2
49px  — h1
61px  — display
76px  — hero
```

### Alternative: Tailwind Default Scale
```
text-xs    12px  — captions
text-sm    14px  — helper, secondary
text-base  16px  — body (default)
text-lg    18px  — comfortable body
text-xl    20px  — lead
text-2xl   24px  — h4
text-3xl   30px  — h3
text-4xl   36px  — h2
text-5xl   48px  — h1
text-6xl   60px  — display
text-7xl   72px  — hero
text-8xl   96px  — billboard
```

### CSS Custom Properties for Type Scale

```css
:root {
  --text-xs: 0.75rem;    /* 12px */
  --text-sm: 0.875rem;   /* 14px */
  --text-base: 1rem;     /* 16px */
  --text-lg: 1.125rem;   /* 18px */
  --text-xl: 1.25rem;    /* 20px */
  --text-2xl: 1.5rem;    /* 24px */
  --text-3xl: 1.875rem;  /* 30px */
  --text-4xl: 2.25rem;   /* 36px */
  --text-5xl: 3rem;      /* 48px */
  --text-6xl: 3.75rem;   /* 60px */
  --text-7xl: 4.5rem;    /* 72px */
}
```

---

## Font Size Rules

### Minimum Sizes
| Context | Minimum Size | Recommended |
|---------|-------------|-------------|
| Main body text | 16px | 16–18px |
| Mobile body text | 15px | 16px |
| Helper/secondary | 13px | 14px |
| Captions/badges | 11px | 12px |
| Absolute minimum | 11px | never below 11px |

**Why 16px?**
- Default browser size — users have set this as their expected base
- Below 16px causes physical strain for average-vision users at arm's length
- WCAG 1.4.4 requires text scalable to 200% — below 16px means 8px at 50%

### Size Hierarchy Rules
- Never use the same size for elements of different importance
- Establish hierarchy through size difference of at least 2 steps (not 16px vs 17px)
- Minimum size difference between adjacent hierarchy levels: 4px
- H1 should be at least 2× body size in display contexts

---

## Line Height (Leading)

| Text Type | Line Height | CSS Value |
|-----------|------------|-----------|
| Body text | 1.5–1.7 | `line-height: 1.6` |
| Long-form content | 1.6–1.8 | `line-height: 1.7` |
| Headings | 1.1–1.25 | `line-height: 1.15` |
| Navigation links | 1.0–1.2 | `line-height: 1.1` |
| Captions | 1.4–1.6 | `line-height: 1.5` |
| Code/monospace | 1.5–1.6 | `line-height: 1.6` |

**Why it matters:** Too tight (< 1.4) → lines merge visually. Too loose (> 2.0) → paragraphs feel disconnected. The eye needs white space to track line to line without losing its place.

```css
/* Base body */
body {
  line-height: 1.6;
  font-size: 1rem;
}

/* Headings — tighter because large text needs less leading */
h1, h2, h3 {
  line-height: 1.15;
}

/* Small text needs more breathing room proportionally */
.caption {
  font-size: 0.75rem;
  line-height: 1.5;
}
```

---

## Line Length (Measure)

**The optimal line length for body text: 45–75 characters (50–65ch ideal)**

Too short (<30ch): Too many line breaks interrupt reading flow
Too long (>80ch): Eye loses its place when returning to next line

```css
/* Constrain reading line length */
.prose {
  max-width: 65ch; /* ideal for body text */
}

/* Slightly wider for UI text */
.description {
  max-width: 75ch;
}

/* Never constrain headings — they should span their container */
h1, h2 {
  max-width: none; /* or omit */
}
```

### Line Length in Different Contexts
| Content Type | Recommended |
|-------------|-------------|
| Long-form article/blog | 55–65ch |
| UI descriptions/help text | 45–60ch |
| Email body | 50–60ch |
| Cards | 30–50ch |
| Captions | 25–40ch |
| Narrow columns | 25–35ch |

---

## Font Weight

```
100 — Thin (avoid for body; use sparingly for display)
200 — ExtraLight (avoid for body)
300 — Light (body text in display-only contexts)
400 — Regular (standard body text)
500 — Medium (UI labels, navigation, form labels)
600 — SemiBold (subheadings, card titles, emphasis)
700 — Bold (headings, strong emphasis, CTAs)
800 — ExtraBold (large display headlines)
900 — Black/Heavy (billboard, hero statements)
```

**Rules:**
- Body text: 400 (Regular)
- Form labels: 500 (Medium)
- Subheadings: 600 (SemiBold)
- Main headings: 700 (Bold)
- Never use 100–300 for text below 20px
- Never use 800–900 below 30px (chunky and hard to read)

---

## Font Pairing

### Principle: Contrast + Harmony
A good pairing has clear contrast (so the two fonts don't fight) but shared proportions (so they look related).

### The Three Reliable Systems

**1. Serif heading + Sans-serif body** (classic editorial)
```css
--font-heading: 'Playfair Display', Georgia, serif;
--font-body: 'Inter', system-ui, sans-serif;
```

**2. Display sans + Regular sans** (contemporary tech)
```css
--font-heading: 'Plus Jakarta Sans', sans-serif;
--font-body: 'Inter', system-ui, sans-serif;
```

**3. System UI (max performance)**
```css
--font-body: system-ui, -apple-system, BlinkMacSystemFont, 
             'Segoe UI', Roboto, 'Helvetica Neue', Arial, sans-serif;
/* No heading font — use weight and size for hierarchy */
```

### Curated Pairings

| Heading | Body | Feel |
|---------|------|------|
| Playfair Display | Inter | Elegant editorial |
| Fraunces | DM Sans | Literary warmth |
| Plus Jakarta Sans | Inter | Modern SaaS |
| Syne | Manrope | Creative/agency |
| Outfit | Nunito | Friendly/consumer |
| Space Grotesk | Inter | Technical/developer |
| Cormorant Garamond | Lato | Luxury fashion |
| Abril Fatface | Lato | Bold consumer |
| Raleway | Open Sans | Clean professional |

### Loading Fonts Correctly

```html
<!-- In HTML head - preconnect first, then load -->
<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
<link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&family=Plus+Jakarta+Sans:wght@600;700;800&display=swap" rel="stylesheet">
```

```css
/* In CSS */
:root {
  --font-sans: 'Inter', system-ui, sans-serif;
  --font-heading: 'Plus Jakarta Sans', var(--font-sans);
  --font-mono: 'JetBrains Mono', 'Fira Code', monospace;
}

body {
  font-family: var(--font-sans);
  font-feature-settings: 'kern' 1, 'liga' 1; /* Better letter spacing */
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}
```

**Always provide fallback fonts.** The system-ui fallback should look acceptable while custom fonts load.

---

## Heading Hierarchy

### Semantic Rules
- One `<h1>` per page (main topic)
- `<h2>` for major sections
- `<h3>` for subsections within h2
- Never skip levels (h1 → h3 without h2)
- Don't use heading elements for styling — use CSS classes

```html
<!-- ✅ Correct semantic structure -->
<h1>Getting Started with YUIUX</h1>
  <h2>Installation</h2>
    <h3>npm</h3>
    <h3>yarn</h3>
  <h2>Configuration</h2>
    <h3>Basic setup</h3>
    <h3>Advanced options</h3>

<!-- ❌ Wrong: skipped level, heading for styling -->
<h1>Page Title</h1>
<h3>This should be h2</h3>
<h2 style="font-size: 14px; font-weight: 400">Not really a heading</h2>
```

### Visual Heading System
```css
/* Heading base styles */
h1, h2, h3, h4, h5, h6 {
  font-family: var(--font-heading);
  font-weight: 700;
  line-height: 1.15;
  color: var(--foreground);
  /* No margin-top — let parent control spacing */
}

h1 { font-size: clamp(2rem, 5vw, 3rem); }
h2 { font-size: clamp(1.5rem, 3vw, 2.25rem); }
h3 { font-size: clamp(1.25rem, 2.5vw, 1.75rem); }
h4 { font-size: 1.25rem; }
h5 { font-size: 1.125rem; }
h6 { font-size: 1rem; font-weight: 600; }
```

---

## Letter Spacing (Tracking)

| Use | Tracking | CSS |
|-----|---------|-----|
| Body text | 0 or -0.01em | `letter-spacing: -0.01em` |
| Large headings (>40px) | -0.02 to -0.04em | `letter-spacing: -0.03em` |
| Small caps / UI labels | +0.05 to +0.1em | `letter-spacing: 0.08em` |
| ALL CAPS labels | +0.05 to +0.15em | `letter-spacing: 0.12em` |
| Monospace/code | 0 | `letter-spacing: 0` |

```css
/* Large hero heading — tighter tracking looks premium */
.hero-title {
  font-size: clamp(3rem, 8vw, 6rem);
  font-weight: 800;
  letter-spacing: -0.04em;
  line-height: 1.0;
}

/* ALL CAPS badge/label — needs wider tracking to be readable */
.badge-text {
  font-size: 11px;
  font-weight: 600;
  letter-spacing: 0.08em;
  text-transform: uppercase;
}
```

---

## Text Rendering

```css
/* For all body text — smooth rendering on retina displays */
body {
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-rendering: optimizeLegibility;
}

/* For headings — subpixel for clarity on non-retina */
h1, h2, h3 {
  -webkit-font-smoothing: subpixel-antialiased;
}
```

---

## Responsive Typography

### The clamp() Function
```css
/* clamp(min, preferred, max) */
/* Scales smoothly between viewport sizes */

h1 {
  font-size: clamp(2rem, 5vw + 1rem, 4rem);
  /* Min: 32px, Scales with viewport, Max: 64px */
}

body {
  font-size: clamp(1rem, 2.5vw, 1.125rem);
  /* Min: 16px, Max: 18px — scales slightly */
}
```

### Breakpoint Type Adjustments
```css
/* Mobile first */
body { font-size: 15px; line-height: 1.6; }
h1   { font-size: 2rem; }
h2   { font-size: 1.5rem; }

@media (min-width: 768px) {
  body { font-size: 16px; }
  h1   { font-size: 2.5rem; }
  h2   { font-size: 2rem; }
}

@media (min-width: 1280px) {
  h1   { font-size: 3.5rem; }
  h2   { font-size: 2.5rem; }
}
```

---

## Typography Anti-Patterns

### 1. All-Caps Body Text
```css
/* ❌ Never */
.body-text { text-transform: uppercase; }

/* ✅ Fine for short labels only */
.badge { 
  text-transform: uppercase; 
  font-size: 11px;
  letter-spacing: 0.08em;
  font-weight: 600;
}
```
**Why:** ALL CAPS removes word shape recognition — readers process words by shape, not letter-by-letter. ALL CAPS reduces reading speed by 13–20%.

### 2. Justified Text on Web
```css
/* ❌ Don't use on web */
.text { text-align: justify; }

/* ✅ Left-align body text */
.text { text-align: left; }
```
**Why:** Browsers create "rivers" of whitespace in justified text without hyphenation support. Uneven spacing is harder to read than a ragged right edge.

### 3. Light Text on Light Background
```css
/* ❌ */
.helper { color: #d1d5db; } /* gray-300 on white → fails contrast */

/* ✅ */
.helper { color: #6b7280; } /* gray-500 on white → 4.63:1 passes AA */
```

### 4. Long Lines Without Max-Width
```css
/* ❌ Paragraph spanning 100% of a 1440px screen */
p { width: 100%; }

/* ✅ Constrained for readability */
p { max-width: 65ch; }
```

### 5. No Fallback Fonts
```css
/* ❌ */
body { font-family: 'Custom Font'; }

/* ✅ */
body { font-family: 'Custom Font', system-ui, -apple-system, sans-serif; }
```

### 6. Line Height Too Tight on Body Text
```css
/* ❌ */
p { line-height: 1.2; } /* Fine for headings, awful for body */

/* ✅ */
p { line-height: 1.6; }
```

### 7. Font Weight Without Optical Size
```css
/* ❌ Black weight at 12px */
.tiny-label { font-size: 12px; font-weight: 900; }

/* ✅ Black weight only at large sizes */
.hero { font-size: 4rem; font-weight: 900; }
.tiny-label { font-size: 12px; font-weight: 600; }
```

### 8. Generic Quotation Marks
```html
<!-- ❌ Straight quotes -->
<p>"This is a quote"</p>

<!-- ✅ Curly/typographic quotes -->
<p>"This is a quote"</p>
<!-- Use CSS if HTML source is hard to control: -->
<style>
q { quotes: "\201C" "\201D" "\2018" "\2019"; }
</style>
```

---

## Readability Checklist

Before finalizing any text:

- [ ] Body text ≥16px on desktop, ≥15px on mobile
- [ ] Line height ≥1.5 for body, ≥1.1 for headings
- [ ] Line length ≤75ch for body text
- [ ] Text/background contrast ≥4.5:1 (WCAG AA)
- [ ] Heading hierarchy is semantic (h1→h2→h3, no skips)
- [ ] Font weight appropriate for size (no heavy weights at small sizes)
- [ ] Fallback fonts defined for all custom fonts
- [ ] `prefers-reduced-motion` respected for any text animation
- [ ] Text scales correctly when browser zoom is 200%
- [ ] No justified text on web
- [ ] No ALL CAPS for body text
- [ ] Letter spacing correct for context (tighter at large, wider at small+caps)
- [ ] `-webkit-font-smoothing: antialiased` applied

---

## Complete Typography System (Copy-Paste)

```css
/* ===== YUIUX Typography System ===== */

/* Font import */
@import url('https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&family=Plus+Jakarta+Sans:wght@600;700;800&display=swap');

/* Custom properties */
:root {
  /* Fonts */
  --font-sans: 'Inter', system-ui, -apple-system, sans-serif;
  --font-heading: 'Plus Jakarta Sans', var(--font-sans);
  --font-mono: 'JetBrains Mono', 'Fira Code', 'Cascadia Code', monospace;
  
  /* Scale */
  --text-xs:   0.75rem;   /* 12px */
  --text-sm:   0.875rem;  /* 14px */
  --text-base: 1rem;      /* 16px */
  --text-lg:   1.125rem;  /* 18px */
  --text-xl:   1.25rem;   /* 20px */
  --text-2xl:  1.5rem;    /* 24px */
  --text-3xl:  1.875rem;  /* 30px */
  --text-4xl:  2.25rem;   /* 36px */
  --text-5xl:  3rem;      /* 48px */
  --text-6xl:  3.75rem;   /* 60px */
  --text-7xl:  4.5rem;    /* 72px */
  
  /* Line heights */
  --leading-tight:  1.15;
  --leading-snug:   1.375;
  --leading-normal: 1.5;
  --leading-relaxed:1.625;
  --leading-loose:  2;
  
  /* Letter spacing */
  --tracking-tighter: -0.05em;
  --tracking-tight:   -0.025em;
  --tracking-normal:  0em;
  --tracking-wide:    0.025em;
  --tracking-wider:   0.05em;
  --tracking-widest:  0.1em;
}

/* Base */
*, *::before, *::after { box-sizing: border-box; }

html { font-size: 16px; }

body {
  font-family: var(--font-sans);
  font-size: var(--text-base);
  line-height: var(--leading-relaxed);
  color: var(--foreground);
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-rendering: optimizeLegibility;
}

/* Headings */
h1, h2, h3, h4, h5, h6 {
  font-family: var(--font-heading);
  font-weight: 700;
  line-height: var(--leading-tight);
  letter-spacing: var(--tracking-tight);
  color: var(--foreground);
}

h1 { font-size: clamp(var(--text-3xl), 5vw, var(--text-5xl)); }
h2 { font-size: clamp(var(--text-2xl), 4vw, var(--text-4xl)); }
h3 { font-size: clamp(var(--text-xl), 3vw, var(--text-3xl)); }
h4 { font-size: var(--text-xl); }
h5 { font-size: var(--text-lg); }
h6 { font-size: var(--text-base); font-weight: 600; }

/* Body text */
p {
  max-width: 65ch;
  margin-block: 0;
}

p + p { margin-top: 1em; }

/* Lead paragraph */
.lead {
  font-size: var(--text-xl);
  line-height: var(--leading-relaxed);
  color: var(--muted-foreground);
}

/* Small text */
small, .text-sm {
  font-size: var(--text-sm);
  line-height: var(--leading-normal);
}

/* Captions */
.caption {
  font-size: var(--text-xs);
  line-height: var(--leading-normal);
  color: var(--muted-foreground);
}

/* Labels */
label, .label {
  font-size: var(--text-sm);
  font-weight: 500;
  line-height: var(--leading-tight);
}

/* Code */
code {
  font-family: var(--font-mono);
  font-size: 0.9em;
  background: var(--muted);
  padding: 0.1em 0.3em;
  border-radius: 3px;
}

pre code {
  background: none;
  padding: 0;
  font-size: var(--text-sm);
}

/* Links */
a {
  color: var(--primary);
  text-decoration: underline;
  text-decoration-color: color-mix(in srgb, var(--primary) 50%, transparent);
  text-underline-offset: 3px;
  transition: text-decoration-color 150ms;
}

a:hover {
  text-decoration-color: var(--primary);
}

/* Strong */
strong { font-weight: 600; }

/* Prose max-width class */
.prose {
  max-width: 65ch;
}

.prose-wide {
  max-width: 75ch;
}

/* Reduced motion */
@media (prefers-reduced-motion: reduce) {
  * { transition-duration: 0.01ms !important; animation-duration: 0.01ms !important; }
}
```
