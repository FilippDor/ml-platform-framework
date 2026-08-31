---
name: add-pillar-module
description: Scaffold a new module in ml-project-template's src/ or a shared module in platform-framework, following the one-job-per-file convention. Use when extending the platform's pillar coverage.
---

Add a new pillar-related module. Before writing anything:

1. Confirm which repo this belongs in: is this logic identical across
   every project (→ `platform-framework`, and note it must later be
   copied into `ml-project-template` as a self-contained snapshot — see
   `platform-framework/README.md` for why it's copy, not submodule/import),
   or project-specific (→ the individual project's `src/`)?
2. One file, one job — match the existing pattern in `src/` (`schema.py`,
   `registry.py`, etc.). Don't add a second responsibility to an existing
   file; create a new one.
3. Docstring at the top of the file states which Pillar (1-7) this
   belongs to, matching the platform scoping doc's pillar definitions.
4. Add a corresponding test in `tests/` before considering this done —
   the CI gate requires it.
5. If this changes a function signature another module depends on
   (e.g. `ProjectConfig`), update every call site in the same PR, not
   as a follow-up.
