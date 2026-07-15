---
name: feedback-loading-empty-error-states
description: Skeletons, spinners, empty states, error boundaries, and optimistic UI errors and fixes for web UI.
---

# Feedback: Loading, Empty & Error States

These three states are often built last and worst, yet a real user encounters them constantly — first load, no results, and something going wrong are not edge cases, they're the everyday texture of using software.

## Anti-pattern: Blank white screen while data loads
**What's wrong:** A component renders nothing (or an empty container) until an API response returns, with zero visual indication anything is happening.
**Why it matters:** A blank screen is indistinguishable from a broken page — the user can't tell if the app is working, frozen, or crashed, and the Doherty Threshold research shows perceived responsiveness collapses without feedback within ~400ms.
**Fix:** Show a loading indicator (skeleton preferred over spinner for content-shaped areas) the instant a fetch begins, matched to the shape of the eventual content:

```jsx
{isLoading ? <CardSkeleton count={3} /> : <CardList items={data} />}
```

## Anti-pattern: Spinner used for everything, including content-shaped areas
**What's wrong:** A generic centered spinner shown in place of a list, table, or card grid that has a predictable shape.
**Why it matters:** A spinner conveys "something is happening" but nothing about *what's coming* — this causes more perceived wait time than a skeleton that previews the eventual layout, and causes a jarring layout jump once real content pops in.
**Fix:** Use skeleton placeholders shaped like the real content (same card dimensions, same number of lines) for structured content; reserve spinners for small, size-unknown, or brief operations (e.g. inside a button, a small inline action).

```jsx
<div className="skeleton-card">
  <div className="skeleton-line" style={{width: "60%"}} />
  <div className="skeleton-line" style={{width: "90%"}} />
  <div className="skeleton-line" style={{width: "40%"}} />
</div>
```

## Anti-pattern: Loading state with no timeout/escape hatch
**What's wrong:** A spinner that spins forever if the request hangs or the server never responds, with no way for the user to retry or cancel.
**Why it matters:** Users have no way to distinguish "still loading, be patient" from "this is broken and will never finish" — leaving them stuck.
**Fix:** Set a reasonable request timeout; if exceeded, transition to an error state with a retry action rather than spinning indefinitely.

## Anti-pattern: Generic empty state ("No data")
**What's wrong:** An empty list/table renders just the words "No data" or nothing at all, with no explanation or next step.
**Why it matters:** An empty state is often a first-run experience — the user doesn't yet know whether "empty" means "you haven't created anything yet," "your filters excluded everything," or "something's broken."
**Fix:** Every empty state explains *why* it's empty and offers a specific action:

```
Empty because nothing created yet:
  "No projects yet — create your first one to get started."
  [ + New project ]

Empty because of active filters:
  "No results match your filters."
  [ Clear filters ]

Empty because of a search query:
  "No results for “abc123”. Try a different search term."
```

## Anti-pattern: Generic, technical, or code-based error messages
**What's wrong:** "Error 500." "Something went wrong." A raw stack trace or exception message shown to the end user.
**Why it matters:** None of this tells the user whether the problem is on their end (bad input, connectivity) or the system's, nor what to do about it — and exposing internals can also leak information.
**Fix:** Translate every user-facing error into plain language plus a next step:

```
Bad:  "Error: NetworkError when attempting to fetch resource."
Good: "Couldn't connect. Check your internet connection and try again."
       [ Retry ]

Bad:  "500 Internal Server Error"
Good: "Something went wrong on our end. We've been notified — try again in a moment."
       [ Retry ]  [ Contact support ]
```

## Anti-pattern: No error boundary — a thrown exception blanks the whole page
**What's wrong:** An unhandled render error in one component crashes the entire React tree, leaving a blank white screen with only a console error visible to developers.
**Why it matters:** A single bug in one widget shouldn't be able to take down the entire page for the user, and a blank screen with zero UI gives them no path to recover short of a manual full reload.
**Fix:** Wrap risky/independent sections of the UI in error boundaries with a friendly, scoped fallback:

```jsx
<ErrorBoundary fallback={<WidgetErrorFallback onRetry={refetch} />}>
  <AnalyticsWidget />
</ErrorBoundary>
```
The rest of the page (nav, other widgets) should keep working even if one section fails.

## Anti-pattern: Errors shown as an intrusive modal for low-severity issues
**What's wrong:** A blocking modal dialog interrupts the user for a minor, recoverable issue (e.g. "failed to load a secondary widget") when a quieter inline message or toast would do.
**Why it matters:** Modals demand a decision before the user can do anything else — reserving that interruption for genuinely blocking issues (payment failure, data-loss risk) keeps it meaningful; overusing it for minor issues trains users to reflexively dismiss every dialog without reading it.
**Fix:** Match error presentation to severity:
- **Inline** (next to the field/widget that failed): field-level or widget-level errors that don't block the rest of the page.
- **Toast/banner**: transient, non-blocking issues (e.g. "Couldn't refresh notifications — retrying").
- **Modal/full-page**: only for errors that must be resolved before the user can proceed at all (e.g. payment failed at checkout, session expired mid-critical-action).

## Anti-pattern: Optimistic UI that lies about the true state on failure
**What's wrong:** A "like" button fills in instantly (optimistic update), the request fails silently, and the UI never reverts — the user believes the action succeeded when it didn't.
**Why it matters:** Optimistic UI is a legitimate, valuable pattern for perceived speed, but only if failure is handled — otherwise it actively misinforms the user about the real state of their data.
**Fix:** Always pair optimistic updates with a rollback + visible error on failure:

```js
async function toggleLike() {
  setLiked(true); // optimistic
  try {
    await api.like(postId);
  } catch (e) {
    setLiked(false); // rollback
    showToast("Couldn't like this post — try again");
  }
}
```

## Anti-pattern: Partial failure in a batch operation reported as one opaque result
**What's wrong:** A bulk action ("delete 12 items") that partially fails reports either a blanket success or a blanket failure with no detail on which items actually succeeded.
**Why it matters:** The user can't tell what state their data is actually in, and can't take corrective action on just the failed subset.
**Fix:** Report batch results with per-item detail when failures are partial: "10 of 12 deleted. 2 failed — [view details / retry]."

## Anti-pattern: Toast notifications that disappear before they can be read, or stack indefinitely
**What's wrong:** A toast auto-dismisses after 1.5 seconds (too fast for many users to read), or multiple toasts stack up unbounded, covering the screen.
**Why it matters:** A message the user can't finish reading before it vanishes has failed at its only job; an unbounded stack of toasts becomes its own UI problem.
**Fix:** Give toasts a minimum readable duration (~4–6 seconds, longer for longer messages, and pause the timer on hover/focus), cap the number of simultaneously visible toasts (collapse or queue additional ones), and always give the user a manual dismiss control.

## Anti-pattern: Success state with no confirmation at all
**What's wrong:** An action completes successfully but nothing visibly changes to confirm it — no toast, no state update, no visual acknowledgment.
**Why it matters:** Per the visibility-of-system-status heuristic, silence after a successful action is indistinguishable from silence after a failed one; users are left to guess or manually re-check.
**Fix:** Every meaningful action gets a confirmation signal proportionate to its importance — a subtle inline checkmark for a minor autosave, a toast for a standalone action, a redirect/confirmation screen for a major flow (checkout, account creation).

## Quick Checklist
- [ ] Every async view shows a loading indicator immediately, matched to the eventual content's shape
- [ ] Skeletons used for structured content; spinners reserved for small/brief/unknown-size operations
- [ ] Requests have a timeout with a retry/error fallback, never spin forever
- [ ] Every empty state explains why and offers a specific next action
- [ ] Every error message is plain-language, specific, and actionable — never a raw code/stack trace
- [ ] Error boundaries scope failures to the affected section, not the whole page
- [ ] Error presentation (inline/toast/modal) matches actual severity
- [ ] Optimistic UI always rolls back and surfaces a visible error on failure
- [ ] Batch/bulk operations report per-item results on partial failure
- [ ] Toasts stay readable long enough, are dismissible, and don't stack unbounded
- [ ] Every meaningful successful action gets a visible confirmation signal
