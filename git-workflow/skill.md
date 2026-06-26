---
name: git-workflow
description: Apply consistent git commit message style and PR conventions across all projects. Use when writing commit messages, creating PRs, or reviewing branch hygiene. Invoke explicitly via /git-workflow.
---

# Git Workflow Conventions

## Commit messages

Format: `<type>(<scope>): <subject>`

- **type**: `feat` | `fix` | `refactor` | `test` | `docs` | `chore` | `style` | `perf`
- **scope**: the module or layer affected, e.g. `wallet`, `auth`, `order`, `infra`
- **subject**: imperative present tense, lowercase, no period. Max 72 chars total on first line.

Examples:
```
feat(wallet): add withdrawal hold lock
fix(auth): handle expired OTP token correctly
test(order): seed demo order data for member 1
chore: add dev-standards submodule to .claude/skills
```

Multi-line body (optional): blank line after subject, then free prose explaining *why*, not *what*.

Footer: always add `Co-Authored-By: Claude Sonnet 4.6 <noreply@anthropic.com>` when Claude wrote
or significantly edited the commit content.

## Branch naming

`<type>/<short-description>` — e.g. `feature/wallet-order-module`, `fix/otp-expiry`, `chore/standards-submodule`

## PR conventions

- Title: same format as commit subject (type + scope + imperative description)
- Body sections: **Summary** (bullet points of what changed), **Test plan** (checklist of what to verify)
- Keep PRs focused: one feature or fix per PR. Refactors that touch many files get their own PR.
- Target branch: `deploy/digitalocean-droplet` for ClipVerse; `main`/`master` for other projects (check repo).
- Never force-push to main/master. Never skip pre-commit hooks (`--no-verify`).

## Branch hygiene

- Delete merged branches after PR close.
- Rebase feature branches on the base branch before opening PR (no merge commits in feature history).
- One commit per logical change; squash fixup commits before merge if they add noise.

## When to invoke this skill

- Before writing any `git commit -m` message
- When creating a PR with `gh pr create`
- When the user asks for a commit message or PR description
