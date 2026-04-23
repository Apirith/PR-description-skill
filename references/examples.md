# Worked Examples

---

## Example 1 — Feature PR from a git diff

### Input

```
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

User prompt: `Write a PR description for this diff.`

### Output

```
feat(auth): add OAuth2 login with Google

## Summary
Adds Google as an OAuth2 login provider. Introduces a callback handler that validates CSRF state, exchanges the authorization code for tokens, and creates a user session. No changes to the existing email/password flow.

## Changes

✨ Feature
- Add Google OAuth2 provider config to `providers.ts`
- Add `handleGoogleCallback` to validate state parameter and exchange code for tokens

## Testing
- Manually tested full OAuth flow with a test Google account in the dev environment
- Verified CSRF state mismatch returns 403
- Existing email/password login confirmed unaffected

## Notes for Reviewer
- `GOOGLE_CLIENT_ID` must be set in `.env` — not included in this PR; see the deployment runbook
- State validation uses `req.session.oauthState` — ensure session middleware is initialized before this route

## Checklist
- [ ] Tests added / updated
- [ ] Docs updated (if applicable)
- [ ] No breaking changes (or breaking changes documented below)
- [ ] Self-reviewed before submitting
```

---

## Example 2 — Hotfix from a commit log

### Input

```
git log --oneline main..HEAD
```

```
a3f91bc fix: handle null user in session middleware
```

User prompt: `Generate a PR description from these commits.`

### Output

```
fix(auth): handle null user in session middleware

## Summary
Patches a crash that occurred when the session middleware encountered a request with no authenticated user. The null check was missing, causing an unhandled exception on unauthenticated routes.

## Changes

🐛 Fix
- Add null guard for `req.session.user` in session middleware to prevent unhandled exception on unauthenticated requests

## Testing
- Verified unauthenticated requests to protected routes return 401 instead of 500
- Existing authenticated-user flow unchanged

## Notes for Reviewer
Minimal change — single null check. Risk is low but worth confirming the guard condition covers all entry paths into the middleware.

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
I refactored the database query layer — removed N+1 queries on the user list endpoint
and added an index on created_at. No behavior change for users.
```

### Output

```
perf(db): eliminate N+1 queries and index created_at on user list

## Summary
Refactors the user list query to load related records in a single batched query instead of one query per row. Adds a database index on `created_at` to support efficient ordering. No user-visible behavior change.

## Changes

♻️ Refactor
- Replace per-row relation queries with a single batched `JOIN` on the user list endpoint
- Add index on `users.created_at` to support sort-order queries efficiently

## Testing
- Ran the user list endpoint against a 10k-row fixture; query count dropped from ~10,001 to 2
- Verified response payload is identical before and after
- No regression in existing user-list tests

## Notes for Reviewer
The index migration must run before deploying the query change — both are in this PR. If you need to deploy them separately, the query is safe to run without the index (just slower).

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
[TODO: 2–4 sentence description of what was changed and why. What was failing? What is the retry strategy?]

## Changes

🐛 Fix
- [TODO: describe the retry mechanism added — e.g., exponential backoff, max attempts, dead-letter queue]
- [TODO: list any related changes — error logging, alerting, config values]

## Testing
[TODO: describe how the retry behavior was tested — unit tests, manual trigger of a failing job, integration test]

## Notes for Reviewer
[TODO: any non-obvious design decisions, edge cases, or areas that need focused review]

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

User prompt: `Clean up and restructure this PR description.`

### Output

```
feat(ui): add dark mode theme

## Summary
Adds a dark mode option to the application. Users can toggle between light and dark themes; the preference is persisted across sessions. Responds to the most-requested item in the Q1 feedback survey.

## Changes

✨ Feature
- Add dark mode theme tokens to the design system
- Add theme toggle control to the user settings page
- Persist theme preference to `localStorage`

## Testing
[TODO: describe what was tested — e.g., verified toggle in Chrome/Firefox/Safari, confirmed preference persists on page reload, checked contrast ratios meet WCAG AA]

## Notes for Reviewer
[TODO: any components that were tricky to theme, or areas that still need a follow-up pass]

## Checklist
- [ ] Tests added / updated
- [ ] Docs updated (if applicable)
- [ ] No breaking changes (or breaking changes documented below)
- [ ] Self-reviewed before submitting
```
