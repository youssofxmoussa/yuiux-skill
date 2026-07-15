---
name: forms-inputs
description: Labels, validation timing, error recovery, multi-step forms, autofill, and autosave errors and fixes for web UI.
---

# Forms & Inputs

Forms are the highest-friction, highest-abandonment surface in most software. Every error here directly costs conversions/task completion.

## Anti-pattern: Placeholder used as the only label
**What's wrong:** `<input placeholder="Email">` with no `<label>` at all.
**Why it matters:** The placeholder disappears the moment the user starts typing — if they get interrupted or need to re-verify what field they're in, the context is gone. It also fails screen readers unless additionally exposed via `aria-label`, and fails low-vision users who need larger, persistent text.
**Fix:** Always pair a real, persistently visible label with every input:

```html
<label for="email">Email</label>
<input id="email" type="email" placeholder="you@example.com" />
```
Placeholder text should show a *format example*, not substitute for the label.

## Anti-pattern: Floating labels that overlap input text at small sizes or with long values
**What's wrong:** A label that animates to overlap the input's top border when the field is empty, but collides with entered text if the value is long or the input is narrow.
**Why it matters:** This is a purely aesthetic pattern that frequently breaks on real (not demo) data and adds implementation complexity for a marginal visual gain.
**Fix:** If using floating labels, test with realistic long values at the smallest supported viewport; prefer a simple always-visible label above the field when in doubt — it's more robust and equally clean when spaced well.

## Anti-pattern: Validating on every keystroke before the user has finished typing
**What's wrong:** An email field shows "Invalid email" the instant the user types the first character.
**Why it matters:** Punishing the user for an incomplete answer to a question they haven't finished answering feels hostile and erodes trust in every subsequent message the form shows.
**Fix:** Validate on blur (when the user leaves the field) for format-checkable fields, and on submit for everything else. Exception: *positive* real-time feedback (a green check once a valid format is reached) is fine and encouraged — the rule is against *negative* feedback before the user is done.

```js
function handleBlur(e) {
  if (!isValidEmail(e.target.value)) setError("Enter a valid email address");
}
```

## Anti-pattern: Validating only on submit, after the user has filled out the whole form
**What's wrong:** A 10-field form that only reveals a typo in field 2 after the user submits and scrolls back to the top.
**Why it matters:** This forces the user to re-discover an error they could have fixed immediately, and if multiple fields have issues they're now solving them one submit-cycle at a time.
**Fix:** Combine per-field blur validation (catches issues as the user moves through the form) with a final submit-time validation pass (catches anything missed, e.g. cross-field validation). On submit failure, scroll to and focus the first invalid field, and list every error, not just the first one found.

## Anti-pattern: Generic, blaming, or code-based error messages
**What's wrong:** "Invalid input." "Error 400." "Something went wrong." "You entered it wrong."
**Why it matters:** None of these tell the user what to actually do next; the burden of diagnosis falls entirely on them.
**Fix:** Name the field and the specific fix:

```
Bad:  "Invalid input"
Good: "Password must be at least 8 characters and include a number"

Bad:  "Error"
Good: "We couldn't find an account with that email. Check the spelling or create a new account."
```

## Anti-pattern: Clearing the form (or just the failed field) on validation/submit failure
**What's wrong:** A failed submit wipes some or all of what the user typed, forcing them to re-enter everything.
**Why it matters:** This is one of the most rage-inducing patterns in web forms — it directly punishes the user for the exact moment they were trying to correct.
**Fix:** Never clear user input on any failure path — client validation, server validation, or network error. Preserve every field's value and only surface the error alongside it.

## Anti-pattern: Password fields with no visibility toggle
**What's wrong:** A password `<input type="password">` with no way to reveal what was actually typed.
**Why it matters:** Users frequently mistype passwords, especially complex ones, and have no way to verify before submitting — leading to failed logins/signups they can't self-diagnose.
**Fix:** Add a show/hide toggle (eye icon) on every password field:

```html
<div class="password-field">
  <input id="pwd" type="password" />
  <button type="button" aria-label="Show password" onclick="toggleVisibility()">👁</button>
</div>
```

## Anti-pattern: Overly restrictive or opaque password requirements
**What's wrong:** Requiring a password to meet 6 different rules (uppercase, lowercase, digit, symbol, no repeated chars, not on a banned list) without showing which rules are met as the user types, or rejecting long/complex passwords with an arbitrary max length.
**Why it matters:** Users can't self-correct against rules they can't see, and short arbitrary max-lengths actively push users toward weaker, shorter passwords.
**Fix:** Show a live checklist of requirements that updates as the user types (checkmarks appearing per rule met), keep requirements to the essentials (length is the single strongest driver of password strength — 8+ characters, ideally encourage even longer), and never impose an unreasonably low maximum length.

## Anti-pattern: No autofill support
**What's wrong:** Missing or incorrect `autocomplete`/`name`/`type` attributes, so browser/password-manager autofill can't populate the form.
**Why it matters:** Autofill is one of the biggest friction-reducers on the modern web; breaking it forces every returning user to hand-type their email/address/card details every time.
**Fix:** Use correct semantic input types and standard `autocomplete` tokens:

```html
<input type="email" name="email" autocomplete="email" />
<input type="tel" name="phone" autocomplete="tel" />
<input name="cc-number" autocomplete="cc-number" inputmode="numeric" />
<input type="password" name="new-password" autocomplete="new-password" />
```

## Anti-pattern: Wrong `inputmode`/`type` for the data being collected
**What's wrong:** A numeric field (phone, zip code, card number, OTP) rendered as `type="text"`, forcing mobile users into the full alphanumeric keyboard.
**Why it matters:** This is a small thing that costs real time on every single mobile form fill, multiplied across every user.
**Fix:** Match `type`/`inputmode` to the data:

```html
<input type="tel" inputmode="numeric" pattern="[0-9]*" />  <!-- phone/OTP -->
<input type="email" inputmode="email" />
<input type="number" inputmode="decimal" />                <!-- price -->
<input type="search" inputmode="search" />
```

## Anti-pattern: Losing progress on multi-step forms/wizards
**What's wrong:** A user fills 3 of 5 wizard steps, accidentally refreshes or navigates back, and loses everything.
**Why it matters:** Long forms already have high abandonment risk; losing progress to an accidental navigation is a preventable, avoidable failure that guarantees the user has to start over (and may just give up instead).
**Fix:** Persist wizard state to `localStorage`/`sessionStorage` (or the backend if authenticated) on every step change, and restore it automatically if the user returns.

```js
useEffect(() => {
  localStorage.setItem("signup-wizard", JSON.stringify(formState));
}, [formState]);
```

## Anti-pattern: No visible progress indicator on multi-step flows
**What's wrong:** A wizard with no indication of how many steps remain.
**Why it matters:** Users tolerate a known amount of remaining effort far better than an unknown amount (uncertainty itself is a cost) — per the Zeigarnik effect, a visible "step 2 of 4" also motivates completion.
**Fix:** Always show current step and total steps, and where feasible let users jump back to a previous step without losing later-step data.

## Anti-pattern: Submit button stays clickable during submission
**What's wrong:** No disabled/loading state on the submit button, so a slow network response invites the user to click again (or double-click out of impatience), firing duplicate requests.
**Why it matters:** Duplicate submissions can create duplicate orders, duplicate accounts, or duplicate records — a real functional bug caused purely by a UX omission.
**Fix:** Disable the submit control and show a loading state the instant submission starts, re-enable only on error:

```jsx
<button disabled={isSubmitting} type="submit">
  {isSubmitting ? <Spinner /> : "Create account"}
</button>
```

## Anti-pattern: Required-field indicators inconsistent or missing
**What's wrong:** Some required fields marked with `*`, others not, with no consistent rule; or all fields look identical with no indication of which are required until submit fails.
**Why it matters:** Users can't plan how much of the form they actually need to complete, and get surprised by a submit-time error for a field they assumed was optional.
**Fix:** Mark every required field consistently (asterisk + a "* required" legend, or — often clearer — mark the rarer *optional* fields explicitly with "(optional)" if most fields are required).

## Anti-pattern: Destructive form actions (Reset/Clear all) treated like normal buttons
**What's wrong:** A "Clear form" or "Reset" button styled identically to "Save," sitting right next to it.
**Why it matters:** A single misclick wipes everything the user just entered, with no recovery.
**Fix:** Avoid a hard reset/clear button in most forms entirely; if one is genuinely needed, style it as clearly secondary/lower-emphasis, separate it spatially from primary actions, and require confirmation before it takes effect.

## Anti-pattern: Long forms with no ability to save and resume, or no autosave
**What's wrong:** A long application/profile form that must be completed in one sitting or is lost.
**Why it matters:** Real users get interrupted; a form that can't survive an interruption puts an artificial time-pressure on a task that shouldn't have one.
**Fix:** For forms of meaningful length (beyond a simple login/signup), autosave periodically (debounced, e.g. every few seconds of inactivity or on field blur) and communicate save status ("Saved" / "Saving…") so the user trusts it's safe to leave.

## Quick Checklist
- [ ] Every input has a persistent, real `<label>` — placeholders never substitute for labels
- [ ] Validation happens on blur/submit, never punishing incomplete typing
- [ ] Errors name the field and the specific fix, never generic/blaming copy
- [ ] Input values are never cleared on any failure path
- [ ] Password fields have a show/hide toggle; requirements are visible and reasonable (length-first)
- [ ] Correct `type`/`autocomplete`/`inputmode` set on every field for autofill and correct mobile keyboards
- [ ] Multi-step forms persist progress and show a step indicator
- [ ] Submit buttons disable/show loading state immediately on submission to prevent duplicates
- [ ] Required vs. optional fields are marked consistently
- [ ] Destructive form actions (reset/clear) are de-emphasized and confirmed
- [ ] Long forms autosave and communicate save status
