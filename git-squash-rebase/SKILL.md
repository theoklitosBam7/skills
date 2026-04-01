---
name: git-squash-rebase
description: >
  Guides the user through safely rebasing a stacked feature branch (F2) onto
  main after a dependent branch (F1) was squash-merged. Use this skill whenever
  the user mentions: stacked branches, a branch built on top of another branch,
  squash-merge conflicts on rebase, duplicate commits after rebase, "rebase onto
  main after PR merged", "my rebase is creating conflicts from the old branch",
  "how do I update F2 after F1 was merged", or any variation of rebasing a
  child branch after the parent was squash-merged. Also trigger when the user
  says things like "I built a branch on top of another branch and now I need to
  update it" or "my stacked PR workflow is broken".
---

# Git Squash-Rebase: Stacked Branch Workflow

## The Core Problem

When a parent branch (F1) is **squash-merged** into main, Git creates a brand
new commit with no SHA relationship to the original F1 commits. If a child
branch (F2) was built on top of F1, a naive `git rebase main` will:

1. See all F1 commits as "not yet in main"
2. Replay them again → **duplicate content + conflicts**

The fix is `git rebase --onto`, which lets you specify exactly which commits to
skip.

---

## Step 0 — Understand the User's State

Before running any commands, ask the user to confirm:

- [ ] **Which branch is F1?** (the one that was squash-merged)
- [ ] **Which branch is F2?** (the stacked child branch to update)
- [ ] **Does F2 have uncommitted working changes?** (stash needed if yes)
- [ ] **Does F2 have commits of its own?** (or is it only working changes?)

This affects which steps apply. All four steps below are safe to run regardless
— just clarify before proceeding.

---

## The Workflow

### Step 1 — Stash uncommitted work (if any)

If F2 has uncommitted changes (modified/untracked files), stash them first:

```bash
git stash
```

Skip this step if the working tree is clean.

---

### Step 2 — Update main

```bash
git checkout main
git pull origin main
```

This pulls the squashed F1 commit into the local main.

---

### Step 3 — Rebase F2 using `--onto`

```bash
git rebase --onto main <F1-branch-name> <F2-branch-name>
```

**What this means:**
- `--onto main` → land the result on top of main
- `<F1-branch-name>` → exclude everything up to (and including) F1's tip
- `<F2-branch-name>` → operate on this branch

This replays **only F2's own commits** on top of main, skipping all F1 commits.

> **No commits on F2 yet?** The command still works — it just moves the branch
> pointer to main's tip with nothing to replay. Clean and instant.

---

### Step 4 — Restore stashed changes (if stashed)

```bash
git checkout <F2-branch-name>
git stash pop
```

Skip this step if nothing was stashed in Step 1.

---

## Conflict Resolution (if conflicts occur)

Conflicts during `--onto` rebase mean F2's own commits touch code that changed
in main independently of F1. Resolve normally:

```bash
# For each conflicting file:
git add <resolved-file>
git rebase --continue

# To abort and start over:
git rebase --abort
```

---

## Verifying the Result

After the rebase, confirm the history looks correct:

```bash
# Check F2's log — should show main's squashed F1 commit, then F2's commits
git log --oneline main..HEAD

# Confirm no F1 commit SHAs appear
git log --oneline <F1-branch-name> | head -5
# None of these should appear in F2's log above
```

---

## Command Reference Cheatsheet

| Scenario | Command |
|---|---|
| F2 has uncommitted changes | `git stash` first |
| Update main | `git checkout main && git pull origin main` |
| Rebase F2 onto main, skipping F1 | `git rebase --onto main F1 F2` |
| Restore stashed changes | `git checkout F2 && git stash pop` |
| Abort a broken rebase | `git rebase --abort` |

---

## Mental Model

```
--onto <target> <exclude-up-to> <branch>
         │              │            │
         │              │            └─ branch to rebase
         │              └─ "skip everything that was in here"
         └─ "land the result here"
```

Think of it as: *"Move `<branch>` to `<target>`, but only bring commits that
were NOT already in `<exclude-up-to>`."*

---

## Deeper Stacks (F3 on top of F2, etc.)

If there is a third branch F3 stacked on F2, repeat the pattern after
updating F2:

```bash
git rebase --onto F2 <old-F2-tip> F3
```

Where `<old-F2-tip>` is the SHA of F2's previous tip before the rebase
(capture it with `git rev-parse F2` before starting).

For deeper stacks, work from the bottom up: update F2 first, then F3, then F4,
etc.
