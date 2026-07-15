# 23 — Onboarding UX

> Onboarding is not a tour of your product. It's the shortest path to the user's first moment of value.

---

## The First-Value Principle

The goal of onboarding is to get the user to their **Aha! moment** as fast as possible. The Aha! moment is the instant when the user understands what makes your product valuable to them personally.

```
Slack's Aha! moment:    First team message received
GitHub's Aha! moment:   First code pushed
Figma's Aha! moment:    First design shared and commented on
Notion's Aha! moment:   First page linked from another page
```

Every onboarding step that doesn't contribute to reaching that moment should be eliminated or deferred.

---

## Onboarding Patterns

### 1. Progressive Disclosure Onboarding

Show the product immediately, reveal features as they become relevant:

```tsx
function NewUserOverlay({ step, onDismiss, onNext }: OnboardingStepProps) {
  const steps = [
    {
      target: '#create-project-btn',
      title: 'Start here',
      description: 'Create your first project to get started.',
      placement: 'bottom',
    },
    {
      target: '#invite-team',
      title: 'Invite your team',
      description: 'Collaborate with up to 5 people for free.',
      placement: 'right',
    },
  ];
  
  const current = steps[step];
  if (!current) return null;
  
  return (
    <Spotlight target={current.target}>
      <Popover
        placement={current.placement}
        role="dialog"
        aria-modal="true"
        aria-labelledby="onboarding-title"
      >
        <button aria-label="Close walkthrough" onClick={onDismiss}>×</button>
        <h3 id="onboarding-title">{current.title}</h3>
        <p>{current.description}</p>
        <div className="onboarding-footer">
          <span>{step + 1} of {steps.length}</span>
          <button onClick={onNext} className="btn btn-primary btn-sm">
            {step === steps.length - 1 ? 'Done' : 'Next'}
          </button>
        </div>
      </Popover>
    </Spotlight>
  );
}
```

### 2. Setup Wizard

For complex products requiring configuration before value is available:

```tsx
interface SetupStep {
  id: string;
  title: string;
  description: string;
  required: boolean;
  completed: boolean;
  href: string;
}

function SetupWizard({ steps }: { steps: SetupStep[] }) {
  const completedCount = steps.filter(s => s.completed).length;
  const progress = (completedCount / steps.length) * 100;
  const allRequired = steps.filter(s => s.required).every(s => s.completed);
  
  return (
    <div className="setup-wizard">
      <div className="setup-header">
        <h2>Set up your workspace</h2>
        <p>{completedCount} of {steps.length} steps completed</p>
      </div>
      
      <div
        className="setup-progress"
        role="progressbar"
        aria-valuenow={progress}
        aria-valuemin={0}
        aria-valuemax={100}
        aria-label={`Setup ${Math.round(progress)}% complete`}
      >
        <div className="setup-progress-fill" style={{ width: `${progress}%` }} />
      </div>
      
      <ul className="setup-steps" role="list">
        {steps.map(step => (
          <li key={step.id} className={`setup-step ${step.completed ? 'completed' : ''}`}>
            <a href={step.href} className="setup-step-link">
              <span className={`setup-step-icon ${step.completed ? 'completed' : ''}`} aria-hidden>
                {step.completed ? '✓' : '○'}
              </span>
              <div className="setup-step-content">
                <p className="setup-step-title">
                  {step.title}
                  {step.required && !step.completed && (
                    <span className="setup-step-required"> (required)</span>
                  )}
                </p>
                <p className="setup-step-desc">{step.description}</p>
              </div>
              <ChevronRightIcon aria-hidden />
            </a>
          </li>
        ))}
      </ul>
      
      {allRequired && (
        <div className="setup-complete-banner">
          <CheckCircleIcon aria-hidden />
          All required steps complete! You're ready to go.
        </div>
      )}
    </div>
  );
}
```

### 3. Empty State Onboarding

Turn empty states into onboarding prompts:

```tsx
// ❌ Generic empty state
<EmptyState title="No projects" />

// ✅ Onboarding empty state — guides new users
function ProjectsFirstVisit({ isFirstTime }: { isFirstTime: boolean }) {
  if (isFirstTime) {
    return (
      <div className="onboarding-empty">
        <div className="onboarding-illustration">
          <FolderIcon aria-hidden />
        </div>
        <h2>Create your first project</h2>
        <p>
          Projects help you organize your work, collaborate with your team,
          and track progress. Most teams start by importing an existing project
          or creating a new one from scratch.
        </p>
        <div className="onboarding-actions">
          <button className="btn btn-primary" onClick={openCreateDialog}>
            Create project
          </button>
          <button className="btn btn-outline" onClick={openImportDialog}>
            Import existing project
          </button>
        </div>
        <a href="/templates" className="onboarding-link">
          Start from a template →
        </a>
      </div>
    );
  }
  
  return (
    <EmptyState
      title="No projects"
      description="Create a project to get started."
      action={{ label: 'Create project', onClick: openCreateDialog }}
    />
  );
}
```

---

## Checklist-Based Onboarding

```tsx
// Persistent checklist widget (often in sidebar or header)
function OnboardingChecklist({ userId }) {
  const { checklist, markComplete, dismiss } = useOnboardingChecklist(userId);
  
  const remaining = checklist.filter(item => !item.completed);
  const progress = ((checklist.length - remaining.length) / checklist.length) * 100;
  
  if (checklist.every(item => item.completed) && !checklist.dismissed) {
    return (
      <div className="onboarding-complete">
        <span>🎉</span>
        <p>Setup complete! You're ready to go.</p>
        <button onClick={dismiss}>Dismiss</button>
      </div>
    );
  }
  
  return (
    <details className="onboarding-checklist" open>
      <summary>
        <span>Get started</span>
        <span className="checklist-count">
          {checklist.length - remaining.length}/{checklist.length}
        </span>
        <div
          className="checklist-mini-progress"
          style={{ background: `linear-gradient(to right, hsl(var(--primary)) ${progress}%, hsl(var(--muted)) ${progress}%)` }}
          role="presentation"
        />
      </summary>
      
      <ul>
        {checklist.map(item => (
          <li key={item.id} className={`checklist-item ${item.completed ? 'done' : ''}`}>
            <a href={item.href} className="checklist-link">
              <span
                className="checklist-check"
                aria-label={item.completed ? 'Completed' : 'Not completed'}
              >
                {item.completed ? '✓' : '○'}
              </span>
              <span className="checklist-text">{item.title}</span>
            </a>
          </li>
        ))}
      </ul>
    </details>
  );
}
```

---

## Onboarding Email Sequence

Each email in the sequence should:
1. Have one goal (not three)
2. Be triggered by behavior, not just time
3. Show the user what they're missing (not what features exist)

```
Day 0 (immediate):    Welcome + confirm email
Day 1 (if not active): "Here's how [customer name] gets value from X"
Day 3 (if no first action): "Most teams [complete key action] first"
Day 7 (if no retention): Re-engagement with a specific use case
Day 14 (churning):    "Is something not working? We're here to help"
```

---

## Onboarding Anti-Patterns

| Anti-Pattern | Problem | Fix |
|-------------|---------|-----|
| Required email verification before use | Kills activation rate | Allow limited use first, verify later |
| 10+ onboarding questions at signup | Abandonment | Ask max 3, defer rest to profile |
| Product tour on first login | Users aren't ready — they're exploring | Show hints when user encounters empty states |
| Forced completion before access | Frustrating | Make onboarding optional and dismissible |
| Generic empty states | Wasted onboarding opportunity | Make empty states context-aware for new users |
| All features at once | Cognitive overload | Progressive disclosure |
| No progress indicator in setup wizard | Users don't know how much is left | Always show X of N steps |
| No skip option | Condescending, assumed need | Always allow skip |

---

## Onboarding Checklist

- [ ] First-time user experience is distinct from returning user
- [ ] Onboarding reaches Aha! moment in ≤ 3 steps
- [ ] Setup wizard has clear progress (step X of N)
- [ ] Each step can be skipped or deferred
- [ ] Onboarding state persisted (user doesn't see it twice)
- [ ] Empty states guide new users with action buttons
- [ ] Welcome email sent immediately after signup
- [ ] No more than 3 required fields at signup
- [ ] Optional fields clearly marked as optional
- [ ] Product tour uses spotlight/coach marks, not modal blocking
- [ ] Checklist dismissible after completion
- [ ] "Skip setup" always available
