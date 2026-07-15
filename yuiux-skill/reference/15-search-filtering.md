# 15 — Search & Filtering

> Search and filtering are how users navigate data. The goal is the shortest path from intent to result.

---

## Search Patterns

### Instant Search (Client-Side)

```tsx
function useSearch<T>(items: T[], searchFn: (item: T, query: string) => boolean) {
  const [query, setQuery] = useState('');
  
  const results = useMemo(() => {
    const q = query.trim().toLowerCase();
    if (!q) return items;
    return items.filter(item => searchFn(item, q));
  }, [items, query, searchFn]);
  
  return { query, setQuery, results };
}

// Usage:
function ProductSearch({ products }: { products: Product[] }) {
  const { query, setQuery, results } = useSearch(products, (p, q) =>
    p.name.toLowerCase().includes(q) ||
    p.category.toLowerCase().includes(q) ||
    p.description?.toLowerCase().includes(q)
  );
  
  return (
    <div>
      <SearchInput
        value={query}
        onChange={setQuery}
        placeholder="Search products…"
        resultCount={results.length}
        totalCount={products.length}
      />
      {results.length === 0 ? (
        <SearchEmptyState query={query} />
      ) : (
        <ProductGrid products={results} />
      )}
    </div>
  );
}
```

### Debounced API Search

```tsx
function useDebounce<T>(value: T, delay = 300): T {
  const [debouncedValue, setDebouncedValue] = useState(value);
  
  useEffect(() => {
    const timer = setTimeout(() => setDebouncedValue(value), delay);
    return () => clearTimeout(timer);
  }, [value, delay]);
  
  return debouncedValue;
}

function useApiSearch<T>(fetchFn: (query: string) => Promise<T[]>) {
  const [query, setQuery] = useState('');
  const [results, setResults] = useState<T[]>([]);
  const [loading, setLoading] = useState(false);
  const debouncedQuery = useDebounce(query, 300);
  
  useEffect(() => {
    if (!debouncedQuery.trim()) {
      setResults([]);
      return;
    }
    
    let cancelled = false;
    setLoading(true);
    
    fetchFn(debouncedQuery)
      .then(data => { if (!cancelled) setResults(data); })
      .catch(console.error)
      .finally(() => { if (!cancelled) setLoading(false); });
    
    return () => { cancelled = true; };
  }, [debouncedQuery, fetchFn]);
  
  return { query, setQuery, results, loading };
}
```

### Search Input Component

```tsx
function SearchInput({
  value,
  onChange,
  onClear,
  placeholder = 'Search…',
  resultCount,
  totalCount,
  loading,
}: SearchInputProps) {
  const inputRef = useRef<HTMLInputElement>(null);
  const searchId = useId();
  const statusId = useId();
  
  return (
    <div className="search-input-wrapper">
      <div className="search-input-container">
        {loading ? (
          <Spinner size="sm" aria-hidden className="search-icon" />
        ) : (
          <SearchIcon className="search-icon" aria-hidden />
        )}
        
        <input
          ref={inputRef}
          id={searchId}
          type="search"
          role="searchbox"
          value={value}
          onChange={e => onChange(e.target.value)}
          placeholder={placeholder}
          autoComplete="off"
          autoCorrect="off"
          spellCheck="false"
          aria-label={placeholder}
          aria-describedby={resultCount !== undefined ? statusId : undefined}
          className="search-input"
        />
        
        {value && (
          <button
            type="button"
            aria-label="Clear search"
            onClick={() => { onChange(''); onClear?.(); inputRef.current?.focus(); }}
            className="search-clear"
          >
            <XIcon aria-hidden />
          </button>
        )}
      </div>
      
      {/* Screen reader result count announcement */}
      {resultCount !== undefined && (
        <p
          id={statusId}
          role="status"
          aria-live="polite"
          aria-atomic="true"
          className="sr-only"
        >
          {resultCount === totalCount
            ? `${totalCount} results`
            : `${resultCount} of ${totalCount} results`
          }
        </p>
      )}
    </div>
  );
}
```

```css
.search-input-wrapper { position: relative; }

.search-input-container {
  display: flex;
  align-items: center;
  gap: 8px;
  padding: 0 12px;
  height: 40px;
  border: 1px solid hsl(var(--border));
  border-radius: var(--radius);
  background: hsl(var(--background));
  transition: border-color 150ms, box-shadow 150ms;
}

.search-input-container:focus-within {
  border-color: hsl(var(--ring));
  box-shadow: 0 0 0 3px hsl(var(--ring) / 0.15);
}

.search-icon { width: 16px; height: 16px; color: hsl(var(--muted-foreground)); flex-shrink: 0; }

.search-input {
  flex: 1;
  border: none;
  outline: none;
  background: transparent;
  font-size: 14px;
  color: hsl(var(--foreground));
  min-width: 0;
}
.search-input::placeholder { color: hsl(var(--muted-foreground)); }
/* Remove browser default clear button */
.search-input::-webkit-search-cancel-button { display: none; }

.search-clear {
  background: none; border: none; cursor: pointer; padding: 0;
  color: hsl(var(--muted-foreground)); display: flex; align-items: center;
  flex-shrink: 0;
}
.search-clear:hover { color: hsl(var(--foreground)); }
```

---

## Combobox / Autocomplete

```tsx
function Combobox<T>({
  options,
  onSelect,
  getLabel,
  getValue,
  placeholder = 'Search…',
  loading,
  onQueryChange,
}: ComboboxProps<T>) {
  const [open, setOpen] = useState(false);
  const [query, setQuery] = useState('');
  const [activeIndex, setActiveIndex] = useState(-1);
  const listRef = useRef<HTMLUListElement>(null);
  const inputRef = useRef<HTMLInputElement>(null);
  const listId = useId();
  
  function handleKeyDown(e: React.KeyboardEvent) {
    if (!open) {
      if (e.key === 'ArrowDown' || e.key === 'ArrowUp') { setOpen(true); return; }
      return;
    }
    
    switch(e.key) {
      case 'ArrowDown':
        e.preventDefault();
        setActiveIndex(i => Math.min(i + 1, options.length - 1));
        break;
      case 'ArrowUp':
        e.preventDefault();
        setActiveIndex(i => Math.max(i - 1, 0));
        break;
      case 'Enter':
        e.preventDefault();
        if (activeIndex >= 0) {
          onSelect(options[activeIndex]);
          setOpen(false);
          setQuery(getLabel(options[activeIndex]));
        }
        break;
      case 'Escape':
        setOpen(false);
        break;
      case 'Home': e.preventDefault(); setActiveIndex(0); break;
      case 'End':  e.preventDefault(); setActiveIndex(options.length - 1); break;
    }
  }
  
  return (
    <div className="combobox" role="combobox" aria-haspopup="listbox" aria-expanded={open}>
      <input
        ref={inputRef}
        type="text"
        value={query}
        onChange={e => {
          setQuery(e.target.value);
          setOpen(true);
          setActiveIndex(-1);
          onQueryChange?.(e.target.value);
        }}
        onFocus={() => setOpen(true)}
        onKeyDown={handleKeyDown}
        placeholder={placeholder}
        aria-autocomplete="list"
        aria-controls={listId}
        aria-activedescendant={activeIndex >= 0 ? `option-${getValue(options[activeIndex])}` : undefined}
        className="combobox-input"
      />
      
      {open && options.length > 0 && (
        <ul
          ref={listRef}
          id={listId}
          role="listbox"
          className="combobox-list"
        >
          {loading ? (
            <li className="combobox-loading">Searching…</li>
          ) : (
            options.map((option, index) => (
              <li
                key={getValue(option)}
                id={`option-${getValue(option)}`}
                role="option"
                aria-selected={index === activeIndex}
                className={`combobox-option ${index === activeIndex ? 'active' : ''}`}
                onMouseDown={e => e.preventDefault()} // prevent blur
                onClick={() => {
                  onSelect(option);
                  setQuery(getLabel(option));
                  setOpen(false);
                }}
              >
                {getLabel(option)}
              </li>
            ))
          )}
        </ul>
      )}
    </div>
  );
}
```

---

## Filter Patterns

### Filter Sidebar

```tsx
interface FilterConfig {
  key: string;
  label: string;
  type: 'checkbox' | 'radio' | 'range' | 'date';
  options?: { value: string; label: string; count?: number }[];
  min?: number;
  max?: number;
}

function FilterSidebar({ filters, value, onChange, onReset }) {
  const activeCount = Object.values(value).flat().filter(Boolean).length;
  
  return (
    <aside className="filter-sidebar" aria-label="Filters">
      <div className="filter-header">
        <h3>Filters</h3>
        {activeCount > 0 && (
          <button type="button" onClick={onReset} className="filter-reset">
            Clear all ({activeCount})
          </button>
        )}
      </div>
      
      {filters.map(filter => (
        <FilterGroup
          key={filter.key}
          filter={filter}
          value={value[filter.key]}
          onChange={val => onChange({ ...value, [filter.key]: val })}
        />
      ))}
    </aside>
  );
}

function FilterGroup({ filter, value, onChange }) {
  const [expanded, setExpanded] = useState(true);
  const panelId = `filter-panel-${filter.key}`;
  
  return (
    <div className="filter-group">
      <button
        type="button"
        aria-expanded={expanded}
        aria-controls={panelId}
        onClick={() => setExpanded(!expanded)}
        className="filter-group-header"
      >
        {filter.label}
        {!expanded && value?.length > 0 && (
          <span className="filter-active-count">{value.length}</span>
        )}
        <ChevronDownIcon aria-hidden className={`filter-chevron ${expanded ? 'expanded' : ''}`} />
      </button>
      
      <div id={panelId} hidden={!expanded} className="filter-group-panel">
        {filter.type === 'checkbox' && (
          <fieldset>
            <legend className="sr-only">{filter.label}</legend>
            {filter.options?.map(option => (
              <label key={option.value} className="filter-option">
                <input
                  type="checkbox"
                  checked={(value ?? []).includes(option.value)}
                  onChange={e => {
                    const arr = value ?? [];
                    onChange(e.target.checked
                      ? [...arr, option.value]
                      : arr.filter(v => v !== option.value)
                    );
                  }}
                />
                <span>{option.label}</span>
                {option.count !== undefined && (
                  <span className="option-count" aria-label={`${option.count} items`}>
                    {option.count}
                  </span>
                )}
              </label>
            ))}
          </fieldset>
        )}
      </div>
    </div>
  );
}
```

### Filter Tags (Active Filter Display)

```tsx
function ActiveFilters({ filters, value, onRemove, onClearAll }) {
  const activeFilters = Object.entries(value).flatMap(([key, val]) => {
    const filter = filters.find(f => f.key === key);
    if (!filter || !val?.length) return [];
    
    return (Array.isArray(val) ? val : [val]).map(v => ({
      filterKey: key,
      value: v,
      label: filter.options?.find(o => o.value === v)?.label ?? v,
      filterLabel: filter.label,
    }));
  });
  
  if (!activeFilters.length) return null;
  
  return (
    <div className="active-filters" role="group" aria-label="Active filters">
      {activeFilters.map(f => (
        <span key={`${f.filterKey}-${f.value}`} className="filter-tag">
          <span className="sr-only">{f.filterLabel}:</span>
          {f.label}
          <button
            type="button"
            onClick={() => onRemove(f.filterKey, f.value)}
            aria-label={`Remove ${f.filterLabel}: ${f.label} filter`}
            className="filter-tag-remove"
          >
            <XIcon aria-hidden />
          </button>
        </span>
      ))}
      <button type="button" onClick={onClearAll} className="filter-clear-all">
        Clear all
      </button>
    </div>
  );
}
```

```css
.active-filters {
  display: flex;
  flex-wrap: wrap;
  gap: 8px;
  align-items: center;
}

.filter-tag {
  display: inline-flex;
  align-items: center;
  gap: 4px;
  padding: 4px 8px;
  background: hsl(var(--primary) / 0.1);
  color: hsl(var(--primary));
  border: 1px solid hsl(var(--primary) / 0.3);
  border-radius: 20px;
  font-size: 13px;
  font-weight: 500;
}

.filter-tag-remove {
  background: none; border: none; cursor: pointer; padding: 0;
  color: inherit; display: flex; align-items: center;
  width: 16px; height: 16px; opacity: 0.7; border-radius: 50%;
}
.filter-tag-remove:hover { opacity: 1; background: hsl(var(--primary) / 0.15); }
```

---

## Sort Controls

```tsx
function SortSelect({ options, value, onChange }) {
  return (
    <div className="sort-control">
      <label htmlFor="sort-select" className="sr-only">Sort by</label>
      <select
        id="sort-select"
        value={value}
        onChange={e => onChange(e.target.value)}
        className="sort-select"
      >
        {options.map(option => (
          <option key={option.value} value={option.value}>
            Sort: {option.label}
          </option>
        ))}
      </select>
      <ChevronDownIcon aria-hidden className="sort-select-icon" />
    </div>
  );
}
```

---

## URL State for Search & Filters

```tsx
// Persist all search/filter state in URL
function useSearchFilterState() {
  const [searchParams, setSearchParams] = useSearchParams();
  
  const query   = searchParams.get('q') ?? '';
  const page    = Number(searchParams.get('page') ?? 1);
  const sort    = searchParams.get('sort') ?? 'created_desc';
  const status  = searchParams.getAll('status');
  
  function setQuery(q: string) {
    setSearchParams(prev => {
      const next = new URLSearchParams(prev);
      if (q) next.set('q', q);
      else next.delete('q');
      next.delete('page'); // reset page on new query
      return next;
    }, { replace: true });
  }
  
  function setFilter(key: string, values: string[]) {
    setSearchParams(prev => {
      const next = new URLSearchParams(prev);
      next.delete(key);
      values.forEach(v => next.append(key, v));
      next.delete('page');
      return next;
    });
  }
  
  function clearAll() {
    setSearchParams({});
  }
  
  return { query, setQuery, page, sort, status, setFilter, clearAll };
}
```

---

## Search Checklist

- [ ] Search input uses `type="search"` (not `type="text"`)
- [ ] Search input has `role="searchbox"` or wraps in `<form role="search">`
- [ ] Debounce API searches by 300ms minimum
- [ ] Cancel previous in-flight request on new query (AbortController or cancelled flag)
- [ ] Result count announced via `aria-live="polite"` on change
- [ ] Empty state for "no results" is different from "no data"
- [ ] Clear button clears input and restores focus to input
- [ ] Combobox implements full keyboard nav (arrows, enter, escape, home, end)
- [ ] Filter state persisted in URL params for shareability
- [ ] Active filters shown as dismissible tags
- [ ] "Clear all" available when any filters are active
- [ ] Filter groups are collapsible with aria-expanded
- [ ] Checkbox filter groups use `<fieldset>` + `<legend>`
