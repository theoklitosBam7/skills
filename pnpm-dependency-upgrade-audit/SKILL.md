---
name: pnpm-dependency-upgrade-audit
disable-model-invocation: true
description: >-
  Audit outdated direct dependencies in pnpm monorepos across the root
  and every workspace package, checking release age, engines, peer dependencies,
  semver ranges, changelogs, toolchain compatibility, and code usage before
  recommending safe upgrade batches. Use when asked to run pnpm outdated,
  assess dependency upgrades, or investigate whether latest versions are safe
  without modifying manifests or lockfiles.
---

# pnpm Dependency Upgrade Audit

## Scope

Perform a read-only investigation by default. Do not run `pnpm update`, `pnpm install`, `corepack use`, or edit manifests and lockfiles unless the user separately authorizes implementation. Use `pnpm` for package-manager commands; do not substitute npm or yarn.

Produce an evidence-based report that distinguishes:

- candidates suitable for a focused upgrade batch;
- candidates requiring targeted tests or migration work;
- candidates that should be deferred because of release-age, engine, peer, or toolchain constraints.

## Workflow

### 1. Discover the workspace and constraints

Inspect:

- root `package.json` for `packageManager`, scripts, engines, catalogs, and dependency ranges;
- `pnpm-workspace.yaml` for workspace globs, `minimumReleaseAge`, `nodeVersionFile`, catalogs, overrides, and build policies;
- the referenced Node version file and CI workflow Node versions;
- every workspace `package.json` for direct dependencies, scripts, and package-specific engines.

Resolve workspace directories from the workspace globs rather than assuming a fixed `apps/*` or `packages/*` layout.

### 2. Run `pnpm outdated` in every project context

Run the command separately from the root and each workspace package. A root invocation does not reliably provide the same direct-dependency view as invoking it from a member package.

Use absolute paths or a subshell for each directory so the working directory cannot drift between iterations. In zsh, do not store the command result in a variable named `status`; zsh reserves that name. Treat exit code 1 as expected when outdated dependencies are found.

Example pattern:

```sh
set +e
repo_dir="$(pwd)"
for workspace_dir in . apps/example packages/example; do
  printf '\n[%s]\n' "$workspace_dir"
  (cd "$repo_dir/$workspace_dir" && pnpm outdated)
  rc=$?
  printf 'exit code: %s\n' "$rc"
done
exit 0
```

Record package, current version, reported latest version, and workspace for every result. Deduplicate packages for compatibility analysis, but retain all workspace occurrences in the report.

### 3. Check release-age policy

Read `minimumReleaseAge` from `pnpm-workspace.yaml`; its unit is minutes. Capture the current UTC time with `date -u`.

Treat `pnpm outdated` as potentially policy-filtered. For each candidate, compare the registry’s actual latest version with the reported version using `pnpm view` and inspect the candidate’s publish timestamp. A newer registry version may be intentionally absent from `pnpm outdated` because it has not aged past the configured threshold.

Do not recommend a version that is still inside the release-age window. Call out the exact eligibility time when it matters, especially when the newer release fixes a regression in the older policy-eligible result.

### 4. Validate package compatibility

For each unique candidate, inspect concise registry metadata:

```sh
pnpm view "$package@$version" version engines peerDependencies --json
```

Compare:

- package engine requirements with the project engine floor and the pinned development/CI Node version;
- peer requirements with installed peers and `pnpm peers check` output;
- manifest ranges with the reported target—`pnpm update` without `--latest` will not cross a range boundary;
- catalog entries and workspace-wide consumers;
- coupled packages that must move together, such as a plugin and its linter;
- ESM/CommonJS changes, native binaries, formatter output, generated artifacts, and runtime behavior.

Run `pnpm peers check` as a baseline. Separate pre-existing peer failures from regressions that a proposed upgrade introduces.

Use this risk heuristic:

- patch or same-major minor update with compatible engines and peers: lower risk, still validate;
- major update, `0.x` minor update, ESM-only transition, native package, formatter, linter, compiler, or runtime: focused review required;
- engine mismatch, unresolved peer conflict, known regression, or unsupported toolchain integration: defer or block.

### 5. Review primary release information and repository usage

For major or behavior-sensitive candidates, browse the package’s official release notes, migration guide, changelog, or repository. Prefer primary sources such as the maintainer’s repository, official documentation, and official release pages.

Search the repository with `rg` for APIs and configuration affected by the release. Inspect the relevant source, test, and build usage rather than judging compatibility from package versions alone. Pay special attention to:

- parser/token APIs where whitespace or token boundaries feed source maps;
- Vue, Volar, or `vue-tsc` integrations when changing TypeScript;
- store libraries that add or change devtools peers;
- ESM-only command runners in package scripts;
- Electron IPC, window, security, download, and native-image behavior;
- lint rules, boundaries configuration, formatter output, and autofixes;
- jsdom/browser behavior used by tests.

### 6. Classify and report

Report the raw `pnpm outdated` results grouped by workspace, then a unique-package risk assessment. For every nontrivial recommendation include:

- current and candidate versions;
- release-age status;
- engine and peer compatibility;
- relevant breaking or behavioral changes;
- repository usage that increases or reduces risk;
- the smallest validation suite needed.

Recommend focused batches, not one bulk upgrade. Keep independent migrations separate—for example, compiler/toolchain changes, runtime changes, parser behavior changes, and lint/format tooling changes should not be mixed without a reason.

End by stating clearly whether files were changed. A read-only audit should leave manifests, lockfiles, source, and generated files untouched.
