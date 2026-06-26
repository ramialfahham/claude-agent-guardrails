---
name: setup-dbt-project
description: Scaffold a standard dbt project — layer folders, layering rules, SQLFluff config, packages, and a GitHub Actions CI that runs build + tests. Asks which warehouse. Run this in a new/empty repo.
disable-model-invocation: true
---

# Set up a new dbt project

Lay down the standard dbt project structure in the current repo. Scaffold only —
do not invent models or business logic.

## Steps

1. **Check the repo.** Confirm this is the right repo and there is no existing
   `dbt_project.yml`. If one exists, stop and ask the user.

2. **Ask the user** (don't assume):
   - `project_name` — snake_case, e.g. `acme_analytics`
   - **which warehouse** — BigQuery, Snowflake, Postgres, DuckDB, Redshift,
     Databricks, or any other dbt adapter
   - the connection basics for that warehouse (project/account/host/database etc.)

3. **Copy the template tree** from `${CLAUDE_PLUGIN_ROOT}/templates/dbt/` into the
   repo root, then replace every `__PROJECT_NAME__` with `project_name`:
   - `dbt_project.yml`, `packages.yml`, `requirements.txt`, `.sqlfluff`,
     `.gitignore`, `profiles.example.yml`, `docs/layering.md`, `models/README.md`
   - create the five model layer folders, each with an empty `.gitkeep`:
     `models/staging`, `models/base`, `models/core`, `models/intermediate`,
     `models/marts`
   - also create empty `seeds/`, `macros/`, `tests/`, `analyses/`
   - copy `templates/dbt/github-ci.yml` to `.github/workflows/ci.yml`

4. **Apply the chosen warehouse** to the copied files:
   - `requirements.txt` — add the matching adapter (`dbt-bigquery`,
     `dbt-snowflake`, `dbt-postgres`, `dbt-duckdb`, `dbt-redshift`,
     `dbt-databricks`, …)
   - `profiles.example.yml` — keep the block for that warehouse, rename it to
     `dev`, add a `ci` copy, and delete the other warehouses' blocks. For a
     warehouse not shown, write the correct output per that adapter's docs.
   - `.sqlfluff` — set `dialect` to the warehouse's dialect (bigquery, snowflake,
     postgres, duckdb, redshift, databricks; `ansi` if unsure)
   - `.github/workflows/ci.yml` — add the warehouse's connection env vars (each
     reading a GitHub Actions secret) and, only if the warehouse needs it, an
     auth step (e.g. BigQuery writes the service-account JSON from a secret)

5. **Add the project's guardrail files** (so the guards have the docs they reference):
   - copy `${CLAUDE_PLUGIN_ROOT}/review_routing.json` to `.claude/review_routing.json`
   - copy `${CLAUDE_PLUGIN_ROOT}/templates/repo/working-agreement.md` to
     `.claude/working-agreement.md`
   - copy `${CLAUDE_PLUGIN_ROOT}/templates/repo/CLAUDE.md` to `CLAUDE.md` (only if none
     exists); replace `__PROJECT_NAME__` and fill the `<…>` placeholders
   - create `.claude/active_work.md` with a one-line starting note

6. **Tell the user the next steps** (do NOT run these for them):
   - `pip install -r requirements.txt`
   - copy `profiles.example.yml` to `~/.dbt/profiles.yml` (or set
     `DBT_PROFILES_DIR`) and fill in credentials
   - `dbt deps` then `dbt build`
   - in the GitHub repo, add the Actions secrets the CI file references

Then confirm what was created and stop.
