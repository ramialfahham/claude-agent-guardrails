# claude-agent-guardrails

A reusable Claude Code plugin for my data/dbt projects. Install it once into any
project and get the same guardrails everywhere — no copying, no reinventing.

## What's in it

**1. Keeps the agent on track each session**
- Hands over notes between chats, so a new session continues where the last stopped
- Makes the agent restate the task and get your OK before writing code
- Re-checks the plan right after you approve it

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

## Per-project files (not part of this plugin)

Each repo still keeps its own short `CLAUDE.md` (what the project is) and
`.claude/active_work.md` (the handover note the session hooks read and update).

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
