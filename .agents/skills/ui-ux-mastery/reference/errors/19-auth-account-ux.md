---
name: auth-account-ux
description: Sign-up, login, password reset, session expiry, and permission-prompt errors and fixes for web UI.
---

# Auth & Account UX

## Anti-pattern: Login and signup forms are hard to tell apart or hard to switch between
**What's wrong:** A user lands on "Sign up" when they actually have an account (or vice versa) with no easy, obvious way to switch, forcing them to navigate away and search for the other link.
**Why it matters:** This is an extremely common real-world mistake (bookmarked the wrong link, followed an old email) — making the recovery path obvious costs nothing and prevents real frustration.
**Fix:** Always include a visible link on both screens: "Already have an account? Log in" / "New here? Create an account," and detect the common case (existing email entered on the signup form) to proactively suggest logging in instead.

## Anti-pattern: "Forgot password" flow buried, hard to find, or itself broken
**What's wrong:** The forgot-password link is small, low-contrast, or placed somewhere non-obvious; or the reset email never arrives / the reset link expires too quickly with no clear message about why.
**Why it matters:** Password reset is disproportionately used by frustrated, already-annoyed users trying to regain access — friction here compounds an already bad moment and risks losing the user entirely.
**Fix:** Make "Forgot password?" clearly visible directly next to the password field on login, confirm the reset email was sent with a specific next step ("Check your inbox — link expires in 1 hour"), and if the link has expired, say so explicitly with an easy way to request a new one rather than a generic error.

## Anti-pattern: Password requirements not shown until after a failed attempt
**What's wrong:** A signup form rejects a password with "Password too weak" only after submission, without having shown the requirements up front.
**Why it matters:** See `06-forms-inputs.md` — forces a guess-and-check cycle instead of getting it right the first time.
**Fix:** Show requirements as a live checklist before and during typing (see the forms reference for the exact pattern).

## Anti-pattern: Silent session expiry
**What's wrong:** A user's session expires mid-task and the next action either fails silently, redirects to login with no explanation, or (worst case) appears to succeed but doesn't actually save.
**Why it matters:** Users have no way to distinguish "my action failed" from "my action succeeded but I got logged out," and may lose meaningful in-progress work.
**Fix:** Detect session expiry proactively (or on the next authenticated request) and show an explicit message ("Your session expired for security. Sign in again to continue.") that preserves any in-progress work where feasible (e.g. restore a draft after re-login) and returns the user to where they were, not just to a generic dashboard.

## Anti-pattern: Two-factor/verification codes with no resend, no countdown, and unclear expiry
**What's wrong:** A 6-digit code sent via email/SMS with no indication of how long it's valid, and no way to request a new one without restarting the whole flow.
**Why it matters:** Codes routinely arrive late or get missed; a user unable to resend is stuck.
**Fix:** Show a visible countdown/expiry, a "Didn't get a code? Resend" action (rate-limited but not hidden), and clear messaging if a code is entered after expiry ("That code expired. We've sent a new one.").

## Anti-pattern: Logout with no confirmation for a session with unsaved state, but confirmation friction added to logout in general
**What's wrong:** Two opposite failure modes: (a) logout instantly discards an in-progress unsaved form with zero warning, or (b) logout — a fully reversible, low-stakes action — is needlessly gated behind a confirmation dialog "Are you sure you want to log out?"
**Why it matters:** Confirmation should be reserved for costly/irreversible actions (see `24-error-prevention-recovery.md`); logout itself is not one, so a confirmation dialog for it is just friction. But losing unsaved work silently on logout is a real cost that does deserve a warning.
**Fix:** Don't confirm logout itself; do warn (or autosave) if the user has genuinely unsaved changes in progress at the moment logout is triggered.

## Anti-pattern: Permission prompts (location, camera, notifications, microphone) requested immediately on page load
**What's wrong:** A browser permission prompt fires the instant a page loads, before the user has any context for why it's being asked.
**Why it matters:** Users reflexively deny permission requests they don't understand the purpose of — and once denied, most browsers make it hard to re-prompt, potentially blocking a feature the user might have genuinely wanted later with proper context.
**Fix:** Request permissions only at the point of relevant use, with an explanatory UI element shown first ("Enable location to find stores near you" with a button that then triggers the actual browser prompt), never unprompted on load.

## Anti-pattern: Account deletion is either impossible to find or dangerously easy to trigger
**What's wrong:** No visible way to delete an account at all (forcing a support request), or conversely, account deletion reachable in a single unconfirmed click.
**Why it matters:** Both extremes fail users — one traps them, the other risks catastrophic accidental data loss.
**Fix:** Make account deletion discoverable (in settings, not hidden), but require explicit confirmation proportionate to its irreversibility — typically re-entering a password or typing a confirmation phrase, plus clear disclosure of exactly what will be lost and whether there's a grace/undo period before permanent deletion.

## Anti-pattern: Social/SSO login options presented with unclear or inconsistent branding
**What's wrong:** "Continue with Google" buttons that don't follow the provider's official branding guidelines, or SSO options that are unclear about what account/organization they'll authenticate against.
**Why it matters:** Provider branding guidelines exist partly for user trust/recognition — a mismatched button can look like a phishing attempt; ambiguity about which account will be used causes login-to-the-wrong-account mistakes, especially for users with multiple Google/Microsoft accounts.
**Fix:** Follow each provider's official button/branding guidelines exactly, and where technically feasible, show which account is about to be used before finalizing (most OAuth flows already surface this at the provider's consent screen — don't obscure it with custom UI that hides that step).

## Anti-pattern: Password managers actively fought against
**What's wrong:** JavaScript that disables paste into password fields, or custom-styled inputs that break password manager autofill/detection.
**Why it matters:** Password managers are a core part of how security-conscious users manage credentials; blocking paste specifically was historically done to "prevent weak copy-pasted passwords" but is now recognized as actively harmful — it pushes users toward weaker, memorable, reused passwords instead.
**Fix:** Never disable paste on password fields, and use standard `<input type="password" autocomplete="...">` markup that password managers can reliably detect and fill (see `06-forms-inputs.md`).

## Quick Checklist
- [ ] Login/signup screens link to each other clearly; wrong-form entry is proactively detected where feasible
- [ ] "Forgot password" is clearly visible next to the password field; reset emails and expiry are clearly communicated
- [ ] Password requirements are shown before/during typing, not only after a failed submit
- [ ] Session expiry is communicated explicitly, preserves in-progress work where feasible, and returns the user to their prior context
- [ ] Verification codes show expiry/countdown and support resend
- [ ] Logout itself isn't gated behind confirmation, but unsaved in-progress work is protected
- [ ] Permission prompts are requested only at point of relevant use, with context shown first
- [ ] Account deletion is discoverable but requires proportionate confirmation and clear disclosure
- [ ] Social/SSO buttons follow official provider branding; account selection is transparent
- [ ] Password fields never block paste; markup supports password manager autofill
