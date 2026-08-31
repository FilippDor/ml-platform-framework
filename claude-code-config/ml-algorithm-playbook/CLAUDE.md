# ml-algorithm-playbook

The knowledge base every ML project's algorithm choice, metrics, and
validation approach should be pulled from — not decided ad hoc.

- Every algorithm entry follows the fixed template (see `README.md` and
  the `add-playbook-entry` skill): problem fit, pros/cons, required
  metrics, data assumptions, when to prefer it, **backtest method**,
  **forward-test requirement**, and a linked code block.
- Backtest method must point to the correct file in
  `/validation-frameworks/` — time-dependent problems get
  `time-series-backtesting.md`, never plain k-fold.
- Code blocks in `/code-blocks/` must use only the approved library set
  from the platform scoping doc — no exceptions without a logged
  justification.
- New entries or changes go through a PR reviewed by the Platform Admin —
  no separate approval board at this scale.
