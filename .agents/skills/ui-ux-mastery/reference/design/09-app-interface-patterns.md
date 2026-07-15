---
name: app-interface-patterns
description: Mobile/native app interface rules — accessibility, touch, navigation, forms, performance, animation, typography, safe areas, theming — condensed from a 30-row cross-platform (iOS/Android/React Native) checklist. Raw table in data/app-interface.csv.
---

# App Interface Patterns (iOS / Android / React Native)

Full raw data (30 rows: category, issue, description, do/don't, code examples, severity) lives in `../../data/app-interface.csv`. This file is the condensed, prose version; consult the CSV for exact code snippets per rule.

## Accessibility (Critical severity items first)
- Icon-only buttons need `accessibilityLabel`/`label`; inputs need a paired visible `<Text>` label *and* an accessibility label — placeholder-only inputs fail screen readers.
- Interactive elements expose correct `accessibilityRole` (button/link/checkbox) rather than relying on bare `View`/`onTouchStart` handlers.
- Async status changes use `accessibilityLiveRegion="polite"` so screen readers announce updates without the user needing to re-focus.
- Decorative icons are marked `accessible={false}` so screen readers don't read every glyph on a busy screen.

## Touch (Critical/High)
- Primary touch targets ≥44×44pt; use `hitSlop` to pad small icons rather than shrinking the tappable area.
- ≥8dp spacing between adjacent touchables to prevent mis-taps.
- Never let a custom gesture handler (e.g. `PanResponder` spanning the full screen) swallow the system back-gesture or vertical scroll — scope custom gestures to their specific component (e.g. a horizontal carousel inside a vertical scroll view).

## Navigation (Critical/High/Medium)
- Back navigation must be predictable and preserve screen state (`navigation.goBack()` — never `BackHandler.exitApp()` on first back-press).
- Bottom tab bars: 3-5 primary items; overflow items go behind a "More" tab, not crammed into a 7-icon bar.
- Modals/sheets need an explicit close button in addition to swipe-down/backdrop-tap.
- Returning to a previously visited tab/screen should restore its scroll position and form state (`unmountOnBlur: false`), not reset silently.

## Feedback & Forms (High/Medium)
- Any network operation shows a visible loading indicator (`ActivityIndicator` or skeleton) for anything the user might perceive as a pause — never leave a button silently frozen.
- Success is confirmed (toast/checkmark/banner); failure shows an inline error next to the field *and* a summary, not just a red border with no explanation.
- Validate on blur/submit, not on every keystroke (per-keystroke validation causes visible jank and interrupts typing flow).
- Match `keyboardType` to the field's content (`email-address`, `numeric`, `phone-pad`) and chain `onSubmitEditing` to auto-advance focus to the next field.
- Password fields always offer a show/hide toggle rather than forcing blind typing.

## Performance (High/Medium)
- Lists over ~50 items use `FlatList`/`SectionList` with `keyExtractor`, never a `ScrollView` wrapping a raw `.map()`.
- Images are requested at display resolution with appropriate `resizeMode`, not full-resolution assets shrunk visually.
- High-frequency handlers (scroll, search-as-you-type) are debounced/throttled.

## Animation (Critical/Medium)
- Micro-interaction durations: 150-300ms with platform-native easing (ease-out enter, ease-in exit) — avoid long, linear animations on core UI.
- Always check the OS "reduce motion" accessibility setting and skip/simplify complex animation when it's enabled — this is a Critical-severity rule, not optional polish.
- Reserve continuously looping animation for loaders and genuinely live data; decorative elements should not loop forever.

## Typography, Safe Areas & Theming (High)
- Base body text ≥14-16pt and support platform dynamic type scaling (`allowFontScaling` on, don't globally disable it).
- Wrap screens in `SafeAreaView`/apply safe-area insets so content never sits under a notch or gesture bar.
- Dark mode uses purpose-built semantic tokens with verified contrast — never reuse light-mode gray values directly against a dark background (common cause of failing contrast in "quick" dark-mode ports).

## Anti-pattern (Critical)
- Never make a core action reachable *only* through a hidden gesture (shake-to-undo, secret swipe) with no visible UI affordance — always provide a discoverable button alongside any gesture shortcut.
