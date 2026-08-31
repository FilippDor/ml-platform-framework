# ml-project-template

The EDA team's standardized starting point for a new ML project on Snowflake.

- Self-contained — everything in `src/` is a snapshot copied from
  `platform-framework` at creation time, not a live dependency. See
  README.md for why.
- One job per file in `src/`: `schema.py` (Pillar 1), `feature_engineering.py`
  (Pillar 2), `train.py` + `validation.py` (backtest is mandatory and
  Playbook-determined, never ad hoc), `evaluate.py` (Pillar 4),
  `registry.py` (Pillar 3), `score.py` (Pillar 7), `monitoring.py` (Pillar 5).
- Notebooks in `notebooks/` are thin orchestrators only — no business
  logic in cells.
- Two environments: Dev (real data, per-user RBAC, all work happens here)
  and Prod (separate Snowflake account, reached only via a tagged release
  through `.github/workflows/release.yml`).
- Before writing training code, check `ml-algorithm-playbook` for the
  matching algorithm entry — it specifies the required backtest method,
  forward-test requirement, and metrics. Don't decide these ad hoc.
- Approved libraries only — see `pyproject.toml` and the platform scoping
  doc's library standardization table. Deviating requires a logged
  exception.
