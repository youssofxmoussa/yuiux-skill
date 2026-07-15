---
name: accessibility-deep-dive
description: Keyboard operability, screen reader support, ARIA patterns, focus management, and WCAG 2.2 errors mapped to concrete code fixes.
---

# Accessibility Deep Dive

Accessibility is not a separate feature to bolt on — it's the baseline correctness bar for "does this actually work." Every rule below is a real, common failure with a concrete fix.

## Anti-pattern: Div/span soup with ARIA bolted on instead of semantic HTML
**What's wrong:** Rebuilding a button, checkbox, or link out of generic elements plus `role` attributes instead of using the native element.
**Why it matters:** Native elements give you keyboard behavior, focus management, and assistive-tech announcement for free and battle-tested; hand-rolled ARIA reimplementations routinely miss edge cases (Space vs Enter handling, focus order, state announcement on change).
**Fix:** "No ARIA is better than bad ARIA" — always prefer the native element:

```html
<!-- Bad -->
<div role="button" tabindex="0" onclick="..." onkeydown="...">Save</div>

<!-- Good -->
<button onclick="...">Save</button>
```

## Anti-pattern: Missing or generic `alt` text on images
**What's wrong:** `<img src="chart.png">` with no `alt`, or `alt="image"` / `alt="photo123.jpg"`.
**Why it matters:** Screen reader users get either nothing or a meaningless filename in place of the image's actual content/purpose.
**Fix:** Describe the *purpose* of the image in context, not a literal visual description:

```html
<!-- Informative image -->
<img src="revenue-chart.png" alt="Revenue grew 34% from Q1 to Q2 2026" />

<!-- Purely decorative image -->
<img src="divider-flourish.png" alt="" />

<!-- Functional image (e.g. inside a link/button) -->
<a href="/profile"><img src="avatar.png" alt="Your profile" /></a>
```

## Anti-pattern: Icon-only interactive elements with no accessible name
**What's wrong:** A `<button>` containing only an SVG icon, no text, no `aria-label`.
**Why it matters:** Announced to screen reader users as just "button" — no indication of what it does.
**Fix:** `aria-label` on the control itself (see `07-buttons-controls.md` for the full pattern).

## Anti-pattern: Keyboard trap or missing keyboard path to any control
**What's wrong:** A custom dropdown, date picker, or modal that can be entered with the mouse but can't be exited or fully operated with Tab/Shift+Tab/Enter/Escape/arrow keys.
**Why it matters:** Keyboard-only users (motor impairments, power users, and anyone whose mouse/trackpad is temporarily unavailable) are simply unable to complete the task.
**Fix:** Every interactive component must be fully operable with keyboard alone: Tab to reach it, Enter/Space to activate, Escape to close/cancel, arrow keys for internal navigation (menus, tabs, radio groups), and focus must never get permanently stuck.

## Anti-pattern: Focus not managed when content changes dynamically
**What's wrong:** Opening a modal doesn't move focus into it; closing a modal doesn't return focus to the element that opened it; a client-side route change doesn't reset/announce focus at all.
**Why it matters:** Sighted keyboard users and screen reader users both rely on focus position to know where they are — leaving it on a now-hidden or now-irrelevant element disorients them completely.
**Fix:**
```js
// On modal open
modalRef.current?.focus(); // focus first focusable element inside

// On modal close
triggerButtonRef.current?.focus(); // return focus to opener

// On route change (SPA)
document.getElementById("page-heading")?.focus();
```

## Anti-pattern: Modal/dialog doesn't trap focus
**What's wrong:** Tabbing while a modal is open moves focus to elements behind/beneath it in the page.
**Why it matters:** A sighted user might not notice, but a screen reader user tabbing through a "closed" background page while a modal is technically open becomes hopelessly lost.
**Fix:** Trap Tab/Shift+Tab cycling within the modal's focusable elements while it's open (use a native `<dialog>` element with `showModal()` where feasible — it handles this natively — or a well-tested focus-trap utility otherwise), and set `aria-modal="true"` plus `role="dialog"`/`role="alertdialog"`.

```html
<dialog id="confirm-delete">
  <h2>Delete this item?</h2>
  <button onclick="confirmDelete()">Delete</button>
  <button onclick="dialog.close()">Cancel</button>
</dialog>
<script>document.getElementById("confirm-delete").showModal();</script>
```

## Anti-pattern: No visible focus indicator (or removed entirely)
**What's wrong:** `outline: none` (or `outline: 0`) applied to interactive elements globally, with nothing to replace it.
**Why it matters:** This is one of the single most damaging, common accessibility failures on the web — it makes keyboard navigation functionally unusable, since users can no longer see where they are.
**Fix:** Never remove focus styling without an equally visible replacement, and prefer `:focus-visible` so the ring only shows for keyboard focus (not every mouse click), avoiding the (invalid) excuse that "the ring looks bad on click":

```css
button:focus-visible {
  outline: 2px solid var(--color-primary);
  outline-offset: 2px;
}
```

## Anti-pattern: Illogical or broken tab order
**What's wrong:** `tabindex` values set to arbitrary positive numbers (`tabindex="5"`, `tabindex="12"`) to "fix" ordering, creating a tab sequence that doesn't match the visual/logical reading order.
**Why it matters:** Positive tabindex values create a separate, fragile ordering system that's nearly impossible to maintain correctly as the page evolves, and any mismatch between visual and tab order is deeply disorienting.
**Fix:** Rely on natural DOM order for tab sequence (matching visual layout) rather than positive `tabindex`. Only use `tabindex="0"` (insert into natural order) or `tabindex="-1"` (programmatically focusable, but skipped in Tab sequence — useful for focus-management targets like headings after a route change).

## Anti-pattern: Form errors not announced to screen readers
**What's wrong:** A validation error appears visually (red text under the field) but a screen reader user tabbing past doesn't hear it because it's not associated with the input or announced when it appears.
**Why it matters:** The error is invisible to anyone not looking directly at that exact spot on screen at the exact moment it appears.
**Fix:** Associate the error with the input via `aria-describedby`, mark the input `aria-invalid="true"`, and use a live region so screen readers announce dynamically-appearing errors:

```html
<input id="email" aria-invalid="true" aria-describedby="email-error" />
<p id="email-error" role="alert">Enter a valid email address</p>
```

## Anti-pattern: Live-updating content with no announcement
**What's wrong:** A cart total, a "3 new messages" counter, or a search-results count updates visually with no mechanism for screen reader users to be notified.
**Why it matters:** Sighted users see the change happen; screen reader users have no idea anything changed unless they happen to re-navigate to that exact spot.
**Fix:** Wrap dynamically-updating status content in an `aria-live` region (`polite` for non-urgent updates, `assertive` only for urgent/critical ones — overusing `assertive` is itself disruptive):

```html
<p aria-live="polite">3 items in cart</p>
```

## Anti-pattern: Heading levels used for visual styling instead of document structure
**What's wrong:** An `<h4>` used somewhere purely because its default size "looks right" in a spot that isn't actually a fourth-level heading in the page's logical structure, or heading levels skipped (h1 → h3 with no h2).
**Why it matters:** Screen reader users navigate pages by heading level as a primary wayfinding tool ("jump to next h2") — a structure that doesn't reflect actual content hierarchy breaks that navigation method entirely.
**Fix:** Choose heading level by document structure first, then style that level to look how you want via CSS — never the reverse. Keep levels sequential without skipping.

## Anti-pattern: Form inputs grouped with no `<fieldset>`/`<legend>`
**What's wrong:** A related group of radio buttons or checkboxes (e.g. "Shipping method") rendered as plain inputs with a visual heading above them that isn't programmatically associated.
**Why it matters:** A screen reader user tabbing into the third radio button in the group hears only "radio button, not selected," with no indication of what question they're answering.
**Fix:**
```html
<fieldset>
  <legend>Shipping method</legend>
  <label><input type="radio" name="ship" value="standard" /> Standard</label>
  <label><input type="radio" name="ship" value="express" /> Express</label>
</fieldset>
```

## Anti-pattern: Data tables with no header association
**What's wrong:** A `<table>` built from plain `<td>` cells with no `<th>` elements or `scope` attributes.
**Why it matters:** Screen reader users navigating cell-by-cell lose track of which column/row they're in without header announcement.
**Fix:**
```html
<table>
  <thead>
    <tr><th scope="col">Name</th><th scope="col">Status</th></tr>
  </thead>
  <tbody>
    <tr><td>Acme Corp</td><td>Active</td></tr>
  </tbody>
</table>
```

## Anti-pattern: Skip-to-content link missing
**What's wrong:** No way for a keyboard/screen-reader user to bypass a long repeated navigation menu to get straight to the main content on every single page load.
**Why it matters:** Forces every keyboard user to tab through the entire nav on every page before reaching what they actually came for.
**Fix:**
```html
<a href="#main-content" class="skip-link">Skip to main content</a>
...
<main id="main-content">...</main>
```
```css
.skip-link {
  position: absolute;
  left: -9999px;
}
.skip-link:focus {
  left: 8px;
  top: 8px;
  z-index: var(--z-toast);
}
```

## Anti-pattern: Language not declared, or declared incorrectly for multi-language content
**What's wrong:** Missing `lang` attribute on `<html>`, or a page mixing languages with no per-section `lang` marking.
**Why it matters:** Screen readers use `lang` to select the correct pronunciation engine/voice — without it, foreign or default-language content may be read with the wrong accent/rules, becoming unintelligible.
**Fix:** `<html lang="en">` always set; wrap any embedded different-language passage with its own `lang` attribute.

## Anti-pattern: Motion-sensitive or flashing content with no safeguard
**What's wrong:** Content that flashes more than 3 times per second, or large-area rapid flashing/strobing effects.
**Why it matters:** Can trigger seizures in users with photosensitive epilepsy — this is a hard safety issue, not a preference.
**Fix:** Never ship flashing content above the 3-flashes-per-second threshold (WCAG 2.3.1); avoid large-area strobing effects entirely.

## Anti-pattern: Reliance on hover-only interactions
**What's wrong:** Critical information or controls (a "delete" action, extra detail) revealed only on `:hover`, with no touch or keyboard equivalent.
**Why it matters:** Touch devices have no hover state at all, and keyboard-only users can't trigger `:hover` either — hover-only content is invisible to a meaningful share of users.
**Fix:** Anything revealed on hover must also be reachable via focus (`:focus-within`/`:focus-visible`) and via an explicit tap/click affordance on touch, not hover alone.

## WCAG 2.2 Quick-Reference Table (most commonly violated criteria)

| Criterion | Requirement | Common fix location |
| --- | --- | --- |
| 1.1.1 Non-text Content | Images have text alternatives | `alt` attributes |
| 1.3.1 Info and Relationships | Structure conveyed in code, not just visually | Semantic HTML, `fieldset`/`legend`, `th`/`scope` |
| 1.4.3 Contrast (Minimum) | 4.5:1 text, 3:1 large text | Color tokens, see `03-color-contrast-theming.md` |
| 1.4.11 Non-text Contrast | 3:1 for UI components/graphics | Borders, icons, focus indicators |
| 2.1.1 Keyboard | Everything operable via keyboard | Semantic elements, no keyboard traps |
| 2.4.3 Focus Order | Tab order matches logical/visual order | Natural DOM order, no positive `tabindex` |
| 2.4.7 Focus Visible | Visible focus indicator on every focusable element | `:focus-visible` styling, never `outline: none` alone |
| 2.5.5 Target Size | ≥44×44px CSS px targets | Padding on icon buttons, spacing between targets |
| 3.3.1 Error Identification | Errors identified in text, not color alone | `aria-invalid`, associated error text |
| 3.3.3 Error Suggestion | Errors suggest a fix | Specific, actionable error copy |
| 4.1.2 Name, Role, Value | Every control has an accessible name/role/state | `aria-label`, native elements, `aria-checked`/`aria-expanded` etc. |
| 4.1.3 Status Messages | Status changes announced without requiring focus move | `aria-live` regions |

## Quick Checklist
- [ ] Native semantic elements used before reaching for ARIA
- [ ] Every informative image has descriptive `alt`; decorative images use `alt=""`
- [ ] Full keyboard operability: reach, activate, escape, and navigate every control without a mouse
- [ ] Focus is explicitly managed on modal open/close and on SPA route changes
- [ ] Modals trap focus and are marked `aria-modal`/`role="dialog"`
- [ ] Visible focus indicator present on every focusable element, never globally removed
- [ ] Tab order follows natural DOM order, no positive `tabindex`
- [ ] Form errors are associated via `aria-describedby`/`aria-invalid` and announced via `role="alert"`
- [ ] Dynamic status updates use `aria-live`
- [ ] Heading levels reflect real document structure, sequential, never skipped
- [ ] Related form controls are grouped in `fieldset`/`legend`
- [ ] Data tables use proper `th`/`scope` header association
- [ ] Skip-to-content link present
- [ ] `lang` attribute set correctly, including per-section for mixed-language content
- [ ] No flashing content above 3/second; no hover-only critical interactions
