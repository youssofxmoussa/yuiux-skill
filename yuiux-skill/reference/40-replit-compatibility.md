# 40 — Replit Compatibility

> Replit has specific conventions for monorepos, artifact routing, port configuration, and environment setup. This file documents every UX-relevant pattern adapted for the Replit environment.

---

## Replit Environment Overview

```
Runtime:    NixOS Linux container
Package mgr: pnpm workspaces (monorepo)
Preview:    Proxied iframe — localhost:<PORT> → REPLIT_DEV_DOMAIN/<path>
Secrets:    Replit Secrets (not .env files)
Deploy:     Replit Deployments (separate production environment)
```

---

## Port Configuration

```typescript
// ✅ Always use PORT environment variable — never hardcode
const PORT = parseInt(process.env.PORT ?? '3000', 10);

// Vite — vite.config.ts
export default defineConfig({
  server: {
    port:         Number(process.env.PORT ?? 5173),
    host:         '0.0.0.0',           // required — bind to all interfaces
    allowedHosts: 'all',               // required — iframe proxy uses different origin
    strictPort:   false,               // allow Vite to find an open port
  },
  preview: {
    port:  Number(process.env.PORT ?? 5173),
    host:  '0.0.0.0',
    allowedHosts: 'all',
  },
});

// Express — server.ts
const app = express();
app.listen(PORT, '0.0.0.0', () => {
  console.log(`Server listening on port ${PORT}`);
});
```

---

## Artifact Routing and BASE_URL

Each artifact is mounted at a path prefix (e.g., `/my-app`). All internal links and API calls must be relative to this prefix:

```typescript
// Get the base URL for the current artifact
const BASE_URL = import.meta.env.BASE_URL; // set by Vite from artifact config
// e.g., "/my-app/" or "/" for root artifact

// ✅ Prefix all API calls
const apiUrl = `${BASE_URL}api/users`;
fetch(apiUrl);

// ✅ React Router — use basename
import { BrowserRouter } from 'react-router-dom';

<BrowserRouter basename={import.meta.env.BASE_URL}>
  <App />
</BrowserRouter>

// ❌ Never hardcode the full domain in app code
const apiUrl = `https://${process.env.REPLIT_DEV_DOMAIN}/api/users`; // wrong!
// (REPLIT_DEV_DOMAIN is for shell/curl only, not app code)
```

---

## Environment Variables and Secrets

```typescript
// ✅ Access secrets via process.env (server-side)
const apiKey = process.env.MY_API_KEY; // set in Replit Secrets panel

// ✅ Expose to client via Vite define/env
// vite.config.ts
export default defineConfig({
  define: {
    'import.meta.env.MY_PUBLIC_KEY': JSON.stringify(process.env.MY_PUBLIC_KEY),
  },
});

// ✅ Client code
const key = import.meta.env.VITE_MY_PUBLIC_KEY; // VITE_ prefix exposes to client

// ❌ Never commit .env files with real secrets
// ❌ Never log secret values
// ❌ Never put secret values in client-side code
```

---

## pnpm Monorepo Structure

```
workspace/
├── package.json          ← root workspace config
├── pnpm-workspace.yaml   ← defines workspace packages
├── artifacts/
│   ├── my-app/           ← web app artifact
│   │   ├── package.json
│   │   ├── vite.config.ts
│   │   └── src/
│   └── mobile/           ← mobile artifact
│       ├── package.json
│       └── app/
└── packages/
    └── shared/           ← shared library
        ├── package.json
        └── src/
```

```yaml
# pnpm-workspace.yaml
packages:
  - 'artifacts/*'
  - 'packages/*'
```

```json
// artifacts/my-app/package.json
{
  "name": "@workspace/my-app",
  "dependencies": {
    "@workspace/shared": "workspace:*"
  }
}
```

---

## Replit-Specific UX Patterns

### Preview Pane Considerations

The user always sees the app through a proxied iframe. Consequences:

```typescript
// ✅ Always use relative URLs or BASE_URL-prefixed URLs
// ✅ Never embed localhost URLs in app code
// ✅ CORS: your own API doesn't need CORS headers (same-origin via proxy)

// OG/social meta tags: use production URL from Replit Deployments
// During development: meta tags don't matter for preview pane
```

### Cache Busting in Development

```typescript
// server.ts — disable caching in development
if (process.env.NODE_ENV !== 'production') {
  app.use((req, res, next) => {
    res.setHeader('Cache-Control', 'no-store, no-cache, must-revalidate');
    res.setHeader('Pragma', 'no-cache');
    next();
  });
}
```

### Static Assets

```typescript
// Express: serve static files from build output
app.use(import.meta.env.BASE_URL, express.static('dist'));

// Catch-all for SPA client-side routing
app.get(`${import.meta.env.BASE_URL}*`, (req, res) => {
  res.sendFile(path.join(__dirname, 'dist', 'index.html'));
});
```

---

## Database Patterns (Replit PostgreSQL)

```typescript
// Replit provides DATABASE_URL in environment for connected PostgreSQL databases
import { Pool } from 'pg';

const pool = new Pool({
  connectionString: process.env.DATABASE_URL,
  ssl: process.env.NODE_ENV === 'production'
    ? { rejectUnauthorized: false }
    : false,
});

// Connection test on startup
pool.query('SELECT 1').then(() => {
  console.log('Database connected');
}).catch(err => {
  console.error('Database connection failed:', err.message);
  process.exit(1);
});
```

```typescript
// With Drizzle ORM (recommended)
import { drizzle } from 'drizzle-orm/node-postgres';
import { Pool } from 'pg';
import * as schema from './schema';

const pool = new Pool({ connectionString: process.env.DATABASE_URL });
export const db = drizzle(pool, { schema });
```

---

## Object Storage (App Storage)

```typescript
// Replit App Storage for file uploads
import { Client } from '@replit/object-storage';

const client = new Client();

// Upload
async function uploadFile(key: string, buffer: Buffer, mimeType: string) {
  await client.uploadFromBytes(key, buffer, {
    contentType: mimeType,
  });
  return `/${key}`;
}

// Download
async function getFile(key: string): Promise<Buffer> {
  return client.downloadAsBytes(key);
}

// List
async function listFiles(prefix: string) {
  return client.list(prefix);
}
```

---

## Replit Auth Integration

```typescript
// Replit Auth — built-in OpenID Connect SSO
// From the replit-auth skill:

// server.ts
import { setupAuth } from './auth';
setupAuth(app);

// Routes
app.get('/api/auth/user', requireAuth, (req, res) => {
  res.json(req.user);
});

// React hook
function useAuth() {
  const { data: user, isLoading } = useQuery({
    queryKey: ['/api/auth/user'],
    retry: false,
  });
  return { user, isLoading, isAuthenticated: !!user };
}

// Protected route
function ProtectedRoute({ children }) {
  const { user, isLoading } = useAuth();
  
  if (isLoading) return <LoadingScreen />;
  if (!user) return <Navigate to="/login" />;
  return children;
}
```

---

## Workflow Configuration

```toml
# artifact.toml — managed by Replit, do not edit directly
# Use the artifact skills to configure
[services.web]
  command = "pnpm run dev"
  port    = 3000
  path    = "/my-app"

[services.api]
  command = "pnpm run server"
  port    = 8080
  path    = "/api"
```

---

## Replit Checklist

- [ ] Server binds to `0.0.0.0` (not `localhost` or `127.0.0.1`)
- [ ] Port reads from `process.env.PORT` environment variable
- [ ] Vite `server.allowedHosts: 'all'` (iframe proxy requirement)
- [ ] All internal API calls use `${BASE_URL}api/...` prefix
- [ ] React Router uses `basename={import.meta.env.BASE_URL}`
- [ ] No hardcoded `localhost` URLs in app code
- [ ] No hardcoded `REPLIT_DEV_DOMAIN` in app code (shell only)
- [ ] Secrets managed via Replit Secrets panel (not .env committed files)
- [ ] `VITE_` prefix on environment variables exposed to client
- [ ] Database connection uses `DATABASE_URL` from environment
- [ ] SSL enabled for database in production
- [ ] Static file serving includes base path prefix
- [ ] SPA catch-all route serves index.html for client routing
- [ ] No caching in development (helps with preview pane refreshes)
