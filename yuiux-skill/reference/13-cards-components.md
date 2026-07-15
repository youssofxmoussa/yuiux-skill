# 13 — Cards & Component Patterns

> Cards are the atomic unit of modern UI. Every card is a container of meaning — not just a visual box.

---

## Card Anatomy

```
┌─────────────────────────────────────────┐
│  [Media / Image]                        │ ← optional
├─────────────────────────────────────────┤
│  [Category / Label]         [Badge]     │ ← optional header meta
│  Title                                  │ ← required
│  Subtitle / Description                 │ ← optional
├─────────────────────────────────────────┤
│  [Content area]                         │ ← optional
├─────────────────────────────────────────┤
│  [Avatar + Meta]         [Actions]      │ ← optional footer
└─────────────────────────────────────────┘
```

---

## Base Card Component

```tsx
interface CardProps {
  children: React.ReactNode;
  as?: 'article' | 'div' | 'li' | 'section';
  href?: string;        // if provided, card is a link
  onClick?: () => void; // if provided, card is interactive
  className?: string;
  padding?: 'none' | 'sm' | 'md' | 'lg';
}

function Card({ children, as: Tag = 'div', href, onClick, className, padding = 'md' }: CardProps) {
  // Semantic: if href, render as <a>; if onClick, add role/tabindex
  if (href) {
    return (
      <a href={href} className={`card card--link card--pad-${padding} ${className ?? ''}`}>
        {children}
      </a>
    );
  }
  
  if (onClick) {
    return (
      <Tag
        role="button"
        tabIndex={0}
        onClick={onClick}
        onKeyDown={e => { if (e.key === 'Enter' || e.key === ' ') onClick(); }}
        className={`card card--interactive card--pad-${padding} ${className ?? ''}`}
      >
        {children}
      </Tag>
    );
  }
  
  return (
    <Tag className={`card card--pad-${padding} ${className ?? ''}`}>
      {children}
    </Tag>
  );
}

// Subcomponents
Card.Header  = ({ children }) => <div className="card-header">{children}</div>;
Card.Body    = ({ children }) => <div className="card-body">{children}</div>;
Card.Footer  = ({ children }) => <div className="card-footer">{children}</div>;
Card.Title   = ({ children, as: Tag = 'h3' }) => <Tag className="card-title">{children}</Tag>;
Card.Meta    = ({ children }) => <p className="card-meta">{children}</p>;
Card.Image   = ({ src, alt, aspectRatio = '16/9' }) => (
  <div className="card-image" style={{ aspectRatio }}>
    <img src={src} alt={alt} loading="lazy" />
  </div>
);
```

```css
/* Base card */
.card {
  background: hsl(var(--card));
  border: 1px solid hsl(var(--border));
  border-radius: var(--radius);
  overflow: hidden;
  display: flex;
  flex-direction: column;
  position: relative;
}

/* Padding variants */
.card--pad-none { --card-pad: 0; }
.card--pad-sm   { --card-pad: 12px; }
.card--pad-md   { --card-pad: 24px; }
.card--pad-lg   { --card-pad: 32px; }

.card-header, .card-body, .card-footer { padding: var(--card-pad, 24px); }
.card-header { padding-bottom: 0; }
.card-footer { padding-top: 0; }

/* Interactive card */
.card--interactive {
  cursor: pointer;
  transition: box-shadow 150ms ease, transform 150ms ease;
}
.card--interactive:hover {
  box-shadow: var(--shadow-md);
  transform: translateY(-2px);
}

/* Link card — no nested interactive elements */
.card--link {
  text-decoration: none;
  cursor: pointer;
  display: block;
  color: inherit;
  transition: box-shadow 150ms ease, transform 150ms ease;
}
.card--link:hover {
  box-shadow: var(--shadow-md);
  transform: translateY(-2px);
}
.card--link:focus-visible {
  outline: 2px solid hsl(var(--ring));
  outline-offset: 2px;
}

/* Card image */
.card-image {
  overflow: hidden;
  flex-shrink: 0;
}
.card-image img {
  width: 100%;
  height: 100%;
  object-fit: cover;
  display: block;
  transition: transform 300ms ease;
}
.card--interactive:hover .card-image img,
.card--link:hover .card-image img {
  transform: scale(1.03);
}

/* Typography inside card */
.card-title {
  font-size: 16px;
  font-weight: 600;
  color: hsl(var(--foreground));
  margin: 0 0 4px;
  line-height: 1.3;
}

.card-meta {
  font-size: 13px;
  color: hsl(var(--muted-foreground));
  margin: 0;
}

/* Card footer */
.card-footer {
  display: flex;
  align-items: center;
  justify-content: space-between;
  margin-top: auto;
  border-top: 1px solid hsl(var(--border));
}
```

---

## Card Variants

### Profile / User Card

```tsx
function UserCard({ user }: { user: User }) {
  return (
    <article className="card">
      <div className="user-card-content">
        <img
          src={user.avatarUrl}
          alt=""
          className="user-card-avatar"
          loading="lazy"
        />
        <div className="user-card-info">
          <h3 className="user-card-name">{user.displayName}</h3>
          <p className="user-card-role">{user.role}</p>
          <p className="user-card-email">{user.email}</p>
        </div>
      </div>
    </article>
  );
}
```

### Product Card

```tsx
function ProductCard({ product }: { product: Product }) {
  return (
    <article className="card">
      <Card.Image src={product.imageUrl} alt={product.name} />
      <Card.Body>
        <p className="product-category">{product.category}</p>
        <h3 className="card-title">{product.name}</h3>
        <div className="product-price-row">
          <span className="product-price">{formatCurrency(product.price)}</span>
          {product.originalPrice && (
            <del className="product-original-price">{formatCurrency(product.originalPrice)}</del>
          )}
        </div>
        <div className="product-rating">
          <StarRating rating={product.rating} />
          <span>({product.reviewCount})</span>
        </div>
      </Card.Body>
      <Card.Footer>
        <button className="btn btn-primary w-full">Add to cart</button>
      </Card.Footer>
    </article>
  );
}
```

### Metric / Stat Card

```tsx
function MetricCard({ metric }: { metric: Metric }) {
  return (
    <div className="card card--pad-md stat-card">
      <div className="metric-header">
        <p className="metric-label">{metric.label}</p>
        <metric.icon className="metric-icon" aria-hidden />
      </div>
      <p className="metric-value">{metric.formattedValue}</p>
      {metric.change !== undefined && (
        <p className={`metric-change ${metric.change >= 0 ? 'positive' : 'negative'}`}>
          <span aria-hidden>{metric.change >= 0 ? '↑' : '↓'}</span>
          <span className="sr-only">{metric.change >= 0 ? 'Up' : 'Down'}</span>
          {' '}{Math.abs(metric.change)}% vs last {metric.period}
        </p>
      )}
    </div>
  );
}
```

### Notification / Alert Card

```tsx
type AlertVariant = 'info' | 'success' | 'warning' | 'error';

function AlertCard({ variant = 'info', title, description, onDismiss }: AlertCardProps) {
  const config = {
    info:    { icon: InfoIcon,    bg: 'bg-blue-50',   text: 'text-blue-800',   border: 'border-blue-200'  },
    success: { icon: CheckCircle, bg: 'bg-green-50',  text: 'text-green-800',  border: 'border-green-200' },
    warning: { icon: AlertTriangle, bg: 'bg-amber-50', text: 'text-amber-800', border: 'border-amber-200' },
    error:   { icon: XCircle,    bg: 'bg-red-50',    text: 'text-red-800',    border: 'border-red-200'   },
  };
  
  const { icon: Icon, bg, text, border } = config[variant];
  
  return (
    <div
      role={variant === 'error' ? 'alert' : 'status'}
      className={`alert-card ${bg} ${border} ${text}`}
    >
      <Icon className="alert-icon" aria-hidden />
      <div className="alert-content">
        {title && <p className="alert-title">{title}</p>}
        {description && <p className="alert-desc">{description}</p>}
      </div>
      {onDismiss && (
        <button
          type="button"
          onClick={onDismiss}
          aria-label="Dismiss alert"
          className="alert-dismiss"
        >
          <XIcon aria-hidden />
        </button>
      )}
    </div>
  );
}
```

---

## Card Grid Layouts

```css
/* Auto-responsive card grid */
.card-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(280px, 1fr));
  gap: 24px;
}

/* Fixed column grids */
.card-grid-2 { display: grid; grid-template-columns: repeat(2, 1fr); gap: 20px; }
.card-grid-3 { display: grid; grid-template-columns: repeat(3, 1fr); gap: 20px; }
.card-grid-4 { display: grid; grid-template-columns: repeat(4, 1fr); gap: 16px; }

/* Responsive fixed grids */
.card-grid-responsive {
  display: grid;
  grid-template-columns: 1fr;
  gap: 16px;
}
@media (min-width: 640px)  { .card-grid-responsive { grid-template-columns: repeat(2, 1fr); gap: 20px; } }
@media (min-width: 1024px) { .card-grid-responsive { grid-template-columns: repeat(3, 1fr); gap: 24px; } }

/* Equal height cards in a row */
.card-grid > .card { height: 100%; }
```

---

## Badge Component

```tsx
type BadgeVariant = 'default' | 'secondary' | 'success' | 'warning' | 'error' | 'outline';

function Badge({ variant = 'default', children, dot }: BadgeProps) {
  return (
    <span className={`badge badge--${variant}`}>
      {dot && <span className="badge-dot" aria-hidden />}
      {children}
    </span>
  );
}
```

```css
.badge {
  display: inline-flex;
  align-items: center;
  gap: 4px;
  padding: 2px 8px;
  border-radius: 20px;
  font-size: 11px;
  font-weight: 600;
  white-space: nowrap;
  line-height: 1.5;
}

.badge--default   { background: hsl(var(--primary)); color: hsl(var(--primary-foreground)); }
.badge--secondary { background: hsl(var(--secondary)); color: hsl(var(--secondary-foreground)); }
.badge--success   { background: #dcfce7; color: #166534; }
.badge--warning   { background: #fef9c3; color: #854d0e; }
.badge--error     { background: #fee2e2; color: #991b1b; }
.badge--outline   { background: transparent; border: 1px solid hsl(var(--border)); color: hsl(var(--foreground)); }

.badge-dot {
  width: 6px;
  height: 6px;
  border-radius: 50%;
  background: currentColor;
  flex-shrink: 0;
}
```

---

## Avatar Component

```tsx
interface AvatarProps {
  src?: string;
  name: string;
  size?: 'xs' | 'sm' | 'md' | 'lg' | 'xl';
}

function Avatar({ src, name, size = 'md' }: AvatarProps) {
  const [imgError, setImgError] = useState(false);
  const initials = getInitials(name);
  const bg = getAvatarColor(name);
  
  return (
    <div
      className={`avatar avatar--${size}`}
      style={imgError || !src ? { background: bg } : undefined}
      aria-label={name}
    >
      {src && !imgError ? (
        <img
          src={src}
          alt=""
          onError={() => setImgError(true)}
          loading="lazy"
        />
      ) : (
        <span aria-hidden>{initials}</span>
      )}
    </div>
  );
}

function getInitials(name: string): string {
  return name
    .split(' ')
    .slice(0, 2)
    .map(w => w[0]?.toUpperCase() ?? '')
    .join('');
}

function getAvatarColor(name: string): string {
  const colors = [
    '#7c3aed', '#2563eb', '#059669', '#d97706', '#dc2626',
    '#7c3aed', '#0891b2', '#65a30d', '#c2410c', '#4f46e5',
  ];
  let hash = 0;
  for (let i = 0; i < name.length; i++) hash = name.charCodeAt(i) + ((hash << 5) - hash);
  return colors[Math.abs(hash) % colors.length];
}
```

```css
.avatar {
  border-radius: 50%;
  overflow: hidden;
  display: flex;
  align-items: center;
  justify-content: center;
  flex-shrink: 0;
  background: hsl(var(--muted));
  color: white;
  font-weight: 600;
  user-select: none;
}

.avatar img { width: 100%; height: 100%; object-fit: cover; }

.avatar--xs { width: 24px; height: 24px; font-size: 10px; }
.avatar--sm { width: 32px; height: 32px; font-size: 12px; }
.avatar--md { width: 40px; height: 40px; font-size: 14px; }
.avatar--lg { width: 56px; height: 56px; font-size: 18px; }
.avatar--xl { width: 80px; height: 80px; font-size: 24px; }
```

---

## Component Checklist

- [ ] Cards that link use `<a href>` (not `<div onClick>`)
- [ ] Interactive cards without hrefs have `role="button"` + `tabIndex={0}` + keyboard handler
- [ ] Card images have `alt` attribute (empty for purely decorative product images — the name is the label)
- [ ] Card grids use CSS Grid with auto-fill for responsive behavior
- [ ] Cards in a row have equal heights (use `height: 100%` on card in grid)
- [ ] Status badges always include text (never color alone)
- [ ] Avatars have `aria-label` with the person's name
- [ ] Avatar image has `alt=""` (decorative, name is aria-label on container)
- [ ] Avatars gracefully fall back to initials when image fails/missing
- [ ] Alert cards use `role="alert"` for errors, `role="status"` for info
- [ ] Dismiss buttons have `aria-label="Dismiss alert"`
