# 00 — YUIUX Quick Reference

> The single-page cheat sheet. When you need the rule NOW, this file has it. For depth, go to the numbered reference file.

---

## Emergency Pre-Ship Checklist (60 seconds)

```
FORMS
[ ] Every <input> has a <label> with matching htmlFor/id
[ ] Errors shown inline, name the field, suggest the fix
[ ] Submit button shows loading state during async
[ ] Failed submit preserves entered data

BUTTONS & CONTROLS  
[ ] Every button has 6 states: default/hover/focus/active/disabled/loading
[ ] No <div onClick> — use <button> or <a> semantically
[ ] Hit targets ≥44×44px on mobile (padding trick if needed)
[ ] Icon-only controls have aria-label

ACCESSIBILITY
[ ] Tab through entire page — can you complete every task?
[ ] Focus ring visible on every focusable element
[ ] WCAG AA contrast: 4.5:1 normal text, 3:1 large (≥18px bold or ≥24px)
[ ] Meaningful images have alt text; decorative have alt=""
[ ] Modals trap focus; Escape closes; returns focus to trigger

STATES
[ ] Every async view has: loading skeleton, error message, empty state
[ ] No blank white screen ever shown to user

PERFORMANCE
[ ] Action response visible within 100ms
[ ] Images lazy-loaded below the fold
[ ] No layout shift after initial paint

MOBILE
[ ] No horizontal scroll at 320px width
[ ] Touch targets ≥44×44px
[ ] Text readable without zooming (≥15px body)

COPY
[ ] No error message blames the user
[ ] Every error tells what happened AND what to do
[ ] Same concept = same word, everywhere in the app

DESTRUCTIVE ACTIONS
[ ] Delete/reset/deactivate requires confirm OR provides undo
[ ] Confirmation copy describes what will be lost
```

---

## The 10 Nielsen Heuristics — One Line Each

| # | Heuristic | Violation Example | Fix |
|---|-----------|------------------|-----|
| 1 | Visibility of system status | Button does nothing for 2s | Show loading state immediately |
| 2 | Match real world | "Invalidate session" button | Say "Log out" |
| 3 | User control & freedom | Delete with no undo | Add undo or confirm |
| 4 | Consistency & standards | "Save" in one place, "Update" elsewhere | Use one term |
| 5 | Error prevention | Required field only marked after submit | Mark required upfront |
| 6 | Recognition not recall | No breadcrumb | Show current location always |
| 7 | Flexibility & efficiency | Power user stuck with wizard | Add keyboard shortcuts |
| 8 | Aesthetic & minimalist | 3 sidebars, 8 nav items visible | Remove, hide, or group |
| 9 | Help recover from errors | "Error occurred" | Describe problem + action |
| 10 | Help & documentation | No empty state guidance | Add contextual help text |

---

## WCAG 2.2 — Critical Criteria Quick Reference

| Criterion | What it requires | Level |
|-----------|-----------------|-------|
| 1.1.1 Non-text content | Images need alt text | A |
| 1.3.1 Info & relationships | Structure via markup, not just visual | A |
| 1.3.3 Sensory characteristics | Don't say "click the red button" | A |
| 1.4.1 Use of color | Color not sole means of conveying info | A |
| 1.4.3 Contrast (minimum) | 4.5:1 normal, 3:1 large text | AA |
| 1.4.4 Resize text | Text resizable to 200% without loss | AA |
| 1.4.10 Reflow | Content visible at 320px without scroll | AA |
| 1.4.11 Non-text contrast | UI components: 3:1 against adjacent | AA |
| 1.4.13 Content on hover/focus | Hover content dismissible, persistent, hoverable | AA |
| 2.1.1 Keyboard | All functionality via keyboard | A |
| 2.1.2 No keyboard trap | Focus not stuck in component | A |
| 2.4.3 Focus order | Meaningful tab order | A |
| 2.4.7 Focus visible | Keyboard focus indicator visible | AA |
| 2.4.11 Focus not obscured | Focused element not fully hidden (2.2) | AA |
| 2.5.3 Label in name | Accessible name includes visible label | A |
| 2.5.8 Target size minimum | 24×24px minimum, 44×44 recommended (2.2) | AA |
| 3.3.1 Error identification | Error identified in text | A |
| 3.3.2 Labels or instructions | Instructions provided for inputs | A |
| 3.3.3 Error suggestion | Errors suggest fix when possible | AA |
| 3.3.4 Error prevention (legal) | Legal/financial: reversible or confirmable | AA |
| 4.1.2 Name, role, value | ARIA roles/states/properties set correctly | A |
| 4.1.3 Status messages | Status delivered to AT without focus | AA |

---

## Color Contrast Quick Calculator

```
Luminance of a color:
  L = 0.2126 * R + 0.7152 * G + 0.0722 * B
  (where R,G,B are linearized: v/12.92 if v≤0.03928, else ((v+0.055)/1.055)^2.4)

Contrast ratio = (L1 + 0.05) / (L2 + 0.05)  [L1 = lighter]

Pass thresholds:
  Normal text:  AA = 4.5:1  |  AAA = 7:1
  Large text:   AA = 3:1    |  AAA = 4.5:1  (≥18px normal OR ≥14px bold)
  UI elements:  AA = 3:1    (buttons, inputs, icons)
```

Common failures:
- `#767676` on `#fff` → 4.48:1 ❌ (barely fails AA)
- `#6b7280` (gray-500) on `#fff` → 4.63:1 ✅
- `#9ca3af` (gray-400) on `#fff` → 2.85:1 ❌
- `#374151` (gray-700) on `#fff` → 10.7:1 ✅

---

## Spacing Scale (8px Base System)

```
4px   — micro gaps (icon to label, badge padding)
8px   — tight (list item internal padding, inline gaps)
12px  — compact (input padding, small card padding)
16px  — base (standard component padding)
20px  — comfortable (form group gap)
24px  — section internal padding
32px  — section gap
48px  — major section gap
64px  — hero/feature spacing
96px  — page-level vertical rhythm
```

---

## Type Scale (Major Third — 1.25)

```
10px — micro label (avoid if possible)
12px — caption, badge text
14px — small body, helper text
16px — base body (minimum for main content)
20px — lead/intro paragraph
24px — h4
30px — h3
38px — h2
48px — h1
60px — display
76px — hero
```

Line height rules:
- Body text: 1.5–1.7
- Headings: 1.1–1.25
- Max line length: 65–75ch (45–55ch for narrow columns)

---

## Button States Spec (Copy This)

```css
/* Every button MUST implement all 6 states */

/* 1. Default */
.btn { background: var(--primary); color: var(--primary-foreground); }

/* 2. Hover */
.btn:hover { background: var(--primary-hover); }

/* 3. Focus */
.btn:focus-visible { 
  outline: 2px solid var(--ring); 
  outline-offset: 2px; 
}

/* 4. Active */
.btn:active { 
  background: var(--primary-active); 
  transform: translateY(1px); 
}

/* 5. Disabled */
.btn:disabled { 
  opacity: 0.5; 
  cursor: not-allowed; 
  pointer-events: none; 
}

/* 6. Loading */
.btn[aria-busy="true"] { 
  cursor: wait; 
  /* show spinner, hide or fade text */ 
}
```

---

## Form Error Pattern (Copy This)

```tsx
// CORRECT: inline error, never wipes input, accessible
<div>
  <label htmlFor="email">Email address</label>
  <input
    id="email"
    type="email"
    value={value}
    aria-invalid={!!error}
    aria-describedby={error ? "email-error" : undefined}
    onChange={(e) => setValue(e.target.value)}
  />
  {error && (
    <p id="email-error" role="alert" className="error-msg">
      {error} {/* "Enter a valid email address like name@example.com" */}
    </p>
  )}
</div>
```

Error message formula: **[What's wrong] + [How to fix it]**
- ❌ "Invalid email"
- ✅ "Enter a valid email address like name@example.com"
- ❌ "You didn't fill this out"
- ✅ "Email address is required"

---

## Loading State Hierarchy

```
< 100ms  → No indicator (user doesn't perceive delay)
100–300ms → Pulse/shimmer on affected element only
300ms–1s  → Skeleton screen matching layout
1s–3s     → Progress bar or spinner + message
> 3s      → Progress + estimated time + cancel option
```

---

## Hit Target Formula

```
Visual size: can be smaller
Interactive area: minimum 44×44px

CSS approach:
.small-btn {
  /* visual */
  width: 24px;
  height: 24px;
  
  /* interactive area via padding */
  padding: 10px;
  
  /* or via pseudo-element */
  position: relative;
}
.small-btn::before {
  content: '';
  position: absolute;
  inset: -10px; /* expands click area by 10px each side */
}
```

---

## Motion Budget

```
Micro-interaction (button press, toggle):  50–100ms
UI transition (panel open, dropdown):      150–250ms
Page/route transition:                     250–350ms
Complex animation (chart draw):            500–800ms
Ambient animation (hero):                  2000ms+ with infinite loop

prefers-reduced-motion MUST disable:
- All transform-based animations
- Opacity transitions > 100ms  
- Background/gradient animations
- Parallax effects
- Auto-playing videos
```

```css
@media (prefers-reduced-motion: reduce) {
  *, *::before, *::after {
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.01ms !important;
    scroll-behavior: auto !important;
  }
}
```

---

## Token Naming Convention

```css
/* PRIMITIVE — raw values, never used directly in components */
--color-blue-600: #2563EB;
--color-gray-100: #F3F4F6;
--spacing-4: 1rem;

/* SEMANTIC — meaning aliases, used in base styles */
--color-primary: var(--color-blue-600);
--color-background: var(--color-white);
--color-text: var(--color-gray-900);

/* COMPONENT — component-specific, override per-component */
--button-bg: var(--color-primary);
--input-border: var(--color-gray-300);
--card-shadow: var(--shadow-sm);
```

**Rule:** Never use a primitive token in a component. Always go through semantic.

---

## Microcopy Formulas

### Error messages
```
Formula: [What happened] + [Why] + [How to fix]

"Password must be at least 8 characters"              ← what + fix
"Your session expired — please sign in again"         ← what + why
"File too large. Maximum size is 5MB"                 ← what + constraint
"Something went wrong. Try again or contact support"  ← what + options
```

### Button labels
```
Action buttons: Verb + Noun
  ❌ "Submit"     ✅ "Create account"
  ❌ "OK"         ✅ "Delete project"
  ❌ "Confirm"    ✅ "Send $50"

Destructive confirm: Name what's lost
  ❌ "Delete"     ✅ "Delete workspace and all 14 projects"
  ❌ "Yes"        ✅ "Yes, cancel my subscription"
```

### Empty state copy
```
Formula: What's here + Why it's empty + What to do

"No projects yet"
"Create your first project to get started"
[Create project button]
```

### Placeholder text
```
❌ Don't use placeholder as label (disappears on focus)
❌ Don't use "Enter your email..." (states the obvious)
✅ Use for format hints: "MM/DD/YYYY" or "name@company.com"
✅ Leave empty if label is self-explanatory
```

---

## shadcn/ui Token Reference (for Lovable/Replit)

```css
/* These CSS custom properties are set in globals.css / index.css */
--background          /* page background */
--foreground          /* primary text */
--primary             /* brand primary color */
--primary-foreground  /* text on primary background */
--secondary           /* secondary action color */
--secondary-foreground
--muted               /* muted/subdued areas */
--muted-foreground    /* placeholder, helper text */
--accent              /* hover states, highlights */
--accent-foreground
--destructive         /* error, delete actions */
--destructive-foreground
--border              /* borders */
--input               /* input field border */
--ring                /* focus ring color */
--radius              /* border radius base */
```

---

## Laws of UX — Decision Shortcuts

| Law | Principle | Apply when |
|-----|-----------|-----------|
| Fitts's Law | Larger + closer = easier to click | Sizing CTAs, primary actions |
| Hick's Law | More choices = longer decision time | Navigation, dropdown lists |
| Miller's Law | Working memory holds ~7 items | Menu items, form fields per step |
| Jakob's Law | Users prefer familiar patterns | Navigation placement, UI patterns |
| Peak-End Rule | Judged by peak + last moment | Onboarding end, success states |
| Aesthetic-Usability | Attractive = perceived as more usable | Every UI surface |
| Doherty Threshold | Response >400ms = frustration | Loading states, feedback |
| Tesler's Law | Complexity can't be removed, only moved | Simplification trade-offs |
| Postel's Law | Be liberal in what you accept | Form input parsing |
| Von Restorff Effect | Distinctive = memorable | CTA highlighting |
| Zeigarnik Effect | Incomplete = remembered | Progress indicators |
| Serial Position | First/last items best remembered | Navigation order |

---

## Anti-Pattern Quick Hits

| Symptom | Anti-Pattern | Fix |
|---------|-------------|-----|
| Users click non-clickable things | Affordance mismatch | Make it look clickable or un-link |
| Users miss the primary CTA | Button hierarchy collapse | One primary per view |
| Double-submit on slow networks | Missing loading state | Disable + spinner on submit |
| "I don't know what just happened" | Absent feedback | Confirm every mutation |
| "I accidentally deleted everything" | Destructive action unfenced | Require confirm/undo |
| "It looks broken on my phone" | Responsive failure | Test at 320px |
| "I can't use this without a mouse" | Keyboard inaccessible | Audit focus order |
| "It's too noisy / overwhelming" | Information overload | Progressive disclosure |
| "I don't trust this site" | Missing trust signals | Add social proof, security badges |
| "It's slow" | No loading states | Optimistic UI + skeleton |

---

*For depth on any topic, go to the numbered reference file. This quick reference is for in-context recall — the full rules with code live in the reference files.*
