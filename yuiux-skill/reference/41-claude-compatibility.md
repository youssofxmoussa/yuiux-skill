# 41 — Claude Compatibility & Prompting

> This file documents how to inject the YUIUX skill into Claude conversations, how to structure prompts for UI/UX work, and how to get the highest quality output from Claude when building interfaces.

---

## Skill Injection

To activate the full YUIUX skill in a Claude session:

```
System prompt injection pattern:

You are an expert UI/UX engineer. Before building any UI, always consult the
YUIUX skill at yuiux-skill/SKILL.md and the relevant reference files in
yuiux-skill/reference/. Apply every principle in the skill proactively —
especially the Pre-Ship Gate checklist (Gates A–E in SKILL.md) before
considering any UI complete.

Key files to reference for common tasks:
  Forms:           reference/06-forms-inputs.md
  Buttons:         reference/07-buttons-controls.md
  Accessibility:   reference/10-accessibility-deep-dive.md
  Loading states:  reference/09-feedback-loading-empty-error-states.md
  Motion:          reference/05-motion-animation-microinteractions.md
  Dark mode:       reference/29-dark-mode-theming.md
  Mobile:          reference/25-mobile-first-touch.md
  Performance:     reference/17-performance-web-vitals.md
```

---

## Prompting Patterns for UI Work

### Request a Complete Component

```
Prompt template:

Build a [component name] component for [context/platform].

Requirements:
- Stack: [React + Tailwind / Lovable / Replit / etc.]
- Variants: [list variants]
- States: default, hover, focus, active, disabled, loading, error, empty
- Accessibility: WCAG 2.2 AA — keyboard navigation, ARIA, focus management
- Responsive: mobile-first, works at 320px–1440px
- Motion: respect prefers-reduced-motion
- Theme: light + dark mode via CSS custom properties

Reference: yuiux-skill/reference/[relevant-file].md

Example prompt for a data table:
Build a DataTable component for a Lovable (React + shadcn/ui + Tailwind) project.
Requirements: sortable columns, row selection via checkboxes, pagination, mobile
responsive (card layout at < 640px), keyboard accessible, dark mode, skeleton
loading state, empty state with CTA. Reference: yuiux-skill/reference/12-tables-data-display.md
```

### Request a UX Audit

```
Prompt template:

Audit the [component/page/flow] in [file path] against:
1. WCAG 2.2 AA accessibility (reference/10-accessibility-deep-dive.md)
2. UX error prevention (reference/06-forms-inputs.md for forms)
3. Performance (reference/17-performance-web-vitals.md)
4. Mobile UX (reference/25-mobile-first-touch.md)
5. Pre-Ship Gate checklist (yuiux-skill/SKILL.md Gates A–E)

For each issue found:
- State the problem
- Rate severity: critical / major / minor / cosmetic
- Provide the specific code fix
```

### Request a Design Review

```
Prompt template:

Review this design decision: [describe the UI decision]

Consider:
- Does it follow the component hierarchy from reference/20-component-architecture.md?
- Does it use semantic design tokens (reference/19-design-tokens.md)?
- Does it handle all states (reference/09-feedback-loading-empty-error-states.md)?
- Does the copy follow rules in reference/27-content-strategy-microcopy.md?
- Does it respect the destructive action spec (reference/33-destructive-actions-confirmation.md)?

Provide: recommendation + rationale + code example.
```

---

## Claude Response Formatting for UI Work

When Claude is generating UI code, request this format:

```
Please structure your response as:

1. **Decision** — the specific approach chosen and why
2. **Component code** — complete, copy-pasteable TypeScript/TSX
3. **CSS/Styles** — if separate from component
4. **Accessibility notes** — ARIA, keyboard, screen reader behavior
5. **States handled** — list each state the component covers
6. **What's NOT included** — explicit list of states/edge cases not covered
7. **Integration notes** — how to connect to real data/events
```

---

## Skill Reference Quick Map

```
For this task...                          Read this file...
─────────────────────────────────────────────────────────────────
Building a form                           06-forms-inputs.md
Building a button                         07-buttons-controls.md
Building navigation                       08-navigation-ia.md
Adding loading/empty/error states         09-feedback-loading-empty-error-states.md
Making something accessible              10-accessibility-deep-dive.md
Building a modal/dialog/drawer            11-modals-dialogs-overlays.md
Building a data table                     12-tables-data-display.md
Building cards                            13-cards-components.md
Adding toast notifications                14-notifications-toasts.md
Adding search/filtering                   15-search-filtering.md
Making it responsive                      16-responsive-design.md
Optimizing performance                    17-performance-web-vitals.md
Choosing/using icons                      18-icons-imagery.md
Setting up design tokens                  19-design-tokens.md
Architecting components                   20-component-architecture.md
Building charts/dashboards                21-data-visualization-charts.md
Building login/signup                     22-authentication-ux.md
Building onboarding                       23-onboarding-ux.md
Building a dashboard                      24-dashboard-design.md
Mobile patterns                           25-mobile-first-touch.md
Design system setup                       26-design-system-foundations.md
Writing UI copy                           27-content-strategy-microcopy.md
Testing with users                        28-user-testing-validation.md
Adding dark mode                          29-dark-mode-theming.md
Adding i18n/RTL                           30-internationalization.md
Building checkout                         31-payment-checkout-ux.md
Building settings pages                   32-settings-preferences.md
Destructive actions                       33-destructive-actions-confirmation.md
Drag and drop                             34-drag-drop-gestures.md
Infinite scroll/pagination                35-infinite-scroll-pagination.md
AI chat UI                                36-ai-ui-patterns.md
Design handoff                            37-design-handoff.md
Adding analytics/error tracking           38-observability-analytics.md
Building in Lovable                       39-lovable-compatibility.md
Building in Replit                        40-replit-compatibility.md
Prompting Claude for UI work              41-claude-compatibility.md (this file)
```

---

## Anti-Pattern Flags

When reviewing Claude's output, flag these as violations of the skill:

```
🚫 Icon without aria-hidden or aria-label
   → See: reference/18-icons-imagery.md

🚫 Form error without specific resolution message
   → See: reference/06-forms-inputs.md

🚫 Button copy: "Submit", "OK", "Yes", "Confirm"
   → See: reference/27-content-strategy-microcopy.md

🚫 Modal without focus trap or Escape to close
   → See: reference/11-modals-dialogs-overlays.md

🚫 Loading state missing (only happy path coded)
   → See: reference/09-feedback-loading-empty-error-states.md

🚫 Hardcoded color value instead of CSS variable
   → See: reference/19-design-tokens.md

🚫 Destructive action with "Are you sure?" + "Yes/No"
   → See: reference/33-destructive-actions-confirmation.md

🚫 Font size < 16px on mobile inputs (iOS zoom bug)
   → See: reference/25-mobile-first-touch.md

🚫 Animation without prefers-reduced-motion check
   → See: reference/05-motion-animation-microinteractions.md

🚫 Hero image with loading="lazy" (kills LCP)
   → See: reference/17-performance-web-vitals.md

🚫 Image without width and height (causes CLS)
   → See: reference/17-performance-web-vitals.md

🚫 div with onClick instead of button/link
   → See: reference/10-accessibility-deep-dive.md

🚫 Placeholder used as field label (label removed when typing)
   → See: reference/06-forms-inputs.md

🚫 Dark mode with hardcoded white/black instead of CSS variables
   → See: reference/29-dark-mode-theming.md

🚫 Touch target < 44px (mobile usability failure)
   → See: reference/25-mobile-first-touch.md
```

---

## Skill Auto-Activation Triggers

The following request keywords should automatically trigger reading the relevant reference file:

```javascript
const TRIGGER_MAP = {
  // Keywords → Reference files to load
  'form|input|field|textarea|select|checkbox|radio|validation|error message':
    'reference/06-forms-inputs.md',
  
  'button|CTA|call to action|submit':
    'reference/07-buttons-controls.md',
  
  'nav|navigation|menu|sidebar|breadcrumb|tab':
    'reference/08-navigation-ia.md',
  
  'loading|skeleton|spinner|empty state|error state|404':
    'reference/09-feedback-loading-empty-error-states.md',
  
  'accessibility|a11y|aria|screen reader|keyboard|wcag':
    'reference/10-accessibility-deep-dive.md',
  
  'modal|dialog|drawer|sheet|popover|tooltip|overlay':
    'reference/11-modals-dialogs-overlays.md',
  
  'table|data grid|sortable|pagination':
    'reference/12-tables-data-display.md',
  
  'animation|transition|motion|hover|microinteraction':
    'reference/05-motion-animation-microinteractions.md',
  
  'dark mode|theme|color scheme|light mode':
    'reference/29-dark-mode-theming.md',
  
  'mobile|touch|responsive|phone|tablet|breakpoint':
    'reference/25-mobile-first-touch.md',
  
  'performance|lcp|cls|inp|web vitals|lazy load|bundle':
    'reference/17-performance-web-vitals.md',
  
  'login|signup|sign in|auth|password|mfa|2fa':
    'reference/22-authentication-ux.md',
  
  'delete|destructive|confirm|undo|irreversible':
    'reference/33-destructive-actions-confirmation.md',
  
  'dashboard|kpi|metric|chart|analytics|stat':
    'reference/24-dashboard-design.md',
  
  'ai|chat|stream|gpt|claude|llm|prompt':
    'reference/36-ai-ui-patterns.md',
  
  'checkout|payment|stripe|cart|purchase|billing':
    'reference/31-payment-checkout-ux.md',
  
  'drag|drop|reorder|sortable|kanban|dnd':
    'reference/34-drag-drop-gestures.md',
  
  'lovable|shadcn|framer motion|lucide':
    'reference/39-lovable-compatibility.md',
  
  'replit|monorepo|artifact|PORT|BASE_URL':
    'reference/40-replit-compatibility.md',
};
```

---

## Quality Gates for Claude UI Output

Before presenting any UI output as complete, Claude should verify:

```
Gate A — Functionality
  ✓ All happy path states work
  ✓ Loading state implemented
  ✓ Error state implemented
  ✓ Empty state implemented
  ✓ Edge cases handled (empty string, very long text, 0 items, 10,000 items)

Gate B — Accessibility
  ✓ All interactive elements are keyboard accessible
  ✓ Focus management correct (modals, menus)
  ✓ ARIA labels on icon-only buttons
  ✓ Form fields have associated labels
  ✓ Color is not the only indicator of state

Gate C — Responsive
  ✓ Works at 320px width (smallest phones)
  ✓ Works at 768px (tablet)
  ✓ Works at 1280px+ (desktop)
  ✓ Touch targets ≥ 44px on mobile

Gate D — Performance
  ✓ No unnecessary re-renders (memo, useCallback where warranted)
  ✓ No missing keys in lists
  ✓ Heavy computation in useMemo
  ✓ Images: width/height set, loading attribute correct

Gate E — Polish
  ✓ Animations respect prefers-reduced-motion
  ✓ Dark mode works
  ✓ Copy follows Verb+Noun formula
  ✓ Error messages explain what to do, not just what failed
  ✓ Destructive actions have confirmation
```

---

## Claude Skill Checklist

- [ ] Skill SKILL.md injected in system prompt for UI-heavy sessions
- [ ] Trigger keywords mapped to relevant reference files
- [ ] Output format includes all states (loading/error/empty/edge cases)
- [ ] Anti-pattern flags reviewed before accepting Claude's UI output
- [ ] All 5 Pre-Ship Quality Gates checked before calling UI complete
- [ ] Reference file consulted for each major UI domain in the task
