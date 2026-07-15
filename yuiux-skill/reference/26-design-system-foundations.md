# 26 — Design System Foundations

> A design system is not a component library. It's a shared language — a set of decisions, documented so clearly that anyone on the team can make consistent choices independently.

---

## What a Design System Contains

```
1. Design Tokens       — values (colors, spacing, type)
2. Component Library   — reusable UI components
3. Patterns            — compositions of components
4. Content Guidelines  — voice, tone, microcopy rules
5. Motion System       — animation principles and specs
6. Accessibility Rules — WCAG compliance requirements
7. Documentation       — usage, do/don't, examples
```

---

## Design System Document Structure

```
design-system/
├── OVERVIEW.md            — vision and principles
├── tokens/
│   ├── colors.md
│   ├── spacing.md
│   ├── typography.md
│   └── motion.md
├── components/
│   ├── button.md          — variants, states, usage, code
│   ├── form.md
│   ├── card.md
│   └── ...
├── patterns/
│   ├── data-entry.md
│   ├── empty-states.md
│   └── error-handling.md
├── content/
│   ├── voice-tone.md
│   └── writing-rules.md
└── accessibility/
    └── checklist.md
```

---

## Component Documentation Template

Every component in the system needs:

```markdown
# Button

## Overview
Short description of what the button does and when to use it.

## Variants
| Variant | When to Use | Never Use For |
|---------|-------------|---------------|
| Primary | Main CTA, one per view | Destructive actions |
| Secondary | Supporting actions | Primary action |
| Destructive | Dangerous, irreversible actions | Cancel actions |
| Ghost | Tertiary, low-emphasis actions | Primary CTA |

## States
Default / Hover / Focus / Active / Disabled / Loading

## Anatomy
[diagram with labeled parts]

## Props
| Prop | Type | Default | Description |
|------|------|---------|-------------|
| variant | 'default' \| 'secondary' \| 'destructive' \| 'ghost' \| 'link' | 'default' | Visual style |
| size | 'sm' \| 'md' \| 'lg' | 'md' | Size scale |
| loading | boolean | false | Shows spinner, disables interaction |
| disabled | boolean | false | Disables interaction |
| asChild | boolean | false | Renders as child element |

## Code
\`\`\`tsx
<Button variant="default" size="md">
  Save changes
</Button>
\`\`\`

## ✅ Do
- Use verb + noun copy: "Save changes", "Create project"
- Use one primary button per view
- Show loading state during async operations

## ❌ Don't
- Use "OK", "Submit", "Yes" as button copy
- Use two primary buttons side-by-side
- Use for navigation (use <a> or Link)

## Accessibility
- Uses native <button> element
- type="button" unless inside a form
- aria-busy="true" during loading
- Disabled state: aria-disabled + explanation
```

---

## Versioning a Design System

```
Major version (2.0.0): Breaking changes — component removed, API changed
Minor version (1.3.0): New components, backward-compatible features
Patch version (1.2.1): Bug fixes, accessibility improvements, documentation

Migration guides required for every major version.
Deprecation period: at least one minor version before removal.
```

```tsx
// Deprecation notice in code
/** @deprecated Use <Button variant="outline"> instead. Will be removed in v3.0. */
function OutlineButton(props) {
  console.warn('[DesignSystem] <OutlineButton> is deprecated. Use <Button variant="outline">.');
  return <Button variant="outline" {...props} />;
}
```

---

## Design Tokens as Source of Truth

The design system should have ONE source of truth for all values. All platforms (web, mobile, design tools) derive their values from the same source.

```json
// tokens.json — format-agnostic source
{
  "color": {
    "primary": {
      "50":  { "value": "#eff6ff" },
      "100": { "value": "#dbeafe" },
      "600": { "value": "#2563eb", "attributes": { "category": "color" } }
    },
    "semantic": {
      "primary":     { "value": "{color.primary.600}" },
      "text":        { "value": "{color.neutral.900}" },
      "text-muted":  { "value": "{color.neutral.500}" }
    }
  },
  "spacing": {
    "1": { "value": "4px" },
    "2": { "value": "8px" },
    "4": { "value": "16px" }
  }
}
```

```bash
# Style Dictionary transforms tokens to any platform
npx style-dictionary build

# Outputs:
#   css/variables.css       → web CSS custom properties
#   js/tokens.ts            → TypeScript exports
#   ios/Colors.swift        → iOS Swift colors
#   android/colors.xml      → Android XML
```

---

## Theming Architecture

```tsx
// Theme definition
interface Theme {
  colors: {
    primary:    string;
    background: string;
    foreground: string;
    border:     string;
    // ...
  };
  spacing: Record<string, string>;
  radii: Record<string, string>;
  fonts: { sans: string; mono: string; };
  shadows: Record<string, string>;
}

// Theme provider
function ThemeProvider({ theme, children }: { theme: Theme; children: ReactNode }) {
  const cssVars = Object.entries(flattenObject(theme))
    .map(([key, value]) => `${toCssVar(key)}: ${value}`)
    .join(';');
  
  return (
    <div style={{ [cssVars as string]: '' }}>
      {children}
    </div>
  );
}

// White-label theming
const acmeTheme: Theme = {
  colors: {
    primary: '#c0392b',  // Acme's red
    // ...
  },
  // ...
};
```

---

## Design System Governance

### Who Owns It
- **Design system team** — owns core components, tokens, documentation
- **Product teams** — consume the system, file requests for new components
- **Everyone** — responsible for following guidelines

### When to Add to the System
- Component appears in 3+ places across product
- Pattern solves a recurring design problem
- Pattern represents a decision the system should enforce

### When NOT to Add
- One-off component for a single product surface
- Highly context-specific logic
- Experiment/A/B test variants

### RFC Process
```
1. File an RFC (request for comment) with design + implementation proposal
2. Review period: 1–2 weeks
3. Approval from design system + at least 2 consuming teams
4. Implementation in system + documentation
5. Migration guide if deprecating old patterns
```

---

## Component Audit

Run a component audit before building the system:

```bash
# List all unique component types in codebase
rg -r 'className="btn' --only-matching | sort | uniq -c | sort -rn

# Count variations of a component
grep -r 'Button' src/components --include="*.tsx" | wc -l
```

```tsx
// Audit script: find non-system component usage
const NON_SYSTEM_COMPONENTS = [
  '<button', '<input', '<select', '<textarea',
  'style={{', '<div onClick',
];

// Run ESLint rules to enforce system usage:
// - no-native-html-button
// - no-inline-styles
// - no-div-onClick
```

---

## Documentation Site

Use Storybook, or a custom documentation site:

```tsx
// Story for each component
export default {
  title: 'UI/Button',
  component: Button,
  argTypes: {
    variant: {
      control: 'select',
      options: ['default', 'secondary', 'destructive', 'ghost', 'link'],
    },
    size: {
      control: 'select',
      options: ['sm', 'md', 'lg'],
    },
    loading: { control: 'boolean' },
    disabled: { control: 'boolean' },
  },
};

// Primary story
export const Primary = {
  args: { variant: 'default', children: 'Save changes' },
};

// Loading state
export const Loading = {
  args: { variant: 'default', loading: true, children: 'Saving...' },
};

// All variants
export const AllVariants = () => (
  <div style={{ display: 'flex', gap: 8 }}>
    {['default', 'secondary', 'destructive', 'ghost', 'link'].map(v => (
      <Button key={v} variant={v}>{v}</Button>
    ))}
  </div>
);
```

---

## Checklist for Design System Readiness

### Tokens
- [ ] All color values defined as tokens (no hardcoded hex)
- [ ] All spacing on 4px grid with tokens
- [ ] Semantic token layer over primitives
- [ ] Dark mode tokens defined
- [ ] Motion tokens defined

### Components
- [ ] All components documented with variants, states, and props
- [ ] All components have TypeScript types exported
- [ ] All components have accessibility documentation
- [ ] All components tested in isolation (Storybook or similar)

### Process
- [ ] Contribution guidelines documented
- [ ] RFC process defined
- [ ] Breaking change and deprecation policy defined
- [ ] Changelog maintained
- [ ] Design tokens versioned alongside code
