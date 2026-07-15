---
name: onboarding-first-run
description: Empty-state onboarding, progressive disclosure, tooltips, and product tour errors and fixes for web UI.
---

# Onboarding & First-Run Experience

## Anti-pattern: Forced multi-screen tour before the user can do anything
**What's wrong:** A new user must click through 5–7 modal slides explaining features before reaching the actual product.
**Why it matters:** Users overwhelmingly prefer learning by doing over being lectured to before they've even seen what they're learning about; a forced tour delays the moment of real value and is usually skipped/ignored anyway, at the cost of frustration for having to skip it.
**Fix:** Get the user to their first real, working interaction as fast as possible. Prefer contextual, in-place guidance (tooltips that appear the first time a relevant feature is visible) over a blocking upfront tour. If a tour is truly warranted (a genuinely non-obvious product), keep it to 2–3 screens maximum and always make it skippable from screen one.

## Anti-pattern: Empty states with no onboarding value
**What's wrong:** A brand-new user's dashboard/list shows a bare "No items" message identical to what a returning user sees after deleting everything — no guidance on what to do first.
**Why it matters:** The empty state is often the very first meaningful screen a new user interacts with — it's prime real estate for guiding the first action, not just reporting an absence of data.
**Fix:** Design a distinct first-run empty state with a clear call-to-action and, where valuable, a short explanation of what will appear here once populated:

```
"Your projects will show up here.
Create your first one to get started."
[ + New project ]
```

## Anti-pattern: Feature tooltips shown to everyone forever, including experienced users
**What's wrong:** A "New!" badge or coaching tooltip that never goes away once seen, or reappears on every page load regardless of whether the user already interacted with it.
**Why it matters:** Persistent tooltips for things a user already knows about become visual noise and, over time, something users learn to tune out entirely — reducing the effectiveness of the pattern even for genuinely new features later.
**Fix:** Dismiss coaching UI permanently (persisted per-user, not just per-session) once seen or once the related action has been taken at least once.

## Anti-pattern: Progressive disclosure entirely absent — full complexity shown to first-time users
**What's wrong:** A first-time user is shown every advanced option, setting, and edge-case control at once, identical to what a power user sees after months of use.
**Why it matters:** Per Hick's Law, more visible choices increase decision time and cognitive load — overwhelming a first-time user with full complexity before they've built a mental model of the basics increases early abandonment.
**Fix:** Show the essential path by default; put advanced/rare options behind an explicit "Advanced" toggle, secondary menu, or reveal them only once a relevant precondition is met (e.g. show bulk-edit tools only once there are enough items to bulk-edit).

## Anti-pattern: No sample/seed content for new accounts
**What's wrong:** A brand-new account starts completely blank, so the first thing the user experiences is an empty product they have to populate before they can evaluate whether it's useful.
**Why it matters:** Users struggle to imagine a fully-featured product from a blank slate; seeing example data helps them evaluate real capabilities immediately (a chart with example numbers is far more legible than the concept "there will be a chart here").
**Fix:** Where appropriate for the product, seed new accounts with clearly-labeled example/demo content ("Example project — feel free to delete") that demonstrates the product's actual value immediately, rather than requiring the user to build up state from nothing before seeing anything.

## Anti-pattern: Signup asks for more information than needed to start
**What's wrong:** A lengthy signup form collecting company size, industry, phone number, and job title before the user has even seen the product.
**Why it matters:** Every additional required field before first value is a fresh chance for abandonment; users haven't yet built trust or established that the product is worth the effort of a long form.
**Fix:** Minimize the signup form to only what's strictly required to create an account (typically email + password, or a social/SSO login) and collect any additional profiling information progressively later, ideally tied to a specific benefit ("Add your company name to customize your invoices").

## Anti-pattern: No confirmation that account creation actually succeeded
**What's wrong:** After signup, the user lands directly in the app with no acknowledgment that an account was created, no welcome message, no indication of what just happened.
**Why it matters:** Per the Peak-End rule, the very start of a relationship with a product is a moment users remember — an unceremonious, unclear transition undersells a moment that should build confidence.
**Fix:** Confirm success explicitly (a brief welcome screen, toast, or clearly distinct first-run dashboard state) before dropping the user into the general product experience.

## Anti-pattern: Required email verification blocks all product access with no explanation
**What's wrong:** A user signs up, is immediately locked out of everything until they click a verification link, with no indication of why or what to do if the email doesn't arrive.
**Why it matters:** This is a hard stop at the exact moment the user's motivation is highest and most fragile; email delivery isn't instant or fully reliable, and no fallback path (resend, spam-folder reminder, alternate verification) leaves users stuck.
**Fix:** Where verification is required, let the user explore a reasonable amount of the product unauthenticated-but-registered where feasible, and always show clear status ("We sent a link to you@example.com — check your spam folder, or resend") with a visible resend action and a visible countdown/cooldown if rate-limited.

## Anti-pattern: No empty-state guidance differentiating "nothing yet" from "nothing matches your filter"
**What's wrong:** Identical empty-state copy shown whether the user has genuinely created nothing, or has created plenty but their current filters/search exclude all of it.
**Why it matters:** A returning user who filtered too aggressively and sees "create your first item" is misled into thinking their existing data vanished.
**Fix:** Detect and differentiate the cause (see the empty-state pattern in `09-feedback-loading-empty-error-states.md`) — first-run vs. filtered vs. search-no-results each need distinct copy and actions.

## Quick Checklist
- [ ] No forced multi-screen tour blocks the first real interaction; guidance is contextual and skippable
- [ ] First-run empty states are distinct, explanatory, and action-oriented — not identical to a generic "no data" state
- [ ] Coaching tooltips dismiss permanently once seen or once the related action is taken
- [ ] Advanced/rare options are hidden behind progressive disclosure, not shown to first-time users by default
- [ ] New accounts have example/seed content where appropriate to demonstrate value immediately
- [ ] Signup collects only what's essential to start; further profiling is deferred and tied to a clear benefit
- [ ] Account creation success is explicitly confirmed, not silently assumed
- [ ] Required verification flows show clear status and offer a resend path, without fully blocking exploration where feasible
- [ ] Empty-state copy differentiates "nothing created yet" from "filtered/searched to nothing"
