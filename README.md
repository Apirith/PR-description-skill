# 🧠 GitHub PR Description Skill for Claude

> A plug-and-play Claude skill that generates polished, structured Pull Request descriptions in seconds — from git diffs, commit logs, branch names, or plain English.

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

### Step 1 — Clone or download this repo

```bash
git clone https://github.com/Apirith/github-pr-description-skill.git
cd github-pr-description-skill
```

Or download the ZIP from the green **Code** button above.

---

### Step 2 — Locate your Claude skills directory

Claude looks for skills in a specific directory depending on your setup:

| Setup | Skills directory |
|---|---|
| Claude Desktop (Mac) | `~/Library/Application Support/Claude/skills/` |
| Claude Desktop (Windows) | `%APPDATA%\Claude\skills\` |
| Custom / self-hosted | wherever your `CLAUDE_SKILLS_PATH` env var points |

If the `skills/` folder doesn't exist yet, create it:

```bash
# Mac / Linux
mkdir -p ~/Library/Application\ Support/Claude/skills/

# Windows (PowerShell)
New-Item -ItemType Directory -Path "$env:APPDATA\Claude\skills" -Force
```

---

### Step 3 — Copy the skill into place

```bash
# Mac / Linux
cp -r github-pr-description ~/Library/Application\ Support/Claude/skills/

# Windows (PowerShell)
Copy-Item -Recurse github-pr-description "$env:APPDATA\Claude\skills\"
```

Your skills directory should now look like this:

```
skills/
└── github-pr-description/
    ├── SKILL.md
    └── references/
        └── examples.md
```

---

### Step 4 — Restart Claude

Quit and reopen Claude Desktop. The skill loads at startup — no further configuration needed.

---

## Usage

Once installed, just talk to Claude naturally. The skill triggers automatically when you paste a diff, commit log, or describe your changes.

### From a git diff

```bash
git diff main...your-feature-branch
```

Paste the output into Claude and say:

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

Claude will infer the likely intent from the branch name and generate a structured template with `[TODO]` placeholders for any sections it can't fill automatically.

---

### From plain English

```
I refactored the database query layer — removed N+1 queries on the user list 
endpoint and added an index on created_at. No behavior change for users.
```

No git commands needed. Claude will categorize, structure, and format it.

---

### Improving an existing draft

Paste your rough draft and say:

```
Clean up and restructure this PR description.
```

---

## Output Format

Every generated PR description follows this structure:

```markdown
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
```

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

## Worked Examples

Full before/after examples are in [`references/examples.md`](references/examples.md), covering:

- Feature PR from a git diff
- Hotfix from a commit log
- Refactor with no behavior change
- Branch-name-only input with placeholders

---

## Customisation

### Use your own PR template

If your team has a PR template (`.github/pull_request_template.md`), paste it alongside your diff and tell Claude to fill it in. The skill will respect your template's structure exactly rather than replacing it.

### Adjust tone or length

After generating, just ask:
- `"Make this more concise"` — strips it down for a quick fix
- `"Expand the reviewer notes"` — adds more context for complex changes
- `"Add a rollback section"` — useful for risky deploys

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

- Claude Desktop (any recent version) or access to Claude with skills support
- No API keys, no external dependencies, no setup beyond copying the folder

---

## Contributing

Found a case the skill handles poorly? Open an issue with:
1. The input you provided
2. What Claude generated
3. What you expected instead

PRs welcome — especially for new example types in `references/examples.md`.


