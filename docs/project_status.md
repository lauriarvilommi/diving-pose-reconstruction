# Project Status

## Current stage

**Stage 0 — Research setup, data acquisition, and pre-data baseline-protocol freeze**

## Current working observation

Preliminary tests with conventional pose models suggest a catastrophic failure mode in difficult diving configurations:

- the athlete may cease to be retained as a coherent person target,
- joints may be hallucinated on background structures such as rails or advertising boards,
- or the approximate flight path may remain plausible while articulated joint locations become unstable noise.

This observation has changed Experiment 001 from a simple "pose accuracy" baseline into a hierarchical failure-localization experiment.

## In progress

- request access to FineDiving-family datasets,
- clarify FineDiving-Pose usage terms,
- freeze Experiment 001 before detailed dataset inspection,
- separate athlete retention from articulated-pose failure,
- define whole-frame / oracle-crop / foreground / background controls,
- define tracker-only and global-state baselines,
- define critical-frame failure labels and timing,
- select baseline pose estimator(s),
- establish data and provenance policy.

## Baseline protocol status

**Version:** v0.1 draft / pre-data freeze candidate

Current diagnostic sequence:

```text
B0a whole-frame detector + pose
B0b oracle athlete bounding box + same pose model
B0c oracle foreground mask + same pose model
B0d background-only negative control
B0e temporally tracked athlete crop + same pose model
T0  tracker-only baseline
G0  global-state / silhouette baseline
```

Reconstruction sequence after localization control:

```text
B1 + temporal pose reconstruction
B2 + projection-aware body geometry / biomechanics
B3 + joint and dive-phase priors
B4 + camera-aware flight physics
```

## Important methodological corrections

- Do not assume constant observed 2D bone lengths.
- Distinguish world-space athlete motion from camera-induced image motion.
- Prefer source/session/camera-grouped held-out evaluation over purely random dive splits.
- Distinguish causal reconstruction from offline smoothing where practical.
- Treat model confidence and geometric plausibility as different quantities.

## Not started

- dataset inspection after access approval,
- baseline inference on the target corpus,
- manual critical-frame annotation,
- temporal reconstruction implementation,
- biomechanical optimization,
- phase-prior modeling,
- flight-physics modeling,
- synthetic sequence generation.

## Current decision

Do **not** build a large synthetic-data pipeline before the real failure modes have been measured and localized.

## Next milestone

1. send dataset-access requests,
2. freeze `baseline_protocol.md` v0.1 before inspecting the requested datasets in detail,
3. obtain / audit the initial datasets,
4. select the Experiment 001 corpus using the predeclared sampling principles,
5. run the diagnostic B0/T0/G0 baselines.
