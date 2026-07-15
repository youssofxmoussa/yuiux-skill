# 28 — User Testing & Validation

> You can't design for users without talking to them. Every assumption about user behavior is a bet. Testing makes the odds explicit.

---

## Testing Methods by Stage

| Stage | Method | Time | Cost | Fidelity Needed |
|-------|--------|------|------|----------------|
| Discovery | User interviews | 1-2 hours/participant | Medium | Low — sketches fine |
| Concept | Card sorting | 30 min | Low | Low |
| Design | Cognitive walkthrough | 1 hour | Low | Medium — wireframes |
| Prototype | Usability testing | 45-60 min | Medium | Medium-High |
| Live | Analytics + heatmaps | Ongoing | Low | Production |
| Optimization | A/B testing | 2-4 weeks | Low | Production |

---

## Usability Testing

### The 5-User Rule
5 users catch 85% of usability problems (Nielsen). Run 5 tests, fix issues, run 5 more.

### Test Protocol

```
Before the test:
  1. Define tasks (not questions — observable tasks)
  2. Prepare the environment (prototype, recorder, note-taker)
  3. Brief the participant: "We're testing the design, not you"

During the test:
  1. Ask participant to think aloud
  2. Don't help them — observe, don't guide
  3. Note: hesitations, errors, confusions, workarounds
  4. After task: "On a scale of 1-5, how easy was this?"

After the test:
  1. Debrief: "What was most confusing? What worked well?"
  2. Note-taker reviews recordings
  3. Synthesis session with team
```

### Task Writing

```
❌ Leading/hypothetical tasks:
"Imagine you want to buy a shirt. Would you click on the Shop button?"
"Do you think the checkout is easy?"

✅ Observable, realistic tasks:
"You want to buy a blue shirt in size M. Please do that."
"You received an order and want to return it. Please start that process."
```

---

## Heuristic Evaluation

Rate each of Nielsen's 10 heuristics 0–4:
- 0: Not a problem
- 1: Cosmetic issue (fix if time permits)
- 2: Minor (low priority)
- 3: Major (important to fix)
- 4: Catastrophic (must fix before launch)

```
1. Visibility of system status
   □ Does the UI always communicate what's happening?
   □ Does the user know what's loading, processing, saved?

2. Match between system and real world
   □ Do labels use user's language (not technical jargon)?
   □ Do icons represent familiar metaphors?

3. User control and freedom
   □ Can users undo/redo actions?
   □ Is there a clear exit from any state?

4. Consistency and standards
   □ Do similar things look and behave the same?
   □ Do we follow platform conventions?

5. Error prevention
   □ Are potential errors caught before they happen?
   □ Are destructive actions confirmed?

6. Recognition over recall
   □ Are options visible (not hidden in memory)?
   □ Are instructions available when needed?

7. Flexibility and efficiency
   □ Do power users have shortcuts?
   □ Can users customize common workflows?

8. Aesthetic and minimalist design
   □ Is every element necessary?
   □ Does anything compete with primary tasks?

9. Help users recover from errors
   □ Are errors in plain language?
   □ Do error messages suggest solutions?

10. Help and documentation
    □ Is help available in context?
    □ Is documentation searchable and task-based?
```

---

## A/B Testing

### When to A/B Test

```
✅ Valid:
  - Two equally-viable design directions with unclear winner
  - High-traffic pages where improvement has significant impact
  - Single variable change (button color, CTA copy, headline)

❌ Invalid:
  - Low traffic (< 1000 conversions/variant) — results not significant
  - Testing multiple variables simultaneously — can't isolate cause
  - Testing when you know the answer — use test to confirm, not explore
```

### Statistical Significance

```
Minimum sample size per variant:
  - 95% confidence level
  - 20% minimum detectable effect
  - Baseline conversion rate: 5%
  → ~1500 visitors per variant

Never call a test early.
Run until you hit the predetermined sample size or 2-4 weeks (whichever is longer).
```

### A/B Test Setup

```tsx
// Simple A/B test with cookie-based assignment
function useABTest(testName: string, variants: string[]) {
  const [variant, setVariant] = useState<string | null>(null);
  
  useEffect(() => {
    // Check for existing assignment
    const stored = getCookie(`ab_${testName}`);
    if (stored && variants.includes(stored)) {
      setVariant(stored);
      return;
    }
    
    // Assign randomly
    const assigned = variants[Math.floor(Math.random() * variants.length)];
    setCookie(`ab_${testName}`, assigned, { maxAge: 30 * 24 * 3600 });
    setVariant(assigned);
    
    // Track assignment
    analytics.track('AB Test Assigned', { test: testName, variant: assigned });
  }, [testName, variants]);
  
  function trackConversion(event: string) {
    analytics.track(event, { test: testName, variant });
  }
  
  return { variant, trackConversion };
}

// Usage:
function CheckoutPage() {
  const { variant, trackConversion } = useABTest('checkout-cta', ['control', 'v2']);
  
  return (
    <button
      onClick={() => {
        trackConversion('Checkout Completed');
        completePurchase();
      }}
    >
      {variant === 'v2' ? 'Complete purchase' : 'Place order'}
    </button>
  );
}
```

---

## Analytics Events

### What to Track

```
User lifecycle:
  - sign_up_started, sign_up_completed, sign_up_error
  - sign_in, sign_out
  - onboarding_step_viewed, onboarding_step_completed, onboarding_skipped

Feature usage:
  - feature_viewed, feature_used, feature_abandoned
  - [feature]_created, [feature]_updated, [feature]_deleted

Navigation:
  - page_viewed (automatic via router)
  - modal_opened, modal_closed, modal_dismissed
  - tab_changed, filter_applied, sort_changed

Errors:
  - error_shown (with type, message, context)
  - api_error (with endpoint, status, duration)

Performance:
  - page_load_time, ttfb, lcp, cls, inp
```

### Event Schema

```typescript
interface AnalyticsEvent {
  name: string;
  properties: {
    // Always include:
    timestamp: string;   // ISO 8601
    session_id: string;
    user_id?: string;    // null if anonymous
    page_path: string;
    
    // Event-specific:
    [key: string]: string | number | boolean | null;
  };
}

// Example:
analytics.track('Project Created', {
  project_id: project.id,
  project_type: project.type,
  template_used: project.fromTemplate ?? null,
  team_size: project.memberCount,
  time_to_create_ms: Date.now() - startTime,
});
```

---

## Heatmaps & Session Recordings

Use: Hotjar, PostHog, FullStory, Microsoft Clarity

```
What to look for in heatmaps:
  - Click dead zones (users clicking non-interactive elements — add interaction)
  - Rage clicks (repeated clicks = frustration)
  - Scroll depth (what percent see below the fold)
  - Form abandonment (which field causes drop-off)

What to look for in session recordings:
  - Where users get stuck (long pauses)
  - Where users go back unexpectedly
  - What users try before succeeding
  - What users do when something fails
```

---

## Accessibility Testing

```
Automated tools catch 30-40% of accessibility issues.
Manual testing required for the rest.

Automated:
  - axe DevTools (browser extension)
  - Lighthouse Accessibility audit
  - Pa11y (CI/CD)

Manual:
  - Keyboard-only navigation test
  - Screen reader test (NVDA + Chrome, VoiceOver + Safari)
  - High contrast mode test
  - 200% zoom test
  - Color contrast checker
```

---

## Feedback Collection in Product

```tsx
// In-product feedback widget
function FeedbackWidget({ context }: { context: string }) {
  const [rating, setRating] = useState<number | null>(null);
  const [feedback, setFeedback] = useState('');
  const [submitted, setSubmitted] = useState(false);
  
  async function handleSubmit() {
    await submitFeedback({ context, rating, feedback });
    setSubmitted(true);
  }
  
  if (submitted) {
    return (
      <div role="status">
        <CheckCircleIcon aria-hidden />
        Thanks for your feedback!
      </div>
    );
  }
  
  return (
    <div className="feedback-widget">
      <p>Was this page helpful?</p>
      <div role="group" aria-label="Helpfulness rating">
        {[1, 2, 3, 4, 5].map(n => (
          <button
            key={n}
            type="button"
            onClick={() => setRating(n)}
            aria-pressed={rating === n}
            aria-label={`${n} star${n !== 1 ? 's' : ''}`}
            className={`star-btn ${rating !== null && n <= rating ? 'active' : ''}`}
          >
            ★
          </button>
        ))}
      </div>
      {rating && rating <= 3 && (
        <>
          <label htmlFor="feedback-text">What could be improved?</label>
          <textarea
            id="feedback-text"
            value={feedback}
            onChange={e => setFeedback(e.target.value)}
            rows={3}
            placeholder="Tell us what wasn't working…"
          />
        </>
      )}
      {rating && (
        <button type="button" onClick={handleSubmit} className="btn btn-sm btn-primary">
          Submit feedback
        </button>
      )}
    </div>
  );
}
```

---

## Testing Checklist

- [ ] Usability tests run with 5+ representative users before launch
- [ ] Tasks defined as observable actions, not leading questions
- [ ] Heuristic evaluation against all 10 Nielsen heuristics
- [ ] A/B tests have statistical significance plan before starting
- [ ] A/B tests run minimum 2 weeks and predetermined sample size
- [ ] Analytics events tracking all key user actions
- [ ] Error events tracked with context for debugging
- [ ] Heatmaps configured on high-traffic pages
- [ ] Accessibility: automated scan + keyboard test + screen reader
- [ ] User feedback widget on complex flows (forms, checkouts)
- [ ] Performance metrics (LCP, CLS, INP) tracked in production
