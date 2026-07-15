---
name: checkout-ecommerce
description: Cart, checkout, pricing display, trust signals, and payment-error UX errors and fixes for web development.
---

# Checkout & E-commerce UX

Checkout is the single highest-stakes flow in commerce software — the Peak-End rule applies with maximum force here, since it's the very last step before money changes hands.

## Anti-pattern: Forced account creation before checkout
**What's wrong:** Requiring a full account signup (with password, email verification) before a first-time buyer can complete a purchase.
**Why it matters:** This is one of the most well-documented causes of cart abandonment — a purchase-ready user is asked to do unrelated extra work (choosing a password, verifying an email) at exactly the moment they wanted to be done.
**Fix:** Always offer guest checkout as a first-class path; offer account creation as an optional, low-friction step (ideally just "save this info for next time," or an easy post-purchase account creation using the info they already entered) rather than a gate.

## Anti-pattern: Hidden fees/costs revealed only at the final step
**What's wrong:** Shipping, taxes, or service fees appear for the first time on the last checkout screen, after the user has already invested several steps.
**Why it matters:** This is a classic "drip pricing" pattern that damages trust the moment it's discovered, and in many jurisdictions is now explicitly regulated against — but even where legal, it reliably increases abandonment right at the finish line.
**Fix:** Show a realistic total cost estimate (including shipping/tax where calculable) as early as possible — ideally on the product/cart page, updated live as the user selects options — never for the first time on the final review screen.

## Anti-pattern: Cart quantity/removal controls are fiddly or ambiguous
**What's wrong:** A tiny "-" button with no visible resulting quantity update, or a "remove" action that's a small "x" easy to miss or easy to hit by accident right next to the quantity stepper.
**Why it matters:** Cart manipulation happens right before the highest-intent moment; friction or accidental removals here directly cost conversions.
**Fix:** Make quantity steppers large enough to tap confidently, show the current quantity clearly between the +/- controls, and separate "remove item" from the quantity control with enough spacing/visual distinction to avoid accidental clicks — ideally with an undo option if removed.

## Anti-pattern: No visible order summary during the checkout steps
**What's wrong:** Once the user moves past the cart into checkout, there's no persistent recap of what they're actually buying and the running total.
**Why it matters:** Users want ongoing confirmation of what they're committing to, especially across multi-step checkouts (shipping → payment → review).
**Fix:** Keep an order summary panel (items, quantities, running total) visible throughout every step of checkout, not just on the initial cart page.

## Anti-pattern: Payment form doesn't support autofill or common card-entry conventions
**What's wrong:** Custom-built card number/expiry/CVC fields with incorrect `autocomplete` attributes or non-standard input segmentation that breaks browser/password-manager autofill.
**Why it matters:** See `06-forms-inputs.md` — payment forms are exactly where autofill friction costs real conversions.
**Fix:** Use correct autocomplete tokens (`cc-number`, `cc-exp`, `cc-csc`, `cc-name`) and `inputmode="numeric"`; where using a hosted payment element (Stripe Elements, etc.), prefer it over a fully custom implementation since it already solves this correctly.

## Anti-pattern: Payment failure gives no specific reason or recovery path
**What's wrong:** "Payment failed" with no indication of why (declined card? wrong CVC? expired card? network issue on our end?) and no clear next step.
**Why it matters:** At the exact moment a user is trying to give a business money, an unexplained failure is maximally frustrating and likely to end the purchase attempt entirely.
**Fix:** Surface the most specific reason available from the payment processor in plain language ("Your card was declined. Try a different card or contact your bank.") and always leave the rest of the form intact so the user can retry with a different payment method without re-entering shipping/contact info.

## Anti-pattern: No trust signals near the payment step
**What's wrong:** A checkout form with no security badges, no clear statement of what data is collected/stored, no recognizable payment logos (Visa/Mastercard/etc.), especially on a less-established brand's site.
**Why it matters:** Entering payment information is inherently a trust-sensitive moment; the absence of expected trust signals can itself raise suspicion, even for a legitimate business.
**Fix:** Show recognized payment method logos, a clear (not overly long) statement about security/data handling near the payment fields, and if using a well-known payment processor, its branding (e.g. "Payments securely processed by Stripe") — this is a legitimate trust signal, not just decoration.

## Anti-pattern: Pricing display is ambiguous about billing frequency or currency
**What's wrong:** "$49" shown with no indication of whether that's one-time, monthly, or annual; no currency symbol/code shown when serving international users.
**Why it matters:** Users make purchase decisions based on price — ambiguity here either causes bill-shock later (eroding trust) or an abandoned purchase from uncertainty.
**Fix:** Always pair a price with its billing frequency ("$49/month," "$490/year — save 17%") and an explicit currency indicator for any audience where currency isn't a safe assumption.

## Anti-pattern: Discount/promo code field prominently placed with no code to enter, or hard to find with a valid one
**What's wrong:** A visible "Enter promo code" field showing right at the point of highest purchase intent, inviting users with no code to go hunt for one elsewhere (search "brand promo code," landing on a coupon-aggregator site, and potentially abandoning the purchase entirely) — or conversely, burying the field so users with a valid code can't find where to apply it.
**Why it matters:** An overly prominent, always-visible empty promo field actively invites cart abandonment for users who otherwise had no intention of looking for a discount; but hiding it entirely frustrates users who do have a legitimate code.
**Fix:** Use a collapsed, clearly-labeled but low-visual-weight "Have a promo code?" toggle rather than an always-expanded, prominent input field.

## Anti-pattern: No order confirmation, or confirmation that doesn't clearly state the transaction succeeded
**What's wrong:** After payment, the user lands on a page that doesn't clearly say "Order confirmed" or doesn't show an order number/summary, leaving ambiguity about whether the purchase actually went through.
**Why it matters:** This is the Peak-End moment of the entire flow — an unclear ending leaves the user anxious and likely to double-check (or double-purchase) out of uncertainty.
**Fix:** Show an unambiguous confirmation (order number, items purchased, total charged, expected delivery/access date) and always also send a confirmation email as a durable record.

## Anti-pattern: "Buy now" / irreversible purchase actions with no final review step
**What's wrong:** A single click immediately charges the card with no review screen showing exactly what's about to be purchased and for how much.
**Why it matters:** Per error-prevention principles, an irreversible, financially consequential action deserves a review step before commitment — accidental purchases erode trust fast and are costly to reverse (refunds, support burden).
**Fix:** Always show a final review (items, total, shipping/billing destination, payment method) with an explicit final confirming action before the charge is actually made — unless the product context is a genuinely established one-click-purchase pattern the user has explicitly opted into (e.g. a saved "buy again" flow they've used before).

## Quick Checklist
- [ ] Guest checkout is available; account creation is optional, not a gate
- [ ] Full cost (including tax/shipping where calculable) is shown as early as possible, never first revealed at the final step
- [ ] Cart quantity and remove controls are easy to use accurately, with undo on accidental removal
- [ ] Order summary stays visible throughout every checkout step
- [ ] Payment fields use correct autocomplete/inputmode, or a trusted hosted payment element
- [ ] Payment failures give a specific, plain-language reason and preserve all other entered info
- [ ] Recognized trust signals (payment logos, processor branding, security statement) are present near payment fields
- [ ] Price displays always pair amount with billing frequency and currency
- [ ] Promo code entry is present but low-prominence, not an always-expanded empty field
- [ ] Order confirmation is unambiguous (order number, summary, total) and backed by a confirmation email
- [ ] Irreversible/financial actions have a final review step before commitment
