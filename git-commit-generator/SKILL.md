---
name: git-commit-generator
description: >
  Generate Conventional Commit messages from staged git changes. Analyzes diffs and produces
  properly formatted commit messages following the Conventional Commits specification.
  USE FOR: writing commit messages, generating conventional commits, crafting git commit
  messages, formatting commits with type(scope): subject pattern.
  DO NOT USE FOR: general git operations, branching, merging, or rebasing.
---

# Git Commit Generator

Generate Conventional Commit messages from staged git changes.

## Workflow

1. Run `git diff --cached --name-status` to identify changed files and their status
2. Run `git diff --cached` to review the actual staged changes
3. Analyze the changes and generate a commit message following the rules below
4. Output ONLY the commit message — no markdown, no code fences, no commentary

## Output Format

```
type(scope): subject

Body paragraph one.
Body paragraph two.

BREAKING CHANGE: description (if applicable)
```

- Blank line between subject and body
- Blank line between body and footers
- Each body line <= 72 chars
- Body bullets start with `- `

## Subject Rules

- Imperative mood (e.g. "add feature", not "added feature" or "adds feature")
- Start lowercase
- No trailing period
- <= 50 characters

## Body Rules

- Always use bullet points (prefixed with `- `) when a body is included
- Each bullet line <= 72 characters
- Imperative mood (e.g. "Update", "Add", "Fix")
- Explain WHAT and WHY, not HOW
- Omit body entirely if the subject is self-explanatory
- Use domain-specific terms; avoid generic phrases like "update files" or "make changes"
- Never mention "staged", "diff", or counts of files/lines

## Scope

- If the user provides a scope, use it as-is
- Otherwise, infer a concise scope from the top-level directories or module changed
- Prefer a broad scope when many files are affected across areas
- Omit scope if unclear or when changes are global

## Commit Types

| Type | Use When |
|------|----------|
| `feat` | New feature or significant enhancement |
| `fix` | Bug fix |
| `docs` | Documentation changes only |
| `style` | Formatting, whitespace, semicolons — no logic change |
| `refactor` | Code restructuring without behavior change |
| `perf` | Performance improvement |
| `build` | Build system or dependency changes |
| `ci` | CI/CD configuration changes |
| `test` | Adding or updating tests |
| `chore` | Maintenance tasks, tooling, non-src changes |
| `revert` | Reverting a previous commit |

If the user provides a commit type, use it as-is. Otherwise, infer the most appropriate type.

## BREAKING CHANGE Footer

Add a `BREAKING CHANGE: <description>` footer when changes remove or rename public APIs, alter expected behavior, or require migration steps from consumers.

## Examples

### Simple change (no body needed)

```
fix(auth): handle expired token redirect
```

### Change with body

```
feat(api): add pagination to user listings

- Accept `page` and `per_page` query parameters
- Return total count in response header
- Default to 20 items per page
```

### Breaking change

```
refactor(client): replace callback API with promises

- Migrate all client methods from callback pattern to async/await
- Remove deprecated callback-based public methods

BREAKING CHANGE: all client methods now return Promises instead of accepting callbacks
```

## Common Pitfalls

- Do NOT wrap the output in markdown code fences
- Do NOT include any preamble or postamble (e.g. "Here is your commit message:")
- Do NOT invent changes not visible in the diff
- Do NOT use past tense — always imperative mood
