# 32 — Settings & Preferences

> Settings are where trust is built or broken. Users go to settings when something isn't working or they want control. Design settings to feel safe, organized, and predictable.

---

## Settings Design Principles

1. **Immediate save by default** — no "Save" button for simple toggles
2. **Explicit save for complex forms** — group changes, show save button
3. **Organize by topic, not by implementation** — user's mental model, not the database schema
4. **Dangerous settings at the bottom** — account deletion, data export, logout
5. **Explain what each setting does** — never label only, always describe
6. **Show current state** — user should know what's on/off at a glance
7. **Confirm before irreversible changes** — password change, email change, account deletion

---

## Settings Layout

```tsx
function SettingsLayout() {
  const sections = [
    { id: 'profile',       label: 'Profile',         icon: UserIcon },
    { id: 'account',       label: 'Account',         icon: ShieldIcon },
    { id: 'notifications', label: 'Notifications',   icon: BellIcon },
    { id: 'appearance',    label: 'Appearance',      icon: PaintbrushIcon },
    { id: 'billing',       label: 'Billing & plans', icon: CreditCardIcon },
    { id: 'integrations',  label: 'Integrations',    icon: PuzzleIcon },
    { id: 'danger',        label: 'Danger zone',     icon: AlertTriangleIcon },
  ];
  
  const { section = 'profile' } = useParams();
  
  return (
    <div className="settings-layout">
      {/* Sidebar nav */}
      <nav aria-label="Settings sections" className="settings-nav">
        <ul role="list">
          {sections.map(s => {
            const Icon = s.icon;
            return (
              <li key={s.id}>
                <Link
                  to={`/settings/${s.id}`}
                  aria-current={section === s.id ? 'page' : undefined}
                  className={`settings-nav-item ${section === s.id ? 'active' : ''} ${s.id === 'danger' ? 'danger' : ''}`}
                >
                  <Icon aria-hidden className="settings-nav-icon" />
                  {s.label}
                </Link>
              </li>
            );
          })}
        </ul>
      </nav>
      
      {/* Content */}
      <main className="settings-main" id="settings-content">
        <Routes>
          <Route path="profile"       element={<ProfileSettings />} />
          <Route path="account"       element={<AccountSettings />} />
          <Route path="notifications" element={<NotificationSettings />} />
          <Route path="appearance"    element={<AppearanceSettings />} />
          <Route path="billing"       element={<BillingSettings />} />
          <Route path="danger"        element={<DangerZoneSettings />} />
          <Route index                element={<Navigate to="profile" replace />} />
        </Routes>
      </main>
    </div>
  );
}
```

---

## Settings Section Pattern

```tsx
// Each settings section follows this structure
function SettingsSection({
  title,
  description,
  children,
}: {
  title: string;
  description?: string;
  children: React.ReactNode;
}) {
  return (
    <section className="settings-section" aria-labelledby="section-title">
      <div className="settings-section-header">
        <h2 id="section-title" className="settings-section-title">{title}</h2>
        {description && <p className="settings-section-desc">{description}</p>}
      </div>
      <div className="settings-section-body">{children}</div>
    </section>
  );
}

// Settings row — single setting
function SettingsRow({
  label,
  description,
  control,
  divider = true,
}: SettingsRowProps) {
  return (
    <div className={`settings-row ${divider ? 'with-divider' : ''}`}>
      <div className="settings-row-label">
        <p className="settings-row-title">{label}</p>
        {description && <p className="settings-row-desc">{description}</p>}
      </div>
      <div className="settings-row-control">{control}</div>
    </div>
  );
}
```

---

## Immediate Save vs Explicit Save

```tsx
// Immediate save — for simple toggles and selects
function NotificationSettings() {
  const { settings, update, saving } = useSettings();
  
  async function handleToggle(key: string, value: boolean) {
    await update({ [key]: value });
    toast.success('Saved');
  }
  
  return (
    <SettingsSection title="Notifications" description="Control what you're notified about.">
      <SettingsRow
        label="Email notifications"
        description="Receive email updates about your account activity"
        control={
          <Switch
            checked={settings.emailNotifications}
            onChange={v => handleToggle('emailNotifications', v)}
            aria-label="Email notifications"
          />
        }
      />
      <SettingsRow
        label="Marketing emails"
        description="Tips, feature announcements, and product updates"
        control={
          <Switch
            checked={settings.marketingEmails}
            onChange={v => handleToggle('marketingEmails', v)}
            aria-label="Marketing emails"
          />
        }
      />
    </SettingsSection>
  );
}

// Explicit save — for complex forms with multiple related fields
function ProfileSettings() {
  const { profile } = useProfile();
  const { register, handleSubmit, formState } = useForm({ defaultValues: profile });
  const [saving, setSaving] = useState(false);
  
  async function onSubmit(data) {
    setSaving(true);
    try {
      await updateProfile(data);
      toast.success('Profile updated');
    } catch {
      toast.error('Failed to update profile. Try again.');
    } finally {
      setSaving(false);
    }
  }
  
  return (
    <SettingsSection title="Profile" description="This is how others see you on the platform.">
      <form onSubmit={handleSubmit(onSubmit)}>
        <FormField label="Display name" {...register('displayName')} required />
        <FormField label="Bio" {...register('bio')} as="textarea" rows={3} />
        <AvatarUpload current={profile.avatarUrl} />
        
        <div className="settings-form-footer">
          <LoadingButton type="submit" loading={saving} disabled={!formState.isDirty}>
            Save changes
          </LoadingButton>
          {formState.isDirty && (
            <span className="unsaved-indicator" aria-live="polite">
              Unsaved changes
            </span>
          )}
        </div>
      </form>
    </SettingsSection>
  );
}
```

---

## Dangerous Settings

```tsx
function DangerZoneSettings() {
  return (
    <SettingsSection
      title="Danger zone"
      description="Irreversible and destructive actions."
    >
      <div className="danger-zone">
        {/* Export before delete — good UX to offer data export */}
        <div className="danger-item">
          <div>
            <p className="danger-item-title">Export your data</p>
            <p className="danger-item-desc">
              Download a copy of all your data before deleting your account.
            </p>
          </div>
          <button type="button" className="btn btn-outline" onClick={requestDataExport}>
            Export data
          </button>
        </div>
        
        {/* Account deletion — last and most prominent danger */}
        <div className="danger-item danger-item--critical">
          <div>
            <p className="danger-item-title">Delete account</p>
            <p className="danger-item-desc">
              Permanently delete your account and all associated data.
              This action cannot be undone.
            </p>
          </div>
          <DeleteAccountButton />
        </div>
      </div>
    </SettingsSection>
  );
}

function DeleteAccountButton() {
  const [dialogOpen, setDialogOpen] = useState(false);
  const [confirmation, setConfirmation] = useState('');
  const [deleting, setDeleting] = useState(false);
  const { user } = useAuth();
  
  const CONFIRM_TEXT = user.email;
  const confirmed = confirmation === CONFIRM_TEXT;
  
  async function handleDelete() {
    if (!confirmed) return;
    setDeleting(true);
    try {
      await deleteAccount();
      // Redirect to homepage after deletion
      window.location.href = '/?account_deleted=true';
    } catch {
      toast.error('Failed to delete account. Contact support if this persists.');
      setDeleting(false);
    }
  }
  
  return (
    <>
      <button
        type="button"
        className="btn btn-destructive"
        onClick={() => setDialogOpen(true)}
      >
        Delete account
      </button>
      
      {dialogOpen && (
        <AlertDialog
          open
          onClose={() => setDialogOpen(false)}
          title="Delete your account?"
          description="This will permanently delete your account, all your projects, and all associated data. This cannot be undone."
        >
          <label htmlFor="confirm-delete">
            To confirm, type your email address: <strong>{CONFIRM_TEXT}</strong>
          </label>
          <input
            id="confirm-delete"
            type="text"
            value={confirmation}
            onChange={e => setConfirmation(e.target.value)}
            placeholder={CONFIRM_TEXT}
            autoComplete="off"
            autoFocus
          />
          <div className="dialog-actions">
            <button type="button" onClick={() => setDialogOpen(false)}>
              Cancel
            </button>
            <LoadingButton
              type="button"
              onClick={handleDelete}
              disabled={!confirmed}
              loading={deleting}
              className="btn btn-destructive"
            >
              Permanently delete account
            </LoadingButton>
          </div>
        </AlertDialog>
      )}
    </>
  );
}
```

---

## Settings Checklist

- [ ] Settings organized by user's mental model (not database schema)
- [ ] Simple toggles save immediately (no "Save" button)
- [ ] Complex forms have explicit "Save" button with dirty state indicator
- [ ] All settings have descriptions explaining what they do
- [ ] Dangerous settings at the bottom in clearly labeled "Danger zone"
- [ ] Account deletion requires typing email confirmation
- [ ] Data export offered before account deletion
- [ ] Settings nav has `aria-current="page"` for current section
- [ ] Success toast shows after each save (immediate or explicit)
- [ ] Error shown if save fails (never silent failure)
- [ ] Unsaved changes warned before navigating away (beforeunload)
- [ ] Email change requires re-verification
- [ ] Password change requires current password
- [ ] Settings deep-linkable via URL (e.g., /settings/notifications)
