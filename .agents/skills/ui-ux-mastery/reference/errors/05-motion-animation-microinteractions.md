---
name: motion-animation-microinteractions
description: Easing, duration budgets, transitions, micro-interactions, and reduced-motion errors and fixes for web UI.
---

# Motion, Animation & Micro-interactions

Apple platforms are defined as much by motion as by visuals — every transition tells the user where something came from and where it's going. On the web, motion is usually either missing entirely (abrupt state changes) or overused (animation for its own sake). Both are errors.

## Anti-pattern: No transition on state changes at all
**What's wrong:** A dropdown appears/disappears instantly with a hard cut; a modal pops into existence with no transition; a list item vanishes the instant it's deleted.
**Why it matters:** Instant cuts make it harder to track cause and effect ("what just happened, and where did it go?") and make the whole UI feel mechanical rather than responsive.
**Fix:** Any element that appears, disappears, or changes state should transition, even briefly (150–250ms is enough for most micro-transitions):

```css
.dropdown {
  transition: opacity 150ms ease-out, transform 150ms ease-out;
  transform: translateY(-4px);
  opacity: 0;
}
.dropdown[data-open="true"] {
  transform: translateY(0);
  opacity: 1;
}
```

## Anti-pattern: Animation with no directional/spatial logic
**What's wrong:** A modal fades in from the center with no relationship to what triggered it; a sheet that should slide from the bottom instead just fades; a "next" navigation transitions identically to "back."
**Why it matters:** Motion is supposed to communicate spatial relationships (this element came from there, this one is now beneath that one). Motion with no directional logic is just decoration, and inconsistent direction (forward and back looking the same) actively confuses the user's mental model of navigation.
**Fix:** Match the animation direction to the real relationship: sheets/drawers slide from the edge they're anchored to and return there on close; forward navigation slides content in from one side, back navigation reverses it; a popover grows from its trigger element, not from nowhere.

## Anti-pattern: Animations that are too slow
**What's wrong:** A 600–800ms transition on a dropdown, tooltip, or hover state — anything a user triggers frequently and expects to feel instantaneous.
**Why it matters:** Overly long transitions make a UI feel sluggish and unresponsive, and compound painfully when a user is doing a repeated task (e.g. opening/closing a menu 20 times).
**Fix:** Match duration to frequency and distance of motion:

| Interaction type | Typical duration |
| --- | --- |
| Micro (hover, toggle, small icon state change) | 100–150ms |
| Small UI transitions (dropdown, tooltip, popover) | 150–250ms |
| Medium transitions (modal open/close, sheet slide) | 200–350ms |
| Large/page-level transitions | 300–500ms |

Never exceed ~500ms for anything triggered by direct user interaction; longer durations are reserved for passive/ambient animation the user isn't actively waiting on.

## Anti-pattern: Linear easing on everything
**What's wrong:** `transition-timing-function: linear` (or the unstated default, which is often close to linear-feeling) used for all motion.
**Why it matters:** Real-world motion accelerates and decelerates; linear motion feels robotic and mechanical because nothing in physical experience moves at constant velocity from a standstill.
**Fix:** Use easing curves that mimic natural motion:
- **Ease-out** (`cubic-bezier(0.16, 1, 0.3, 1)` or similar) for elements entering the screen — fast start, gentle settle. Feels responsive.
- **Ease-in** for elements leaving — slow start, fast exit. Feels like it's being dismissed intentionally.
- **Ease-in-out** for elements that move between two visible states (e.g. a toggle switch, a tab indicator sliding).

```css
.enter { transition: transform 200ms cubic-bezier(0.16, 1, 0.3, 1); }
.exit  { transition: transform 150ms cubic-bezier(0.4, 0, 1, 1); }
```

## Anti-pattern: Animating layout properties instead of compositor-friendly properties
**What's wrong:** Animating `width`, `height`, `top`, `left`, or `margin` for movement/resizing effects.
**Why it matters:** These properties trigger layout recalculation on every frame, causing janky, dropped-frame animation especially on lower-powered devices — exactly where smooth motion matters most for perceived quality.
**Fix:** Animate `transform` (translate/scale/rotate) and `opacity` instead — both are handled by the GPU compositor without triggering layout:

```css
/* Bad: causes layout thrash */
.card:hover { width: 320px; }

/* Good: GPU-composited */
.card:hover { transform: scale(1.02); }
```

## Anti-pattern: `prefers-reduced-motion` ignored
**What's wrong:** Parallax, auto-playing background animation, large scale/slide transitions applied unconditionally regardless of the user's OS-level motion preference.
**Why it matters:** A meaningful share of users enable reduced motion specifically because animation causes vestibular discomfort, nausea, or migraines — shipping motion that ignores this preference is not a stylistic choice, it's an accessibility failure with real physical consequences for affected users.
**Fix:** Always provide a reduced/no-motion fallback:

```css
@media (prefers-reduced-motion: reduce) {
  * {
    animation-duration: 0.001ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.001ms !important;
    scroll-behavior: auto !important;
  }
}
```
For essential motion that communicates information (e.g. a progress indicator), keep it but strip decorative parallax/bounce/auto-play effects specifically.

## Anti-pattern: No feedback animation on direct manipulation
**What's wrong:** A button, checkbox, or draggable item that doesn't visually respond the instant it's pressed/touched — feedback only appears after the action fully completes.
**Why it matters:** Immediate micro-feedback (a button depressing 1–2px on `:active`, a checkbox's check-mark drawing in) is what makes an interface feel directly manipulable rather than just "a form you fill out and submit into the void."
**Fix:** Add a fast (<100ms), always-present feedback state for the moment of contact, separate from the (possibly slower) result of the action itself:

```css
.button:active {
  transform: scale(0.97);
  transition: transform 80ms ease-out;
}
```

## Anti-pattern: Skeleton/spinner animation that itself is distracting
**What's wrong:** An aggressively pulsing or spinning loading indicator that draws more visual attention than the content it's standing in for.
**Why it matters:** Loading states should be calm — they exist to reassure, not to demand attention while the user waits.
**Fix:** Use a slow, subtle shimmer/pulse (1.5–2s cycle, low opacity delta) for skeletons rather than a fast, high-contrast animation.

## Anti-pattern: Autoplaying carousels/animations that never stop
**What's wrong:** A hero carousel that auto-advances every 3 seconds indefinitely, with no pause control, competing for attention with whatever the user is trying to read.
**Why it matters:** Continuous motion in peripheral vision is cognitively costly to ignore — it's a proven distraction from the user's actual task, and auto-advancing carousels specifically have some of the worst engagement/click-through rates of any common web pattern (users rarely see the 2nd+ slide before it changes on them).
**Fix:** Avoid auto-advancing carousels for anything the user needs to actually read or choose from; if used at all (e.g. a purely decorative background), provide a visible pause/stop control and stop indefinite motion once the user starts interacting with the page.

## Quick Checklist
- [ ] Every appear/disappear/state-change has a transition, none are a hard instant cut
- [ ] Motion direction matches real spatial relationships (sheets from their edge, back ≠ forward)
- [ ] Durations match interaction frequency (100–250ms for frequent micro-interactions, up to ~500ms for page-level transitions, never longer for user-triggered motion)
- [ ] Ease-out for entrances, ease-in for exits, ease-in-out for state toggles — never linear
- [ ] Animations use `transform`/`opacity`, not layout-triggering properties
- [ ] `prefers-reduced-motion: reduce` is honored app-wide
- [ ] Direct-manipulation elements (buttons, drags, toggles) give sub-100ms feedback on contact
- [ ] Loading indicators are calm, not visually aggressive
- [ ] No indefinitely auto-advancing carousels without a pause control
