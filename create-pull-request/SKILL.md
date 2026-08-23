---
name: create-pull-request
disable-model-invocation: true
description: Create a GitHub pull request with local git and the GitHub MCP server.
---

# Create a pull request

Create one pull request from the current branch. Keep local git work and GitHub API work separate:

- Use local `git` commands to inspect commits, review diffs, and publish the branch.
- Use the GitHub MCP server for GitHub identity, repository files, existing PRs, and PR creation.
- Ask before committing, stashing, discarding, rebasing, or force-pushing user work.

## 1. Check the repository

Run:

```bash
git status --short --branch
git branch --show-current
git remote get-url origin
git log --oneline -5
```

Parse `origin` as `OWNER/REPO` from either `git@github.com:OWNER/REPO.git` or `https://github.com/OWNER/REPO.git`. Call `github_get_me` before any other GitHub MCP call.

Find the base branch from `refs/remotes/origin/HEAD`. If it is missing, run `git fetch origin --prune`, then use `git remote show origin`.

If the working tree has changes, show them and stop. Ask whether the user will commit them or wants the agent to commit them. Continue only after the changes are committed or the user confirms that the branch is ready. If the current branch is `main` or `master`, stop and ask the user to switch branches.

Done when `OWNER`, `REPO`, the current branch, and the base branch are known, and the branch is ready for review.

## 2. Understand the change

Compare the branch with its base:

```bash
git log "origin/$BASE..HEAD" --oneline --no-decorate
git diff --stat "origin/$BASE...HEAD"
git diff "origin/$BASE...HEAD"
```

Read changed files when the diff does not explain the behaviour. Infer the PR title, summary, change type, related issue, and test procedure from the branch name, commits, diff, and conversation. Do not invent missing facts. Ask one focused question for any required fact that cannot be supported. Use `N/A` for a missing issue only after the user confirms it.

Done when the PR title, scope, issue value, change type, and test evidence are recorded.

## 3. Read the PR template

Use `github_get_file_contents` with `owner`, `repo`, and the base ref to check these paths:

- `.github/pull_request_template.md`
- `.github/PULL_REQUEST_TEMPLATE`

If the directory contains multiple templates, read each candidate and choose the one that matches the changed area. Ask the user when no single template applies. If neither path exists, use the repository's documented PR format or a short body with summary, testing, and checklist sections.

Fill the selected template in place. Preserve its headings, ordering, checkboxes, and HTML comments. Replace placeholders with the gathered facts. Keep unrelated template text unchanged.

Done when the body follows one repository template exactly, or a fallback body is ready because the repository has no template.

## 4. Publish the branch

Check for an existing open PR before creating one. Use `github_search_pull_requests` with a query scoped to `repo:OWNER/REPO`, `is:pr`, and the current head branch. If a PR already exists, show its URL and ask whether the user wants to update it instead.

Publish local commits with git:

```bash
git push --set-upstream origin HEAD
```

The GitHub MCP `github_push_files` operation writes file contents as a new commit. It does not publish the current local commit history, so use `git push` for this workflow.

If the branch was rebased and requires a force push, pause and ask for explicit approval. Do not use `--force`.

Done when the remote contains the current branch and no duplicate open PR exists, or the existing PR has been handed back to the user.

## 5. Create the PR with GitHub MCP

Call `github_create_pull_request` with:

```json
{
  "owner": "OWNER",
  "repo": "REPO",
  "head": "BRANCH",
  "base": "BASE",
  "title": "PR title",
  "body": "Template-compliant body",
  "draft": false
}
```

Set `draft` to `true` only when the user asks for a draft. Add `reviewers` only when the user names reviewers. Use the exact owner, repository, branch, and base values from the repository checks.

Done when the MCP response contains the new PR number and URL.

## 6. Report the result

Give the user the PR URL and number. State that repository CI can now run and list any tests that were not run. Mention an existing PR instead of creating a second one when that path was taken.

Done when the user has the URL and knows the remaining checks or follow-up action.

## Common cases

- **No commits ahead of base:** report that there is no change to submit and ask whether the user meant another branch.
- **Push fails:** report the exact git error. Check remote access and branch protection before suggesting another action.
- **An open PR already exists:** use `github_update_pull_request` only after the user asks to update it. Otherwise return the existing URL.
- **The branch conflicts with base:** report the conflict and ask whether the user wants to resolve or rebase it. Keep the branch unchanged until they choose.
- **MCP access fails:** report the failing tool and its error. Ask the user to connect or authenticate the GitHub MCP server.
