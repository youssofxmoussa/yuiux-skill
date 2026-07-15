# 31 — Payment & Checkout UX

> Every extra field, every page, every moment of uncertainty in checkout costs conversions. The best checkout is the one with the fewest decisions.

---

## Checkout Principles

1. **One-page when possible** — fewer navigation steps = fewer drop-offs
2. **Minimum required fields** — don't ask for phone if you don't need it
3. **No surprise costs** — show total (including shipping, tax) before final confirmation
4. **Trust signals throughout** — security badge, accepted cards, return policy
5. **Guest checkout always** — never force account creation to buy
6. **Show what's being purchased** — summary stays visible throughout

---

## Checkout Flow

```
Cart → (Shipping info) → Payment → Confirmation

Each page:
  - Shows order summary (always visible on desktop, collapsible on mobile)
  - Shows progress indicator
  - Has clear "back" option
  - Shows trust signals
```

```tsx
function CheckoutFlow() {
  const [step, setStep] = useState<'contact' | 'shipping' | 'payment' | 'confirm'>('contact');
  const [order, setOrder] = useState<Partial<OrderData>>({});
  
  const steps = [
    { id: 'contact', label: 'Contact' },
    { id: 'shipping', label: 'Shipping' },
    { id: 'payment', label: 'Payment' },
    { id: 'confirm', label: 'Review' },
  ];
  
  const currentIndex = steps.findIndex(s => s.id === step);
  
  return (
    <div className="checkout-layout">
      {/* Left: Form steps */}
      <main className="checkout-main">
        {/* Progress */}
        <nav aria-label="Checkout steps">
          <ol className="checkout-progress">
            {steps.map((s, i) => (
              <li
                key={s.id}
                aria-current={s.id === step ? 'step' : undefined}
                className={`checkout-step ${i < currentIndex ? 'complete' : i === currentIndex ? 'active' : ''}`}
              >
                {i < currentIndex ? (
                  <button type="button" onClick={() => setStep(s.id as typeof step)}>
                    {s.label}
                  </button>
                ) : (
                  <span>{s.label}</span>
                )}
              </li>
            ))}
          </ol>
        </nav>
        
        {step === 'contact' && (
          <ContactStep
            data={order}
            onNext={data => { setOrder(prev => ({ ...prev, ...data })); setStep('shipping'); }}
          />
        )}
        {step === 'shipping' && (
          <ShippingStep
            data={order}
            onNext={data => { setOrder(prev => ({ ...prev, ...data })); setStep('payment'); }}
            onBack={() => setStep('contact')}
          />
        )}
        {step === 'payment' && (
          <PaymentStep
            data={order}
            onNext={data => { setOrder(prev => ({ ...prev, ...data })); setStep('confirm'); }}
            onBack={() => setStep('shipping')}
          />
        )}
        {step === 'confirm' && (
          <ConfirmStep
            order={order}
            onBack={() => setStep('payment')}
          />
        )}
      </main>
      
      {/* Right: Order summary */}
      <aside className="checkout-summary" aria-label="Order summary">
        <OrderSummary items={order.items} />
      </aside>
    </div>
  );
}
```

---

## Contact / Address Form

```tsx
function ContactStep({ data, onNext }) {
  const form = useForm({
    defaultValues: data,
    resolver: zodResolver(contactSchema),
  });
  
  return (
    <form onSubmit={form.handleSubmit(onNext)}>
      <h2>Contact information</h2>
      
      {/* Guest checkout — no account required */}
      <p>
        Have an account? <a href="/login?next=/checkout">Sign in</a> for faster checkout.
      </p>
      
      <FormField
        control={form.control}
        name="email"
        label="Email address"
        type="email"
        autoComplete="email"
        description="Order confirmation will be sent to this email"
        required
      />
      
      <h3>Shipping address</h3>
      
      <div className="form-row">
        <FormField name="firstName" label="First name" autoComplete="given-name" required />
        <FormField name="lastName"  label="Last name"  autoComplete="family-name" required />
      </div>
      
      <FormField
        name="address1"
        label="Address"
        autoComplete="address-line1"
        required
      />
      <FormField
        name="address2"
        label="Apartment, suite, etc."
        autoComplete="address-line2"
      />
      
      <div className="form-row form-row--3">
        <FormField name="city"    label="City"       autoComplete="address-level2" required />
        <FormField name="state"   label="State"      autoComplete="address-level1" required />
        <FormField name="zipCode" label="ZIP code"   autoComplete="postal-code" required />
      </div>
      
      <FormField
        name="country"
        label="Country"
        as="select"
        autoComplete="country-name"
        required
      >
        <CountryOptions />
      </FormField>
      
      <button type="submit" className="btn btn-primary checkout-next">
        Continue to shipping
      </button>
    </form>
  );
}
```

---

## Payment Form

```tsx
// Stripe Elements — never handle raw card numbers yourself
import { CardElement, useStripe, useElements } from '@stripe/react-stripe-js';

function PaymentStep({ onNext, onBack }) {
  const stripe = useStripe();
  const elements = useElements();
  const [error, setError] = useState('');
  const [loading, setLoading] = useState(false);
  
  async function handleSubmit(e: React.FormEvent) {
    e.preventDefault();
    if (!stripe || !elements) return;
    
    setLoading(true);
    setError('');
    
    const { error: stripeError, paymentMethod } = await stripe.createPaymentMethod({
      type: 'card',
      card: elements.getElement(CardElement)!,
    });
    
    if (stripeError) {
      setError(stripeError.message ?? 'Payment failed. Please try again.');
      setLoading(false);
      return;
    }
    
    onNext({ paymentMethodId: paymentMethod.id });
  }
  
  return (
    <form onSubmit={handleSubmit}>
      <h2>Payment</h2>
      
      {/* Express checkout options first */}
      <div className="express-checkout">
        <p className="express-checkout-label">Express checkout</p>
        <div className="express-checkout-buttons">
          <ApplePayButton />
          <GooglePayButton />
        </div>
        <div className="divider">Or pay with card</div>
      </div>
      
      {/* Card form */}
      <div className="card-form-field">
        <label htmlFor="card-element">Card details</label>
        <div id="card-element-wrapper">
          <CardElement
            id="card-element"
            options={{
              style: {
                base: {
                  fontSize: '16px',
                  color: 'hsl(var(--foreground))',
                  fontFamily: 'Inter, system-ui, sans-serif',
                  '::placeholder': { color: 'hsl(var(--muted-foreground))' },
                },
                invalid: { color: 'hsl(var(--destructive))' },
              },
            }}
          />
        </div>
      </div>
      
      {error && (
        <div role="alert" className="form-error">
          <AlertCircleIcon aria-hidden />
          {error}
        </div>
      )}
      
      {/* Trust signals */}
      <div className="payment-trust">
        <LockIcon aria-hidden />
        <span>Your payment info is encrypted and secure</span>
        <img src="/images/card-brands.svg" alt="Visa, Mastercard, Amex, Discover accepted" />
      </div>
      
      <div className="checkout-actions">
        <button type="button" onClick={onBack} className="btn btn-ghost">
          ← Back
        </button>
        <LoadingButton type="submit" loading={loading} className="btn btn-primary">
          Review order
        </LoadingButton>
      </div>
    </form>
  );
}
```

---

## Order Confirmation

```tsx
function OrderConfirmation({ order }: { order: CompletedOrder }) {
  return (
    <div className="order-confirmation" role="status" aria-live="polite">
      <div className="confirmation-icon">
        <CheckCircleIcon aria-hidden />
      </div>
      
      <h1>Order confirmed!</h1>
      <p>Order <strong>#{order.id}</strong></p>
      <p>
        We've sent a confirmation to <strong>{order.email}</strong>.
        Your order will ship within 2–3 business days.
      </p>
      
      <section aria-labelledby="order-summary-heading">
        <h2 id="order-summary-heading">Your order</h2>
        {/* Line items */}
        {order.items.map(item => (
          <OrderItem key={item.id} item={item} />
        ))}
        <OrderTotal subtotal={order.subtotal} shipping={order.shipping} tax={order.tax} total={order.total} />
      </section>
      
      <a href="/orders" className="btn btn-primary">
        View your orders
      </a>
    </div>
  );
}
```

---

## Checkout UX Rules

```
1. Show accepted payment methods before the payment step
2. Real-time card validation (Stripe Elements handles this)
3. Never auto-advance out of a field (jarring, error-prone)
4. Auto-format card numbers (XXXX XXXX XXXX XXXX) — Stripe handles
5. Show "Submit" button copy as the price: "Pay $42.99"
6. Disable submit until all required fields complete
7. Show "What's included" if pricing isn't obvious
8. Error recovery: never erase the card number on error
```

---

## Checkout Checklist

- [ ] Guest checkout available (no forced account creation)
- [ ] Progress bar shows current checkout step
- [ ] Order summary visible throughout (sidebar on desktop, collapsible on mobile)
- [ ] Total shown before final confirmation (no surprise fees)
- [ ] Email field in contact step (for receipt before account creation)
- [ ] `autocomplete` attributes on all address fields
- [ ] Apple Pay / Google Pay express options visible
- [ ] Card input using Stripe Elements (never raw card number handling)
- [ ] Payment errors show human-readable message
- [ ] Submit button shows price: "Pay $42.99"
- [ ] Security badge/lock icon near payment form
- [ ] Accepted card brands shown near payment form
- [ ] Return/refund policy link visible before confirmation
- [ ] Order confirmation page with order number
- [ ] Confirmation email sent immediately
- [ ] No redirect to third-party payment page without warning
