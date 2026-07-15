# 06 — Forms & Inputs

> Forms are where users give you their data, their trust, and their patience. Every error in a form costs all three.

---

## The Form Contract

When a user fills out a form, they make an implicit contract: "I will give you accurate information if you make it easy to do so and treat my mistakes with respect." Every form design decision either honors or violates this contract.

---

## Labels

### The Absolute Rule
**Every form control must have an associated label.** No exceptions. Placeholder text is not a label.

```html
<!-- ✅ Explicit label — best practice -->
<label for="email">Email address</label>
<input id="email" type="email" name="email" />

<!-- ✅ Wrapping label — also valid -->
<label>
  Email address
  <input type="email" name="email" />
</label>

<!-- ✅ aria-label — for cases where visible label isn't possible -->
<input type="search" aria-label="Search products" />

<!-- ✅ aria-labelledby — for labels defined elsewhere -->
<h2 id="billing-heading">Billing information</h2>
<input aria-labelledby="billing-heading" type="text" />

<!-- ❌ Placeholder as label — disappears on focus, fails accessibility -->
<input type="email" placeholder="Email address" />

<!-- ❌ No label at all -->
<input type="email" />
```

### Why Placeholder Fails as Label
1. Disappears when user starts typing — they forget what the field is for
2. Has lower contrast (typically ~3.6:1 — fails WCAG AA)
3. Screen readers may not read it consistently
4. Can't distinguish between filled and empty fields at a glance

### Label Best Practices
```tsx
// Required fields — mark them clearly
function FieldLabel({ htmlFor, children, required }: LabelProps) {
  return (
    <label htmlFor={htmlFor} className="field-label">
      {children}
      {required && (
        <span className="required-mark" aria-hidden="true"> *</span>
      )}
      {required && (
        <span className="sr-only">(required)</span>
      )}
    </label>
  );
}

// Helper text — linked to input via aria-describedby
function FormField({ id, label, helperText, errorText, children }) {
  const helperId = helperText ? `${id}-helper` : undefined;
  const errorId  = errorText  ? `${id}-error`  : undefined;
  
  return (
    <div className="form-field">
      <label htmlFor={id}>{label}</label>
      {React.cloneElement(children, {
        id,
        'aria-describedby': [helperId, errorId].filter(Boolean).join(' ') || undefined,
        'aria-invalid': !!errorText,
      })}
      {helperText && (
        <p id={helperId} className="helper-text">{helperText}</p>
      )}
      {errorText && (
        <p id={errorId} className="error-text" role="alert">
          {errorText}
        </p>
      )}
    </div>
  );
}
```

---

## Input Types

Always use the most specific input type for the data:

```html
<input type="text">     — generic text (use sparingly)
<input type="email">    — email addresses (triggers email keyboard on mobile)
<input type="password"> — passwords (masks input)
<input type="number">   — numeric values (triggers number keyboard)
<input type="tel">      — phone numbers (triggers phone keyboard on mobile)
<input type="url">      — URLs (triggers URL keyboard)
<input type="search">   — search fields (adds clear button in some browsers)
<input type="date">     — date picker (native date UI)
<input type="time">     — time picker
<input type="datetime-local"> — date + time
<input type="color">    — color picker
<input type="range">    — slider
<input type="file">     — file upload
<input type="checkbox"> — boolean toggle
<input type="radio">    — single selection from group
```

**Benefit:** Correct `type` gives users the right keyboard on mobile, enables browser autofill, and may show native UI controls.

---

## Autocomplete Attributes

Always provide `autocomplete` attributes for personal data — enables browser and password manager autofill:

```html
<!-- Personal info -->
<input type="text"     autocomplete="name">
<input type="text"     autocomplete="given-name">
<input type="text"     autocomplete="family-name">
<input type="email"    autocomplete="email">
<input type="tel"      autocomplete="tel">
<input type="url"      autocomplete="url">

<!-- Address -->
<input type="text"     autocomplete="street-address">
<input type="text"     autocomplete="address-line1">
<input type="text"     autocomplete="address-line2">
<input type="text"     autocomplete="address-level2"> <!-- city -->
<input type="text"     autocomplete="address-level1"> <!-- state -->
<input type="text"     autocomplete="postal-code">
<select                autocomplete="country">

<!-- Authentication -->
<input type="text"     autocomplete="username">
<input type="email"    autocomplete="email">
<input type="password" autocomplete="current-password">
<input type="password" autocomplete="new-password">
<input type="text"     autocomplete="one-time-code">

<!-- Payment -->
<input type="text"     autocomplete="cc-name">
<input type="text"     autocomplete="cc-number">
<input type="text"     autocomplete="cc-exp">
<input type="text"     autocomplete="cc-csc">
```

---

## Validation Timing

The worst thing you can do is validate on every keypress. The best is to validate at the right moment.

### Validation Rules by Field Type

| Trigger | When to Use | Examples |
|---------|-------------|----------|
| On submit | Required fields, most fields | Name, address, most fields |
| On blur (when user leaves field) | Format-dependent fields | Email, phone, date, URL |
| After blur + on change | Fields with real-time feedback value | Password strength, username availability |
| Never on keydown | Most fields | (causes hostile error flashing) |
| On keyup (with debounce 300ms) | Only async checks | Username availability, email existence |

```tsx
function EmailField() {
  const [value, setValue] = useState('');
  const [error, setError] = useState('');
  const [touched, setTouched] = useState(false);
  
  function validate(val: string) {
    if (!val) return 'Email address is required';
    if (!val.includes('@')) return 'Enter a valid email like name@example.com';
    if (!val.match(/^[^\s@]+@[^\s@]+\.[^\s@]+$/)) return 'Enter a valid email address';
    return '';
  }
  
  function handleBlur() {
    setTouched(true);
    setError(validate(value));
  }
  
  function handleChange(e: React.ChangeEvent<HTMLInputElement>) {
    setValue(e.target.value);
    // Only re-validate on change AFTER the field has been touched
    if (touched) setError(validate(e.target.value));
  }
  
  return (
    <div className="form-field">
      <label htmlFor="email">Email address</label>
      <input
        id="email"
        type="email"
        value={value}
        onChange={handleChange}
        onBlur={handleBlur}
        aria-invalid={touched && !!error}
        aria-describedby={error ? 'email-error' : undefined}
        autoComplete="email"
      />
      {touched && error && (
        <p id="email-error" role="alert" className="error-text">
          {error}
        </p>
      )}
    </div>
  );
}
```

---

## Error Messages

### The Formula

**[What's wrong] + [Why] + [How to fix it]**

```
❌ "Invalid"
✅ "Enter a valid email address like name@example.com"

❌ "Error"
✅ "Password must be at least 8 characters"

❌ "This field is required"
✅ "Email address is required to create your account"

❌ "Invalid date"
✅ "Enter a date in MM/DD/YYYY format (e.g. 07/14/2026)"

❌ "Too long"
✅ "Bio must be 160 characters or fewer (yours is 247 characters)"

❌ "Already exists"
✅ "That username is taken — try adding numbers or try a different name"
```

### Error Display Rules

```tsx
// Error must:
// 1. Be visible without scrolling
// 2. Be placed immediately below or beside the field
// 3. Use role="alert" so screen readers announce it
// 4. Never disappear while the error still exists
// 5. Use color + icon (not color alone)

function ErrorMessage({ id, message }: { id: string; message: string }) {
  return (
    <p 
      id={id} 
      role="alert"
      className="flex items-center gap-1.5 text-sm text-destructive mt-1"
    >
      <AlertCircleIcon className="w-4 h-4 flex-shrink-0" aria-hidden />
      {message}
    </p>
  );
}
```

### Form-Level Error Summary

For forms with multiple errors, show a summary at the top:

```tsx
function FormErrorSummary({ errors }: { errors: Record<string, string> }) {
  const errorList = Object.entries(errors).filter(([, msg]) => msg);
  if (!errorList.length) return null;
  
  return (
    <div 
      role="alert"
      aria-live="polite"
      className="form-error-summary"
      tabIndex={-1} // allow programmatic focus
    >
      <h3>Please fix these errors:</h3>
      <ul>
        {errorList.map(([field, message]) => (
          <li key={field}>
            <a href={`#${field}`}>{message}</a>
          </li>
        ))}
      </ul>
    </div>
  );
}

// Focus the summary after failed submit:
const summaryRef = useRef<HTMLDivElement>(null);
function handleFailedSubmit() {
  setErrors(newErrors);
  summaryRef.current?.focus();
}
```

---

## Submit Behavior

### Never Wipe Input on Failed Submit

```tsx
// ❌ Wrong: loses user's work on error
async function handleSubmit(e) {
  e.preventDefault();
  const response = await submit(formData);
  if (response.error) {
    setFormData({}); // NEVER DO THIS
    setError(response.error);
  }
}

// ✅ Correct: preserve data, show inline errors
async function handleSubmit(e) {
  e.preventDefault();
  setSubmitting(true);
  try {
    await submit(formData);
    onSuccess();
  } catch (error) {
    setErrors(parseErrors(error)); // show errors inline
    // formData unchanged — user's work preserved
    scrollToFirstError();
  } finally {
    setSubmitting(false);
  }
}
```

### Submit Button States

```tsx
function SubmitButton({ submitting, disabled, children }) {
  return (
    <button
      type="submit"
      disabled={submitting || disabled}
      aria-busy={submitting}
      aria-disabled={disabled}
    >
      {submitting ? (
        <>
          <Spinner aria-hidden />
          <span>Saving…</span>
        </>
      ) : children}
    </button>
  );
}
```

### Scroll to First Error
```tsx
function scrollToFirstError(errors: Record<string, string>) {
  const firstErrorField = Object.keys(errors).find(field => errors[field]);
  if (!firstErrorField) return;
  
  const el = document.getElementById(firstErrorField);
  if (el) {
    el.scrollIntoView({ behavior: 'smooth', block: 'center' });
    el.focus();
  }
}
```

---

## Multi-Step Forms

### Progress Indication
```tsx
function FormProgress({ steps, currentStep }) {
  return (
    <nav aria-label="Form progress">
      <ol className="step-list">
        {steps.map((step, index) => {
          const status = index < currentStep ? 'complete' 
                       : index === currentStep ? 'current' 
                       : 'upcoming';
          return (
            <li 
              key={step.id}
              aria-current={status === 'current' ? 'step' : undefined}
              className={`step step--${status}`}
            >
              <span className="step-number" aria-hidden>
                {status === 'complete' ? '✓' : index + 1}
              </span>
              <span className="step-label">{step.label}</span>
            </li>
          );
        })}
      </ol>
    </nav>
  );
}
```

### State Preservation Between Steps
```tsx
// Save step data to prevent loss on navigation
function MultiStepForm() {
  const [formData, setFormData] = useState<FormData>({});
  const [currentStep, setCurrentStep] = useState(0);
  
  function updateStep(stepData: Partial<FormData>) {
    setFormData(prev => ({ ...prev, ...stepData })); // merge, never replace
  }
  
  function goBack() {
    setCurrentStep(prev => Math.max(0, prev - 1));
    // formData is preserved — user sees their previous answers
  }
  
  function goNext(stepData: Partial<FormData>) {
    updateStep(stepData);
    setCurrentStep(prev => prev + 1);
  }
  
  return (
    <div>
      <FormProgress steps={STEPS} currentStep={currentStep} />
      <StepComponent
        key={currentStep}
        initialData={formData} // pre-fill with saved data
        onNext={goNext}
        onBack={goBack}
      />
    </div>
  );
}
```

---

## Autosave

```tsx
// Autosave pattern for long forms
function AutosaveForm() {
  const [formData, setFormData] = useState(initialData);
  const [saveStatus, setSaveStatus] = useState<'idle' | 'saving' | 'saved' | 'error'>('idle');
  const saveTimeout = useRef<NodeJS.Timeout>();
  
  function handleChange(field: string, value: unknown) {
    setFormData(prev => ({ ...prev, [field]: value }));
    
    // Debounce autosave by 1 second
    clearTimeout(saveTimeout.current);
    setSaveStatus('idle');
    saveTimeout.current = setTimeout(async () => {
      setSaveStatus('saving');
      try {
        await saveDraft(formData);
        setSaveStatus('saved');
        setTimeout(() => setSaveStatus('idle'), 3000); // clear status after 3s
      } catch {
        setSaveStatus('error');
      }
    }, 1000);
  }
  
  return (
    <div>
      <SaveStatusIndicator status={saveStatus} />
      <form>
        {/* fields */}
      </form>
    </div>
  );
}

function SaveStatusIndicator({ status }) {
  const messages = {
    idle:   '',
    saving: 'Saving…',
    saved:  'Saved',
    error:  'Save failed — check your connection',
  };
  
  return status !== 'idle' ? (
    <p aria-live="polite" aria-atomic className="save-status">
      {messages[status]}
    </p>
  ) : null;
}
```

---

## Password Fields

```tsx
function PasswordField({ id, label, showStrength }: PasswordFieldProps) {
  const [value, setValue] = useState('');
  const [visible, setVisible] = useState(false);
  
  const strength = showStrength ? getPasswordStrength(value) : null;
  
  return (
    <div className="form-field">
      <label htmlFor={id}>{label}</label>
      <div className="input-with-action">
        <input
          id={id}
          type={visible ? 'text' : 'password'}
          value={value}
          onChange={e => setValue(e.target.value)}
          autoComplete="new-password"
          aria-describedby={[
            strength ? `${id}-strength` : '',
            `${id}-requirements`,
          ].filter(Boolean).join(' ')}
        />
        <button
          type="button"
          onClick={() => setVisible(!visible)}
          aria-label={visible ? 'Hide password' : 'Show password'}
          aria-pressed={visible}
        >
          {visible ? <EyeOffIcon aria-hidden /> : <EyeIcon aria-hidden />}
        </button>
      </div>
      
      {showStrength && value && (
        <div id={`${id}-strength`} aria-live="polite">
          <StrengthBar strength={strength} />
          <span className="sr-only">
            Password strength: {strengthLabel(strength)}
          </span>
        </div>
      )}
      
      <ul id={`${id}-requirements`} className="requirements-list">
        <li data-met={value.length >= 8}>At least 8 characters</li>
        <li data-met={/[A-Z]/.test(value)}>One uppercase letter</li>
        <li data-met={/[0-9]/.test(value)}>One number</li>
      </ul>
    </div>
  );
}

function getPasswordStrength(password: string): number {
  let score = 0;
  if (password.length >= 8)  score++;
  if (password.length >= 12) score++;
  if (/[A-Z]/.test(password)) score++;
  if (/[0-9]/.test(password)) score++;
  if (/[^A-Za-z0-9]/.test(password)) score++;
  return Math.min(4, score);
}
```

---

## Select & Combobox

```tsx
// Native select — for small, stable lists
<div className="form-field">
  <label htmlFor="country">Country</label>
  <select id="country" name="country" autoComplete="country">
    <option value="">Select a country</option>
    <option value="US">United States</option>
    <option value="GB">United Kingdom</option>
    {/* ... */}
  </select>
</div>

// Combobox — for large or searchable lists
// Use @radix-ui/react-select or cmdk for accessible implementation
import { Select, SelectTrigger, SelectValue, SelectContent, SelectItem } from '@/components/ui/select';

<div className="form-field">
  <label htmlFor="timezone">Time zone</label>
  <Select onValueChange={setTimezone} defaultValue={timezone}>
    <SelectTrigger id="timezone" aria-label="Select time zone">
      <SelectValue placeholder="Choose a time zone" />
    </SelectTrigger>
    <SelectContent>
      {timezones.map(tz => (
        <SelectItem key={tz.value} value={tz.value}>{tz.label}</SelectItem>
      ))}
    </SelectContent>
  </Select>
</div>
```

---

## Textarea

```tsx
function AutoResizingTextarea({ id, label, maxLength, ...props }) {
  const ref = useRef<HTMLTextAreaElement>(null);
  const [value, setValue] = useState('');
  
  function handleChange(e: React.ChangeEvent<HTMLTextAreaElement>) {
    setValue(e.target.value);
    // Auto-resize
    if (ref.current) {
      ref.current.style.height = 'auto';
      ref.current.style.height = `${ref.current.scrollHeight}px`;
    }
  }
  
  return (
    <div className="form-field">
      <label htmlFor={id}>{label}</label>
      <textarea
        ref={ref}
        id={id}
        value={value}
        onChange={handleChange}
        maxLength={maxLength}
        aria-describedby={maxLength ? `${id}-count` : undefined}
        style={{ overflow: 'hidden', resize: 'none' }}
        rows={3}
        {...props}
      />
      {maxLength && (
        <p 
          id={`${id}-count`}
          className="char-count"
          aria-live="polite"
          aria-atomic
        >
          <span aria-hidden>{value.length}</span>
          <span className="sr-only">{value.length} of {maxLength}</span>
          <span aria-hidden> / {maxLength}</span>
        </p>
      )}
    </div>
  );
}
```

---

## File Upload Field

```tsx
function FileUploadField({ accept, maxSizeMB = 10, onFile }) {
  const [dragging, setDragging] = useState(false);
  const [file, setFile] = useState<File | null>(null);
  const [error, setError] = useState('');
  const inputRef = useRef<HTMLInputElement>(null);
  
  function processFile(file: File) {
    setError('');
    if (file.size > maxSizeMB * 1024 * 1024) {
      setError(`File is too large. Maximum size is ${maxSizeMB} MB.`);
      return;
    }
    if (accept && !accept.split(',').some(type => file.type.match(type.trim()))) {
      setError(`File type not supported. Accepted types: ${accept}`);
      return;
    }
    setFile(file);
    onFile(file);
  }
  
  return (
    <div className="form-field">
      <label htmlFor="file-upload">Upload file</label>
      <div
        className={`drop-zone ${dragging ? 'dragging' : ''}`}
        onDragOver={e => { e.preventDefault(); setDragging(true); }}
        onDragLeave={() => setDragging(false)}
        onDrop={e => {
          e.preventDefault();
          setDragging(false);
          const f = e.dataTransfer.files[0];
          if (f) processFile(f);
        }}
        role="button"
        tabIndex={0}
        onClick={() => inputRef.current?.click()}
        onKeyDown={e => { if (e.key === 'Enter' || e.key === ' ') inputRef.current?.click(); }}
        aria-label="Upload file — click or drag and drop"
      >
        <input
          ref={inputRef}
          id="file-upload"
          type="file"
          accept={accept}
          className="sr-only"
          onChange={e => { const f = e.target.files?.[0]; if (f) processFile(f); }}
          aria-describedby={error ? 'file-error' : undefined}
        />
        {file ? (
          <p><FileIcon aria-hidden /> {file.name} ({formatFileSize(file.size)})</p>
        ) : (
          <p>Drop a file here or click to browse</p>
        )}
      </div>
      {error && (
        <p id="file-error" role="alert" className="error-text">{error}</p>
      )}
    </div>
  );
}
```

---

## Form Checklist

- [ ] Every input has a `<label>` with matching `htmlFor`/`id`
- [ ] Required fields marked with both visual indicator and `aria-required`
- [ ] Helper text linked via `aria-describedby`
- [ ] Error messages linked via `aria-describedby` + `aria-invalid`
- [ ] Error messages use `role="alert"` for screen reader announcement
- [ ] Validation fires on blur, not on every keypress
- [ ] Re-validates on change only AFTER first blur (touched state)
- [ ] Form errors shown inline, below each field
- [ ] Multi-error forms show summary at top
- [ ] Failed submit does NOT wipe entered data
- [ ] Submit button shows loading state during async
- [ ] Correct `type` attribute for each input (email, tel, etc.)
- [ ] `autocomplete` attributes set for personal data
- [ ] Password field has show/hide toggle
- [ ] Multi-step forms preserve state when going back
- [ ] Long forms have autosave with visible status indicator
- [ ] Error copy follows [what's wrong + how to fix] formula
- [ ] Never uses placeholder as primary label
