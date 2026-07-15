# 33 — Destructive Actions & Confirmation

> The cost of a false negative (user hesitates) is low. The cost of a false positive (user loses data) is catastrophic. Design destructive actions to be hard to trigger accidentally and easy to recover from.

---

## The Spectrum of Destruction

```
Level 1: Recoverable, low stakes
  → No confirmation needed
  Example: Remove item from cart, collapse sidebar, dismiss notification

Level 2: Recoverable, medium stakes
  → Brief undo toast (5–10 seconds)
  Example: Delete email, archive project, remove team member

Level 3: Recoverable with friction, high stakes
  → Confirmation dialog with clear consequences
  Example: Delete project (can be restored from trash), cancel subscription

Level 4: Irreversible, catastrophic
  → Confirmation dialog + type-to-confirm
  Example: Delete account, permanently delete data, drop database, revoke API key
```

---

## Level 2: Undo Toast Pattern

```tsx
// Soft delete with undo window
function useDestructiveAction<T>(
  action: (item: T) => Promise<void>,
  undo: (item: T) => Promise<void>,
  options = { undoWindowMs: 6000 }
) {
  return async function execute(item: T, label: string) {
    // Optimistic removal from UI
    removeFromUI(item);
    
    let undoClicked = false;
    
    const { dismiss } = toast(
      <div className="toast-with-undo">
        <span>{label}</span>
        <button
          type="button"
          className="toast-undo-btn"
          onClick={() => {
            undoClicked = true;
            dismiss();
            restoreToUI(item);
            toast.success('Restored');
          }}
        >
          Undo
        </button>
      </div>,
      { duration: options.undoWindowMs }
    );
    
    // Wait for undo window, then commit
    await new Promise(resolve => setTimeout(resolve, options.undoWindowMs));
    
    if (!undoClicked) {
      try {
        await action(item);
      } catch {
        restoreToUI(item);
        toast.error(`Failed to delete. The item has been restored.`);
      }
    }
  };
}

// Usage:
const deleteFile = useDestructiveAction(
  file => api.delete(`/files/${file.id}`),
  file => api.restore(`/files/${file.id}`)
);

<button onClick={() => deleteFile(file, `"${file.name}" deleted`)}>
  Delete
</button>
```

---

## Level 3: Confirmation Dialog

```tsx
function ConfirmDialog({
  open,
  onClose,
  onConfirm,
  title,
  description,
  confirmLabel,
  cancelLabel = 'Cancel',
  variant = 'default',
  loading = false,
}: ConfirmDialogProps) {
  // Focus the cancel button by default — safer default
  const cancelRef = useRef<HTMLButtonElement>(null);
  
  useEffect(() => {
    if (open) cancelRef.current?.focus();
  }, [open]);
  
  return (
    <AlertDialog open={open} onOpenChange={onClose}>
      <AlertDialogContent role="alertdialog" aria-modal="true">
        <AlertDialogHeader>
          {/* Icon for visual clarity */}
          <div
            className={`dialog-icon-wrapper ${variant === 'destructive' ? 'destructive' : ''}`}
            aria-hidden
          >
            {variant === 'destructive'
              ? <AlertTriangleIcon />
              : <HelpCircleIcon />
            }
          </div>
          <AlertDialogTitle>{title}</AlertDialogTitle>
          <AlertDialogDescription>{description}</AlertDialogDescription>
        </AlertDialogHeader>
        
        <AlertDialogFooter>
          {/* Cancel first (left/top) — easier to reach, safer default */}
          <AlertDialogCancel ref={cancelRef} asChild>
            <button type="button">{cancelLabel}</button>
          </AlertDialogCancel>
          
          {/* Confirm last (right/bottom) — requires deliberate action */}
          <LoadingButton
            type="button"
            loading={loading}
            onClick={onConfirm}
            className={variant === 'destructive' ? 'btn btn-destructive' : 'btn btn-primary'}
          >
            {confirmLabel}
          </LoadingButton>
        </AlertDialogFooter>
      </AlertDialogContent>
    </AlertDialog>
  );
}

// Composable hook for managing confirmation dialogs
function useConfirmDialog() {
  const [state, setState] = useState<{
    open: boolean;
    config: ConfirmDialogConfig | null;
    resolve: ((confirmed: boolean) => void) | null;
  }>({ open: false, config: null, resolve: null });
  
  const confirm = useCallback((config: ConfirmDialogConfig): Promise<boolean> => {
    return new Promise(resolve => {
      setState({ open: true, config, resolve });
    });
  }, []);
  
  function handleClose(confirmed: boolean) {
    state.resolve?.(confirmed);
    setState({ open: false, config: null, resolve: null });
  }
  
  const Dialog = () => state.config ? (
    <ConfirmDialog
      open={state.open}
      onClose={() => handleClose(false)}
      onConfirm={() => handleClose(true)}
      {...state.config}
    />
  ) : null;
  
  return { confirm, Dialog };
}

// Usage pattern:
function ProjectList() {
  const { confirm, Dialog } = useConfirmDialog();
  
  async function handleDelete(project: Project) {
    const confirmed = await confirm({
      title: `Delete "${project.name}"?`,
      description: `This will permanently delete the project and all ${project.fileCount} files. Members will lose access immediately.`,
      confirmLabel: 'Delete project',
      cancelLabel: 'Keep project',
      variant: 'destructive',
    });
    
    if (!confirmed) return;
    
    try {
      await deleteProject(project.id);
      toast.success('Project deleted');
    } catch {
      toast.error('Failed to delete project. Try again.');
    }
  }
  
  return (
    <>
      {projects.map(p => (
        <ProjectCard key={p.id} project={p} onDelete={handleDelete} />
      ))}
      <Dialog />
    </>
  );
}
```

---

## Level 4: Type-to-Confirm

```tsx
function TypeToConfirmDialog({
  open,
  onClose,
  onConfirm,
  title,
  description,
  confirmText,          // user must type this exact string
  confirmLabel,
  loading,
}: TypeToConfirmProps) {
  const [input, setInput] = useState('');
  const confirmed = input === confirmText;
  
  // Reset input when dialog opens
  useEffect(() => {
    if (open) setInput('');
  }, [open]);
  
  return (
    <AlertDialog open={open} onOpenChange={onClose}>
      <AlertDialogContent>
        <AlertDialogHeader>
          <div className="dialog-icon-wrapper destructive" aria-hidden>
            <AlertOctagonIcon />
          </div>
          <AlertDialogTitle>{title}</AlertDialogTitle>
          <AlertDialogDescription>{description}</AlertDialogDescription>
        </AlertDialogHeader>
        
        <div className="type-to-confirm">
          <p>
            To confirm, type <strong>{confirmText}</strong> below:
          </p>
          <input
            type="text"
            value={input}
            onChange={e => setInput(e.target.value)}
            placeholder={confirmText}
            autoComplete="off"
            autoFocus
            className={`type-confirm-input ${confirmed ? 'confirmed' : ''}`}
            aria-label={`Type "${confirmText}" to confirm`}
            aria-describedby="type-confirm-hint"
          />
          <p id="type-confirm-hint" className="sr-only">
            Type {confirmText} exactly to enable the {confirmLabel} button.
          </p>
        </div>
        
        <AlertDialogFooter>
          <AlertDialogCancel onClick={onClose}>Cancel</AlertDialogCancel>
          <LoadingButton
            type="button"
            loading={loading}
            disabled={!confirmed}
            onClick={onConfirm}
            className="btn btn-destructive"
            aria-disabled={!confirmed}
          >
            {confirmLabel}
          </LoadingButton>
        </AlertDialogFooter>
      </AlertDialogContent>
    </AlertDialog>
  );
}
```

---

## Placement and Styling Rules

```tsx
// Destructive button placement
// Always: Secondary/cancel button BEFORE the destructive button

// ❌ Destructive on left — easy to click accidentally
<>
  <button className="btn btn-destructive">Delete</button>
  <button className="btn btn-outline">Cancel</button>
</>

// ✅ Destructive on right — requires deliberate reach
<>
  <button className="btn btn-outline">Cancel</button>
  <button className="btn btn-destructive">Delete</button>
</>

// Exception: on mobile (stacked), destructive goes BELOW cancel
// because stacked layout removes left/right ordering
```

```css
/* Destructive button styling — always clearly dangerous */
.btn-destructive {
  background: var(--color-danger);
  color: white;
  border: 1px solid var(--color-danger);
}
.btn-destructive:hover {
  background: var(--color-danger-hover);
  border-color: var(--color-danger-hover);
}
.btn-destructive:focus-visible {
  outline-color: var(--color-danger);
}
.btn-destructive:disabled {
  background: var(--color-danger);
  opacity: 0.5;
  cursor: not-allowed;
}
```

---

## Copy Rules for Destructive Actions

```
Buttons:
  ❌ "Yes" / "OK" / "Confirm"
  ✅ "Delete project" / "Cancel subscription" / "Permanently delete account"
  Rule: Button copy = verb + object (names exactly what will be destroyed)

Dialog headings:
  ❌ "Are you sure?"
  ✅ "Delete 'Marketing Campaign'?" (names the specific thing)

Dialog description:
  ❌ "This action cannot be undone."
  ✅ "This will delete the campaign and all 14 scheduled posts. Analytics data will be preserved."
  Rule: Specific consequences, not vague warnings

Cancel buttons:
  ❌ "No"
  ✅ "Cancel" or "Keep campaign"
  Rule: Cancel should feel safe — "Keep [thing]" is even clearer
```

---

## Checklist

- [ ] No destructive action at Level 1 skips confirmation (unless undo is implemented)
- [ ] Level 2 actions (recoverable) use undo toast with ≥5s window
- [ ] Level 3 actions use confirmation dialog naming the specific item
- [ ] Level 4 actions use type-to-confirm (email or resource name)
- [ ] Cancel button appears BEFORE destructive button (left or top)
- [ ] Cancel button is focused by default in dialogs (not confirm)
- [ ] Destructive buttons use red/danger styling, not default
- [ ] Dialog heading names the specific resource (not "Are you sure?")
- [ ] Dialog description states specific consequences (not "cannot be undone")
- [ ] Primary button copy is Verb + Object ("Delete project", not "OK")
- [ ] Loading state shown while async delete is in progress
- [ ] Error recovery if delete fails (item restored, error shown)
- [ ] Data export offered before account deletion
