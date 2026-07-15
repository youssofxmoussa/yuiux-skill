---
name: philosophy-and-principles
description: Foundational mental models behind every rule in this skill — Nielsen's heuristics, the Laws of UX, and Apple HIG's pillars, all mapped to concrete web behaviors.
---

# Philosophy & Principles

Every specific fix in this skill traces back to one of a small number of underlying principles. When you hit a UX judgment call not explicitly covered elsewhere, reason from these first.

## 1. Nielsen's 10 Usability Heuristics (web-mapped)

### 1.1 Visibility of system status
The user should never have to wonder "did that work?" Every action gets a response within 100ms — even if the real work takes longer, acknowledge receipt instantly (button press state, then spinner, then result).

**Common violation:** clicking "Save" and nothing visibly changes for 2 seconds while a network request is in flight.
**Fix:** immediately switch the button to a loading/disabled state on click, before the response returns.

### 1.2 Match between system and the real world
Use words, icons, and concepts users already have, not internal engineering vocabulary. "Archive" not "Soft-delete." "Team" not "Tenant." A trash can icon for delete, not an abstract glyph you invented.

### 1.3 User control and freedom
Provide an exit from every flow: cancel on every modal, back on every wizard step, undo on every destructive action. Never trap a user in a flow with no way out except closing the tab.

### 1.4 Consistency and standards
If "Save" is a solid blue button in one place, it's a solid blue button everywhere. If Enter submits a form on one page, it submits every form. Breaking platform conventions (e.g. underlined blue text that isn't a link) actively misleads users who've learned those conventions elsewhere on the web.

### 1.5 Error prevention
The best error message is the one that never needed to appear. Prefer constraints that make errors impossible (disable a submit button until required fields are valid, disable dates before today in a "start date" picker) over validation messages that catch the error after the fact.

### 1.6 Recognition rather than recall
Don't make users remember information across screens. Show recently viewed items, keep filters visible, autofill from previous entries, show what's currently selected rather than requiring the user to remember it.

### 1.7 Flexibility and efficiency of use
Support both novice and expert paths: a novice clicks through a wizard; an expert can use a keyboard shortcut, bulk action, or command palette to skip steps. Don't force everyone through the slowest path.

### 1.8 Aesthetic and minimalist design
Every element on screen must earn its place. Remove any label, icon, border, or decoration that doesn't help the user complete their task or understand the content. Clutter is not information density — it's noise that hides the signal.

### 1.9 Help users recognize, diagnose, and recover from errors
Errors must be expressed in plain language (no error codes as the primary message), precisely indicate the problem, and constructively suggest a solution.

### 1.10 Help and documentation
Even a well-designed product occasionally needs inline help — but it should be contextual (a tooltip next to the confusing field), searchable, and never a wall of text a user has to read before they can proceed.

## 2. The Laws of UX (selected, most actionable for web)

- **Jakob's Law**: Users spend most of their time on other sites/apps. They expect yours to work the same way. Don't invent novel interaction patterns for common tasks (search, login, checkout) — novelty here is a cost, not a feature.
- **Fitts's Law**: The time to acquire a target is a function of distance and size. Primary actions should be large and close to where the user's attention already is; don't force a mouse trip across the screen for the most common action.
- **Hick's Law**: The time to make a decision increases with the number and complexity of choices. Reduce visible options at any one decision point; use progressive disclosure (advanced options behind a toggle) instead of showing everything at once.
- **Miller's Law**: The average person can hold about 4 (not the old "7") chunks of information in working memory. Group related fields/items visually (cards, sections) so users process chunks, not individual data points.
- **Postel's Law (Robustness Principle)**: Be liberal in what you accept from users, strict in what you send back. Accept "1/2/2026", "Jan 2 2026", and "2026-01-02" as valid date input where feasible; always render dates back in one consistent, unambiguous format.
- **Peak-End Rule**: Users judge an experience disproportionately by its most intense moment and its ending. A rocky middle can be forgiven; a bad final step (confusing checkout confirmation, silent failure at the last click) is remembered as the whole experience being bad. Invest extra polish in the last step of any flow.
- **Von Restorff Effect (Isolation Effect)**: An item that stands out visually is more likely to be remembered/noticed. Use this deliberately for the one primary action per screen — and be aware that if everything is bold and colorful, nothing stands out.
- **Aesthetic-Usability Effect**: Users perceive more aesthetically pleasing designs as easier to use, even when usability is identical, and are more forgiving of small friction in a beautiful UI. This is not an excuse to skip real usability work, but it means visual polish is not "just polish" — it changes how tolerant users are of the rest of the product.
- **Zeigarnik Effect**: People remember interrupted/incomplete tasks better than completed ones, and feel a pull to finish them. Use this for onboarding progress bars ("2 of 4 steps complete") to motivate completion — and be aware it means an interrupted checkout is more frustrating than a completed one, so protect saved progress.
- **Doherty Threshold**: Productivity/engagement rises sharply when a system responds within 400ms. Anything the user initiates should visibly respond well under that number, even if the underlying work continues after.

## 3. Apple HIG's Three Pillars, Web-Translated

### Clarity
- Text is legible at the sizes you actually ship (test your smallest body size at arm's length, not just in the editor).
- Icons always pair with a label or have unambiguous, widely-recognized meaning. If you have to ask "will a first-time user know what this icon means?", it needs a label.
- One primary action per screen, visually distinct from secondary/tertiary actions.

### Deference
- Content is the interface. Chrome should recede: subtle borders/dividers instead of heavy boxes around everything, restrained use of shadows, no decorative gradients that don't communicate anything.
- Don't let branding/theming compete with the user's actual content and task.

### Depth
- Elevation (shadow, blur, scale, motion) should map to real hierarchy: modals appear "above" the page and animate in from a state that implies that; dropdowns emerge from their trigger; sheets slide from the edge they'll return to.
- Depth cues should always agree with each other — a "elevated" card with a heavy shadow but no scale/motion difference on interaction feels inert, not deep.

## 4. The Meta-Principle

If a UI decision doesn't clearly serve clarity, deference, depth, feedback, or consistency, it's decoration — and decoration that doesn't earn its place is itself a UX error, because it competes with the user's attention for nothing in return. When in doubt, remove it and see if anything real was lost.
