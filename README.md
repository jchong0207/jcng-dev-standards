# jcng-dev-standards

Portable coding standards and workflow skills for Claude Code agents.

## What's in here

All skills under `skills/` are auto-discovered by Claude Code when this repo is mounted as a
submodule at `.claude/skills/` inside a project repo.

## Adding to a new project

```bash
git submodule add https://github.com/jchong0207/jcng-dev-standards.git .claude/skills
git commit -m "chore: add dev-standards submodule to .claude/skills"
```

## First clone on a new machine

```bash
git clone <project-repo>
git submodule update --init --recursive
```

## Updating skills in a consumer project

```bash
git submodule update --remote .claude/skills
git commit -m "chore: update dev-standards submodule"
```

## What stays global (not here)

- `working-with-jira` — workspace credentials, stays in `~/.claude/skills/`
- `writing-to-confluence` — workspace credentials, stays in `~/.claude/skills/`
- superpowers plugin itself — managed by the plugin installer
