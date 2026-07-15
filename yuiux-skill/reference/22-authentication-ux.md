# 22 — Authentication UX

> Authentication is the first experience every user has with your product. Make it feel safe, smooth, and human.

---

## Sign-In / Sign-Up Principles

1. **Ask for the minimum** — only request what you need right now
2. **Preserve entered data** on errors — never wipe the form
3. **Show password strength** on sign-up — guide toward a good password
4. **Use social auth when appropriate** — reduces friction massively
5. **Don't require account for browsing** — defer auth until necessary
6. **Remember the user** — sensible defaults for `remember me`
7. **Explain why you need data** — "Email is required to send your receipt"

---

## Login Form

```tsx
function LoginForm({ onSuccess, onForgotPassword }: LoginFormProps) {
  const [form, setForm] = useState({ email: '', password: '' });
  const [errors, setErrors] = useState<Record<string, string>>({});
  const [submitting, setSubmitting] = useState(false);
  const [generalError, setGeneralError] = useState('');
  
  function validate() {
    const newErrors: Record<string, string> = {};
    if (!form.email) newErrors.email = 'Email address is required';
    else if (!isValidEmail(form.email)) newErrors.email = 'Enter a valid email address';
    if (!form.password) newErrors.password = 'Password is required';
    return newErrors;
  }
  
  async function handleSubmit(e: React.FormEvent) {
    e.preventDefault();
    const validationErrors = validate();
    if (Object.keys(validationErrors).length) {
      setErrors(validationErrors);
      return;
    }
    
    setSubmitting(true);
    setGeneralError('');
    
    try {
      await signIn(form.email, form.password);
      onSuccess();
    } catch (error) {
      // ✅ Generic error — don't reveal which field is wrong (security)
      setGeneralError(
        isNetworkError(error)
          ? 'Check your connection and try again.'
          : 'Email or password is incorrect. Please try again.'
      );
      // ✅ Never wipe the form on error
    } finally {
      setSubmitting(false);
    }
  }
  
  return (
    <form onSubmit={handleSubmit} noValidate aria-label="Sign in to your account">
      <h1>Sign in</h1>
      
      {/* General error — not tied to a specific field */}
      {generalError && (
        <div role="alert" className="form-error-banner">
          <AlertCircleIcon aria-hidden />
          {generalError}
        </div>
      )}
      
      <FormField
        id="email"
        label="Email address"
        type="email"
        value={form.email}
        onChange={v => setForm(prev => ({ ...prev, email: v }))}
        error={errors.email}
        autoComplete="email"
        required
        inputMode="email"
      />
      
      <FormField
        id="password"
        label="Password"
        type="password"
        value={form.password}
        onChange={v => setForm(prev => ({ ...prev, password: v }))}
        error={errors.password}
        autoComplete="current-password"
        required
      >
        <div className="field-row">
          <PasswordInput
            id="password"
            value={form.password}
            onChange={v => setForm(prev => ({ ...prev, password: v }))}
          />
          <a href="/forgot-password" className="forgot-link">
            Forgot password?
          </a>
        </div>
      </FormField>
      
      <LoadingButton type="submit" loading={submitting} className="btn btn-primary w-full">
        Sign in
      </LoadingButton>
      
      <p className="auth-footer">
        Don't have an account? <a href="/signup">Create account</a>
      </p>
    </form>
  );
}
```

---

## Sign-Up Form

```tsx
function SignUpForm({ onSuccess }: { onSuccess: () => void }) {
  const [step, setStep] = useState<'credentials' | 'profile'>('credentials');
  const [form, setForm] = useState({
    email: '', password: '', confirmPassword: '',
    firstName: '', lastName: '',
  });
  
  // Step 1: credentials
  // Step 2: profile (optional — defer until needed)
  
  return (
    <form aria-label="Create your account">
      {step === 'credentials' && (
        <>
          <h1>Create account</h1>
          <p>Already have an account? <a href="/login">Sign in</a></p>
          
          {/* Social auth first — highest conversion */}
          <SocialAuthButtons />
          
          <div className="divider" aria-label="or">Or continue with email</div>
          
          <FormField id="email" label="Email address" type="email" autoComplete="email" required />
          
          <PasswordField
            id="password"
            label="Password"
            showStrength
            autoComplete="new-password"
            required
          />
          
          <FormField
            id="confirm-password"
            label="Confirm password"
            type="password"
            autoComplete="new-password"
            required
          />
          
          <p className="auth-legal">
            By creating an account, you agree to our{' '}
            <a href="/terms">Terms of Service</a> and{' '}
            <a href="/privacy">Privacy Policy</a>.
          </p>
          
          <button type="submit" className="btn btn-primary w-full">
            Create account
          </button>
        </>
      )}
    </form>
  );
}

// Social auth buttons
function SocialAuthButtons() {
  return (
    <div className="social-auth">
      <button
        type="button"
        onClick={() => signInWithProvider('google')}
        className="btn btn-outline w-full"
        aria-label="Continue with Google"
      >
        <GoogleIcon aria-hidden />
        Continue with Google
      </button>
      <button
        type="button"
        onClick={() => signInWithProvider('github')}
        className="btn btn-outline w-full"
        aria-label="Continue with GitHub"
      >
        <GitHubIcon aria-hidden />
        Continue with GitHub
      </button>
    </div>
  );
}
```

---

## Password Reset Flow

```
Step 1: Enter email → show confirmation message
Step 2: User clicks link in email → goes to /reset-password?token=...
Step 3: Enter new password → show success → redirect to login
```

```tsx
// Step 1: Request reset
function ForgotPasswordForm() {
  const [email, setEmail] = useState('');
  const [sent, setSent] = useState(false);
  const [submitting, setSubmitting] = useState(false);
  
  if (sent) {
    return (
      <div className="auth-success" role="status">
        <CheckCircleIcon aria-hidden />
        <h2>Check your email</h2>
        <p>
          We've sent a password reset link to <strong>{email}</strong>.
          The link expires in 1 hour.
        </p>
        <p>
          Didn't receive the email?{' '}
          <button type="button" onClick={() => setSent(false)}>
            Try again
          </button>
        </p>
        <a href="/login">Back to sign in</a>
      </div>
    );
  }
  
  async function handleSubmit(e: React.FormEvent) {
    e.preventDefault();
    setSubmitting(true);
    try {
      // Always show success — don't reveal whether email exists
      await requestPasswordReset(email);
      setSent(true);
    } catch {
      // Same success message even on error — security best practice
      setSent(true);
    } finally {
      setSubmitting(false);
    }
  }
  
  return (
    <form onSubmit={handleSubmit}>
      <a href="/login">← Back to sign in</a>
      <h1>Reset your password</h1>
      <p>Enter the email address associated with your account and we'll send you a link to reset your password.</p>
      
      <FormField
        id="reset-email"
        label="Email address"
        type="email"
        value={email}
        onChange={setEmail}
        autoComplete="email"
        required
      />
      
      <LoadingButton type="submit" loading={submitting} className="btn btn-primary w-full">
        Send reset link
      </LoadingButton>
    </form>
  );
}
```

---

## Authentication Security UX

### Never Reveal Which Field Is Wrong

```tsx
// ❌ Reveals which field failed — enables enumeration attack
if (error.code === 'USER_NOT_FOUND') {
  setErrors({ email: 'No account with this email address' });
}

// ✅ Generic message — doesn't reveal which was wrong
setGeneralError('Email or password is incorrect. Please try again.');
```

### Never Reveal If Email Exists in Forgot Password

```tsx
// ❌ Reveals whether email is registered
if (error.code === 'USER_NOT_FOUND') {
  setError('No account with this email address.');
}

// ✅ Always show same message
// (see example above — setSent(true) even on error)
```

### Rate Limiting Feedback

```tsx
// Show helpful message when rate limited
if (error.code === 'RATE_LIMITED') {
  setGeneralError(
    `Too many attempts. Try again in ${error.retryAfterSeconds} seconds.`
  );
}
```

### Session Expiry

```tsx
function useSessionExpiryWarning() {
  const [showWarning, setShowWarning] = useState(false);
  const [timeLeft, setTimeLeft] = useState(0);
  
  useEffect(() => {
    const warnBefore = 5 * 60 * 1000; // warn 5 min before expiry
    
    // Subscribe to session expiry
    return onSessionExpiringSoon(remaining => {
      if (remaining <= warnBefore) {
        setShowWarning(true);
        setTimeLeft(Math.floor(remaining / 1000));
      }
    });
  }, []);
  
  return { showWarning, timeLeft };
}

function SessionExpiryBanner() {
  const { showWarning, timeLeft } = useSessionExpiryWarning();
  
  if (!showWarning) return null;
  
  const minutes = Math.floor(timeLeft / 60);
  const seconds = timeLeft % 60;
  
  return (
    <AlertBanner
      variant="warning"
      title="Your session is about to expire"
      description={`You'll be logged out in ${minutes}:${seconds.toString().padStart(2, '0')}. Save your work to avoid losing changes.`}
      action={{ label: 'Stay logged in', onClick: refreshSession }}
    />
  );
}
```

---

## MFA (Multi-Factor Authentication) UX

```tsx
function OTPVerificationForm({ onVerified, onResend }: OTPProps) {
  const [otp, setOtp] = useState(['', '', '', '', '', '']);
  const refs = useRef<Array<HTMLInputElement | null>>([]);
  
  function handleChange(index: number, value: string) {
    if (!/^\d?$/.test(value)) return; // digits only
    
    const newOtp = [...otp];
    newOtp[index] = value;
    setOtp(newOtp);
    
    // Auto-advance to next input
    if (value && index < 5) refs.current[index + 1]?.focus();
  }
  
  function handleKeyDown(index: number, e: React.KeyboardEvent) {
    if (e.key === 'Backspace' && !otp[index] && index > 0) {
      refs.current[index - 1]?.focus();
    }
  }
  
  function handlePaste(e: React.ClipboardEvent) {
    e.preventDefault();
    const text = e.clipboardData.getData('text').replace(/\D/g, '').slice(0, 6);
    const newOtp = [...otp];
    text.split('').forEach((char, i) => { newOtp[i] = char; });
    setOtp(newOtp);
    refs.current[Math.min(text.length, 5)]?.focus();
  }
  
  const code = otp.join('');
  
  return (
    <form onSubmit={e => { e.preventDefault(); onVerified(code); }}>
      <h2>Enter verification code</h2>
      <p>We sent a 6-digit code to your authenticator app.</p>
      
      <div
        className="otp-inputs"
        role="group"
        aria-label="One-time passcode"
      >
        {otp.map((digit, index) => (
          <input
            key={index}
            ref={el => { refs.current[index] = el; }}
            type="text"
            inputMode="numeric"
            pattern="\d{1}"
            maxLength={1}
            value={digit}
            onChange={e => handleChange(index, e.target.value)}
            onKeyDown={e => handleKeyDown(index, e)}
            onPaste={index === 0 ? handlePaste : undefined}
            aria-label={`Digit ${index + 1} of 6`}
            autoComplete={index === 0 ? 'one-time-code' : undefined}
            className="otp-input"
          />
        ))}
      </div>
      
      <button
        type="submit"
        disabled={code.length < 6}
        className="btn btn-primary w-full"
      >
        Verify
      </button>
      
      <p>
        Didn't receive a code?{' '}
        <button type="button" onClick={onResend}>Resend code</button>
      </p>
    </form>
  );
}
```

---

## Auth Checklist

- [ ] Login form: generic error (not field-specific) on wrong credentials
- [ ] Forgot password: same message whether email exists or not
- [ ] Password field has show/hide toggle
- [ ] Sign-up has password strength indicator
- [ ] Confirm password field on sign-up
- [ ] `autocomplete` attributes set on all auth fields
- [ ] `type="email"` on email fields, `type="password"` on password fields
- [ ] Failed login doesn't wipe entered email
- [ ] Loading state on submit button while authenticating
- [ ] Session expiry warning with option to extend
- [ ] MFA code input: auto-advance, paste support, backspace support
- [ ] Rate limiting shown with time until retry
- [ ] Legal links (terms, privacy) visible before sign-up submission
- [ ] Social auth options available (reduces friction)
