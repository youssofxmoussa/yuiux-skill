# 37 — Design Handoff

> The gap between design and implementation is where quality dies. A good handoff makes that gap as narrow as possible.

---

## The Handoff Problem

Traditional design handoff:
```
Designer: "Here's the Figma file."
Developer: "Which font size is this? What happens on hover? Is this 16px or 18px? What's the error state?"
Designer: "Oh, I didn't design that."
```

Good design handoff:
```
Designer delivers: every state, every variant, every error, every breakpoint,
                   named components, annotated interactions, exported tokens.
Developer delivers: a faithful, accessible implementation with zero guesswork.
```

---

## What Must Be in a Handoff

```
For every component:
  ✓ Default state
  ✓ Hover state
  ✓ Focus state
  ✓ Active/pressed state
  ✓ Disabled state
  ✓ Loading state
  ✓ Error state
  ✓ Empty state
  ✓ All variants (primary, secondary, small, large, etc.)
  ✓ All breakpoints (mobile, tablet, desktop)
  ✓ Spacing and sizing annotations
  ✓ Typography specs (family, weight, size, line-height, tracking)
  ✓ Color tokens (not hex — token names)
  ✓ Interaction spec (what happens on click, hover, keyboard)
  ✓ Animation specs (enter/exit, duration, easing)
  ✓ Accessibility notes (ARIA, focus behavior, screen reader text)
```

---

## Figma Token Export

```json
// Design tokens from Figma via Tokens Studio plugin
// tokens.json — exported and committed to repo
{
  "$themes": [],
  "global": {
    "color": {
      "primary": {
        "50":  { "$value": "#eff6ff", "$type": "color" },
        "600": { "$value": "#2563eb", "$type": "color" }
      },
      "semantic": {
        "primary":    { "$value": "{color.primary.600}", "$type": "color" },
        "text":       { "$value": "{color.neutral.900}", "$type": "color" }
      }
    },
    "spacing": {
      "4":  { "$value": "16", "$type": "spacing" },
      "6":  { "$value": "24", "$type": "spacing" }
    },
    "borderRadius": {
      "md": { "$value": "6", "$type": "borderRadius" },
      "lg": { "$value": "8", "$type": "borderRadius" }
    },
    "fontSizes": {
      "base": { "$value": "16", "$type": "fontSizes" },
      "lg":   { "$value": "18", "$type": "fontSizes" }
    }
  }
}
```

```bash
# Transform tokens to CSS/JS via Style Dictionary
npx style-dictionary build --config sd.config.json

# sd.config.json
{
  "source": ["tokens.json"],
  "platforms": {
    "css": {
      "transformGroup": "css",
      "buildPath": "src/styles/",
      "files": [{ "destination": "tokens.css", "format": "css/variables" }]
    },
    "js": {
      "transformGroup": "js",
      "buildPath": "src/",
      "files": [{ "destination": "tokens.ts", "format": "javascript/es6" }]
    }
  }
}
```

---

## Redline Annotations

Redlines are spacing/sizing annotations in the design file. With design tokens, you annotate token names instead of pixel values:

```
❌ Old way (pixel annotations):
  Margin: 16px
  Font size: 14px
  Border radius: 6px

✅ Token-based annotations:
  Margin: --space-4 (16px)
  Font size: --text-small-size (14px)
  Border radius: --radius-component (6px)
```

This means one source of truth — if the token changes, both design and code update.

---

## Interaction Specification

Every interactive element needs an interaction spec:

```markdown
## Button — Interaction Spec

### States
| State    | Background          | Border | Text Color | Cursor |
|----------|---------------------|--------|------------|--------|
| Default  | --color-primary     | none   | white      | pointer |
| Hover    | --color-primary-hover | none | white    | pointer |
| Focus    | --color-primary     | 2px --color-focus-ring offset 2px | white | auto |
| Active   | --color-primary-active | none | white   | pointer |
| Disabled | --color-primary 40% opacity | none | white 40% | not-allowed |
| Loading  | --color-primary     | none | transparent | not-allowed |

### Transitions
- Background color: 150ms ease-out
- Transform on press: scale(0.97), 75ms ease-in

### Focus Behavior
- Focus visible on keyboard navigation only (focus-visible)
- Focus ring: 2px solid --color-focus-ring, offset 2px
- Tab order: natural document order

### Loading State
- Spinner replaces icon (not label)
- Pointer events: none
- aria-busy="true"
- Width does not change (prevents layout shift)
```

---

## Motion Specification

```markdown
## Dropdown — Animation Spec

### Enter
- Duration: 200ms
- Easing: cubic-bezier(0, 0, 0.2, 1) (ease-out)
- Properties:
  - opacity: 0 → 1
  - transform: translateY(-8px) → translateY(0)
- Trigger: menu opens

### Exit
- Duration: 150ms
- Easing: cubic-bezier(0.4, 0, 1, 1) (ease-in)
- Properties:
  - opacity: 1 → 0
  - transform: translateY(0) → translateY(-8px)
- Trigger: menu closes (click outside, Escape, item selected)

### Reduced motion
- All transforms removed
- Opacity: instant (no fade)
- Respect prefers-reduced-motion: reduce
```

---

## Component Inventory Spreadsheet

Useful for large projects — tracks every component's design and implementation status:

```
Component          | Variants | States | Mobile? | Accessibility | Implemented | Notes
-------------------|----------|--------|---------|---------------|-------------|-------
Button             | 5        | 6      | ✓       | Full          | ✓           |
Input              | 3        | 5      | ✓       | Full          | ✓           |
Modal              | 2        | 3      | ✓       | Full          | ✓           |
Combobox           | 1        | 4      | ✗       | Partial       | In progress | Mobile design missing
DatePicker         | 1        | 3      | ✗       | None          | Not started | Need design
DataTable          | 2        | 5      | ✗       | None          | Not started | Need mobile spec
```

---

## Developer Handoff Notes Template

```markdown
## Component: UserCard

### What it is
Displays a user's avatar, name, role, and action menu. Used in:
- Team settings page (list)
- Project members sidebar (compact variant)

### Variants
1. **Default** — full card with avatar, name, role, and actions
2. **Compact** — avatar + name only, no actions

### Key implementation notes
- Avatar uses `<UserAvatar>` component with fallback initials
- Action menu: click → dropdown (not hover)
- Keyboard: Tab into card, Tab to action button, Space/Enter to open menu
- Deactivated users: full opacity, `aria-disabled="true"` on actions
- Role badge colors come from `--color-role-{admin|editor|viewer}`

### States
| State      | Visual change          |
|------------|------------------------|
| Loading    | Use `<UserCardSkeleton>` |
| Error      | Hidden — show in parent error boundary |
| Hover      | Surface shadow increases |
| Selected   | Blue left border + --color-primary-subtle bg |

### Responsive behavior
- Mobile (< 640px): Compact variant forced, no actions (use swipe-to-action)
- Tablet (640–1024px): Default variant, actions in 3-dot menu
- Desktop (≥ 1024px): Default variant, action buttons visible
```

---

## Handoff Checklist

- [ ] All component states designed and annotated
- [ ] All responsive breakpoints shown
- [ ] Color values reference token names, not hex
- [ ] Spacing values reference token names, not px
- [ ] Typography annotated with all properties (size, weight, line-height)
- [ ] Interaction spec written for all interactive components
- [ ] Animation specs include duration, easing, properties, trigger
- [ ] Reduced motion variant specified for all animations
- [ ] Accessibility notes included (ARIA roles, focus behavior, sr text)
- [ ] Tokens exported from Figma as JSON (or maintained in code)
- [ ] Component inventory tracking design vs implementation status
- [ ] Developer notes template filled for complex components
- [ ] Edge cases documented (long text, empty, single item, maximum items)
- [ ] Error states designed for all data-fetching components
