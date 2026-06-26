# __PROJECT_NAME__

<one paragraph: what this project is and does>

## Working agreement (read first)

Read [`.claude/working-agreement.md`](.claude/working-agreement.md) before doing anything.
The essence:

- Every change runs **Explore → Plan → Confirm → Implement → Verify**. Confirm means
  **wait for an explicit "go" before editing or running anything.**
- Never commit or push to `main`. Branch, open a PR, let the user merge.
- The decisions in §6 (product, naming, anything permanent, new mechanisms, cost) are the
  user's — escalate, don't decide.

## Stack

- <language / warehouse / key tools>

## Layout

- <key folders>
- dbt layer rules: [`docs/layering.md`](docs/layering.md)

## Guardrails

This repo uses the [`claude-agent-guardrails`](https://github.com/ramialfahham/claude-agent-guardrails)
plugin: session handover, plan-back gate, pre-push checks, blinded reviewers, and a
blocking review gate. Task contracts live in `.claude/task/`, review routing in
`.claude/review_routing.json`. The session handover is `.claude/active_work.md` — keep it
current so a fresh chat continues from the documented state.
