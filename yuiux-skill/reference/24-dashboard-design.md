# 24 — Dashboard Design

> A dashboard is not a data dump. It answers the question: "What do I need to know and do right now?"

---

## Dashboard Design Principles

### 1. Lead with the Most Important Metric

The first thing a user sees should answer "How are we doing?" Everything else is detail.

```
Not: alphabetical order of metrics
Not: metrics by technical complexity
Yes: metrics ordered by decision impact
```

### 2. Progressive Disclosure

```
Level 1: Summary KPIs (top-level view, always visible)
Level 2: Trend charts (context for the KPIs)
Level 3: Breakdown tables (details on demand)
Level 4: Individual records (drill-down)
```

### 3. Action-Orientation

Every metric should lead to an action or decision. If a number on a dashboard never changes anyone's behavior, remove it.

---

## Dashboard Layout Pattern

```tsx
function Dashboard() {
  const { data, loading } = useDashboardData();
  
  return (
    <main className="dashboard">
      {/* Level 1: Summary KPIs */}
      <section aria-labelledby="metrics-heading">
        <h2 id="metrics-heading" className="sr-only">Key metrics</h2>
        <div className="kpi-grid">
          {loading
            ? Array.from({ length: 4 }, (_, i) => <StatCardSkeleton key={i} />)
            : data?.metrics.map(metric => (
                <StatCard key={metric.id} metric={metric} />
              ))
          }
        </div>
      </section>
      
      {/* Level 2: Trend Charts */}
      <section aria-labelledby="trends-heading" className="dashboard-charts">
        <h2 id="trends-heading">Trends</h2>
        <div className="chart-grid">
          <RevenueChart data={data?.revenueByDay} loading={loading} />
          <UserGrowthChart data={data?.usersByWeek} loading={loading} />
        </div>
      </section>
      
      {/* Level 3: Detail Tables */}
      <section aria-labelledby="detail-heading">
        <h2 id="detail-heading">Recent activity</h2>
        <Tabs
          tabs={[
            { id: 'transactions', label: 'Transactions' },
            { id: 'users', label: 'New users' },
            { id: 'errors', label: 'Errors' },
          ]}
          defaultTab="transactions"
        >
          {activeTab => (
            <>
              {activeTab === 'transactions' && <TransactionsTable />}
              {activeTab === 'users' && <NewUsersTable />}
              {activeTab === 'errors' && <ErrorsTable />}
            </>
          )}
        </Tabs>
      </section>
    </main>
  );
}
```

---

## KPI Card Design

```tsx
interface KpiMetric {
  id: string;
  label: string;
  value: string | number;
  formatted: string;
  previousPeriodFormatted: string;
  change: number;  // percentage
  changePeriod: string;  // "vs last 30 days"
  trend: 'positive' | 'negative' | 'neutral';
  sparkline?: number[];
  icon?: React.ComponentType<{ className?: string }>;
  link?: string;   // drill-down link
}

function StatCard({ metric }: { metric: KpiMetric }) {
  const Icon = metric.icon;
  const isGood = metric.trend === 'positive';
  const isBad  = metric.trend === 'negative';
  
  return (
    <article className="stat-card" aria-label={metric.label}>
      <div className="stat-card-header">
        <p className="stat-label">{metric.label}</p>
        {Icon && (
          <div className="stat-icon-wrapper" aria-hidden>
            <Icon className="stat-icon" />
          </div>
        )}
      </div>
      
      <div className="stat-value-row">
        <p
          className="stat-value"
          aria-label={`${metric.label}: ${metric.formatted}`}
        >
          {metric.formatted}
        </p>
        {metric.sparkline && (
          <Sparkline
            data={metric.sparkline}
            trend={metric.trend === 'positive' ? 'up' : metric.trend === 'negative' ? 'down' : 'flat'}
          />
        )}
      </div>
      
      <div className="stat-footer">
        {metric.change !== 0 && (
          <span
            className={`stat-change ${isGood ? 'positive' : isBad ? 'negative' : 'neutral'}`}
            aria-label={`${metric.change > 0 ? 'Up' : 'Down'} ${Math.abs(metric.change)}% ${metric.changePeriod}`}
          >
            <span aria-hidden>{metric.change > 0 ? '↑' : '↓'}</span>
            {' '}{Math.abs(metric.change)}%
          </span>
        )}
        <span className="stat-period">{metric.changePeriod}</span>
        
        {metric.link && (
          <a href={metric.link} className="stat-link">
            View details <ArrowRightIcon aria-hidden />
          </a>
        )}
      </div>
    </article>
  );
}
```

```css
.kpi-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(220px, 1fr));
  gap: 16px;
}

.stat-card {
  padding: 24px;
  border: 1px solid hsl(var(--border));
  border-radius: var(--radius);
  background: hsl(var(--card));
  display: flex;
  flex-direction: column;
  gap: 8px;
  transition: box-shadow 150ms;
}

.stat-card:hover { box-shadow: var(--shadow-md); }

.stat-card-header {
  display: flex;
  align-items: flex-start;
  justify-content: space-between;
}

.stat-label {
  font-size: 13px;
  font-weight: 500;
  color: hsl(var(--muted-foreground));
  margin: 0;
}

.stat-icon-wrapper {
  width: 36px;
  height: 36px;
  border-radius: var(--radius);
  background: hsl(var(--primary) / 0.1);
  display: flex;
  align-items: center;
  justify-content: center;
}

.stat-icon { width: 18px; height: 18px; color: hsl(var(--primary)); }

.stat-value-row {
  display: flex;
  align-items: flex-end;
  justify-content: space-between;
  gap: 12px;
}

.stat-value {
  font-size: 28px;
  font-weight: 700;
  color: hsl(var(--foreground));
  line-height: 1;
  margin: 0;
  font-variant-numeric: tabular-nums;
}

.stat-footer {
  display: flex;
  align-items: center;
  gap: 6px;
  flex-wrap: wrap;
}

.stat-change {
  font-size: 12px;
  font-weight: 600;
  display: inline-flex;
  align-items: center;
  gap: 2px;
}
.stat-change.positive { color: #16a34a; }
.stat-change.negative { color: #dc2626; }
.stat-change.neutral  { color: hsl(var(--muted-foreground)); }

.stat-period { font-size: 12px; color: hsl(var(--muted-foreground)); }

.stat-link {
  font-size: 12px;
  color: hsl(var(--primary));
  text-decoration: none;
  margin-left: auto;
  display: inline-flex;
  align-items: center;
  gap: 4px;
}
.stat-link:hover { text-decoration: underline; }
```

---

## Date Range Filter

```tsx
function DateRangeSelector({ value, onChange }) {
  const presets = [
    { label: 'Today',       value: 'today',      days: 0 },
    { label: 'Yesterday',   value: 'yesterday',   days: 1 },
    { label: 'Last 7 days', value: '7d',          days: 7 },
    { label: 'Last 30 days',value: '30d',         days: 30 },
    { label: 'Last 90 days',value: '90d',         days: 90 },
    { label: 'Last 12 months',value: '12m',       days: 365 },
    { label: 'Custom',      value: 'custom',      days: null },
  ];
  
  return (
    <div className="date-range-selector">
      <select
        value={value.preset}
        onChange={e => onChange({ preset: e.target.value })}
        aria-label="Select date range"
        className="date-preset-select"
      >
        {presets.map(p => (
          <option key={p.value} value={p.value}>{p.label}</option>
        ))}
      </select>
      
      {value.preset === 'custom' && (
        <div className="date-custom-range">
          <label htmlFor="date-from" className="sr-only">From date</label>
          <input
            type="date"
            id="date-from"
            value={value.from}
            max={value.to}
            onChange={e => onChange({ ...value, from: e.target.value })}
          />
          <span aria-hidden>→</span>
          <label htmlFor="date-to" className="sr-only">To date</label>
          <input
            type="date"
            id="date-to"
            value={value.to}
            min={value.from}
            max={new Date().toISOString().split('T')[0]}
            onChange={e => onChange({ ...value, to: e.target.value })}
          />
        </div>
      )}
    </div>
  );
}
```

---

## Dashboard Real-Time Updates

```tsx
// Auto-refresh dashboard data
function useDashboardAutoRefresh(intervalMs = 60_000) {
  const [data, setData] = useState(null);
  const [lastRefreshed, setLastRefreshed] = useState<Date | null>(null);
  const [refreshing, setRefreshing] = useState(false);
  
  const refresh = useCallback(async () => {
    setRefreshing(true);
    try {
      const fresh = await fetchDashboardData();
      setData(fresh);
      setLastRefreshed(new Date());
    } finally {
      setRefreshing(false);
    }
  }, []);
  
  useEffect(() => {
    refresh();
    const interval = setInterval(refresh, intervalMs);
    return () => clearInterval(interval);
  }, [refresh, intervalMs]);
  
  return { data, lastRefreshed, refreshing, refresh };
}

// Display refresh status
function RefreshIndicator({ lastRefreshed, refreshing, onRefresh }) {
  return (
    <div className="refresh-indicator" aria-live="polite" aria-atomic>
      {refreshing ? (
        <span><Spinner size="sm" aria-hidden /> Refreshing…</span>
      ) : lastRefreshed ? (
        <span>
          Updated {formatRelativeTime(lastRefreshed)}
          <button
            type="button"
            onClick={onRefresh}
            aria-label="Refresh dashboard data"
            className="refresh-btn"
          >
            <RefreshIcon aria-hidden />
          </button>
        </span>
      ) : null}
    </div>
  );
}
```

---

## Dashboard Layout Skeleton

```tsx
function DashboardSkeleton() {
  return (
    <div className="dashboard" aria-busy="true" aria-label="Loading dashboard">
      {/* KPI row */}
      <div className="kpi-grid">
        {Array.from({ length: 4 }, (_, i) => (
          <div key={i} className="stat-card">
            <div className="skeleton skeleton-text" style={{ width: '60%' }} />
            <div className="skeleton skeleton-text" style={{ width: '40%', height: '2em', marginTop: 12 }} />
            <div className="skeleton skeleton-text" style={{ width: '80%', marginTop: 8 }} />
          </div>
        ))}
      </div>
      
      {/* Charts */}
      <div className="chart-grid">
        <div className="skeleton skeleton-image" style={{ height: 260 }} />
        <div className="skeleton skeleton-image" style={{ height: 260 }} />
      </div>
    </div>
  );
}
```

---

## Dashboard Anti-Patterns

| Anti-Pattern | Problem | Fix |
|-------------|---------|-----|
| All metrics on one screen | Cognitive overload | Prioritize — show max 8 KPIs at top level |
| No comparison period | Numbers meaningless without context | Always show vs previous period |
| Static dashboard | Data goes stale | Auto-refresh, show last updated time |
| Metrics without drill-down | Users can't investigate anomalies | Link each metric to its detail view |
| Different date ranges on same dashboard | Confusing comparisons | Single date range selector at top |
| Color as only indicator for trend | Colorblind users miss it | Color + icon (↑/↓) + text |
| No skeleton loading | Flash of content | Full skeleton for all data states |
| Too many chart types | Visual inconsistency | Pick 2–3 chart types and stick with them |

---

## Dashboard Checklist

- [ ] KPIs ordered by decision impact, not alphabetically
- [ ] Every metric has a comparison period (vs last 30 days, etc.)
- [ ] Date range selector at dashboard top level
- [ ] Auto-refresh with "last updated" indicator
- [ ] Drill-down links on every metric card
- [ ] Change indicators use color + icon + text (not color alone)
- [ ] Full skeleton loading state for all sections
- [ ] Section headings for screen readers (even if visually hidden)
- [ ] Charts have accessible titles and data table alternatives
- [ ] Empty state when no data in selected period
- [ ] Dashboard state (date range, tab) persisted in URL
- [ ] Print-friendly layout option for reports
