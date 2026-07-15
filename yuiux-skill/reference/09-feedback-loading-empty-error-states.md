# 09 — Feedback, Loading, Empty & Error States

> The most common UX failure is a blank white screen. Every async operation, every empty list, every error condition needs a designed state — not a missing one.

---

## The State Matrix

Every data-dependent view must design ALL four states:

```
┌─────────────────────────────────────────────┐
│           DATA VIEW STATE MATRIX            │
├───────────────┬─────────────────────────────┤
│ LOADING       │ Something is being fetched  │
│ EMPTY         │ Fetch succeeded, no data    │
│ ERROR         │ Fetch failed                │
│ POPULATED     │ Fetch succeeded, has data   │
└───────────────┴─────────────────────────────┘

Missing any one = incomplete feature.
```

---

## Loading States

### Timing Rules

```
< 100ms:  No indicator (imperceptible)
100–300ms: Pulse on affected element
300ms–1s:  Skeleton screen matching content layout
1s–3s:    Skeleton + optional progress message
3s+:      Progress bar + "This is taking longer than expected"
Background operation: Progress indicator in corner, dismissible
```

### Skeleton Screens

Skeleton screens should match the actual content layout exactly:

```tsx
// ✅ Skeleton matches the actual layout
function UserCardSkeleton() {
  return (
    <div className="user-card" aria-hidden="true">
      <div className="skeleton skeleton-avatar" />
      <div className="user-card-body">
        <div className="skeleton skeleton-text" style={{ width: '60%' }} />
        <div className="skeleton skeleton-text" style={{ width: '40%', marginTop: 8 }} />
      </div>
    </div>
  );
}

function UserCard({ user }: { user: User | null }) {
  if (!user) return <UserCardSkeleton />;
  
  return (
    <div className="user-card">
      <img src={user.avatar} alt="" className="user-avatar" />
      <div className="user-card-body">
        <p className="user-name">{user.name}</p>
        <p className="user-email">{user.email}</p>
      </div>
    </div>
  );
}
```

```css
/* Skeleton base */
.skeleton {
  background: linear-gradient(
    90deg,
    hsl(var(--muted)) 25%,
    hsl(var(--muted) / 0.5) 50%,
    hsl(var(--muted)) 75%
  );
  background-size: 200% 100%;
  animation: shimmer 1.5s linear infinite;
  border-radius: 4px;
}

@keyframes shimmer {
  0%   { background-position: 200% center; }
  100% { background-position: -200% center; }
}

/* Common skeleton shapes */
.skeleton-text   { height: 1em; border-radius: 4px; }
.skeleton-heading{ height: 1.5em; border-radius: 4px; }
.skeleton-avatar { width: 40px; height: 40px; border-radius: 50%; }
.skeleton-image  { aspect-ratio: 16/9; border-radius: var(--radius); }
.skeleton-button { height: 36px; width: 100px; border-radius: var(--radius); }

@media (prefers-reduced-motion: reduce) {
  .skeleton { animation: none; }
}
```

### Spinner (Use Sparingly)

Spinners are appropriate only for short, indeterminate operations. For layout-level loading, always prefer skeletons.

```tsx
function Spinner({ size = 'md', label = 'Loading…' }: SpinnerProps) {
  return (
    <div role="status" aria-label={label} className={`spinner-wrapper spinner-wrapper--${size}`}>
      <svg
        className="spinner"
        xmlns="http://www.w3.org/2000/svg"
        fill="none"
        viewBox="0 0 24 24"
        aria-hidden="true"
      >
        <circle cx="12" cy="12" r="10" stroke="currentColor" strokeWidth="4" className="spinner-track" />
        <path
          d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4z"
          fill="currentColor"
          className="spinner-head"
        />
      </svg>
      <span className="sr-only">{label}</span>
    </div>
  );
}
```

```css
.spinner {
  animation: spin 0.75s linear infinite;
  color: hsl(var(--primary));
}

.spinner-track { opacity: 0.25; }
.spinner-head  { opacity: 0.75; }

@keyframes spin {
  from { transform: rotate(0deg); }
  to   { transform: rotate(360deg); }
}

@media (prefers-reduced-motion: reduce) {
  .spinner { animation: none; }
}

/* Sizes */
.spinner-wrapper--sm { width: 16px; height: 16px; }
.spinner-wrapper--md { width: 24px; height: 24px; }
.spinner-wrapper--lg { width: 40px; height: 40px; }
.spinner-wrapper--xl { width: 64px; height: 64px; }
```

### Full-Page Loading

```tsx
function PageLoading({ message = 'Loading…' }) {
  return (
    <div
      className="page-loading"
      role="status"
      aria-live="polite"
      aria-label={message}
    >
      <Spinner size="lg" />
      <p className="page-loading-text">{message}</p>
    </div>
  );
}
```

```css
.page-loading {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  gap: 16px;
  min-height: 400px;
  color: hsl(var(--muted-foreground));
}
```

---

## Empty States

Empty states are the primary onboarding opportunity. Never show a blank view — always give users context and a path forward.

### Empty State Anatomy

```
┌─────────────────────────────────────┐
│                                     │
│         [Illustration/Icon]         │
│                                     │
│         What's empty here           │
│                                     │
│     Explanation of why it's empty   │
│     and what this section does      │
│                                     │
│       [Primary Action Button]       │
│       [Secondary Link/Action]       │
│                                     │
└─────────────────────────────────────┘
```

### Empty State Copy Formula

```
Heading: What's (not) here
Subtext: Why + What to do
CTA: Verb + Noun action

Examples:
  "No projects yet"
  "Create your first project to start collaborating with your team."
  [Create project]

  "No results found"
  "Try a different search term or remove some filters."
  [Clear filters]

  "No messages"
  "You're all caught up! Messages from your team will appear here."
```

### Empty State Component

```tsx
interface EmptyStateProps {
  icon?: React.ComponentType<{ className?: string }>;
  illustration?: string;
  title: string;
  description?: string;
  action?: {
    label: string;
    onClick: () => void;
  };
  secondaryAction?: {
    label: string;
    onClick: () => void;
  };
}

function EmptyState({ icon: Icon, illustration, title, description, action, secondaryAction }: EmptyStateProps) {
  return (
    <div className="empty-state" role="status">
      {illustration ? (
        <img src={illustration} alt="" className="empty-state-illustration" aria-hidden />
      ) : Icon ? (
        <div className="empty-state-icon-wrapper">
          <Icon className="empty-state-icon" aria-hidden />
        </div>
      ) : null}
      
      <h3 className="empty-state-title">{title}</h3>
      
      {description && (
        <p className="empty-state-description">{description}</p>
      )}
      
      {action && (
        <div className="empty-state-actions">
          <Button onClick={action.onClick}>{action.label}</Button>
          {secondaryAction && (
            <Button variant="ghost" onClick={secondaryAction.onClick}>
              {secondaryAction.label}
            </Button>
          )}
        </div>
      )}
    </div>
  );
}
```

```css
.empty-state {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  text-align: center;
  padding: 48px 24px;
  gap: 12px;
  color: hsl(var(--muted-foreground));
  min-height: 320px;
}

.empty-state-illustration {
  width: 160px;
  height: 160px;
  object-fit: contain;
  margin-bottom: 8px;
}

.empty-state-icon-wrapper {
  width: 64px;
  height: 64px;
  border-radius: 50%;
  background: hsl(var(--muted));
  display: flex;
  align-items: center;
  justify-content: center;
  margin-bottom: 8px;
}

.empty-state-icon {
  width: 32px;
  height: 32px;
  color: hsl(var(--muted-foreground));
}

.empty-state-title {
  font-size: 18px;
  font-weight: 600;
  color: hsl(var(--foreground));
  margin: 0;
}

.empty-state-description {
  font-size: 14px;
  max-width: 360px;
  line-height: 1.6;
  margin: 0;
}

.empty-state-actions {
  display: flex;
  gap: 8px;
  margin-top: 8px;
}
```

### Context-Specific Empty States

```tsx
// ✅ Specific to what's missing
function ProjectsEmptyState() {
  return (
    <EmptyState
      icon={FolderIcon}
      title="No projects yet"
      description="Create your first project to start organizing your work and collaborating with your team."
      action={{ label: 'Create project', onClick: openCreateDialog }}
    />
  );
}

// ✅ Search zero results — different from "no data"
function SearchEmptyState({ query }: { query: string }) {
  return (
    <EmptyState
      icon={SearchIcon}
      title={`No results for "${query}"`}
      description="Try a different search term, or check your spelling."
      secondaryAction={{ label: 'Clear search', onClick: clearSearch }}
    />
  );
}

// ✅ Filter zero results
function FilterEmptyState({ filterCount }: { filterCount: number }) {
  return (
    <EmptyState
      icon={FilterIcon}
      title="No items match your filters"
      description={`${filterCount} filter${filterCount !== 1 ? 's' : ''} applied. Try removing some to see more results.`}
      action={{ label: 'Clear all filters', onClick: clearFilters }}
    />
  );
}
```

---

## Error States

### Error Categories

| Type | Cause | Response |
|------|-------|----------|
| **Network error** | No connection | Retry button, offline indicator |
| **Server error (5xx)** | Backend problem | Generic message, retry |
| **Not found (404)** | Item deleted or bad URL | Navigate back, search |
| **Permission error (403)** | Unauthorized | Explain what access needed |
| **Validation error** | Bad input | Inline, specific, actionable |
| **Rate limit (429)** | Too many requests | When to try again |
| **Timeout** | Slow network | Cancel + retry |

### Inline Error (Component Level)

```tsx
function ErrorMessage({ 
  title = 'Something went wrong',
  description,
  onRetry
}: ErrorMessageProps) {
  return (
    <div className="error-state" role="alert">
      <AlertCircleIcon className="error-state-icon" aria-hidden />
      <div className="error-state-content">
        <p className="error-state-title">{title}</p>
        {description && (
          <p className="error-state-description">{description}</p>
        )}
        {onRetry && (
          <button type="button" onClick={onRetry} className="error-retry-btn">
            Try again
          </button>
        )}
      </div>
    </div>
  );
}
```

### Error Boundary (React)

```tsx
class ErrorBoundary extends React.Component<
  { children: React.ReactNode; fallback?: React.ComponentType<{ error: Error; reset: () => void }> },
  { error: Error | null }
> {
  state = { error: null };
  
  static getDerivedStateFromError(error: Error) {
    return { error };
  }
  
  componentDidCatch(error: Error, info: React.ErrorInfo) {
    // Log to error tracking service
    console.error('ErrorBoundary caught:', error, info);
    // logError(error, { componentStack: info.componentStack });
  }
  
  reset = () => this.setState({ error: null });
  
  render() {
    if (this.state.error) {
      const FallbackComponent = this.props.fallback ?? DefaultErrorFallback;
      return <FallbackComponent error={this.state.error} reset={this.reset} />;
    }
    return this.props.children;
  }
}

function DefaultErrorFallback({ error, reset }: { error: Error; reset: () => void }) {
  return (
    <div className="error-boundary" role="alert">
      <AlertTriangleIcon aria-hidden />
      <h2>Something went wrong</h2>
      <p>
        An unexpected error occurred. This has been reported automatically.
      </p>
      <div className="error-boundary-actions">
        <button type="button" onClick={reset}>Try again</button>
        <a href="/">Go to home</a>
      </div>
      {process.env.NODE_ENV === 'development' && (
        <pre className="error-detail">{error.message}</pre>
      )}
    </div>
  );
}
```

### Not Found (404) Page

```tsx
function NotFoundPage() {
  const navigate = useNavigate();
  
  return (
    <div className="not-found-page">
      <div className="not-found-code" aria-hidden>404</div>
      <h1>Page not found</h1>
      <p>The page you're looking for doesn't exist or has been moved.</p>
      <div className="not-found-actions">
        <button type="button" onClick={() => navigate(-1)}>
          ← Go back
        </button>
        <a href="/">Go to home</a>
      </div>
    </div>
  );
}
```

---

## Optimistic UI

Show success immediately, then confirm with server. Revert if server fails.

```tsx
function TodoList() {
  const [todos, setTodos] = useState<Todo[]>(initialTodos);
  const [pendingDeletes, setPendingDeletes] = useState<Set<string>>(new Set());
  
  async function deleteTodo(id: string) {
    // Optimistic: hide immediately
    setTodos(prev => prev.filter(t => t.id !== id));
    setPendingDeletes(prev => new Set([...prev, id]));
    
    try {
      await api.deleteTodo(id);
      setPendingDeletes(prev => { const next = new Set(prev); next.delete(id); return next; });
    } catch (error) {
      // Revert: restore the item
      setTodos(prev => [...prev, findTodo(id)].sort(byCreatedAt));
      setPendingDeletes(prev => { const next = new Set(prev); next.delete(id); return next; });
      toast.error('Could not delete item. Try again.');
    }
  }
  
  async function toggleTodo(id: string) {
    const todo = todos.find(t => t.id === id)!;
    // Optimistic toggle
    setTodos(prev => prev.map(t => t.id === id ? { ...t, done: !t.done } : t));
    
    try {
      await api.updateTodo(id, { done: !todo.done });
    } catch {
      // Revert
      setTodos(prev => prev.map(t => t.id === id ? { ...t, done: todo.done } : t));
      toast.error('Could not update item.');
    }
  }
  
  return (
    <ul>
      {todos.map(todo => (
        <li 
          key={todo.id}
          className={pendingDeletes.has(todo.id) ? 'opacity-50' : ''}
        >
          <TodoItem 
            todo={todo}
            onToggle={() => toggleTodo(todo.id)}
            onDelete={() => deleteTodo(todo.id)}
          />
        </li>
      ))}
    </ul>
  );
}
```

---

## Data Fetching State Management

```tsx
// Complete state management pattern
type FetchState<T> = 
  | { status: 'idle' }
  | { status: 'loading' }
  | { status: 'success'; data: T }
  | { status: 'error'; error: Error };

function useData<T>(fetchFn: () => Promise<T>) {
  const [state, setState] = useState<FetchState<T>>({ status: 'idle' });
  
  const fetch = useCallback(async () => {
    setState({ status: 'loading' });
    try {
      const data = await fetchFn();
      setState({ status: 'success', data });
    } catch (error) {
      setState({ status: 'error', error: error as Error });
    }
  }, [fetchFn]);
  
  useEffect(() => { fetch(); }, [fetch]);
  
  return { state, refetch: fetch };
}

// Usage — covers all states
function UserProfile({ userId }) {
  const { state, refetch } = useData(() => api.getUser(userId));
  
  if (state.status === 'loading') return <ProfileSkeleton />;
  if (state.status === 'error') return (
    <ErrorMessage
      title="Could not load profile"
      description="Check your connection and try again."
      onRetry={refetch}
    />
  );
  if (state.status === 'idle' || !('data' in state)) return null;
  if (!state.data) return (
    <EmptyState
      title="Profile not found"
      description="This user doesn't exist or has been removed."
    />
  );
  
  return <ProfileContent user={state.data} />;
}
```

---

## State Checklist

- [ ] Loading skeleton designed for every data-dependent view
- [ ] Skeleton matches actual content layout (same element sizes/positions)
- [ ] Empty state has title + description + action for every list/table
- [ ] Search zero results has different empty state from "no data"
- [ ] Filter zero results offers "clear filters" action
- [ ] Error state has retry button for retriable errors
- [ ] Error boundary wraps all page-level components
- [ ] Not found page has navigation options (back + home)
- [ ] Network errors give specific message (not generic)
- [ ] Optimistic updates revert on server error with toast notification
- [ ] `aria-live="polite"` on dynamic regions that change without user action
- [ ] `role="status"` on loading indicators
- [ ] `role="alert"` on error messages that appear dynamically
- [ ] Skeleton has `aria-hidden="true"` (it's decorative)
- [ ] Spinner has `aria-label` with current operation description
