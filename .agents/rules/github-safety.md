# GitHub & Repository Safety Rules

These rules govern file operations and Git hygiene in this repository. For curriculum immutability and one-phase-at-a-time boundaries, follow `AGENTS.md` as the authoritative source.

## Inspection & Preservation

- Always inspect `git status`, active branch, and diffs (including untracked files) before modifying files or preparing commits.
- Preserve existing work: never overwrite, delete, or reorganize existing learner notes, labs, or checkpoint feedback without explicit confirmation.
- Avoid accidental destructive Git operations (such as hard resets, force pushes, or discarding unstaged changes) unless explicitly requested.

## Secrets & Review Discipline

- Never commit secrets, API keys, passwords, tokens, connection strings with credentials, or `.env` files.
- Review both staged and unstaged changes (and untracked files) to keep diffs clean, minimal, and scoped to the current task.
- **Never push unreviewed changes directly to `main`.** Wait for learner review and explicit confirmation before pushing.
- Clearly summarize every file created, modified, or deleted after each change.
