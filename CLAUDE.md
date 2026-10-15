# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project status: greenfield

As of this file's creation the repo contains only `README.md` and a single commit —
**no source code, no package manifest, no build or test tooling exists yet.** Do not
assume any command below the "Intended stack" section works; verify before running.

When scaffolding lands (pyproject/uv lock, dlt pipelines, dbt project, FastAPI app),
replace this section with the real commands for install, run, test, and lint.

## What this is

koi data studio — AI-powered data ingestion, transformation, and delivery, built as a
medallion-architecture ELT pipeline:

**Ingestion** — **dlt** loads each source system into its own landing database/schema.
No transformation happens here; it's raw extract-and-load, one landing area per source.

**Transformation** — **dbt**, moving data through three layers:

| Layer | Move | What happens |
|---|---|---|
| Bronze | landing → bronze | Views over landing: current (active records) + history (previous records) |
| Silver | bronze → silver | Data vault model, built with the **automate_dv** package |
| Gold | silver → gold | Kimball-style fact/dimension tables, plus documented metric views forming a semantic layer |

**Delivery** — chosen per use case, not both by default:
- **plotly** when the use case is BI (dashboards, charts, exploratory analysis)
- **fastapi** when the use case is a data product / application (an HTTP API serving
  other software)

**Storage** — also chosen per use case:
- **sqlite** for transactional workloads
- **duckdb** for analytical workloads

Treat delivery and storage as per-feature decisions, not a fixed stack: a new capability
should pick the tool that matches its workload rather than defaulting to whichever one
is already in use elsewhere in the repo.

## Conventions to follow as code is added

- **Python-first.** dlt, dbt, automate_dv, FastAPI, and Plotly are all Python; keep one
  virtualenv and one dependency manifest for the whole project rather than per-stage
  environments.
- **Directories mirror the medallion layers.** Separate top-level packages/dbt folders
  for ingestion (dlt sources/pipelines), and for bronze, silver, and gold dbt models, so
  each layer can be built and tested independently.
- **Respect the layer boundaries.** Bronze only reshapes landing data into
  current/history views — no business logic there. Silver is the data vault
  (hubs/links/satellites via automate_dv) — no star-schema modeling there. Gold is where
  Kimball facts/dimensions and semantic-layer metrics live — that's the only layer
  consumers (BI or API) should query directly.
- **dbt owns SQL correctness.** Transformation logic and its tests belong in dbt
  models/schema tests, not hand-written SQL inside FastAPI or Plotly code.
- **Delivery and storage are chosen, not fixed.** Before adding a new endpoint, dataset,
  or dashboard, decide BI-vs-product and transactional-vs-analytical explicitly, and
  document that choice near the code — don't assume the whole app uses one database or
  one delivery mechanism.
- **Database files are artifacts, not source.** `.duckdb` / `.sqlite` files must stay
  gitignored; the repo should be reproducible by re-running dlt + dbt.
- **Secrets stay out of the tree.** dlt reads credentials from `.dlt/secrets.toml` or
  environment variables — never commit either.

## Environment notes

- Windows 11; PowerShell is the primary shell (a Bash tool is also available — each
  takes its own syntax).
- Repo lives under OneDrive (`C:\Users\Steven\OneDrive\Documents\Projects\koi_data_studio`).
  Be aware that OneDrive sync can touch files mid-operation; prefer keeping generated
  database files and virtualenvs gitignored and, if possible, out of synced paths.
- System Python is 3.14.4. No `uv`, `dlt`, or `dbt` on PATH yet — installing project
  tooling is a prerequisite for any pipeline work.
