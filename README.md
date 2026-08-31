# ML Platform Framework

Give the EDA team the same "just build it" experience for ML that they have
for BI with Power BI: standardized inputs, standardized building blocks,
automated plumbing, and an agent that knows the whole framework well enough
to generate compliant code.

This repo is the **umbrella / entry point**. The framework itself lives in
four separate repos (scope doc §12.1).

## Read in this order

1. **[`ELEVATOR_PITCH.md`](ELEVATOR_PITCH.md)** — one page, for a
   stakeholder or a quick recall of the whole idea.
2. **[`ML_Platform_Framework_Scope.md`](ML_Platform_Framework_Scope.md)** —
   the full scoping document: the seven pillars, gap analysis, locked
   naming/ownership decisions, MVP plan.
3. **[`claude-code-config/`](claude-code-config/)** — the skills
   (`new-ml-project`, `add-playbook-entry`, `add-pillar-module`) and
   subagents that automate working with the framework. Install per its
   `README.md`.

## The four repos

| Repo | What it is |
|---|---|
| **[platform-framework](https://github.com/FilippDor/platform-framework)** | Canonical, versioned source of the shared infra modules (`config`, `schema`, `registry`, `validation`, `monitoring`, `feature_store`) + `platform.yaml` + `scripts/bootstrap.py` + the drift checker. Currently **v0.4.0**. Copied into each project at creation time — not a runtime dependency. |
| **[ml-project-template](https://github.com/FilippDor/ml-project-template)** | The analyst starting point. Thin orchestration notebooks + `src/` (one job per file) + tests + CI. Self-contained, runnable the moment it's cloned. |
| **[ml-algorithm-playbook](https://github.com/FilippDor/ml-algorithm-playbook)** | The knowledge base for algorithm choice, required metrics, and — critically — the **required backtest and forward-test method per algorithm**. |
| **[ml-retention-churn-risk](https://github.com/FilippDor/ml-retention-churn-risk)** | The **pilot**: Telco customer churn, run end-to-end through the template against a live Snowflake Dev account — schema → features → Feature Store → train (k-fold backtest) → evaluate → Model Registry → batch scoring → drift monitoring. Proof the plumbing works. |

## Status

- **All seven pillars exercised end-to-end** at MVP depth by the pilot.
  `CHURN_RISK` is registered in `MODEL_REGISTRY.RETENTION`, scored rows
  land in `ML_DEV.CHURN_RISK.SCORES`, drift rows in
  `MONITORING.PUBLIC.DRIFT_LOG`.
- **One config file per project.** Everything a project needs is in its
  root `project.yaml` — identity, Snowflake object names, the feature spec,
  training + monitoring tunables. `src/config.py::load_config()` parses it;
  only `ML_ENVIRONMENT` / `SNOWFLAKE_CONNECTION` / `MLFLOW_TRACKING_URI` are
  env-var overrides. Secrets stay in `~/.snowflake/connections.toml`.
- **Snowflake bootstrap** (`platform-framework/scripts/bootstrap.py`, which
  reads `platform.yaml`) has been applied to the Dev account: databases
  `GOLDEN_LAYER`, `FEATURE_STORE`, `MODEL_REGISTRY`, `MONITORING`.
- **Not done yet:** the Prod account + tagged-release promotion path
  (`ml-project-template/.github/workflows/release.yml` is written but
  untested), the Pillar 6 observability dashboard, on-demand UDF scoring,
  and the regression / forecasting / clustering project types. See the
  scope doc's MVP plan for the remaining hardening work.

## Next step for a new project

Install `claude-code-config`, then `/new-ml-project` — it clones
`ml-project-template`, stamps `.framework-version`, and checks the Playbook
for a matching algorithm entry before any training code is written.
