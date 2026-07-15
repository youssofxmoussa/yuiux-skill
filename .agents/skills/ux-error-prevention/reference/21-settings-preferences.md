---
name: settings-preferences
description: Settings organization, destructive settings, and default-value errors and fixes for web UI.
---

# Settings & Preferences

## Anti-pattern: Settings organized by internal data model rather than user mental model
**What's wrong:** Settings grouped by how the backend happens to store them (e.g. a "Profile v2" section reflecting an internal migration) rather than by what a user is trying to accomplish.
**Why it matters:** Users navigate settings by task ("I want to change my email," "I want fewer notifications"), not by implementation detail — mismatched grouping makes settings feel like a maze.
**Fix:** Group settings by user-facing task/topic (Account, Notifications, Privacy, Billing, Appearance) regardless of backend structure, and name each section using the vocabulary a user would use, not internal terminology.

## Anti-pattern: A single flat list of dozens of settings with no grouping or search
**What's wrong:** Every preference crammed into one long scrollable page.
**Why it matters:** See Miller's Law (`01-philosophy-and-principles.md`) — an ungrouped wall of options is hard to scan and easy to get lost in, especially for infrequently-visited settings.
**Fix:** Group by section (with real visual separation, not just a subtle header) and add a settings search once the count grows past roughly 15–20 individual options.

## Anti-pattern: Changes require an explicit "Save" with no feedback, or ambiguous auto-save with no feedback either
**What's wrong:** A toggle/setting change either requires clicking a separate "Save" button the user might miss, or auto-saves silently with zero confirmation, leaving genuine ambiguity about whether the change actually took effect.
**Why it matters:** Both patterns can leave the user unsure whether their change persisted, especially before navigating away.
**Fix:** For most individual settings (toggles, dropdowns), auto-save on change AND show a brief, explicit confirmation ("Saved" text or checkmark near the control) — don't make the user guess which pattern is in effect.

## Anti-pattern: Dangerous settings visually indistinguishable from safe ones
**What's wrong:** "Delete all data," "Make this workspace public," and "Change display name" all styled identically in the same list.
**Why it matters:** Users scanning quickly (the norm for settings pages) can't tell which options carry real risk without reading every single one carefully.
**Fix:** Visually separate destructive/high-risk settings into their own clearly-labeled section (often "Danger Zone" or similar), styled with the reserved danger color (see `03-color-contrast-theming.md`), physically distanced from routine settings.

## Anti-pattern: No sensible defaults — user forced to configure everything before the product is usable
**What's wrong:** Every setting starts blank/unset, requiring the user to manually configure notification preferences, display options, etc. before getting normal behavior.
**Why it matters:** Good defaults are themselves a UX decision — forcing configuration before basic usability shifts unnecessary work onto every single user.
**Fix:** Ship considered, sensible defaults for every setting (favoring what most users would want) and let settings exist purely for the minority who want to change them — never require configuration as a precondition for standard use.

## Anti-pattern: Notification settings are all-or-nothing
**What's wrong:** A single "Enable notifications" toggle covering everything from critical security alerts to promotional content.
**Why it matters:** Users who want security alerts but not marketing emails are forced to choose between unwanted noise and missing important information — many will choose "off" entirely, losing alerts they'd have wanted.
**Fix:** Break notification settings into meaningful categories (Security, Activity, Marketing/Product updates, etc.) with independent controls, and keep genuinely critical/security notifications (e.g. "new login from unrecognized device") non-optional or opt-out-with-friction rather than bundled with marketing toggles.

## Anti-pattern: No preview of the effect of a setting before committing
**What's wrong:** A "compact density" or "dark mode" toggle in settings with no live preview — the user has to save/apply and navigate elsewhere to see the actual effect.
**Why it matters:** Especially for visual settings, seeing the effect immediately is central to confidently choosing the right option.
**Fix:** Apply visual settings (theme, density, font size) live/instantly as the user toggles them, rather than requiring a save-and-check cycle.

## Anti-pattern: Settings that silently affect other users (in a shared/team context) with no warning
**What's wrong:** Changing a workspace-level setting silently changes behavior for every teammate, with no indication in the UI that the scope is broader than "just me."
**Why it matters:** A user might reasonably assume a settings page only affects their own account; unexpectedly affecting collaborators can cause real damage (locked-out teammates, changed permissions) with no warning.
**Fix:** Clearly label the scope of every setting (e.g. "This applies to your account only" vs. "This applies to the whole workspace — all members will be affected") and require extra confirmation for settings with team-wide/shared impact.

## Quick Checklist
- [ ] Settings are grouped by user task/mental model, not internal data structure
- [ ] Long settings lists are sectioned, with search added once the option count grows large
- [ ] Setting changes auto-save with an explicit, visible confirmation
- [ ] Destructive/high-risk settings are visually separated and styled distinctly
- [ ] Every setting ships with a sensible default; configuration is optional, not required for basic use
- [ ] Notification settings are broken into meaningful, independently-controllable categories
- [ ] Visual settings apply live so users can preview before committing
- [ ] Settings with team-wide/shared scope are clearly labeled and confirmed accordingly
