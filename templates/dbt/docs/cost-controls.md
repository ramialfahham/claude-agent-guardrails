# Keeping runs economic

How this project avoids paying to rebuild things that didn't change. Three levers — the
first two ship in the scaffold, the third is a pattern to copy when you add a scheduled
build. All warehouse-agnostic.

## 1. Materialization — don't store what you can recompute cheaply

Staging, base, and intermediate models are **views** (see `dbt_project.yml`): no storage,
nothing to rebuild. Only `core` and `marts` materialize as tables. This is the single
biggest cost lever and it's already the default. See [`layering.md`](layering.md) for what
belongs in each layer.

## 2. CI builds only what changed

`.github/workflows/ci.yml` is set up so PRs don't pay for builds nobody asked for:

- **Path filter** (`changes` job, on by default): a docs-only or non-dbt PR skips the
  warehouse build entirely. The always-on `gate` job keeps a stable required status check
  so branch protection still works — make `gate` the required check, not `build`.
- **Slim builds** (`state:modified+`, opt-in): when the DAG grows and full builds start
  hurting, enable the commented-out block in `ci.yml`. PRs then build only the models you
  changed plus their downstream children and tests, diffed against a manifest compiled
  from `main` at PR time — no stored state, no `--defer`. `main`-push and dispatch keep
  the full build so prod stays complete.

## 3. Scheduled builds — skip the run when there's no new data

Not scaffolded (it needs an ingestion step that can report whether anything arrived), but
the pattern is simple: have ingestion emit a flag, and gate the dbt steps on it so a
nightly job does nothing when the source had no new rows.

```yaml
# .github/workflows/dbt-scheduled.yml (sketch)
- name: Run ingestion
  id: ingest
  run: python -m ingestion.main          # writes new_data=true|false to its step output

- name: dbt build
  if: steps.ingest.outputs.new_data == 'true'
  run: dbt build --target prod
```

The same idea works in any orchestrator (Airflow, Dagster, cron): answer "did anything
change?" before paying for a build.

## A note on dbt-project-evaluator

The structural-lint package (`package:dbt_project_evaluator`) builds small, schema-isolated
metadata models off the dbt graph and information schema — not your data — so it's cheap.
CI's `dbt build` builds **and tests** it on every dbt PR on purpose: its tests fail the
build when a model is missing a test or description, or the DAG shape drifts — that's the
structural gate. Tune or silence individual checks with a `dbt_project_evaluator_exceptions`
seed (see the package docs). Exclude it only from a scheduled prod **data** build, where it
adds nothing: `dbt build --exclude package:dbt_project_evaluator`.
