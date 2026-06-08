---
name: git-squash-rebase
description: >
  Safely rebase a stacked child branch onto main after its parent branch was
  squash-merged, avoiding duplicate parent commits and rebase conflicts caused
  by replaying already-merged work. Use when the user mentions stacked branches,
  dependent PRs, a branch built on another branch, squash-merge rebase problems,
  duplicate commits after rebasing, updating a child branch after the parent PR
  merged, or phrases like "rebase onto main after PR merged", "my rebase is
  replaying the old branch", "how do I update F2 after F1 was squash-merged",
  or "my stacked PR workflow is broken".
---

# Git Squash-Rebase: Stacked Branch Workflow

## Core problem

When a parent branch (F1) is **squash-merged** into `main`, Git creates a new
commit on `main` with no SHA relationship to the original F1 commits. If a child
branch (F2) was built on top of F1, a naive `git rebase main` may replay the old
F1 commits again, causing duplicate commits and avoidable conflicts.

Use `git rebase --onto` to say: replay only F2's own commits onto updated
`main`, excluding the commits already reachable from the old F1 tip.

## Preconditions to establish first

Before giving or running commands, identify:

- **Base branch**: usually `main` (confirm if the repo uses `master`, `develop`, etc.)
- **F1 / parent branch**: the branch that was squash-merged
- **F2 / child branch**: the branch to update
- **Old F1 tip**: the commit at the top of F1 before/when F2 branched from it
- **Working tree state**: whether F2 has uncommitted or untracked changes
- **Remote state**: whether F2 has already been pushed and will need a force-with-lease push

Do not guess branch names. If the F1 branch was deleted or moved, ask the user
for the old F1 tip SHA, recover it from reflog, or inspect the PR history.

## Safe workflow

### 1. Save current state

```bash
git status --short
git branch --show-current
git rev-parse <F2-branch-name>
git rev-parse <F1-branch-name>  # or the old F1 tip SHA, if branch is gone/moved
```

If there are uncommitted changes on F2, stash them, including untracked files:

```bash
git stash push -u -m "before stacked rebase of <F2-branch-name>"
```

Skip the stash when the working tree is clean.

### 2. Update the base branch

Prefer `fetch` plus an explicit local update so the command works even when the
current branch is not `main`:

```bash
git fetch origin
git switch main
git pull --ff-only origin main
```

Replace `main` with the confirmed base branch if needed.

### 3. Rebase F2 with `--onto`

```bash
git rebase --onto main <old-F1-tip-or-F1-branch-name> <F2-branch-name>
```

Meaning:

- `main` = new base to land on
- `<old-F1-tip-or-F1-branch-name>` = exclude commits reachable from the old parent tip
- `<F2-branch-name>` = branch to rewrite

This replays commits in the range:

```text
<old-F1-tip-or-F1-branch-name>..<F2-branch-name>
```

onto `main`.

If F2 has no commits of its own, the branch simply moves to `main` with nothing
to replay.

### 4. Resolve conflicts, if any

Conflicts now should be from F2's own changes against updated `main`, not from
replaying all of F1. Resolve normally:

```bash
git status
# edit conflicted files
git add <resolved-file>
git rebase --continue
```

Abort if the wrong commits are being replayed or the chosen F1 boundary is wrong:

```bash
git rebase --abort
```

### 5. Restore stashed work

```bash
git switch <F2-branch-name>
git stash pop
```

Skip when nothing was stashed. Resolve stash-pop conflicts like ordinary working
tree conflicts.

### 6. Verify before pushing

```bash
# Should list only F2's own commits after main
git log --oneline main..HEAD

# Should be empty or only intentional differences from F2
git diff --stat main...HEAD

# Compare with the intended replay range if unsure
git log --oneline <old-F1-tip-or-F1-branch-name>..<F2-branch-name>
```

Important: `git log main..HEAD` should **not** include old F1 commit SHAs. The
squashed F1 commit is already on `main`, so it also will not appear in
`main..HEAD`.

### 7. Push the rewritten child branch

If F2 was already pushed, update the remote with lease protection:

```bash
git push --force-with-lease origin <F2-branch-name>
```

Do not use plain `--force` unless explicitly requested and understood.

## Common pitfalls

- **Using the wrong F1 boundary**: `--onto main F1 F2` only works if `F1` still
  points at the old parent tip. If F1 was deleted, recreated, rebased, or moved,
  use the exact old tip SHA instead.
- **Running naive `git rebase main` first**: abort if still in progress, then use
  `--onto` with the correct old parent boundary.
- **Dirty working tree**: stash or commit before rebasing. Include `-u` if there
  are untracked files.
- **Multiple children**: record old tips before rewriting branches.

## Deeper stacks

Work bottom-up. Before rebasing F2, capture the old tips of any descendants:

```bash
git rev-parse F2  # save as <old-F2-tip>
```

After F2 is updated, rebase F3 onto the new F2 while excluding the old F2 tip:

```bash
git rebase --onto F2 <old-F2-tip> F3
```

Repeat for F4, F5, etc.

## Mental model

```text
git rebase --onto <new-base> <exclude-up-to> <branch-to-move>
```

Read it as: move `<branch-to-move>` onto `<new-base>`, bringing only commits
after `<exclude-up-to>`.
