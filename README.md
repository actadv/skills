# actadv-skills

A Claude Code plugin marketplace with curated skills for software development.

## Plugins

| Plugin | Skills | Description |
|--------|--------|-------------|
| **actadv** | `review`, `security-audit`, `develop` | Code review, security audit, and parallel issue development |
| **git-workflow** | `smart-commit`, `pr-create`, `changelog` | Smart commits, PRs, and changelogs |
| **documentation** | `generate-docs`, `explain` | Generate docs and explain code |

## Install

Add this marketplace to Claude Code:

```
/plugin marketplace add actadv/skills
```

Install a plugin:

```
/plugin install code-review@actadv-skills
```

Use a skill:

```
/actadv:review src/api/
/actadv:security-audit
/actadv:develop 3
/git-workflow:smart-commit
/git-workflow:pr-create main
/git-workflow:changelog v1.0.0
/documentation:generate-docs src/lib/
/documentation:explain src/lib/parser.ts:42
```

## Local development

Test a plugin locally:

```
claude --plugin-dir ./plugins/code-review
```

Test the full marketplace:

```
/plugin marketplace add ./
```

## License

MIT
