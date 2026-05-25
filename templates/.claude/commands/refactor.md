---
name: refactor
description: Targeted cleanup after a feature — identify drift and apply small, focused surgery
allowed-tools: Bash, Read, Edit, Glob
---

# Refactor

Run after completing a feature or fix, before `/pr`. Keeps the codebase from accumulating entropy.

## Step 1 — Scope the session

```bash
git diff origin/main...HEAD --stat
git log --oneline origin/main..HEAD
```

List only the files changed in this branch. The refactor stays within that scope — don't touch files unrelated to this work.

## Step 2 — Scan for signals

For each changed file, check:

**Size**: files over 300 lines are candidates for splitting by responsibility.

**Duplication**: patterns repeated 3+ times that could be extracted into a shared function.

**Naming**: generic names (`data`, `result`, `handler`, `temp`) that could be more specific.

**Depth**: blocks nested beyond 2 levels — candidates for early return or extraction.

**Dead code**: imports, variables, or functions defined but never used.

Run:
```bash
# Files over 300 lines in the diff scope
git diff origin/main...HEAD --name-only | xargs wc -l 2>/dev/null | sort -rn | head -10
```

## Step 3 — Propose before acting

List findings:

```
Refactor candidates
───────────────────
[file:line] — [issue: what and why it matters]
[file:line] — [issue: what and why it matters]
...
```

Ask: **"Apply all, apply specific ones, or skip?"**

Do not make changes until the user confirms.

## Step 4 — Apply

Each change is atomic:
- One extraction per commit: `refactor(scope): extract [name] from [file]`
- One rename per commit if it touches multiple files
- Never mix refactoring with behavior changes in the same commit

After each change, verify tests still pass:
```bash
# Run project test command from CLAUDE.md
```

## Rules

- Stay within the branch's changed files — no scope creep
- Never change behavior, only structure
- If a refactor would require touching more than 5 unrelated files, flag it as a separate issue instead
- Commit each logical change independently
