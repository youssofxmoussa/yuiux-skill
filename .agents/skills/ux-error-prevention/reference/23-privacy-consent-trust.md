---
name: privacy-consent-trust
description: Consent banners, permission prompts, and data-transparency errors and fixes for web UI.
---

# Privacy, Consent & Trust

## Anti-pattern: Cookie/consent banner with no real choice ("dark pattern" consent)
**What's wrong:** A prominent "Accept all" button paired with a "Manage preferences" link that's smaller, lower-contrast, or buried behind extra clicks to actually decline/customize.
**Why it matters:** Beyond the ethical problem, this is now explicitly non-compliant in many jurisdictions (GDPR and similar regimes require consent to be as easy to refuse as to accept) and damages trust the moment a user notices the asymmetry.
**Fix:** Give "Accept all," "Reject non-essential," and "Manage preferences" equal visual prominence, with a genuinely equal number of steps to decline as to accept.

## Anti-pattern: Consent banner reappears every visit despite a prior choice
**What's wrong:** The cookie banner shows again on every page load or every session, even after the user already made a choice.
**Why it matters:** Repetition trains users to blindly click through without reading — the opposite of informed consent — and is simply annoying.
**Fix:** Persist the user's choice (cookie or local storage, scoped appropriately) and don't re-show the banner unless the choice has genuinely expired (per applicable regulation) or the underlying data practices materially change.

## Anti-pattern: Consent banner blocks the entire page
**What's wrong:** A full-screen modal overlay that prevents any interaction with the page until a cookie choice is made, even for content that doesn't require cookies to view (e.g. a public blog post).
**Why it matters:** Forces a decision before the user has any context for why it matters or what the site even is, and blocks legitimate use of content that doesn't need consent at all.
**Fix:** Prefer a non-blocking banner (bottom or top of screen) that lets the user continue browsing while deciding, reserving a full block only where legally required for the specific content/data practice involved.

## Anti-pattern: Permission requests with no stated reason
**What's wrong:** A raw browser permission prompt (location, notifications, camera) fires with no explanation of why the app wants it.
**Why it matters:** See `19-auth-account-ux.md` — unexplained requests get reflexively denied, often permanently.
**Fix:** Show an in-app explanation immediately before triggering the actual browser prompt, stated in terms of user benefit ("Allow location to show restaurants near you").

## Anti-pattern: Data collection/usage not disclosed near the point of collection
**What's wrong:** A form collects sensitive information (health data, financial details, precise location) with no nearby indication of how it will be used or stored.
**Why it matters:** Users make different disclosure decisions depending on stated purpose — collecting without context erodes trust and, for many data categories, is a compliance gap as well.
**Fix:** Add a brief, specific note near any sensitive-data field about its purpose and handling ("Used only to calculate shipping — never shared with third parties"), linking to a full privacy policy for detail rather than requiring it to be read in full at that moment.

## Anti-pattern: Marketing consent bundled with functional consent
**What's wrong:** A single checkbox/agreement covers both "I agree to the terms of service" (required to use the product) and "I agree to receive marketing emails" (should be separable), forcing acceptance of both to proceed.
**Why it matters:** Bundling required functional consent with optional marketing consent removes real choice and is a common regulatory red flag.
**Fix:** Separate required terms/privacy acceptance from optional marketing opt-in into distinct, independently controllable checkboxes — never pre-check the marketing one.

## Anti-pattern: No visible way to see, export, or delete one's own data
**What's wrong:** A product stores meaningful personal data but offers no in-app way to view what's stored, export it, or delete it — requiring a support ticket for any of these.
**Why it matters:** Beyond the compliance angle in many jurisdictions, being unable to see or control your own data is a fundamental trust gap for any product handling personal information.
**Fix:** Provide self-service access to view, export (a portable format like CSV/JSON), and delete personal data directly in account settings wherever feasible, with account deletion following the confirmation pattern described in `19-auth-account-ux.md`.

## Anti-pattern: Third-party trackers/scripts loaded before any consent decision
**What's wrong:** Analytics, ad, or social-media tracking scripts execute immediately on page load, before the user has responded to the consent banner asking about exactly that.
**Why it matters:** Makes the consent banner functionally meaningless — the tracking already happened regardless of the answer.
**Fix:** Gate non-essential third-party script loading behind the actual consent decision — load them only after (and only if) the user accepts, not preemptively.

## Anti-pattern: Vague or absent explanation of AI-driven features
**What's wrong:** A feature that uses AI/ML to make decisions, recommendations, or generate content with no indication to the user that AI is involved, or how confident/reliable it is.
**Why it matters:** Users reasonably calibrate their trust differently for AI-generated versus human-verified content or decisions; withholding that context can lead to over-trusting an imperfect system, especially for consequential outputs (medical, legal, financial suggestions).
**Fix:** Clearly label AI-generated or AI-assisted content/decisions, and where the output can be materially wrong, include an appropriate caveat and a path to human review or correction.

## Quick Checklist
- [ ] Consent choices (accept/reject/customize) have equal visual prominence and equal step-count
- [ ] A prior consent choice persists and isn't re-prompted unnecessarily
- [ ] Consent banners don't block unrelated page content unless legally required
- [ ] Permission prompts are preceded by an in-app explanation of purpose
- [ ] Sensitive-data fields carry a nearby note on purpose/handling
- [ ] Required (terms/privacy) and optional (marketing) consent are separate, independently controlled, and marketing is never pre-checked
- [ ] Users can self-service view/export/delete their own personal data
- [ ] Non-essential third-party trackers load only after actual consent, not before
- [ ] AI-generated/assisted content or decisions are clearly labeled, with caveats where output can be materially wrong
