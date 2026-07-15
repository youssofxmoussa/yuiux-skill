# 27 — Content Strategy & Microcopy

> Every word in a UI is a design decision. The difference between "Submit" and "Create my account" is the difference between a transaction and a relationship.

---

## Voice and Tone

### Voice (Constant)
Your brand's personality — doesn't change.

Example brand voices:
- **Stripe**: Professional, precise, treats developers as experts
- **Slack**: Friendly, warm, conversational but not silly
- **Notion**: Thoughtful, empowering, calm
- **Mailchimp**: Playful, friendly, occasionally witty

### Tone (Variable)
Adapts to the user's emotional state and context.

```
User action/context → Appropriate tone

User signs up         → Warm, welcoming, excited
User succeeds         → Celebratory, affirming (briefly)
User makes mistake    → Helpful, clear, not accusatory
User faces error      → Calm, reassuring, action-oriented
User does danger act  → Direct, serious, clear consequences
User contacts support → Empathetic, patient, efficient
```

---

## Microcopy Formulas

### Button Copy
```
Formula: [Verb] + [Noun] (optional: + [Context])

❌ Generic          ✅ Specific
OK                 Save changes
Submit             Create my account
Yes                Delete workspace
Confirm            Send $250 to @username
Next               Next: shipping address
Done               Finish and publish
Continue           Continue to payment
```

### Error Messages
```
Formula: [What failed] + [Why it failed] + [How to fix it]

❌ "Error"
✅ "Couldn't send message. Your session expired. Sign in again to send."

❌ "Invalid email"
✅ "Enter a valid email like name@example.com"

❌ "Connection failed"
✅ "Can't connect to the server. Check your internet connection and try again."

❌ "Something went wrong"
✅ "We couldn't load your projects. This is on our end — try refreshing the page."
```

### Success Messages
```
Formula: [What succeeded] (+ [what happens next])

❌ "Done!"
✅ "Project created. You'll receive an email when your team joins."

❌ "Success"
✅ "Changes saved"

❌ "Great job!"
✅ "File uploaded successfully"

Limit celebratory language — reserve it for genuine wins (first purchase, goal reached)
```

### Empty States
```
Formula: [What's missing] → [Why] → [What to do]

❌ "No data available"
✅ "No projects yet. Create your first project to start collaborating."

❌ "0 results"
✅ "No results for 'ux design'. Try a different search term or clear your filters."
```

### Placeholder Text
```
Never use placeholder as label.
Placeholder should only show example format, not instructions.

❌ "Enter your email address here" (instruction in placeholder)
❌ "Email" (just the label — put this in the label, not placeholder)

✅ Label: "Email address"  Placeholder: "you@company.com"
✅ Label: "Phone"          Placeholder: "+1 (555) 000-0000"
✅ Label: "Date"           Placeholder: "MM/DD/YYYY"
```

---

## Headline Writing Rules

```
1. Lead with the benefit, not the feature
   ❌ "Advanced export options"
   ✅ "Get your data in any format"

2. Active voice over passive
   ❌ "Your account has been created"
   ✅ "Account created"

3. Be specific
   ❌ "New features available"
   ✅ "Table columns can now be sorted by clicking any header"

4. Present tense for persistent UI, past tense for completed actions
   ❌ "You have saved your changes" (awkward)
   ✅ "Changes saved"
   ✅ "Save your changes" (present, action prompt)

5. Don't announce, just show
   ❌ "We're excited to announce that you can now invite guests!"
   ✅ "Invite guests" (just the feature, in the UI)
```

---

## Tone Rules for Specific Contexts

### Confirmation Dialogs

```
Heading:    Specific question (names the thing being affected)
Body:       Clear consequences (what will be destroyed/changed)
Primary:    Action verb + object (matches heading)
Cancel:     "Cancel" or "Keep [thing]" (never just "No")

Example:
  Heading: Delete "Marketing Q3 2026" campaign?
  Body:    This will permanently delete the campaign and all 14 scheduled posts.
           Your analytics data will be preserved.
  Primary: [Delete campaign]   ← destructive styling (red)
  Cancel:  [Keep campaign]     ← default styling

NOT:
  Heading: Are you sure?
  Body:    This action cannot be undone.
  Primary: [Yes]   ← ambiguous
  Cancel:  [No]    ← ambiguous
```

### Disabled States

```
Explain why the action is unavailable.

❌ Just a grayed out button with no explanation

✅ Button + tooltip: "Publish is disabled until you add at least one product."
✅ Button + inline note: "Add a payment method to enable this feature."
✅ "Upgrade to Pro to export data" (with upgrade link)
```

### Asking for Personal Data

```
Always explain why you need it.

❌ "Phone number" (just a label)
✅ Label: "Phone number"
   Helper: "We'll only use this to send you account security alerts."

❌ "Birthday"
✅ Label: "Date of birth"
   Helper: "Required to verify you're 18+ for this service."
```

---

## Writing for Internationalization

Even if you're only building in English now, write copy that translates well:

```
1. Avoid idioms and colloquialisms
   ❌ "Piece of cake!"     (idiom)
   ❌ "Hang tight…"        (idiom)
   ✅ "Done!" / "Loading…"

2. Don't embed numbers in strings
   ❌ "You have 3 messages" (can't pluralize easily)
   ✅ t('messages', { count: 3 }) → "3 messages" / "1 message"

3. Leave room for expansion (some languages are 30–50% longer)
   English: "Save changes"  (12 chars)
   German:  "Änderungen speichern"  (21 chars)
   Design for 40% more width.

4. Avoid gendered language where possible
   ❌ "He or she can access…"
   ✅ "Users can access…" / "You can access…"

5. Use formal forms by default when in doubt
   Spanish: "usted" over "tú" for B2B
```

---

## Capitalization Rules

```
Title Case: Headings, dialog titles, navigation items, button labels, menu items
Sentence case: Body text, descriptions, helper text, placeholder text, tooltips

Title Case examples:
  "Create New Project"        (navigation action)
  "Account Settings"          (page heading)
  "Permanently Delete Workspace" (dialog heading)

Sentence case examples:
  "Enter a valid email address"   (error message)
  "Your account was created"      (success toast)
  "Projects help you organize…"  (description)
  
NEVER ALL CAPS for text (use CSS text-transform instead)
Okay to use ALL CAPS for: abbreviations (USA, PDF, 2FA), acronyms
```

---

## Number Formatting

```typescript
const formatters = {
  // User-facing amounts
  currency: (amount: number, currency = 'USD') =>
    new Intl.NumberFormat('en-US', { style: 'currency', currency }).format(amount),
  // → "$1,234.56"
  
  // Compact for dashboards
  compact: (n: number) =>
    new Intl.NumberFormat('en-US', { notation: 'compact' }).format(n),
  // → "12.4K" / "1.2M"
  
  // Percentages
  percent: (n: number, digits = 1) =>
    new Intl.NumberFormat('en-US', { 
      style: 'percent', 
      minimumFractionDigits: digits,
      maximumFractionDigits: digits 
    }).format(n),
  // → "12.4%"
  
  // Large numbers with comma separator
  number: (n: number) =>
    new Intl.NumberFormat('en-US').format(n),
  // → "1,234,567"
  
  // File sizes
  fileSize: (bytes: number) => {
    const units = ['B', 'KB', 'MB', 'GB', 'TB'];
    let i = 0;
    while (bytes >= 1024 && i < units.length - 1) { bytes /= 1024; i++; }
    return `${bytes.toFixed(i > 0 ? 1 : 0)} ${units[i]}`;
  },
  // → "1.2 MB"
};
```

---

## Date and Time Formatting

```typescript
const dateFormatters = {
  // Absolute: when precision matters
  full: new Intl.DateTimeFormat('en-US', { 
    dateStyle: 'long', timeStyle: 'short' 
  }),
  // → "July 14, 2026 at 3:30 PM"
  
  date: new Intl.DateTimeFormat('en-US', { dateStyle: 'medium' }),
  // → "Jul 14, 2026"
  
  // Relative: when recency matters
  relative: (date: Date) => {
    const now = Date.now();
    const diff = now - date.getTime();
    const seconds = Math.floor(diff / 1000);
    
    if (seconds < 60)  return 'Just now';
    if (seconds < 3600) return `${Math.floor(seconds / 60)}m ago`;
    if (seconds < 86400) return `${Math.floor(seconds / 3600)}h ago`;
    if (seconds < 604800) return `${Math.floor(seconds / 86400)}d ago`;
    
    // Fall back to absolute for anything older
    return dateFormatters.date.format(date);
  },
};

// Show absolute date on hover (tooltip) when using relative
function RelativeTime({ date }: { date: Date }) {
  return (
    <time
      dateTime={date.toISOString()}
      title={dateFormatters.full.format(date)}
    >
      {dateFormatters.relative(date)}
    </time>
  );
}
```

---

## Microcopy Checklist

- [ ] All button copy follows Verb + Noun formula
- [ ] Error messages explain cause + resolution (not just "Error")
- [ ] Empty states have descriptive title + what to do
- [ ] Placeholder text shows format example, not instruction
- [ ] Labels are in every input (not only placeholders)
- [ ] Confirmation dialogs name the specific thing being affected
- [ ] Confirmation dialog primary button names the action ("Delete campaign")
- [ ] Success messages are brief and past tense
- [ ] Numbers formatted with `Intl.NumberFormat` (locale-aware)
- [ ] Dates show absolute on hover when using relative format
- [ ] No idioms or colloquialisms (for internationalization)
- [ ] Strings don't embed counts inline (use i18n pluralization)
- [ ] Headings: Title Case; body text: Sentence case
- [ ] Disabled buttons explain why (tooltip or inline)
- [ ] Personal data requests explain why the data is needed
