---
name: add-playbook-entry
description: Draft a new algorithm entry for ml-algorithm-playbook following the required fixed template. Use when adding a new algorithm to the Playbook.
context: fork
---

Draft a new `ml-algorithm-playbook` entry. The fixed template (non-negotiable
— every field below is required, per Section 6 of the platform scoping doc):

1. Problem types it solves / doesn't solve
2. Pros & cons
3. Required/expected metrics + acceptable thresholds
4. Data requirements & assumptions
5. When to prefer it vs. alternatives already in the Playbook
6. Backtesting method — must reference the correct file in
   `/validation-frameworks/`: `time-series-backtesting.md` for any
   time-dependent problem, `tabular-cross-validation.md` otherwise. Never
   leave this to inference — get it right, since a wrong backtest method
   silently produces an overconfident model.
7. Forward-looking test requirement — shadow/champion-challenger period,
   with a concrete minimum duration or observation count before promotion
   to sole decision-maker.
8. Link to a code block in `/code-blocks/`, written using only the
   approved library set from the platform scoping doc's library
   standardization table — no exceptions without a logged justification.

Place the file at `/algorithms/<family>/<algorithm-name>.md`. Cross-check
against existing entries to avoid duplicating an already-covered case —
if this algorithm is a minor variant of an existing entry, note the
relationship instead of writing a redundant standalone file.
