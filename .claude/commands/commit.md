---
description: Stage changes, generate commit message, and commit
allowed-tools: Bash, Read
---

Generate commit message(s) and commit the changes.

## Instructions

1. Check if there are staged files (`git diff --cached --name-only`)
2. If files are already staged → leave unstaged files alone, commit only what's staged
3. If nothing is staged → review ALL uncommitted changes (`git diff`, `git status`)
4. **Group changes into logical chunks** — each commit should represent one coherent change:
   - Separate concerns: e.g., a bug fix and a docs update are two commits, not one
   - Group by purpose: files that work together for the same feature/fix go in one commit
   - Tests belong with the code they test in the same commit
   - Config/CI changes that support a feature can be grouped with that feature OR split out if they're independently meaningful
   - When in doubt, prefer smaller, more focused commits over large ones
5. For each logical chunk:
   a. Stage the relevant files (`git add <specific files>`)
   b. Analyze the staged changes (`git diff --cached`)
   c. Generate a conventional commit message
   d. Display the commit message in chat
   e. Commit with the generated message (`git commit -m "..."`)
6. If all changes are tightly related and form a single logical unit, a single commit is fine — don't split artificially

## Output format
```
<type>(<scope>): <concise summary>

- [bullet points if needed]
```

Use conventional commit types: `feat`, `fix`, `refactor`, `docs`, `test`, `chore`, `style`, `perf`

Keep the summary under 72 characters.