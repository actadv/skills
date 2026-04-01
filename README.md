# actadv-skills

A Claude Code plugin marketplace with curated skills for software development.

## Plugins

| Plugin | Skills | Description |
|--------|--------|-------------|
| **code-review** | `review`, `security-audit` | Deep code review and security auditing |
| **git-workflow** | `smart-commit`, `pr-create`, `changelog` | Smart commits, PRs, and changelogs |
| **documentation** | `generate-docs`, `explain` | Generate docs and explain code |
| **testing** | `write-tests`, `test-coverage` | Write tests and find coverage gaps |
| **refactor** | `refactor`, `simplify` | Guided refactoring and simplification |

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
/code-review:review src/api/
/code-review:security-audit
/git-workflow:smart-commit
/git-workflow:pr-create main
/git-workflow:changelog v1.0.0
/documentation:generate-docs src/lib/
/documentation:explain src/lib/parser.ts:42
/testing:write-tests src/utils/validate.ts
/testing:test-coverage src/
/refactor:refactor src/services/
/refactor:simplify src/utils/transform.ts
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
