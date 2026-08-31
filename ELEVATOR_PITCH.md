# ML Platform Framework — Elevator Pitch

**The problem:** the EDA team builds ML the way most teams do before they have a platform — every project reinvents its own schema, its own feature logic, its own way of registering and scoring a model. There's no shared standard for validation, no consistent path to production, and no way to answer "which of our models are actually reliable right now" without checking each one by hand.

**The idea:** give ML the same experience the team already has with Power BI — a standardized way to build, backed by shared infrastructure, so an analyst's time goes into solving the business problem, not rebuilding plumbing.

**What that means concretely:**
- A **project template** every new ML effort starts from — schema, features, training, validation, registry, scoring, and monitoring are already wired correctly, self-contained, and running the moment it's cloned.
- A **shared framework** of proven infrastructure code — versioned, so improvements can be tracked and deliberately rolled out rather than silently drifting apart across projects.
- An **algorithm playbook** that encodes which approach fits which problem, including the two things most commonly done wrong: the correct way to backtest a model without leaking future data into the past, and the forward-testing period required before a model is trusted with real decisions.
- A **clean promotion path** from Dev (where all real experimentation happens, on real data, under existing access controls) to Prod (a separate, tightly controlled environment) — one tagged release, one automated pipeline, full traceability back to the exact reviewed code that's running.
- **Drift monitoring** that doesn't just flag "something changed" — it ranks which features moved, why they likely moved, and gives a plain-language summary ready to bring into a conversation with the business, not just a data-team dashboard.

**Why now:** the tech stack (Snowflake, native ML/registry/feature store tooling, Git, Claude) already supports this — the gap isn't tooling, it's standardization. Every week without it is another project's worth of one-off decisions that won't transfer to the next one.

**What's built:** a working project template, a versioned framework repo, an initial governance and naming standard, and the Claude Code tooling to scaffold new projects and enforce the standard automatically going forward.

**The ask:** a pilot project to prove the full path end-to-end — Dev to Prod, one real business problem — before rolling this out as the team's default way of building ML.
