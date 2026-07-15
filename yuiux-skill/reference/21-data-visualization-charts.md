# 21 — Data Visualization & Charts

> A chart is only as good as the insight it communicates. Before coding anything, ask: "What decision should this chart help the user make?"

---

## Chart Selection Guide

| Data Type | Best Chart | Avoid |
|-----------|-----------|-------|
| Trend over time | Line chart | Pie chart |
| Category comparison | Bar chart (vertical) | 3D chart |
| Part-to-whole | Pie/donut (≤5 slices) | Pie with >7 slices |
| Distribution | Histogram | Pie chart |
| Correlation | Scatter plot | Bar chart |
| Ranking | Horizontal bar chart | Pie chart |
| Geographic | Map | Bar chart |
| Flow/funnel | Funnel chart | Line chart |
| Progress | Progress bar / gauge | Scatter plot |
| Single number | Big number + sparkline | Full chart |

**The Universal Rule:** If you can communicate the same insight with text + a number, don't use a chart.

---

## Chart Accessibility

### Every chart needs:
1. **Title** — what the chart shows
2. **Data labels** or **legend** — what each series/color means
3. **Axis labels with units** — what the scales mean
4. **Alt text** or **data table** for screen readers
5. **Color + pattern** for colorblind users (never color alone)

```tsx
// Accessible chart wrapper
function AccessibleChart({ 
  title, 
  description,
  children,
  data,
  columns,
}: AccessibleChartProps) {
  const tableId = useId();
  const [showTable, setShowTable] = useState(false);
  
  return (
    <figure aria-labelledby={`${tableId}-title`} aria-describedby={`${tableId}-desc`}>
      <figcaption>
        <h3 id={`${tableId}-title`}>{title}</h3>
        {description && <p id={`${tableId}-desc`}>{description}</p>}
      </figcaption>
      
      {/* Visual chart — hidden from screen readers, data table provides the info */}
      <div aria-hidden="true">{children}</div>
      
      {/* Screen reader accessible data */}
      <button 
        type="button"
        onClick={() => setShowTable(!showTable)}
        className="btn btn-ghost btn-sm"
        aria-expanded={showTable}
        aria-controls={tableId}
      >
        {showTable ? 'Hide' : 'View'} data table
      </button>
      
      <div id={tableId} hidden={!showTable}>
        <DataTable data={data} columns={columns} caption={title} />
      </div>
    </figure>
  );
}
```

### Colorblind-Safe Palettes

```tsx
// Never use red+green as the only differentiator
// ❌ Colorblind ambiguous
const BAD_COLORS = ['#22c55e', '#ef4444']; // green and red look similar

// ✅ Colorblind-safe — IBM Carbon palette for data viz
const ACCESSIBLE_CHART_COLORS = [
  '#648FFF', // blue
  '#785EF0', // purple
  '#DC267F', // magenta
  '#FE6100', // orange
  '#FFB000', // yellow
];

// ✅ OR: use pattern/texture in addition to color
// Add dashes, dots, or hatching to distinguish data series
```

---

## Recharts Patterns (React)

### Line Chart

```tsx
import {
  LineChart, Line, XAxis, YAxis, CartesianGrid,
  Tooltip, Legend, ResponsiveContainer,
} from 'recharts';

interface ChartDataPoint { date: string; revenue: number; users: number; }

function RevenueChart({ data }: { data: ChartDataPoint[] }) {
  return (
    <AccessibleChart
      title="Revenue and users over time"
      description="Monthly revenue ($) and active users from January to December 2026"
      data={data}
      columns={[
        { key: 'date',    header: 'Month' },
        { key: 'revenue', header: 'Revenue ($)' },
        { key: 'users',   header: 'Active users' },
      ]}
    >
      <ResponsiveContainer width="100%" height={300}>
        <LineChart
          data={data}
          margin={{ top: 8, right: 16, left: 0, bottom: 0 }}
          role="img"
          aria-label="Revenue and users chart — see data table for details"
        >
          <CartesianGrid strokeDasharray="3 3" stroke="hsl(var(--border))" />
          <XAxis
            dataKey="date"
            tick={{ fill: 'hsl(var(--muted-foreground))', fontSize: 12 }}
            tickLine={false}
            axisLine={false}
          />
          <YAxis
            tick={{ fill: 'hsl(var(--muted-foreground))', fontSize: 12 }}
            tickLine={false}
            axisLine={false}
            tickFormatter={v => `$${(v / 1000).toFixed(0)}k`}
          />
          <Tooltip
            contentStyle={{
              background: 'hsl(var(--card))',
              border: '1px solid hsl(var(--border))',
              borderRadius: 'var(--radius)',
              color: 'hsl(var(--foreground))',
              fontSize: 13,
            }}
            formatter={(value: number, name: string) => [
              name === 'revenue' ? `$${value.toLocaleString()}` : value.toLocaleString(),
              name === 'revenue' ? 'Revenue' : 'Users',
            ]}
            labelStyle={{ color: 'hsl(var(--foreground))', fontWeight: 600, marginBottom: 4 }}
          />
          <Legend
            iconType="circle"
            wrapperStyle={{ fontSize: 13, color: 'hsl(var(--muted-foreground))' }}
          />
          <Line
            type="monotone"
            dataKey="revenue"
            name="Revenue"
            stroke="#648FFF"
            strokeWidth={2}
            dot={false}
            activeDot={{ r: 4, strokeWidth: 0 }}
          />
          <Line
            type="monotone"
            dataKey="users"
            name="Users"
            stroke="#DC267F"
            strokeWidth={2}
            dot={false}
            activeDot={{ r: 4, strokeWidth: 0 }}
            strokeDasharray="5 5" /* pattern for colorblind */
          />
        </LineChart>
      </ResponsiveContainer>
    </AccessibleChart>
  );
}
```

### Bar Chart

```tsx
import { BarChart, Bar, Cell } from 'recharts';

function CategoryBarChart({ data }: { data: { category: string; value: number }[] }) {
  const max = Math.max(...data.map(d => d.value));
  
  return (
    <AccessibleChart title="Sales by category" data={data} columns={[
      { key: 'category', header: 'Category' },
      { key: 'value', header: 'Sales' },
    ]}>
      <ResponsiveContainer width="100%" height={280}>
        <BarChart data={data} layout="vertical" margin={{ left: 80 }}>
          <XAxis type="number" hide />
          <YAxis 
            type="category"
            dataKey="category"
            tick={{ fontSize: 13, fill: 'hsl(var(--foreground))' }}
            tickLine={false}
            axisLine={false}
            width={80}
          />
          <Tooltip
            contentStyle={{ background: 'hsl(var(--card))', border: '1px solid hsl(var(--border))' }}
            formatter={(v: number) => [v.toLocaleString(), 'Sales']}
          />
          <Bar dataKey="value" radius={[0, 4, 4, 0]}>
            {data.map((entry, index) => (
              <Cell
                key={index}
                fill={entry.value === max ? 'hsl(var(--primary))' : 'hsl(var(--primary) / 0.4)'}
              />
            ))}
          </Bar>
        </BarChart>
      </ResponsiveContainer>
    </AccessibleChart>
  );
}
```

### Donut / Pie Chart

```tsx
import { PieChart, Pie, Cell, Tooltip } from 'recharts';

// Only use pie/donut for ≤5 slices. For more, use bar chart.
function DonutChart({ data }: { data: { label: string; value: number; color: string }[] }) {
  const total = data.reduce((sum, d) => sum + d.value, 0);
  
  return (
    <AccessibleChart
      title="Traffic sources"
      data={data.map(d => ({ ...d, percentage: `${((d.value / total) * 100).toFixed(1)}%` }))}
      columns={[
        { key: 'label', header: 'Source' },
        { key: 'value', header: 'Visits' },
        { key: 'percentage', header: 'Percentage' },
      ]}
    >
      <div className="donut-chart-wrapper">
        <ResponsiveContainer width={200} height={200}>
          <PieChart>
            <Pie
              data={data}
              cx="50%"
              cy="50%"
              innerRadius={60}
              outerRadius={90}
              paddingAngle={2}
              dataKey="value"
              nameKey="label"
            >
              {data.map((entry, index) => (
                <Cell key={index} fill={entry.color} />
              ))}
            </Pie>
            <Tooltip
              formatter={(v: number) => [v.toLocaleString(), '']}
            />
          </PieChart>
        </ResponsiveContainer>
        
        {/* Center label */}
        <div className="donut-center">
          <p className="donut-total">{total.toLocaleString()}</p>
          <p className="donut-label">Total</p>
        </div>
        
        {/* Legend */}
        <ul className="donut-legend">
          {data.map(d => (
            <li key={d.label} className="donut-legend-item">
              <span className="donut-legend-dot" style={{ background: d.color }} aria-hidden />
              <span className="donut-legend-label">{d.label}</span>
              <span className="donut-legend-value">
                {((d.value / total) * 100).toFixed(1)}%
              </span>
            </li>
          ))}
        </ul>
      </div>
    </AccessibleChart>
  );
}
```

---

## Sparklines

For small inline trend indicators:

```tsx
import { LineChart, Line, ResponsiveContainer } from 'recharts';

function Sparkline({ data, trend }: { data: number[]; trend: 'up' | 'down' | 'flat' }) {
  const chartData = data.map((value, index) => ({ value, index }));
  const color = trend === 'up' ? '#22c55e' : trend === 'down' ? '#ef4444' : '#94a3b8';
  
  return (
    <div aria-hidden="true" className="sparkline">
      <ResponsiveContainer width={80} height={32}>
        <LineChart data={chartData}>
          <Line
            type="monotone"
            dataKey="value"
            stroke={color}
            strokeWidth={1.5}
            dot={false}
          />
        </LineChart>
      </ResponsiveContainer>
    </div>
  );
}

// Usage in stat card:
<StatCard
  title="Revenue"
  value="$42,891"
  trend="up"
  sparklineData={[28, 32, 29, 35, 38, 42]}
  change={12}
  changeLabel="vs last month"
/>
```

---

## Real-time Charts

```tsx
// Streaming data chart that updates efficiently
function RealTimeChart({ dataFeed }: { dataFeed: Observable<DataPoint> }) {
  const [data, setData] = useState<DataPoint[]>([]);
  const MAX_POINTS = 60; // keep last 60 data points
  
  useEffect(() => {
    const sub = dataFeed.subscribe(point => {
      setData(prev => {
        const next = [...prev, point];
        return next.slice(-MAX_POINTS); // rolling window
      });
    });
    
    return () => sub.unsubscribe();
  }, [dataFeed]);
  
  return (
    <ResponsiveContainer width="100%" height={200}>
      <LineChart data={data}>
        <XAxis dataKey="timestamp" hide />
        <YAxis domain={['auto', 'auto']} />
        <Line
          type="monotone"
          dataKey="value"
          stroke="hsl(var(--primary))"
          dot={false}
          isAnimationActive={false} // CRITICAL: disable animation for real-time
        />
      </LineChart>
    </ResponsiveContainer>
  );
}
```

---

## Chart Tooltip Design

```tsx
function CustomTooltip({ active, payload, label }: TooltipProps) {
  if (!active || !payload?.length) return null;
  
  return (
    <div className="chart-tooltip">
      <p className="chart-tooltip-label">{label}</p>
      {payload.map(entry => (
        <div key={entry.name} className="chart-tooltip-row">
          <span
            className="chart-tooltip-dot"
            style={{ background: entry.color }}
            aria-hidden
          />
          <span className="chart-tooltip-name">{entry.name}</span>
          <span className="chart-tooltip-value">
            {typeof entry.value === 'number'
              ? entry.value.toLocaleString()
              : entry.value
            }
          </span>
        </div>
      ))}
    </div>
  );
}
```

```css
.chart-tooltip {
  background: hsl(var(--card));
  border: 1px solid hsl(var(--border));
  border-radius: var(--radius);
  padding: 10px 14px;
  box-shadow: var(--shadow-md);
  font-size: 13px;
  line-height: 1.4;
  min-width: 140px;
}

.chart-tooltip-label {
  font-weight: 600;
  color: hsl(var(--foreground));
  margin: 0 0 8px;
  font-size: 12px;
  text-transform: uppercase;
  letter-spacing: 0.05em;
  color: hsl(var(--muted-foreground));
}

.chart-tooltip-row {
  display: flex;
  align-items: center;
  gap: 8px;
  margin-top: 4px;
}

.chart-tooltip-dot {
  width: 8px;
  height: 8px;
  border-radius: 50%;
  flex-shrink: 0;
}

.chart-tooltip-name {
  flex: 1;
  color: hsl(var(--muted-foreground));
}

.chart-tooltip-value {
  font-weight: 600;
  color: hsl(var(--foreground));
  font-variant-numeric: tabular-nums;
}
```

---

## Chart Anti-Patterns

| Anti-Pattern | Problem | Fix |
|-------------|---------|-----|
| Pie chart with >5 slices | Impossible to compare | Use bar chart |
| 3D charts | Distorts values, hard to read | Use flat charts |
| Y-axis not starting at zero | Makes differences look exaggerated | Start at zero for bar charts |
| Too many series (>4) | Visual noise | Reduce to key series, add "others" |
| No axis labels | Users can't tell units | Always label axes with units |
| No tooltip | Users can't get exact values | Always add tooltip |
| Chart without data table alternative | Screen reader inaccessible | Provide toggle to view data table |
| Animation on real-time data | Jitters, performance issues | Disable animation with `isAnimationActive={false}` |
| Color-only series differentiation | Colorblind inaccessible | Add patterns, shapes, or dashes |
| No empty/loading state | White space | Design all states |

---

## Chart Checklist

- [ ] Chart has a descriptive title
- [ ] Axes have labels with units
- [ ] Tooltip shows exact values on hover
- [ ] Legend explains each series
- [ ] Color + pattern differentiates series (not color alone)
- [ ] Alt text or data table available for screen readers
- [ ] `aria-label` on chart container
- [ ] Empty state designed for when data is unavailable
- [ ] Loading skeleton while data fetches
- [ ] Y-axis starts at zero for bar charts (unless comparing small changes)
- [ ] Real-time charts have animation disabled
- [ ] Pie/donut charts have ≤5 slices
- [ ] Colors pass 3:1 contrast against chart background
- [ ] Chart is responsive (ResponsiveContainer or similar)
- [ ] Number formatting consistent (locale-aware)
