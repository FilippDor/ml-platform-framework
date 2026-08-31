# EDA Team ML Platform Framework — Scoping Document

## 1. Vision / North Star

Give the EDA team the same "just build it" experience for ML that they already have for BI with Power BI: standardized inputs, standardized building blocks, automated plumbing, and an AI agent that knows the whole framework well enough to generate compliant code and make recommendations.

**Two deliverables, one ecosystem:**

| # | Deliverable | Purpose |
|---|---|---|
| 1 | **ML Platform Framework** | The infrastructure: schema standards, feature store, model registry, weights/scores storage, monitoring, drift detection, observability — all automated as much as possible. |
| 2 | **ML Knowledge Repo** ("Algorithm Playbook") | A separate Git repo cataloguing every ML algorithm the team is allowed/expected to use: pros/cons, required metrics, problem-fit, and pre-built code blocks. This is the "brain" the agent reasons from before recommending a path. |

The **agent** (Claude, orchestrated via Snowflake Cortex/Coco and Glean for enterprise knowledge retrieval) sits on top of both: it reads the Knowledge Repo to reason about *which* approach fits a problem, and reads the Platform Framework to generate *compliant* code (correct schema, correct registry calls, correct monitoring hooks) automatically.

---

## 2. Tech Stack Mapping

| Layer | Tool | Role |
|---|---|---|
| Storage / compute / training | **Snowflake** | Single engine — golden layer as source of truth, feature tables, model training (Snowpark ML / Cortex ML functions), scoring |
| Transformation / modelling layer | **Python (Snowpark)** | Schema/table creation, feature engineering, all logic lives in standardized `.py` modules reading directly from the golden layer — no dbt |
| Orchestration | **Notebooks** | Thin orchestration layer — call the standardized `.py` modules in sequence, kept out of the analyst's way for actual logic |
| Code & version control | **Git** | Three repos: `ml-project-template` (the analyst starting point), `ml-platform-framework` (shared libraries/infra), and `ml-algorithm-playbook` (knowledge base) |
| AI — code generation & reasoning | **Claude** | Primary agent for reasoning, scoping, code generation, doc generation |
| AI — native Snowflake agent | **Coco (Snowflake Cortex)** | In-warehouse AI actions: querying feature store/registry metadata, running Cortex ML functions, drift detection queries directly on Snowflake data |
| AI — enterprise knowledge retrieval | **Glean** | Surfaces prior project docs, tickets, decisions across company knowledge sources so the agent doesn't reinvent context |

---

## 3. The Seven Pillars (Full North Star Scope)

For each pillar, I've noted the **golden standard** end-state and the **MVP** cut.

### Pillar 1 — Schema Generation & Standards
- **Golden standard:** A shared Python library (`platform_framework.schema`) with functions that create standardized schemas/tables per project type (classification, regression, forecasting, clustering) directly via Snowpark — enforced naming conventions, required metadata tags, auto-documentation generated from the code itself (docstrings → docs).
- **MVP:** One shared schema-creation module in the template repo, reading from the golden layer, covering a single project type end to end.

### Pillar 2 — Datasets & Feature Store
- **Golden standard:** Centralized Snowflake feature store (feature tables tagged by domain/owner/freshness), reusable feature definitions, point-in-time correctness for training/serving consistency, auto-versioning.
- **MVP:** One shared Snowflake schema convention for features (`FEATURE_STORE.<domain>.<feature_table>`) with a lightweight registration process (a Python module in the template repo + a metadata table logging owner, refresh cadence, source).

### Pillar 3 — Model Registry
- **Golden standard:** Central registry (Snowflake Model Registry, native to Snowpark ML) tracking every model version, lineage back to features/data version, approval status, deployment stage.
- **MVP:** Adopt Snowflake's native Model Registry for one pilot project; standardize the registration script as a reusable template.

### Pillar 4 — Weights, Scores & Artifact Storage
- **Golden standard:** Standardized artifact storage convention (stage-based in Snowflake) for model weights, evaluation scores, and metadata, versioned and linked to the registry entry.
- **MVP:** One standardized Snowflake stage structure + a template script that logs scores/weights consistently for the pilot project.

### Pillar 5 — Monitoring & Data Drift
- **Golden standard:** Automated drift checks (feature drift, prediction drift, label drift where available) running on a schedule via Snowflake tasks, with alerting thresholds and dashboards.
- **MVP:** One scheduled Snowflake task running a basic statistical drift check (e.g., PSI or KS-test) on the pilot model's key features, logging results to a monitoring table.

### Pillar 6 — Observability
- **Golden standard:** End-to-end lineage and health dashboard — from raw data → features → model → predictions → drift/performance → **compute cost per project/pillar** — queryable and visual (PBI-style, since that's the team's comfort zone).
- **MVP:** A single dashboard (could be PBI on top of Snowflake monitoring tables) showing pilot project health only, including basic compute cost tagging from day one.

### Pillar 7 — Deployment & Serving
- **Golden standard:** Two approved serving patterns, both standardized as reusable template functions: **batch scoring** via a scheduled Snowflake task calling `score.py`, and **on-demand scoring** via a Snowpark UDF wrapping the registered model. Every deployment records a "previous stable version" pointer in the registry for rollback. **No model is promoted to sole decision-maker without first passing its Playbook-defined forward-test requirement** — a shadow/champion-challenger period comparing live predictions against the current stable model (or against actuals, once known) before it's trusted fully.
- **MVP:** Batch scoring only, for the pilot model — one scheduled task, standardized `score.py` contract, manual rollback procedure documented (automated rollback comes later). Forward-test period can be manually tracked for the pilot rather than automated.

---

## 4. The Project Template Repo (Analyst Starting Point)

Every analyst forks/clones `ml-project-template` to start a new ML project. It's a fully scaffolded end-to-end skeleton — the analyst fills in project-specific logic inside the standardized files, they don't invent structure.

Suggested structure:
```
ml-project-template/
├── notebooks/
│   ├── 01_orchestrate_ingestion.ipynb     # calls ingestion.py — no logic lives here
│   ├── 02_orchestrate_features.ipynb      # calls feature_engineering.py
│   ├── 03_orchestrate_training.ipynb      # calls train.py
│   ├── 04_orchestrate_scoring.ipynb       # calls score.py
│   └── 05_orchestrate_monitoring.ipynb    # calls monitoring.py
├── src/
│   ├── config.py                # project metadata: name, owner, domain, project type
│   ├── schema.py                # creates project schema/tables directly from golden layer (Snowpark)
│   ├── ingestion.py              # reads from golden layer, applies project-level filters
│   ├── feature_engineering.py    # feature logic + registers features to Feature Store
│   ├── train.py                  # model training, pulls algorithm choice from Playbook code-blocks
│   ├── evaluate.py                # computes required metrics per Playbook spec for chosen algorithm
│   ├── registry.py                # logs model version to Snowflake Model Registry
│   ├── score.py                   # batch/online scoring
│   └── monitoring.py              # drift checks, logs to monitoring tables
├── tests/                         # unit tests per module, required before merge
├── README.md                      # how to use this template, links to Platform Framework + Playbook
└── pyproject.toml / requirements.txt
```

**Why this shape works well for your goals:**
- Notebooks stay thin (orchestration only) — no business logic buried in cells, so it's reviewable and testable like normal code.
- Every `.py` file has one job, which is exactly what lets the agent generate or modify a single file confidently without touching the rest.
- Reading directly from the golden layer removes the need for a separate transformation layer (dbt) — `schema.py` and `ingestion.py` own that responsibility in Python, under version control, and can be unit tested.
- Because the structure is fixed, "scalability" comes from consistency: any analyst's project looks like any other analyst's project, so review, debugging, and platform-level automation (registry hooks, monitoring hooks) can assume a known shape.

---

## 5. Environments, Access & CI/CD (Updated — Two-Environment Model Confirmed)

Good news: two of the gaps flagged earlier are already solved by your existing platform, not things to design from scratch.

### 5.1 What already exists (don't rebuild this)
- **Dev environment**: a real Snowflake account with real golden-layer data and **per-user RBAC already assigned** — this is where all analyst work happens.
- **Authoring path**: analyst creates a repo → imports it into a **Snowflake Workspace via Git integration** → runs directly against real Dev data under their own role.
- **Prod environment**: a fully separate Snowflake account. Only proven, "well-established" repos get promoted here; result/prediction tables live in Prod.
- Because each Workspace's session is scoped to its own account, `get_active_session()` in the template's `src/` modules automatically resolves to whichever account the code is running in — **no manual account-switching logic needed** in `config.py`. Only two `Environment` values are needed: `DEV` and `PROD`.

### 5.2 How promotion actually works (mechanically)
Since Prod is a separate account, "promotion" means: the same reviewed repo is imported into a **Prod-side Snowflake Workspace/Git Repository object**, and run there against Prod's own golden layer — not an artifact copy from Dev.

1. Analyst's work in Dev is reviewed and merged/tagged in Git.
2. On a tagged release, a deploy step refreshes the **Prod Snowflake Git Repository object** to that tag (`ALTER GIT REPOSITORY <prod_repo> FETCH`), so Prod now has the exact reviewed code.
3. The standardized pipeline (`schema.py` → `feature_engineering.py` → `train.py` → `evaluate.py` → `registry.py` → `score.py`) runs inside Prod, producing Prod's own registered model and result tables.
4. Metrics are checked against an acceptance threshold before the model is tagged `stable` in Prod's Model Registry.

This keeps a clean separation of concerns exactly as you described: Dev is where problems get solved, Prod is where the solved version runs — and every Prod object traces back to a specific reviewed commit.

### 5.3 RBAC — what's actually still needed
Individual analyst RBAC in Dev is already handled. The only new role needed is scoped to automation, not people:

| Role | Where | Can do |
|---|---|---|
| *(existing per-user roles)* | Dev | Already assigned — no platform work needed here |
| `ML_DEPLOY_SERVICE` | Prod | Used only by the deploy step; refreshes the Prod Git Repository object and runs the pipeline. No individual analyst holds this role. |
| `MODEL_REGISTRY_APPROVER` | Prod | Optional human gate before the deploy step runs, if you want a manual checkpoint rather than fully automatic promotion |

### 5.4 CI/CD pipeline (in the template repo)
- **On every PR** (Dev work): `pytest` → `ruff` → block merge on failure.
- **On a tagged release**: a separate deploy job authenticates as `ML_DEPLOY_SERVICE`, fetches the tag into Prod's Git Repository object, and triggers the pipeline run in Prod — the only path that ever writes to Prod.

### 5.5 Rollback runbook (within Prod)
1. Prod's Model Registry always retains the previous `stable`-tagged version.
2. If a newly promoted model underperforms: re-point the serving task/UDF to the previous stable version — no retraining, no cross-account work.
3. Incident gets logged in the observability dashboard for the postmortem.

---

## 6. The ML Algorithm Playbook (Separate Repo)

Structure this as a **queryable knowledge base**, not just documentation — the agent needs to parse it reliably.

Suggested repo structure:
```
ml-algorithm-playbook/
├── README.md                     # how to use this repo + how the agent queries it
├── /algorithms/
│   ├── classification/
│   │   ├── logistic-regression.md
│   │   ├── xgboost.md
│   │   └── ...
│   ├── regression/
│   ├── clustering/
│   ├── forecasting/
│   └── anomaly-detection/
├── /decision-trees/               # "if problem looks like X, consider Y" flowcharts
├── /metrics/                      # required metrics per problem type + how to interpret them
├── /validation-frameworks/        # golden-standard backtest + forward-test methods, by problem family
│   ├── tabular-cross-validation.md      # k-fold / stratified k-fold — non-time-dependent problems
│   ├── time-series-backtesting.md       # walk-forward / expanding-window / blocked splits
│   └── forward-testing.md               # champion-challenger, shadow deployment, out-of-time holdout
└── /code-blocks/                  # ready-to-use snippets tied 1:1 to each algorithm, following platform standards
```

**Each algorithm file should follow a fixed template** (critical for agent reliability):
- Problem types it solves / doesn't solve
- Pros & cons
- Required/expected metrics + acceptable thresholds
- Data requirements & assumptions (e.g. linearity, feature scaling needs)
- When to prefer it vs. alternatives
- **Backtesting method** — which `/validation-frameworks/` approach applies (e.g. time series algorithms must reference `time-series-backtesting.md`, never plain k-fold, since random splits leak future information into the past)
- **Forward-looking test requirement** — how the model is validated once live, before it's fully trusted (e.g. shadow-mode period length, champion-challenger comparison window, minimum forward observations before promotion to sole decision-maker)
- Link to the standardized code block in `/code-blocks/`

This is what lets the agent do the "critical thinking" step: it matches the problem description → decision tree → shortlist of algorithms → their tradeoffs → **their required validation approach** → generates the actual code block wired into the platform framework (correct schema, logs to registry, hooks into monitoring, runs the correct backtest before training is considered "done").

**Why this can't be left to individual judgment:** the single most common ML mistake on tabular time-dependent data is validating with a random k-fold split, which silently leaks future information into training and makes a model look far better than it will ever perform live. Making the backtest method a required, non-optional field per algorithm — rather than a step the analyst decides on the fly — is what actually prevents that class of error at scale.

---


## 7. What the Agent Needs (Context Package)

For Claude (or Coco) to reliably generate compliant code and reasoning, it needs standing access to:
1. The Project Template repo (fixed structure, so the agent knows exactly which file to edit for a given task)
2. The Platform Framework repo (shared libraries, naming conventions, schema-creation functions)
3. The Algorithm Playbook repo
4. Live Snowflake metadata (via Coco/Cortex) — what feature tables exist, what's already registered
5. Glean-indexed institutional knowledge — past project decisions, so it doesn't contradict precedent

This likely means: a system prompt / project-level instructions document that says "here's the framework, here's the playbook, here's how to reason before generating code" — this could itself be Phase 1 deliverable.

---

## 8. MVP Plan (Small Team, Prove Value Fast)

Given "everything must be planned, nothing missing" but MVP execution:

**MVP scope = one pilot ML project run end-to-end through the template repo, exercising a thin version of all 7 pillars**, not each pillar built to full depth. This proves the shape of the whole framework before hardening any one piece.

| Phase | Deliverable | Est. focus |
|---|---|---|
| 0 | Finalize this scoping doc + naming/standards conventions doc + environment/RBAC decisions | Planning |
| 1 | Template repo skeleton (`notebooks/` + `src/` structure, empty modules with clear contracts, CI/CD pipeline wired) | Template Repo |
| 2 | `schema.py` — reads golden layer, creates project schema (Pillar 1) | Pillar 1 |
| 3 | `feature_engineering.py` + Feature Store convention (Pillar 2) | Pillar 2 |
| 4 | `registry.py` — Model Registry setup for pilot model (Pillar 3) | Pillar 3 |
| 5 | `evaluate.py` — weights/scores logging template (Pillar 4) | Pillar 4 |
| 6 | `monitoring.py` — basic drift check task on pilot model (Pillar 5) | Pillar 5 |
| 7 | Single health dashboard for pilot, incl. cost tagging (Pillar 6) | Pillar 6 |
| 8 | `score.py` — batch scoring via scheduled task, manual rollback documented (Pillar 7) | Pillar 7 |
| 9 | Algorithm Playbook v0 — 5-8 most-used algorithms, each with backtest + forward-test method specified | Playbook |
| 10 | `validation.py` in template repo implementing the two core backtest methods (k-fold, walk-forward) | Validation |
| 11 | Agent context package wired to template repo, framework repo, and Playbook | Agent |

Once the pilot runs cleanly end-to-end through the template, each pillar gets hardened toward the golden standard in parallel workstreams, and the template repo's contracts (function signatures, expected inputs/outputs per file) get locked down as the enforced standard.

---

## 9. Library & Tooling Standardization

One default per category, with an approved short list of alternatives for edge cases the default doesn't cover well. This is the "approved list" every template file and every Playbook code-block is built against — anything not on this list needs a conscious exception, not a default choice.

| Category | Default | Approved alternatives | Notes |
|---|---|---|---|
| Data manipulation | **Snowpark DataFrame** (`snowflake.snowpark`) | pandas (only for small, already-pulled-local data) | Keeps compute in Snowflake; avoids pulling large golden-layer data client-side |
| Classical ML — tabular | **scikit-learn** + **XGBoost** | LightGBM, CatBoost (for specific data shapes — high cardinality categoricals, large datasets) | Covers most classification/regression work; Playbook entries specify which algorithm maps to which library |
| Time series | **Snowflake Cortex ML Forecasting** (native) | statsmodels, Prophet, sktime (for cases Cortex forecasting doesn't cover — multivariate, custom seasonality) | Default keeps forecasting in-warehouse and consistent with the rest of the stack |
| Hyperparameter tuning | **Optuna** | scikit-learn `GridSearchCV`/`RandomizedSearchCV` (small search spaces only) | Optuna integrates cleanly with MLflow logging |
| Experiment tracking | **MLflow** (native in Snowflake ML) | — (no alternative approved; this is a hard standard so every experiment is comparable) | Every `train.py` run logs to MLflow by convention |
| Model registry | **Snowflake Model Registry** (`snowflake.ml.registry`) | — (no alternative) | Ties directly into Pillar 3 |
| Feature store | **Snowflake Feature Store** (`snowflake.ml.feature_store`) | — (no alternative) | Ties directly into Pillar 2 |
| Drift / monitoring | **Evidently** | scipy.stats (for lightweight custom statistical tests where Evidently is overkill) | Evidently covers most feature/prediction drift needs out of the box |
| Visualization (analysis/EDA) | **matplotlib + seaborn** | plotly (when interactivity is needed, e.g. for the observability dashboard) | Static plots inside notebooks; plotly reserved for dashboard-facing work |
| Testing | **pytest** | — (no alternative) | Required for every `src/` module before merge |
| Code quality | **ruff** (lint + format) | black (formatting only, if a team prefers it) | ruff is faster and covers both linting and formatting |
| Type checking | **mypy** | — | Optional for MVP, required at golden-standard maturity |
| Dependency management | **Poetry** (`pyproject.toml`) | pip + `requirements.txt` (only for the lightest projects) | Poetry gives locked, reproducible environments per project |
| Notebook environment | **Snowflake Notebooks** | Jupyter (local, only if Snowflake Notebooks can't cover a specific need) | Keeps orchestration close to the data and consistent with the Snowpark-first approach |

**How this plugs into what's already built:**
- The **template repo's** `pyproject.toml` pins these defaults, so every new project starts with the approved stack already installed.
- The **Algorithm Playbook** code-blocks are written using this exact library set — an algorithm's code-block for, say, XGBoost assumes scikit-learn's API conventions, MLflow logging, and Snowflake Model Registry registration are already wired in.
- The **agent** only ever generates code using this list unless explicitly told to deviate, which keeps its output predictable and reviewable.

---

## 10. Gap Analysis

Everything designed so far covers the *build* — schema, features, registry, storage, monitoring, template, library standards. What's still missing is what makes a framework survive contact with a real team and real production models.

| Area | Gap | Risk if unaddressed | Fix |
|---|---|---|---|
| **Environments** | No dev/staging/prod separation defined for Snowflake schemas, feature store, or registry | Analysts experiment directly against what could become production objects; no safe place to break things | Define a 3-tier schema naming/permission convention (`DEV_`, `STG_`, `PROD_`) before Phase 1; template repo's `config.py` should read environment from a variable, not hardcode it |
| **Deployment / serving** | Model Registry pillar stops at "model is registered" — no defined path to actually serving predictions in production | Every project reinvents how a model goes live (batch job? scheduled task? UDF?) — exactly the fragmentation you're trying to eliminate | Add a **Pillar 7: Deployment & Serving** — standardize on Snowflake scheduled tasks for batch scoring and Snowpark UDFs for on-demand scoring as the two approved patterns |
| **CI/CD** | Template repo has `tests/` but no defined pipeline that actually runs them, lints code, or gates merges | Standards exist on paper but aren't enforced; drift between "the standard" and what's actually in analysts' repos | Add a GitHub Actions (or equivalent) pipeline to the template: run pytest + ruff on every PR, block merge on failure |
| **Access control / security** | No RBAC model for who can write to feature store, register models, or read PII-containing golden layer columns | Data governance/compliance exposure; feature store and registry become a free-for-all | Define Snowflake role hierarchy per pillar (e.g. `FEATURE_STORE_WRITER`, `MODEL_REGISTRY_APPROVER`) as part of Phase 0 |
| **Rollback** | No procedure for "the model we promoted is bad" or "a feature update broke five downstream projects" | Incidents become ad hoc firefighting instead of a known playbook | Registry should track a "previous stable version" pointer; write a one-page rollback runbook per pillar |
| **Cost governance** | Scheduled drift checks, training jobs, and notebooks all consume Snowflake compute with no monitoring standard | Costs scale silently with adoption; no visibility until the bill is a problem | Add compute usage tagging (by project/pillar) to the observability dashboard (Pillar 6) from day one, not as an afterthought |
| **Shared library versioning** | ~~`platform_framework` (schema functions, registry helpers) has no semantic versioning plan~~ **Resolved** — `platform-framework/VERSION` + each project's `.framework-version` file record which snapshot a project was copied from; `scripts/check_framework_drift.py` diffs a project against the current canonical version on demand | A framework update silently breaks every existing project depending on it | — |
| **Feature registration drift** | `register_features` is intentionally duplicated (canonical copy in `platform-framework/feature_store.py`, self-contained copy in every project's `feature_engineering.py`) with nothing to catch silent divergence | Copies quietly diverge over time; a bug fixed in one place stays broken everywhere else | **Resolved** — `check_framework_drift.py` also diffs this specific function's body across the two locations, not just whole-file comparisons |
| **Feature store governance** | No process to prevent duplicate/near-duplicate features or deprecate stale ones | Feature store degrades into the same sprawl it was meant to fix | Lightweight review step before a new feature is registered (even just a PR reviewer checking for existing equivalents) |
| **Agent output trust** | No defined checkpoint for validating agent-generated code before it's merged/run against real data | Agent hallucination or a stale Playbook read produces subtly wrong code that "looks" compliant | Agent-generated code always goes through the same PR + pytest + review gate as human-written code — no bypass |
| **Adoption / change management** | No onboarding plan, training, or feedback loop for the analysts who'll actually use this | This is the biggest real risk — PBI succeeded partly because it was *easy*; a technically excellent framework nobody adopts is a failed project | Build onboarding docs + a short internal training session into the MVP deliverables, not as a "later" item |
| **Success metrics** | No quantitative definition of what "working" looks like | Impossible to know if the pilot succeeded or to justify expanding beyond MVP | Define 3-5 KPIs before Phase 0 finishes (see Section 10) |
| **Pilot selection criteria** | "Pick a real, low-risk use case" is a principle, not a filter | Wrong pilot choice (too complex, too trivial, or too political) undermines the whole proof-of-concept | Score candidate pilots against: data readiness, stakeholder patience, moderate-not-trivial complexity, willingness of the analyst to be "first" |

---

## 11. How to Ensure Success

**1. Define success numerically before you build anything.** Suggested KPIs for the pilot:
- Time from "project kickoff" to "first model registered" (baseline vs. framework-assisted)
- % of the seven pillars the pilot project actually exercises end-to-end without manual workarounds
- Number of PR review comments related to "didn't follow the standard" (should trend toward zero over time)
- Analyst satisfaction/friction score after using the template (a quick survey is enough)

**2. Stage the rollout — don't go straight to "full team."**
Pilot (1 analyst, 1 project) → early adopters (2-3 analysts, their own real projects) → full team rollout. Each stage should feed fixes back into the template/framework before the next stage starts.

**3. Make the golden path the easy path.** If following the framework is more friction than not, adoption fails regardless of how good the architecture is. The template repo + agent should make the compliant way *faster* than going rogue, not just "correct."

**4. Put a human gate on everything the agent generates**, at least until you have enough track record to trust it — same PR/test/review process as human code. This also builds the team's trust in the agent incrementally rather than asking for blind trust up front.

**5. Assign explicit ownership now, not later.** At minimum: one person accountable for the Platform Framework repo, one for the Algorithm Playbook's content quality, and one for the agent's context/instructions. Ambiguous ownership is one of the most common reasons internal platforms decay.

**6. Version everything that other things depend on**: the template repo, the shared framework library, and the Playbook's code-blocks. Treat them like a product with a changelog, not a folder that mutates silently.

**7. Build the feedback loop into the MVP, not after it.** A lightweight mechanism (even a shared doc or Slack channel) for analysts to flag "the standard doesn't fit this case" — otherwise you'll only find out the framework has gaps when people quietly stop using it.

---

## 12. Locked Decisions

Resolved and finalized — Claude Code and any future contributor should treat these as settled, not open:

### 12.1 Repo layout
Three repos, GitHub org-hosted:
- `ml-project-template` — the analyst starting point (self-contained, no external runtime dependency)
- `platform-framework` — canonical, versioned source of shared infra modules; **copied into new projects at creation time**, not submoduled or pip-installed. See `platform-framework/README.md` for the full rationale and tradeoff.
- `ml-algorithm-playbook` — the algorithm knowledge base

### 12.2 Naming convention
| Object | Convention | Example |
|---|---|---|
| Structural repos | fixed names above | `ml-project-template` |
| Project repos | `ml-<domain>-<short-name>`, kebab-case | `ml-retention-churn-risk` |
| Snowflake schema | `<PROJECT_NAME>`, UPPER_SNAKE_CASE, matches repo short name | `CHURN_RISK` |
| Feature Store views | `FEATURE_STORE.<DOMAIN>.<PROJECT_NAME>_FEATURES` | `FEATURE_STORE.RETENTION.CHURN_RISK_FEATURES` |
| Model Registry entries | `<PROJECT_NAME>`, versions `v1, v2, ...`, `stable` tag via comment | `CHURN_RISK`, version `v3`, comment `stable` |
| Monitoring tables | centralized, not per-project | `MONITORING.DRIFT_LOG`, `MONITORING.SHADOW_PREDICTIONS` |
| Git branches | `main` (protected), `feature/<short-desc>`, tags `vX.Y.Z` for releases | `feature/add-drift-check`, `v1.2.0` |

### 12.3 Ownership
- **Platform Admin (you, for now)**: owns `platform-framework`, `ml-project-template`, `ml-algorithm-playbook` governance, RBAC/CI-CD service accounts, and Model Registry approval in Prod.
- **Individual analysts**: own their own project repo day-to-day. PRs against the three structural repos require Platform Admin review.
- Ownership is single-threaded through you deliberately for the MVP — revisit and delegate as the team using this framework grows.

### 12.4 Governance process (MVP-scale)
New Playbook algorithm or Feature Store entry → PR against the relevant repo → Platform Admin review against the fixed template (Section 6) → merge. No separate approval board needed at this scale; add one if/when contribution volume outgrows a single reviewer.

### 12.5 Exception process
Deviating from a "no alternative" library standard (MLflow, Model Registry, Feature Store, pytest) requires a one-line justification in the PR description and Platform Admin sign-off — logged, not silently allowed.

