# claude-agent-guardrails

A reusable Claude Code plugin for my data/dbt projects. Install it once into any
project and get the same guardrails everywhere — no copying, no reinventing.

## What's in it

**1. Keeps the agent on track each session**
- Hands over notes between chats, so a new session continues where the last stopped
- Makes the agent restate the task and get your OK before writing code
- Re-checks the plan right after you approve it
- `/validate` runs CI's checks locally before you push; `/status` shows branch, PRs, CI + the handover

**2. Reviews the work before a commit**
- Four reviewer roles: scope, analytics-engineer (dbt), CTO (tooling/CI), data-engineer (ingestion)
- A gate that blocks the commit until the required reviewers have passed
- Routing (which reviewer is needed for which files) + templates

**3. Safety before push**
- Won't push unless tests/build have passed
- Confirms you're on a feature branch, not main

**4. Set up a new dbt project**
- A command that drops in the standard folder layout, layer rules, SQLFluff config,
  and a CI file that runs build + tests

## Install

```bash
# once per machine
/plugin marketplace add ramialfahham/claude-agent-guardrails

# in any project
/plugin install claude-agent-guardrails@rami-guardrails
```

It's public, so teammates run the same two commands. (Or clone it and point
`/plugin marketplace add` at the local folder path.)

## Per-project files (the setup command writes these into your repo)

`setup-dbt-project` drops in a `CLAUDE.md`, a `.claude/working-agreement.md` (the rules
the hooks and reviewers reference — five-step protocol, branch discipline, decision rights,
escalation, anti-patterns), `.claude/review_routing.json`, a read-only dbt MCP config (`.mcp.json`, lineage for the
agent + reviewers — needs `uv`/`uvx`), a gitleaks secret-scan (`.gitleaks.toml` +
`.github/workflows/security-secrets.yml`), a `.pre-commit-config.yaml`, and a starter
`.claude/active_work.md` handover. Editable templates live in `templates/repo/`.

## Build status

- [x] Folder structure + manifest
- [x] Process-spine hooks (handover, plan-back, plan→implement, pre-push)
- [x] Reviewer agents (scope, analytics-engineer, CTO, data-engineer)
- [x] Blocking review gate + routing + templates
- [x] `setup-dbt-project` command + dbt template files

## How the review gate works

1. You stage your change (`git add ...`).
2. You run the reviewers the routing requires and write `.claude/task/review.md`
   (copy `task/REVIEW_TEMPLATE.md`), pasting the staged-diff hash.
3. `git commit` is blocked until the review matches the staged change, every
   required reviewer passed, and any escalation has an answer.

Tests are the *first* safety net: the dbt CI workflow (next build step) runs
build + tests and blocks the merge. The review gate is the second net.
