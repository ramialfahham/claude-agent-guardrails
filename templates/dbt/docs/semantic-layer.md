# The semantic layer (optional)

Where metrics are defined **once** and read everywhere else (working-agreement §10). This
is opt-in — set it up when you want a single source of truth for metrics that BI tools and
ad-hoc queries can share. It uses dbt's semantic layer (MetricFlow).

## Definition vs serving — know the boundary

- **Definitions** (what this kit sets up): `semantic_models` and `metrics` in YAML, on top
  of your marts. Open-source and warehouse-agnostic. Validate and query them locally with
  the MetricFlow CLI (`mf`, from the `dbt-metricflow` package) — no dbt Cloud needed.
- **Serving** — the hosted Semantic Layer API (JDBC/GraphQL, BI-tool integrations) — is a
  **dbt Cloud** feature. You don't need it to define or test metrics; you need it only to
  expose them to BI tools through the managed API. The kit sets up the definition half; the
  serving half is yours to add in dbt Cloud if you want it.

## Where it sits

Marts stay BI-ready tables. Semantic models sit **on top of** marts: a semantic model reads
a mart (its input) and exposes entities, dimensions, and measures; metrics are defined on
those measures. The mart a metric reads from is the metric's *input*, not a second
definition — that's the §10 distinction (defining once vs independently re-deriving).

Put the YAML under `models/semantic/` — any `.yml` under the model path is picked up.

## Install

The setup command adds this on opt-in; the extra matches your adapter:

```
pip install 'dbt-metricflow[dbt-bigquery]'
#                          ^ or dbt-snowflake | dbt-postgres | dbt-duckdb | dbt-redshift | dbt-databricks
```

## Worked example

Copy this once you have a mart to build on. It references `ref('fct_orders')`, so add it
only when that model exists — an empty `models/semantic/` keeps `dbt parse` green.

```yaml
# models/semantic/orders.yml
semantic_models:
  - name: orders
    model: ref('fct_orders')
    defaults:
      agg_time_dimension: order_date
    entities:
      - name: order_id
        type: primary
      - name: customer_id
        type: foreign
    dimensions:
      - name: order_date
        type: time
        type_params:
          time_granularity: day
      - name: status
        type: categorical
    measures:
      - name: order_amount
        description: Order amount in the order's currency.
        agg: sum
        expr: amount

metrics:
  - name: revenue
    label: Revenue
    description: Total order revenue.
    type: simple
    type_params:
      measure: order_amount
```

## Validate and query locally (no dbt Cloud)

```
dbt parse                 # the semantic YAML compiles with the rest of the project
mf validate-configs       # deeper semantic-layer validation
mf query --metrics revenue --group-by metric_time   # run a metric (add a grain, e.g. metric_time__day, per the MetricFlow docs)
```

If a consumer needs a metric that doesn't exist yet, add it here and reference it — never
re-derive it in a second mart, the consumption layer, or a BI tool (§10).
