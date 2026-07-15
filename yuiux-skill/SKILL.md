# YUIUX — The Ultimate UI/UX Master Skill

> **Version:** 3.0.0 · **Author:** youssofxmoussa · **License:** MIT  
> **Compatible with:** Claude · Lovable · Replit · Cursor · Windsurf · Any AI Builder  
> **Contact:** [t.me/youssofxmoussa](https://t.me/youssofxmoussa)

---

```
██╗   ██╗██╗   ██╗██╗██╗   ██╗██╗  ██╗
╚██╗ ██╔╝██║   ██║██║██║   ██║╚██╗██╔╝
 ╚████╔╝ ██║   ██║██║██║   ██║ ╚███╔╝ 
  ╚██╔╝  ██║   ██║██║██║   ██║ ██╔██╗ 
   ██║   ╚██████╔╝██║╚██████╔╝██╔╝ ██╗
   ╚═╝    ╚═════╝ ╚═╝ ╚═════╝ ╚═╝  ╚═╝
The Ultimate UI/UX Intelligence System
```

---

## What This Skill Is

YUIUX is the most comprehensive UI/UX skill ever assembled for AI-powered development tools. It covers every domain of UI/UX engineering — from design tokens and typography, to accessibility, motion, authentication, AI interfaces, and platform-specific patterns.

Every rule in this skill states:
- **What** the error or pattern is
- **Why** it matters (which heuristic, WCAG criterion, or psychological principle it violates)
- **Exactly how** to fix it, with code

This is a **working library**, not a manifesto.

---

## Platform Compatibility

### 🟣 Lovable
Lovable generates React + Tailwind + shadcn/ui apps. When using this skill with Lovable:
- All component references use shadcn/ui naming (`Button`, `Input`, `Card`, `Dialog`, etc.)
- CSS custom properties use shadcn/ui token conventions (`--primary`, `--secondary`, `--background`, `--foreground`, `--muted`, `--accent`, `--destructive`)
- Motion uses Framer Motion (pre-installed in Lovable)
- Icon references use `lucide-react`
- See `reference/39-lovable-compatibility.md` for complete Lovable-specific guidance

### 🔵 Claude (Anthropic)
Claude (API, Claude.ai, Claude Code) generates framework-agnostic code. When using with Claude:
- Rules are stated framework-independently; adapt syntax to whatever stack is in context
- Claude should read the relevant reference file for each domain before generating UI
- Use the Pre-Ship Gate before finalizing any UI output
- See `reference/41-claude-compatibility.md` for Claude-specific prompting patterns

### 🟢 Replit
Replit's agent generates React + Vite + Express apps in a monorepo. When using with Replit:
- CSS custom properties align with the Vite app's `index.css` token system
- Components follow React + TypeScript conventions
- API hooks come from `@workspace/api-client-react`
- See `reference/40-replit-compatibility.md` for Replit-specific patterns

---

## The Five Pillars (Apple HIG, Web-Translated)

Every decision in this skill traces back to five pillars:

### 1. Clarity
Every screen has one obvious primary action. Text is legible at every supported size. Icons never stand alone without a label or universally understood meaning. Nothing exists on screen without a reason a user could state back to you.

**Violations:** Icon-only buttons with no tooltip, six equally-weighted CTAs, 11px body text, unlabeled graph axes.

### 2. Deference  
Content is the interface. Decorative chrome (gradients, borders, shadows) recedes until needed to disambiguate structure. A UI should be memorable for how easy it made the task, not for its decoration.

**Violations:** Heavy card borders that compete with content, decorative gradients behind critical text, shadow layers that don't communicate elevation.

### 3. Depth
Visual layers communicate hierarchy and spatial relationships. A modal is genuinely "in front of" the page, proven by its transition. Depth is functional — it tells users what is temporary vs. permanent.

**Violations:** Modals that appear without transition, sheets that don't dim the page behind them, tooltips that appear instantly with no spatial relationship.

### 4. Feedback
Every action produces an immediate, proportionate response. A button press depresses. A save shows confirmation. A delete shows what was removed and how to undo it. Silence after an action is the #1 way software feels broken.

**Violations:** Buttons that do nothing visible for 2 seconds, forms that submit silently, deletes with no confirmation or undo.

### 5. Consistency
The same control looks and behaves the same everywhere. A user predicts a new screen's behavior from screens they've already used. Terminology never shifts mid-app.

**Violations:** "Cancel" in one place, "Dismiss" in another for identical flows; primary button on left in one dialog, right in another; "workspace" vs "project" used interchangeably.

---

## Core Rule

> **Every interactive element must communicate three things at all times: what it is, what will happen if used, and what just happened.**

Nearly every entry in this skill is a violation of one of those three, at a specific layer (visual, semantic, timing, or copy).

---

## Reference Index

Read the file that matches what you're building. Always read this index first; read the specific file before writing code.

### Foundation
| # | File | Covers |
|---|------|--------|
| 00 | `reference/00-quick-reference.md` | Cheat-sheet: all rules in one page, Pre-Ship Gate, emergency fixes |
| 01 | `reference/01-philosophy-and-principles.md` | Nielsen heuristics, Laws of UX, Apple HIG, Gestalt, mental models |
| 02 | `reference/02-typography-readability.md` | Type scale, line length, hierarchy, font pairing, legibility |
| 03 | `reference/03-color-contrast-theming.md` | Palette, WCAG math, dark mode, semantic color, color psychology |
| 04 | `reference/04-layout-spacing-grid.md` | Spacing scales, grid, alignment, visual rhythm, responsive breakpoints |
| 05 | `reference/05-motion-animation-microinteractions.md` | Easing, duration budgets, transitions, micro-interactions, reduced motion |

### Components & Patterns
| # | File | Covers |
|---|------|--------|
| 06 | `reference/06-forms-inputs.md` | Labels, validation, error recovery, multi-step, autofill, autosave |
| 07 | `reference/07-buttons-controls.md` | Button hierarchy, all states, hit targets, toggles, sliders, checkboxes |
| 08 | `reference/08-navigation-ia.md` | Nav patterns, breadcrumbs, tabs, sidebars, deep linking, back-button |
| 09 | `reference/09-feedback-loading-empty-error-states.md` | Skeletons, spinners, empty states, error boundaries, optimistic UI |
| 11 | `reference/11-modals-dialogs-overlays.md` | Modals, drawers, popovers, tooltips, focus traps |
| 12 | `reference/12-tables-data-display.md` | Accessible tables, sorting, selection, pagination, mobile strategies |
| 13 | `reference/13-cards-components.md` | Card variants, grid, badge, avatar |
| 14 | `reference/14-notifications-toasts.md` | Toast system, alert banners, notification bell |
| 15 | `reference/15-search-filtering.md` | Search UX, filters, facets, sort, zero-result states |
| 21 | `reference/21-data-visualization-charts.md` | Chart types, Recharts, accessible charts, tooltips |
| 31 | `reference/31-payment-checkout-ux.md` | Cart, checkout flow, Stripe elements, trust signals |
| 32 | `reference/32-settings-preferences.md` | Settings organization, immediate vs explicit save, danger zone |
| 33 | `reference/33-destructive-actions-confirmation.md` | Confirm dialogs, undo patterns, type-to-confirm |
| 34 | `reference/34-drag-drop-gestures.md` | dnd-kit, keyboard DnD, file drop zones |
| 35 | `reference/35-infinite-scroll-pagination.md` | Virtual scroll, intersection observer, load more |

### Accessibility & Performance
| # | File | Covers |
|---|------|--------|
| 10 | `reference/10-accessibility-deep-dive.md` | Keyboard, screen reader, ARIA, focus management, WCAG 2.2 |
| 16 | `reference/16-responsive-design.md` | Mobile-first, fluid spacing, clamp(), images, print |
| 17 | `reference/17-performance-web-vitals.md` | LCP, CLS, INP, code splitting, lazy loading |
| 25 | `reference/25-mobile-first-touch.md` | Touch targets, gestures, safe areas, bottom sheet |

### UX Writing & Patterns
| # | File | Covers |
|---|------|--------|
| 22 | `reference/22-authentication-ux.md` | Sign-up, login, password reset, MFA, session expiry |
| 23 | `reference/23-onboarding-ux.md` | Empty-state onboarding, progressive disclosure, checklists |
| 24 | `reference/24-dashboard-design.md` | KPI cards, date range, real-time updates, glanceability |
| 27 | `reference/27-content-strategy-microcopy.md` | Button copy, error messages, empty states, number formatting |
| 28 | `reference/28-user-testing-validation.md` | Usability testing, heuristic evaluation, A/B testing |
| 30 | `reference/30-internationalization.md` | RTL, text expansion, Intl API, i18next |
| 36 | `reference/36-ai-ui-patterns.md` | Streaming text, chat UI, AI loading states, uncertainty |

### Design Systems
| # | File | Covers |
|---|------|--------|
| 18 | `reference/18-icons-imagery.md` | Icon systems, SVG, accessibility, OG images |
| 19 | `reference/19-design-tokens.md` | Three-layer tokens, Tailwind config, CSS variables |
| 20 | `reference/20-component-architecture.md` | Compound components, composition, state patterns |
| 26 | `reference/26-design-system-foundations.md` | Token hierarchy, versioning, governance, documentation |
| 29 | `reference/29-dark-mode-theming.md` | Class-based toggle, FOUC prevention, image adaptation |
| 37 | `reference/37-design-handoff.md` | Figma tokens, interaction specs, redlines |
| 38 | `reference/38-observability-analytics.md` | Sentry, PostHog, feature flags, privacy |
| 42 | `reference/42-css-architecture.md` | CSS layers, Tailwind, CSS Modules, container queries |

### Platform Compatibility
| # | File | Covers |
|---|------|--------|
| 39 | `reference/39-lovable-compatibility.md` | shadcn/ui, Framer Motion, lucide-react, TanStack Query |
| 40 | `reference/40-replit-compatibility.md` | Monorepo, PORT env, BASE_URL, artifact routing |
| 41 | `reference/41-claude-compatibility.md` | Prompting patterns, trigger map, quality gates |

---

## Pre-Ship Gate

Before calling **any** UI work finished — before `presentArtifact`, before showing the user — walk the rendered app through every row below.

> **Rule:** A checked row means you verified it in the running browser, not just in code.

### Gate A — Foundations
- [ ] **Typography**: body text ≥16px on desktop, ≥15px on mobile; line-height ≥1.5; max line-length ≤75ch → `02`
- [ ] **Color**: every text/background pair passes WCAG AA (4.5:1 normal, 3:1 large); no info conveyed by color alone → `03`
- [ ] **Layout**: no horizontal scroll at 320px; grid consistent; breathing room between sections → `04`
- [ ] **Motion**: all transitions respect `prefers-reduced-motion`; nothing loops without user intent → `05`

### Gate B — Components
- [ ] **Forms**: every input has a real `<label>`; errors name the field and the fix; input never wiped on failed submit → `06`
- [ ] **Controls**: no clickable `<div>`; every state designed (default/hover/focus/active/disabled/loading); hit targets ≥44px touch → `07`
- [ ] **Navigation**: current location always visible; every reachable state has a way forward; back behaves correctly → `08`
- [ ] **States**: loading/empty/error exist for every async view; no blank white screens; error boundary catches exceptions → `09`
- [ ] **Modals**: focus trapped inside; Escape closes; returns focus to trigger on close → `15`

### Gate C — Accessibility
- [ ] **Keyboard**: full keyboard operability — every action achievable without mouse → `10`
- [ ] **Focus**: visible focus rings on every focusable element; skip-link present → `10`
- [ ] **Screen reader**: icon-only buttons have `aria-label`; live regions announce changes; heading hierarchy intact → `10`
- [ ] **WCAG**: passes axe/Lighthouse with zero critical violations → `10`

### Gate D — Quality
- [ ] **Performance**: perceived feedback within 100ms of any action; heavy assets lazy-loaded; no layout shift → `11`
- [ ] **Responsive**: verified at 320px / 768px / 1280px; touch targets ≥44px; no text truncation that hides meaning → `12`
- [ ] **Copy**: no message blames the user; every error is actionable; terminology consistent app-wide → `14`
- [ ] **Destructive actions**: confirmed OR undoable; irreversible never fires on one accidental click → `24`
- [ ] **Consistency**: same component looks and behaves same in every occurrence → `25`

### Gate E — Platform
- [ ] **Lovable**: if Lovable-generated — all tokens from `globals.css`, no hardcoded hex in components → `39`
- [ ] **Replit**: if Replit-generated — tokens in `index.css`, hooks from `@workspace/api-client-react` → `40`
- [ ] **i18n**: text not clipped in 30% longer languages; RTL layout if applicable → `22`
- [ ] **Privacy**: consent obtained before tracking; data minimization in collection → `23`

> **If any box is unchecked, that is a defect, not a nice-to-have — fix it before presenting the work.**

---

## UX Error Taxonomy (Master List)

Every error category with severity rating and fix reference:

| Category | Examples | Severity | Ref |
|----------|----------|----------|-----|
| Form Labels | Missing `<label>`, placeholder-only fields | 🔴 Critical | `06` |
| Form Validation | Errors on keydown, wipe on failed submit, no inline errors | 🔴 Critical | `06` |
| Button States | No hover/focus/disabled state, clickable `<div>` | 🔴 Critical | `07` |
| Keyboard Access | Tab traps, no keyboard nav, clickable non-focusable elements | 🔴 Critical | `10` |
| Color Contrast | Text below 4.5:1 AA | 🔴 Critical | `03` |
| Focus Indicators | `outline: none` with no replacement | 🔴 Critical | `10` |
| Alt Text | Meaningful images without `alt`, decorative with text alt | 🔴 Critical | `10` |
| Error Messages | "Error 422", user-blaming copy, no recovery path | 🟠 High | `14` |
| Loading States | No feedback for >200ms operations | 🟠 High | `09` |
| Empty States | Blank screen with no guidance | 🟠 High | `09` |
| Touch Targets | Targets <44×44px on mobile | 🟠 High | `12` |
| Destructive Actions | Delete with no confirmation or undo | 🟠 High | `24` |
| Navigation | No current location indicator, broken back button | 🟠 High | `08` |
| Responsive | Horizontal scroll, content overlap on mobile | 🟠 High | `12` |
| Modal Focus | Focus not trapped, doesn't close on Escape | 🟠 High | `15` |
| Toast/Alert Placement | Toast covering primary action, auto-dismissing errors | 🟡 Medium | `15` |
| Typography | Line length >80ch, line-height <1.4, tiny body text | 🟡 Medium | `02` |
| Motion | No `prefers-reduced-motion` support, >500ms transitions | 🟡 Medium | `05` |
| Onboarding | No empty-state guidance, no first-run help | 🟡 Medium | `13` |
| Search | No zero-result state, no search suggestions | 🟡 Medium | `16` |
| i18n | Hardcoded strings, RTL layout not mirrored | 🟡 Medium | `22` |
| Privacy | Opt-in not before tracking, invasive permission requests | 🟡 Medium | `23` |
| Consistency | Same concept, different term in different places | 🟡 Medium | `25` |
| Performance | LCP >2.5s, CLS >0.1, no lazy loading | 🟡 Medium | `11` |
| Microcopy | Vague CTAs ("Click here"), jargon in errors | 🟡 Medium | `14` |
| Settings | Destructive settings without warning, confusing grouping | 🟢 Low | `21` |
| Auth UX | Unhelpful password rules, silent session expiry | 🟢 Low | `19` |
| File Upload | No progress, no retry on failure | 🟢 Low | `20` |
| Data Tables | No sort affordance, no pagination feedback | 🟢 Low | `17` |

---

## How to Use This Skill

### While Building
1. Identify what you're building (form, nav, modal, table, upload, auth screen)
2. Read the matching reference file's checklist **before** writing code
3. It is cheaper to build it right than retrofit it

### While Reviewing
1. Run the **Pre-Ship Gate** against the actual rendered UI (not just the code)
2. Check the UX Error Taxonomy for anything in scope
3. Verify every gate row in a real browser or device

### When Something "Feels Off"
Read `reference/01-philosophy-and-principles.md` — most vague "this feels cheap" complaints map to a violation of clarity, feedback, or consistency.

### When Auditing an Existing App
Work through `reference/26-anti-pattern-catalog.md` top to bottom. It's organized so you can grep for symptoms ("blank screen", "double submit", "tiny tap target") and jump straight to the fix.

### For Brand Work
Start with `reference/27-brand-identity.md`, then `reference/28-design-tokens-architecture.md`, then `reference/29-component-library.md`.

### For a New Design System
Read `reference/28-design-tokens-architecture.md` → `reference/25-design-systems-consistency.md` → `reference/35-shadcn-tailwind-integration.md` in that order.

---

## Detection Decision Tree

When you find a UX issue, run this tree to classify and fix it:

```
Issue found
├── Does it block any user from completing a task?
│   └── YES → 🔴 Critical — fix before ANY other work
│       ├── Keyboard inaccessible? → reference/10
│       ├── Form unsubmittable? → reference/06
│       ├── CTA unclickable? → reference/07
│       └── Error with no recovery? → reference/24
│
├── Does it cause user confusion or data loss risk?
│   └── YES → 🟠 High — fix in current sprint
│       ├── Missing loading state? → reference/09
│       ├── Destructive action unfenced? → reference/24
│       ├── Mobile unusable? → reference/12
│       └── Error message unhelpful? → reference/14
│
├── Does it degrade quality or professionalism?
│   └── YES → 🟡 Medium — fix before launch
│       ├── Typography issue? → reference/02
│       ├── Color/contrast? → reference/03
│       ├── Motion excessive? → reference/05
│       └── Copy unclear? → reference/14
│
└── Is it an enhancement or nice-to-have?
    └── YES → 🟢 Low — backlog
```

---

## AI Integration Patterns

When an AI agent (Claude, Lovable, Replit) uses this skill, it should:

1. **Before generating any UI**: Read the Pre-Ship Gate checklist for scope
2. **Before writing a form**: Read `reference/06-forms-inputs.md`
3. **Before writing any component**: Read `reference/07-buttons-controls.md` for the state specs
4. **Before finalizing**: Walk through Gate A-E above
5. **On any "make it look better" request**: Read `reference/01-philosophy-and-principles.md` first, not `reference/26-anti-pattern-catalog.md` — principles before patterns

### Prompt Injection Pattern (for Claude/Lovable/Replit)

When using YUIUX as a system context, inject this at the top of any UI generation prompt:

```
[YUIUX ACTIVE]
Before generating any UI, apply YUIUX rules:
- Every interactive element must communicate: what it is, what happens, what happened
- All form inputs must have real <label> elements
- All buttons must have all 6 states designed (default/hover/focus/active/disabled/loading)
- Minimum touch target: 44×44px
- Color contrast: ≥4.5:1 (text), ≥3:1 (large text/UI)
- No information conveyed by color alone
- All async operations must have loading + error + empty states
- prefers-reduced-motion must be respected
- No clickable <div> or <span> — use semantic elements
- Error copy must be actionable, never blame the user
```

---

---

*YUIUX — Built by [youssofxmoussa](https://t.me/youssofxmoussa). Every pixel has a reason. Every interaction has a response. Every user deserves clarity.*
