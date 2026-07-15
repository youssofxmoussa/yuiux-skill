---
name: charts-dataviz
description: Data-type to chart-type decision table (25 data patterns) with accessibility grades, library recommendations, and mandatory non-visual fallbacks. Raw table in data/charts.csv.
---

# Charts & Data Visualization

Full raw data (25 rows: data type, best chart type, secondary options, when/when-not to use, volume thresholds, color guidance, accessibility grade, a11y notes, fallback, library recommendation) lives in `../../data/charts.csv`. Grep it when picking a chart for a specific data shape.

## Decision heuristic

Pick the chart type from the *data relationship*, not visual preference:

- **Trend over time** → Line/Area chart. Never use for <4 data points (use a stat card instead) or with a time axis but no rate-of-change story.
- **Compare discrete categories** → Bar chart, sorted descending by default. Beyond 15-20 categories, switch to horizontal bar or a searchable table.
- **Part-to-whole** → Donut/pie *only* for ≤5 categories with one dominant segment. Beyond that or when precision matters, use a 100% stacked bar. Pie/donut is accessibility-graded **C** — always ship a stacked-bar or table alternative alongside it, never as the sole representation.
- **Correlation/distribution** → Scatter/bubble plot. Needs ≥20 points to read as a pattern; below that, list the values.
- **Intensity across a grid/time** → Heatmap/choropleth, always with a numeric legend with scale ticks (never color-only).
- **Sequential funnel/flow** → Funnel/Sankey for 3-8 stages; beyond 8, group minor steps into "Other." Show conversion % as text at every stage, never color-only.
- **KPI vs target** → Gauge/bullet chart; bullet chart scales far better than gauges for a dashboard grid of 3+ KPIs. Always show the number as visible text next to the visual — never hover-only.
- **Hierarchical proportions** → Treemap/sunburst, max 2-3 nesting levels before it becomes unreadable. Accessibility-graded C-D — pair with a collapsible indented list as the primary accessible view.
- **Network/relationship data** → Force-directed graph, graded **D** (worst accessibility of any chart type) — never ship as the sole representation; always include an adjacency-list table.
- **3D spatial data** → Only for genuine scientific/engineering Z-axis need. Graded **D**; never use as a primary chart in a standard product UI — always pair with a 2D projection + table.

## Accessibility rules (apply to every chart, regardless of grade)

- Never encode meaning in color alone — pair with shape, pattern, line-style, or text label for any chart with more than 2 series.
- Every chart graded B or below in the CSV **must** ship a non-visual fallback: sortable data table, text summary, or list view. This is not optional polish — treat it as a shipping blocker for public-facing dashboards.
- Value labels should be visible by default on bar/funnel/bullet charts rather than hover-only, so touch users and screen-magnifier users aren't excluded.
- Real-time/streaming charts need a pause/resume control and must respect `prefers-reduced-motion` (freeze animation, keep data updating).

## Volume thresholds (performance)

Render technology should scale with data volume: SVG is fine under ~500-1000 points; beyond that, move to Canvas/WebGL and downsample or aggregate. The CSV specifies exact thresholds per chart type — check it before rendering anything with an unbounded dataset (e.g., raw event logs, per-second telemetry).

## Library defaults

- General-purpose React: **Recharts** or **Chart.js** cover trend/bar/pie/radar/scatter needs with good accessibility defaults.
- Heavy customization / novel chart types (Sankey, treemap, network, sunburst): **D3.js**.
- Financial candlestick/OHLC: **Lightweight Charts** (TradingView) purpose-built for this.
- Interactive dashboards needing zoom/pan/drill on many chart types: **Plotly** or **ApexCharts**.
