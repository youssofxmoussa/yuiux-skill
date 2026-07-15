# 05 — Motion, Animation & Micro-interactions

> Motion communicates. Every animation either explains something or distracts from it. There is no neutral motion.

---

## The Purpose Test

Before adding any animation, answer: **What does this motion communicate?**

Valid answers:
- "It shows the element is responding to my action" (feedback)
- "It shows where this element came from / went to" (spatial orientation)
- "It draws attention to a change that matters" (signaling)
- "It gives the UI personality and delight" (only valid after all functional purposes are served)

Invalid answers:
- "It looks cool"
- "It fills time while waiting"
- "The designer asked for it"

---

## Duration Budget

```
Micro: 0–100ms    — direct manipulation response (button press, toggle)
Fast: 100–200ms   — simple transitions (hover fade, small element appear)
Base: 200–300ms   — standard UI transitions (dropdown, tooltip, sidebar)
Slow: 300–500ms   — complex transitions (modal open, page route, panel slide)
Very slow: 500ms+ — only for large-area or complex animated sequences
```

**Never exceed 500ms for a UI transition the user triggered.** At 500ms, motion stops feeling responsive and starts feeling slow.

```css
:root {
  --duration-micro: 75ms;
  --duration-fast:  150ms;
  --duration-base:  200ms;
  --duration-slow:  300ms;
  --duration-slower:400ms;
  --duration-page:  350ms;
}
```

---

## Easing Functions

```css
:root {
  /* Standard easings */
  --ease-linear:     linear;
  --ease-in:         cubic-bezier(0.4, 0, 1, 1);
  --ease-out:        cubic-bezier(0, 0, 0.2, 1);
  --ease-in-out:     cubic-bezier(0.4, 0, 0.2, 1);
  
  /* Spring-like easings */
  --ease-spring:     cubic-bezier(0.34, 1.56, 0.64, 1); /* slight overshoot */
  --ease-back-in:    cubic-bezier(0.36, 0, 0.66, -0.56);
  --ease-back-out:   cubic-bezier(0.34, 1.56, 0.64, 1);
  
  /* Specialized */
  --ease-sharp:      cubic-bezier(0.4, 0, 0.6, 1); /* quick in/out */
  --ease-standard:   cubic-bezier(0.2, 0, 0, 1);   /* Material standard */
  --ease-decelerate: cubic-bezier(0, 0, 0, 1);     /* entering elements */
  --ease-accelerate: cubic-bezier(0.3, 0, 1, 1);   /* exiting elements */
}
```

### When to Use Each Easing

| Easing | Use For |
|--------|---------|
| `ease-out` | Elements entering the screen (decelerate on arrival) |
| `ease-in` | Elements leaving the screen (accelerate on departure) |
| `ease-in-out` | Elements moving between two positions |
| `ease-spring` | Playful, physical responses (toggle, expand) |
| `linear` | Spinners, progress bars, ambient loops |
| `ease-sharp` | Quick UI feedback, tooltips, hover states |

---

## prefers-reduced-motion (Required)

```css
/* ===== ALWAYS INCLUDE THIS ===== */
@media (prefers-reduced-motion: reduce) {
  *,
  *::before,
  *::after {
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.01ms !important;
    scroll-behavior: auto !important;
  }
}
```

### What to Disable vs. Preserve

```css
/* Disable: decorative/entertainment animation */
@media (prefers-reduced-motion: reduce) {
  .hero-parallax { transform: none !important; }
  .floating-orb  { animation: none !important; }
  .typing-effect { animation: none !important; }
  .counter-animation { transition: none !important; }
}

/* Preserve: essential feedback animations (can be instant) */
/* Spinners, loading indicators, progress bars: 
   Replace animated with static spinner image or just opacity */
@media (prefers-reduced-motion: reduce) {
  .spinner {
    animation: none;
    opacity: 0.7;
    /* Static visual is fine — just not spinning */
  }
}
```

### React Hook for Reduced Motion
```tsx
function useReducedMotion(): boolean {
  const [prefersReduced, setPrefersReduced] = useState(() => {
    if (typeof window === 'undefined') return false;
    return window.matchMedia('(prefers-reduced-motion: reduce)').matches;
  });
  
  useEffect(() => {
    const mq = window.matchMedia('(prefers-reduced-motion: reduce)');
    const handler = (e: MediaQueryListEvent) => setPrefersReduced(e.matches);
    mq.addEventListener('change', handler);
    return () => mq.removeEventListener('change', handler);
  }, []);
  
  return prefersReduced;
}

// Usage:
function AnimatedCard() {
  const reducedMotion = useReducedMotion();
  
  return (
    <motion.div
      initial={{ opacity: 0, y: reducedMotion ? 0 : 20 }}
      animate={{ opacity: 1, y: 0 }}
      transition={{ 
        duration: reducedMotion ? 0 : 0.3,
        ease: 'easeOut'
      }}
    >
      Content
    </motion.div>
  );
}
```

---

## Micro-interaction Patterns

### Button Press Feedback
```css
/* Subtle scale on press — immediate tactile feedback */
button {
  transition: 
    background-color var(--duration-fast) var(--ease-out),
    transform var(--duration-micro) var(--ease-out),
    box-shadow var(--duration-fast) var(--ease-out);
}

button:hover  { box-shadow: var(--shadow-md); }
button:active { transform: scale(0.98) translateY(1px); box-shadow: none; }
```

### Hover States
```css
/* Card hover — elevation lift */
.card {
  transition: 
    box-shadow var(--duration-fast) var(--ease-out),
    transform var(--duration-fast) var(--ease-out);
}

.card:hover {
  box-shadow: var(--shadow-lg);
  transform: translateY(-2px);
}

/* Link hover — underline reveal */
.link {
  text-decoration: none;
  background-image: linear-gradient(currentColor, currentColor);
  background-size: 0% 1px;
  background-repeat: no-repeat;
  background-position: left bottom;
  transition: background-size var(--duration-base) var(--ease-out);
}

.link:hover { background-size: 100% 1px; }
```

### Toggle/Switch Animation
```tsx
// Satisfying toggle with spring physics
<motion.div
  className="toggle-thumb"
  animate={{ x: isOn ? 20 : 0 }}
  transition={{
    type: 'spring',
    stiffness: 500,
    damping: 30,
  }}
/>
```

### Checkbox Check Animation
```css
/* Checkmark draws itself */
.checkbox-check {
  stroke-dasharray: 20;
  stroke-dashoffset: 20;
  transition: stroke-dashoffset 200ms ease-out;
}

input:checked + .checkbox-check {
  stroke-dashoffset: 0;
}
```

### Form Field Focus
```css
/* Focus ring slides in */
.input {
  outline: 2px solid transparent;
  outline-offset: 2px;
  transition: 
    outline-color var(--duration-fast) var(--ease-out),
    box-shadow var(--duration-fast) var(--ease-out);
}

.input:focus-visible {
  outline-color: var(--ring);
  box-shadow: 0 0 0 3px color-mix(in srgb, var(--ring) 20%, transparent);
}
```

---

## Enter/Exit Animations

### Appear from Below (Standard)
```css
@keyframes slideUpFade {
  from { 
    opacity: 0; 
    transform: translateY(8px); 
  }
  to { 
    opacity: 1; 
    transform: translateY(0); 
  }
}

.animate-enter {
  animation: slideUpFade var(--duration-base) var(--ease-out) both;
}
```

### Staggered List Entrance
```tsx
// With Framer Motion
const container = {
  hidden: {},
  show: {
    transition: {
      staggerChildren: 0.05, // 50ms between each item
    },
  },
};

const item = {
  hidden: { opacity: 0, y: 10 },
  show: { 
    opacity: 1, 
    y: 0,
    transition: { duration: 0.25, ease: 'easeOut' }
  },
};

function AnimatedList({ items }) {
  const prefersReduced = useReducedMotion();
  
  return (
    <motion.ul 
      variants={prefersReduced ? {} : container} 
      initial="hidden" 
      animate="show"
    >
      {items.map(item => (
        <motion.li key={item.id} variants={prefersReduced ? {} : item}>
          <ListItem data={item} />
        </motion.li>
      ))}
    </motion.ul>
  );
}
```

### Scale Pop (For Modals, Dropdowns)
```css
@keyframes scalePop {
  from { 
    opacity: 0; 
    transform: scale(0.95); 
  }
  to { 
    opacity: 1; 
    transform: scale(1); 
  }
}

.modal-content {
  animation: scalePop var(--duration-base) var(--ease-spring) both;
  transform-origin: center;
}
```

### Slide In from Side (Drawers/Sidebars)
```css
@keyframes slideInRight {
  from { transform: translateX(100%); }
  to   { transform: translateX(0); }
}

@keyframes slideOutRight {
  from { transform: translateX(0); }
  to   { transform: translateX(100%); }
}

.drawer[data-state="open"]   { animation: slideInRight var(--duration-slow) var(--ease-decelerate) both; }
.drawer[data-state="closed"] { animation: slideOutRight var(--duration-slow) var(--ease-accelerate) both; }
```

---

## Page & Route Transitions

### Fade Transition (Safest)
```tsx
// With react-router-dom + Framer Motion
function AnimatedOutlet() {
  const location = useLocation();
  const reducedMotion = useReducedMotion();
  
  return (
    <AnimatePresence mode="wait">
      <motion.div
        key={location.pathname}
        initial={{ opacity: 0 }}
        animate={{ opacity: 1 }}
        exit={{ opacity: 0 }}
        transition={{ duration: reducedMotion ? 0 : 0.2 }}
      >
        <Outlet />
      </motion.div>
    </AnimatePresence>
  );
}
```

### Slide Route Transition
```tsx
const pageVariants = {
  initial: { opacity: 0, x: 20 },
  animate: { opacity: 1, x: 0 },
  exit:    { opacity: 0, x: -20 },
};

function Page({ children }) {
  const reducedMotion = useReducedMotion();
  
  return (
    <motion.div
      variants={reducedMotion ? {} : pageVariants}
      initial="initial"
      animate="animate"
      exit="exit"
      transition={{ duration: 0.25, ease: 'easeInOut' }}
    >
      {children}
    </motion.div>
  );
}
```

---

## Loading & Progress Animations

### Skeleton Pulse
```css
@keyframes skeleton-pulse {
  0%, 100% { opacity: 1; }
  50%       { opacity: 0.4; }
}

.skeleton {
  background: var(--muted);
  border-radius: 4px;
  animation: skeleton-pulse 1.5s ease-in-out infinite;
}

/* Match actual content layout */
.skeleton-text     { height: 1em; width: 80%; margin-bottom: 0.5em; }
.skeleton-heading  { height: 1.5em; width: 60%; }
.skeleton-avatar   { width: 40px; height: 40px; border-radius: 50%; }
.skeleton-image    { aspect-ratio: 16/9; width: 100%; }
```

### Shimmer Effect
```css
@keyframes shimmer {
  0%   { background-position: -200% center; }
  100% { background-position: 200% center; }
}

.skeleton-shimmer {
  background: linear-gradient(
    90deg,
    var(--muted) 25%,
    color-mix(in srgb, var(--muted) 50%, var(--background)) 50%,
    var(--muted) 75%
  );
  background-size: 200% 100%;
  animation: shimmer 1.5s linear infinite;
}
```

### Spinner
```css
@keyframes spin {
  from { transform: rotate(0deg); }
  to   { transform: rotate(360deg); }
}

.spinner {
  width: 20px;
  height: 20px;
  border: 2px solid color-mix(in srgb, var(--primary) 25%, transparent);
  border-top-color: var(--primary);
  border-radius: 50%;
  animation: spin 0.6s linear infinite;
}

/* Reduce motion: static representation */
@media (prefers-reduced-motion: reduce) {
  .spinner { animation: none; border-top-color: var(--primary); }
}
```

### Progress Bar
```tsx
function ProgressBar({ value }: { value: number }) {
  return (
    <div 
      role="progressbar" 
      aria-valuenow={value} 
      aria-valuemin={0} 
      aria-valuemax={100}
      className="progress-track"
    >
      <motion.div
        className="progress-fill"
        animate={{ width: `${value}%` }}
        transition={{ duration: 0.4, ease: 'easeOut' }}
      />
    </div>
  );
}
```

---

## Scroll-Triggered Animations

```tsx
// Intersection Observer based — no JS animation library needed
function useInView(threshold = 0.1) {
  const ref = useRef<HTMLElement>(null);
  const [inView, setInView] = useState(false);
  
  useEffect(() => {
    const el = ref.current;
    if (!el) return;
    
    const observer = new IntersectionObserver(
      ([entry]) => { if (entry.isIntersecting) setInView(true); },
      { threshold }
    );
    
    observer.observe(el);
    return () => observer.unobserve(el);
  }, [threshold]);
  
  return { ref, inView };
}

// Usage with Framer Motion
function AnimatedSection({ children }) {
  const { ref, inView } = useInView();
  const reducedMotion = useReducedMotion();
  
  return (
    <motion.div
      ref={ref}
      initial={{ opacity: 0, y: reducedMotion ? 0 : 30 }}
      animate={inView ? { opacity: 1, y: 0 } : {}}
      transition={{ duration: 0.5, ease: 'easeOut' }}
    >
      {children}
    </motion.div>
  );
}
```

---

## Transition Properties

### What to Animate (Safe)
```css
/* GPU-accelerated — smooth, no layout thrash */
.safe {
  transition: 
    opacity,
    transform,
    filter,
    backdrop-filter;
}
```

### What NOT to Animate (Causes Layout Thrash)
```css
/* ❌ These cause reflow — janky on most devices */
.dangerous {
  transition: 
    width,      /* use transform: scaleX() instead */
    height,     /* use transform: scaleY() instead */
    margin,     /* avoid animating */
    padding,    /* avoid animating */
    top,        /* use transform: translateY() instead */
    left,       /* use transform: translateX() instead */
    border-width; /* avoid animating */
}
```

### will-change (Use Sparingly)
```css
/* Use ONLY when you know the element will animate */
/* Overuse wastes GPU memory */
.will-animate {
  will-change: transform, opacity;
}

/* Remove after animation completes */
.animation-complete {
  will-change: auto;
}
```

---

## Animation Anti-Patterns

| Anti-Pattern | Problem | Fix |
|-------------|---------|-----|
| Animation > 500ms | Feels slow, not responsive | Cap at 350ms for UI transitions |
| Animating width/height | Causes reflow, jank | Use transform: scale() |
| Infinite loop without pause | Distracts, annoys | Only loop ambient/background animations |
| No ease — all linear | Feels mechanical and robotic | Use ease-out for enter, ease-in for exit |
| Animating every element | Visual noise, slow | Choose max 2–3 animated elements per view |
| No reduced-motion support | Causes nausea for vestibular disorders | Always add @media prefers-reduced-motion |
| Bounce/spring everywhere | Unprofessional, tiring | Reserve spring for specific delight moments |
| Hover animations on mobile | Mobile has no hover | Suppress motion-sensitive hover effects on touch |
| Transition: all | Animates unexpected properties, performance risk | Always specify exact properties |

---

## Motion Checklist

- [ ] `@media (prefers-reduced-motion: reduce)` applied globally
- [ ] All transitions specify exact properties (not `transition: all`)
- [ ] Enter animations use `ease-out`; exit animations use `ease-in`
- [ ] No transition > 500ms for user-triggered UI
- [ ] Spinners/progress remain visible (not animated off) in reduced-motion
- [ ] Staggered list animations: max 50ms stagger delay
- [ ] GPU-accelerated properties only (`opacity`, `transform`, `filter`)
- [ ] `will-change` removed after animation completes
- [ ] Scroll-triggered animations use IntersectionObserver, not scroll events
- [ ] Page transitions don't prevent interaction during animation
