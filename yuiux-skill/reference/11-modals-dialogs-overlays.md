# 11 — Modals, Dialogs & Overlays

> A modal is a contract: "I need your complete attention before we continue." Use them sparingly. Every modal you add is a demand on the user's focus.

---

## When to Use a Modal

### ✅ Appropriate
- **Irreversible or high-stakes confirmations**: "Delete workspace permanently?"
- **Short forms that interrupt flow**: "Create new project" without leaving current page
- **Quick data entry**: "Add team member" in one step
- **Media lightboxes**: Expanded image viewing
- **Alerts requiring acknowledgment**: Security warning, legal notice

### ❌ Inappropriate
- **Complex multi-step workflows** → Use a full page instead
- **Content that should be linkable** → Use a page or drawer
- **Supplementary info** → Use a tooltip, popover, or aside
- **Error messages** → Use inline error + form validation
- **Progress indication** → Use inline feedback with skeleton
- **On mobile — anything opening a modal from a modal** → Never nest modals

---

## Modal Anatomy

```
┌─────────────────────────────────────────┐ ← backdrop (accessible via click/Escape)
│                                         │
│  ┌─────────────────────────────────┐    │
│  │ Title                      [×] │ ← header
│  │─────────────────────────────────│
│  │                                 │
│  │  Content area                   │ ← body (scrollable if needed)
│  │  Description text               │
│  │                                 │
│  │─────────────────────────────────│
│  │          [Cancel] [Confirm]     │ ← footer (right-aligned actions)
│  └─────────────────────────────────┘
│                                         │
└─────────────────────────────────────────┘
```

---

## Complete Modal Implementation

```tsx
import { useEffect, useRef, useState, useCallback } from 'react';
import { createPortal } from 'react-dom';

interface DialogProps {
  open: boolean;
  onClose: () => void;
  title: string;
  description?: string;
  children: React.ReactNode;
  size?: 'sm' | 'md' | 'lg' | 'xl';
  closeOnBackdrop?: boolean;
}

function Dialog({ 
  open, 
  onClose, 
  title, 
  description,
  children, 
  size = 'md',
  closeOnBackdrop = true,
}: DialogProps) {
  const dialogRef = useRef<HTMLDivElement>(null);
  const previousFocusRef = useRef<HTMLElement | null>(null);
  
  // Store and restore focus
  useEffect(() => {
    if (open) {
      previousFocusRef.current = document.activeElement as HTMLElement;
      // Focus dialog or first focusable element
      requestAnimationFrame(() => {
        const focusable = dialogRef.current?.querySelector<HTMLElement>(
          'button, [href], input, select, textarea, [tabindex]:not([tabindex="-1"])'
        );
        (focusable ?? dialogRef.current)?.focus();
      });
    } else {
      previousFocusRef.current?.focus();
    }
  }, [open]);
  
  // Focus trap
  const handleKeyDown = useCallback((e: React.KeyboardEvent) => {
    if (e.key === 'Escape') { onClose(); return; }
    if (e.key !== 'Tab') return;
    
    const focusable = [...(dialogRef.current?.querySelectorAll<HTMLElement>(
      'button:not(:disabled), [href], input:not(:disabled), select:not(:disabled), textarea:not(:disabled), [tabindex]:not([tabindex="-1"])'
    ) ?? [])].filter(el => el.offsetParent !== null);
    
    if (!focusable.length) return;
    const first = focusable[0];
    const last = focusable[focusable.length - 1];
    
    if (e.shiftKey && document.activeElement === first) {
      e.preventDefault(); last.focus();
    } else if (!e.shiftKey && document.activeElement === last) {
      e.preventDefault(); first.focus();
    }
  }, [onClose]);
  
  // Lock body scroll
  useEffect(() => {
    if (open) {
      const scrollY = window.scrollY;
      document.body.style.cssText = `overflow: hidden; position: fixed; top: -${scrollY}px; width: 100%;`;
      return () => {
        document.body.style.cssText = '';
        window.scrollTo(0, scrollY);
      };
    }
  }, [open]);
  
  if (!open) return null;
  
  return createPortal(
    <>
      {/* Backdrop */}
      <div
        className="dialog-backdrop"
        aria-hidden="true"
        onClick={closeOnBackdrop ? onClose : undefined}
      />
      
      {/* Dialog */}
      <div
        ref={dialogRef}
        role="dialog"
        aria-modal="true"
        aria-labelledby="dialog-title"
        aria-describedby={description ? 'dialog-description' : undefined}
        className={`dialog dialog--${size}`}
        tabIndex={-1}
        onKeyDown={handleKeyDown}
      >
        {/* Header */}
        <div className="dialog-header">
          <h2 id="dialog-title" className="dialog-title">{title}</h2>
          <button
            type="button"
            onClick={onClose}
            aria-label="Close dialog"
            className="dialog-close"
          >
            <XIcon aria-hidden />
          </button>
        </div>
        
        {/* Description */}
        {description && (
          <p id="dialog-description" className="dialog-description">{description}</p>
        )}
        
        {/* Body */}
        <div className="dialog-body">
          {children}
        </div>
      </div>
    </>,
    document.body
  );
}
```

```css
/* Backdrop */
.dialog-backdrop {
  position: fixed;
  inset: 0;
  background: rgba(0, 0, 0, 0.5);
  backdrop-filter: blur(2px);
  z-index: var(--z-modal-bg);
  animation: fadeIn 200ms ease;
}

/* Dialog container */
.dialog {
  position: fixed;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  z-index: var(--z-modal);
  background: hsl(var(--background));
  border: 1px solid hsl(var(--border));
  border-radius: 12px;
  box-shadow: var(--shadow-xl);
  display: flex;
  flex-direction: column;
  max-height: calc(100vh - 48px);
  width: calc(100vw - 48px);
  animation: scaleIn 200ms ease;
  outline: none;
}

/* Size variants */
.dialog--sm { max-width: 400px; }
.dialog--md { max-width: 560px; }
.dialog--lg { max-width: 720px; }
.dialog--xl { max-width: 960px; }

/* Header */
.dialog-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 20px 24px 0;
  flex-shrink: 0;
}

.dialog-title {
  font-size: 18px;
  font-weight: 600;
  color: hsl(var(--foreground));
  margin: 0;
}

.dialog-close {
  width: 32px;
  height: 32px;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 6px;
  border: none;
  background: transparent;
  color: hsl(var(--muted-foreground));
  cursor: pointer;
  transition: background 150ms, color 150ms;
}
.dialog-close:hover { background: hsl(var(--accent)); color: hsl(var(--foreground)); }

/* Description */
.dialog-description {
  padding: 8px 24px 0;
  font-size: 14px;
  color: hsl(var(--muted-foreground));
  margin: 0;
}

/* Body */
.dialog-body {
  padding: 20px 24px 24px;
  overflow-y: auto;
  flex: 1;
}

/* Animations */
@keyframes fadeIn {
  from { opacity: 0; }
  to   { opacity: 1; }
}

@keyframes scaleIn {
  from { opacity: 0; transform: translate(-50%, -50%) scale(0.95); }
  to   { opacity: 1; transform: translate(-50%, -50%) scale(1); }
}

@media (prefers-reduced-motion: reduce) {
  .dialog-backdrop, .dialog { animation: none; }
}

/* Mobile: full-screen sheet */
@media (max-width: 640px) {
  .dialog {
    top: auto;
    bottom: 0;
    left: 0;
    right: 0;
    transform: none;
    width: 100%;
    max-width: 100%;
    max-height: 90vh;
    border-radius: 16px 16px 0 0;
    animation: slideUpMobile 300ms ease;
  }
  
  @keyframes slideUpMobile {
    from { transform: translateY(100%); }
    to   { transform: translateY(0); }
  }
}
```

---

## Confirmation Dialog

```tsx
interface ConfirmDialogProps {
  open: boolean;
  onConfirm: () => Promise<void> | void;
  onCancel: () => void;
  title: string;
  description: string;
  confirmLabel?: string;
  cancelLabel?: string;
  variant?: 'default' | 'destructive';
}

function ConfirmDialog({
  open, onConfirm, onCancel,
  title, description,
  confirmLabel = 'Confirm',
  cancelLabel = 'Cancel',
  variant = 'default',
}: ConfirmDialogProps) {
  const [loading, setLoading] = useState(false);
  
  async function handleConfirm() {
    setLoading(true);
    try {
      await onConfirm();
    } finally {
      setLoading(false);
    }
  }
  
  return (
    <Dialog open={open} onClose={onCancel} title={title} description={description} size="sm">
      <div className="dialog-footer">
        <button type="button" onClick={onCancel} disabled={loading} className="btn btn-outline">
          {cancelLabel}
        </button>
        <LoadingButton
          onClick={handleConfirm}
          loading={loading}
          loadingText={`${confirmLabel}ing…`}
          className={`btn btn-${variant}`}
        >
          {confirmLabel}
        </LoadingButton>
      </div>
    </Dialog>
  );
}

// Usage:
<ConfirmDialog
  open={showDelete}
  onCancel={() => setShowDelete(false)}
  onConfirm={async () => {
    await deleteWorkspace(workspace.id);
    setShowDelete(false);
    navigate('/workspaces');
  }}
  title={`Delete "${workspace.name}"?`}
  description={`This will permanently delete ${workspace.name} and all its ${workspace.projectCount} projects. This action cannot be undone.`}
  confirmLabel="Delete workspace"
  cancelLabel="Keep workspace"
  variant="destructive"
/>
```

---

## Drawer / Sheet

```tsx
interface DrawerProps {
  open: boolean;
  onClose: () => void;
  title: string;
  side?: 'left' | 'right' | 'bottom';
  width?: string;
  children: React.ReactNode;
}

function Drawer({ open, onClose, title, side = 'right', width = '400px', children }: DrawerProps) {
  return (
    <>
      {open && (
        <div className="dialog-backdrop" onClick={onClose} aria-hidden />
      )}
      <div
        role="dialog"
        aria-modal="true"
        aria-labelledby="drawer-title"
        className={`drawer drawer--${side} ${open ? 'drawer--open' : ''}`}
        style={{ ['--drawer-width' as string]: width }}
        aria-hidden={!open}
      >
        <div className="drawer-header">
          <h2 id="drawer-title">{title}</h2>
          <button onClick={onClose} aria-label="Close panel">
            <XIcon aria-hidden />
          </button>
        </div>
        <div className="drawer-body">{children}</div>
      </div>
    </>
  );
}
```

```css
.drawer {
  position: fixed;
  top: 0;
  bottom: 0;
  width: var(--drawer-width, 400px);
  background: hsl(var(--background));
  border-left: 1px solid hsl(var(--border));
  box-shadow: var(--shadow-xl);
  z-index: var(--z-modal);
  display: flex;
  flex-direction: column;
  transform: translateX(100%);
  transition: transform 300ms cubic-bezier(0, 0, 0.2, 1);
  will-change: transform;
}

.drawer--right { right: 0; }
.drawer--left  { left: 0; border-left: none; border-right: 1px solid hsl(var(--border)); transform: translateX(-100%); }

.drawer--open { transform: translateX(0); }

@media (prefers-reduced-motion: reduce) {
  .drawer { transition: none; }
}
```

---

## Popover

```tsx
function Popover({ trigger, content, align = 'start', side = 'bottom' }) {
  const [open, setOpen] = useState(false);
  const triggerRef = useRef<HTMLButtonElement>(null);
  const contentRef = useRef<HTMLDivElement>(null);
  
  // Close on outside click
  useEffect(() => {
    if (!open) return;
    function handleOutside(e: MouseEvent) {
      if (!contentRef.current?.contains(e.target as Node) &&
          !triggerRef.current?.contains(e.target as Node)) {
        setOpen(false);
      }
    }
    document.addEventListener('mousedown', handleOutside);
    return () => document.removeEventListener('mousedown', handleOutside);
  }, [open]);
  
  // Close on Escape
  useEffect(() => {
    if (!open) return;
    function handleEsc(e: KeyboardEvent) {
      if (e.key === 'Escape') { setOpen(false); triggerRef.current?.focus(); }
    }
    document.addEventListener('keydown', handleEsc);
    return () => document.removeEventListener('keydown', handleEsc);
  }, [open]);
  
  return (
    <div className="popover-wrapper">
      <button
        ref={triggerRef}
        type="button"
        aria-expanded={open}
        aria-haspopup="dialog"
        onClick={() => setOpen(!open)}
      >
        {trigger}
      </button>
      {open && (
        <div
          ref={contentRef}
          role="dialog"
          className={`popover-content popover-content--${side} popover-content--${align}`}
        >
          {content}
        </div>
      )}
    </div>
  );
}
```

---

## Tooltip

```tsx
// Tooltip — info only, not interactive
function Tooltip({ content, children, side = 'top' }) {
  const [visible, setVisible] = useState(false);
  const timeoutRef = useRef<NodeJS.Timeout>();
  
  function show() {
    clearTimeout(timeoutRef.current);
    timeoutRef.current = setTimeout(() => setVisible(true), 300); // delay
  }
  
  function hide() {
    clearTimeout(timeoutRef.current);
    setVisible(false);
  }
  
  const id = useId();
  
  return (
    <div
      className="tooltip-wrapper"
      onMouseEnter={show}
      onMouseLeave={hide}
      onFocus={show}
      onBlur={hide}
    >
      {React.cloneElement(children, { 'aria-describedby': visible ? id : undefined })}
      {visible && (
        <div
          id={id}
          role="tooltip"
          className={`tooltip tooltip--${side}`}
        >
          {content}
        </div>
      )}
    </div>
  );
}
```

```css
.tooltip-wrapper { position: relative; display: inline-flex; }

.tooltip {
  position: absolute;
  z-index: var(--z-tooltip);
  background: hsl(var(--foreground));
  color: hsl(var(--background));
  font-size: 12px;
  line-height: 1.4;
  padding: 6px 10px;
  border-radius: 6px;
  white-space: nowrap;
  max-width: 280px;
  white-space: normal;
  pointer-events: none; /* tooltips are not interactive */
  box-shadow: var(--shadow-md);
  animation: tooltipFade 150ms ease;
}

.tooltip--top    { bottom: calc(100% + 8px); left: 50%; transform: translateX(-50%); }
.tooltip--bottom { top: calc(100% + 8px); left: 50%; transform: translateX(-50%); }
.tooltip--left   { right: calc(100% + 8px); top: 50%; transform: translateY(-50%); }
.tooltip--right  { left: calc(100% + 8px); top: 50%; transform: translateY(-50%); }

@keyframes tooltipFade {
  from { opacity: 0; }
  to   { opacity: 1; }
}

/* Tooltip arrow */
.tooltip--top::after {
  content: '';
  position: absolute;
  top: 100%;
  left: 50%;
  transform: translateX(-50%);
  border: 5px solid transparent;
  border-top-color: hsl(var(--foreground));
}
```

---

## Alert Dialog vs. Modal vs. Popover vs. Tooltip

| Pattern | Use When | ARIA Role | Dismissible By |
|---------|----------|-----------|----------------|
| **Alert Dialog** | Requires user decision before proceeding | `alertdialog` | Explicit button only |
| **Dialog/Modal** | Focused task, can abandon | `dialog` | Escape, backdrop, close button |
| **Drawer/Sheet** | Complex panel, secondary content | `dialog` | Escape, close button |
| **Popover** | Contextual content, can dismiss easily | `dialog` | Escape, outside click |
| **Tooltip** | Brief supplementary info, non-interactive | `tooltip` | Mouse leave / blur |
| **Toast** | Background notification, auto-dismisses | `status` or `alert` | Auto + manual |

```tsx
// alertdialog — cannot be dismissed by clicking backdrop or pressing Escape alone
<div role="alertdialog" aria-modal="true" aria-labelledby="title" aria-describedby="desc">
  <h2 id="title">Save changes before leaving?</h2>
  <p id="desc">Your changes will be lost if you don't save them.</p>
  <button>Save changes</button>
  <button>Discard changes</button>
  {/* NO close/cancel button that does nothing — user MUST choose */}
</div>
```

---

## Modal Checklist

- [ ] Modal uses `role="dialog"` + `aria-modal="true"` + `aria-labelledby`
- [ ] Focus trapped inside while open
- [ ] Escape key closes the modal
- [ ] Focus returned to trigger element on close
- [ ] Body scroll locked while modal is open
- [ ] Backdrop click closes non-critical modals
- [ ] Alert dialogs (`role="alertdialog"`) require explicit choice — no backdrop close
- [ ] No nested modals (never stack two modals)
- [ ] Animation ≤ 300ms and respects `prefers-reduced-motion`
- [ ] Tooltip uses `role="tooltip"` and `aria-describedby` on trigger
- [ ] Tooltip is NOT interactive (no links/buttons inside)
- [ ] Drawer/sheet has close button visible without scrolling
- [ ] Mobile: dialog becomes bottom sheet
- [ ] Confirmation dialogs: primary action on right, destructive = red
