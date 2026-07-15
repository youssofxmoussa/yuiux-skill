# 07 — Buttons & Controls

> A button is a promise. It promises that if clicked, something will happen. A button that breaks that promise — by doing nothing, by being unclear, by being unreachable — violates the user's trust.

---

## Button Hierarchy

Every view must have at most ONE primary button. Multiple primary buttons cancel each other out — they compete visually and leave the user uncertain which to click.

```
PRIMARY     — The main action. Highest visual weight. One per view.
SECONDARY   — Supporting action. Lower visual weight. Multiple allowed.
TERTIARY    — Minimal action. Least weight. Ghost/text style.
DESTRUCTIVE — Danger action. Red. Usually secondary visual weight.
LINK        — Navigation. Inline text style. Not a form action.
```

```tsx
// ✅ Correct hierarchy — one primary per section
<div className="dialog-footer">
  <Button variant="ghost">Cancel</Button>
  <Button variant="destructive">Delete workspace</Button>
</div>

// ✅ Primary + secondary
<div className="actions">
  <Button variant="outline">Save as draft</Button>
  <Button variant="default">Publish</Button>
</div>

// ❌ Two primary buttons — hierarchy lost
<div className="actions">
  <Button variant="default">Save</Button>
  <Button variant="default">Cancel</Button>
</div>
```

---

## The 6 Required States

Every button must implement all 6 states. A button without all states is incomplete.

| State | Visual Signal | When |
|-------|--------------|------|
| **Default** | Base appearance | Normal, resting state |
| **Hover** | Slightly darker bg, elevation | Mouse over |
| **Focus** | Visible ring (2px offset) | Keyboard focus |
| **Active** | Scale/press depression | Click/press held |
| **Disabled** | 50% opacity, not-allowed cursor | Action unavailable |
| **Loading** | Spinner + "ing" copy, wait cursor | Async operation in progress |

```css
/* Complete button state implementation */
.btn {
  /* Default */
  display: inline-flex;
  align-items: center;
  justify-content: center;
  gap: 8px;
  padding: 8px 16px;
  border-radius: var(--radius);
  font-size: 14px;
  font-weight: 500;
  line-height: 1;
  white-space: nowrap;
  cursor: pointer;
  border: 1px solid transparent;
  background: hsl(var(--primary));
  color: hsl(var(--primary-foreground));
  transition:
    background-color 150ms ease-out,
    box-shadow 150ms ease-out,
    transform 75ms ease-out,
    opacity 150ms ease-out;
  user-select: none;
  -webkit-tap-highlight-color: transparent;
}

/* Hover */
.btn:hover:not(:disabled):not([aria-busy="true"]) {
  background: hsl(var(--primary) / 0.9);
  box-shadow: 0 2px 8px hsl(var(--primary) / 0.3);
}

/* Focus */
.btn:focus-visible {
  outline: 2px solid hsl(var(--ring));
  outline-offset: 2px;
}

/* Active */
.btn:active:not(:disabled):not([aria-busy="true"]) {
  transform: scale(0.98) translateY(1px);
  box-shadow: none;
}

/* Disabled */
.btn:disabled,
.btn[aria-disabled="true"] {
  opacity: 0.5;
  cursor: not-allowed;
  pointer-events: none;
}

/* Loading */
.btn[aria-busy="true"] {
  cursor: wait;
  opacity: 0.8;
  pointer-events: none;
}
```

---

## Button Variants (Full System)

```css
/* PRIMARY — main CTA */
.btn-primary {
  background: hsl(var(--primary));
  color: hsl(var(--primary-foreground));
  border-color: transparent;
}
.btn-primary:hover { background: hsl(var(--primary) / 0.88); }

/* SECONDARY — supporting action */
.btn-secondary {
  background: hsl(var(--secondary));
  color: hsl(var(--secondary-foreground));
  border-color: transparent;
}
.btn-secondary:hover { background: hsl(var(--secondary) / 0.8); }

/* OUTLINE — tertiary action */
.btn-outline {
  background: transparent;
  color: hsl(var(--foreground));
  border-color: hsl(var(--border));
}
.btn-outline:hover {
  background: hsl(var(--accent));
  color: hsl(var(--accent-foreground));
}

/* GHOST — minimal action */
.btn-ghost {
  background: transparent;
  color: hsl(var(--foreground));
  border-color: transparent;
}
.btn-ghost:hover {
  background: hsl(var(--accent));
  color: hsl(var(--accent-foreground));
}

/* DESTRUCTIVE — danger action */
.btn-destructive {
  background: hsl(var(--destructive));
  color: hsl(var(--destructive-foreground));
  border-color: transparent;
}
.btn-destructive:hover { background: hsl(var(--destructive) / 0.88); }

/* LINK — navigation style */
.btn-link {
  background: transparent;
  color: hsl(var(--primary));
  border: none;
  text-decoration: underline;
  text-underline-offset: 3px;
  padding: 0;
  height: auto;
}
.btn-link:hover { text-decoration-color: hsl(var(--primary)); }
```

---

## Button Sizes

```css
/* SIZE SCALE */
.btn-sm {
  height: 32px;
  padding: 0 12px;
  font-size: 12px;
  border-radius: calc(var(--radius) - 2px);
}

.btn-md { /* default */
  height: 40px;
  padding: 0 16px;
  font-size: 14px;
}

.btn-lg {
  height: 48px;
  padding: 0 24px;
  font-size: 16px;
  border-radius: calc(var(--radius) + 2px);
}

.btn-xl {
  height: 56px;
  padding: 0 32px;
  font-size: 18px;
}

/* ICON BUTTON (square) */
.btn-icon {
  width: 40px;
  height: 40px;
  padding: 0;
}

.btn-icon-sm {
  width: 32px;
  height: 32px;
  padding: 0;
}

.btn-icon-lg {
  width: 48px;
  height: 48px;
  padding: 0;
}
```

---

## Icon Buttons

Icon-only buttons **must** have accessible labels:

```tsx
// ❌ No accessible name — screen reader says "button"
<button onClick={deleteItem}>
  <TrashIcon />
</button>

// ✅ aria-label
<button aria-label="Delete item" onClick={deleteItem}>
  <TrashIcon aria-hidden />
</button>

// ✅ Tooltip + aria-label
<TooltipProvider>
  <Tooltip>
    <TooltipTrigger asChild>
      <button aria-label="Delete item" onClick={deleteItem}>
        <TrashIcon aria-hidden />
      </button>
    </TooltipTrigger>
    <TooltipContent>Delete item</TooltipContent>
  </Tooltip>
</TooltipProvider>

// ✅ Visually hidden label
<button onClick={deleteItem}>
  <TrashIcon aria-hidden />
  <span className="sr-only">Delete item</span>
</button>
```

---

## Hit Targets

**Minimum 44×44px interactive area** for all touchable elements. Visual size can be smaller.

```css
/* Expand click area without changing visual size */

/* Method 1: Padding */
.small-icon-btn {
  width: 24px;
  height: 24px;
  padding: 10px; /* visual 24px, target 44px */
}

/* Method 2: Pseudo-element */
.small-icon-btn {
  position: relative;
  width: 24px;
  height: 24px;
}

.small-icon-btn::before {
  content: '';
  position: absolute;
  inset: -10px; /* expands by 10px on each side → 44px total */
}

/* Method 3: min-height/min-width */
.chip {
  min-height: 44px;
  padding-inline: 12px;
  display: inline-flex;
  align-items: center;
}
```

---

## Semantic HTML Rules

```tsx
// ❌ Non-semantic — no keyboard support, no accessibility
<div onClick={handleClick} className="btn">Click me</div>
<span onClick={handleClick}>Submit</span>
<a onClick={handleClick}>Save</a> {/* no href — not navigation */}

// ✅ Always use <button> for actions
<button type="button" onClick={handleClick}>Click me</button>
<button type="submit">Submit</button>

// ✅ Use <a> only for navigation (with href)
<a href="/settings">Settings</a>

// ✅ type="button" prevents accidental form submission
<form>
  <button type="button" onClick={openPreview}>Preview</button>
  <button type="submit">Publish</button>
</form>
```

---

## Loading State Button

```tsx
interface LoadingButtonProps extends React.ButtonHTMLAttributes<HTMLButtonElement> {
  loading?: boolean;
  loadingText?: string;
}

function LoadingButton({ 
  loading = false, 
  loadingText, 
  children, 
  disabled,
  ...props 
}: LoadingButtonProps) {
  return (
    <button
      {...props}
      disabled={loading || disabled}
      aria-busy={loading}
      aria-live="polite"
      aria-atomic
    >
      {loading ? (
        <>
          <svg
            className="spinner"
            xmlns="http://www.w3.org/2000/svg"
            fill="none"
            viewBox="0 0 24 24"
            aria-hidden="true"
            width="16"
            height="16"
          >
            <circle
              className="opacity-25"
              cx="12"
              cy="12"
              r="10"
              stroke="currentColor"
              strokeWidth="4"
            />
            <path
              className="opacity-75"
              fill="currentColor"
              d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4z"
            />
          </svg>
          <span>{loadingText ?? children}</span>
        </>
      ) : (
        children
      )}
    </button>
  );
}
```

---

## Toggle Buttons

```tsx
// Single toggle (bold, italic, etc.)
function ToggleButton({ pressed, onPressedChange, children, label }) {
  return (
    <button
      type="button"
      role="button"
      aria-pressed={pressed}
      aria-label={label}
      onClick={() => onPressedChange(!pressed)}
      data-state={pressed ? 'on' : 'off'}
      className="toggle-btn"
    >
      {children}
    </button>
  );
}

// Toggle group (exclusive selection)
function ToggleGroup({ value, onChange, options }) {
  return (
    <div role="group" aria-label="View options">
      {options.map(option => (
        <button
          key={option.value}
          type="button"
          role="radio"
          aria-checked={value === option.value}
          onClick={() => onChange(option.value)}
          className={`toggle-group-item ${value === option.value ? 'selected' : ''}`}
        >
          {option.label}
        </button>
      ))}
    </div>
  );
}
```

---

## Switch / Toggle

```tsx
// Accessible switch (not checkbox — switch has different semantics)
function Switch({ checked, onChange, label, id }) {
  return (
    <div className="switch-wrapper">
      <label htmlFor={id}>{label}</label>
      <button
        id={id}
        type="button"
        role="switch"
        aria-checked={checked}
        onClick={() => onChange(!checked)}
        className={`switch ${checked ? 'switch--on' : 'switch--off'}`}
      >
        <span className="switch-thumb" />
        <span className="sr-only">{checked ? 'On' : 'Off'}</span>
      </button>
    </div>
  );
}
```

```css
.switch {
  position: relative;
  width: 44px;
  height: 24px;
  border-radius: 12px;
  border: none;
  cursor: pointer;
  transition: background 200ms ease;
  background: hsl(var(--muted));
}

.switch[aria-checked="true"] {
  background: hsl(var(--primary));
}

.switch-thumb {
  position: absolute;
  top: 2px;
  left: 2px;
  width: 20px;
  height: 20px;
  border-radius: 50%;
  background: white;
  transition: transform 200ms cubic-bezier(0.34, 1.56, 0.64, 1);
  box-shadow: 0 1px 3px rgba(0,0,0,0.2);
}

.switch[aria-checked="true"] .switch-thumb {
  transform: translateX(20px);
}
```

---

## Checkbox

```tsx
// Custom checkbox with proper accessibility
function Checkbox({ id, label, checked, onChange, indeterminate, description }) {
  const ref = useRef<HTMLInputElement>(null);
  
  useEffect(() => {
    if (ref.current) {
      ref.current.indeterminate = indeterminate ?? false;
    }
  }, [indeterminate]);
  
  return (
    <div className="checkbox-wrapper">
      <div className="checkbox-control">
        <input
          ref={ref}
          type="checkbox"
          id={id}
          checked={checked}
          onChange={e => onChange(e.target.checked)}
          aria-describedby={description ? `${id}-desc` : undefined}
          className="checkbox-input sr-only"
        />
        <label htmlFor={id} className="checkbox-label">
          <span className="checkbox-box" aria-hidden>
            {checked && <CheckIcon />}
            {indeterminate && <MinusIcon />}
          </span>
          {label}
        </label>
      </div>
      {description && (
        <p id={`${id}-desc`} className="checkbox-description">
          {description}
        </p>
      )}
    </div>
  );
}
```

---

## Radio Group

```tsx
function RadioGroup({ name, label, options, value, onChange }) {
  return (
    <fieldset>
      <legend>{label}</legend>
      <div className="radio-group">
        {options.map(option => (
          <label key={option.value} className="radio-label">
            <input
              type="radio"
              name={name}
              value={option.value}
              checked={value === option.value}
              onChange={() => onChange(option.value)}
              aria-describedby={option.description ? `${name}-${option.value}-desc` : undefined}
            />
            <span>{option.label}</span>
            {option.description && (
              <span id={`${name}-${option.value}-desc`} className="radio-description">
                {option.description}
              </span>
            )}
          </label>
        ))}
      </div>
    </fieldset>
  );
}
```

---

## Slider / Range

```tsx
function Slider({ id, label, min, max, step = 1, value, onChange, formatValue }) {
  return (
    <div className="slider-field">
      <div className="slider-header">
        <label htmlFor={id}>{label}</label>
        <output htmlFor={id} aria-live="polite">
          {formatValue ? formatValue(value) : value}
        </output>
      </div>
      <input
        id={id}
        type="range"
        min={min}
        max={max}
        step={step}
        value={value}
        onChange={e => onChange(Number(e.target.value))}
        aria-valuemin={min}
        aria-valuemax={max}
        aria-valuenow={value}
        aria-valuetext={formatValue ? formatValue(value) : String(value)}
        className="slider"
      />
      <div className="slider-labels" aria-hidden>
        <span>{formatValue ? formatValue(min) : min}</span>
        <span>{formatValue ? formatValue(max) : max}</span>
      </div>
    </div>
  );
}
```

```css
.slider {
  -webkit-appearance: none;
  width: 100%;
  height: 6px;
  border-radius: 3px;
  background: linear-gradient(
    to right,
    hsl(var(--primary)) 0%,
    hsl(var(--primary)) var(--pct, 50%),
    hsl(var(--muted)) var(--pct, 50%),
    hsl(var(--muted)) 100%
  );
  outline: none;
  cursor: pointer;
}

.slider::-webkit-slider-thumb {
  -webkit-appearance: none;
  width: 20px;
  height: 20px;
  border-radius: 50%;
  background: white;
  border: 2px solid hsl(var(--primary));
  cursor: pointer;
  transition: box-shadow 150ms;
}

.slider:focus-visible::-webkit-slider-thumb {
  box-shadow: 0 0 0 4px hsl(var(--ring) / 0.3);
}
```

---

## Disabled vs. Unavailable

```tsx
// ❌ Disabled button with no explanation
<Button disabled>Publish</Button>

// ✅ Option 1: Explain why disabled (tooltip)
<TooltipProvider>
  <Tooltip>
    <TooltipTrigger asChild>
      <span> {/* wrapper needed — disabled buttons don't fire mouse events */}
        <Button disabled aria-describedby="publish-reason">Publish</Button>
      </span>
    </TooltipTrigger>
    <TooltipContent id="publish-reason">
      Add at least one image before publishing
    </TooltipContent>
  </Tooltip>
</TooltipProvider>

// ✅ Option 2: Inline explanation
<div>
  <Button disabled>Publish</Button>
  <p className="text-sm text-muted-foreground">
    Add at least one image to publish
  </p>
</div>

// ✅ Option 3: Allow click, show explanation in response
<Button onClick={handlePublishAttempt}>Publish</Button>
// On click if not ready: show inline warning
```

---

## Button Copy Rules

```
Action buttons: Verb + Noun
  ❌ "OK"              ✅ "Save changes"
  ❌ "Submit"          ✅ "Create account"
  ❌ "Yes"             ✅ "Delete project"
  ❌ "Confirm"         ✅ "Send $500"
  ❌ "Continue"        ✅ "Next: shipping"

Loading state: verb becomes "-ing"
  "Save changes"  → "Saving changes…"
  "Delete"        → "Deleting…"
  "Send email"    → "Sending email…"

Destructive confirm: name what's destroyed
  ❌ "Delete"          ✅ "Delete 'Summer 2026' album and 47 photos"
  ❌ "Yes, delete it"  ✅ "Yes, cancel my Pro subscription"
```

---

## Button Checklist

- [ ] One primary button per view/dialog
- [ ] All 6 states implemented: default/hover/focus/active/disabled/loading
- [ ] `type="button"` on non-submit buttons inside forms
- [ ] `type="submit"` on submit buttons
- [ ] Icon-only buttons have `aria-label`
- [ ] Hit targets ≥ 44×44px on touch
- [ ] Disabled buttons have explanation (tooltip or inline)
- [ ] Loading buttons show spinner + verbal feedback
- [ ] Destructive buttons use destructive variant + clear copy
- [ ] No `<div>` or `<span>` as buttons
- [ ] Toggle buttons use `aria-pressed`
- [ ] Switch uses `role="switch"` and `aria-checked`
- [ ] Radio groups use `<fieldset>` + `<legend>`
- [ ] Checkboxes use `<input type="checkbox">` (not custom div)
- [ ] Slider has `aria-valuetext` for screen reader description
