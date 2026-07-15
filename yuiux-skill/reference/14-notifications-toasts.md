# 14 — Notifications, Toasts & Alerts

> Notifications are interruptions. Every alert you show consumes user attention. Only show what users need to see, when they need to see it.

---

## The Notification Taxonomy

| Type | Urgency | Dismissal | Examples |
|------|---------|-----------|---------|
| **Toast / Snackbar** | Low–Medium | Auto + manual | "File saved", "Email sent" |
| **Alert Banner** | Medium | Manual | "Your plan expires in 3 days" |
| **Inline Error** | High | On fix | Form validation errors |
| **Dialog / Alert** | Critical | Explicit | "You have unsaved changes" |
| **System Notification** | Varies | OS-controlled | Push notifications |
| **Badge** | Ambient | On view | Unread count (12) |

---

## Toast / Snackbar

### When to Use
```
✅ Confirmation after action: "Project created"
✅ Background operation complete: "Export ready to download"
✅ Non-critical error: "Failed to save. Try again."
✅ Undo notification: "Item deleted. Undo"

❌ Critical errors requiring user action (use alert dialog)
❌ Information that needs to be read carefully (use inline or banner)
❌ Multiple simultaneous toasts (max 1–2 visible at once)
❌ Long error messages (toast should be ≤80 chars)
```

### Toast Component

```tsx
type ToastVariant = 'default' | 'success' | 'warning' | 'error';

interface Toast {
  id: string;
  message: string;
  description?: string;
  variant: ToastVariant;
  duration?: number; // ms, 0 = persistent
  action?: { label: string; onClick: () => void };
}

// Toast store (simple)
const toastQueue = new EventTarget();

function toast(options: Omit<Toast, 'id'>) {
  const t: Toast = { id: crypto.randomUUID(), duration: 5000, ...options };
  toastQueue.dispatchEvent(new CustomEvent('add', { detail: t }));
  return t.id;
}

toast.success = (message: string, opts?: Partial<Toast>) =>
  toast({ message, variant: 'success', ...opts });

toast.error = (message: string, opts?: Partial<Toast>) =>
  toast({ message, variant: 'error', duration: 0, ...opts }); // errors persistent by default

toast.warning = (message: string, opts?: Partial<Toast>) =>
  toast({ message, variant: 'warning', ...opts });

// Hook
function useToasts() {
  const [toasts, setToasts] = useState<Toast[]>([]);
  
  useEffect(() => {
    function addToast(e: Event) {
      const t = (e as CustomEvent<Toast>).detail;
      setToasts(prev => [...prev.slice(-4), t]); // max 5 visible
      
      if (t.duration && t.duration > 0) {
        setTimeout(() => removeToast(t.id), t.duration);
      }
    }
    
    toastQueue.addEventListener('add', addToast);
    return () => toastQueue.removeEventListener('add', addToast);
  }, []);
  
  function removeToast(id: string) {
    setToasts(prev => prev.filter(t => t.id !== id));
  }
  
  return { toasts, removeToast };
}

// Toast container
function ToastContainer() {
  const { toasts, removeToast } = useToasts();
  
  return (
    <div
      aria-live="polite"
      aria-label="Notifications"
      className="toast-container"
      role="region"
    >
      {toasts.map(t => (
        <ToastItem key={t.id} toast={t} onDismiss={() => removeToast(t.id)} />
      ))}
    </div>
  );
}

// Individual toast
function ToastItem({ toast: t, onDismiss }: { toast: Toast; onDismiss: () => void }) {
  const [exiting, setExiting] = useState(false);
  
  function dismiss() {
    setExiting(true);
    setTimeout(onDismiss, 200); // wait for exit animation
  }
  
  const icons = {
    default: null,
    success: CheckCircleIcon,
    warning: AlertTriangleIcon,
    error:   XCircleIcon,
  };
  
  const Icon = icons[t.variant];
  
  return (
    <div
      role={t.variant === 'error' ? 'alert' : 'status'}
      aria-atomic="true"
      className={`toast toast--${t.variant} ${exiting ? 'toast--exit' : ''}`}
    >
      {Icon && <Icon className="toast-icon" aria-hidden />}
      <div className="toast-content">
        <p className="toast-message">{t.message}</p>
        {t.description && <p className="toast-description">{t.description}</p>}
        {t.action && (
          <button
            type="button"
            onClick={() => { t.action!.onClick(); dismiss(); }}
            className="toast-action"
          >
            {t.action.label}
          </button>
        )}
      </div>
      <button
        type="button"
        onClick={dismiss}
        aria-label="Dismiss notification"
        className="toast-dismiss"
      >
        <XIcon aria-hidden />
      </button>
    </div>
  );
}
```

```css
/* Toast container */
.toast-container {
  position: fixed;
  bottom: 24px;
  right: 24px;
  z-index: var(--z-toast);
  display: flex;
  flex-direction: column-reverse;
  gap: 8px;
  max-width: 420px;
  width: calc(100vw - 48px);
  pointer-events: none; /* container is click-through */
}

@media (max-width: 640px) {
  .toast-container {
    bottom: 0;
    right: 0;
    left: 0;
    max-width: 100%;
    width: 100%;
    padding: 0 0 env(safe-area-inset-bottom, 0);
  }
}

/* Individual toast */
.toast {
  display: flex;
  align-items: flex-start;
  gap: 12px;
  padding: 12px 16px;
  border-radius: 8px;
  background: hsl(var(--foreground));
  color: hsl(var(--background));
  box-shadow: var(--shadow-lg);
  border: 1px solid rgba(255,255,255,0.1);
  pointer-events: auto; /* toast itself is clickable */
  animation: toastEnter 300ms cubic-bezier(0, 0, 0.2, 1);
  max-width: 100%;
  word-break: break-word;
}

.toast--exit {
  animation: toastExit 200ms ease forwards;
}

@keyframes toastEnter {
  from { opacity: 0; transform: translateX(100%) scale(0.95); }
  to   { opacity: 1; transform: translateX(0) scale(1); }
}

@keyframes toastExit {
  from { opacity: 1; transform: translateX(0); max-height: 200px; }
  to   { opacity: 0; transform: translateX(100%); max-height: 0; }
}

@media (max-width: 640px) {
  @keyframes toastEnter {
    from { opacity: 0; transform: translateY(100%); }
    to   { opacity: 1; transform: translateY(0); }
  }
}

@media (prefers-reduced-motion: reduce) {
  .toast, .toast--exit { animation: none; }
}

/* Variant colors */
.toast--success  { background: #166534; }
.toast--warning  { background: #92400e; }
.toast--error    { background: #991b1b; }

/* Toast parts */
.toast-icon    { width: 18px; height: 18px; flex-shrink: 0; margin-top: 1px; }
.toast-content { flex: 1; min-width: 0; }
.toast-message { font-size: 14px; font-weight: 500; margin: 0; }
.toast-description { font-size: 13px; opacity: 0.85; margin: 2px 0 0; }
.toast-action  {
  background: none; border: none; padding: 0; cursor: pointer;
  font-size: 13px; font-weight: 600; text-decoration: underline;
  color: inherit; margin-top: 6px; display: inline-block;
}
.toast-dismiss {
  background: none; border: none; cursor: pointer; padding: 0;
  color: inherit; opacity: 0.7; flex-shrink: 0;
  width: 20px; height: 20px; display: flex; align-items: center;
}
.toast-dismiss:hover { opacity: 1; }
```

---

## Alert Banner

For persistent page-level messages:

```tsx
interface AlertBannerProps {
  variant?: 'info' | 'success' | 'warning' | 'error';
  title?: string;
  description: string;
  action?: { label: string; href?: string; onClick?: () => void };
  dismissible?: boolean;
  onDismiss?: () => void;
}

function AlertBanner({
  variant = 'info',
  title,
  description,
  action,
  dismissible = false,
  onDismiss,
}: AlertBannerProps) {
  const config = {
    info:    { icon: InfoIcon,         bg: 'bg-blue-50 border-blue-200',   text: 'text-blue-900'   },
    success: { icon: CheckCircleIcon,  bg: 'bg-green-50 border-green-200', text: 'text-green-900'  },
    warning: { icon: AlertTriangle,    bg: 'bg-amber-50 border-amber-200', text: 'text-amber-900'  },
    error:   { icon: XCircleIcon,      bg: 'bg-red-50 border-red-200',     text: 'text-red-900'    },
  };
  
  const { icon: Icon, bg, text } = config[variant];
  
  return (
    <div
      role={variant === 'error' ? 'alert' : 'status'}
      aria-live={variant === 'error' ? 'assertive' : 'polite'}
      className={`alert-banner ${bg} ${text}`}
    >
      <Icon className="alert-banner-icon" aria-hidden />
      <div className="alert-banner-content">
        {title && <strong className="alert-banner-title">{title}</strong>}
        <span className="alert-banner-desc">{description}</span>
        {action && (
          action.href ? (
            <a href={action.href} className="alert-banner-action">{action.label}</a>
          ) : (
            <button type="button" onClick={action.onClick} className="alert-banner-action">
              {action.label}
            </button>
          )
        )}
      </div>
      {dismissible && (
        <button
          type="button"
          onClick={onDismiss}
          aria-label="Dismiss this notification"
          className="alert-banner-dismiss"
        >
          <XIcon aria-hidden />
        </button>
      )}
    </div>
  );
}
```

```css
.alert-banner {
  display: flex;
  align-items: flex-start;
  gap: 12px;
  padding: 12px 16px;
  border: 1px solid;
  border-radius: var(--radius);
  font-size: 14px;
  line-height: 1.5;
}

.alert-banner-icon { width: 18px; height: 18px; flex-shrink: 0; margin-top: 1px; }
.alert-banner-content { flex: 1; min-width: 0; }
.alert-banner-title { margin-right: 8px; }
.alert-banner-action {
  background: none; border: none; padding: 0; cursor: pointer;
  text-decoration: underline; color: inherit; font: inherit;
  font-weight: 600; margin-left: 8px;
}
.alert-banner-dismiss {
  background: none; border: none; cursor: pointer;
  padding: 0; opacity: 0.7; flex-shrink: 0;
}
.alert-banner-dismiss:hover { opacity: 1; }
```

---

## Notification Bell / Inbox

```tsx
function NotificationBell({ notifications, onView, onMarkAllRead }) {
  const unreadCount = notifications.filter(n => !n.read).length;
  
  return (
    <Popover
      trigger={
        <button
          type="button"
          aria-label={`Notifications${unreadCount > 0 ? `, ${unreadCount} unread` : ''}`}
          aria-haspopup="dialog"
          className="notification-bell-btn"
        >
          <BellIcon aria-hidden />
          {unreadCount > 0 && (
            <span 
              className="notification-badge"
              aria-hidden // count is in aria-label above
            >
              {unreadCount > 99 ? '99+' : unreadCount}
            </span>
          )}
        </button>
      }
      content={
        <div className="notification-panel" role="dialog" aria-label="Notifications">
          <div className="notification-panel-header">
            <h3>Notifications</h3>
            {unreadCount > 0 && (
              <button type="button" onClick={onMarkAllRead} className="btn btn-ghost btn-sm">
                Mark all read
              </button>
            )}
          </div>
          
          {notifications.length === 0 ? (
            <div className="notification-empty">
              <BellOffIcon aria-hidden />
              <p>No notifications</p>
            </div>
          ) : (
            <ul role="list" className="notification-list">
              {notifications.map(n => (
                <li key={n.id} className={`notification-item ${!n.read ? 'unread' : ''}`}>
                  <button type="button" onClick={() => onView(n)} className="notification-row">
                    {!n.read && <span className="unread-dot" aria-label="Unread" />}
                    <div className="notification-content">
                      <p className="notification-title">{n.title}</p>
                      <p className="notification-time">{formatRelativeTime(n.createdAt)}</p>
                    </div>
                  </button>
                </li>
              ))}
            </ul>
          )}
        </div>
      }
    />
  );
}
```

---

## Progress Notifications

```tsx
// Toast with progress bar (long operations)
function ProgressToast({ operationId, title }: { operationId: string; title: string }) {
  const [progress, setProgress] = useState(0);
  const [status, setStatus] = useState<'running' | 'done' | 'error'>('running');
  
  useEffect(() => {
    const interval = subscribeToProgress(operationId, (pct) => {
      setProgress(pct);
      if (pct >= 100) setStatus('done');
    });
    return () => interval.unsubscribe();
  }, [operationId]);
  
  return (
    <div
      role="status"
      aria-live="polite"
      aria-label={`${title}: ${status === 'done' ? 'complete' : `${Math.round(progress)}%`}`}
      className="toast toast--default"
    >
      <div className="toast-content">
        <div className="toast-header">
          <p className="toast-message">{title}</p>
          <span className="toast-percent" aria-hidden>{Math.round(progress)}%</span>
        </div>
        <div 
          className="toast-progress-track"
          role="progressbar"
          aria-valuenow={progress}
          aria-valuemin={0}
          aria-valuemax={100}
        >
          <div
            className="toast-progress-fill"
            style={{ width: `${progress}%` }}
          />
        </div>
        {status === 'done' && (
          <p className="toast-success">Complete!</p>
        )}
      </div>
    </div>
  );
}
```

---

## Notification Timing

```
Action result toast:  Show immediately on action completion
Error toast:         Show immediately, no auto-dismiss (errors require acknowledgment)
Success toast:       Auto-dismiss after 4–5 seconds
Info toast:          Auto-dismiss after 5–7 seconds  
Warning banner:      Persist until dismissed by user
System alert:        Persist + manual dismiss

Never:
  ❌ Multiple toasts in rapid succession from same action
  ❌ Toasts that dismiss in < 2 seconds (not enough time to read)
  ❌ Error toasts that auto-dismiss (user may miss them)
  ❌ Toasts with long content (>80 chars) — use banner instead
```

---

## Notification Anti-Patterns

| Anti-Pattern | Problem | Fix |
|-------------|---------|-----|
| Toast for critical errors | User may miss it | Use dialog or inline error |
| Auto-dismiss on errors | Error message disappears before user reads | Keep errors until dismissed |
| Multiple simultaneous toasts | Overwhelming, visual noise | Stack and limit to 3 max |
| Toast disappears too fast (<2s) | Can't read it | Minimum 4 seconds |
| No dismiss button on persistent banners | Trapped | Always add dismiss on banners |
| Using alert on every API success | Alert fatigue | Only toast meaningful completions |
| Toast blocked by mobile keyboard | Overlaps input | Use top position on mobile forms |
| No ARIA live region | Screen readers miss notifications | Always `aria-live` on toast container |

---

## Notification Checklist

- [ ] Toast container has `aria-live="polite"` and `role="region"`
- [ ] Error toasts use `role="alert"` (assertive announcement)
- [ ] Error toasts don't auto-dismiss (duration: 0 or very long)
- [ ] Success/info toasts auto-dismiss after 4–5 seconds
- [ ] Every toast has a dismiss button
- [ ] Toast dismiss button has `aria-label="Dismiss notification"`
- [ ] Max 3–5 toasts visible at once (older ones drop off)
- [ ] Toasts animate in/out with `prefers-reduced-motion` support
- [ ] Notification bell badge count included in `aria-label`
- [ ] Alert banners have `role="alert"` or `role="status"`
- [ ] Persistent banners are dismissible
- [ ] Progress toasts use `role="progressbar"` with `aria-valuenow`
- [ ] Action button in toast (e.g., "Undo") dismisses toast on click
- [ ] Mobile toasts don't overlap active inputs/keyboard
