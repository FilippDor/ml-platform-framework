# platform-framework

Canonical, versioned source of the shared infra modules used across every
ML project: `config.py`, `schema.py`, `registry.py`, `validation.py`,
`monitoring.py`, `feature_store.py`.

- **Not a runtime dependency.** Projects get a full copy of these files at
  creation time (via the `new-ml-project` skill), not a submodule or pip
  install. See README.md for the full rationale.
- Changes here don't automatically reach existing projects — propagation
  is a deliberate, per-change decision, logged in this repo's changelog.
- Any change to a function signature here is a breaking change for every
  project that copied it before the change — call this out explicitly
  when making one, and note it in the changelog.
- Same one-job-per-file convention as `ml-project-template` — don't
  combine responsibilities into one module.
