# Commit Convention

This project follows [Conventional Commits](https://www.conventionalcommits.org/) with a concise narrative body style.

## Format

```
<type>(<scope>): <subject>

<body>
```

## Subject Line

- Use **imperative mood**: "add", "fix", "remove" (not "added", "fixes", "removed")
- Max **50 characters**
- Lowercase, no period at the end
- Describe **what** the commit does

## Body

- Separate from subject with a blank line
- Wrap at **72 characters** per line
- Explain **why** the change was made, not what (the diff shows what)
- Write in a concise narrative style
- Optional for trivial changes

## Types

| Type | When to use | Example |
|------|-------------|---------|
| `feat` | New service, feature, or capability | `feat(mysql): add mysql8-4 service` |
| `fix` | Bug fix or correction | `fix(pg): correct healthcheck command` |
| `docs` | Documentation only | `docs: add troubleshooting section` |
| `chore` | Maintenance, config, dependencies | `chore: update .gitignore` |
| `refactor` | Restructure without changing behavior | `refactor: reorganize volume paths` |

## Scope (optional)

Use the service or area affected:

| Scope | Area |
|-------|------|
| `mysql` | MySQL services |
| `pg` | PostgreSQL services |
| `docs` | Documentation |
| `config` | docker-compose, .env, labels |

Omit scope when the change is project-wide.

## Examples

### Good

```
feat(pg): add pg17 service

PostgreSQL 17 added as a general purpose database with
healthcheck, log rotation, and localhost-only binding.
```

```
fix(mysql): bind ports to localhost only

Ports were exposed to 0.0.0.0 allowing access from any
device on the local network. Restricted to 127.0.0.1.
```

```
docs: add prerequisites section
```

```
chore: add .editorconfig for yaml consistency
```

### Bad

```
updated stuff                          # vague, no type
feat: Add MySQL 8.1 Service.          # uppercase, period
fix(mysql): fixed the port issue      # past tense
feat: add mysql and update docs       # two concerns in one commit
```

## Rules

1. One logical change per commit
2. Never include secrets in commit messages
3. If the subject is self-explanatory, the body is optional
4. Reference issues when applicable: `fix(pg): resolve startup crash (#12)`
5. Never add `Co-Authored-By` trailers from AI tools or code generators
