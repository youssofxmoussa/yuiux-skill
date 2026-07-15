---
name: buttons-controls
description: Button hierarchy, interactive states, hit targets, and toggle/switch/slider/checkbox errors and fixes for web UI.
---

# Buttons & Controls

## Anti-pattern: Clickable `<div>`/`<span>` instead of a real `<button>`/`<a>`
**What's wrong:** `<div onClick={...}>Save</div>` used for an action that should be a semantic control.
**Why it matters:** A `div` isn't focusable by default, doesn't respond to Enter/Space, isn't announced as a button to screen readers, and needs `role`, `tabIndex`, and manual key handling bolted on to fake what a real element gives for free — and it's easy to miss one of those and ship something keyboard/screen-reader users simply cannot activate.
**Fix:** Use `<button>` for in-page actions and `<a href>` for navigation, always:

```html
<!-- Bad -->
<div class="btn" onclick="save()">Save</div>

<!-- Good -->
<button type="button" onclick="save()">Save</button>
```

## Anti-pattern: More than one visually "loud" primary button per screen
**What's wrong:** Three buttons on a page all styled as solid, high-contrast, brand-colored "primary" buttons.
**Why it matters:** If everything is emphasized, nothing is — the user has to read every option instead of being guided to the one action that matters most.
**Fix:** Establish a strict hierarchy and enforce it everywhere:
- **Primary** (solid, high-contrast fill): the one main action per screen/section.
- **Secondary** (outlined or subtle fill): alternative but common actions.
- **Tertiary/ghost** (text-only, no border): low-emphasis actions (e.g. "Cancel").
- **Destructive**: reserved danger color, used only for irreversible/harmful actions (see `03-color-contrast-theming.md`).

```
[ Save changes ]   Cancel          <- primary + tertiary, correct
[ Save ] [ Delete ] [ Export ]     <- three loud buttons, no hierarchy, wrong
```

## Anti-pattern: Button text describes the mechanism, not the outcome
**What's wrong:** "Submit," "OK," "Confirm," "Yes" used for every consequential action regardless of what it actually does.
**Why it matters:** A generic label forces the user to rely on surrounding context (or trust) rather than the button itself to know what will happen — risky for anything consequential.
**Fix:** Label buttons with the specific outcome:

```
Bad:  [ Submit ]           on a form that creates a paid subscription
Good: [ Start subscription — $12/mo ]

Bad:  [ OK ]                on a delete-confirmation dialog
Good: [ Delete photo ]
```

## Anti-pattern: Not designing every interactive state
**What's wrong:** A button only has a "default" style; hover, focus, active/pressed, disabled, and loading are either unstyled (relying on browser defaults) or missing.
**Why it matters:** Each state exists to answer a different question ("can I click this?", "did I just click it?", "is it currently doing something?") — skipping any of them leaves the user without an answer at exactly the moment they need one.
**Fix:** Design all six states explicitly for every button/control type:

```css
.btn { /* default */ }
.btn:hover { /* subtle lift/darken, signals interactivity */ }
.btn:focus-visible { outline: 2px solid var(--color-primary); outline-offset: 2px; }
.btn:active { transform: scale(0.97); }
.btn:disabled { opacity: 0.5; cursor: not-allowed; }
.btn[data-loading="true"] { /* spinner replaces label, stays same size */ }
```

## Anti-pattern: Disabled buttons with no explanation why
**What's wrong:** A grayed-out "Continue" button with no indication of what's missing to enable it.
**Why it matters:** The user is left guessing — did they miss a field? Is there a system error? Is this feature just not available?
**Fix:** Either (a) keep the button enabled and show a clear validation error on click explaining what's missing, or (b) if truly disabled, surface a tooltip or nearby text stating exactly why ("Add at least one item to continue"). Prefer (a) in most cases — it's more discoverable and doesn't require hover to diagnose.

## Anti-pattern: Loading state changes the button's size or removes its label entirely with no context
**What's wrong:** Button text is replaced by a spinner and the button shrinks/reflows, shifting surrounding layout.
**Why it matters:** Layout shift during exactly the moment a user is waiting on feedback is disorienting, and losing the label removes confirmation of what's actually in progress.
**Fix:** Fix the button's dimensions before entering loading state (`min-width` sized to the longest label variant) and prefer a spinner *alongside* a shortened label ("Saving…") over a bare spinner with no text.

## Anti-pattern: Hit targets too small for a mouse, let alone touch
**What's wrong:** Icon buttons rendered at 16–20px with no extra padding around the clickable area.
**Why it matters:** Fitts's Law: small, precise targets take disproportionately longer and more error-prone effort to hit. On touch specifically, fingers are far less precise than a mouse cursor.
**Fix:** Visual icon size can stay small, but the *clickable/tappable* area should be at least 44×44px (Apple HIG's long-standing minimum, matched by WCAG 2.5.5's 44×44 CSS px target size guidance) via padding, not by inflating the icon itself:

```css
.icon-button {
  width: 44px;
  height: 44px;
  display: inline-flex;
  align-items: center;
  justify-content: center;
}
.icon-button svg { width: 20px; height: 20px; }
```

## Anti-pattern: Adjacent small targets with no spacing, causing mis-taps
**What's wrong:** A row of icon buttons (edit/delete/duplicate) packed edge-to-edge with no gap, especially on touch.
**Why it matters:** Even with each target individually ≥44px, zero spacing between a "duplicate" and a "delete" button turns a fat-finger tap into a destructive mistake.
**Fix:** Maintain at least 8px of gap between adjacent independent tap targets, more if any of them is destructive.

## Anti-pattern: Toggle/switch state ambiguous — is "on" left or right, filled or outline?
**What's wrong:** A custom switch component where the on/off visual states aren't clearly distinct, or the direction convention is inconsistent across the app.
**Why it matters:** A switch that requires the user to click it to find out what it currently means has failed at its one job — showing current state at a glance.
**Fix:** Follow the universal convention: filled/colored track = on, positioned to the side that matches the platform-standard reading direction (typically right = on in LTR locales), with the thumb clearly at one extreme or the other — never ambiguous mid-travel as a resting state.

## Anti-pattern: Checkbox/radio group with no clear indication of single vs. multiple selection
**What's wrong:** Using checkboxes for a mutually-exclusive choice (should be radio buttons) or radio buttons for a multi-select list (should be checkboxes).
**Why it matters:** The shape itself (square vs. circle) is a long-standing convention users rely on to predict behavior without reading instructions — using the wrong shape actively misleads.
**Fix:** Checkboxes (square) = zero-or-more independent selections. Radio buttons (circle) = exactly one from a set. Never substitute one for the other regardless of custom styling.

## Anti-pattern: Sliders with no visible current value or increment feedback
**What's wrong:** A range slider that shows only the handle position with no numeric readout, and no indication of what values are selectable.
**Why it matters:** Users can't verify precision (is this exactly 50, or 49.6?) and can't set a specific target value confidently.
**Fix:** Show the live numeric value near the handle (updating as it's dragged) and provide a text-input alternative for precise entry when the value matters (e.g. budget filters, quantity).

## Anti-pattern: Dropdown/select styled to look identical to a static label
**What's wrong:** A native `<select>` or custom dropdown with no chevron/caret icon or visual affordance suggesting it's interactive.
**Why it matters:** Without a clear "this can be changed" signal, users may not realize a value is even editable.
**Fix:** Always include a visible chevron icon and treat the whole control with the same interactive-state styling (hover/focus) as buttons.

## Anti-pattern: Icon-only buttons with no accessible label
**What's wrong:** `<button><svg .../></button>` with no `aria-label`, relying purely on the icon's shape to convey meaning.
**Why it matters:** Screen reader users hear nothing meaningful ("button" with no name); sighted users unfamiliar with the icon (not everyone recognizes a floppy-disk "save" icon, for instance) have no fallback.
**Fix:** Always add `aria-label` (or visually-hidden text) to icon-only controls:

```html
<button aria-label="Delete item" type="button">
  <TrashIcon />
</button>
```

## Quick Checklist
- [ ] Every action is a real `<button>`/`<a>`, never a clickable `div`/`span`
- [ ] Exactly one visually "loud" primary action per screen/section; clear secondary/tertiary/destructive hierarchy
- [ ] Button labels describe the specific outcome, not "Submit"/"OK"
- [ ] Default, hover, focus-visible, active, disabled, and loading states are all explicitly designed
- [ ] Disabled buttons explain why (or, preferably, stay enabled and validate on click)
- [ ] Loading state keeps button dimensions fixed and retains a text label where possible
- [ ] All tap/click targets are ≥44×44px regardless of visual icon size
- [ ] Adjacent independent targets have ≥8px gap, more for destructive ones
- [ ] Switches/checkboxes/radios follow their universal shape/state conventions, never swapped
- [ ] Sliders show a live numeric readout
- [ ] Dropdowns/selects visually signal they're interactive (chevron + hover/focus states)
- [ ] Icon-only buttons always carry an `aria-label`
