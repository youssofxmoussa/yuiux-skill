# 17 — Performance & Web Vitals

> Performance is a UX problem. A 100ms delay feels immediate. A 1s delay is noticeable. A 3s delay causes abandonment. Every millisecond is a UX decision.

---

## Core Web Vitals (CWV) — The Targets

| Metric | Good | Needs Improvement | Poor | What It Measures |
|--------|------|-------------------|------|-----------------|
| **LCP** (Largest Contentful Paint) | ≤ 2.5s | 2.5–4.0s | > 4.0s | How fast the main content appears |
| **INP** (Interaction to Next Paint) | ≤ 200ms | 200–500ms | > 500ms | How fast the page responds to interactions |
| **CLS** (Cumulative Layout Shift) | ≤ 0.1 | 0.1–0.25 | > 0.25 | How much the layout unexpectedly shifts |

Secondary metrics (still matter):
- **FCP** (First Contentful Paint): ≤ 1.8s — first any pixel renders
- **TTFB** (Time to First Byte): ≤ 800ms — server response time
- **TBT** (Total Blocking Time): ≤ 200ms — main thread blocking

---

## LCP — Largest Contentful Paint

### What Counts as LCP Element
- Large images (`<img>`, `<picture>`)
- Video posters
- Background images with CSS
- Block-level text elements (if large enough)

Most often: the **hero image** or **hero heading** on landing pages.

### Optimizing LCP

```html
<!-- ✅ Preload the hero image (LCP element) -->
<link
  rel="preload"
  as="image"
  href="hero.webp"
  fetchpriority="high"
/>

<!-- ✅ Hero image: no lazy loading, high priority -->
<img
  src="hero.webp"
  alt="Hero image description"
  loading="eager"
  fetchpriority="high"
  decoding="async"
  width="1280"
  height="720"
/>

<!-- ❌ Never lazy-load the LCP image -->
<img src="hero.webp" loading="lazy" />
```

```tsx
// Next.js / React — image optimization
import Image from 'next/image';

// Hero image — highest priority
<Image
  src="/hero.webp"
  alt="Hero"
  width={1280}
  height={720}
  priority={true}    // disables lazy loading, adds preload
  quality={85}
/>

// Non-hero images — lazy load
<Image
  src="/product.webp"
  alt="Product name"
  width={400}
  height={300}
  loading="lazy"
/>
```

### Critical CSS Inlining

```html
<!-- Inline critical above-fold CSS to avoid render-blocking -->
<style>
  /* Minimal styles for above-fold content */
  body { margin: 0; font-family: system-ui; background: #fff; }
  .hero { min-height: 100dvh; display: flex; align-items: center; }
  /* ... */
</style>

<!-- Defer non-critical CSS -->
<link rel="preload" href="styles.css" as="style" onload="this.onload=null;this.rel='stylesheet'">
<noscript><link rel="stylesheet" href="styles.css"></noscript>
```

---

## CLS — Cumulative Layout Shift

### Top Causes and Fixes

**1. Images without dimensions**
```html
<!-- ❌ No dimensions — shifts layout when image loads -->
<img src="product.jpg" alt="Product" />

<!-- ✅ Dimensions set — browser reserves space -->
<img src="product.jpg" alt="Product" width="400" height="300" />
```

```css
/* ✅ CSS aspect-ratio reserves space */
.card-image {
  aspect-ratio: 16/9;
  overflow: hidden;
}
.card-image img { width: 100%; height: 100%; object-fit: cover; }
```

**2. Ads / embeds without size**
```html
<!-- ❌ Ad slot shifts content when ad loads -->
<div class="ad-slot"></div>

<!-- ✅ Reserve exact space for ad -->
<div class="ad-slot" style="min-height: 250px; width: 300px;"></div>
```

**3. Dynamically injected content above existing content**
```tsx
// ❌ Banner injected above nav shifts everything down
function App() {
  const [banner, setBanner] = useState<string | null>(null);
  useEffect(() => {
    fetchBanner().then(setBanner);
  }, []);
  
  return (
    <>
      {banner && <Banner message={banner} />} {/* causes CLS */}
      <Nav />
      <Main />
    </>
  );
}

// ✅ Reserve space for potential banner
function App() {
  const [banner, setBanner] = useState<string | null>(null);
  
  return (
    <>
      <div className="banner-slot" style={{ minHeight: banner ? 'auto' : '48px' }}>
        {banner && <Banner message={banner} />}
      </div>
      <Nav />
      <Main />
    </>
  );
}
```

**4. Web fonts causing text shift (FOUT/FOIT)**
```css
/* Prevent FOUT with font-display */
@font-face {
  font-family: 'Brand';
  src: url('brand.woff2') format('woff2');
  font-display: optional; /* best for CLS: falls back to system font if not loaded immediately */
  /* font-display: swap; — good for LCP, may cause CLS */
}

/* OR: use a font metric override to match fallback closely */
@font-face {
  font-family: 'Brand-Fallback';
  src: local('Arial');
  ascent-override: 90%;
  descent-override: 21%;
  line-gap-override: 0%;
  size-adjust: 106%;
}
```

---

## INP — Interaction to Next Paint

INP measures how long it takes for the browser to paint after a user interaction. Formerly FID (First Input Delay).

### Long Tasks

Any task on the main thread > 50ms blocks the browser. Break them up:

```tsx
// ❌ Long synchronous task blocks main thread
function processLargeDataset(data: Item[]) {
  return data.map(expensiveTransform).filter(isValid).sort(compareItems);
  // If data.length > 10,000 this may block for 200ms+
}

// ✅ Yield to main thread periodically
async function processLargeDatasetAsync(data: Item[]): Promise<Item[]> {
  const results: Item[] = [];
  const CHUNK = 1000;
  
  for (let i = 0; i < data.length; i += CHUNK) {
    const chunk = data.slice(i, i + CHUNK);
    results.push(...chunk.map(expensiveTransform).filter(isValid));
    
    // Yield — allow browser to handle user interactions
    await new Promise(resolve => setTimeout(resolve, 0));
  }
  
  return results.sort(compareItems);
}

// ✅ Web Worker for heavy computation
const worker = new Worker('/workers/data-processor.js');
worker.postMessage({ data: largeDataset });
worker.onmessage = (e) => setResults(e.data.results);
```

### Event Handler Optimization

```tsx
// ❌ Heavy synchronous work in click handler
function handleFilter(newValue: string) {
  const filtered = allItems
    .filter(item => complexMatch(item, newValue)) // blocks for 150ms
    .sort(complexSort); // another 50ms
  setItems(filtered);
}

// ✅ Optimistic update + deferred computation
function handleFilter(newValue: string) {
  setFilterValue(newValue); // instant UI update
  startTransition(() => {   // defer the heavy re-render
    setFilteredItems(computeFilter(allItems, newValue));
  });
}

// React 18: useTransition for non-urgent updates
const [isPending, startTransition] = useTransition();
```

---

## Image Optimization

```bash
# Convert to WebP with cwebp (better compression, same quality)
cwebp -q 85 input.jpg -o output.webp

# Convert to AVIF (even better, newer)
avifenc input.jpg output.avif

# Batch convert
for f in *.jpg; do cwebp -q 85 "$f" -o "${f%.jpg}.webp"; done
```

```tsx
// Serve modern formats with fallback
<picture>
  <source type="image/avif" srcset="image.avif" />
  <source type="image/webp" srcset="image.webp" />
  <img src="image.jpg" alt="Description" width="800" height="600" loading="lazy" />
</picture>
```

### Image Size Rules

```
Hero images:     ≤ 200KB (LCP critical path)
Product images:  ≤ 100KB at display size
Thumbnails:      ≤ 30KB
Icons/logos:     SVG (no raster size limit)
Avatar images:   ≤ 20KB at 2x display size
```

---

## Font Loading

```html
<!-- 1. Preconnect to font origin -->
<link rel="preconnect" href="https://fonts.googleapis.com" />
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin />

<!-- 2. Preload critical font files -->
<link
  rel="preload"
  as="font"
  type="font/woff2"
  href="/fonts/brand-400.woff2"
  crossorigin
/>

<!-- 3. Self-host fonts (avoid third-party DNS overhead) -->
```

```css
/* Efficient font loading */
@font-face {
  font-family: 'Brand';
  src: url('/fonts/brand-400.woff2') format('woff2');
  font-weight: 400;
  font-style: normal;
  font-display: swap;  /* show fallback immediately, swap when font loads */
}

/* Limit font weights loaded */
/* Only load what you actually use */
/* ❌ Importing entire font family: */
/* @import url('https://fonts.googleapis.com/css2?family=Inter:wght@100;200;300;400;500;600;700;800;900'); */

/* ✅ Only load used weights: */
/* @import url('https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600'); */
```

---

## Bundle Size

```bash
# Analyze bundle
npx vite-bundle-visualizer  # Vite
npx webpack-bundle-analyzer # Webpack
npx source-map-explorer 'build/static/js/*.js' # Create React App

# Check package size before installing
npx bundlephobia package-name
```

```tsx
// Code splitting — load routes lazily
import { lazy, Suspense } from 'react';

const Settings = lazy(() => import('./pages/Settings'));
const Dashboard = lazy(() => import('./pages/Dashboard'));

function App() {
  return (
    <Suspense fallback={<PageLoading />}>
      <Routes>
        <Route path="/dashboard" element={<Dashboard />} />
        <Route path="/settings" element={<Settings />} />
      </Routes>
    </Suspense>
  );
}

// Tree shaking — import specifically, not entire library
// ❌
import _ from 'lodash';
const result = _.map(items, transform);

// ✅
import { map } from 'lodash-es';
const result = map(items, transform);

// ✅ Even better: use native
const result = items.map(transform);
```

---

## Rendering Optimization

```tsx
// memo — skip re-render if props unchanged
const ExpensiveList = memo(function ExpensiveList({ items, onSelect }) {
  return (
    <ul>
      {items.map(item => (
        <li key={item.id} onClick={() => onSelect(item)}>
          {item.name}
        </li>
      ))}
    </ul>
  );
});

// useMemo — memoize expensive computations
const sortedItems = useMemo(
  () => [...items].sort((a, b) => a.name.localeCompare(b.name)),
  [items]
);

// useCallback — stable function reference for memo'd children
const handleSelect = useCallback(
  (item: Item) => { setSelected(item); navigate(`/items/${item.id}`); },
  [navigate] // only recreate if navigate changes
);

// Virtual list — for large datasets (> 100 items)
import { FixedSizeList } from 'react-window';

function VirtualList({ items }: { items: Item[] }) {
  return (
    <FixedSizeList
      height={600}
      itemCount={items.length}
      itemSize={72}         // fixed row height
      width="100%"
    >
      {({ index, style }) => (
        <div style={style}>
          <ListItem item={items[index]} />
        </div>
      )}
    </FixedSizeList>
  );
}
```

---

## Performance Monitoring

```tsx
// Measure and report Web Vitals
import { getCLS, getFID, getLCP, getTTFB, getFCP, getINP } from 'web-vitals';

function reportWebVitals(metric) {
  // Log to console in development
  if (process.env.NODE_ENV === 'development') {
    console.log(metric.name, metric.value.toFixed(2), metric.rating);
  }
  
  // Send to analytics in production
  if (process.env.NODE_ENV === 'production') {
    fetch('/api/vitals', {
      method: 'POST',
      body: JSON.stringify({
        name: metric.name,
        value: metric.value,
        rating: metric.rating, // 'good', 'needs-improvement', 'poor'
        url: window.location.pathname,
      }),
    });
  }
}

getCLS(reportWebVitals);
getFID(reportWebVitals);  
getLCP(reportWebVitals);
getTTFB(reportWebVitals);
getFCP(reportWebVitals);
getINP(reportWebVitals);
```

---

## Performance Checklist

### Images
- [ ] Hero/LCP image has `loading="eager"` and `fetchpriority="high"`
- [ ] Non-hero images have `loading="lazy"`
- [ ] All images have `width` and `height` attributes (prevent CLS)
- [ ] Images served in WebP or AVIF format
- [ ] Images compressed to appropriate size for display dimensions

### Fonts
- [ ] Font files self-hosted (not external CDN)
- [ ] Only used font weights loaded (400, 500, 600 — not 100–900)
- [ ] `font-display: swap` or `optional` set
- [ ] Critical fonts preloaded with `<link rel="preload">`

### JavaScript
- [ ] Routes code-split with `React.lazy()`
- [ ] No large packages imported when only a small function needed
- [ ] Long-running computations moved to Web Workers or chunked
- [ ] Event handlers don't do > 50ms of synchronous work

### Network
- [ ] Static assets served with long-term caching headers
- [ ] API responses cached where appropriate
- [ ] Images and fonts served from CDN

### Monitoring
- [ ] Web Vitals tracked and reported
- [ ] LCP ≤ 2.5s in production
- [ ] CLS ≤ 0.1 in production
- [ ] INP ≤ 200ms in production
