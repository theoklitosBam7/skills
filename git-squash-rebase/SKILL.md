---
name: git-squash-rebase
description: >
  Squash-rebase stacked branches after a parent PR was squash-merged. Use when
  updating a child branch built on a merged parent, fixing duplicate parent
  commits after rebasing, or repairing a stacked PR workflow that is replaying
  old branch commits.
---

# Git Squash-Rebase

## Mental model

After a parent branch is **squash-merged**, `main` contains a new squash commit,
not the parent's original commits. A child branch still based on those original
commits must be moved with:

```text
git rebase --onto <new-base> <old-parent-tip> <child-branch>
```

Read it as: move `<child-branch>` onto `<new-base>`, bringing only commits after
`<old-parent-tip>`.

## Workflow

### 1. Establish the boundary

Identify, without guessing:

- `<new-base>`: usually `main`, but confirm the repo's base branch.
- `<parent-branch>`: the branch that was squash-merged.
- `<child-branch>`: the branch to update.
- `<old-parent-tip>`: the parent tip before the squash merge, or the commit where
  the child stopped being parent work and started being child work.
- Working tree state: clean, dirty tracked files, and untracked files.
- Remote state: whether the child branch was pushed.

Useful probes:

```bash
git status --short
git branch --show-current
git rev-parse <child-branch>
git rev-parse <parent-branch>  # only valid if it still points at the old parent tip
```

If the parent branch was deleted, moved, rebased, or recreated, ask for the old
parent tip SHA or recover it from reflog / PR history before continuing.

**Complete when:** the base, child, old parent tip, working tree state, and remote
push status are all known.

### 2. Protect local work

If the child branch has uncommitted changes, stash tracked and untracked files:

```bash
git stash push -u -m "before squash-rebase of <child-branch>"
```

Skip this when the working tree is clean.

**Complete when:** no uncommitted work can be overwritten by the rebase, or the
user explicitly chooses another safe path such as committing it.

### 3. Update the new base

```bash
git fetch origin
git switch <new-base>
git pull --ff-only origin <new-base>
```

**Complete when:** the local `<new-base>` is up to date with its upstream.

### 4. Move the child with `--onto`

```bash
git rebase --onto <new-base> <old-parent-tip> <child-branch>
```

This replays:

```text
<old-parent-tip>..<child-branch>
```

onto `<new-base>`. If the child has no commits of its own, it simply moves to the
new base.

**Complete when:** the rebase finishes, or it stops only on conflicts from the
child's own commits against the updated base.

### 5. Resolve or abort

For expected conflicts:

```bash
git status
# edit conflicted files
git add <resolved-file>
git rebase --continue
```

Abort if Git is replaying parent commits, the boundary is wrong, or the conflict
set is clearly from already-merged parent work:

```bash
git rebase --abort
```

**Complete when:** the rebase finishes with the intended child commits only, or
it is aborted before rewriting the wrong history further.

### 6. Restore stashed work

If step 2 created a stash:

```bash
git switch <child-branch>
git stash pop
```

Resolve stash-pop conflicts as ordinary working tree conflicts.

**Complete when:** any stashed work is restored or intentionally left in the
stash with the stash reference noted.

### 7. Verify before pushing

```bash
# Should list only the child branch's own commits
git log --oneline <new-base>..HEAD

# Should show only intentional child differences
git diff --stat <new-base>...HEAD

# Optional boundary check
git log --oneline <old-parent-tip>..<child-branch>
```

The old parent commit SHAs must not appear in `<new-base>..HEAD`. The squash
commit from the parent PR is already on `<new-base>`, so it should not appear
there either.

**Complete when:** every commit and diff shown is intended child work.

### 8. Push with lease protection

If the child branch was already pushed:

```bash
git push --force-with-lease origin <child-branch>
```

Do not use plain `--force` unless the user explicitly asks for it and accepts the
risk.

**Complete when:** the remote child branch points at the verified rewritten
history, or the user chooses not to push yet.

## Deeper stacks

Work bottom-up. Before moving a branch, save the old tip needed by its child:

```bash
git rev-parse F2  # save as <old-F2-tip>
```

After F2 is moved, rebase F3 onto the new F2 while excluding the old F2 tip:

```bash
git rebase --onto F2 <old-F2-tip> F3
```

Repeat for each descendant. Each step is complete only when the next child's
exclude boundary has been recorded before its parent is rewritten.

## Pitfalls

- `git rebase --onto <new-base> <parent-branch> <child-branch>` is safe only if
  `<parent-branch>` still points at the old parent tip.
- A naive `git rebase <new-base>` may replay parent commits. Abort it and restart
  with the correct `--onto` boundary.
- Dirty worktrees need a stash or commit before rebasing; include `-u` when
  untracked files matter.
- Multiple children require saving old tips before rewriting their parents.
