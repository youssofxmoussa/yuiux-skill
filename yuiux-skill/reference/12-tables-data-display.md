# 12 — Tables & Data Display

> Tables are for comparing structured data across categories. If you're not comparing, you're not using a table — you're using a list.

---

## When to Use a Table

```
Use table when:
  ✅ Multiple attributes per item (name, status, date, amount)
  ✅ Users need to compare rows
  ✅ Users need to sort/filter the data
  ✅ Data has clear column-based categories

Use list when:
  ✅ Single attribute per item
  ✅ Cards are more appropriate (visual items)
  ✅ Order matters but not comparison
  ✅ Content is narrative, not categorical
```

---

## Accessible Table Markup

```tsx
function DataTable<T extends Record<string, unknown>>({ 
  data, 
  columns, 
  caption,
  sortable = false,
}: TableProps<T>) {
  const [sort, setSort] = useState<{ key: string; dir: 'asc' | 'desc' } | null>(null);
  
  const sortedData = useMemo(() => {
    if (!sort) return data;
    return [...data].sort((a, b) => {
      const aVal = a[sort.key];
      const bVal = b[sort.key];
      const dir = sort.dir === 'asc' ? 1 : -1;
      if (aVal < bVal) return -dir;
      if (aVal > bVal) return dir;
      return 0;
    });
  }, [data, sort]);
  
  function toggleSort(key: string) {
    setSort(prev => 
      prev?.key === key 
        ? prev.dir === 'asc' ? { key, dir: 'desc' } : null
        : { key, dir: 'asc' }
    );
  }
  
  return (
    <div className="table-wrapper" role="region" aria-label={caption} tabIndex={0}>
      <table>
        {caption && <caption>{caption}</caption>}
        <thead>
          <tr>
            {columns.map(col => (
              <th
                key={col.key}
                scope="col"
                className={col.align ? `text-${col.align}` : ''}
                aria-sort={
                  sort?.key === col.key 
                    ? sort.dir === 'asc' ? 'ascending' : 'descending'
                    : sortable && col.sortable !== false ? 'none' : undefined
                }
              >
                {sortable && col.sortable !== false ? (
                  <button
                    type="button"
                    onClick={() => toggleSort(col.key as string)}
                    className="sort-btn"
                    aria-label={`Sort by ${col.header}`}
                  >
                    {col.header}
                    <SortIcon
                      aria-hidden
                      active={sort?.key === col.key}
                      direction={sort?.dir}
                    />
                  </button>
                ) : col.header}
              </th>
            ))}
          </tr>
        </thead>
        <tbody>
          {sortedData.length === 0 ? (
            <tr>
              <td colSpan={columns.length} className="table-empty">
                No data to display
              </td>
            </tr>
          ) : (
            sortedData.map((row, rowIndex) => (
              <tr key={row.id as string ?? rowIndex}>
                {columns.map(col => (
                  <td key={col.key} className={col.align ? `text-${col.align}` : ''}>
                    {col.render ? col.render(row) : String(row[col.key] ?? '—')}
                  </td>
                ))}
              </tr>
            ))
          )}
        </tbody>
      </table>
    </div>
  );
}
```

```css
/* Table base styles */
.table-wrapper {
  width: 100%;
  overflow-x: auto;
  border: 1px solid hsl(var(--border));
  border-radius: var(--radius);
  outline: none;
}

.table-wrapper:focus-visible {
  outline: 2px solid hsl(var(--ring));
  outline-offset: 2px;
}

table {
  width: 100%;
  border-collapse: collapse;
  font-size: 14px;
}

caption {
  font-size: 16px;
  font-weight: 600;
  text-align: left;
  padding: 0 0 12px;
  color: hsl(var(--foreground));
  caption-side: top;
}

/* Header */
thead tr {
  border-bottom: 1px solid hsl(var(--border));
}

th {
  padding: 10px 16px;
  text-align: left;
  font-size: 12px;
  font-weight: 600;
  text-transform: uppercase;
  letter-spacing: 0.05em;
  color: hsl(var(--muted-foreground));
  white-space: nowrap;
  background: hsl(var(--muted) / 0.3);
}

/* Sort button inside th */
.sort-btn {
  display: inline-flex;
  align-items: center;
  gap: 6px;
  background: none;
  border: none;
  padding: 0;
  cursor: pointer;
  font: inherit;
  font-size: 12px;
  font-weight: 600;
  text-transform: uppercase;
  letter-spacing: 0.05em;
  color: hsl(var(--muted-foreground));
  white-space: nowrap;
}
.sort-btn:hover { color: hsl(var(--foreground)); }

/* Body rows */
tbody tr {
  border-bottom: 1px solid hsl(var(--border));
  transition: background 100ms;
}
tbody tr:last-child { border-bottom: none; }
tbody tr:hover { background: hsl(var(--muted) / 0.5); }

td {
  padding: 12px 16px;
  color: hsl(var(--foreground));
  vertical-align: middle;
}

/* Alignment */
.text-right { text-align: right; }
.text-center { text-align: center; }

/* Numbers — right-aligned for easy comparison */
td.numeric { 
  text-align: right; 
  font-variant-numeric: tabular-nums; 
  font-family: var(--font-mono, 'JetBrains Mono', monospace);
}

/* Empty state */
.table-empty {
  text-align: center;
  padding: 48px 24px;
  color: hsl(var(--muted-foreground));
  font-size: 14px;
}

/* Sticky header */
.table-sticky thead {
  position: sticky;
  top: 0;
  z-index: 1;
}
```

---

## Column Patterns

### Status Badge Column

```tsx
type Status = 'active' | 'inactive' | 'pending' | 'error';

function StatusBadge({ status }: { status: Status }) {
  const config = {
    active:   { label: 'Active',   className: 'badge-success' },
    inactive: { label: 'Inactive', className: 'badge-neutral' },
    pending:  { label: 'Pending',  className: 'badge-warning' },
    error:    { label: 'Error',    className: 'badge-error'   },
  };
  const { label, className } = config[status];
  return (
    <span className={`badge ${className}`}>
      <span className="badge-dot" aria-hidden /> {label}
    </span>
  );
}
```

### Action Column

```tsx
function ActionsCell({ row, onEdit, onDelete }) {
  return (
    <div className="table-actions">
      <button
        type="button"
        onClick={() => onEdit(row)}
        aria-label={`Edit ${row.name}`}
        className="btn btn-ghost btn-icon btn-icon-sm"
      >
        <PencilIcon aria-hidden />
      </button>
      <button
        type="button"
        onClick={() => onDelete(row)}
        aria-label={`Delete ${row.name}`}
        className="btn btn-ghost btn-icon btn-icon-sm text-destructive"
      >
        <TrashIcon aria-hidden />
      </button>
    </div>
  );
}
```

### Expandable Row

```tsx
function ExpandableRow({ row, expandContent }) {
  const [expanded, setExpanded] = useState(false);
  const contentId = `row-${row.id}-content`;
  
  return (
    <>
      <tr>
        <td>
          <button
            type="button"
            aria-expanded={expanded}
            aria-controls={contentId}
            onClick={() => setExpanded(!expanded)}
            className="expand-btn"
          >
            <ChevronRightIcon
              aria-hidden
              className={`expand-icon ${expanded ? 'expanded' : ''}`}
            />
          </button>
        </td>
        {/* other cells */}
      </tr>
      {expanded && (
        <tr id={contentId}>
          <td colSpan={columns.length} className="expand-content">
            {expandContent(row)}
          </td>
        </tr>
      )}
    </>
  );
}
```

---

## Selection (Checkboxes)

```tsx
function SelectableTable({ data, columns }) {
  const [selected, setSelected] = useState<Set<string>>(new Set());
  const allSelected = selected.size === data.length && data.length > 0;
  const someSelected = selected.size > 0 && !allSelected;
  
  function toggleAll() {
    setSelected(allSelected ? new Set() : new Set(data.map(r => r.id)));
  }
  
  function toggleOne(id: string) {
    setSelected(prev => {
      const next = new Set(prev);
      next.has(id) ? next.delete(id) : next.add(id);
      return next;
    });
  }
  
  return (
    <>
      {selected.size > 0 && (
        <div className="bulk-actions" role="toolbar" aria-label="Bulk actions">
          <span>{selected.size} selected</span>
          <button type="button" onClick={deleteSelected}>Delete selected</button>
          <button type="button" onClick={() => setSelected(new Set())}>Deselect all</button>
        </div>
      )}
      
      <table>
        <thead>
          <tr>
            <th>
              <input
                type="checkbox"
                aria-label={allSelected ? 'Deselect all rows' : 'Select all rows'}
                checked={allSelected}
                ref={el => { if (el) el.indeterminate = someSelected; }}
                onChange={toggleAll}
              />
            </th>
            {/* other headers */}
          </tr>
        </thead>
        <tbody>
          {data.map(row => (
            <tr 
              key={row.id}
              aria-selected={selected.has(row.id)}
              className={selected.has(row.id) ? 'selected' : ''}
            >
              <td>
                <input
                  type="checkbox"
                  aria-label={`Select ${row.name}`}
                  checked={selected.has(row.id)}
                  onChange={() => toggleOne(row.id)}
                />
              </td>
              {/* other cells */}
            </tr>
          ))}
        </tbody>
      </table>
    </>
  );
}
```

---

## Pagination

```tsx
function Pagination({ page, pageSize, total, onChange }: PaginationProps) {
  const totalPages = Math.ceil(total / pageSize);
  const from = (page - 1) * pageSize + 1;
  const to = Math.min(page * pageSize, total);
  
  return (
    <nav aria-label="Pagination" className="pagination">
      <p className="pagination-summary" aria-live="polite" aria-atomic>
        Showing {from}–{to} of {total} results
      </p>
      
      <div className="pagination-controls">
        <button
          type="button"
          onClick={() => onChange(page - 1)}
          disabled={page === 1}
          aria-label="Previous page"
          className="btn btn-outline btn-sm"
        >
          ← Previous
        </button>
        
        {/* Page numbers */}
        <div className="pagination-pages" role="group" aria-label="Page navigation">
          {getPageNumbers(page, totalPages).map((p, i) => (
            p === '...' ? (
              <span key={`ellipsis-${i}`} className="pagination-ellipsis">…</span>
            ) : (
              <button
                key={p}
                type="button"
                onClick={() => onChange(p as number)}
                aria-current={p === page ? 'page' : undefined}
                aria-label={`Page ${p}${p === page ? ' (current)' : ''}`}
                className={`pagination-page ${p === page ? 'active' : ''}`}
              >
                {p}
              </button>
            )
          ))}
        </div>
        
        <button
          type="button"
          onClick={() => onChange(page + 1)}
          disabled={page === totalPages}
          aria-label="Next page"
          className="btn btn-outline btn-sm"
        >
          Next →
        </button>
      </div>
    </nav>
  );
}

function getPageNumbers(current: number, total: number): (number | '...')[] {
  if (total <= 7) return Array.from({ length: total }, (_, i) => i + 1);
  
  if (current <= 3) return [1, 2, 3, 4, '...', total];
  if (current >= total - 2) return [1, '...', total-3, total-2, total-1, total];
  return [1, '...', current-1, current, current+1, '...', total];
}
```

---

## Responsive Table Strategies

### 1. Horizontal Scroll (Default)

```css
.table-wrapper {
  overflow-x: auto;
  -webkit-overflow-scrolling: touch;
}
```

### 2. Priority Columns (Hide on Mobile)

```css
/* Mark less important columns */
.col-secondary { } /* always visible */

@media (max-width: 768px) {
  .col-secondary { display: none; }
}
```

### 3. Card Layout on Mobile

```css
@media (max-width: 640px) {
  /* Hide table structure on mobile */
  table, thead, tbody, th, td, tr { display: block; }
  thead { display: none; } /* visually hide — data-label provides context */
  
  tr {
    padding: 16px;
    margin-bottom: 8px;
    border: 1px solid hsl(var(--border));
    border-radius: var(--radius);
    position: relative;
  }
  
  td {
    display: flex;
    gap: 8px;
    padding: 4px 0;
    border: none;
  }
  
  td::before {
    content: attr(data-label);
    font-weight: 600;
    font-size: 12px;
    color: hsl(var(--muted-foreground));
    min-width: 80px;
    flex-shrink: 0;
  }
}
```

```tsx
// Add data-label attributes for mobile card view
{columns.map(col => (
  <td key={col.key} data-label={col.header}>
    {col.render ? col.render(row) : String(row[col.key] ?? '—')}
  </td>
))}
```

---

## Number Formatting

```tsx
// Always format numbers consistently in tables
const formatters = {
  currency: new Intl.NumberFormat('en-US', { style: 'currency', currency: 'USD' }),
  percent: new Intl.NumberFormat('en-US', { style: 'percent', maximumFractionDigits: 1 }),
  integer: new Intl.NumberFormat('en-US', { maximumFractionDigits: 0 }),
  decimal: new Intl.NumberFormat('en-US', { minimumFractionDigits: 2, maximumFractionDigits: 2 }),
  compact: new Intl.NumberFormat('en-US', { notation: 'compact' }),
  fileSize: (bytes: number) => {
    const units = ['B', 'KB', 'MB', 'GB'];
    let i = 0;
    while (bytes >= 1024 && i < units.length - 1) { bytes /= 1024; i++; }
    return `${bytes.toFixed(i > 0 ? 1 : 0)} ${units[i]}`;
  },
};

// Usage:
<td className="numeric">{formatters.currency.format(row.amount)}</td>
<td className="numeric">{formatters.percent.format(row.growthRate)}</td>
```

---

## Stat/KPI Display

```tsx
// Key metric display — not a table, but structured number presentation
interface StatCardProps {
  title: string;
  value: string | number;
  change?: number;   // positive = up, negative = down
  changeLabel?: string;
  description?: string;
  loading?: boolean;
}

function StatCard({ title, value, change, changeLabel, description, loading }: StatCardProps) {
  if (loading) return <StatCardSkeleton />;
  
  const changePositive = change !== undefined && change > 0;
  const changeNegative = change !== undefined && change < 0;
  
  return (
    <div className="stat-card">
      <p className="stat-label">{title}</p>
      <p className="stat-value">{value}</p>
      {change !== undefined && (
        <p className={`stat-change ${changePositive ? 'up' : changeNegative ? 'down' : 'neutral'}`}>
          <span aria-hidden>{changePositive ? '↑' : changeNegative ? '↓' : '→'}</span>
          <span className="sr-only">{changePositive ? 'Up' : changeNegative ? 'Down' : 'No change'}</span>
          {' '}{Math.abs(change)}%
          {changeLabel && <span className="stat-change-label"> {changeLabel}</span>}
        </p>
      )}
      {description && <p className="stat-description">{description}</p>}
    </div>
  );
}
```

```css
.stat-card {
  padding: 24px;
  border: 1px solid hsl(var(--border));
  border-radius: var(--radius);
  background: hsl(var(--card));
}

.stat-label {
  font-size: 13px;
  font-weight: 500;
  color: hsl(var(--muted-foreground));
  margin: 0 0 8px;
}

.stat-value {
  font-size: 30px;
  font-weight: 700;
  color: hsl(var(--foreground));
  line-height: 1;
  margin: 0 0 8px;
  font-variant-numeric: tabular-nums;
}

.stat-change {
  font-size: 13px;
  font-weight: 500;
  margin: 0;
  display: flex;
  align-items: center;
  gap: 4px;
}
.stat-change.up   { color: hsl(142 76% 36%); } /* green-600 */
.stat-change.down { color: hsl(0 84% 60%); }   /* red-500 */
.stat-change.neutral { color: hsl(var(--muted-foreground)); }
```

---

## Data Table Checklist

- [ ] `<caption>` on every `<table>` describing its content
- [ ] `scope="col"` on header cells in `<thead>`
- [ ] `scope="row"` on row headers if applicable
- [ ] Sortable columns have `aria-sort` attribute
- [ ] Sort buttons inside `<th>` with `aria-label`
- [ ] Bulk action toolbar has `role="toolbar"` + `aria-label`
- [ ] Indeterminate checkbox state programmatically set (`.indeterminate = true`)
- [ ] Row selection reflected in `aria-selected` on `<tr>`
- [ ] Pagination nav has `aria-label="Pagination"`
- [ ] "Showing N–M of total" count is `aria-live="polite"`
- [ ] Current page button has `aria-current="page"`
- [ ] Numbers right-aligned with `font-variant-numeric: tabular-nums`
- [ ] Currency/numbers consistently formatted with `Intl.NumberFormat`
- [ ] Table scrolls horizontally on mobile (overflow-x: auto)
- [ ] No fixed heights on table containers (allow content to grow)
- [ ] Empty state in table body spans all columns
- [ ] Status cells always color + text (never color alone)
