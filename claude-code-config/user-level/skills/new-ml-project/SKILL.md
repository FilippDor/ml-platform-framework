---
name: new-ml-project
description: Scaffold a new ML project by cloning ml-project-template and filling in project-specific config. Use when starting a new ML project for the EDA team.
---

Scaffold a new ML project. Steps:

1. Ask the user for: project name (kebab-case, e.g. `churn-risk`), domain
   (e.g. `RETENTION`), project type (`classification` | `regression` |
   `forecasting` | `clustering`), and owner (email/handle) — unless already
   given in the conversation.
2. Clone `ml-project-template` from the GitHub org into a new local
   directory named `ml-<domain-lowercase>-<project-name>`.
3. Remove the template's `.git` history and `git init` a fresh repo, so
   this becomes a new independent repo, not a fork tracking the template.
4. Fill in **`project.yaml`** at the repo root — the single config file that
   drives every module and script. Set `project.name` (UPPER_SNAKE_CASE,
   matches Section 12.2 of the platform scoping doc), `project.owner`,
   `project.domain`, `project.type`, and `environment: DEV`. Keep the
   `snowflake:` block on the platform defaults unless the account differs.
   Fill the `features:` block from the project's real source columns.
   Do **not** put secrets here — the Snowflake password lives in
   `~/.snowflake/connections.toml`. `.env` is optional and holds only the
   three env-var overrides (see `.env.example`).
5. Copy `platform-framework/VERSION`'s current content into this new
   project's `.framework-version` file — overwrite the template's stale
   copy with whatever `platform-framework` is actually at right now. This
   is the only record of which framework snapshot the project started
   from; don't skip it.
6. Check `ml-algorithm-playbook` for an existing entry matching this
   project's type before writing any training code — don't hand-roll an
   algorithm choice the Playbook already covers.
7. Confirm `pyproject.toml`'s pinned libraries still match the current
   platform-framework library standardization table before running
   `poetry install` — flag any drift to the user rather than silently
   using an outdated pin.
8. Run `pytest` once to confirm the scaffold is healthy before handing
   back to the user.

Do not invent new file structure — every module in `src/` has one job
(see the template's README.md). If a task doesn't fit an existing file,
ask the user before creating a new one.
