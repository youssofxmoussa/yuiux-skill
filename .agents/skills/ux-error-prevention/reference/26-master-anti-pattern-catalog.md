---
name: master-anti-pattern-catalog
description: Full cross-referenced index of every anti-pattern across the skill, plus additional miscellaneous UX errors and fixes not covered in the topic-specific files.
---

# Master Anti-Pattern Catalog

This file has two parts: **Part 1** is a single scannable table indexing every anti-pattern covered across all 25 topic files, for fast lookup when you know roughly what you're looking for but not which file it's in. **Part 2** covers additional miscellaneous anti-patterns that don't fit neatly into any single topic file above.

## Part 1: Full Index

| # | Anti-pattern | File |
|---|---|---|
| 1 | Ignoring Nielsen's heuristics / Laws of UX | `01-philosophy-and-principles.md` |
| 2 | Text below 16px for body copy | `02-typography-readability.md` |
| 3 | Line length too long/short for comfortable reading | `02-typography-readability.md` |
| 4 | Justified text creating rivers of whitespace | `02-typography-readability.md` |
| 5 | Long passages in uppercase | `02-typography-readability.md` |
| 6 | Skipped/non-sequential heading levels | `02-typography-readability.md`, `10-accessibility-deep-dive.md` |
| 7 | Too many typefaces/weights on one screen | `02-typography-readability.md` |
| 8 | No `font-display` fallback causing invisible text | `02-typography-readability.md` |
| 9 | Non-tabular numerals in changing/tabular numeric displays | `02-typography-readability.md` |
| 10 | Truncated text with no accessible full-text fallback | `02-typography-readability.md` |
| 11 | No design-token color system | `03-color-contrast-theming.md`, `25-design-systems-consistency.md` |
| 12 | Color as the only signal of meaning/state | `03-color-contrast-theming.md` |
| 13 | Text/UI contrast below WCAG thresholds | `03-color-contrast-theming.md` |
| 14 | Low-contrast placeholder text mistaken for real content | `03-color-contrast-theming.md` |
| 15 | Dark mode built as a naive color inversion | `03-color-contrast-theming.md` |
| 16 | Ignoring `prefers-color-scheme` | `03-color-contrast-theming.md` |
| 17 | Danger color reused for non-destructive UI | `03-color-contrast-theming.md` |
| 18 | Oversaturated palette everywhere | `03-color-contrast-theming.md` |
| 19 | Indistinct hover/focus/active/disabled color states | `03-color-contrast-theming.md`, `07-buttons-controls.md` |
| 20 | No spacing scale / arbitrary spacing values | `04-layout-spacing-grid.md` |
| 21 | Inconsistent spacing on the same component type | `04-layout-spacing-grid.md`, `25-design-systems-consistency.md` |
| 22 | Elements not aligned to a shared grid | `04-layout-spacing-grid.md` |
| 23 | Poor grouping/proximity of related elements | `04-layout-spacing-grid.md` |
| 24 | Full-width body text with no max content width | `04-layout-spacing-grid.md` |
| 25 | Insufficient whitespace around dense content | `04-layout-spacing-grid.md` |
| 26 | Fixed pixel widths that don't respond fluidly | `04-layout-spacing-grid.md` |
| 27 | Fixed height instead of `min-height` on variable content | `04-layout-spacing-grid.md` |
| 28 | No z-index scale, arbitrary stacking order | `04-layout-spacing-grid.md` |
| 29 | Layout shift (poor CLS) from unreserved space | `04-layout-spacing-grid.md`, `11-performance-perceived-performance.md` |
| 30 | No transition on appear/disappear/state changes | `05-motion-animation-microinteractions.md` |
| 31 | Animation with no spatial/directional logic | `05-motion-animation-microinteractions.md` |
| 32 | Animations too slow for frequent interactions | `05-motion-animation-microinteractions.md` |
| 33 | Linear easing used everywhere | `05-motion-animation-microinteractions.md` |
| 34 | Animating layout properties instead of transform/opacity | `05-motion-animation-microinteractions.md` |
| 35 | Ignoring `prefers-reduced-motion` | `05-motion-animation-microinteractions.md` |
| 36 | No immediate feedback animation on direct manipulation | `05-motion-animation-microinteractions.md` |
| 37 | Visually aggressive loading animation | `05-motion-animation-microinteractions.md` |
| 38 | Auto-advancing carousels with no pause control | `05-motion-animation-microinteractions.md` |
| 39 | Placeholder used as the only label | `06-forms-inputs.md` |
| 40 | Floating labels that collide with long values | `06-forms-inputs.md` |
| 41 | Validating on every keystroke before input is complete | `06-forms-inputs.md` |
| 42 | Validating only on final submit | `06-forms-inputs.md` |
| 43 | Generic/blaming/code-based error messages | `06-forms-inputs.md`, `09-feedback-loading-empty-error-states.md`, `14-microcopy-content-strategy.md` |
| 44 | Clearing form input on failure | `06-forms-inputs.md` |
| 45 | No password visibility toggle | `06-forms-inputs.md` |
| 46 | Opaque or overly restrictive password requirements | `06-forms-inputs.md`, `19-auth-account-ux.md` |
| 47 | No autofill support (`autocomplete`/`name`/`type`) | `06-forms-inputs.md` |
| 48 | Wrong `inputmode`/`type` for the data collected | `06-forms-inputs.md` |
| 49 | Losing progress on multi-step forms | `06-forms-inputs.md` |
| 50 | No progress indicator on wizards | `06-forms-inputs.md` |
| 51 | Submit button stays clickable during submission (duplicate submits) | `06-forms-inputs.md` |
| 52 | Inconsistent required-field marking | `06-forms-inputs.md` |
| 53 | Destructive reset/clear button styled like a normal action | `06-forms-inputs.md` |
| 54 | No autosave on long forms | `06-forms-inputs.md` |
| 55 | Clickable `div`/`span` instead of real button/link | `07-buttons-controls.md`, `10-accessibility-deep-dive.md` |
| 56 | Multiple loud "primary" buttons with no hierarchy | `07-buttons-controls.md` |
| 57 | Button labels describing mechanism, not outcome | `07-buttons-controls.md`, `14-microcopy-content-strategy.md` |
| 58 | Missing hover/focus/active/disabled/loading states | `07-buttons-controls.md` |
| 59 | Disabled buttons with no explanation | `07-buttons-controls.md` |
| 60 | Loading state shifts button size/removes label context | `07-buttons-controls.md` |
| 61 | Tap targets smaller than 44×44px | `07-buttons-controls.md`, `10-accessibility-deep-dive.md`, `12-mobile-responsive-touch.md` |
| 62 | Adjacent small targets with no spacing | `07-buttons-controls.md`, `12-mobile-responsive-touch.md` |
| 63 | Ambiguous toggle/switch on-state | `07-buttons-controls.md` |
| 64 | Checkbox used for exclusive choice, or radio for multi-select | `07-buttons-controls.md` |
| 65 | Sliders with no live numeric readout | `07-buttons-controls.md` |
| 66 | Dropdown with no interactive affordance | `07-buttons-controls.md` |
| 67 | Icon-only buttons with no `aria-label` | `07-buttons-controls.md`, `10-accessibility-deep-dive.md` |
| 68 | No indication of current nav location | `08-navigation-ia.md` |
| 69 | Icon-only nav with no labels | `08-navigation-ia.md` |
| 70 | Hamburger menu on desktop with no space constraint | `08-navigation-ia.md` |
| 71 | Deep/inconsistent menu structures | `08-navigation-ia.md` |
| 72 | Missing breadcrumbs on deep hierarchies | `08-navigation-ia.md` |
| 73 | Tabs used for sequential or frequently-compared content | `08-navigation-ia.md` |
| 74 | Broken browser back-button behavior in SPAs | `08-navigation-ia.md` |
| 75 | Refresh loses UI state (filters, tabs, open items) | `08-navigation-ia.md`, `16-search-filtering.md` |
| 76 | No home/logo escape hatch | `08-navigation-ia.md` |
| 77 | Search buried despite high content volume | `08-navigation-ia.md` |
| 78 | Dead-end states with no forward action | `08-navigation-ia.md` |
| 79 | Inconsistent scroll-position handling on navigation | `08-navigation-ia.md` |
| 80 | Blank screen with no loading indicator | `09-feedback-loading-empty-error-states.md` |
| 81 | Generic spinner used for content-shaped areas instead of skeletons | `09-feedback-loading-empty-error-states.md` |
| 82 | Loading state with no timeout/escape hatch | `09-feedback-loading-empty-error-states.md` |
| 83 | Generic empty state with no explanation/action | `09-feedback-loading-empty-error-states.md`, `13-onboarding-first-run.md` |
| 84 | No error boundary — one failure blanks the whole page | `09-feedback-loading-empty-error-states.md`, `24-error-prevention-recovery.md` |
| 85 | Error severity mismatched to presentation (modal vs. toast vs. inline) | `09-feedback-loading-empty-error-states.md`, `15-notifications-alerts-modals.md` |
| 86 | Optimistic UI with no rollback on failure | `09-feedback-loading-empty-error-states.md` |
| 87 | Partial batch failure reported as one opaque result | `09-feedback-loading-empty-error-states.md` |
| 88 | Toasts too fast to read, or stacking unbounded | `09-feedback-loading-empty-error-states.md`, `15-notifications-alerts-modals.md` |
| 89 | No confirmation signal after a successful action | `09-feedback-loading-empty-error-states.md` |
| 90 | Div/span with bolted-on ARIA instead of semantic HTML | `10-accessibility-deep-dive.md` |
| 91 | Missing/generic `alt` text | `10-accessibility-deep-dive.md` |
| 92 | Keyboard traps / no full keyboard operability | `10-accessibility-deep-dive.md` |
| 93 | Focus not managed on modal open/close or route change | `10-accessibility-deep-dive.md` |
| 94 | Modal doesn't trap focus | `10-accessibility-deep-dive.md` |
| 95 | Focus indicator removed with no replacement | `10-accessibility-deep-dive.md` |
| 96 | Positive `tabindex` breaking natural tab order | `10-accessibility-deep-dive.md` |
| 97 | Form errors not announced to screen readers | `10-accessibility-deep-dive.md` |
| 98 | Live-updating content with no `aria-live` | `10-accessibility-deep-dive.md` |
| 99 | Heading levels chosen for style, not structure | `10-accessibility-deep-dive.md` |
| 100 | Related inputs with no `fieldset`/`legend` grouping | `10-accessibility-deep-dive.md` |
| 101 | Data tables with no header association (`th`/`scope`) | `10-accessibility-deep-dive.md` |
| 102 | No skip-to-content link | `10-accessibility-deep-dive.md` |
| 103 | Missing/incorrect `lang` attribute | `10-accessibility-deep-dive.md` |
| 104 | Flashing content above seizure-risk threshold | `10-accessibility-deep-dive.md` |
| 105 | Hover-only interactions with no keyboard/touch equivalent | `10-accessibility-deep-dive.md`, `12-mobile-responsive-touch.md` |
| 106 | No feedback within ~100ms of user action | `11-performance-perceived-performance.md` |
| 107 | Large unoptimized images | `11-performance-perceived-performance.md` |
| 108 | Everything loaded eagerly, no lazy loading/code splitting | `11-performance-perceived-performance.md` |
| 109 | No debounce/throttle on high-frequency handlers | `11-performance-perceived-performance.md`, `16-search-filtering.md` |
| 110 | Main thread blocked by heavy synchronous computation | `11-performance-perceived-performance.md` |
| 111 | Slow Time-to-Interactive despite fast visual paint | `11-performance-perceived-performance.md` |
| 112 | Unvirtualized/unpaginated large lists | `11-performance-perceived-performance.md`, `17-data-tables-dashboards.md` |
| 113 | Sequential network waterfall instead of parallel requests | `11-performance-perceived-performance.md` |
| 114 | No caching, re-fetching everything on every navigation | `11-performance-perceived-performance.md` |
| 115 | Long operations with no progressive status | `11-performance-perceived-performance.md` |
| 116 | Desktop-first design shrunk down instead of mobile-first | `12-mobile-responsive-touch.md` |
| 117 | Missing/zoom-disabling viewport meta tag | `12-mobile-responsive-touch.md` |
| 118 | Unintended horizontal page scroll | `12-mobile-responsive-touch.md` |
| 119 | Bottom actions obstructed by OS chrome/keyboard | `12-mobile-responsive-touch.md` |
| 120 | Custom gestures conflicting with native browser gestures | `12-mobile-responsive-touch.md` |
| 121 | Accidental double-tap-zoom on rapid taps | `12-mobile-responsive-touch.md` |
| 122 | Device capability assumed purely from viewport width | `12-mobile-responsive-touch.md` |
| 123 | Layout breaks on orientation change | `12-mobile-responsive-touch.md` |
| 124 | Input font-size <16px causing iOS auto-zoom | `12-mobile-responsive-touch.md` |
| 125 | Native controls unnecessarily reimplemented | `12-mobile-responsive-touch.md` |
| 126 | Forced multi-screen tour blocking first use | `13-onboarding-first-run.md` |
| 127 | No onboarding value in first-run empty states | `13-onboarding-first-run.md` |
| 128 | Coaching tooltips that never dismiss permanently | `13-onboarding-first-run.md` |
| 129 | No progressive disclosure — full complexity shown up front | `13-onboarding-first-run.md` |
| 130 | No sample/seed content for new accounts | `13-onboarding-first-run.md` |
| 131 | Signup collecting more than needed to start | `13-onboarding-first-run.md` |
| 132 | No confirmation that account creation succeeded | `13-onboarding-first-run.md` |
| 133 | Email verification blocks all access with no fallback | `13-onboarding-first-run.md`, `19-auth-account-ux.md` |
| 134 | Blaming/accusatory tone in copy | `14-microcopy-content-strategy.md` |
| 135 | Vague, non-actionable error copy | `14-microcopy-content-strategy.md` |
| 136 | Internal/technical jargon exposed to end users | `14-microcopy-content-strategy.md` |
| 137 | Inconsistent terminology for the same concept | `14-microcopy-content-strategy.md`, `25-design-systems-consistency.md` |
| 138 | Button/link label mismatched with destination heading | `14-microcopy-content-strategy.md` |
| 139 | Ambiguous Yes/No confirmation framing | `14-microcopy-content-strategy.md`, `24-error-prevention-recovery.md` |
| 140 | Marketing copy overpromising vs. actual delivery | `14-microcopy-content-strategy.md` |
| 141 | Leftover placeholder/lorem-ipsum text | `14-microcopy-content-strategy.md` |
| 142 | Tone mismatched to stakes (humor in serious contexts) | `14-microcopy-content-strategy.md` |
| 143 | Cryptic abbreviations with no full-text fallback | `14-microcopy-content-strategy.md` |
| 144 | Ambiguous relative/raw timestamps | `14-microcopy-content-strategy.md`, `22-internationalization-localization.md` |
| 145 | Modal overused for low-severity situations | `15-notifications-alerts-modals.md` |
| 146 | Toasts used for info the user needs to act on later | `15-notifications-alerts-modals.md` |
| 147 | Notification fatigue from excessive/unbatched alerts | `15-notifications-alerts-modals.md` |
| 148 | Modal with no close control (X/Escape/backdrop) | `15-notifications-alerts-modals.md` |
| 149 | Unsaved changes silently discarded on modal dismiss | `15-notifications-alerts-modals.md` |
| 150 | Nested/stacked modals | `15-notifications-alerts-modals.md` |
| 151 | Native `alert()`/`confirm()`/`prompt()` used in production | `15-notifications-alerts-modals.md` |
| 152 | Non-dismissible banners permanently consuming space | `15-notifications-alerts-modals.md` |
| 153 | Inconsistent toast position/timing across the app | `15-notifications-alerts-modals.md`, `25-design-systems-consistency.md` |
| 154 | Critical errors communicated only via easily-missed toast | `15-notifications-alerts-modals.md` |
| 155 | Search box with no visible search affordance | `16-search-filtering.md` |
| 156 | Search requiring exact match/case/spelling | `16-search-filtering.md` |
| 157 | Search firing on every keystroke with no debounce | `16-search-filtering.md` |
| 158 | Zero-result state with no guidance | `16-search-filtering.md` |
| 159 | Filters producing zero results with no indication of cause | `16-search-filtering.md` |
| 160 | Filter state lost on navigation/refresh | `16-search-filtering.md` |
| 161 | No result counts shown per filter option | `16-search-filtering.md` |
| 162 | Sort change silently resetting scroll/selection | `16-search-filtering.md` |
| 163 | Typeahead suggestions inconsistent with actual search behavior | `16-search-filtering.md` |
| 164 | Unclear search scope | `16-search-filtering.md` |
| 165 | Table header not fixed/sticky when scrolling | `17-data-tables-dashboards.md` |
| 166 | Numeric columns not right-aligned | `17-data-tables-dashboards.md` |
| 167 | Loading and zero-result table states visually identical | `17-data-tables-dashboards.md` |
| 168 | Pagination with no total-size context | `17-data-tables-dashboards.md` |
| 169 | Infinite scroll used where addressable position matters | `17-data-tables-dashboards.md` |
| 170 | No column customization/persistence on frequently-used tables | `17-data-tables-dashboards.md` |
| 171 | Charts missing axis labels/units/legend | `17-data-tables-dashboards.md` |
| 172 | Chart type mismatched to the data relationship | `17-data-tables-dashboards.md` |
| 173 | Live-updating tables reordering under an active cursor | `17-data-tables-dashboards.md` |
| 174 | Dashboards with no visual hierarchy across metrics | `17-data-tables-dashboards.md` |
| 175 | No export/copy option for valuable table data | `17-data-tables-dashboards.md` |
| 176 | Forced account creation before checkout | `18-checkout-ecommerce.md` |
| 177 | Hidden fees revealed only at the final checkout step | `18-checkout-ecommerce.md` |
| 178 | Fiddly/ambiguous cart quantity or removal controls | `18-checkout-ecommerce.md` |
| 179 | No visible order summary throughout checkout | `18-checkout-ecommerce.md` |
| 180 | Payment form breaking autofill conventions | `18-checkout-ecommerce.md` |
| 181 | Payment failure with no specific reason/recovery path | `18-checkout-ecommerce.md` |
| 182 | No trust signals near the payment step | `18-checkout-ecommerce.md` |
| 183 | Ambiguous pricing (frequency/currency unclear) | `18-checkout-ecommerce.md` |
| 184 | Promo code field mis-prioritized (too prominent or too hidden) | `18-checkout-ecommerce.md` |
| 185 | Unclear order confirmation after purchase | `18-checkout-ecommerce.md` |
| 186 | Irreversible purchase with no final review step | `18-checkout-ecommerce.md`, `24-error-prevention-recovery.md` |
| 187 | Login/signup hard to switch between | `19-auth-account-ux.md` |
| 188 | Forgot-password flow buried or broken | `19-auth-account-ux.md` |
| 189 | Silent session expiry losing in-progress work | `19-auth-account-ux.md` |
| 190 | 2FA/verification codes with no resend/expiry indication | `19-auth-account-ux.md` |
| 191 | Logout needlessly gated behind confirmation, or unsaved state silently lost on logout | `19-auth-account-ux.md` |
| 192 | Permission prompts fired immediately on load with no context | `19-auth-account-ux.md`, `23-privacy-consent-trust.md` |
| 193 | Account deletion impossible to find or dangerously easy to trigger | `19-auth-account-ux.md` |
| 194 | SSO buttons violating provider branding/account clarity | `19-auth-account-ux.md` |
| 195 | Password managers actively fought against (paste blocked) | `19-auth-account-ux.md` |
| 196 | No upload/download progress indication | `20-file-upload-download.md` |
| 197 | Drag-and-drop with no drag-over visual feedback | `20-file-upload-download.md` |
| 198 | Drag-and-drop as the only upload path, no button fallback | `20-file-upload-download.md` |
| 199 | File validation only after full upload completes | `20-file-upload-download.md` |
| 200 | No retry on upload failure, forcing a full restart | `20-file-upload-download.md` |
| 201 | No way to cancel an in-progress upload | `20-file-upload-download.md` |
| 202 | Batch uploads with no per-file status | `20-file-upload-download.md` |
| 203 | Downloads with no confirmation | `20-file-upload-download.md` |
| 204 | Large exports generated synchronously, blocking the UI | `20-file-upload-download.md` |
| 205 | No file preview/review before upload confirmation | `20-file-upload-download.md` |
| 206 | Settings organized by backend model, not user mental model | `21-settings-preferences.md` |
| 207 | Flat, ungrouped, unsearchable settings list | `21-settings-preferences.md` |
| 208 | Settings changes with unclear save/auto-save feedback | `21-settings-preferences.md` |
| 209 | Dangerous settings visually indistinguishable from safe ones | `21-settings-preferences.md`, `24-error-prevention-recovery.md` |
| 210 | No sensible defaults, forcing configuration before usability | `21-settings-preferences.md` |
| 211 | All-or-nothing notification settings | `21-settings-preferences.md` |
| 212 | No live preview of visual settings before committing | `21-settings-preferences.md` |
| 213 | Team-scoped settings changed with no scope warning | `21-settings-preferences.md` |
| 214 | Layout assuming English-length text | `22-internationalization-localization.md` |
| 215 | No RTL layout support | `22-internationalization-localization.md` |
| 216 | Hardcoded locale-specific date/number/currency formatting | `22-internationalization-localization.md` |
| 217 | UI text baked into images | `22-internationalization-localization.md` |
| 218 | Naive string-concatenation pluralization | `22-internationalization-localization.md` |
| 219 | Sentences built from concatenated fragments, not templates | `22-internationalization-localization.md` |
| 220 | Culture-specific icon/color meaning assumed universal | `22-internationalization-localization.md` |
| 221 | Machine-translated copy shipped with no native review | `22-internationalization-localization.md` |
| 222 | Language switcher hard to find or non-persistent | `22-internationalization-localization.md` |
| 223 | Consent banner with asymmetric accept/reject friction | `23-privacy-consent-trust.md` |
| 224 | Consent banner reappearing despite a prior choice | `23-privacy-consent-trust.md` |
| 225 | Consent banner blocking unrelated page content | `23-privacy-consent-trust.md` |
| 226 | Data collection with no purpose disclosure at point of collection | `23-privacy-consent-trust.md` |
| 227 | Marketing consent bundled with required functional consent | `23-privacy-consent-trust.md` |
| 228 | No self-service view/export/delete of personal data | `23-privacy-consent-trust.md` |
| 229 | Third-party trackers loaded before consent decision | `23-privacy-consent-trust.md` |
| 230 | AI-driven features with no disclosure/caveat | `23-privacy-consent-trust.md` |
| 231 | Destructive actions with no confirmation | `24-error-prevention-recovery.md` |
| 232 | Confirmation dialogs overused on low-stakes actions | `24-error-prevention-recovery.md` |
| 233 | No undo path for reversible actions | `24-error-prevention-recovery.md` |
| 234 | Reversibility implied incorrectly by wording | `24-error-prevention-recovery.md` |
| 235 | Bulk destructive actions with no per-item review | `24-error-prevention-recovery.md` |
| 236 | Invalid states not structurally prevented at input time | `24-error-prevention-recovery.md` |
| 237 | Auto-save with no version history to recover a bad state | `24-error-prevention-recovery.md` |
| 238 | High-risk warnings with no proportionate friction | `24-error-prevention-recovery.md` |
| 239 | No shared design tokens, ad hoc hardcoded values | `25-design-systems-consistency.md` |
| 240 | Same UI concept reimplemented multiple divergent ways | `25-design-systems-consistency.md` |
| 241 | Interaction patterns behaving differently across the app | `25-design-systems-consistency.md` |
| 242 | Inconsistent icon/terminology mapping for the same concept | `25-design-systems-consistency.md` |
| 243 | Uncontrolled component-variant proliferation | `25-design-systems-consistency.md` |
| 244 | Mismatched visual "eras" across old and new screens | `25-design-systems-consistency.md` |
| 245 | Spacing/alignment drift between adjacent components | `25-design-systems-consistency.md` |

## Part 2: Additional Miscellaneous Anti-Patterns

### Anti-pattern: No favicon, or a generic/default one
**What's wrong:** A shipped product with the framework's default favicon (or none at all) still showing in browser tabs.
**Why it matters:** With many tabs open, a missing/generic favicon makes the product impossible to spot at a glance — a small but real everyday cost for a returning user.
**Fix:** Ship a proper favicon (including at minimum a 32×32 and a larger PNG/SVG variant, plus `apple-touch-icon` for iOS home-screen bookmarks) as a standard part of setup.

### Anti-pattern: Generic or missing page `<title>` and meta description
**What's wrong:** Every page shares one generic `<title>` ("App"), or link previews (social shares) show blank/placeholder text.
**Why it matters:** Breaks browser tab identification (especially with multiple tabs open), search engine indexing, and social share previews — all real, user-facing consequences of a purely "backend" oversight.
**Fix:** Set a unique, descriptive `<title>` per page/route, and Open Graph / meta description tags for anything likely to be shared.

### Anti-pattern: No 404 or generic-server-error page customized for the product
**What's wrong:** A broken link shows the hosting platform's raw default error page rather than anything branded/helpful.
**Why it matters:** Breaks continuity of experience at exactly the failure moment users most need reassurance and a way forward — see `08-navigation-ia.md`'s dead-end pattern.
**Fix:** Build custom 404/500 pages consistent with the rest of the product, each offering a clear next action.

### Anti-pattern: Copy-to-clipboard actions with no confirmation
**What's wrong:** Clicking a "copy" icon (for a link, code snippet, API key) copies silently with zero visible acknowledgment.
**Why it matters:** Users can't be sure the copy actually worked (clipboard permission issues, browser quirks) without an explicit signal.
**Fix:** Show a brief inline confirmation ("Copied!" tooltip or icon swap) immediately after a successful copy action.

### Anti-pattern: Long-running background jobs with no way to know they finished unless the tab stays open
**What's wrong:** A report/export/AI job runs for minutes; if the user navigates away or closes the tab, there's no way to find out it completed.
**Why it matters:** Forces users to babysit a tab for an operation that should be fire-and-forget.
**Fix:** Persist job status server-side and surface completion via a durable channel — an in-app notification center the user sees on return, and/or email for long jobs — not only an in-memory tab-bound state.

### Anti-pattern: No offline handling — the app breaks silently when connectivity drops
**What's wrong:** Losing network mid-session causes requests to fail with generic errors or hang indefinitely, with no explicit "you're offline" state.
**Why it matters:** Users on unreliable connections (mobile, transit, poor wifi) hit this constantly; without explicit offline detection, every dropped request looks like an app bug rather than a connectivity issue.
**Fix:** Detect offline state (`navigator.onLine` plus request-failure heuristics) and show a persistent, clear "You're offline" indicator; queue/retry actions where feasible once connectivity returns, rather than silently failing.

### Anti-pattern: Keyboard shortcuts with no discoverability and no way to see what's available
**What's wrong:** Power-user keyboard shortcuts exist but are undocumented anywhere in the UI.
**Why it matters:** Shortcuts that can't be discovered provide zero value to anyone who doesn't already know about them from outside the product.
**Fix:** Provide a shortcuts reference (commonly triggered via `?` or in a help menu) listing all available shortcuts, and consider a brief contextual hint the first time a relevant action is available.

### Anti-pattern: Drag-to-reorder with no keyboard alternative
**What's wrong:** Reordering a list (tasks, priorities) is only possible via mouse/touch drag, with no keyboard-accessible alternative.
**Why it matters:** Drag interactions are among the hardest patterns to make keyboard/screen-reader accessible, and are frequently shipped without any fallback at all.
**Fix:** Pair drag-to-reorder with an explicit keyboard alternative (e.g. focus an item, then use "Move up"/"Move down" buttons or arrow-key + modifier reordering with announced position changes via `aria-live`).

### Anti-pattern: Print stylesheet missing for content users are likely to print
**What's wrong:** Invoices, receipts, tickets, or reports render for print with the full app chrome (nav, sidebar, buttons) included, wasting paper and looking unprofessional.
**Why it matters:** Some content types are routinely printed regardless of how "web-native" the product otherwise is; an unstyled print output undermines an otherwise polished product at a moment (often a receipt or contract) where it matters.
**Fix:** Provide a `@media print` stylesheet for print-likely content that hides navigational chrome and formats cleanly for paper.

### Anti-pattern: Favoring novelty interaction patterns over established conventions with no real benefit
**What's wrong:** Inventing a custom, unfamiliar interaction (a non-standard scroll-jacking effect, an unconventional custom scrollbar, a bespoke non-standard select control) purely for visual distinctiveness, when a standard pattern would serve the same function with zero relearning cost.
**Why it matters:** Per Jakob's Law (`01-philosophy-and-principles.md`), users bring expectations from every other product they've used — novel patterns impose a real, unrewarded learning cost when they don't provide a genuine functional improvement over the familiar option.
**Fix:** Default to established conventions for functional interactions; reserve genuine novelty for moments where it serves the brand/concept meaningfully (see the creative-direction guidance for brand-forward work) and never at the expense of basic usability.

### Anti-pattern: Testing only on the latest version of one browser
**What's wrong:** UI verified only in the developer's own up-to-date Chrome, never checked against Safari, Firefox, or older/other-engine browsers actually used by the target audience.
**Why it matters:** CSS features, form behaviors, and even basic layout can render differently across engines — cross-browser bugs are a common, avoidable source of "broken for some users, fine for others" reports.
**Fix:** At minimum spot-check core flows in the major engines relevant to your audience (Chromium, WebKit/Safari, Firefox) before shipping a significant UI change, with particular attention to newer CSS features that may have inconsistent support.

## Quick Checklist (Part 2 additions)
- [ ] Proper favicon and app icons are set, not framework defaults
- [ ] Every page has a unique, descriptive title and appropriate share/meta tags
- [ ] Custom, on-brand 404/error pages exist with a forward action
- [ ] Copy-to-clipboard actions show explicit confirmation
- [ ] Long-running jobs persist status server-side and notify durably, not just in-tab
- [ ] Offline state is explicitly detected and communicated, with retry/queue where feasible
- [ ] Keyboard shortcuts are discoverable via a reference, not hidden knowledge
- [ ] Drag-to-reorder has a keyboard-accessible alternative
- [ ] Print-likely content has a dedicated print stylesheet
- [ ] Novel interaction patterns are reserved for genuine benefit, not novelty for its own sake
- [ ] Core flows are spot-checked across major browser engines before shipping
