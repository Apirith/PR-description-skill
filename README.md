# 🧠 GitHub PR Description Skill for Claude

> A plug-and-play Claude skill that generates polished, structured Pull Request descriptions in seconds — from git diffs, commit logs, branch names, or plain English.

---

## Quick Demo

**Input** — paste a diff and say `Write a PR description for this`

**Output** — reviewer-ready in seconds:

---

feat(auth): add OAuth2 login with Google

## Summary
Adds Google as an OAuth2 login provider. Introduces a callback handler that validates CSRF state, exchanges the authorization code for tokens, and creates a user session. No changes to the existing email/password flow.

## Changes

✨ Feature
- Add Google OAuth2 provider config to `providers.ts`
- Add `handleGoogleCallback` to validate state and exchange code for tokens

## Testing
- Manually tested full OAuth flow with a test Google account in dev
- Verified CSRF state mismatch returns 403

## Checklist
- [ ] Tests added/updated
- [ ] No breaking changes
- [ ] Self-reviewed before submitting

---

## What It Does

Writing good PR descriptions is tedious. This skill eliminates the blank-page problem by generating a complete, reviewer-ready PR description from whatever you have on hand — a raw diff, a `git log`, a branch name, or even just a quick summary of what you changed.

It adapts automatically to context: lean and fast for solo hotfixes, thorough and structured for team feature branches going into review.

**Output includes:**
- A Conventional Commits-style PR title
- Summary, Changes, Testing, Reviewer Notes, and Checklist sections
- Emoji-grouped change categories for scannability
- `[TODO]` placeholders when input is thin (e.g. branch name only)

---

## Installation

### Claude Code (recommended)

**Global install** — available in every project:

```bash
git clone https://github.com/Apirith/PR-description-skill.git
cp -r PR-description-skill/github-pr-description ~/.claude/skills/
```

**Project install** — scoped to one repo:

```bash
# from your project root
mkdir -p .claude/skills
cp -r path/to/PR-description-skill/github-pr-description .claude/skills/
```

Your skills directory should look like this:

```
.claude/skills/
└── github-pr-description/
    ├── SKILL.md
    └── references/
        └── examples.md
```

Reload with `/skills` in Claude Code, or restart Claude Code. No further configuration needed.

---

### Claude Desktop

| Setup | Skills directory |
|---|---|
| Mac | `~/Library/Application Support/Claude/skills/` |
| Windows | `%APPDATA%\Claude\skills\` |

```bash
# Mac / Linux
cp -r github-pr-description ~/Library/Application\ Support/Claude/skills/

# Windows (PowerShell)
Copy-Item -Recurse github-pr-description "$env:APPDATA\Claude\skills\"
```

Quit and reopen Claude Desktop. The skill loads at startup.

---

## Usage

### Claude Code — slash command

```
/pr-description
```

Claude reads the current repo's diff automatically. No paste needed.

---

### From a git diff

```bash
git diff main...your-feature-branch
```

Paste the output and say:

```
Write a PR description for this diff.
```

---

### From a commit log

```bash
git log --oneline main..HEAD
```

Paste the output and say:

```
Generate a PR description from these commits.
```

---

### From a branch name only

```
Write a PR description for the branch: fix/retry-failed-jobs
```

Claude infers intent from the branch name and generates a structured template with `[TODO]` placeholders for anything it can't fill automatically.

---

### From plain English

```
I refactored the database query layer — removed N+1 queries on the user list
endpoint and added an index on created_at. No behavior change for users.
```

No git commands needed.

---

### Improving an existing draft

```
Clean up and restructure this PR description.
```

Paste your rough draft alongside — Claude will restructure and fill gaps.

---

## Output Format

Every generated PR description follows this structure:

---

`<type>(<scope>): <short description>`

## Summary
A 2–4 sentence overview of what changed and why.

## Changes

✨ Feature
- Add X to Y

🐛 Fix
- Resolve Z when condition W

♻️ Refactor
- Extract A into standalone module

## Testing
What was tested, how, and what reviewers should verify.

## Notes for Reviewer
Optional context, caveats, or areas that need focused attention.

## Checklist
- [ ] Tests added / updated
- [ ] Docs updated (if applicable)
- [ ] No breaking changes (or breaking changes documented below)
- [ ] Self-reviewed before submitting

---

## Change Categories

| Emoji | Label | When to use |
|---|---|---|
| ✨ | Feature | New user-facing behavior |
| 🐛 | Fix | Bug fixes |
| ♻️ | Refactor | Code changes with no behavior change |
| 🧹 | Chore | Deps, build, CI, config |
| 📝 | Docs | Documentation only |
| ⚠️ | Breaking | Changes requiring action from consumers |

Only categories with actual changes are included — no empty sections.

---

## PR Title Format

Titles follow [Conventional Commits](https://www.conventionalcommits.org/) spec:

```
<type>(<scope>): <short description>
```

**Examples:**

```
feat(auth): add OAuth2 login with Google
fix(payments): handle timeout on Stripe webhook
refactor(api): extract rate limiting into middleware
chore(deps): upgrade Next.js to 14.2
docs(readme): add installation instructions
```

If no scope is identifiable, it's omitted: `fix: correct null check in user lookup`

---

## Customisation

### Use your own PR template

Paste your `.github/pull_request_template.md` alongside your diff and say:

```
Fill in our PR template using this diff.
```

The skill respects your template's structure exactly.

### Adjust tone or length

- `"Make this more concise"` — strips it down for a quick fix
- `"Expand the reviewer notes"` — adds more context for complex changes
- `"Add a rollback section"` — useful for risky deploys
- `"Write this for a non-technical reviewer"` — plain language, no jargon

---

## Worked Examples

Full before/after examples in [`references/examples.md`](references/examples.md):

- Feature PR from a git diff
- Hotfix from a commit log
- Refactor with no behavior change
- Branch-name-only with placeholders
- Improving a rough draft
- Multi-file monorepo PR

---

## File Structure

```
github-pr-description/
├── SKILL.md                  # Core skill definition (loaded by Claude)
├── README.md                 # This file
└── references/
    └── examples.md           # Complete worked examples
```

---

## Requirements

- Claude Code or Claude Desktop (any recent version)
- No API keys, no external dependencies, no setup beyond copying the folder

---

## Contributing

Found a case the skill handles poorly? Open an issue with:
1. The input you provided
2. What Claude generated
3. What you expected instead

PRs welcome — especially new example types in `references/examples.md`.

---

## License

MIT — use it, fork it, extend it.

---

*Built with Claude · Made by [Apirith](https://github.com/Apirith)*
