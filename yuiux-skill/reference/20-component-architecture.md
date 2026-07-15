# 20 — Component Architecture

> A well-designed component system is like a well-organized kitchen. Every tool has its place, every tool does one job well, and you can always find what you need.

---

## Component Classification

### Atomic Design Hierarchy

```
ATOMS            — primitive building blocks
  Button, Input, Badge, Avatar, Icon, Spinner

MOLECULES        — simple combinations of atoms
  FormField (Label + Input + Error)
  SearchInput (Input + Icon + Clear button)
  Card Header (Avatar + Title + Badge)

ORGANISMS        — complex, self-contained components
  Navigation, Form, DataTable, UserCard, ProductGrid

TEMPLATES        — page-level layouts
  DashboardLayout, AuthLayout, SettingsLayout

PAGES            — full page implementations
  DashboardPage, LoginPage, SettingsPage
```

---

## Component Design Principles

### Single Responsibility

Each component does exactly one thing. If you're struggling to name it, it's doing too many things.

```tsx
// ❌ Too many responsibilities
function UserProfileWithEditAndDeleteAndAvatarAndBio({ user, onEdit, onDelete }) {
  // displays user, handles editing, handles deletion, shows bio, manages avatar
}

// ✅ Single responsibilities
function UserAvatar({ user }: { user: User })
function UserName({ user }: { user: User })
function UserBio({ user }: { user: User })
function UserCard({ user, onEdit, onDelete }: UserCardProps) {
  return (
    <article>
      <UserAvatar user={user} />
      <UserName user={user} />
      <UserBio user={user} />
      <UserActions user={user} onEdit={onEdit} onDelete={onDelete} />
    </article>
  );
}
```

### Composition Over Configuration

```tsx
// ❌ Configuration-heavy component
<DataTable
  title="Users"
  description="Manage your team"
  showSearch={true}
  searchPlaceholder="Search users…"
  showFilters={true}
  filterConfig={filterConfig}
  sortable={true}
  selectable={true}
  onSelect={handleSelect}
  showPagination={true}
  pageSize={25}
  showActions={true}
  actionItems={['edit', 'delete']}
  onAction={handleAction}
/>

// ✅ Composable — use only what you need
<TableLayout>
  <TableToolbar>
    <SearchInput placeholder="Search users…" onChange={setSearch} />
    <FilterDropdown config={filterConfig} value={filters} onChange={setFilters} />
    {selected.size > 0 && <BulkActions count={selected.size} onDelete={deleteSelected} />}
  </TableToolbar>
  
  <DataTable
    data={filteredUsers}
    columns={columns}
    sortable
    selectable
    selected={selected}
    onSelectionChange={setSelected}
  />
  
  <Pagination page={page} total={total} pageSize={25} onChange={setPage} />
</TableLayout>
```

### Open/Closed Principle

Components should be open for extension, closed for modification.

```tsx
// ❌ Hardcoded — can't be customized
function Button({ label, onClick }) {
  return (
    <button onClick={onClick} style={{ background: 'blue', color: 'white' }}>
      {label}
    </button>
  );
}

// ✅ Open for extension
interface ButtonProps extends React.ButtonHTMLAttributes<HTMLButtonElement> {
  variant?: 'default' | 'outline' | 'ghost' | 'destructive';
  size?: 'sm' | 'md' | 'lg';
  loading?: boolean;
  icon?: React.ComponentType<{ className?: string }>;
}

function Button({ variant = 'default', size = 'md', loading, icon: Icon, children, className, ...props }: ButtonProps) {
  return (
    <button
      {...props} // spread ALL native button props — className, onClick, disabled, etc.
      disabled={props.disabled || loading}
      aria-busy={loading}
      className={cn(buttonVariants({ variant, size }), className)}
    >
      {loading ? <Spinner aria-hidden /> : Icon && <Icon aria-hidden className={iconSize[size]} />}
      {children}
    </button>
  );
}
```

---

## Props Patterns

### Required vs Optional Props

```tsx
interface CardProps {
  // Required: the component can't render without these
  title: string;
  children: React.ReactNode;
  
  // Optional with sensible defaults
  variant?: 'default' | 'outlined' | 'filled';
  padding?: 'sm' | 'md' | 'lg';
  
  // Optional with undefined default (feature off by default)
  href?: string;
  onClick?: () => void;
  footer?: React.ReactNode;
  className?: string;
}
```

### Render Props / Children as Function

```tsx
// Render prop pattern — gives consumer control over rendering
function DataFetcher<T>({
  url,
  children,
}: {
  url: string;
  children: (state: { data: T | null; loading: boolean; error: Error | null; refetch: () => void }) => React.ReactNode;
}) {
  const { data, loading, error, refetch } = useFetch<T>(url);
  return <>{children({ data, loading, error, refetch })}</>;
}

// Usage:
<DataFetcher<User[]> url="/api/users">
  {({ data, loading, error, refetch }) => {
    if (loading) return <TableSkeleton />;
    if (error) return <ErrorMessage onRetry={refetch} />;
    if (!data?.length) return <EmptyState />;
    return <UserTable users={data} />;
  }}
</DataFetcher>
```

### as/asChild Polymorphic Components

```tsx
// Polymorphic component — can be any HTML element or component
interface BoxProps<T extends React.ElementType> {
  as?: T;
  children?: React.ReactNode;
  className?: string;
}

type PolymorphicProps<T extends React.ElementType> = BoxProps<T> &
  Omit<React.ComponentPropsWithRef<T>, keyof BoxProps<T>>;

function Box<T extends React.ElementType = 'div'>({
  as,
  children,
  className,
  ...props
}: PolymorphicProps<T>) {
  const Component = as ?? 'div';
  return <Component className={className} {...props}>{children}</Component>;
}

// Usage:
<Box as="section" className="hero">...</Box>
<Box as="article" className="card">...</Box>
<Box as={Link} href="/about">...</Box> // rendered as React Router Link
```

### Slot / asChild Pattern (Radix UI)

```tsx
// Radix Slot — renders children's root element instead of its own
import { Slot } from '@radix-ui/react-slot';

function Button({ asChild, children, ...props }) {
  const Comp = asChild ? Slot : 'button';
  return <Comp {...props}>{children}</Comp>;
}

// Usage:
// Renders as <button>Click me</button>
<Button onClick={handleClick}>Click me</Button>

// Renders as <a href="/page" class="button">Click me</a>
<Button asChild>
  <a href="/page">Click me</a>
</Button>
```

---

## State Management Patterns

### Lifting State

```tsx
// ✅ State owned by lowest common ancestor
function ProductFilters() {
  const [filters, setFilters] = useState<Filters>({});
  
  return (
    <>
      <FilterSidebar filters={filters} onChange={setFilters} />
      <ProductGrid filters={filters} />
    </>
  );
}
```

### Uncontrolled vs Controlled Components

```tsx
// Uncontrolled: component manages its own state
function UncontrolledInput({ defaultValue, onChange }) {
  const [value, setValue] = useState(defaultValue ?? '');
  
  function handleChange(e) {
    setValue(e.target.value);
    onChange?.(e.target.value);
  }
  
  return <input value={value} onChange={handleChange} />;
}

// Controlled: parent manages state
function ControlledInput({ value, onChange }) {
  return <input value={value} onChange={e => onChange(e.target.value)} />;
}

// Flexible: support both (pattern used by most library components)
function FlexibleInput({ 
  value: controlledValue,
  defaultValue,
  onChange,
}: { value?: string; defaultValue?: string; onChange?: (v: string) => void }) {
  const isControlled = controlledValue !== undefined;
  const [uncontrolledValue, setUncontrolledValue] = useState(defaultValue ?? '');
  
  const value = isControlled ? controlledValue : uncontrolledValue;
  
  function handleChange(newValue: string) {
    if (!isControlled) setUncontrolledValue(newValue);
    onChange?.(newValue);
  }
  
  return <input value={value} onChange={e => handleChange(e.target.value)} />;
}
```

### Context for Cross-Cutting State

```tsx
// Theme, auth, toast — things many components need
interface ThemeContextValue {
  theme: 'light' | 'dark' | 'system';
  setTheme: (theme: ThemeContextValue['theme']) => void;
}

const ThemeContext = createContext<ThemeContextValue | null>(null);

function ThemeProvider({ children }: { children: React.ReactNode }) {
  const [theme, setTheme] = useState<ThemeContextValue['theme']>('system');
  
  return (
    <ThemeContext.Provider value={{ theme, setTheme }}>
      {children}
    </ThemeContext.Provider>
  );
}

function useTheme() {
  const ctx = useContext(ThemeContext);
  if (!ctx) throw new Error('useTheme must be used within ThemeProvider');
  return ctx;
}
```

---

## Error Boundaries

```tsx
// Wrap every page, major section, and complex widget
function App() {
  return (
    <ErrorBoundary fallback={AppCrash}>
      <ThemeProvider>
        <AuthProvider>
          <Router>
            <Routes>
              <Route path="/*" element={
                <ErrorBoundary fallback={PageError}>
                  <PageLayout />
                </ErrorBoundary>
              } />
            </Routes>
          </Router>
        </AuthProvider>
      </ThemeProvider>
    </ErrorBoundary>
  );
}
```

---

## Component File Structure

```
src/
├── components/
│   ├── ui/                    # Primitive components (atoms/molecules)
│   │   ├── Button.tsx
│   │   ├── Input.tsx
│   │   ├── Card.tsx
│   │   ├── Badge.tsx
│   │   ├── Avatar.tsx
│   │   └── index.ts           # barrel export
│   │
│   ├── layout/                # Layout components
│   │   ├── AppLayout.tsx
│   │   ├── Sidebar.tsx
│   │   ├── Header.tsx
│   │   └── PageContainer.tsx
│   │
│   ├── features/              # Feature-specific components
│   │   ├── users/
│   │   │   ├── UserCard.tsx
│   │   │   ├── UserTable.tsx
│   │   │   └── UserFilters.tsx
│   │   ├── products/
│   │   └── auth/
│   │
│   └── common/                # Shared higher-order components
│       ├── ErrorBoundary.tsx
│       ├── EmptyState.tsx
│       └── LoadingState.tsx
│
├── hooks/                     # Custom hooks
│   ├── useTheme.ts
│   ├── useDebounce.ts
│   ├── useBreakpoint.ts
│   └── useLocalStorage.ts
│
├── contexts/                  # React contexts
│   ├── ThemeContext.tsx
│   └── AuthContext.tsx
│
└── lib/                       # Utilities
    ├── cn.ts                  # className merger
    ├── api.ts                 # API client
    └── utils.ts               # General utils
```

### Component File Template

```tsx
// UserCard.tsx — component file template

// ============================================================
// TYPES
// ============================================================
export interface UserCardProps {
  user: User;
  variant?: 'default' | 'compact';
  onEdit?: (user: User) => void;
  onDelete?: (user: User) => void;
  className?: string;
}

// ============================================================
// SUBCOMPONENTS (internal, not exported)
// ============================================================
function UserCardActions({ user, onEdit, onDelete }: Pick<UserCardProps, 'user' | 'onEdit' | 'onDelete'>) {
  // ...
}

// ============================================================
// MAIN COMPONENT (default export)
// ============================================================
export function UserCard({ user, variant = 'default', onEdit, onDelete, className }: UserCardProps) {
  return (
    <article className={cn('user-card', `user-card--${variant}`, className)}>
      {/* ... */}
      {(onEdit || onDelete) && (
        <UserCardActions user={user} onEdit={onEdit} onDelete={onDelete} />
      )}
    </article>
  );
}

UserCard.displayName = 'UserCard';

// ============================================================
// SKELETON (loading state)
// ============================================================
export function UserCardSkeleton() {
  return (
    <div className="user-card user-card--skeleton" aria-hidden="true">
      <div className="skeleton skeleton-avatar" />
      <div className="skeleton skeleton-text" />
    </div>
  );
}
```

---

## Component Checklist

- [ ] Component has a single, clear responsibility
- [ ] Props interface documented with TypeScript
- [ ] Required props are minimal (fail-safe defaults where possible)
- [ ] Native HTML element props are spread with `...rest` (not swallowed)
- [ ] `className` prop accepted and merged with `cn()`
- [ ] Skeleton/loading state exported alongside component
- [ ] Error boundary wraps any component with side effects
- [ ] No internal state for UI that should be controlled by parent
- [ ] Context not used for state that belongs to local tree
- [ ] No direct DOM manipulation (use React state/refs)
- [ ] Components in appropriate tier (atom/molecule/organism/template)
