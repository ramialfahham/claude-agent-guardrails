---
name: validate
description: Run the dbt project's quality gates locally BEFORE pushing — SQL lint, dbt parse, and (if authenticated) a scoped build + tests — so a PR doesn't bounce off CI. Use before `git push` / opening a PR, or when asked "will CI pass?".
---

# validate

Run the checks CI runs, locally, and report pass/fail per gate — catching failures in
seconds instead of after a push → red CI → fix-commit round trip.

## Gates (run in order; stop and report on the first hard failure)

### Tier 1 — fast, offline (always)

```bash
sqlfluff lint models            # SQL style/correctness (uses .sqlfluff)
dbt parse                       # the project compiles
```

Plus any project-specific offline checks the repo has (python tests via `pytest`,
JSON/YAML validity, custom scripts). If the repo has a CI workflow, mirror its offline
steps here so this stays a faithful local copy of CI.

### Tier 2 — needs the warehouse (run only if you are authenticated)

```bash
dbt deps
dbt build --select state:modified+ --state <ci-manifest>   # changed models + their tests
# or, without a state manifest, a full but slower:  dbt build
dbt build --select package:dbt_project_evaluator           # structure lint: missing tests/docs, DAG shape (cheap, metadata-only)
```

No warehouse auth → skip with a note and let CI run the build. Don't run a full `dbt build`
routinely — it costs query bytes.

## Reporting

One line per gate: `PASS` / `FAIL` (with the first error) / `SKIPPED (no auth)`. If any
gate fails, fix it before pushing — that failure would have been a red CI check.

## Keep in sync

If CI adds or changes a gate, update this list so local validation stays a faithful mirror.
