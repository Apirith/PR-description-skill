# Worked Examples

---

## Example 1 — Feature PR from a git diff

### Input

```bash
git diff main...feat/oauth-google
```

```diff
diff --git a/src/auth/providers.ts b/src/auth/providers.ts
index 3a1f2c0..8b4d9e1 100644
--- a/src/auth/providers.ts
+++ b/src/auth/providers.ts
@@ -1,5 +1,6 @@
 export const providers = [
   { id: 'email', name: 'Email / Password' },
+  { id: 'google', name: 'Google', clientId: process.env.GOOGLE_CLIENT_ID },
 ];

diff --git a/src/auth/callback.ts b/src/auth/callback.ts
new file mode 100644
index 0000000..c2d8f3a
--- /dev/null
+++ b/src/auth/callback.ts
@@ -0,0 +1,28 @@
+import { exchangeCode } from './oauth';
+
+export async function handleGoogleCallback(req, res) {
+  const { code, state } = req.query;
+  if (!state || state !== req.session.oauthState) {
+    return res.status(403).json({ error: 'Invalid state' });
+  }
+  const tokens = await exchangeCode(code);
+  req.session.user = await getUserFromTokens(tokens);
+  res.redirect('/dashboard');
+}
```

**User prompt:** `Write a PR description for this diff.`

### Output

```
feat(auth): add OAuth2 login with Google

## Summary
Adds Google as an OAuth2 login provider. Introduces a callback handler that
validates CSRF state, exchanges the authorization code for tokens, and creates
a user session. No changes to the existing email/password flow.

## Changes

✨ Feature
- Add Google OAuth2 provider config to `providers.ts`
- Add `handleGoogleCallback` to validate state parameter and exchange code for tokens

## Testing
- Manually tested full OAuth flow with a test Google account in dev environment
- Verified CSRF state mismatch returns 403
- Confirmed existing email/password login unaffected

## Notes for Reviewer
- `GOOGLE_CLIENT_ID` must be set in `.env` — not included in this PR; see the deployment runbook
- State validation uses `req.session.oauthState` — ensure session middleware is
  initialized before this route in the middleware chain

## Checklist
- [ ] Tests added / updated
- [ ] Docs updated (if applicable)
- [ ] No breaking changes (or breaking changes documented below)
- [ ] Self-reviewed before submitting
```

---

## Example 2 — Hotfix from a commit log

### Input

```bash
git log --oneline main..HEAD
```

```
a3f91bc fix: handle null user in session middleware
```

**User prompt:** `Generate a PR description from these commits.`

### Output

```
fix(auth): handle null user in session middleware

## Summary
Patches a crash that occurred when session middleware encountered a request
with no authenticated user. The missing null check caused an unhandled exception
on unauthenticated routes, returning a 500 instead of a 401.

## Changes

🐛 Fix
- Add null guard for `req.session.user` in session middleware to prevent
  unhandled exception on unauthenticated requests

## Testing
- Verified unauthenticated requests to protected routes return 401 instead of 500
- Confirmed authenticated-user flow is unchanged

## Notes for Reviewer
Minimal change — single null check. Risk is low but worth confirming the guard
condition covers all entry paths into the middleware.

## Checklist
- [ ] Tests added / updated
- [ ] Docs updated (if applicable)
- [ ] No breaking changes (or breaking changes documented below)
- [ ] Self-reviewed before submitting
```

---

## Example 3 — Refactor with no behavior change

### Input (plain English)

```
I refactored the database query layer — removed N+1 queries on the user list
endpoint and added an index on created_at. No behavior change for users.
```

### Output

```
perf(db): eliminate N+1 queries and index created_at on user list

## Summary
Refactors the user list query to load related records in a single batched query
instead of one query per row. Adds a database index on `created_at` to support
efficient ordering. No user-visible behavior change.

## Changes

♻️ Refactor
- Replace per-row relation queries with a single batched JOIN on the user list endpoint
- Add index on `users.created_at` to support sort-order queries efficiently

## Testing
- Ran the user list endpoint against a 10k-row fixture — query count dropped
  from ~10,001 to 2
- Verified response payload is identical before and after
- No regression in existing user-list tests

## Notes for Reviewer
The index migration must run before deploying the query change — both are in
this PR. If you need to deploy them separately, the query is safe to run without
the index (just slower).

## Checklist
- [ ] Tests added / updated
- [ ] Docs updated (if applicable)
- [ ] No breaking changes (or breaking changes documented below)
- [ ] Self-reviewed before submitting
```

---

## Example 4 — Branch name only (with placeholders)

### Input

```
Write a PR description for the branch: fix/retry-failed-jobs
```

### Output

```
fix(jobs): add retry logic for failed background jobs

## Summary
[TODO: 2–4 sentence description of what was changed and why. What was failing?
What is the retry strategy — exponential backoff, max attempts, dead-letter queue?]

## Changes

🐛 Fix
- [TODO: describe the retry mechanism — e.g., exponential backoff with max 3 attempts]
- [TODO: list related changes — error logging, alerting, config values added]

## Testing
[TODO: describe how retry behavior was tested — unit tests, manual trigger of a
failing job, integration test against a stub queue]

## Notes for Reviewer
[TODO: any non-obvious design decisions, edge cases, or areas needing focused review]

## Checklist
- [ ] Tests added / updated
- [ ] Docs updated (if applicable)
- [ ] No breaking changes (or breaking changes documented below)
- [ ] Self-reviewed before submitting
```

---

## Example 5 — Improving an existing draft

### Input (rough draft)

```
## What
added dark mode

## Why
users asked for it

## Testing
i tested it
```

**User prompt:** `Clean up and restructure this PR description.`

### Output

```
feat(ui): add dark mode theme

## Summary
Adds a dark mode option to the application in response to the most-requested
item in the Q1 feedback survey. Users can toggle between light and dark themes
from the settings page; the preference persists across sessions via localStorage.

## Changes

✨ Feature
- Add dark mode theme tokens to the design system
- Add theme toggle control to the user settings page
- Persist theme preference to `localStorage` on change

## Testing
[TODO: describe what was tested — e.g., verified toggle in Chrome/Firefox/Safari,
confirmed preference persists on reload, checked contrast ratios meet WCAG AA]

## Notes for Reviewer
[TODO: any components that were tricky to theme, or areas still needing a follow-up pass]

## Checklist
- [ ] Tests added / updated
- [ ] Docs updated (if applicable)
- [ ] No breaking changes (or breaking changes documented below)
- [ ] Self-reviewed before submitting
```

---

## Example 6 — Multi-file monorepo PR

### Input

```bash
git diff main...feat/rate-limiting
```

```diff
diff --git a/packages/api/src/middleware/rateLimiter.ts b/packages/api/src/middleware/rateLimiter.ts
new file mode 100644
+++ b/packages/api/src/middleware/rateLimiter.ts
@@ -0,0 +1,34 @@
+import rateLimit from 'express-rate-limit';
+
+export const apiLimiter = rateLimit({
+  windowMs: 15 * 60 * 1000,
+  max: 100,
+  standardHeaders: true,
+  legacyHeaders: false,
+  message: { error: 'Too many requests, please try again later.' },
+});

diff --git a/packages/api/src/routes/index.ts b/packages/api/src/routes/index.ts
@@ -1,6 +1,8 @@
+import { apiLimiter } from '../middleware/rateLimiter';
 import userRoutes from './users';

+router.use('/api', apiLimiter);
 router.use('/api/users', userRoutes);

diff --git a/packages/api/src/config.ts b/packages/api/src/config.ts
@@ -4,5 +4,8 @@
 export const config = {
   port: process.env.PORT || 3000,
+  rateLimit: {
+    windowMs: Number(process.env.RATE_LIMIT_WINDOW_MS) || 900000,
+    max: Number(process.env.RATE_LIMIT_MAX) || 100,
+  },
 };

diff --git a/packages/docs/rate-limiting.md b/packages/docs/rate-limiting.md
new file mode 100644
+++ b/packages/docs/rate-limiting.md
@@ -0,0 +1,12 @@
+# Rate Limiting
+Requests are limited to 100 per 15-minute window per IP...
```

**User prompt:** `Write a PR description for this diff.`

### Output

```
feat(api): add rate limiting to public API endpoints

## Summary
Adds IP-based rate limiting to all `/api` routes to prevent abuse and reduce
infrastructure costs. The limit defaults to 100 requests per 15-minute window
and is fully configurable via environment variables. Includes documentation
update in `packages/docs`.

## Changes

✨ Feature
- Add `apiLimiter` middleware using `express-rate-limit` with standard headers
- Apply limiter globally to all `/api` routes in `routes/index.ts`

🧹 Chore
- Expose `rateLimit` config block in `config.ts` driven by env vars
  (`RATE_LIMIT_WINDOW_MS`, `RATE_LIMIT_MAX`) with sensible defaults

📝 Docs
- Add `packages/docs/rate-limiting.md` explaining the limit, headers, and
  how to configure per-environment

## Testing
- Verified 101st request within a 15-minute window returns 429 with correct
  error message
- Confirmed `RateLimit-*` standard headers are present on all API responses
- Load-tested with k6 to confirm limits hold under concurrent traffic

## Notes for Reviewer
- `RATE_LIMIT_WINDOW_MS` and `RATE_LIMIT_MAX` are not yet in the staging `.env`
  — will need to be added before deploy; defaults are safe for production load
- The limiter is applied at the `/api` prefix level, so all current and future
  routes inherit it automatically

## Checklist
- [ ] Tests added / updated
- [ ] Docs updated (if applicable)
- [ ] No breaking changes (or breaking changes documented below)
- [ ] Self-reviewed before submitting
```
