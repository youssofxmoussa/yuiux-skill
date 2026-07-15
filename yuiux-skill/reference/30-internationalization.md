# 30 — Internationalization (i18n)

> i18n is not just translation. It's designing a UI that works for human beings everywhere — different languages, different writing directions, different number formats, different cultural norms.

---

## i18n Fundamentals

```
i18n  = internationalization (making the app adaptable)
l10n  = localization (adapting for a specific locale)
locale = language + region (e.g., en-US, fr-FR, ar-SA, zh-CN)
```

---

## Text Externalization

Every visible string must be in a message catalog — never hardcoded.

```typescript
// ❌ Hardcoded English
<p>You have 3 messages</p>

// ✅ Externalized key
<p>{t('inbox.messageCount', { count: 3 })}</p>

// Message catalog (en.json)
{
  "inbox": {
    "messageCount_one":   "You have {{count}} message",
    "messageCount_other": "You have {{count}} messages"
  }
}

// Message catalog (fr.json)
{
  "inbox": {
    "messageCount_one":   "Vous avez {{count}} message",
    "messageCount_other": "Vous avez {{count}} messages"
  }
}
```

### i18next Setup (React)

```typescript
// i18n.ts
import i18n from 'i18next';
import { initReactI18next } from 'react-i18next';
import LanguageDetector from 'i18next-browser-languagedetector';

import enUS from './locales/en-US.json';
import frFR from './locales/fr-FR.json';
import arSA from './locales/ar-SA.json';

i18n
  .use(LanguageDetector)
  .use(initReactI18next)
  .init({
    resources: {
      'en-US': { translation: enUS },
      'fr-FR': { translation: frFR },
      'ar-SA': { translation: arSA },
    },
    fallbackLng: 'en-US',
    interpolation: { escapeValue: false }, // React escapes by default
    detection: {
      order: ['localStorage', 'navigator'],
      caches: ['localStorage'],
    },
  });

export default i18n;
```

---

## Pluralization

Different languages have different pluralization rules. Never do this manually:

```typescript
// ❌ English-only pluralization
const msg = count === 1 ? '1 item' : `${count} items`;

// ✅ i18next handles all languages
const msg = t('items', { count }); // uses _one, _other, _few, _many, _zero keys

// Catalog (en-US.json)
{
  "items_zero":  "No items",
  "items_one":   "{{count}} item",
  "items_other": "{{count}} items"
}

// Catalog (ar-SA.json) — Arabic has 6 plural forms
{
  "items_zero":  "لا توجد عناصر",
  "items_one":   "عنصر واحد",
  "items_two":   "عنصران",
  "items_few":   "{{count}} عناصر",
  "items_many":  "{{count}} عنصراً",
  "items_other": "{{count}} عنصر"
}
```

---

## Number Formatting

```typescript
// Always use Intl.NumberFormat — never format numbers manually
const locale = useLocale(); // 'en-US', 'de-DE', 'fr-FR', etc.

// Numbers
new Intl.NumberFormat(locale).format(1234567.89);
// en-US: "1,234,567.89"
// de-DE: "1.234.567,89"  (comma/period swapped!)
// fr-FR: "1 234 567,89"

// Currency
new Intl.NumberFormat(locale, {
  style: 'currency',
  currency: 'EUR',
}).format(1234.56);
// en-US: "€1,234.56"
// de-DE: "1.234,56 €"
// fr-FR: "1 234,56 €"

// Percentage
new Intl.NumberFormat(locale, {
  style: 'percent',
  minimumFractionDigits: 1,
}).format(0.124);
// en-US: "12.4%"
// de-DE: "12,4 %"

// Compact notation
new Intl.NumberFormat(locale, { notation: 'compact' }).format(12400);
// en-US: "12.4K"
// fr-FR: "12,4 k"
// ja-JP: "1.24万"
```

---

## Date and Time Formatting

```typescript
const locale = useLocale();

// Full date
new Intl.DateTimeFormat(locale, { dateStyle: 'long' }).format(date);
// en-US: "July 14, 2026"
// de-DE: "14. Juli 2026"
// ja-JP: "2026年7月14日"

// Relative time
const rtf = new Intl.RelativeTimeFormat(locale, { numeric: 'auto' });
rtf.format(-1, 'day');   // en-US: "yesterday"  de-DE: "gestern"
rtf.format(-3, 'day');   // en-US: "3 days ago"  de-DE: "vor 3 Tagen"
rtf.format(2,  'hour');  // en-US: "in 2 hours"  fr-FR: "dans 2 heures"

// Time zones
new Intl.DateTimeFormat(locale, {
  timeStyle: 'short',
  timeZone: 'America/New_York',
}).format(date);

// Always store dates as UTC, display in user's timezone
const userTimeZone = Intl.DateTimeFormat().resolvedOptions().timeZone;
```

---

## RTL (Right-to-Left) Support

Arabic, Hebrew, Persian, Urdu are RTL. This requires significant layout changes.

```html
<!-- Set direction at document level -->
<html lang="ar" dir="rtl">
```

```css
/* Use logical properties — they flip with writing direction */

/* ❌ Physical properties — don't flip */
.nav { margin-left: 16px; padding-right: 24px; }
.icon { float: left; }
.dropdown { left: 0; }
.badge { right: -4px; top: -4px; }

/* ✅ Logical properties — flip automatically in RTL */
.nav { margin-inline-start: 16px; padding-inline-end: 24px; }
.icon { float: inline-start; }
.dropdown { inset-inline-start: 0; }
.badge { inset-inline-end: -4px; inset-block-start: -4px; }

/* Logical property reference */
margin-left  → margin-inline-start   (left in LTR, right in RTL)
margin-right → margin-inline-end     (right in LTR, left in RTL)
padding-left → padding-inline-start
padding-right→ padding-inline-end
left         → inset-inline-start
right        → inset-inline-end
top          → inset-block-start
bottom       → inset-block-end
border-left  → border-inline-start
border-right → border-inline-end
text-align: left  → text-align: start
text-align: right → text-align: end
```

```tsx
// RTL-aware icon mirroring
// Some icons need to be mirrored in RTL (arrows, chevrons)
// Others must NOT be mirrored (logos, checkmarks, ❌)

function DirectionalIcon({ icon: Icon, ...props }) {
  const { dir } = useDirection(); // 'ltr' | 'rtl'
  
  return (
    <Icon
      {...props}
      style={{
        transform: dir === 'rtl' ? 'scaleX(-1)' : undefined,
      }}
    />
  );
}

// Use for: ChevronRight, ArrowRight, ArrowLeft, ChevronLeft
// Don't use for: CheckCircle, X, *, !, logos
```

---

## String Layout Considerations

Different languages expand text significantly:

```
English:  "Save changes"          (12 chars)
German:   "Änderungen speichern"  (21 chars) — +75%
Finnish:  "Tallenna muutokset"    (18 chars) — +50%
Russian:  "Сохранить изменения"  (19 chars) — +58%

Design rule: design for text 40% longer than English.
Test with pseudo-localization.
```

```bash
# Pseudo-localization — replaces English letters with accented variants
# to simulate text expansion without real translation

"Save changes" → "[Şàvé çhàngéš 1234567890]"
# Longer, accented, detects hardcoded length issues
```

---

## i18n Checklist

- [ ] All user-visible strings in message catalogs (no hardcoded English)
- [ ] Plural forms handled with i18next `_one`, `_other` keys
- [ ] Numbers formatted with `Intl.NumberFormat` (not manual formatting)
- [ ] Dates formatted with `Intl.DateTimeFormat` (not manual formatting)
- [ ] Currency formatted with `Intl.NumberFormat` (includes symbol placement)
- [ ] Dates stored as UTC, displayed in user's timezone
- [ ] RTL layout uses logical CSS properties (`inline-start` not `left`)
- [ ] `dir="rtl"` set on `<html>` for RTL locales
- [ ] Directional icons (arrows, chevrons) mirrored in RTL
- [ ] Text containers allow 40% expansion without breaking layout
- [ ] Tested with pseudo-localization
- [ ] Font supports all target character sets
- [ ] No images containing text (can't translate)
- [ ] Date/time format respects locale (not hardcoded US format)
- [ ] Language switcher accessible in the UI
