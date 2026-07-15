---
name: internationalization-localization
description: RTL layout, text expansion, locale formatting, and translation-pitfall errors and fixes for web UI.
---

# Internationalization & Localization

## Anti-pattern: Layout assumes English-length text
**What's wrong:** Buttons, labels, and nav items sized exactly to fit their English text with no slack, breaking as soon as translated into a language that runs longer (German, Finnish, and many others routinely run 30–200% longer than English for the same meaning).
**Why it matters:** Text overflow, truncation, or wrapping in unexpected places makes a localized UI look broken even when the translation itself is accurate.
**Fix:** Design with generous flexible-width containers (avoid fixed pixel widths on text-holding elements), test key screens with a deliberately long placeholder string (or actual German/Finnish text) even if you only ship English initially, and ensure buttons/labels wrap or the container grows gracefully rather than clipping.

## Anti-pattern: No RTL (right-to-left) support considered
**What's wrong:** A UI built assuming left-to-right reading order everywhere — icons pointing the wrong direction, layout not mirrored — shipped as-is to Arabic/Hebrew/Farsi-speaking users.
**Why it matters:** For RTL languages, failing to mirror the layout (nav on the wrong side, "back" arrows pointing the wrong way, form fields in the wrong reading order) actively disrupts comprehension rather than just looking slightly off.
**Fix:** Use CSS logical properties (`margin-inline-start` instead of `margin-left`, etc.) instead of physical direction properties wherever layout direction should follow text direction, set `dir="rtl"` on `<html>` when serving an RTL locale, and mirror directional icons (arrows, chevrons) that indicate navigation direction — but do not mirror icons whose meaning is direction-independent (e.g. a play button, a checkmark).

```css
.card {
  margin-inline-start: var(--space-4); /* flips automatically under dir="rtl" */
}
```

## Anti-pattern: Dates, numbers, and currency hardcoded to one locale's format
**What's wrong:** Dates always rendered as `MM/DD/YYYY`, numbers always using a `,` thousands separator, regardless of the viewing user's locale.
**Why it matters:** `03/04/2026` means March 4th to an American user and April 3rd to nearly everyone else — this specific ambiguity causes real, costly misunderstandings (missed deadlines, wrong appointments).
**Fix:** Use locale-aware formatting APIs rather than hand-built string formatting:

```js
new Intl.DateTimeFormat(userLocale, { dateStyle: "long" }).format(date);
new Intl.NumberFormat(userLocale, { style: "currency", currency: "EUR" }).format(amount);
```
When ambiguity is unavoidable (e.g. displaying a date without knowing the viewer's locale preference), prefer an unambiguous format like "March 4, 2026" or ISO "2026-03-04" over the ambiguous numeric-only forms.

## Anti-pattern: Text baked into images
**What's wrong:** UI text (labels, badges, instructional graphics) rendered as part of a static image file rather than real, selectable HTML text.
**Why it matters:** Text-in-images can't be translated without regenerating the image, can't be read by screen readers (beyond a generic `alt`), can't be resized/zoomed, and doesn't respect the user's font preferences.
**Fix:** Keep all UI text as real HTML/CSS text, even when it's styled to look highly custom; reserve images for genuinely photographic or illustrative content, not text delivery.

## Anti-pattern: Pluralization handled with naive string concatenation
**What's wrong:** `"${count} item(s)"` or `count + " items"` regardless of whether count is 1, hardcoding English pluralization rules (many languages have far more plural forms — zero, one, few, many, other — with different rules entirely).
**Why it matters:** Naive pluralization either reads awkwardly in English ("1 item(s)") or is flatly wrong once translated to a language with different plural rules.
**Fix:** Use a real i18n/pluralization library (most i18n frameworks provide `Intl.PluralRules` or equivalent) that selects the correct grammatical form per locale rather than hardcoding an English-only rule.

## Anti-pattern: String concatenation used to build sentences around variables
**What's wrong:** `"Hi " + name + ", you have " + count + " new messages"` built by joining fragments, rather than using a single translatable template string.
**Why it matters:** Word order varies by language — concatenated fragments can't be reordered by a translator without breaking the code, so translations end up grammatically wrong or impossible.
**Fix:** Use a single template string per complete sentence/phrase, with named placeholders, run through a proper i18n templating mechanism:

```js
t("greeting.newMessages", { name, count });
// en: "Hi {{name}}, you have {{count}} new messages"
// A translator can freely reorder {{name}}/{{count}} for their language's grammar.
```

## Anti-pattern: Icons/imagery with culture-specific meaning assumed to be universal
**What's wrong:** A thumbs-up, an OK hand gesture, or a specific color association (e.g. red = danger everywhere, white = purity everywhere) used without considering that meaning varies across cultures.
**Why it matters:** Some common Western icon/color conventions carry different — sometimes offensive — meanings elsewhere; treating them as universal risks real cultural missteps for a global audience.
**Fix:** For any icon/color choice carrying cultural weight (rather than a now-truly-universal digital convention like a magnifying glass for search), research target-market connotations if the product ships broadly, and prefer neutral, widely-tested iconography over anything with strong cultural specificity.

## Anti-pattern: Machine-translated copy shipped with no native review
**What's wrong:** UI strings run through automatic translation and shipped without a native speaker ever reviewing them.
**Why it matters:** Machine translation frequently produces grammatically correct but unnatural, awkward, or occasionally embarrassingly wrong phrasing — especially for idioms, tone, and product-specific terminology.
**Fix:** Treat machine translation as a first draft only; have a native speaker (or professional localization review) check tone and correctness before shipping to that locale, especially for anything customer-facing at scale.

## Anti-pattern: Language switcher hard to find or resets on every session
**What's wrong:** No visible way to change language, or the language selection doesn't persist across sessions/page loads.
**Why it matters:** Forces the user to relocate and re-select the language switcher on every visit.
**Fix:** Make the language switcher easy to find (commonly in a header or footer, consistently placed across the site) and persist the choice (cookie/localStorage/account preference) so it doesn't need to be reselected every visit.

## Quick Checklist
- [ ] Text containers are flexible-width, tested against deliberately longer strings than the source language
- [ ] CSS logical properties used instead of physical left/right where layout should mirror for RTL; `dir="rtl"` set correctly for RTL locales; directional icons mirrored appropriately
- [ ] Dates/numbers/currency formatted via locale-aware `Intl` APIs, never hand-built strings
- [ ] UI text is real HTML text, never baked into images
- [ ] Pluralization uses a real i18n library, not naive `count + "s"` logic
- [ ] Sentences are built from single translatable templates with named placeholders, never concatenated fragments
- [ ] Icons/colors with cultural specificity are reviewed for the target market, not assumed universal
- [ ] Machine-translated copy is reviewed by a native speaker before shipping
- [ ] Language switcher is easy to find and the choice persists across sessions
