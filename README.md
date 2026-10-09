# ActiFlow

[![CVPR 2027](https://img.shields.io/badge/CVPR_2027-Intended_Submission-blue)](https://cvpr.thecvf.com/Conferences/2027)
[![Code](https://img.shields.io/badge/Code-Coming_Soon-orange)](https://github.com/cmriat/ActiFlow)

Action-conditioned flow world models for long-horizon robot execution (RSI + embodied WAM).

Codebase for our CVPR 2027 submission — code coming soon.

## Layout

- `actiflow/world_model/` — action-conditioned video world model (Wan2.2, dual-path action conditioning)
- `actiflow/goals/` — observable goal completion verification (A)
- `actiflow/control/` — FIXED160 baseline, A-only and A+B joint controllers
- `actiflow/data/` — RoboTwin / LIBERO loaders, train-only normalization
- `actiflow/evaluation/` — acceptance levels and metrics
- `docs/plan/` — design docs; `docs/paper/` — claim boundaries
- `configs/`, `scripts/`, `experiments/`, `tests/`
