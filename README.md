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
