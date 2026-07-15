# 04 — Layout, Spacing & Grid

> Space is not empty — it is a design element. Every gap, margin, and padding communicates structure, hierarchy, and relationship between elements.

---

## The 8px Base Unit System

All spacing should be multiples of 4px (with 8px as the primary unit):

```
4px  (0.25rem) — micro: icon↔label gap, badge padding
8px  (0.5rem)  — tight: list item internal, inline element gap
12px (0.75rem) — compact: input padding, small card padding  
16px (1rem)    — base: standard component internal padding
20px (1.25rem) — comfortable: form group gap
24px (1.5rem)  — section internal padding, card padding
32px (2rem)    — section gap, generous card padding
40px (2.5rem)  — large section gap
48px (3rem)    — major section break
64px (4rem)    — feature/content section spacing
80px (5rem)    — large section separation
96px (6rem)    — hero/page-level rhythm
128px (8rem)   — maximum section gap for most layouts
```

### CSS Custom Properties
```css
:root {
  --space-1:  0.25rem;  /* 4px */
  --space-2:  0.5rem;   /* 8px */
  --space-3:  0.75rem;  /* 12px */
  --space-4:  1rem;     /* 16px */
  --space-5:  1.25rem;  /* 20px */
  --space-6:  1.5rem;   /* 24px */
  --space-8:  2rem;     /* 32px */
  --space-10: 2.5rem;   /* 40px */
  --space-12: 3rem;     /* 48px */
  --space-16: 4rem;     /* 64px */
  --space-20: 5rem;     /* 80px */
  --space-24: 6rem;     /* 96px */
  --space-32: 8rem;     /* 128px */
}
```

---

## Spacing Rules

### Proximity Rule
Related elements should be closer to each other than to unrelated elements. The gap between an input and its label should be much smaller than the gap between two form groups.

```css
/* ✅ Correct: label and input close, groups far apart */
.form-group {
  display: flex;
  flex-direction: column;
  gap: 6px;         /* label ↔ input: very close (related) */
  margin-bottom: 24px; /* group ↔ group: far (separate) */
}

/* ❌ Wrong: equal spacing everywhere */
.form-group {
  margin-bottom: 16px; /* same as internal gap — hierarchy lost */
}
.form-group label, .form-group input {
  margin-bottom: 16px;
}
```

### Consistent Padding in Components
```css
/* All components in a family use consistent internal padding */
.card-sm    { padding: 16px; }
.card       { padding: 24px; }
.card-lg    { padding: 32px; }

/* Never mix padding values within the same card */
/* ❌ */
.card { padding-top: 24px; padding-bottom: 20px; padding-left: 18px; }
/* ✅ */
.card { padding: 24px; }
```

### Vertical Rhythm
All vertical spacing should be on the 8px grid. Never use arbitrary pixel values like 13px, 22px, or 37px.

```css
/* ✅ On-grid spacing */
.section { padding-block: 64px; }
.content-gap { gap: 32px; }
.element-gap { gap: 16px; }

/* ❌ Off-grid */
.section { padding-top: 45px; padding-bottom: 63px; }
```

---

## Grid Systems

### 12-Column Grid (Standard)

```css
.container {
  width: 100%;
  max-width: 1280px;
  margin-inline: auto;
  padding-inline: 24px; /* gutter */
}

.grid-12 {
  display: grid;
  grid-template-columns: repeat(12, 1fr);
  gap: 24px; /* column gap */
}

/* Common spans */
.col-span-12 { grid-column: span 12; } /* full width */
.col-span-8  { grid-column: span 8;  } /* main content */
.col-span-6  { grid-column: span 6;  } /* half */
.col-span-4  { grid-column: span 4;  } /* third */
.col-span-3  { grid-column: span 3;  } /* quarter */
```

### Responsive Grid
```css
/* Mobile-first grid that adapts to viewport */
.grid {
  display: grid;
  grid-template-columns: 1fr;
  gap: 16px;
}

@media (min-width: 640px) {  /* sm */
  .grid { grid-template-columns: repeat(2, 1fr); gap: 20px; }
}

@media (min-width: 768px) {  /* md */
  .grid { grid-template-columns: repeat(3, 1fr); gap: 24px; }
}

@media (min-width: 1024px) { /* lg */
  .grid { grid-template-columns: repeat(4, 1fr); gap: 24px; }
}
```

### CSS Grid Named Areas (Complex Layouts)
```css
.app-layout {
  display: grid;
  grid-template-areas:
    "header  header"
    "sidebar main"
    "footer  footer";
  grid-template-columns: 240px 1fr;
  grid-template-rows: auto 1fr auto;
  min-height: 100dvh;
}

.app-header  { grid-area: header; }
.app-sidebar { grid-area: sidebar; }
.app-main    { grid-area: main; }
.app-footer  { grid-area: footer; }

/* Collapse sidebar on mobile */
@media (max-width: 768px) {
  .app-layout {
    grid-template-areas:
      "header"
      "main"
      "footer";
    grid-template-columns: 1fr;
  }
  .app-sidebar { display: none; }
}
```

---

## Container Sizes

```css
/* Standard container max-widths */
.container-sm  { max-width: 640px;  }  /* forms, auth pages */
.container-md  { max-width: 768px;  }  /* settings, narrow content */
.container     { max-width: 1024px; }  /* default content */
.container-lg  { max-width: 1280px; }  /* main layouts */
.container-xl  { max-width: 1536px; }  /* wide dashboards */
.container-2xl { max-width: 1920px; }  /* full-bleed layouts */

/* All containers: centered with horizontal padding */
[class*="container"] {
  width: 100%;
  margin-inline: auto;
  padding-inline: clamp(16px, 4vw, 32px);
}
```

---

## Flexbox Patterns

### Common Layout Patterns
```css
/* Horizontal stack with center alignment */
.row {
  display: flex;
  align-items: center;
  gap: 8px;
}

/* Horizontal with space between (header pattern) */
.space-between {
  display: flex;
  align-items: center;
  justify-content: space-between;
}

/* Vertical stack */
.stack {
  display: flex;
  flex-direction: column;
  gap: 16px;
}

/* Centered content */
.center {
  display: flex;
  align-items: center;
  justify-content: center;
}

/* Flex wrap for responsive tags/chips */
.flex-wrap {
  display: flex;
  flex-wrap: wrap;
  gap: 8px;
}
```

### Sidebar + Main Pattern
```css
.sidebar-layout {
  display: flex;
  min-height: 100dvh;
}

.sidebar {
  width: 240px;
  flex-shrink: 0;
  position: sticky;
  top: 0;
  height: 100dvh;
  overflow-y: auto;
}

.main-content {
  flex: 1;
  min-width: 0; /* prevent flex blowout */
  padding: 24px;
  overflow: hidden;
}
```

---

## Breakpoints

### Standard Breakpoints (Tailwind-aligned)

```css
/* Mobile first — styles at root apply to all */
/* Then override for larger screens */

@media (min-width: 480px)  { /* xs: large mobile */ }
@media (min-width: 640px)  { /* sm: small tablet */ }
@media (min-width: 768px)  { /* md: tablet */ }
@media (min-width: 1024px) { /* lg: small desktop */ }
@media (min-width: 1280px) { /* xl: desktop */ }
@media (min-width: 1536px) { /* 2xl: large desktop */ }
```

### Critical Breakpoints to Always Test

| Width | Device | What to check |
|-------|--------|---------------|
| 320px | iPhone SE, small phones | No horizontal scroll, text readable |
| 375px | iPhone 14 standard | Primary mobile experience |
| 414px | iPhone Plus/Max | Slightly larger mobile |
| 768px | iPad portrait | Tablet layout kicks in |
| 1024px | iPad landscape / small laptop | Desktop layout begins |
| 1280px | Standard laptop | Main desktop target |
| 1440px | Wide laptop / monitor | Full desktop experience |

---

## Visual Hierarchy Techniques

### Size Hierarchy
```
Large: primary content, hero, h1
Medium: secondary content, h2, cards  
Small: meta information, labels, helper text
```

### Weight Hierarchy
```
700+ Bold: headings, primary labels
500-600 Medium/SemiBold: secondary labels, nav items
400 Regular: body text, descriptions
300 Light: captions (large sizes only)
```

### Space Hierarchy
```
More space → More important (breathing room = emphasis)
Less space → Less important, or more related to adjacent content
```

### Color Hierarchy
```
High contrast → Primary/important
Medium contrast → Secondary
Low contrast → Tertiary/muted
```

### Combining Hierarchy Cues

```tsx
// ✅ Multiple hierarchy signals aligned
function StatCard({ title, value, description }: Props) {
  return (
    <div className="p-6 rounded-lg border bg-card">
      {/* Low contrast, small, medium weight — least important */}
      <p className="text-sm font-medium text-muted-foreground">{title}</p>
      
      {/* High contrast, large, bold — most important */}
      <p className="text-3xl font-bold text-foreground mt-1">{value}</p>
      
      {/* Low contrast, small — secondary */}
      <p className="text-sm text-muted-foreground mt-1">{description}</p>
    </div>
  );
}
```

---

## Component Spacing Standards

### Cards
```css
.card {
  padding: 24px;          /* standard */
  border-radius: 8px;
  border: 1px solid var(--border);
}

.card-compact {
  padding: 16px;          /* dense UI */
}

.card-spacious {
  padding: 32px 40px;     /* marketing/landing */
}

/* Card header/content/footer pattern */
.card-header  { padding: 24px 24px 0; }
.card-content { padding: 24px; }
.card-footer  { padding: 0 24px 24px; }
```

### Form Layout
```css
.form { 
  display: flex;
  flex-direction: column;
  gap: 20px;             /* between field groups */
  max-width: 480px;      /* forms should never span full wide page */
}

.field-group {
  display: flex;
  flex-direction: column;
  gap: 6px;              /* label to input */
}

.form-actions {
  display: flex;
  gap: 8px;
  justify-content: flex-end; /* right-align by default */
  margin-top: 8px;       /* extra space above actions */
}
```

### Navigation
```css
/* Sidebar nav item */
.nav-item {
  display: flex;
  align-items: center;
  gap: 10px;
  padding: 8px 12px;
  border-radius: 6px;
  font-size: 14px;
  font-weight: 500;
}

/* Top nav */
.top-nav {
  height: 56px;          /* standard header height */
  padding-inline: 24px;
  display: flex;
  align-items: center;
  gap: 32px;             /* space between nav groups */
}
```

### Table
```css
.table-cell {
  padding: 12px 16px;    /* comfortable table cells */
}

.table-cell-compact {
  padding: 8px 12px;     /* dense tables */
}

.table-header-cell {
  padding: 10px 16px;    /* slightly shorter than body cells */
  font-size: 12px;
  font-weight: 600;
  letter-spacing: 0.05em;
  text-transform: uppercase;
  color: var(--muted-foreground);
}
```

---

## Layout Anti-Patterns

### 1. Spacing Inconsistency
```css
/* ❌ Random spacing throughout */
.section-1 { margin-bottom: 37px; }
.section-2 { margin-bottom: 42px; }
.element   { padding: 13px 19px; }

/* ✅ On-grid, consistent */
.section   { margin-bottom: 48px; }
.element   { padding: 12px 20px; }
```

### 2. Full-Width Forms
```css
/* ❌ Form spans entire 1200px page */
.form { width: 100%; }

/* ✅ Constrained for readability and focus */
.form { max-width: 480px; }
```

### 3. Missing Min-Width on Flex Children
```css
/* ❌ Long text in flex child overflows container */
.flex-container { display: flex; }
.flex-child     { /* no min-width — can overflow */ }

/* ✅ */
.flex-child { min-width: 0; overflow: hidden; text-overflow: ellipsis; }
```

### 4. Fixed Heights on Text Containers
```css
/* ❌ Text clips if content is longer than expected */
.card-body { height: 120px; overflow: hidden; }

/* ✅ Let content define height */
.card-body { min-height: 120px; }
```

### 5. Tight Touch Targets
```css
/* ❌ 20px tap target — impossible on mobile */
.chip { padding: 2px 8px; }

/* ✅ Minimum 44px tall for touchable elements */
.chip { 
  padding: 10px 12px; /* makes it 40px tall minimum */
  min-height: 44px;
}
```

### 6. No Visual Break Between Sections
```css
/* ❌ Everything runs together */
.section { margin-bottom: 16px; }

/* ✅ Clear section separation */
.section { 
  padding-block: 64px;
  border-bottom: 1px solid var(--border);
}
/* Or use background alternation */
.section-alt { background: var(--surface); }
```

---

## Alignment Rules

### Text Alignment
```
Left-align: body text, form labels, navigation, table content
Center-align: page-level headings, hero text, empty states, success screens
Right-align: numbers in tables (for alignment), currencies, timestamps in lists
Never justify: body text on web (creates rivers of whitespace)
```

### Icon Alignment
```css
/* Icons must be visually centered, not mathematically centered */
/* Most icon fonts/SVGs need optical adjustment */

.icon-text-pair {
  display: flex;
  align-items: center; /* center-aligned */
  gap: 8px;
}

/* For multi-line text next to icon: top-align */
.icon-text-multiline {
  display: flex;
  align-items: flex-start;
  gap: 12px;
}
```

### Button Content Alignment
```css
button {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  gap: 8px; /* icon to label gap */
}
```

---

## Z-Index Scale

```css
:root {
  --z-below:    -1;   /* Behind everything */
  --z-base:      0;   /* Default */
  --z-raised:    1;   /* Slightly elevated (e.g., sticky table header) */
  --z-dropdown:  10;  /* Dropdowns, popovers */
  --z-sticky:    20;  /* Sticky headers */
  --z-fixed:     30;  /* Fixed position elements */
  --z-modal-bg:  40;  /* Modal backdrop */
  --z-modal:     50;  /* Modal content */
  --z-popover:   60;  /* Popovers above modals */
  --z-toast:     70;  /* Toast notifications */
  --z-tooltip:   80;  /* Tooltips (highest) */
}
```

---

## Layout Checklist

- [ ] All spacing on 4px or 8px grid
- [ ] Related elements grouped closer than unrelated
- [ ] Container max-width set to prevent overly wide content
- [ ] Form max-width ≤ 480–560px
- [ ] Body text max-width ≤ 65ch
- [ ] Flexbox items have `min-width: 0` if they contain text
- [ ] No fixed heights on text containers
- [ ] Z-index values use the standardized scale
- [ ] Grid responsive and tested at all breakpoints
- [ ] Touch targets ≥ 44px on mobile
- [ ] Section-level whitespace generous enough to signal boundaries
