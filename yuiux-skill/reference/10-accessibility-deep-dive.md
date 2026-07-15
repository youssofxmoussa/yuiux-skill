# 10 — Accessibility Deep Dive

> Accessibility is not a feature. It's a quality of the entire product. Every element either includes or excludes a segment of users.

---

## WCAG 2.2 — Critical Criteria

WCAG success criteria are organized by level: A (must), AA (should), AAA (aspirational).
**Target: AA compliance on all public-facing interfaces.**

### Level A — Must Pass

| Criterion | Rule | Common Violation |
|-----------|------|-----------------|
| 1.1.1 | Images need text alternatives | `<img>` without `alt` attribute |
| 1.3.1 | Structure conveyed through markup | Using visual formatting instead of heading tags |
| 1.3.3 | Instructions don't rely on shape/color/position alone | "Click the red button" |
| 1.4.1 | Color not sole means of conveying info | Error indicated only by red border |
| 2.1.1 | All functionality available by keyboard | Click-only interactions |
| 2.1.2 | No keyboard traps | Focus stuck in custom widget |
| 2.4.1 | Skip navigation link provided | No way to bypass repeated navigation |
| 2.4.3 | Focus order preserves meaning | Tab order jumps around illogically |
| 3.1.1 | Page language specified | No `<html lang="en">` |
| 3.3.1 | Errors identified in text | Error shown by color/icon only |
| 3.3.2 | Labels or instructions for inputs | No `<label>` associated with inputs |
| 4.1.2 | Name, role, value for UI components | Custom widget without ARIA |

### Level AA — Should Pass

| Criterion | Rule | Common Violation |
|-----------|------|-----------------|
| 1.4.3 | Text contrast ≥ 4.5:1 (large: ≥ 3:1) | Light gray on white |
| 1.4.4 | Text resizes to 200% without loss | Fixed-px layouts breaking at 200% |
| 1.4.10 | Content visible at 320px without horizontal scroll | Non-responsive layouts |
| 1.4.11 | UI components: ≥ 3:1 contrast | Low-contrast input borders |
| 1.4.13 | Hover/focus content: dismissible, persistent, hoverable | Tooltips that disappear on hover |
| 2.4.7 | Focus indicator visible | `outline: none` with no replacement |
| 2.4.11 | Focus not obscured by sticky elements (2.2 new) | Sticky header hides focused element |
| 2.5.3 | Accessible name includes visible label text | Button label mismatch with aria-label |
| 2.5.8 | Target size minimum 24×24px, 44×44 recommended (2.2) | Tiny clickable icons |
| 3.3.3 | Error suggestions where possible | "Invalid" with no guidance |
| 3.3.4 | Legal/financial: checkable, reversible, or confirmable | One-click irreversible financial action |
| 4.1.3 | Status messages programmatically determinable | Success toast not announced to screen reader |

---

## Keyboard Accessibility

### Full Keyboard Navigation Test

Run this test on every page before shipping:

1. Put cursor outside browser window
2. Press Tab — verify focus appears somewhere sensible
3. Tab through entire page — verify every interactive element is reachable
4. Verify visual focus indicator is visible at every step
5. Activate each element: buttons with Enter/Space, links with Enter
6. Navigate through menus/dropdowns with arrow keys
7. Press Escape to close any open overlay
8. Complete a full user task using ONLY keyboard

### Focus Ring

```css
/* Universal focus visible rule */
*:focus {
  outline: none; /* remove default (will add :focus-visible) */
}

*:focus-visible {
  outline: 2px solid hsl(var(--ring));
  outline-offset: 2px;
  border-radius: var(--radius, 4px);
}

/* High contrast mode */
@media (forced-colors: active) {
  *:focus-visible {
    outline: 3px solid ButtonText;
    outline-offset: 2px;
  }
}

/* For dark backgrounds — invert the ring */
.dark-bg *:focus-visible {
  outline-color: white;
}
```

### Tab Order

```html
<!-- Natural tab order = DOM order. Never use tabindex > 0 -->

<!-- ❌ Broken tab order -->
<div tabindex="3">Third</div>
<div tabindex="1">First</div>
<div tabindex="2">Second</div>

<!-- ✅ Natural tab order from DOM order -->
<div>First</div>
<div>Second</div>
<div>Third</div>

<!-- ✅ tabindex="0" — adds element to natural tab order -->
<div tabindex="0" role="button" onClick={...}>Custom widget</div>

<!-- ✅ tabindex="-1" — focusable by JS, not by Tab -->
<div tabindex="-1" ref={errorRef}>Error summary</div>
<!-- errorRef.current.focus() called programmatically -->
```

### Keyboard Shortcuts for Custom Widgets

```tsx
// Arrow key navigation in a custom listbox
function ListBox({ options, value, onChange }) {
  const [focusedIndex, setFocusedIndex] = useState(0);
  const listRef = useRef<HTMLUListElement>(null);
  
  function handleKeyDown(e: React.KeyboardEvent) {
    switch(e.key) {
      case 'ArrowDown':
        e.preventDefault();
        setFocusedIndex(i => Math.min(i + 1, options.length - 1));
        break;
      case 'ArrowUp':
        e.preventDefault();
        setFocusedIndex(i => Math.max(i - 1, 0));
        break;
      case 'Home':
        e.preventDefault();
        setFocusedIndex(0);
        break;
      case 'End':
        e.preventDefault();
        setFocusedIndex(options.length - 1);
        break;
      case 'Enter':
      case ' ':
        e.preventDefault();
        onChange(options[focusedIndex].value);
        break;
    }
  }
  
  useEffect(() => {
    const items = listRef.current?.querySelectorAll('[role="option"]');
    (items?.[focusedIndex] as HTMLElement)?.focus();
  }, [focusedIndex]);
  
  return (
    <ul
      ref={listRef}
      role="listbox"
      aria-activedescendant={`option-${options[focusedIndex]?.value}`}
      tabIndex={0}
      onKeyDown={handleKeyDown}
    >
      {options.map((option, index) => (
        <li
          key={option.value}
          id={`option-${option.value}`}
          role="option"
          aria-selected={value === option.value}
          tabIndex={focusedIndex === index ? 0 : -1}
          onClick={() => { setFocusedIndex(index); onChange(option.value); }}
        >
          {option.label}
        </li>
      ))}
    </ul>
  );
}
```

---

## ARIA Patterns

### The ARIA Rules (Follow In Order)

```
Rule 1: Use native HTML before ARIA
  <button> over <div role="button">
  <nav> over <div role="navigation">
  <main> over <div role="main">

Rule 2: Don't change native semantics
  <h2 role="tab"> — wrong (changes meaning of heading)
  <button role="link"> — wrong (changes button to link)

Rule 3: All interactive ARIA controls must be keyboard accessible

Rule 4: Don't use role="presentation" or aria-hidden="true" on focusable elements

Rule 5: Interactive elements need accessible names
  <button aria-label="Close dialog"> or <button>Close dialog</button>
```

### Common ARIA Patterns

```html
<!-- Landmark roles — define page structure -->
<header role="banner">...</header>
<nav role="navigation" aria-label="Main navigation">...</nav>
<main role="main">...</main>
<aside role="complementary" aria-label="Related links">...</aside>
<footer role="contentinfo">...</footer>

<!-- Form error — must use role="alert" for immediate announcement -->
<p id="field-error" role="alert">Email address is required</p>
<input aria-describedby="field-error" aria-invalid="true" />

<!-- Status message — polite announcement -->
<p role="status" aria-live="polite">Draft saved</p>

<!-- Important alert — immediate interruption -->
<p role="alert" aria-live="assertive">Your session will expire in 5 minutes</p>

<!-- Progressbar -->
<div 
  role="progressbar" 
  aria-valuemin="0" 
  aria-valuemax="100" 
  aria-valuenow="60"
  aria-label="Upload progress"
>
  60%
</div>

<!-- Expanded/collapsed disclosure -->
<button aria-expanded="false" aria-controls="panel-id">
  Advanced options
</button>
<div id="panel-id" hidden>
  <!-- panel content -->
</div>

<!-- Dialog -->
<div 
  role="dialog" 
  aria-modal="true" 
  aria-labelledby="dialog-title"
  aria-describedby="dialog-desc"
>
  <h2 id="dialog-title">Confirm deletion</h2>
  <p id="dialog-desc">This will permanently delete all 14 files.</p>
  <!-- buttons -->
</div>
```

### aria-label vs aria-labelledby vs aria-describedby

```html
<!-- aria-label: label is a string, no visible label in DOM -->
<button aria-label="Close dialog">
  <XIcon />
</button>

<!-- aria-labelledby: points to existing text in DOM as the label -->
<h2 id="section-title">Account Settings</h2>
<section aria-labelledby="section-title">
  <!-- the h2 IS the accessible name -->
</section>

<!-- aria-describedby: supplementary description (not the name) -->
<input 
  id="password"
  type="password"
  aria-describedby="password-requirements" 
/>
<p id="password-requirements">
  Must be at least 8 characters with one uppercase letter
</p>
```

### When NOT to Use ARIA

```html
<!-- ❌ Redundant ARIA — native HTML already conveys this -->
<button role="button">Click me</button>
<a href="/settings" role="link">Settings</a>
<h2 role="heading" aria-level="2">Title</h2>
<ul role="list"><li role="listitem">Item</li></ul>

<!-- ✅ Just use the native element -->
<button>Click me</button>
<a href="/settings">Settings</a>
<h2>Title</h2>
<ul><li>Item</li></ul>
```

---

## Focus Management

### Modal Focus Trap

```tsx
function Modal({ isOpen, onClose, children, title }) {
  const modalRef = useRef<HTMLDivElement>(null);
  const previousFocusRef = useRef<HTMLElement | null>(null);
  
  useEffect(() => {
    if (!isOpen) return;
    
    // Store the element that opened the modal
    previousFocusRef.current = document.activeElement as HTMLElement;
    
    // Focus the modal (or first focusable element within)
    const firstFocusable = modalRef.current?.querySelector<HTMLElement>(
      'button, [href], input, select, textarea, [tabindex]:not([tabindex="-1"])'
    );
    firstFocusable?.focus() ?? modalRef.current?.focus();
    
    // Trap focus
    function trapFocus(e: KeyboardEvent) {
      if (e.key !== 'Tab' || !modalRef.current) return;
      
      const focusable = [...modalRef.current.querySelectorAll<HTMLElement>(
        'button:not(:disabled), [href], input:not(:disabled), select:not(:disabled), textarea:not(:disabled), [tabindex]:not([tabindex="-1"])'
      )].filter(el => !el.hidden && el.offsetParent !== null);
      
      const first = focusable[0];
      const last = focusable[focusable.length - 1];
      
      if (e.shiftKey) {
        if (document.activeElement === first) {
          e.preventDefault();
          last.focus();
        }
      } else {
        if (document.activeElement === last) {
          e.preventDefault();
          first.focus();
        }
      }
    }
    
    // Close on Escape
    function handleEscape(e: KeyboardEvent) {
      if (e.key === 'Escape') onClose();
    }
    
    document.addEventListener('keydown', trapFocus);
    document.addEventListener('keydown', handleEscape);
    
    return () => {
      document.removeEventListener('keydown', trapFocus);
      document.removeEventListener('keydown', handleEscape);
      // Restore focus to trigger element
      previousFocusRef.current?.focus();
    };
  }, [isOpen, onClose]);
  
  if (!isOpen) return null;
  
  return (
    <div
      ref={modalRef}
      role="dialog"
      aria-modal="true"
      aria-labelledby="modal-title"
      tabIndex={-1}
      className="modal"
    >
      <h2 id="modal-title">{title}</h2>
      {children}
      <button onClick={onClose} aria-label="Close dialog">
        <XIcon aria-hidden />
      </button>
    </div>
  );
}
```

### Focus After Mutation

```tsx
// When content changes, focus the relevant element

// After adding a new item — focus it
async function addItem() {
  const newItem = await api.createItem(data);
  setItems(prev => [...prev, newItem]);
  
  // Focus the new item after render
  requestAnimationFrame(() => {
    document.getElementById(`item-${newItem.id}`)?.focus();
  });
}

// After deleting an item — focus next or previous
function deleteItem(index: number) {
  setItems(prev => prev.filter((_, i) => i !== index));
  
  requestAnimationFrame(() => {
    const items = document.querySelectorAll('.list-item');
    const focusTarget = items[Math.min(index, items.length - 1)] as HTMLElement;
    focusTarget?.focus();
  });
}

// After form error — focus error summary
function handleFailedSubmit() {
  setErrors(newErrors);
  requestAnimationFrame(() => {
    document.getElementById('form-error-summary')?.focus();
  });
}
```

---

## Screen Reader Support

### Alt Text Guidelines

```html
<!-- Informative image: describe what it shows -->
<img src="chart.png" alt="Bar chart showing 40% increase in sign-ups from Q1 to Q2 2026">

<!-- Functional image (button/link): describe the action -->
<a href="/home"><img src="logo.svg" alt="Acme Corp — home"></a>
<button><img src="search.svg" alt="Search"></button>

<!-- Decorative image: empty alt (screen reader skips it) -->
<img src="hero-bg.jpg" alt="">

<!-- Complex image: alt + longer description -->
<img src="complex-chart.png" alt="Sales chart" aria-describedby="chart-desc">
<p id="chart-desc">Full text description of the chart data...</p>

<!-- CSS background images: use aria-label on container -->
<div 
  style="background-image: url(...)" 
  role="img" 
  aria-label="Description of the image"
/>
```

### Screen Reader Only (Visually Hidden) Text

```css
.sr-only {
  position: absolute;
  width: 1px;
  height: 1px;
  padding: 0;
  margin: -1px;
  overflow: hidden;
  clip: rect(0, 0, 0, 0);
  white-space: nowrap;
  border-width: 0;
}

/* sr-only that becomes visible on focus (for skip links) */
.sr-only-focusable:not(:focus):not(:focus-within) {
  position: absolute;
  width: 1px;
  height: 1px;
  padding: 0;
  margin: -1px;
  overflow: hidden;
  clip: rect(0, 0, 0, 0);
  white-space: nowrap;
  border-width: 0;
}
```

```tsx
// Supplementary screen reader text for context
function UserBadge({ name, role }: { name: string; role: string }) {
  return (
    <div className="user-badge">
      <img src={avatarUrl(name)} alt="" /> {/* decorative — name in text */}
      <span>{name}</span>
      <span className="badge">{role}</span>
      <span className="sr-only">, {role}</span> {/* additional context for SR */}
    </div>
  );
}

// Icon with screen reader label
function StarRating({ rating, max = 5 }: { rating: number; max: number }) {
  return (
    <div aria-label={`${rating} out of ${max} stars`}>
      {Array.from({ length: max }, (_, i) => (
        <StarIcon
          key={i}
          aria-hidden
          className={i < rating ? 'star-filled' : 'star-empty'}
        />
      ))}
    </div>
  );
}
```

### Live Regions

```tsx
// Announce dynamic changes without moving focus
function AnnouncementLive() {
  return (
    <>
      {/* Polite — waits for user to finish current task */}
      <div aria-live="polite" aria-atomic className="sr-only" id="polite-live">
        {/* Set text to announce changes: saved drafts, cart updates, etc. */}
      </div>
      
      {/* Assertive — interrupts immediately (use sparingly) */}
      <div aria-live="assertive" aria-atomic className="sr-only" id="assertive-live">
        {/* Set text for urgent: session expiry, error conditions */}
      </div>
    </>
  );
}

// Hook to use live region
function useLiveAnnouncement() {
  function announce(message: string, priority: 'polite' | 'assertive' = 'polite') {
    const el = document.getElementById(`${priority}-live`);
    if (!el) return;
    
    // Must clear and reset to trigger re-announcement of same text
    el.textContent = '';
    requestAnimationFrame(() => {
      el.textContent = message;
    });
  }
  
  return announce;
}
```

---

## Color & Contrast for Accessibility

```tsx
// Programmatic contrast check
function getContrastRatio(color1: string, color2: string): number {
  const l1 = getRelativeLuminance(color1);
  const l2 = getRelativeLuminance(color2);
  const lighter = Math.max(l1, l2);
  const darker = Math.min(l1, l2);
  return (lighter + 0.05) / (darker + 0.05);
}

function getRelativeLuminance(hex: string): number {
  const [r, g, b] = hexToRgb(hex).map(v => {
    v /= 255;
    return v <= 0.03928 ? v / 12.92 : Math.pow((v + 0.055) / 1.055, 2.4);
  });
  return 0.2126 * r + 0.7152 * g + 0.0722 * b;
}

// Usage in development
if (process.env.NODE_ENV === 'development') {
  const ratio = getContrastRatio('#6b7280', '#ffffff');
  if (ratio < 4.5) {
    console.warn(`Contrast ratio ${ratio.toFixed(2)}:1 fails WCAG AA for normal text`);
  }
}
```

---

## Automated Accessibility Testing

### CI Integration with axe-core

```javascript
// vitest / jest test
import { axe, toHaveNoViolations } from 'jest-axe';
import { render } from '@testing-library/react';

expect.extend(toHaveNoViolations);

describe('Accessibility', () => {
  it('Form has no violations', async () => {
    const { container } = render(<LoginForm />);
    const results = await axe(container);
    expect(results).toHaveNoViolations();
  });
  
  it('Dashboard has no violations', async () => {
    const { container } = render(<Dashboard />);
    const results = await axe(container, {
      rules: {
        // Only fail on critical/serious violations in CI
        'color-contrast': { enabled: true },
        'label': { enabled: true },
        'button-name': { enabled: true },
      }
    });
    expect(results).toHaveNoViolations();
  });
});
```

### Playwright E2E Accessibility

```javascript
// In playwright test
import { checkA11y, injectAxe } from 'axe-playwright';

test('Login page accessibility', async ({ page }) => {
  await page.goto('/login');
  await injectAxe(page);
  await checkA11y(page, null, {
    detailedReport: true,
    detailedReportOptions: { html: true },
    axeOptions: { runOnly: ['wcag2a', 'wcag2aa', 'wcag21aa'] }
  });
});
```

---

## Accessibility Checklist

### Foundation
- [ ] `<html lang="en">` (or appropriate language code) on every page
- [ ] One `<h1>` per page, heading hierarchy never skips levels
- [ ] Skip link present, visible on focus, links to `#main-content`
- [ ] Page has logical landmarks: `<header>`, `<nav>`, `<main>`, `<footer>`
- [ ] All images have `alt` attribute (empty for decorative)

### Keyboard
- [ ] All interactive elements reachable with Tab key
- [ ] Tab order follows visual reading order
- [ ] No element has `tabindex` > 0
- [ ] Custom widgets implement arrow key navigation
- [ ] Modal/drawer traps focus when open
- [ ] Modal/drawer closes on Escape and returns focus to trigger
- [ ] Skip link allows bypassing navigation

### Focus
- [ ] Focus ring visible on every focusable element
- [ ] `outline: none` never used without replacement
- [ ] Focus not obscured by sticky headers (WCAG 2.4.11)

### Screen Reader
- [ ] Interactive icon-only elements have `aria-label`
- [ ] Dynamic regions have `aria-live`
- [ ] Form errors use `role="alert"` and `aria-describedby`
- [ ] Loading states have `role="status"` and accessible label
- [ ] Status messages announced via live regions
- [ ] Tables have proper `<caption>`, `scope` on `<th>` elements
- [ ] Custom controls have correct ARIA `role`, `aria-selected`/`aria-expanded`/etc.

### Color & Contrast
- [ ] Body text: ≥ 4.5:1 contrast
- [ ] Large text (≥18px normal / ≥14px bold): ≥ 3:1 contrast
- [ ] UI components (buttons, inputs): ≥ 3:1 contrast
- [ ] Color not sole means of conveying information (always + icon or text)
- [ ] Focus ring meets ≥ 3:1 against adjacent colors
