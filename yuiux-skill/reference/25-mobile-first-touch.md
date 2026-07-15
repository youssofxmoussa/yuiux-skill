# 25 — Mobile-First & Touch UX

> Mobile is not a limited version of your product. It's the primary version for most users. Design for it first, then scale up.

---

## Mobile Design Principles

### 1. Thumb Zone
The natural resting position of the thumb defines what's easy to reach:
```
Easy reach:   Bottom 40% of screen — primary actions
Reachable:    Middle 30% — secondary actions  
Hard reach:   Top 30% — back buttons, hamburger (tolerated)
```

**Implication:** Primary CTAs live at the bottom. Navigation lives at the bottom. Destructive actions require extra reach (a feature, not a bug).

### 2. Touch Target Rules
- Minimum: 44×44px (Apple HIG) / 48×48dp (Material)
- Optimal: 56-64px for primary actions
- Minimum gap between adjacent targets: 8px
- Never: two interactive elements touching each other

### 3. Swipe Gestures (Supplement, Don't Replace)
Gestures should augment tap interactions, never be the only way to perform an action.

---

## Touch-Friendly CSS

```css
/* Prevent accidental text selection on interactive elements */
.btn, .card--interactive, .nav-item {
  user-select: none;
  -webkit-user-select: none;
}

/* Suppress ugly tap highlight on mobile */
* {
  -webkit-tap-highlight-color: transparent;
}

/* Always define :active state — mobile :hover is unreliable */
.btn:active {
  transform: scale(0.97);
  opacity: 0.9;
}

/* Smooth iOS scrolling */
.scrollable {
  overflow-y: auto;
  -webkit-overflow-scrolling: touch;
  overscroll-behavior: contain; /* prevent scroll chaining */
}

/* iOS safe area (notch and home indicator) */
.bottom-nav {
  padding-bottom: env(safe-area-inset-bottom);
}

.mobile-header {
  padding-top: max(16px, env(safe-area-inset-top));
}

/* Prevent content jumping when URL bar hides */
.full-height {
  min-height: 100dvh; /* dynamic viewport height */
}
```

---

## Mobile Form UX

```html
<!-- Always set input types and attributes for mobile keyboards -->

<!-- Email keyboard + @ button -->
<input type="email" inputmode="email" autocomplete="email" />

<!-- Number keyboard (no decimal) -->
<input type="text" inputmode="numeric" pattern="[0-9]*" />

<!-- Phone keyboard -->
<input type="tel" inputmode="tel" autocomplete="tel" />

<!-- URL keyboard with .com button -->
<input type="url" inputmode="url" autocomplete="url" />

<!-- Decimal keyboard -->
<input type="text" inputmode="decimal" />

<!-- Search keyboard with Search action button -->
<input type="search" inputmode="search" />
```

```css
/* Prevent iOS from auto-zooming in on focus (input < 16px triggers zoom) */
/* ALWAYS use 16px or larger for input text on mobile */
input, select, textarea {
  font-size: 16px; /* prevents iOS zoom */
}

/* Or via viewport meta */
/* <meta name="viewport" content="width=device-width, initial-scale=1, maximum-scale=1"> */
/* CAUTION: maximum-scale=1 prevents user zoom — accessibility concern */
/* Better: just make inputs 16px */
```

---

## Swipe Gestures

```tsx
// Pull-to-refresh
function usePullToRefresh(onRefresh: () => Promise<void>) {
  const [pulling, setPulling] = useState(false);
  const [pullDistance, setPullDistance] = useState(0);
  const startY = useRef<number | null>(null);
  const THRESHOLD = 80;
  
  function handleTouchStart(e: TouchEvent) {
    if (window.scrollY > 0) return; // only trigger at top
    startY.current = e.touches[0].clientY;
  }
  
  function handleTouchMove(e: TouchEvent) {
    if (startY.current === null) return;
    const delta = e.touches[0].clientY - startY.current;
    if (delta > 0) {
      setPullDistance(Math.min(delta, THRESHOLD * 1.5));
      if (delta > THRESHOLD) setPulling(true);
    }
  }
  
  async function handleTouchEnd() {
    if (pulling) {
      setPulling(false);
      await onRefresh();
    }
    setPullDistance(0);
    startY.current = null;
  }
  
  return { pullDistance, pulling, handleTouchStart, handleTouchMove, handleTouchEnd };
}

// Swipe-to-delete
function SwipeableListItem({ onDelete, children }) {
  const [swipeX, setSwipeX] = useState(0);
  const startX = useRef<number | null>(null);
  const DELETE_THRESHOLD = 80;
  
  return (
    <div
      className="swipeable-item"
      style={{ transform: `translateX(${Math.min(swipeX, 0)}px)` }}
      onTouchStart={e => { startX.current = e.touches[0].clientX; }}
      onTouchMove={e => {
        if (startX.current === null) return;
        const delta = e.touches[0].clientX - startX.current;
        if (delta < 0) setSwipeX(delta);
      }}
      onTouchEnd={() => {
        if (swipeX < -DELETE_THRESHOLD) onDelete();
        else setSwipeX(0);
        startX.current = null;
      }}
    >
      {/* Delete action revealed on swipe */}
      <div
        className="swipe-action delete"
        aria-hidden
        style={{ opacity: Math.min(-swipeX / DELETE_THRESHOLD, 1) }}
      >
        <TrashIcon /> Delete
      </div>
      {children}
    </div>
  );
}
```

---

## Bottom Sheet

```tsx
function BottomSheet({ open, onClose, title, children, snapPoints = ['50%', '90%'] }) {
  const [snap, setSnap] = useState(0);
  const sheetRef = useRef<HTMLDivElement>(null);
  const startY = useRef<number | null>(null);
  
  // Drag to resize/close
  function handleDragStart(e: React.TouchEvent) {
    startY.current = e.touches[0].clientY;
  }
  
  function handleDragEnd(e: React.TouchEvent) {
    if (startY.current === null) return;
    const delta = e.changedTouches[0].clientY - startY.current;
    
    if (delta > 100 && snap === 0) {
      onClose(); // drag down far enough to close
    } else if (delta > 50) {
      setSnap(Math.max(0, snap - 1)); // snap down
    } else if (delta < -50) {
      setSnap(Math.min(snapPoints.length - 1, snap + 1)); // snap up
    }
    
    startY.current = null;
  }
  
  return (
    <>
      {open && <div className="dialog-backdrop" onClick={onClose} aria-hidden />}
      <div
        ref={sheetRef}
        role="dialog"
        aria-modal="true"
        aria-labelledby="sheet-title"
        className={`bottom-sheet ${open ? 'open' : ''}`}
        style={{ '--snap': snapPoints[snap] } as React.CSSProperties}
        aria-hidden={!open}
      >
        {/* Drag handle */}
        <div
          className="sheet-handle"
          onTouchStart={handleDragStart}
          onTouchEnd={handleDragEnd}
          aria-hidden
        >
          <div className="sheet-handle-bar" />
        </div>
        
        <div className="sheet-header">
          <h2 id="sheet-title">{title}</h2>
          <button onClick={onClose} aria-label="Close">
            <XIcon aria-hidden />
          </button>
        </div>
        
        <div className="sheet-body">{children}</div>
      </div>
    </>
  );
}
```

```css
.bottom-sheet {
  position: fixed;
  bottom: 0;
  left: 0;
  right: 0;
  background: hsl(var(--background));
  border-radius: 20px 20px 0 0;
  border-top: 1px solid hsl(var(--border));
  z-index: var(--z-modal);
  display: flex;
  flex-direction: column;
  
  /* Height driven by snap point */
  height: var(--snap, 50%);
  transform: translateY(100%);
  transition: transform 350ms cubic-bezier(0.32, 0.72, 0, 1), height 350ms;
  will-change: transform;
  
  padding-bottom: env(safe-area-inset-bottom);
}

.bottom-sheet.open { transform: translateY(0); }

.sheet-handle {
  padding: 12px;
  display: flex;
  justify-content: center;
  touch-action: none;
  cursor: grab;
}

.sheet-handle-bar {
  width: 36px;
  height: 4px;
  background: hsl(var(--border));
  border-radius: 2px;
}

.sheet-body {
  flex: 1;
  overflow-y: auto;
  -webkit-overflow-scrolling: touch;
  padding: 0 20px 20px;
}

@media (prefers-reduced-motion: reduce) {
  .bottom-sheet { transition: none; }
}
```

---

## Mobile Navigation

```tsx
// Tab bar with badges
function MobileTabBar() {
  const location = useLocation();
  const tabs = [
    { href: '/home',         icon: HomeIcon,    label: 'Home'     },
    { href: '/search',       icon: SearchIcon,  label: 'Search'   },
    { href: '/notifications',icon: BellIcon,    label: 'Alerts', badge: 3 },
    { href: '/profile',      icon: UserIcon,    label: 'Profile'  },
  ];
  
  return (
    <nav className="mobile-tab-bar" aria-label="Main navigation">
      {tabs.map(tab => {
        const active = location.pathname === tab.href;
        const Icon = tab.icon;
        
        return (
          <Link
            key={tab.href}
            to={tab.href}
            aria-current={active ? 'page' : undefined}
            aria-label={`${tab.label}${tab.badge ? `, ${tab.badge} notifications` : ''}`}
            className={`tab-bar-item ${active ? 'active' : ''}`}
          >
            <div className="tab-bar-icon-wrapper">
              <Icon aria-hidden className="tab-bar-icon" />
              {tab.badge && (
                <span className="tab-badge" aria-hidden>
                  {tab.badge > 99 ? '99+' : tab.badge}
                </span>
              )}
            </div>
            <span className="tab-bar-label">{tab.label}</span>
          </Link>
        );
      })}
    </nav>
  );
}
```

---

## Haptic Feedback (Native Mobile / PWA)

```tsx
// Haptic feedback for mobile web (where supported)
function useHaptics() {
  function impact(style: 'light' | 'medium' | 'heavy' = 'light') {
    if (!('vibrate' in navigator)) return;
    const patterns = { light: [10], medium: [20], heavy: [40] };
    navigator.vibrate(patterns[style]);
  }
  
  function success() {
    if (!('vibrate' in navigator)) return;
    navigator.vibrate([10, 50, 20]);
  }
  
  function error() {
    if (!('vibrate' in navigator)) return;
    navigator.vibrate([20, 30, 20, 30, 40]);
  }
  
  return { impact, success, error };
}

// Usage on delete confirmation:
function DeleteButton({ onDelete }) {
  const haptics = useHaptics();
  
  return (
    <button
      type="button"
      onClick={() => {
        haptics.impact('medium');
        onDelete();
      }}
    >
      Delete
    </button>
  );
}
```

---

## Mobile Checklist

- [ ] Touch targets ≥ 44×44px for all interactive elements
- [ ] 8px minimum gap between adjacent touch targets
- [ ] Primary actions in bottom 40% of screen (thumb zone)
- [ ] Bottom navigation on mobile (not sidebar)
- [ ] All input elements use 16px font size (prevents iOS zoom)
- [ ] `inputmode` attributes set for appropriate keyboard types
- [ ] `autocomplete` attributes on all form fields
- [ ] Bottom sheet used instead of modal for long content on mobile
- [ ] Safe area padding applied (env(safe-area-inset-*))
- [ ] `dvh` used instead of `vh` for full-height elements
- [ ] Pull-to-refresh implemented for list content
- [ ] `overscroll-behavior: contain` on scrollable containers
- [ ] `-webkit-tap-highlight-color: transparent` set globally
- [ ] `:active` state defined for all interactive elements
- [ ] No hover-only interactions (no :hover without :focus fallback)
- [ ] Tested on real device (not just DevTools emulator)
- [ ] Tested with one hand (right thumb only)
