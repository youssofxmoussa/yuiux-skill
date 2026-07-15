# 08 — Navigation & Information Architecture

> Navigation is the skeleton of your app. Users shouldn't feel it — they should feel the content. Navigation fails when users notice it, when they can't find things, or when they can't tell where they are.

---

## The Three Navigation Questions

Every user, at every moment, must be able to answer:
1. **Where am I?** (current location indicator)
2. **Where can I go?** (available destinations visible)
3. **How did I get here?** (path back)

If any of these is unclear, the navigation has failed.

---

## Navigation Patterns

### Top Navigation Bar
Best for: marketing sites, single-level apps, 4–7 primary destinations

```tsx
function TopNav({ items, currentPath }) {
  return (
    <nav aria-label="Main navigation">
      <ul role="list" className="nav-list">
        {items.map(item => {
          const isCurrent = currentPath === item.href;
          return (
            <li key={item.href}>
              <a
                href={item.href}
                aria-current={isCurrent ? 'page' : undefined}
                className={`nav-link ${isCurrent ? 'nav-link--active' : ''}`}
              >
                {item.label}
              </a>
            </li>
          );
        })}
      </ul>
    </nav>
  );
}
```

```css
.nav-link {
  color: var(--foreground-muted);
  text-decoration: none;
  font-weight: 500;
  font-size: 14px;
  padding: 6px 12px;
  border-radius: var(--radius);
  transition: color 150ms, background 150ms;
}

.nav-link:hover {
  color: var(--foreground);
  background: var(--accent);
}

/* Active state — must be visually distinct */
.nav-link--active,
.nav-link[aria-current="page"] {
  color: var(--primary);
  font-weight: 600;
  background: var(--primary-subtle);
}
```

### Sidebar Navigation
Best for: apps with many sections, data-heavy tools, dashboards

```tsx
function Sidebar({ sections, currentPath }) {
  return (
    <aside>
      <nav aria-label="Application navigation">
        {sections.map(section => (
          <div key={section.title} className="nav-section">
            {section.title && (
              <p className="nav-section-title">{section.title}</p>
            )}
            <ul role="list">
              {section.items.map(item => {
                const isCurrent = currentPath.startsWith(item.href);
                return (
                  <li key={item.href}>
                    <a
                      href={item.href}
                      aria-current={isCurrent ? 'page' : undefined}
                      className={`sidebar-link ${isCurrent ? 'sidebar-link--active' : ''}`}
                    >
                      {item.icon && (
                        <item.icon className="sidebar-link-icon" aria-hidden />
                      )}
                      <span>{item.label}</span>
                      {item.badge && (
                        <span className="nav-badge" aria-label={`${item.badge} items`}>
                          {item.badge}
                        </span>
                      )}
                    </a>
                    {/* Sub-items */}
                    {isCurrent && item.children && (
                      <ul className="nav-sub-list">
                        {item.children.map(child => (
                          <li key={child.href}>
                            <a
                              href={child.href}
                              aria-current={currentPath === child.href ? 'page' : undefined}
                              className="sidebar-sub-link"
                            >
                              {child.label}
                            </a>
                          </li>
                        ))}
                      </ul>
                    )}
                  </li>
                );
              })}
            </ul>
          </div>
        ))}
      </nav>
    </aside>
  );
}
```

### Tab Navigation
Best for: content switching within a page, 2–6 tabs

```tsx
function Tabs({ tabs, activeTab, onTabChange }) {
  return (
    <div>
      <div role="tablist" aria-label="Content sections">
        {tabs.map(tab => (
          <button
            key={tab.id}
            role="tab"
            id={`tab-${tab.id}`}
            aria-controls={`panel-${tab.id}`}
            aria-selected={activeTab === tab.id}
            onClick={() => onTabChange(tab.id)}
            className={`tab ${activeTab === tab.id ? 'tab--active' : ''}`}
          >
            {tab.label}
            {tab.count !== undefined && (
              <span className="tab-count">{tab.count}</span>
            )}
          </button>
        ))}
      </div>
      
      {tabs.map(tab => (
        <div
          key={tab.id}
          role="tabpanel"
          id={`panel-${tab.id}`}
          aria-labelledby={`tab-${tab.id}`}
          hidden={activeTab !== tab.id}
          tabIndex={0}
        >
          {tab.content}
        </div>
      ))}
    </div>
  );
}
```

### Mobile Bottom Navigation
Best for: mobile apps, 3–5 primary destinations

```tsx
function BottomNav({ items, currentPath }) {
  return (
    <nav aria-label="Main navigation" className="bottom-nav">
      {items.map(item => {
        const isCurrent = currentPath.startsWith(item.href);
        return (
          <a
            key={item.href}
            href={item.href}
            aria-current={isCurrent ? 'page' : undefined}
            className={`bottom-nav-item ${isCurrent ? 'bottom-nav-item--active' : ''}`}
          >
            <item.icon aria-hidden className="bottom-nav-icon" />
            <span className="bottom-nav-label">{item.label}</span>
          </a>
        );
      })}
    </nav>
  );
}
```

```css
.bottom-nav {
  position: fixed;
  bottom: 0;
  left: 0;
  right: 0;
  display: flex;
  background: var(--background);
  border-top: 1px solid var(--border);
  padding-bottom: env(safe-area-inset-bottom); /* iPhone notch */
  z-index: var(--z-fixed);
}

.bottom-nav-item {
  flex: 1;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 4px;
  padding: 8px 4px;
  min-height: 56px;
  text-decoration: none;
  color: var(--foreground-muted);
  font-size: 11px;
  font-weight: 500;
  transition: color 150ms;
}

.bottom-nav-item--active,
.bottom-nav-item[aria-current="page"] {
  color: var(--primary);
}
```

---

## Breadcrumbs

Required when: pages are 3+ levels deep, content has a clear hierarchy.

```tsx
function Breadcrumbs({ items }) {
  return (
    <nav aria-label="Breadcrumb">
      <ol role="list" className="breadcrumb-list">
        {items.map((item, index) => {
          const isLast = index === items.length - 1;
          return (
            <li key={item.href ?? item.label}>
              {isLast ? (
                <span aria-current="page" className="breadcrumb-current">
                  {item.label}
                </span>
              ) : (
                <>
                  <a href={item.href} className="breadcrumb-link">
                    {item.label}
                  </a>
                  <span aria-hidden className="breadcrumb-separator">/</span>
                </>
              )}
            </li>
          );
        })}
      </ol>
    </nav>
  );
}
```

```css
.breadcrumb-list {
  display: flex;
  align-items: center;
  flex-wrap: wrap;
  gap: 4px;
  list-style: none;
  padding: 0;
  margin: 0;
  font-size: 14px;
}

.breadcrumb-link {
  color: var(--foreground-muted);
  text-decoration: none;
}

.breadcrumb-link:hover { color: var(--foreground); text-decoration: underline; }

.breadcrumb-separator {
  color: var(--foreground-subtle);
  margin-inline: 4px;
}

.breadcrumb-current {
  color: var(--foreground);
  font-weight: 500;
}
```

---

## Skip Navigation Link

Required for keyboard and screen reader accessibility:

```tsx
function SkipLink() {
  return (
    <a 
      href="#main-content"
      className="skip-link"
    >
      Skip to main content
    </a>
  );
}
```

```css
.skip-link {
  position: absolute;
  left: -9999px;
  top: 8px;
  z-index: 9999;
  padding: 8px 16px;
  background: var(--primary);
  color: var(--primary-foreground);
  border-radius: 4px;
  text-decoration: none;
  font-weight: 600;
  font-size: 14px;
}

.skip-link:focus {
  left: 8px; /* visible when focused */
}
```

```html
<!-- In your page structure -->
<body>
  <SkipLink />
  <header>...</header>
  <main id="main-content" tabIndex="-1">
    <!-- main content -->
  </main>
</body>
```

---

## Active State Rules

```css
/* Active state requirements:
   1. Must be different from hover state
   2. Must be visually obvious (not just a color shade change)
   3. Must remain when the element loses keyboard focus
*/

/* ❌ Only color change — barely noticeable */
.nav-link--active { color: blue; }

/* ✅ Multiple visual signals */
.nav-link--active {
  color: var(--primary);
  font-weight: 600;
  background: var(--primary-subtle);
  /* Optional: left border for sidebar */
  border-left: 3px solid var(--primary);
}

/* Sidebar active indicator */
.sidebar-link--active {
  background: hsl(var(--primary) / 0.1);
  color: hsl(var(--primary));
  font-weight: 600;
  position: relative;
}

.sidebar-link--active::before {
  content: '';
  position: absolute;
  left: -12px; /* outside the sidebar item padding */
  top: 0;
  bottom: 0;
  width: 3px;
  background: hsl(var(--primary));
  border-radius: 0 3px 3px 0;
}
```

---

## Back Button Behavior

```tsx
// Correct back navigation — goes to previous location, not just history.back()
function useBackNavigation(defaultRoute: string) {
  const navigate = useNavigate();
  const location = useLocation();
  
  function goBack() {
    // Check if there's history to go back to
    if (window.history.length > 1 && location.key !== 'default') {
      navigate(-1);
    } else {
      // Fallback to default route if no history
      navigate(defaultRoute);
    }
  }
  
  return goBack;
}

// Usage:
function ItemDetailPage() {
  const goBack = useBackNavigation('/items');
  
  return (
    <div>
      <button onClick={goBack} aria-label="Back to items list">
        <ArrowLeftIcon aria-hidden />
        Back
      </button>
      {/* content */}
    </div>
  );
}
```

---

## Deep Linking

Every state in your app should have a URL. Users should be able to bookmark, share, and refresh any page.

```tsx
// ❌ State hidden in JavaScript — not shareable
const [activeTab, setActiveTab] = useState('overview');

// ✅ Tab state in URL — shareable and bookmarkable
function TabbedPage() {
  const [searchParams, setSearchParams] = useSearchParams();
  const activeTab = searchParams.get('tab') ?? 'overview';
  
  function setTab(tab: string) {
    setSearchParams({ tab }, { replace: true }); // replace to not pollute history
  }
  
  return (
    <Tabs
      tabs={TABS}
      activeTab={activeTab}
      onTabChange={setTab}
    />
  );
}

// ❌ Filter state in component — lost on refresh
const [filters, setFilters] = useState({});

// ✅ Filters in URL search params
function FilteredList() {
  const [searchParams, setSearchParams] = useSearchParams();
  const filters = Object.fromEntries(searchParams.entries());
  
  function updateFilter(key: string, value: string) {
    setSearchParams(prev => {
      const next = new URLSearchParams(prev);
      if (value) next.set(key, value);
      else next.delete(key);
      return next;
    });
  }
}
```

---

## Hamburger Menu

```tsx
function MobileMenu({ items }) {
  const [isOpen, setIsOpen] = useState(false);
  
  // Close on route change
  useEffect(() => setIsOpen(false), [location.pathname]);
  
  // Close on Escape
  useEffect(() => {
    if (!isOpen) return;
    const handler = (e: KeyboardEvent) => {
      if (e.key === 'Escape') setIsOpen(false);
    };
    document.addEventListener('keydown', handler);
    return () => document.removeEventListener('keydown', handler);
  }, [isOpen]);
  
  return (
    <>
      <button
        type="button"
        aria-expanded={isOpen}
        aria-controls="mobile-menu"
        aria-label={isOpen ? 'Close menu' : 'Open menu'}
        onClick={() => setIsOpen(!isOpen)}
      >
        {isOpen ? <XIcon aria-hidden /> : <MenuIcon aria-hidden />}
      </button>
      
      {isOpen && (
        <div
          id="mobile-menu"
          className="mobile-menu"
          role="dialog"
          aria-label="Navigation menu"
          aria-modal="true"
        >
          <nav>
            {items.map(item => (
              <a
                key={item.href}
                href={item.href}
                className="mobile-menu-link"
                onClick={() => setIsOpen(false)}
              >
                {item.label}
              </a>
            ))}
          </nav>
        </div>
      )}
      
      {/* Backdrop */}
      {isOpen && (
        <div
          className="menu-backdrop"
          aria-hidden
          onClick={() => setIsOpen(false)}
        />
      )}
    </>
  );
}
```

---

## Dropdown Navigation

```tsx
function NavDropdown({ trigger, items }) {
  const [open, setOpen] = useState(false);
  const triggerRef = useRef<HTMLButtonElement>(null);
  const menuRef = useRef<HTMLDivElement>(null);
  
  // Close on outside click
  useEffect(() => {
    function handler(e: MouseEvent) {
      if (!menuRef.current?.contains(e.target as Node) &&
          !triggerRef.current?.contains(e.target as Node)) {
        setOpen(false);
      }
    }
    if (open) document.addEventListener('mousedown', handler);
    return () => document.removeEventListener('mousedown', handler);
  }, [open]);
  
  // Keyboard navigation within dropdown
  function handleKeyDown(e: KeyboardEvent) {
    if (e.key === 'Escape') { setOpen(false); triggerRef.current?.focus(); }
    if (e.key === 'ArrowDown') { /* focus next item */ }
    if (e.key === 'ArrowUp') { /* focus prev item */ }
  }
  
  return (
    <div className="nav-dropdown" onKeyDown={handleKeyDown}>
      <button
        ref={triggerRef}
        aria-haspopup="menu"
        aria-expanded={open}
        onClick={() => setOpen(!open)}
      >
        {trigger}
        <ChevronDownIcon aria-hidden />
      </button>
      
      {open && (
        <div
          ref={menuRef}
          role="menu"
          aria-orientation="vertical"
          className="dropdown-menu"
        >
          {items.map(item => (
            <a
              key={item.href}
              href={item.href}
              role="menuitem"
              className="dropdown-item"
            >
              {item.icon && <item.icon aria-hidden />}
              {item.label}
            </a>
          ))}
        </div>
      )}
    </div>
  );
}
```

---

## Information Architecture Principles

### Card Sorting (How to Organize Content)
Card sorting is a UX research method to discover how users group information. Even without formal research, apply these principles:

1. **User's mental model > Developer's data model**: Organize by what users call things, not by your database tables
2. **Frequency > Importance**: Most-used items should be most accessible, even if they're not most "important"
3. **Task-based grouping**: Group items by the tasks users do, not by feature category

### Navigation Item Count
```
Top nav: 4–7 items (cognitive load)
Sidebar: up to 20 items (grouping into sections of 4–6)
Tabs: 2–6 tabs (> 6 → consider sidebar or dropdown)
Breadcrumbs: no limit, but truncate middle levels on mobile
Bottom nav (mobile): 3–5 items
```

### Wayfinding Hierarchy

```
Level 0: Logo → always home
Level 1: Primary nav → main sections
Level 2: Section nav → sub-sections within main section
Level 3: Breadcrumb → specific location within sub-section
Level 4: In-page links → within a long page
```

---

## Navigation Anti-Patterns

| Anti-Pattern | Problem | Fix |
|-------------|---------|-----|
| No active state | User doesn't know where they are | Add `aria-current="page"` + visual indicator |
| Navigation only in JS | Not keyboard accessible | Use `<a href>` not `<div onClick>` |
| Deep nesting (4+ levels) | Users get lost | Flatten IA or use search |
| Too many top nav items (>8) | Overwhelm, Hick's Law | Group into dropdowns or remove |
| Inconsistent placement | Violates Jakob's Law | Same nav in same position on every page |
| No back/up affordance | Trapped in deep navigation | Add breadcrumbs or back button |
| Mobile menu without focus trap | Keyboard users escape menu | Trap focus inside mobile menu |
| Hamburger menu with no label | "Three lines" confuses some users | Add "Menu" text label |
| State lost on URL change | Filters reset on browser back | Store state in URL params |

---

## Navigation Checklist

- [ ] Current page/section always visually indicated
- [ ] `aria-current="page"` on active nav link
- [ ] Skip link present for keyboard users
- [ ] All nav items reachable by keyboard (Tab key)
- [ ] Mobile menu has focus trap + Escape closes
- [ ] Hamburger menu label describes action ("Open menu")
- [ ] Back button always works and goes to expected location
- [ ] Breadcrumbs present for 3+ level hierarchy
- [ ] Filter/tab state persisted in URL params
- [ ] Navigation consistent across all pages
- [ ] Mobile bottom nav has safe-area-inset-bottom padding
- [ ] Dropdown menus close on Escape and outside click
- [ ] No nav state that can't be bookmarked/shared
