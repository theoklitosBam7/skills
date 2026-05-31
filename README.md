# Agent Skills

A collection of AI agent skills.

## Installation

- **git-commit-generator** — Generate conventional commit messages from staged changes

  ```
  npx skills@latest add theoklitosBam7/skills/git-commit-generator
  ```

- **git-squash-rebase** — Rebase safely stacked branches

  ```
  npx skills@latest add theoklitosBam7/skills/git-squash-rebase
  ```

- **marp-presentations** — Create beautiful slide decks from Markdown using Marp

  ```
  npx skills@latest add theoklitosBam7/skills/marp-presentations
  ```

- **plan-visualizer** — Rich self-contained HTML plans with Mermaid diagrams and interactive task tracking

  ```
  npx skills@latest add theoklitosBam7/skills/plan-visualizer
  ```

- **react-useeffect-guide** — Guide for correct and efficient React useEffect code

  ```
  npx skills@latest add theoklitosBam7/skills/react-useeffect-guide
  ```

- **babysit** — Keep a PR merge-ready by triaging comments, resolving conflicts, and fixing CI *(originally from Cursor)*

  ```
  npx skills@latest add theoklitosBam7/skills/babysit
  ```

- **split-to-prs** — Split current work into small reviewable PRs *(originally from Cursor)*

  ```
  npx skills@latest add theoklitosBam7/skills/split-to-prs
  ```

## Adding New Skills

To add a new skill, create a directory with:

- `SKILL.md` — documentation and usage instructions
- `agents/` — agent configuration files

## Structure

```
.
├── <skill-name>/              # Skill directory
│   ├── SKILL.md               # Skill documentation and usage
│   └── agents/                # Agent configuration
├── .gitignore
└── README.md                  # This file
```

## License

MIT
