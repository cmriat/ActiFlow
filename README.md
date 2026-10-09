# ActiFlow

[![CVPR 2027](https://img.shields.io/badge/CVPR_2027-Intended_Submission-blue)](https://cvpr.thecvf.com/Conferences/2027)
[![Code](https://img.shields.io/badge/Code-Coming_Soon-orange)](https://github.com/cmriat/ActiFlow)

**Official codebase for our CVPR 2027 submission. Code and models are coming soon — stay tuned.**

Action-conditioned flow world models for long-horizon robot execution, on the RSI (Recursive Self-Improvement) + embodied WAM agenda:

- **Main deliverable**: `W(x_initial, x_history, commanded_actions) → future_video` — initialized from raw Wan2.2-TI2V-5B, reproducing the public BWM dual-path action-conditioning structure, then SFT.
- **A · Observable goal verification**: dual-view RGB + proprioception + goal encoding → per-goal completion probability (revocable), with fallback to the frozen FIXED160 rule under low confidence.
- **B · Action-conditioned consequence prediction**: at chunk boundaries, generates short-horizon futures for both "continue current subgoal" and "switch to next subgoal", scored by a frozen completion estimator.

> The architecture is a public BWM adaptation; we make no originality claim on the network itself. Code is being migrated incrementally per `docs/plan/`.

## Repository layout

```
actiflow/               Main Python package
├── world_model/        The engine of B: action-conditioned video world model (Wan2.2 + BWM)
│   ├── models/         Wan2.2 DiT deltas, dual-path action encoder (context + AdaLN modulation)
│   ├── pipelines/      Denoising pipeline, history/action indexing, autoregressive rollout
│   ├── training/       SFT trainer, flow-matching loss, ACTION/NULL arms
│   └── inference/      predict / autoregressive generation / reload verification
├── goals/              A: goal verification
│   ├── completion/     Completion estimator (frozen visual features + small MLP, revocable)
│   └── decomposition/  Language goal decomposition (inherits U_full400 gains)
├── control/            Controllers
│   ├── fixed160/       Strongest verified baseline (deployment fallback)
│   ├── a_only/         A-only switching (falls back to FIXED160 on low confidence)
│   └── ab_joint/       A+B joint: continue/switch candidate scoring, frozen margin/lambda
├── data/               Data layer
│   ├── robotwin/       RoboTwin HDF5 (14D commanded joint targets, main training)
│   ├── libero/         LIBERO (7D OSC, unlocked by the conditional bridge)
│   └── normalization/  Train-only p01/p99 normalization with clip logging
├── evaluation/         Four-level acceptance & metrics (see below)
└── utils/

configs/                Training/eval configs (world_model / completion / control)
scripts/                CLI entry points (train / eval / data)
docs/
├── plan/               Design docs (overview, action model, final A+B, tasks & acceptance)
└── paper/              Claim boundaries (what we can / cannot claim)
experiments/            EXP-NNN records and artifact pointers (large files excluded)
tests/
third_party/bwm/        Minimal isolated reference copy of official BWM (pinned commit)
```

## World-model acceptance levels

`INTERFACE_READY` → `TRAIN_FIT` → `ACTION_USEFUL_HELDOUT` → `BRANCH_VALIDATED`

Only after `BRANCH_VALIDATED` may the model be plugged into A+B decisions.

## Deployment fallback

Whenever a new candidate fails its gate, deployment falls back to the frozen FIXED160 controller.
