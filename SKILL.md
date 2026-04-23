# GitHub PR Description Skill

## Trigger

Activate this skill whenever the user:
- Pastes a `git diff` or `git log` output and asks for a PR description
- Mentions writing, generating, drafting, or improving a Pull Request description
- Provides a branch name and asks for a PR description
- Describes code changes in plain English and wants them formatted as a PR

## Behavior

Generate a complete, reviewer-ready Pull Request description. Adapt depth to the input:
- **Rich input** (full diff, detailed commit log): Fill every section thoroughly.
- **Thin input** (branch name only, one-liner): Fill what you can; use `[TODO: ...]` placeholders for gaps.
- **Existing draft**: Preserve the user's intent; restructure and polish.

If the user provides a `.github/pull_request_template.md` or any custom template, fill that template exactly — do not substitute the default format.

---

## Output Format

Always produce output in this exact structure (omit sections with zero content, but keep all headers that have content):

```
<type>(<scope>): <short description>

## Summary
2–4 sentences covering: what changed, why it changed, and any notable impact.

## Changes
[Group by category using the emoji labels below. Only include categories that have actual changes.]

✨ Feature
- <bullet>

🐛 Fix
- <bullet>

♻️ Refactor
- <bullet>

🧹 Chore
- <bullet>

📝 Docs
- <bullet>

⚠️ Breaking
- <bullet>

## Testing
Describe what was tested, the method (unit tests, manual, integration), and what reviewers should verify.

## Notes for Reviewer
Optional: highlight tricky areas, design decisions, known limitations, or anything that needs focused attention.

## Checklist
- [ ] Tests added / updated
- [ ] Docs updated (if applicable)
- [ ] No breaking changes (or breaking changes documented below)
- [ ] Self-reviewed before submitting
```

---

## PR Title Rules

Format: `<type>(<scope>): <short description>`

- **type**: `feat`, `fix`, `refactor`, `chore`, `docs`, `test`, `perf`, `ci`
- **scope**: the module, route, component, or layer affected — omit if not clearly identifiable
- **short description**: imperative mood, lowercase, no period, ≤72 chars total

Examples:
```
feat(auth): add OAuth2 login with Google
fix(payments): handle timeout on Stripe webhook
refactor(api): extract rate limiting into middleware
chore(deps): upgrade Next.js to 14.2
fix: correct null check in user lookup
```

---

## Change Category Rules

| Emoji | Label    | Use when…                                              |
|-------|----------|--------------------------------------------------------|
| ✨    | Feature  | New user-facing functionality is added                 |
| 🐛    | Fix      | A bug is corrected                                     |
| ♻️    | Refactor | Code structure changes with no user-visible difference |
| 🧹    | Chore    | Deps, build scripts, CI config, tooling                |
| 📝    | Docs     | Documentation or comments only                         |
| ⚠️    | Breaking | Any change that requires downstream action             |

Only include categories with actual changes. Never emit an empty category section.

---

## Tone and Style

- **Professional but direct** — no filler words, no marketing language
- **Bullets over paragraphs** in the Changes section
- **Active voice** in bullets: "Add X", "Remove Y", "Fix Z" — not "X was added"
- **Reviewer empathy**: flag anything non-obvious, risky, or that warrants extra scrutiny
- **Honest about gaps**: use `[TODO: describe what was tested]` rather than inventing details

---

## Post-Generation Adjustments

After generating, accept and apply these follow-up requests without re-generating from scratch:
- `"Make this more concise"` — trim prose, shorten bullets
- `"Expand the reviewer notes"` — add context, edge cases, risk areas
- `"Add a rollback section"` — append rollback steps after the Checklist
- `"Fill in the TODOs"` — ask the user for the missing information, then fill it in
