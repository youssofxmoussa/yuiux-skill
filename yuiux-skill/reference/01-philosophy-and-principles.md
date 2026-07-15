# 01 — Philosophy & Principles

> The intellectual foundation of YUIUX. Every rule in every other file traces back to something here. Read this when something "feels wrong" but you can't name why.

---

## Why Principles Before Patterns

Patterns age. Principles don't. The hamburger menu, the infinite scroll, the toast notification — these are patterns, and they each have good and bad applications. The principles that tell you *when* to use them don't change.

When an AI generates a UI, it defaults to patterns. It puts a sidebar here, a card grid there, a toast for every action. That's pattern application without principle. The result feels generic — technically correct but emotionally inert. 

YUIUX applies principles first, then selects patterns that express them.

---

## Part 1 — Nielsen's 10 Heuristics (Deep Version)

Jakob Nielsen's heuristics are the foundational principles of usability, validated across 40+ years of empirical research. Every UI failure maps to at least one violation.

### Heuristic 1: Visibility of System Status

**Principle:** The system should always keep users informed about what is going on, through appropriate feedback within reasonable time.

**What this means in practice:**
- Every user action must produce a visible response within 100ms
- Every background operation must communicate its state (progress, success, failure)
- The user must always know: what the system is doing, whether their action was received, what comes next

**Violations:**
- A button that appears to do nothing for 2–3 seconds (no loading state)
- A file upload with no progress indicator
- A form submission that silently redirects without confirmation
- A dashboard that shows stale data without indicating it's stale
- A background sync that fails silently

**Code fix — button with status:**
```tsx
const [status, setStatus] = useState<'idle' | 'loading' | 'success' | 'error'>('idle');

async function handleSubmit() {
  setStatus('loading');
  try {
    await submitForm(data);
    setStatus('success');
    // Auto-reset after 2s
    setTimeout(() => setStatus('idle'), 2000);
  } catch {
    setStatus('error');
  }
}

// In render:
<button 
  onClick={handleSubmit} 
  disabled={status === 'loading'}
  aria-busy={status === 'loading'}
>
  {status === 'loading' ? (
    <><Spinner aria-hidden /> Saving…</>
  ) : status === 'success' ? (
    <><CheckIcon aria-hidden /> Saved</>
  ) : status === 'error' ? (
    'Try again'
  ) : (
    'Save changes'
  )}
</button>
```

**Timing hierarchy (from research):**
- **0.1s (100ms)**: Perceived as instantaneous. Provide feedback within this window for all direct manipulations (button press, click)
- **1s**: User notices delay but stays engaged. Use for async operations that finish quickly
- **10s**: User loses focus. Provide progress + estimated time
- **10s+**: User leaves. Provide background process + notification on complete

### Heuristic 2: Match Between System and Real World

**Principle:** The system should speak the users' language, using words, phrases, and concepts familiar to the user, rather than system-oriented terms.

**What this means:**
- Vocabulary matches the user's mental model, not the developer's database schema
- Metaphors are consistent and familiar (trash can, shopping cart, folder)
- Error messages use plain language, not codes or stack traces
- Dates, numbers, currencies formatted per the user's locale

**Violations:**
- "Invalidate cache" button visible to end users
- "HTTP 403 Forbidden" as an error message
- "Unauthenticated user entity" in error copy
- Showing ISO date strings (2026-07-14) to general audiences
- "Null pointer exception" in a toast notification
- Using developer database table names as UI labels ("user_accounts" instead of "Team members")

**Fix examples:**
```
❌ "Your session token has expired and must be renewed"
✅ "You've been logged out. Sign in again to continue."

❌ "Error: ECONNREFUSED - Could not connect to upstream service"
✅ "Something went wrong on our end. Try refreshing the page."

❌ "Destroy workspace"
✅ "Delete this workspace"

❌ Created: 2026-07-14T09:23:45.000Z
✅ Created: July 14, 2026 at 9:23 AM
```

### Heuristic 3: User Control and Freedom

**Principle:** Users often choose system functions by mistake and will need a clearly marked "emergency exit" to leave the unwanted state without having to go through an extended dialogue.

**What this means:**
- Every action is reversible or at minimum confirmable before execution
- Users can cancel long operations
- "Back" always works and doesn't cause data loss
- Undo is available for destructive operations
- Multi-step processes allow going back without losing progress

**Violations:**
- Delete with no undo or confirmation
- Form that loses all data when back is pressed
- Multi-step wizard with no "Previous" button
- File upload that can't be cancelled mid-progress
- Navigation that doesn't save draft state

**The undo vs. confirm decision:**
```
Prefer UNDO over CONFIRM when:
- The action is reversible (can be undone within 5–30s)
- The action is frequent (constant confirm dialogs = confirm fatigue)
- Example: Archive email, delete a draft, remove a tag

Use CONFIRM over UNDO when:
- The action is truly irreversible (cannot be undone)
- The stakes are high (deleting account, canceling subscription, sending $10k)
- The user is unlikely to trigger it accidentally
- Example: Delete account, submit legal document, transfer funds
```

**Undo toast pattern:**
```tsx
function deleteItem(id: string) {
  // Optimistically remove from UI
  setItems(items.filter(i => i.id !== id));
  
  const toastId = toast({
    message: 'Item deleted',
    action: {
      label: 'Undo',
      onClick: () => {
        // Restore item
        setItems(prev => [...prev, deletedItem]);
        toast.dismiss(toastId);
      }
    },
    duration: 5000, // 5s window to undo
  });
  
  // Only actually delete on server after undo window closes
  setTimeout(() => {
    if (!wasUndone) serverDelete(id);
  }, 5000);
}
```

### Heuristic 4: Consistency and Standards

**Principle:** Users should not have to wonder whether different words, situations, or actions mean the same thing.

**Internal consistency:** Same thing = same word, same visual, same behavior, everywhere in your app.

**External consistency:** Follow platform conventions. Users have pre-existing mental models from OS, browser, and other apps they use.

**Violations:**
- "Cancel" vs "Dismiss" vs "Close" for the same action
- Primary button on the right in one dialog, left in another
- "workspace" and "project" used interchangeably
- Date format changing between pages (Jul 14 vs 07/14 vs 14/7)
- Success states that look different in different parts of the app
- Form validation that triggers on blur in one form, on submit in another

**Consistency audit process:**
1. List all terms used for the same concept (search codebase)
2. List all placements of recurring UI elements (primary button, close icon)
3. List all interaction patterns for similar operations (delete flow, confirm flow)
4. Standardize to one of each

```bash
# Find inconsistent terminology in code
grep -r "workspace\|project" src/ --include="*.tsx" | grep -v node_modules | head -50
# Review and standardize to one term
```

### Heuristic 5: Error Prevention

**Principle:** Even better than good error messages is a careful design which prevents a problem from occurring in the first place.

**Two types:**
1. **Slips** — unconscious errors: accidentally clicking wrong button, hitting enter too early → Fix with confirmations, forgiving input parsing
2. **Mistakes** — conscious wrong decisions: misunderstanding what an action does → Fix with clear affordances, labeling, documentation

**Prevention patterns:**
- Mark required fields BEFORE submit, not after
- Show password strength in real-time as user types
- Disable submit button until form is valid
- Show destructive action consequences in the confirmation: "Delete workspace and all 14 projects and 847 files"
- Parse forgiving inputs: accept "07/14/2026", "July 14", "Jul 14" for date fields
- Prevent double-submit by disabling button immediately on click
- Warn before leaving with unsaved changes

```tsx
// Prevent double-submit
function SubmitButton({ onClick }: { onClick: () => Promise<void> }) {
  const [submitted, setSubmitted] = useState(false);
  
  async function handleClick() {
    if (submitted) return; // prevent double
    setSubmitted(true);
    try {
      await onClick();
    } finally {
      setSubmitted(false); // re-enable on error
    }
  }
  
  return (
    <button onClick={handleClick} disabled={submitted} aria-busy={submitted}>
      {submitted ? 'Submitting…' : 'Submit'}
    </button>
  );
}

// Unsaved changes warning
useEffect(() => {
  if (!hasChanges) return;
  
  const handler = (e: BeforeUnloadEvent) => {
    e.preventDefault();
    e.returnValue = '';
  };
  
  window.addEventListener('beforeunload', handler);
  return () => window.removeEventListener('beforeunload', handler);
}, [hasChanges]);
```

### Heuristic 6: Recognition Rather Than Recall

**Principle:** Minimize the user's memory load by making objects, actions, and options visible. The user should not have to remember information from one part of the interface to use another.

**What this means:**
- Show current location (breadcrumbs, active nav states)
- Show available actions (don't hide options in menus users don't know exist)
- Show previously selected options in filters/preferences
- Show recently used items
- Autocomplete and suggestions reduce recall burden
- Inline help instead of separate documentation

**Violations:**
- Navigation with no active state
- Command palette with no recently used commands
- Filter panel that resets on page refresh
- Form with no default values for common choices
- Search with no recent searches

```tsx
// Recognition over recall: show what's available
// ❌ Empty command palette
<CommandPalette placeholder="Type a command…" />

// ✅ Command palette with recents and suggestions  
<CommandPalette 
  placeholder="Type a command or search…"
  defaultItems={[
    { group: 'Recent', items: recentCommands },
    { group: 'Suggested', items: contextualSuggestions },
  ]}
/>
```

### Heuristic 7: Flexibility and Efficiency of Use

**Principle:** Allow users to tailor frequent actions. Accelerators — unseen by novice users — may often speed up interaction for the expert user.

**What this means:**
- Keyboard shortcuts for power users
- Bulk operations for repetitive tasks
- Customizable views (dense/comfortable layout toggle)
- Saved filters and views
- Default values based on previous choices
- Quick actions in context (hover actions on list items)

**Implementation without overwhelming novice users:**
- Shortcuts visible on hover (tooltip: "Delete (⌫)")
- Bulk selection: checkbox appears on hover or selection
- Advanced options in an expandable "Advanced" section
- Keyboard shortcuts in a discoverable `?` shortcut guide

```tsx
// Efficient bulk operations
function ItemList({ items }: { items: Item[] }) {
  const [selected, setSelected] = useState<Set<string>>(new Set());
  const hasSelected = selected.size > 0;
  
  return (
    <div>
      {hasSelected && (
        <BulkActionBar 
          count={selected.size}
          onDelete={() => bulkDelete([...selected])}
          onArchive={() => bulkArchive([...selected])}
          onClear={() => setSelected(new Set())}
        />
      )}
      {items.map(item => (
        <ItemRow 
          key={item.id}
          item={item}
          selected={selected.has(item.id)}
          onSelect={(checked) => {
            setSelected(prev => {
              const next = new Set(prev);
              checked ? next.add(item.id) : next.delete(item.id);
              return next;
            });
          }}
        />
      ))}
    </div>
  );
}
```

### Heuristic 8: Aesthetic and Minimalist Design

**Principle:** Dialogues should not contain irrelevant or rarely needed information. Every extra unit of information in a dialogue competes with the relevant information.

**What this means:**
- Remove everything that doesn't serve the user's current goal
- Progressive disclosure: show advanced options only when needed
- Don't show empty states for features the user hasn't activated
- Use whitespace as a design element (breathing room = clarity)
- Every UI element earns its place or gets removed

**The "one primary action" rule:**
Every screen should have at most ONE primary action — the thing the user is most likely to want to do right now. Multiple primary actions cancel each other out.

**Violations:**
- Three blue "primary" buttons in a row
- Dashboard showing 20+ metrics at once with equal visual weight
- Form with 30 fields when 5 would suffice
- Navigation with 12 top-level items
- Empty dashboard that shows help text for 10 features the user hasn't used yet

**Progressive disclosure pattern:**
```tsx
// Show complexity progressively
function AdvancedSettings() {
  const [showAdvanced, setShowAdvanced] = useState(false);
  
  return (
    <div>
      {/* Always visible — essential settings */}
      <BasicSettings />
      
      {/* Hidden by default — advanced settings */}
      <button 
        onClick={() => setShowAdvanced(!showAdvanced)}
        aria-expanded={showAdvanced}
        aria-controls="advanced-settings"
      >
        {showAdvanced ? 'Hide advanced settings' : 'Show advanced settings'}
      </button>
      
      <div id="advanced-settings" hidden={!showAdvanced}>
        <AdvancedSettingsContent />
      </div>
    </div>
  );
}
```

### Heuristic 9: Help Users Recognize, Diagnose, and Recover from Errors

**Principle:** Error messages should be expressed in plain language (no codes), precisely indicate the problem, and constructively suggest a solution.

**Error message formula:** [What happened] + [Why] + [What to do]

```
❌ "Error 422"
✅ "Your email address is already in use. Sign in instead, or use a different email."

❌ "Upload failed"
✅ "That file is too large. Maximum size is 10 MB. Try a smaller file or compress this one first."

❌ "Invalid input"
✅ "Enter a phone number in the format (555) 555-5555"

❌ "Something went wrong"
✅ "We couldn't save your changes. Check your internet connection and try again."
```

**Placement rules:**
- Inline: validation errors → immediately below the affected field
- Summary: form-level errors → above the form (or at top of scrolled view)
- Toast: operation errors → non-blocking, 5–8 seconds, dismissible
- Full page: catastrophic errors → centered, with recovery action

### Heuristic 10: Help and Documentation

**Principle:** Even though it is better if the system can be used without documentation, it may be necessary to provide help and documentation.

**Best practices:**
- Contextual help (tooltip, helper text) is better than external documentation
- Empty states are the primary onboarding opportunity — use them
- Help text should be task-oriented: "To export, first select at least one row"
- Search in help is mandatory (no one reads help linearly)
- Provide relevant help at the point of need, not on a separate page

---

## Part 2 — Laws of UX

Psychological principles that govern how users perceive and interact with interfaces.

### Fitts's Law
**"Time to acquire a target is a function of distance and size."**

Closer and larger = easier and faster to click.

**Applications:**
- Primary CTAs: largest, highest-contrast button in their group
- Destructive actions: smaller or further from primary flow
- Right-click context menus: appear at cursor position (distance = 0)
- Floating action buttons: bottom-right (thumb zone on mobile)
- Touch targets: minimum 44×44px even for visually smaller elements

```css
/* Touch target expansion without changing visual size */
.icon-button {
  width: 24px;
  height: 24px;
  padding: 10px; /* Expands target to 44×44px */
}
```

### Hick's Law
**"Time to decide increases logarithmically with the number of choices."**

Double the choices ≠ double the time — it's worse than that.

**Applications:**
- Navigation: maximum 7 top-level items (ideally 5)
- Dropdown menus: maximum 8–10 items before search/grouping
- Settings: group into categories, don't show all at once
- Pricing: 3 plans outperforms 5 plans for conversion
- Onboarding: one task per screen

```
Decision time vs choices (approximate):
2 choices  → baseline
4 choices  → ~50% longer
8 choices  → ~100% longer
16 choices → ~150% longer
```

### Miller's Law
**"Working memory holds ~7 ± 2 items."**

**Applications:**
- Multi-step forms: maximum 5–7 fields per step
- Navigation: 7 items max
- Password rules: maximum 3 explicit requirements (users will forget more)
- Onboarding steps: 5–7 total steps
- Pagination: 7–10 items per page for decision-heavy lists

### Jakob's Law
**"Users spend most of their time on other sites, so they prefer your site to work the same way."**

**Applications:**
- Logo top-left → links to home
- Search in header (right side or center)
- Navigation at top or left
- Cart in header top-right
- Footer with legal links
- Red = error/danger
- Green = success
- Back button behavior matches browser expectations

**When to deviate:** Only when the familiar pattern genuinely doesn't serve the user's task. The burden of proof is high — deviation always has a learning cost.

### Peak-End Rule
**"People judge experiences by their peak (most intense moment) and the end."**

**Applications:**
- Onboarding: make the "first success" moment (peak) as delightful as possible
- Error recovery: the "fixed it" moment should be celebrated
- Checkout: confirmation screen is the last impression — make it excellent
- Cancellation: the "we're sorry to see you go" screen matters
- Loading: a perfect loading experience makes wait time feel shorter

**Implementation:**
```tsx
// ✅ Make the first success feel great
function AccountCreatedSuccess() {
  return (
    <div className="success-screen">
      <LottieAnimation src="confetti.json" autoPlay once />
      <h1>You're all set, {firstName}! 🎉</h1>
      <p>Your workspace is ready. Let's build something.</p>
      <Button onClick={goToDashboard} size="lg">
        Start building →
      </Button>
    </div>
  );
}
```

### Von Restorff Effect (Isolation Effect)
**"An item that is distinctive stands out and is better remembered."**

**Applications:**
- Highlight the recommended pricing plan
- Use a different visual style for the primary CTA
- Callout boxes for critical warnings
- New feature badges

**Warning:** Everything distinctive = nothing distinctive. Use sparingly.

### Zeigarnik Effect
**"People remember uncompleted tasks better than completed ones."**

**Applications:**
- Progress bars in onboarding
- "Profile 60% complete" banners
- Incomplete form saves (draft indicators)
- Streak counters

### Serial Position Effect
**"Users best remember the first and last items in a series."**

**Applications:**
- Put most important navigation items first and last
- Most important form fields at top and bottom of each group
- Critical information in the first and last bullet points

---

## Part 3 — Gestalt Principles

How the visual system organizes information.

### Proximity
**Elements close together are perceived as related.**

```css
/* Good: related elements grouped */
.form-group {
  display: flex;
  flex-direction: column;
  gap: 4px; /* label and input are close — related */
  margin-bottom: 24px; /* groups are far apart — separate */
}
```

**Violation:** Label far from its input; error message not directly below its field.

### Similarity
**Elements that look alike are perceived as belonging together.**

```css
/* All primary actions look the same */
.btn-primary {
  background: var(--primary);
  color: white;
  /* etc */
}

/* All secondary actions look the same */
.btn-secondary {
  background: transparent;
  border: 1px solid var(--border);
  /* etc */
}
```

**Violation:** Buttons that look like links; links that look like buttons (unless intentional).

### Continuity
**The eye follows lines and curves.**

**Applications:** Form fields aligned to a grid; list items with consistent left edge; table columns with clear vertical axis.

### Closure
**People complete incomplete shapes in their minds.**

**Application:** Partially visible carousel cards signal scrollability. Partially visible card below the fold signals more content.

```css
/* Signal scrollability by showing partial next card */
.carousel {
  overflow-x: auto;
  padding-right: 24px; /* shows part of next card */
}

.card {
  min-width: 280px;
  /* The 24px padding shows user there's more to scroll */
}
```

### Figure/Ground
**Objects are perceived as either figure (foreground) or ground (background).**

**Application:** Modal over dimmed background — modal is figure, page is ground. This communicates: "The modal is temporary; the page is still there."

```tsx
// Always dim the page behind a modal
<div className="modal-overlay" aria-hidden="true" onClick={onClose} />
<div role="dialog" aria-modal="true" className="modal">
  {/* content */}
</div>
```

---

## Part 4 — Mental Models

A mental model is the user's internal representation of how something works. UX design's job is to match the UI to the user's mental model — not the developer's implementation model.

### The Gulf of Evaluation
The gap between what the system does and what the user can perceive it doing.

**Reduce with:** Better feedback, better status visibility, better labeling.

### The Gulf of Execution  
The gap between what the user wants to do and what the system allows them to do.

**Reduce with:** Better affordances, shorter paths to goals, forgiving inputs.

### Affordances
What an element looks like it can do:
- A button with shadow → looks pressable
- Underlined text → looks clickable
- A slider → looks draggable
- Strikethrough text → looks crossed out

**False affordances** cause more errors than no affordances: a `<div>` styled like a button that isn't a button will be clicked and nothing will happen.

### Mapping
The relationship between controls and their effects should be natural and logical:
- Volume slider goes left-right; more to the right = louder
- Vertical scroll thumb position matches content position
- Left/right navigation arrows move content left/right

**Violations:** Up arrow that decreases value; scrollbar that moves inversely; controls in unrelated location to their targets.

---

## Part 5 — The Design vs. Engineering Mindset

### What engineers optimize for (and where it goes wrong)
- Completeness: showing all the data
- Correctness: showing exactly what the database returns
- Efficiency: reusing existing components even if wrong semantic
- DRY: one component for all cases, even if it creates bad UX for some

### What designers optimize for (and where it goes wrong)
- Aesthetics: looking good over working well
- Novelty: different patterns even when familiar ones work better
- Perfection: over-engineering edge cases at expense of core flows

### What YUIUX optimizes for
- **Task completion**: can the user do what they came to do?
- **Error prevention**: does the design make errors hard?
- **Learnability**: can a new user figure this out without help?
- **Efficiency**: can a returning user do it faster the 10th time?
- **Error recovery**: when something goes wrong, can the user fix it?
- **Accessibility**: can every user access every feature?

---

## Summary: The Principle Hierarchy

When two principles conflict, this order resolves it:

1. **Accessibility** — no user excluded (legal and ethical obligation)
2. **Error prevention** — users don't lose data or take irreversible actions accidentally
3. **Clarity** — users understand what they're looking at and what to do
4. **Feedback** — users know what happened
5. **Efficiency** — users can accomplish goals quickly
6. **Aesthetics** — the interface is pleasant to use

Aesthetics never trumps any other principle. A beautiful interface that confuses users fails.

---

*Next: `reference/02-typography-readability.md` for type rules that serve clarity.*
