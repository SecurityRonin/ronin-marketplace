---
name: vercel-deployment
description: Use when deploying to Vercel - covers vercel.json configuration, environment variables (printf gotcha), monorepo setup, Next.js issues, preview deployments, and common build errors
---

# Vercel Deployment

## When to Use This Skill

- Configuring `vercel.json` for deployments
- Setting environment variables via CLI
- Deploying monorepos
- Troubleshooting build failures
- Understanding preview vs production deployments

---

## vercel.json Configuration

### Minimal Configuration

```json
{
  "$schema": "https://openapi.vercel.sh/vercel.json"
}
```

Most projects don't need `vercel.json` - Vercel auto-detects frameworks.

### Common Configuration

```json
{
  "$schema": "https://openapi.vercel.sh/vercel.json",
  "buildCommand": "npm run build",
  "outputDirectory": "dist",
  "installCommand": "npm install",
  "framework": "nextjs",
  "regions": ["iad1"],
  "functions": {
    "api/**/*.ts": {
      "memory": 1024,
      "maxDuration": 30
    }
  }
}
```

### Redirects and Rewrites

```json
{
  "redirects": [
    { "source": "/old-page", "destination": "/new-page", "permanent": true }
  ],
  "rewrites": [
    { "source": "/api/:path*", "destination": "https://api.example.com/:path*" }
  ],
  "headers": [
    {
      "source": "/(.*)",
      "headers": [
        { "key": "X-Frame-Options", "value": "DENY" }
      ]
    }
  ]
}
```

---

## Environment Variables

### Critical Gotcha: Trailing Newlines

<EXTREMELY-IMPORTANT>
When piping values to `vercel env add`, use `printf`, NEVER `echo`.

`echo` adds a trailing newline that becomes part of the value, breaking API keys and secrets.
</EXTREMELY-IMPORTANT>

```bash
# ❌ WRONG - echo adds trailing newline
echo "sk-abc123" | vercel env add SECRET_KEY production

# ✅ CORRECT - printf has no trailing newline
printf "sk-abc123" | vercel env add SECRET_KEY production

# ✅ ALSO CORRECT - interactive prompt
vercel env add SECRET_KEY production
# Then paste value when prompted
```

**Symptoms of trailing newline bug:**
- API calls fail with "invalid key"
- Authentication errors despite correct credentials
- Value looks correct in dashboard but doesn't work

**Diagnosis:**
```bash
# Check for trailing newline
vercel env pull .env.local
cat -A .env.local | grep SECRET_KEY
# If you see SECRET_KEY=sk-abc123$ - no newline (good)
# If you see SECRET_KEY=sk-abc123\n$ - has newline (bad)
```

### Environment Types

| Type | When Used | Example |
|------|-----------|---------|
| `production` | Production deployments only | API keys, database URLs |
| `preview` | Preview deployments (PRs, branches) | Staging API keys |
| `development` | Local dev via `vercel dev` | Local overrides |

```bash
# Add to production only
vercel env add API_KEY production

# Add to all environments
vercel env add API_KEY production preview development

# Pull to local .env
vercel env pull .env.local
```

### Framework-Specific Prefixes

| Framework | Public Prefix | Private (server-only) |
|-----------|--------------|----------------------|
| Next.js | `NEXT_PUBLIC_*` | No prefix |
| Vite | `VITE_*` | No prefix |
| Create React App | `REACT_APP_*` | No prefix |

```bash
# Client-accessible (bundled into JS)
vercel env add NEXT_PUBLIC_API_URL production

# Server-only (API routes, SSR)
vercel env add DATABASE_URL production
```

---

## Monorepo Deployment

### Root Configuration

```json
// vercel.json at repo root
{
  "buildCommand": "cd apps/web && npm run build",
  "outputDirectory": "apps/web/dist",
  "installCommand": "npm install",
  "rootDirectory": "apps/web"
}
```

### Multiple Apps from One Repo

Create separate Vercel projects, each with different `rootDirectory`:

**Project 1 (Web App):**
```json
{
  "rootDirectory": "apps/web"
}
```

**Project 2 (API):**
```json
{
  "rootDirectory": "apps/api"
}
```

### Turborepo / pnpm Workspaces

```json
{
  "buildCommand": "cd ../.. && pnpm turbo build --filter=web",
  "outputDirectory": ".next",
  "rootDirectory": "apps/web"
}
```

**Common issue:** Build fails because dependencies aren't installed.

**Fix:** Set install command at root level:
```json
{
  "installCommand": "cd ../.. && pnpm install",
  "buildCommand": "cd ../.. && pnpm turbo build --filter=web",
  "rootDirectory": "apps/web"
}
```

---

## Next.js Specific

### Common Build Errors

**Error: `useX must be used within Provider`**

```
Error: useAuth must be used within an AuthProvider
Error occurred prerendering page "/dashboard"
```

**Cause:** Next.js tries to statically prerender pages that use React Context.

**Fix:** Wrap providers in `layout.tsx`:

```tsx
// src/components/Providers.tsx
'use client'

export default function Providers({ children }) {
  return <AuthProvider>{children}</AuthProvider>
}

// src/app/layout.tsx
import Providers from '../components/Providers'

export default function RootLayout({ children }) {
  return (
    <html>
      <body>
        <Providers>{children}</Providers>
      </body>
    </html>
  )
}
```

---

**Error: `NEXT_PUBLIC_* undefined at build time`**

**Cause:** Environment variable not set in Vercel project settings.

**Fix:**
1. Add variable in Vercel dashboard → Settings → Environment Variables
2. Or use `vercel env add NEXT_PUBLIC_API_URL production`
3. Redeploy (env vars are baked in at build time for `NEXT_PUBLIC_*`)

---

**Error: `Dynamic server usage` / `opted out of static rendering`**

```
Error: Dynamic server usage: cookies
Route /dashboard couldn't be rendered statically
```

**Cause:** Using dynamic functions (`cookies()`, `headers()`) in pages Vercel tries to statically generate.

**Fix:** Export dynamic route config:

```tsx
// Force dynamic rendering
export const dynamic = 'force-dynamic'

// Or force static
export const dynamic = 'force-static'
```

---

### Output Modes

```js
// next.config.js
module.exports = {
  output: 'standalone',  // For Docker/self-hosting
  // output: 'export',   // Static HTML export (no server features)
}
```

**Vercel auto-detects** - usually don't need to set this.

---

## Preview Deployments

### Automatic Previews

Every push to a non-production branch creates a preview deployment:
- `feature-branch` → `project-feature-branch-xxx.vercel.app`
- PR comments show preview URL automatically

### Preview Environment Variables

Preview deployments use `preview` environment variables:

```bash
# Production database
vercel env add DATABASE_URL production
# Value: postgres://prod-db...

# Preview/staging database
vercel env add DATABASE_URL preview
# Value: postgres://staging-db...
```

### Disable Previews

```json
// vercel.json
{
  "git": {
    "deploymentEnabled": {
      "main": true,
      "feature/*": false
    }
  }
}
```

---

## CLI Commands

### Installation & Auth

```bash
npm install -g vercel
vercel login
vercel whoami
```

### Deployment

```bash
# Deploy to preview
vercel

# Deploy to production
vercel --prod

# Deploy without prompts
vercel --prod --yes

# Deploy specific directory
vercel ./dist --prod
```

### Project Management

```bash
# Link local project to Vercel
vercel link

# List deployments
vercel list

# View deployment logs
vercel logs <deployment-url>

# Inspect deployment
vercel inspect <deployment-url>

# Promote deployment to production
vercel promote <deployment-url>
```

### Environment Variables

```bash
# List env vars
vercel env ls

# Add env var
vercel env add VAR_NAME production

# Remove env var
vercel env rm VAR_NAME production

# Pull env vars to local file
vercel env pull .env.local
```

### Rollback

```bash
# List recent deployments
vercel list

# Promote previous deployment to production
vercel promote <previous-deployment-url>

# Or instant rollback in dashboard
# Deployments → ... → Promote to Production
```

---

## Serverless Functions

### API Routes (Next.js)

```typescript
// app/api/hello/route.ts (App Router)
export async function GET(request: Request) {
  return Response.json({ message: 'Hello' })
}

// pages/api/hello.ts (Pages Router)
export default function handler(req, res) {
  res.status(200).json({ message: 'Hello' })
}
```

### Standalone Functions

```typescript
// api/hello.ts
import type { VercelRequest, VercelResponse } from '@vercel/node'

export default function handler(req: VercelRequest, res: VercelResponse) {
  res.status(200).json({ message: 'Hello' })
}
```

### Function Configuration

```json
// vercel.json
{
  "functions": {
    "api/**/*.ts": {
      "memory": 1024,      // MB (128-3008)
      "maxDuration": 30    // seconds (max 300 on Pro)
    }
  }
}
```

### Edge Functions

```typescript
// app/api/edge/route.ts
export const runtime = 'edge'

export async function GET(request: Request) {
  return new Response('Hello from the edge!')
}
```

**Edge vs Serverless:**
| Feature | Edge | Serverless |
|---------|------|------------|
| Cold start | ~0ms | 250-500ms |
| Memory | 128MB | Up to 3GB |
| Duration | 30s | Up to 300s |
| Node.js APIs | Limited | Full |
| Location | All regions | Selected region |

---

## Build Caching

### Turborepo Remote Cache

```bash
# Link to Vercel remote cache
npx turbo login
npx turbo link
```

```json
// turbo.json
{
  "remoteCache": {
    "signature": true
  }
}
```

### Clear Build Cache

If builds are stale or broken:

1. **Dashboard:** Settings → General → Build Cache → Purge
2. **CLI:** Redeploy with `vercel --force`

---

## Common Issues

### Build Timeout

**Error:** `Build exceeded maximum duration`

**Fixes:**
- Upgrade plan (Hobby: 45min, Pro: 45min)
- Optimize build (parallel builds, caching)
- Use `turbo prune` for monorepos

### Memory Exceeded

**Error:** `FATAL ERROR: JavaScript heap out of memory`

**Fix:** Increase Node memory in build command:
```json
{
  "buildCommand": "NODE_OPTIONS='--max-old-space-size=4096' npm run build"
}
```

### Module Not Found

**Error:** `Cannot find module 'x'`

**Causes:**
- Dependency in `devDependencies` but needed at runtime
- Case sensitivity (works on Mac, fails on Linux)
- Missing from `package.json`

**Fix:** Move to `dependencies` or check case sensitivity.

---

## OAuth Integration

### Callback URL Must Match Final Domain

If your domain redirects (e.g., `example.com` → `www.example.com`), the OAuth callback URL must use the **final destination domain**:

```
https://www.example.com/api/auth/callback  ✓
https://example.com/api/auth/callback      ✗ (if it redirects to www)
```

### Debugging OAuth Issues

```bash
# Check redirect URL for corruption
curl -s -I "https://your-app.com/api/auth/github" | grep location
```

Look for `%0A` (newline) or unexpected characters in `client_id` - indicates env var has trailing newline.

**Common errors:**
- `client_id and/or client_secret passed are incorrect` → Check for newlines in env vars
- `404 on callback` → Callback URL mismatch in OAuth app settings

---

## GCP Workload Identity Federation (WIF)

### Audience Mismatch

Vercel OIDC tokens have `aud: "https://vercel.com/{team-slug}"` but GCP providers often default to expecting `https://oidc.vercel.com/{team-slug}`.

**Diagnosis:**
```bash
gcloud iam workload-identity-pools providers describe {provider} \
  --location=global \
  --workload-identity-pool={pool} \
  --project={project} \
  --format="value(oidc.allowedAudiences)"
```

**Fix:** Update allowed audience to match Vercel's token:
```bash
gcloud iam workload-identity-pools providers update-oidc {provider} \
  --location=global \
  --workload-identity-pool={pool} \
  --project={project} \
  --allowed-audiences="https://vercel.com/{team-slug}"
```

### OIDC Package

Use `@vercel/functions/oidc`, NOT the deprecated `@vercel/oidc`:

```typescript
// ❌ Old (deprecated, causes "getToken is not a function")
import { getToken } from '@vercel/oidc'

// ✅ New
import { getVercelOidcToken } from '@vercel/functions/oidc'
```

### Newlines Break WIF

If env vars have trailing newlines, the STS audience string becomes corrupted:
```
"//iam.googleapis.com/projects/123456\n/locations/global..."
```

**Symptoms:**
- Debug endpoint shows `\n` in the `stsAudience` field
- STS exchange fails with "Invalid value for audience"

**Fix:** Re-add each WIF env var using `printf` (not dashboard copy-paste):
```bash
printf "value" | vercel env add GCP_PROJECT_NUMBER production --force
printf "value" | vercel env add GCP_WORKLOAD_IDENTITY_POOL_ID production --force
printf "value" | vercel env add GCP_WORKLOAD_IDENTITY_PROVIDER_ID production --force
printf "value" | vercel env add GCP_SERVICE_ACCOUNT_EMAIL production --force
```

---

## Domains

### Add Custom Domain

```bash
vercel domains add example.com
```

### DNS Configuration

| Type | Name | Value |
|------|------|-------|
| A | @ | 76.76.21.21 |
| CNAME | www | cname.vercel-dns.com |

### SSL

Automatic via Let's Encrypt. No configuration needed.

---

## Quick Reference

```bash
# Deploy to production
vercel --prod

# Add env var (use printf!)
printf "value" | vercel env add KEY production

# Pull env vars locally
vercel env pull .env.local

# View logs
vercel logs <url>

# Rollback
vercel promote <previous-url>

# Clear cache and redeploy
vercel --force --prod
```

---

## Resources

- [Vercel Docs](https://vercel.com/docs)
- [vercel.json Reference](https://vercel.com/docs/projects/project-configuration)
- [Next.js on Vercel](https://vercel.com/docs/frameworks/nextjs)
- [Environment Variables](https://vercel.com/docs/projects/environment-variables)
