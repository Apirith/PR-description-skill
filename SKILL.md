# GitHub PR Description Skill

## Trigger

Activate this skill whenever the user:
- Pastes a `git diff` or `git log` output and asks for a PR description
- Mentions writing, generating, drafting, or improving a Pull Request description
- Provides a branch name and asks for a PR description
- Describes code changes in plain English and wants them formatted as a PR
- Runs `/pr-description` in Claude Code (read the diff automatically via `git diff main` or `git diff --staged`)

---

## Input Handling

### Claude Code (slash command)
When invoked as `/pr-description` with no user-provided input, run the following in order until one succeeds:
1. `git diff main`  full diff against main
2. `git diff --staged`  staged changes only
3. `git diff HEAD~1`  last commit
4. `git log --oneline main..HEAD`  commit messages only

If none return output, ask the user: "No diff found,  are you on a feature branch, or would you like to paste a diff directly?"

### Input confidence levels
Adapt output depth to what you have:

| Input type | Depth |
|---|---|
| Full diff with commit messages | Fill every section thoroughly  no TODOs |
| Diff only (no commit messages) | Infer intent from file paths and change content |
| Commit log only | Fill what commit messages reveal; use `[TODO]` for testing details |
| Branch name only | Infer from naming conventions; use `[TODO]` for all content sections |
| Plain English description | Trust the user's framing; structure and categorize |
| Rough draft | Preserve intent; restructure, fill gaps, apply formatting |

Never invent specific details (test results, ticket numbers, metrics). Use `[TODO: ...]` with a specific prompt rather than a vague placeholder.

### Custom templates
If the user provides a `.github/pull_request_template.md` or any custom template structure, fill that template exactly;  do not substitute the default format.

---

## Output Format

Always produce output in this exact structure. Omit sections with zero content, but never omit the title or Summary.

---

`<type>(<scope>): <short description>`

## Summary
2–4 sentences covering: what changed, why it changed, and any notable impact or scope.

## Changes

- <bullet> ✨ Feature

- <bullet> 🐛 Fix

- <bullet> ♻️ Refactor

- <bullet>🧹 Chore

- <bullet>📝 Docs

- <bullet>⚠️ Breaking


## Testing
Describe what was tested, the method (unit, manual, integration), and what reviewers should verify.

## Notes for Reviewer
Flag tricky areas, design decisions, known limitations, deployment dependencies, or anything warranting extra scrutiny.

## Checklist
- [ ] Tests added/updated
- [ ] Docs updated (if applicable)
- [ ] No breaking changes (or breaking changes documented below)
- [ ] Self-reviewed before submitting

---

## PR Title Rules

Format: `<type>(<scope>): <short description>`

- **type**: `feat`, `fix`, `refactor`, `chore`, `docs`, `test`, `perf`, `ci`
- **scope**: the module, route, component, or package affected;  omit if not clearly identifiable
- **short description**: imperative mood, lowercase, no period, ≤72 chars total title length

### Scope detection heuristics

| Signal | Inferred scope |
|---|---|
| Files in `src/auth/` | `auth` |
| Files in `packages/api/` | `api` |
| Changes to `*.test.ts` only | `test` |
| Changes to `package.json` / lock files | `deps` |
| Changes to `.github/` or CI config | `ci` |
| Changes across many unrelated modules | omit scope |

### Conventional Commits detection
If commit messages already follow Conventional Commits (`feat:`, `fix:`, etc.), derive the PR title type directly from the most significant commit type present. Do not override a user-supplied commit type.

**Examples:**

```
feat(auth): add OAuth2 login with Google
fix(payments): handle timeout on Stripe webhook
refactor(api): extract rate limiting into middleware
chore(deps): upgrade Next.js to 14.2
fix: correct null check in user lookup
```

---

## Change Category Rules

| Emoji | Label | Use when… |
|---|---|---|
| ✨ | Feature | New user-facing functionality is added |
| 🐛 | Fix | A bug is corrected |
| ♻️ | Refactor | Code structure changes with no user-visible difference |
| 🧹 | Chore | Deps, build scripts, CI config, tooling |
| 📝 | Docs | Documentation or comments only |
| ⚠️ | Breaking | Any change that requires downstream action |

Only include categories with actual changes. Never emit an empty category section.

### Breaking change detection signals
Flag a change as ⚠️ Breaking when the diff shows any of:
- Removed or renamed exported functions, classes, or types
- Changed function signatures (removed parameters, changed types)
- Removed API endpoints or changed response shapes
- Database schema changes that are not backwards-compatible
- Changed environment variable names or config keys that deployments depend on

---

## Monorepo Handling

When the diff spans multiple packages:
- Use the most-affected package as the scope (e.g. `api`, `web`, `shared`)
- If changes are evenly spread, omit scope
- Group bullets by package when it aids clarity: `packages/api` changes first, then `packages/web`, then shared/config
- Note cross-package dependencies in Notes for Reviewer if a deployment order matters

---

## Tone and Style

- **Professional but direct**  no filler words, no marketing language
- **Bullets over paragraphs** in the Changes section
- **Active voice** in bullets: "Add X", "Remove Y", "Fix Z"  not "X was added"
- **Reviewer empathy**: flag anything non-obvious, risky, or that warrants extra scrutiny
- **Honest about gaps**: use `[TODO: describe what was tested]` rather than inventing details
- `[TODO]` placeholders must include a specific prompt  never just `[TODO]` alone

---

## Post-Generation Adjustments

After generating, accept and apply these follow-up requests without re-generating from scratch:

- `"Make this more concise"`  trim prose, shorten bullets
- `"Expand the reviewer notes"`  add context, edge cases, risk areas
- `"Add a rollback section"`,  append rollback steps after the Checklist
- `"Fill in the TODOs"`  ask the user for the missing information, then fill in
- `"Write this for a non-technical reviewer"`  plain language, avoid jargon, explain impact over implementation
- `"Add a deployment checklist"`  append environment variable changes, migration steps, feature flag instructions
