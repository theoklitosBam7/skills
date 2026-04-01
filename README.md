# Agent Skills

A collection of AI agent skills.

## Skills

| Skill | Description |
|-------|-------------|
| `git-commit-generator` | Generate conventional commit messages from staged changes |
| `git-squash-rebase` | Interactive git rebase and squash operations |

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
