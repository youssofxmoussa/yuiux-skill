<div align="center">

<!-- Animated SVG header -->
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 900 200" width="900" height="200">
  <defs>
    <linearGradient id="bg" x1="0%" y1="0%" x2="100%" y2="100%">
      <stop offset="0%"   stop-color="#0f172a"/>
      <stop offset="100%" stop-color="#1e293b"/>
    </linearGradient>
    <linearGradient id="text-grad" x1="0%" y1="0%" x2="100%" y2="0%">
      <stop offset="0%"   stop-color="#60a5fa"/>
      <stop offset="50%"  stop-color="#a78bfa"/>
      <stop offset="100%" stop-color="#f472b6"/>
    </linearGradient>
    <linearGradient id="accent" x1="0%" y1="0%" x2="100%" y2="0%">
      <stop offset="0%"   stop-color="#3b82f6"/>
      <stop offset="100%" stop-color="#8b5cf6"/>
    </linearGradient>
    <filter id="glow">
      <feGaussianBlur stdDeviation="3" result="blur"/>
      <feMerge><feMergeNode in="blur"/><feMergeNode in="SourceGraphic"/></feMerge>
    </filter>
  </defs>

  <!-- Background -->
  <rect width="900" height="200" fill="url(#bg)" rx="16"/>

  <!-- Grid lines -->
  <g stroke="#334155" stroke-width="0.5" opacity="0.4">
    <line x1="0" y1="40"  x2="900" y2="40"/>
    <line x1="0" y1="80"  x2="900" y2="80"/>
    <line x1="0" y1="120" x2="900" y2="120"/>
    <line x1="0" y1="160" x2="900" y2="160"/>
    <line x1="150" y1="0" x2="150" y2="200"/>
    <line x1="300" y1="0" x2="300" y2="200"/>
    <line x1="450" y1="0" x2="450" y2="200"/>
    <line x1="600" y1="0" x2="600" y2="200"/>
    <line x1="750" y1="0" x2="750" y2="200"/>
  </g>

  <!-- Animated corner accents -->
  <rect x="16" y="16" width="32" height="2" fill="url(#accent)" rx="1">
    <animate attributeName="opacity" values="0.4;1;0.4" dur="2s" repeatCount="indefinite"/>
  </rect>
  <rect x="16" y="16" width="2" height="32" fill="url(#accent)" rx="1">
    <animate attributeName="opacity" values="0.4;1;0.4" dur="2s" repeatCount="indefinite"/>
  </rect>
  <rect x="852" y="16" width="32" height="2" fill="url(#accent)" rx="1">
    <animate attributeName="opacity" values="1;0.4;1" dur="2s" repeatCount="indefinite"/>
  </rect>
  <rect x="882" y="16" width="2" height="32" fill="url(#accent)" rx="1">
    <animate attributeName="opacity" values="1;0.4;1" dur="2s" repeatCount="indefinite"/>
  </rect>

  <!-- Floating dots -->
  <circle cx="60"  cy="100" r="3" fill="#60a5fa" opacity="0.6">
    <animate attributeName="cy" values="100;90;100" dur="3s" repeatCount="indefinite"/>
    <animate attributeName="opacity" values="0.6;1;0.6" dur="3s" repeatCount="indefinite"/>
  </circle>
  <circle cx="840" cy="100" r="3" fill="#f472b6" opacity="0.6">
    <animate attributeName="cy" values="100;110;100" dur="2.5s" repeatCount="indefinite"/>
    <animate attributeName="opacity" values="0.6;1;0.6" dur="2.5s" repeatCount="indefinite"/>
  </circle>
  <circle cx="450" cy="30" r="2" fill="#a78bfa" opacity="0.5">
    <animate attributeName="cx" values="445;455;445" dur="4s" repeatCount="indefinite"/>
  </circle>

  <!-- Main title -->
  <text
    x="450" y="95"
    text-anchor="middle"
    font-family="'Segoe UI', system-ui, sans-serif"
    font-size="52"
    font-weight="800"
    fill="url(#text-grad)"
    filter="url(#glow)"
    letter-spacing="-1"
  >
    YUIUX SKILL
    <animate attributeName="opacity" values="0;1" dur="0.8s" fill="freeze"/>
  </text>

  <!-- Subtitle -->
  <text
    x="450" y="128"
    text-anchor="middle"
    font-family="'Segoe UI', system-ui, sans-serif"
    font-size="16"
    fill="#94a3b8"
    letter-spacing="3"
  >
    THE COMPLETE UI/UX ENGINEERING REFERENCE
    <animate attributeName="opacity" values="0;1" dur="1.2s" fill="freeze"/>
  </text>

  <!-- Animated underline -->
  <line x1="300" y1="140" x2="600" y2="140" stroke="url(#accent)" stroke-width="2" rx="1">
    <animate attributeName="x1" values="450;300;300" dur="0.8s" fill="freeze"/>
    <animate attributeName="x2" values="450;600;600" dur="0.8s" fill="freeze"/>
  </line>

  <!-- Tag pills -->
  <rect x="254" y="155" width="70" height="24" rx="12" fill="#1e3a8a" opacity="0.8"/>
  <text x="289" y="171" text-anchor="middle" font-size="11" fill="#93c5fd" font-family="monospace">50K+ lines</text>

  <rect x="336" y="155" width="80" height="24" rx="12" fill="#312e81" opacity="0.8"/>
  <text x="376" y="171" text-anchor="middle" font-size="11" fill="#c4b5fd" font-family="monospace">41 topics</text>

  <rect x="428" y="155" width="90" height="24" rx="12" fill="#4a1942" opacity="0.8"/>
  <text x="473" y="171" text-anchor="middle" font-size="11" fill="#f9a8d4" font-family="monospace">WCAG 2.2 AA</text>

  <rect x="530" y="155" width="80" height="24" rx="12" fill="#064e3b" opacity="0.8"/>
  <text x="570" y="171" text-anchor="middle" font-size="11" fill="#6ee7b7" font-family="monospace">Production ✓</text>
</svg>

---

<p align="center">
  <img src="https://img.shields.io/badge/lines-50%2C000%2B-blue?style=for-the-badge&logo=markdown&logoColor=white" alt="50,000+ lines"/>
  <img src="https://img.shields.io/badge/topics-41-violet?style=for-the-badge" alt="41 topics"/>
  <img src="https://img.shields.io/badge/WCAG-2.2%20AA-green?style=for-the-badge&logo=accessibility" alt="WCAG 2.2 AA"/>
  <img src="https://img.shields.io/badge/platforms-Lovable%20%7C%20Replit%20%7C%20Claude-orange?style=for-the-badge" alt="Platforms"/>
  <img src="https://img.shields.io/badge/license-MIT-lightgrey?style=for-the-badge" alt="MIT License"/>
</p>

<p align="center">
  <strong>The most comprehensive UI/UX engineering skill ever written for AI agents.</strong><br/>
  Every pattern. Every anti-pattern. Every edge case. Production-grade, accessible, dark-mode-ready.
</p>

</div>

---

## What Is This?

**YUIUX Skill** is a structured, machine-readable reference for AI coding agents (Claude, Cursor, Lovable, Replit Agent, etc.) that need to produce world-class user interfaces. It covers every domain of UI/UX engineering with:

- ✅ **Copy-paste TypeScript/TSX code** for every pattern
- ✅ **Anti-pattern tables** — what not to do and why
- ✅ **Accessibility** — WCAG 2.2 AA throughout, not bolted on
- ✅ **Dark mode** — every token, every edge case
- ✅ **Checklists** — per-topic pre-ship gates
- ✅ **Platform guides** — Lovable, Replit, Claude-specific notes

---

## 📚 Skill Structure

```
yuiux-skill/
├── SKILL.md                    ← Master index + Pre-Ship Gate (A–E)
└── reference/
    ├── 00-quick-reference.md
    ├── 01-philosophy-and-principles.md
    ├── 02-typography-readability.md
    ├── 03-color-contrast-theming.md
    ├── 04-layout-spacing-grid.md
    ├── 05-motion-animation-microinteractions.md
    ├── 06-forms-inputs.md
    ├── 07-buttons-controls.md
    ├── 08-navigation-ia.md
    ├── 09-feedback-loading-empty-error-states.md
    ├── 10-accessibility-deep-dive.md
    ├── 11-modals-dialogs-overlays.md
    ├── 12-tables-data-display.md
    ├── 13-cards-components.md
    ├── 14-notifications-toasts.md
    ├── 15-search-filtering.md
    ├── 16-responsive-design.md
    ├── 17-performance-web-vitals.md
    ├── 18-icons-imagery.md
    ├── 19-design-tokens.md
    ├── 20-component-architecture.md
    ├── 21-data-visualization-charts.md
    ├── 22-authentication-ux.md
    ├── 23-onboarding-ux.md
    ├── 24-dashboard-design.md
    ├── 25-mobile-first-touch.md
    ├── 26-design-system-foundations.md
    ├── 27-content-strategy-microcopy.md
    ├── 28-user-testing-validation.md
    ├── 29-dark-mode-theming.md
    ├── 30-internationalization.md
    ├── 31-payment-checkout-ux.md
    ├── 32-settings-preferences.md
    ├── 33-destructive-actions-confirmation.md
    ├── 34-drag-drop-gestures.md
    ├── 35-infinite-scroll-pagination.md
    ├── 36-ai-ui-patterns.md
    ├── 37-design-handoff.md
    ├── 38-observability-analytics.md
    ├── 39-lovable-compatibility.md
    ├── 40-replit-compatibility.md
    └── 41-claude-compatibility.md
```

---

## 🗺️ Topic Map

<table>
<tr>
<th>Foundation</th>
<th>Components</th>
<th>Patterns</th>
<th>Platform</th>
</tr>
<tr>
<td>

- [Philosophy](reference/01-philosophy-and-principles.md)
- [Typography](reference/02-typography-readability.md)
- [Color & Contrast](reference/03-color-contrast-theming.md)
- [Layout & Spacing](reference/04-layout-spacing-grid.md)
- [Design Tokens](reference/19-design-tokens.md)
- [Design System](reference/26-design-system-foundations.md)
- [Component Arch](reference/20-component-architecture.md)

</td>
<td>

- [Forms & Inputs](reference/06-forms-inputs.md)
- [Buttons](reference/07-buttons-controls.md)
- [Navigation](reference/08-navigation-ia.md)
- [Modals & Overlays](reference/11-modals-dialogs-overlays.md)
- [Tables](reference/12-tables-data-display.md)
- [Cards](reference/13-cards-components.md)
- [Notifications](reference/14-notifications-toasts.md)
- [Search & Filter](reference/15-search-filtering.md)
- [Charts & Dataviz](reference/21-data-visualization-charts.md)

</td>
<td>

- [Loading States](reference/09-feedback-loading-empty-error-states.md)
- [Motion & Animation](reference/05-motion-animation-microinteractions.md)
- [Accessibility](reference/10-accessibility-deep-dive.md)
- [Dark Mode](reference/29-dark-mode-theming.md)
- [Responsive](reference/16-responsive-design.md)
- [Mobile & Touch](reference/25-mobile-first-touch.md)
- [Authentication](reference/22-authentication-ux.md)
- [Onboarding](reference/23-onboarding-ux.md)
- [Dashboard](reference/24-dashboard-design.md)
- [Settings](reference/32-settings-preferences.md)
- [Checkout](reference/31-payment-checkout-ux.md)
- [Drag & Drop](reference/34-drag-drop-gestures.md)
- [Infinite Scroll](reference/35-infinite-scroll-pagination.md)
- [AI UI Patterns](reference/36-ai-ui-patterns.md)
- [Destructive Actions](reference/33-destructive-actions-confirmation.md)
- [Microcopy](reference/27-content-strategy-microcopy.md)
- [Performance](reference/17-performance-web-vitals.md)
- [i18n / RTL](reference/30-internationalization.md)
- [Observability](reference/38-observability-analytics.md)
- [User Testing](reference/28-user-testing-validation.md)
- [Icons & Imagery](reference/18-icons-imagery.md)

</td>
<td>

- [Lovable](reference/39-lovable-compatibility.md)
- [Replit](reference/40-replit-compatibility.md)
- [Claude](reference/41-claude-compatibility.md)
- [Design Handoff](reference/37-design-handoff.md)

</td>
</tr>
</table>

---

## ⚡ Quick Start

### For AI Agents (Claude, Cursor, etc.)

Paste into your system prompt:

```
You are an expert UI/UX engineer. Before building any UI, always consult the
YUIUX skill at yuiux-skill/SKILL.md and the relevant reference files in
yuiux-skill/reference/. Apply every principle proactively — especially the
Pre-Ship Gate checklist (Gates A–E in SKILL.md) before any UI is complete.
```

### For Developers

1. Clone this repo into your project
2. Open `SKILL.md` for the master index and Pre-Ship Gates
3. Jump to the specific reference file for your current task
4. Copy the code patterns, apply the checklist at the bottom

### For Lovable Projects

All code in this skill is optimized for the Lovable stack:
- **Components**: shadcn/ui
- **Icons**: lucide-react
- **Animation**: Framer Motion
- **Styling**: Tailwind CSS
- **Data**: TanStack Query

→ [Lovable Compatibility Guide](reference/39-lovable-compatibility.md)

---

## 🚦 Pre-Ship Quality Gates

Every UI built with this skill is validated against 5 gates before shipping:

| Gate | What It Checks |
|------|---------------|
| **A — Functionality** | All states: loading, error, empty, edge cases |
| **B — Accessibility** | WCAG 2.2 AA: keyboard, ARIA, contrast, focus |
| **C — Responsive** | 320px → 1440px, touch targets ≥ 44px |
| **D — Performance** | LCP ≤ 2.5s, CLS ≤ 0.1, INP ≤ 200ms |
| **E — Polish** | Copy, motion, dark mode, microcopy |

Full gate spec in [SKILL.md](SKILL.md).

---

## 🔥 Top Anti-Patterns Caught by This Skill

```
🚫 Icon without aria-hidden or aria-label
🚫 Form error without resolution instruction ("Enter a valid email like name@example.com")
🚫 Button copy: "Submit", "OK", "Yes", "Confirm"
🚫 Modal without focus trap or Escape to close
🚫 Loading state missing — only happy path coded
🚫 Hardcoded color value instead of CSS token variable
🚫 "Are you sure?" + "Yes/No" for destructive confirmation
🚫 Font size < 16px on mobile inputs (triggers iOS zoom bug)
🚫 Animation without prefers-reduced-motion check
🚫 Hero image with loading="lazy" (kills LCP score)
🚫 Image without width/height attributes (causes CLS)
🚫 div with onClick instead of button (keyboard inaccessible)
🚫 Placeholder text used as the field label
🚫 Dark mode using hardcoded white/black (not CSS variables)
🚫 Touch target < 44×44px on mobile
```

---

## 📊 Stats

```
Total reference files:    41
Estimated total lines:    50,000+
Code examples:            400+
Anti-pattern tables:      41
Per-topic checklists:     41
Platform guides:          3 (Lovable, Replit, Claude)
WCAG coverage:            AA (A/AA all checkpoints)
Dark mode coverage:       100% of color-bearing components
```

---

## 🧩 Integration Examples

### Inject into Replit Agent

Add to your project's `.agents/skills/yuiux/SKILL.md` path and the agent reads it automatically when performing UI work.

### Inject into Claude Projects

```xml
<skill path="yuiux-skill/SKILL.md">
  Read this skill before any UI work.
  Always apply the Pre-Ship Gate checklist before presenting UI.
  Reference the specific topic file for each domain.
</skill>
```

### Use in Cursor/Windsurf

Add to `.cursorrules` or `.windsurfrules`:
```
For all UI work, reference yuiux-skill/SKILL.md and the relevant
yuiux-skill/reference/*.md file for the component type being built.
```

---

## 🧠 Philosophy

This skill is built on one core insight:

> **Design decisions are code decisions.** Every pixel, every millisecond, every word in a UI is a decision that must be made consistently, intentionally, and with users in mind. Most UI quality failures are not a lack of skill — they are a lack of a shared, explicit standard.

This skill is that standard — explicit enough that any agent or developer can apply it consistently, comprehensive enough that it covers every real production scenario.

---

## 📖 License

MIT License — free to use, fork, and extend.

---

<div align="center">

**Built by [Youssof Moussa](https://t.me/youssofxmoussa)**

*Ship UIs that are accessible, performant, and beautiful — by default.*

<br/>

[![Telegram](https://img.shields.io/badge/Telegram-@youssofxmoussa-2CA5E0?style=for-the-badge&logo=telegram&logoColor=white)](https://t.me/youssofxmoussa)

</div>
