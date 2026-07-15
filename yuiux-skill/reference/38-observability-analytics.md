# 38 — Observability & Analytics

> You cannot improve what you cannot measure. Analytics tells you what users do. Error monitoring tells you when things break. Together they close the feedback loop between shipping and learning.

---

## The Observability Stack

```
Layer 1: Error monitoring     — Sentry, Bugsnag, Rollbar
Layer 2: Performance          — Datadog, New Relic, web-vitals
Layer 3: Product analytics    — PostHog, Mixpanel, Amplitude, Heap
Layer 4: Session replay       — Hotjar, FullStory, PostHog
Layer 5: Feature flags        — LaunchDarkly, PostHog, Unleash
```

---

## Error Monitoring (Sentry)

```tsx
// sentry.ts — initialize once at app root
import * as Sentry from '@sentry/react';
import { BrowserTracing } from '@sentry/tracing';

Sentry.init({
  dsn: import.meta.env.VITE_SENTRY_DSN,
  environment: import.meta.env.MODE,                // 'development' | 'production'
  release: import.meta.env.VITE_APP_VERSION,        // git SHA or version
  enabled: import.meta.env.PROD,                    // disable in development
  
  integrations: [
    new BrowserTracing({
      routingInstrumentation: Sentry.reactRouterV6Instrumentation(
        React.useEffect,
        useLocation,
        useNavigationType,
        createRoutesFromChildren,
        matchRoutes
      ),
    }),
  ],
  
  // Performance monitoring
  tracesSampleRate: 0.1,           // 10% of transactions traced
  
  // Reduce noise: ignore common non-actionable errors
  ignoreErrors: [
    'ResizeObserver loop limit exceeded',
    'Network request failed',
    /^Loading chunk/,              // code splitting race conditions
    /ChunkLoadError/,
  ],
  
  // Scrub sensitive data before sending
  beforeSend(event) {
    if (event.request?.headers?.Authorization) {
      delete event.request.headers.Authorization;
    }
    return event;
  },
});

// Wrap app with error boundary
export default Sentry.withProfiler(App);
```

### Error Boundary with Sentry

```tsx
// Sentry-aware error boundary
import { ErrorBoundary } from '@sentry/react';

function AppErrorBoundary({ children }: { children: React.ReactNode }) {
  return (
    <ErrorBoundary
      fallback={({ error, resetError }) => (
        <div role="alert" className="error-page">
          <AlertOctagonIcon aria-hidden />
          <h2>Something went wrong</h2>
          <p>We've been notified and are working on a fix.</p>
          <button type="button" onClick={resetError}>Try again</button>
          <a href="/">Go to homepage</a>
        </div>
      )}
      onError={(error, componentStack) => {
        // Extra context sent automatically to Sentry
        Sentry.setContext('component_stack', { stack: componentStack });
      }}
    >
      {children}
    </ErrorBoundary>
  );
}
```

### Capture Errors Manually

```tsx
// In async operations
async function fetchDashboard() {
  try {
    return await api.get('/dashboard');
  } catch (error) {
    Sentry.captureException(error, {
      tags: {
        feature: 'dashboard',
        userId: currentUser.id,
      },
      extra: {
        endpoint: '/dashboard',
        requestId: error.requestId,
      },
    });
    throw error;
  }
}

// Add user context after login
function onLogin(user: User) {
  Sentry.setUser({
    id: user.id,
    email: user.email,          // only if user consented
    username: user.username,
  });
}

// Clear on logout
function onLogout() {
  Sentry.setUser(null);
}
```

---

## Product Analytics

### Event Taxonomy

```typescript
// Consistent event naming: [Object] [Action] (past tense)
// Object: the thing being acted on
// Action: what happened

const EVENTS = {
  // Page views (auto-tracked by most tools)
  PAGE_VIEWED: 'Page Viewed',
  
  // Authentication
  SIGN_UP_STARTED:   'Sign Up Started',
  SIGN_UP_COMPLETED: 'Sign Up Completed',
  SIGN_IN:           'Signed In',
  SIGN_OUT:          'Signed Out',
  
  // Feature usage
  PROJECT_CREATED:   'Project Created',
  PROJECT_DELETED:   'Project Deleted',
  PROJECT_SHARED:    'Project Shared',
  FILE_UPLOADED:     'File Uploaded',
  EXPORT_STARTED:    'Export Started',
  EXPORT_COMPLETED:  'Export Completed',
  
  // Search
  SEARCH_PERFORMED:  'Search Performed',
  SEARCH_RESULT_CLICKED: 'Search Result Clicked',
  
  // UI interactions
  MODAL_OPENED:      'Modal Opened',
  FILTER_APPLIED:    'Filter Applied',
  SORT_CHANGED:      'Sort Changed',
  
  // Errors
  ERROR_SHOWN:       'Error Shown',
  API_ERROR:         'API Error',
} as const;
```

### Analytics Client

```typescript
// analytics.ts — thin wrapper around PostHog/Segment/Mixpanel
interface AnalyticsClient {
  identify: (userId: string, traits?: Record<string, unknown>) => void;
  track:    (event: string, properties?: Record<string, unknown>) => void;
  page:     (name?: string, properties?: Record<string, unknown>) => void;
  reset:    () => void;
}

// PostHog implementation
import posthog from 'posthog-js';

posthog.init(import.meta.env.VITE_POSTHOG_KEY, {
  api_host:          import.meta.env.VITE_POSTHOG_HOST ?? 'https://app.posthog.com',
  capture_pageview:  false,          // manual page tracking
  persistence:       'localStorage',
  
  // Session recordings (optional)
  session_recording: {
    maskAllInputs:  true,            // mask all input values for privacy
    maskInputFn:    (text) => '*'.repeat(text.length),
  },
  
  // Feature flags
  bootstrap: {
    featureFlags: initialFeatureFlags,
  },
});

export const analytics: AnalyticsClient = {
  identify(userId, traits) {
    posthog.identify(userId, traits);
  },
  
  track(event, properties) {
    if (import.meta.env.DEV) {
      console.debug('[Analytics]', event, properties);
    }
    posthog.capture(event, properties);
  },
  
  page(name, properties) {
    posthog.capture('$pageview', {
      $current_url: window.location.href,
      page_name:    name,
      ...properties,
    });
  },
  
  reset() {
    posthog.reset();
  },
};
```

### React Integration

```tsx
// Track page views on route change
function AnalyticsPageTracker() {
  const location = useLocation();
  
  useEffect(() => {
    analytics.page(document.title, {
      path: location.pathname,
      search: location.search,
    });
  }, [location]);
  
  return null;
}

// useTrack hook for component-level tracking
function useTrack() {
  const { user } = useAuth();
  
  return useCallback((event: string, properties?: Record<string, unknown>) => {
    analytics.track(event, {
      user_id:    user?.id,
      user_plan:  user?.plan,
      ...properties,
    });
  }, [user]);
}

// Usage:
function ExportButton({ projectId }: { projectId: string }) {
  const track = useTrack();
  
  async function handleExport() {
    track(EVENTS.EXPORT_STARTED, { project_id: projectId, format: 'pdf' });
    
    try {
      await exportProject(projectId, 'pdf');
      track(EVENTS.EXPORT_COMPLETED, { project_id: projectId, format: 'pdf', success: true });
    } catch {
      track(EVENTS.EXPORT_COMPLETED, { project_id: projectId, format: 'pdf', success: false });
    }
  }
  
  return <button onClick={handleExport}>Export PDF</button>;
}
```

---

## Feature Flags

```tsx
// Feature flag hook
function useFeatureFlag(flagKey: string): boolean {
  const [enabled, setEnabled] = useState(false);
  
  useEffect(() => {
    // PostHog
    const isEnabled = posthog.isFeatureEnabled(flagKey);
    setEnabled(!!isEnabled);
    
    // Reload on flag changes
    posthog.onFeatureFlags(() => {
      setEnabled(!!posthog.isFeatureEnabled(flagKey));
    });
  }, [flagKey]);
  
  return enabled;
}

// Usage:
function Dashboard() {
  const newDashboardEnabled = useFeatureFlag('new-dashboard-v2');
  
  return newDashboardEnabled ? <NewDashboard /> : <LegacyDashboard />;
}
```

---

## Privacy Compliance

```tsx
// Respect Do Not Track and GDPR consent
function initAnalytics(userConsent: boolean) {
  if (!userConsent) {
    // Don't initialize tracking at all
    console.debug('[Analytics] Tracking disabled — no consent');
    return;
  }
  
  posthog.init(/* ... */);
}

// Cookie consent banner
function CookieConsentBanner({ onAccept, onDecline }: ConsentProps) {
  return (
    <div
      className="cookie-banner"
      role="dialog"
      aria-modal="false"
      aria-label="Cookie consent"
    >
      <p>
        We use analytics cookies to understand how you use our product.
        We never sell your data.{' '}
        <a href="/privacy">Privacy policy</a>
      </p>
      <div className="cookie-banner-actions">
        <button type="button" onClick={onDecline} className="btn btn-outline">
          Decline optional
        </button>
        <button type="button" onClick={onAccept} className="btn btn-primary">
          Accept analytics
        </button>
      </div>
    </div>
  );
}
```

---

## Observability Checklist

### Error Monitoring
- [ ] Sentry (or equivalent) initialized in production
- [ ] Error boundaries wrap all major page sections
- [ ] User context set after login, cleared on logout
- [ ] Sensitive data scrubbed from error reports
- [ ] Non-actionable errors ignored (ResizeObserver, chunk load, etc.)
- [ ] Source maps uploaded so errors show original code

### Analytics
- [ ] Analytics client initialized with environment-specific DSN
- [ ] User identified after login with minimal safe properties
- [ ] Page views tracked on route change
- [ ] All key user actions tracked with consistent naming
- [ ] Error events tracked with context
- [ ] Tracking disabled in development (or logged to console only)

### Privacy
- [ ] Cookie consent collected before initializing tracking
- [ ] Session recordings mask all input values
- [ ] PII not sent in event properties (emails, phone numbers)
- [ ] Privacy policy links accessible from consent banner
- [ ] Do Not Track header respected

### Feature Flags
- [ ] Feature flag client initialized
- [ ] New features gated behind flags for gradual rollout
- [ ] Flag keys are descriptive strings, not booleans
- [ ] Default state of flags is the safe/old behavior
