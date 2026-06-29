# dbt-agent-kit

A reusable Claude Code plugin for my data/dbt projects. Install it once into any
project and get the same guardrails everywhere — no copying, no reinventing.

## What's in it

**1. Keeps the agent on track each session**
- Hands over notes between chats, so a new session continues where the last stopped
- Makes the agent restate the task and get your OK before writing code
- Re-checks the plan right after you approve it
- `/validate` runs CI's checks locally before you push; `/status` shows branch, PRs, CI + the handover
- Enforces branch discipline: hard-blocks committing/pushing on `main`, `gh pr merge`, and `--amend`/`--no-verify`

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
/plugin marketplace add ramialfahham/dbt-agent-kit

# in any project
/plugin install dbt-agent-kit@dbt-agent-kit
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

## First run on a new project

This is a *starting* scaffold, not a finished setup — use it as-is, then adjust per project
as you learn what that project needs. It's warehouse-agnostic: the setup command injects the
warehouse-specific bits (adapter, profiles, CI auth) for whichever adapter you pick.

Two tiers, by risk:

- **Adopt as-is (low risk).** The whole governance layer — `working-agreement.md` (incl.
  §9 untrusted data, §10 single metric definition), `CLAUDE.md`, the layer rules, the
  reviewer agents, and the hooks. A hook *bug* can't block you — the hooks fail open on
  error — while the guards meant to stop you (branch discipline, the review gate) hard-block
  by design. The docs are just docs. Safe to drop into anything.
- **Expect a small first-run tweak** — only where the kit touches an external system:
  - **CI warehouse auth** — the setup command injects per-warehouse secrets/auth; the usual
    first-PR hiccup is a secret name not matching. You'll see it as the `build` job going red
    (and `gate`, the required check, going red with it) while `changes` stays green.
  - **dbt-project-evaluator's first run** — on a real project it may surface many
    "missing test/description" findings. That's it working, not breaking; tune it with a
    `dbt_project_evaluator_exceptions` seed (see `docs/cost-controls.md`).
  - **The read-only dbt MCP** (`.mcp.json`) — needs `uv`/`uvx` and correct local paths.
  - **dbt floor** — the scaffold expects dbt-core `>=1.10` (required by dbt-project-evaluator);
    if your adapter isn't there yet, `dbt deps` will tell you.
  - **Semantic layer** (opt-in) — definitions and local `mf` validation are OSS; serving via
    the Semantic Layer API needs dbt Cloud. See `docs/semantic-layer.md`.

Heads-up: the scaffold hasn't been exercised end-to-end yet, so treat your first real project
as the shakeout — the rough edges above are the kind that only show up on contact with a
warehouse, and fixing them as you go is the intended workflow.

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
