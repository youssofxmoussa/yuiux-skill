# 18 — Icons & Imagery

> Icons are the punctuation of UI. Used well, they're invisible — they just help. Used poorly, they're noise.

---

## Icon Principles

### 1. Icons Are Not Self-Explanatory

Most icons are ambiguous. A hamburger icon is only understood as "menu" because of widespread convention. A house icon means "home" only by convention. When in doubt, always pair an icon with a text label.

```
Icon alone → Use only for universally understood actions in tight spaces
Icon + label → Default for all navigation, primary actions, and controls
Label alone → Fine when no icon adds value
```

### 2. Meaning Before Style

Pick icons that communicate their meaning first, then look for style consistency. A "save" icon that looks like a house is confusing, regardless of how nice it looks.

### 3. One Icon Library

Mix and match icon libraries creates visual inconsistency. Pick one:
- **Lucide React** — for shadcn/ui and Lovable projects
- **Heroicons** — Tailwind team's library
- **Phosphor Icons** — large, well-designed set
- **Radix Icons** — minimal, matches Radix UI
- **Feather Icons** — clean, minimal

---

## Icon Accessibility

Every icon falls into one of these categories:

```tsx
// 1. DECORATIVE — purely visual, accompanies text that conveys the meaning
<button>
  <SaveIcon aria-hidden="true" className="w-4 h-4" />
  Save
</button>

// 2. FUNCTIONAL — icon IS the interactive element's label (icon button)
<button aria-label="Save document">
  <SaveIcon aria-hidden="true" className="w-4 h-4" />
</button>

// 3. INFORMATIVE — icon conveys information without adjacent text
<span aria-label="High priority">
  <AlertTriangleIcon aria-hidden="true" className="text-warning" />
</span>

// 4. INTERACTIVE WITHIN TEXT — icon inside a link that has text
<a href="/settings">
  Settings
  <ExternalLinkIcon aria-hidden="true" className="w-3 h-3 inline" />
  <span className="sr-only">(opens in new tab)</span>
</a>
```

**The Rule:** If the icon is the only visual indicator of meaning, it needs an accessible name (`aria-label` or `<title>`).

---

## Icon Sizing

```css
/* Icon sizes — always use consistent scale */
.icon-xs { width: 12px; height: 12px; }  /* inline badge indicator */
.icon-sm { width: 14px; height: 14px; }  /* tight spaces, small buttons */
.icon-md { width: 16px; height: 16px; }  /* default, next to text */
.icon-lg { width: 20px; height: 20px; }  /* medium buttons */
.icon-xl { width: 24px; height: 24px; }  /* standalone, nav items */
.icon-2xl{ width: 32px; height: 32px; }  /* large feature icons */
.icon-3xl{ width: 48px; height: 48px; }  /* empty state icons */
.icon-4xl{ width: 64px; height: 64px; }  /* hero icons */
```

### Icon Next to Text

The rule: icons should match the text's **cap height**, not its computed font-size.

```css
/* Cap height is typically 0.7–0.75em for most fonts */
.icon-inline {
  width: 1em;
  height: 1em;
  /* Optical vertical alignment */
  vertical-align: -0.125em;
}

/* Better: use flex with align-items: center */
.button {
  display: inline-flex;
  align-items: center;
  gap: 6px;
}
/* Icon size: 16px for 14px text, 18px for 16px text, 20px for 18px text */
```

---

## SVG Icons

```tsx
// Base SVG icon component
interface IconProps extends React.SVGAttributes<SVGElement> {
  size?: number;
  title?: string;       // accessible title
  decorative?: boolean; // explicitly mark as decorative
}

function Icon({ size = 16, title, decorative, children, ...props }: IconProps) {
  const titleId = useId();
  
  return (
    <svg
      xmlns="http://www.w3.org/2000/svg"
      width={size}
      height={size}
      viewBox="0 0 24 24"
      fill="none"
      stroke="currentColor"
      strokeWidth={2}
      strokeLinecap="round"
      strokeLinejoin="round"
      role={title ? 'img' : 'presentation'}
      aria-hidden={!title && (decorative || true)}
      aria-labelledby={title ? titleId : undefined}
      {...props}
    >
      {title && <title id={titleId}>{title}</title>}
      {children}
    </svg>
  );
}

// Usage
<Icon size={20} aria-hidden>
  <path d="M19 21H5a2 2 0 01-2-2V5a2 2 0 012-2h11l5 5v14a2 2 0 01-2 2z"/>
  <polyline points="17 21 17 13 7 13 7 21"/>
</Icon>
```

### SVG Sprite for Performance

Instead of inline SVG on every instance, use SVG sprites:

```html
<!-- _sprite.svg — hidden, included once at top of body -->
<svg xmlns="http://www.w3.org/2000/svg" style="display:none">
  <symbol id="icon-save" viewBox="0 0 24 24">
    <path d="M19 21H5a2 2 0 01-2-2V5a2 2 0 012-2h11l5 5v14a2 2 0 01-2 2z"/>
    <polyline points="17 21 17 13 7 13 7 21"/>
  </symbol>
  <!-- ... more icons ... -->
</svg>

<!-- Usage anywhere in page -->
<svg aria-hidden="true" width="20" height="20">
  <use href="#icon-save" />
</svg>
```

---

## Favicon & App Icons

```html
<!-- Modern favicon setup -->
<link rel="icon" type="image/svg+xml" href="/favicon.svg" />
<link rel="icon" type="image/png" href="/favicon.png" />
<link rel="apple-touch-icon" href="/apple-touch-icon.png" />
<meta name="theme-color" content="#2563eb" />

<!-- PWA manifest -->
<link rel="manifest" href="/manifest.json" />
```

```json
// manifest.json
{
  "name": "App Name",
  "short_name": "App",
  "icons": [
    { "src": "/icons/192.png", "sizes": "192x192", "type": "image/png" },
    { "src": "/icons/512.png", "sizes": "512x512", "type": "image/png" },
    { "src": "/icons/512-maskable.png", "sizes": "512x512", "type": "image/png", "purpose": "maskable" }
  ],
  "theme_color": "#2563eb",
  "background_color": "#ffffff",
  "display": "standalone"
}
```

---

## Illustration Guidelines

### Do's
- Use illustration to explain complex concepts or empty states
- Consistent illustration style throughout the app
- Illustrations should match the brand's personality
- Use SVG for illustrations (scalable, small file size, themeable)
- Ensure illustrations work in both light and dark mode

### Don'ts
- Don't use stock photo-realistic illustrations alongside hand-drawn ones
- Don't use illustrations just to "fill space"
- Don't use illustrations of faces that exclude diverse representations
- Don't use illustrations with text embedded (can't localize)

```tsx
// Dark mode compatible illustration
function EmptyStateIllustration() {
  return (
    <svg viewBox="0 0 240 180" aria-hidden="true" className="illustration">
      {/* Use currentColor for stroke/fill to adapt to dark mode */}
      <circle cx="120" cy="90" r="60" fill="var(--primary-subtle)" />
      <path
        d="M90 90 L120 60 L150 90"
        stroke="var(--primary)"
        strokeWidth="2"
        fill="none"
      />
      {/* Use CSS variables for theme-adaptive colors */}
    </svg>
  );
}
```

---

## Open Graph & Social Images

```html
<!-- Social sharing metadata -->
<meta property="og:title" content="Page Title" />
<meta property="og:description" content="Page description under 160 characters." />
<meta property="og:image" content="https://yourdomain.com/og-image.png" />
<meta property="og:image:width" content="1200" />
<meta property="og:image:height" content="630" />
<meta property="og:url" content="https://yourdomain.com/page" />
<meta property="og:type" content="website" />

<!-- Twitter/X -->
<meta name="twitter:card" content="summary_large_image" />
<meta name="twitter:image" content="https://yourdomain.com/og-image.png" />
```

**OG Image Dimensions:** 1200×630px. Always include text overlay — images without readable text perform poorly in feeds.

---

## Loading Images Progressively

```tsx
function ProgressiveImage({ src, placeholder, alt, ...props }: ProgressiveImageProps) {
  const [loaded, setLoaded] = useState(false);
  
  return (
    <div className="progressive-image">
      {/* Low-quality placeholder */}
      {!loaded && (
        <img
          src={placeholder}
          alt=""
          aria-hidden
          className="progressive-image-placeholder"
        />
      )}
      
      {/* Full quality image */}
      <img
        src={src}
        alt={alt}
        onLoad={() => setLoaded(true)}
        className={`progressive-image-main ${loaded ? 'loaded' : ''}`}
        {...props}
      />
    </div>
  );
}
```

```css
.progressive-image { position: relative; }

.progressive-image-placeholder {
  position: absolute;
  inset: 0;
  width: 100%;
  height: 100%;
  object-fit: cover;
  filter: blur(10px);
  transform: scale(1.05); /* prevent blur edge artifacts */
}

.progressive-image-main {
  opacity: 0;
  transition: opacity 400ms ease;
  width: 100%;
  height: 100%;
  object-fit: cover;
}

.progressive-image-main.loaded { opacity: 1; }
```

---

## Icon Anti-Patterns

| Anti-Pattern | Problem | Fix |
|-------------|---------|-----|
| Icon without label in navigation | Fails WCAG, confuses users | Add text label or tooltip |
| Mixing icon families | Visual inconsistency | Pick one icon library |
| Arbitrary icon sizes | Off-grid, doesn't align with text | Use size scale (12/14/16/20/24/32/48) |
| Icon without `aria-hidden` when decorative | Screen readers read SVG paths as gibberish | Add `aria-hidden="true"` |
| Icon without `aria-label` when standalone | Screen reader says nothing useful | Add `aria-label` or `<title>` in SVG |
| Non-standard icons for standard actions | Cognitive load, violates mental models | Use standard icons (✉ email, 🔍 search, ✕ close) |
| Too many icons | Visual noise, reduces signal value | Be selective; every icon should earn its place |

---

## Icon & Image Checklist

- [ ] All icons in one library (no mixing families)
- [ ] Decorative icons have `aria-hidden="true"`
- [ ] Icon-only interactive elements have `aria-label`
- [ ] Icon sizes from consistent scale (not arbitrary px)
- [ ] Images have `width` and `height` to prevent CLS
- [ ] Hero/above-fold images: `loading="eager" fetchpriority="high"`
- [ ] Below-fold images: `loading="lazy" decoding="async"`
- [ ] Product/content images served in WebP/AVIF
- [ ] Illustrations work in dark mode (use CSS variables not hardcoded colors)
- [ ] OG image 1200×630px for social sharing
- [ ] Favicon has SVG version for modern browsers
- [ ] PWA icons: 192px and 512px (with maskable variant)
