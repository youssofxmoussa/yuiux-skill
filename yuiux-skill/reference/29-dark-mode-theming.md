# 29 — Dark Mode & Theming

> Dark mode is not just inverting colors. It's redesigning every surface, shadow, and relationship for a different visual environment.

---

## Dark Mode vs. Dark Theme

```
Dark mode:  User system preference (prefers-color-scheme)
Dark theme: User choice within the app (class-based toggle)

Best practice: Support both — respect system preference by default,
               allow override with user setting.
```

---

## Implementation Strategy

```css
/* Strategy 1: CSS-only with @media (simpler, no JS flash) */
:root { --bg: #ffffff; --fg: #0f172a; }

@media (prefers-color-scheme: dark) {
  :root { --bg: #020617; --fg: #f8fafc; }
}

/* Strategy 2: Class-based (allows user override) */
:root { --bg: #ffffff; }
.dark { --bg: #020617; }

/* Strategy 3: Combined (best practice) */
:root { --bg: #ffffff; }

@media (prefers-color-scheme: dark) {
  :root:not([data-theme="light"]) { --bg: #020617; }
}

[data-theme="dark"] { --bg: #020617; }
```

```tsx
// Theme toggle with localStorage persistence + system preference
function useTheme() {
  const [theme, setTheme] = useState<'light' | 'dark' | 'system'>(() => {
    if (typeof window === 'undefined') return 'system';
    return (localStorage.getItem('theme') as 'light' | 'dark' | 'system') ?? 'system';
  });
  
  useEffect(() => {
    const root = document.documentElement;
    const prefersDark = window.matchMedia('(prefers-color-scheme: dark)').matches;
    const isDark = theme === 'dark' || (theme === 'system' && prefersDark);
    
    root.setAttribute('data-theme', isDark ? 'dark' : 'light');
    root.classList.toggle('dark', isDark); // for Tailwind dark: classes
    localStorage.setItem('theme', theme);
  }, [theme]);
  
  return { theme, setTheme };
}

// Theme switcher UI
function ThemeSwitcher() {
  const { theme, setTheme } = useTheme();
  
  return (
    <div role="group" aria-label="Color theme">
      {([
        { value: 'light', icon: SunIcon,      label: 'Light' },
        { value: 'dark',  icon: MoonIcon,     label: 'Dark' },
        { value: 'system',icon: MonitorIcon,  label: 'System' },
      ] as const).map(option => {
        const Icon = option.icon;
        return (
          <button
            key={option.value}
            type="button"
            aria-pressed={theme === option.value}
            onClick={() => setTheme(option.value)}
            aria-label={`${option.label} theme`}
            className={`theme-btn ${theme === option.value ? 'active' : ''}`}
          >
            <Icon aria-hidden className="w-4 h-4" />
            <span>{option.label}</span>
          </button>
        );
      })}
    </div>
  );
}
```

---

## Prevent Flash of Unstyled Content (FOUC)

```html
<!-- Inline script before any CSS loads to apply theme immediately -->
<!-- This MUST run before body renders -->
<head>
  <script>
    (function() {
      const stored = localStorage.getItem('theme');
      const prefersDark = window.matchMedia('(prefers-color-scheme: dark)').matches;
      const isDark = stored === 'dark' || (stored !== 'light' && prefersDark);
      document.documentElement.classList.toggle('dark', isDark);
      document.documentElement.setAttribute('data-theme', isDark ? 'dark' : 'light');
    })();
  </script>
</head>
```

---

## Complete Dark Mode Color System

```css
:root {
  /* Backgrounds */
  --bg-canvas:    #ffffff;   /* page background */
  --bg-surface:   #f8fafc;   /* card, panel */
  --bg-overlay:   #f1f5f9;   /* tooltip, popover */
  --bg-sunken:    #f1f5f9;   /* inputs, code blocks */
  
  /* Text */
  --text-primary:   #0f172a;
  --text-secondary: #475569;
  --text-muted:     #94a3b8;
  --text-disabled:  #cbd5e1;
  --text-inverted:  #ffffff;
  
  /* Brand */
  --brand-primary:  #2563eb;
  --brand-hover:    #1d4ed8;
  --brand-subtle:   #eff6ff;
  
  /* Status */
  --status-success: #16a34a;
  --status-success-bg: #f0fdf4;
  --status-warning: #d97706;
  --status-warning-bg: #fffbeb;
  --status-error:   #dc2626;
  --status-error-bg: #fef2f2;
  
  /* Borders */
  --border-subtle: #e2e8f0;
  --border-default:#cbd5e1;
  --border-strong: #94a3b8;
  
  /* Shadows (light mode: subtle) */
  --shadow-sm: 0 1px 2px rgba(0,0,0,0.05);
  --shadow-md: 0 4px 6px rgba(0,0,0,0.1);
  --shadow-lg: 0 10px 15px rgba(0,0,0,0.1);
}

[data-theme="dark"] {
  /* Backgrounds — NOT just inverted, carefully chosen dark values */
  --bg-canvas:    #020617;   /* darkest — page background */
  --bg-surface:   #0f172a;   /* card, panel — slightly lighter */
  --bg-overlay:   #1e293b;   /* tooltip, popover — most elevated */
  --bg-sunken:    #0a0f1e;   /* inputs, code blocks — slightly darker */
  
  /* Text — NOT pure white, slightly off-white for eye comfort */
  --text-primary:   #f1f5f9;  /* slightly warm off-white */
  --text-secondary: #94a3b8;  /* medium gray */
  --text-muted:     #475569;  /* dark gray */
  --text-disabled:  #334155;  /* very dark gray */
  --text-inverted:  #0f172a;
  
  /* Brand — LIGHTER in dark mode for visibility */
  --brand-primary:  #3b82f6;  /* blue-500 (lighter than blue-600) */
  --brand-hover:    #60a5fa;  /* blue-400 */
  --brand-subtle:   rgba(37, 99, 235, 0.15);
  
  /* Status — LIGHTER in dark mode */
  --status-success: #22c55e;  /* green-500 */
  --status-success-bg: rgba(34, 197, 94, 0.1);
  --status-warning: #f59e0b;  /* amber-500 */
  --status-warning-bg: rgba(245, 158, 11, 0.1);
  --status-error:   #ef4444;  /* red-500 */
  --status-error-bg: rgba(239, 68, 68, 0.1);
  
  /* Borders — dark, subtle */
  --border-subtle: #1e293b;
  --border-default:#334155;
  --border-strong: #475569;
  
  /* Shadows — much more opaque in dark mode (light doesn't scatter) */
  --shadow-sm: 0 1px 2px rgba(0,0,0,0.4);
  --shadow-md: 0 4px 6px rgba(0,0,0,0.5);
  --shadow-lg: 0 10px 15px rgba(0,0,0,0.6);
}
```

---

## Dark Mode Gotchas

### 1. Images Need Treatment

```css
/* Reduce brightness of photos in dark mode */
[data-theme="dark"] img:not([data-no-dark-filter]) {
  filter: brightness(0.85);
}

/* Don't filter logos, icons, illustrations */
[data-theme="dark"] img[data-no-dark-filter] {
  filter: none;
}
```

### 2. Transparent Shadows Become Invisible

```css
/* ❌ Light mode shadow using transparency — invisible in dark mode */
.card {
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
}

/* ✅ Use CSS variable — swap values in dark mode */
.card {
  box-shadow: var(--shadow-md);
}
/* Dark mode value is 0 4px 6px rgba(0, 0, 0, 0.5) */
```

### 3. White Backgrounds on Cards Become Invisible

```css
/* ❌ Hardcoded white cards in dark mode just disappear */
.card { background: white; }

/* ✅ Use semantic token */
.card { background: var(--bg-surface); }
/* Dark mode: --bg-surface: #0f172a */
```

### 4. Border Colors Need to Change

```css
/* In light mode, subtle borders are just visible against white */
/* In dark mode, you need LIGHTER borders to be visible against dark */

/* ❌ Same border doesn't work */
.card { border: 1px solid #e2e8f0; }

/* ✅ Token-based */
.card { border: 1px solid var(--border-subtle); }
/* Dark: --border-subtle: #1e293b (clearly different from #020617 background) */
```

### 5. Focus Rings Need Inversion

```css
/* Light mode: blue ring on white bg is fine */
/* Dark mode: same blue ring on dark bg might be invisible */

:focus-visible {
  outline: 2px solid var(--brand-primary);
  /* Light: #2563eb on white ✅  Dark: #3b82f6 on dark ✅ (already adjusted in token) */
  outline-offset: 2px;
}
```

### 6. Third-Party Embeds

```css
/* Google Maps, video players, etc. don't respect dark mode */
/* Add a hue-rotate to invert map tiles in dark mode */
[data-theme="dark"] .google-map {
  filter: invert(90%) hue-rotate(180deg);
}
/* Invert back: map objects look normal again, bg is dark */
```

---

## Theming with Tailwind

```tsx
// tailwind.config.ts
export default {
  darkMode: 'class', // or 'media' for system-only
  // ...
};

// Component using dark: variants
function Card({ children }) {
  return (
    <div className="
      bg-white dark:bg-slate-900
      border border-slate-200 dark:border-slate-800
      text-slate-900 dark:text-slate-100
      shadow-sm dark:shadow-slate-900/50
      rounded-lg p-6
    ">
      {children}
    </div>
  );
}
```

---

## Testing Dark Mode

```
Manual checklist:
  □ Toggle dark mode — no FOUC (flash of unstyled content)
  □ All text readable (contrast ≥ 4.5:1)
  □ All borders visible
  □ All focus rings visible
  □ Cards visually distinct from page background
  □ Status colors (success/warning/error) visible
  □ Images not too bright / too dark
  □ Shadows visible (not invisible)
  □ Input backgrounds distinct from card backgrounds
  □ Code blocks have appropriate dark styling
  
Automated:
  Run contrast checks in both modes with axe DevTools
```

---

## Dark Mode Checklist

- [ ] System preference respected by default
- [ ] User can override system preference and choice is persisted
- [ ] No FOUC — theme applied before first paint (inline script)
- [ ] All semantic color tokens defined for both modes
- [ ] Background elevation hierarchy maintained in dark (not all same dark)
- [ ] Shadows have higher opacity in dark mode
- [ ] Image brightness reduced in dark mode
- [ ] Focus rings remain visible in dark mode
- [ ] Third-party embeds handled (maps, video players)
- [ ] Status colors lighter in dark mode
- [ ] Borders lighter relative to background in dark mode
- [ ] Automated contrast check in dark mode
