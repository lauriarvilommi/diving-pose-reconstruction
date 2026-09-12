# Experiment 001 — Critical-Frame Pose Reconstruction

## Purpose

Test the core hypothesis as cheaply as possible before building a large synthetic-data generator or training a new pose model.

## Hypothesis

A sequence-aware reconstruction using temporal and biomechanical constraints can improve pose estimates specifically in the frames where a strong frame-wise baseline becomes unreliable.

## Initial corpus

Target size: approximately **50–100 individual dives**, subject to dataset access and content audit.

The corpus should be deliberately enriched for difficult conditions rather than sampled uniformly.

Desired variation:

- springboard and platform,
- forward / backward / reverse / inward families where available,
- tuck / pike / straight positions,
- twisting and non-twisting dives,
- deep take-off flexion,
- strong inversion,
- self-occlusion,
- motion blur,
- rapid opening,
- and vertical entry.

Synchronized diving should initially be excluded because two-person overlap introduces a separate tracking problem.

## Development split

A first practical split may be approximately:

- 40 dives for development / parameter selection,
- 20 dives held out for evaluation,

with the exact split chosen only after the available dataset has been audited.

## Baseline

Use a strong open human-pose estimator without diving-specific retraining.

The exact baseline model will be selected after a reproducibility and licensing audit.

## Critical landmarks

A first observation-oriented event scheme may include:

- `P0` — last clearly reliable frame before the difficult take-off region,
- `P1` — onset of deep flexion / inversion,
- `P2` — take-off / loss of support,
- `P3` — first clearly identifiable airborne pose,
- `P4` — strongest occlusion / blur / inversion region,
- `P5` — opening begins,
- `P6` — entry alignment begins,
- `P7` — last useful pre-contact frame,
- `P8` — first water contact.

These are experimental landmarks, not a claim that they form the final biomechanical ontology of diving.

## Ablation sequence

### E1.0 — Frame-wise baseline

Measure raw keypoints and confidence.

### E1.1 — Temporal continuity

Add sequence-level temporal reconstruction or smoothing.

### E1.2 — Body geometry

Add approximately constant segment lengths.

### E1.3 — Joint constraints

Add plausible joint-angle, angular-velocity, and angular-acceleration penalties.

### E1.4 — Dive-phase priors

Use phase-dependent constraints or distributions.

### E1.5 — Flight physics

Add selected airborne physical constraints.

## Metrics

Do not rely only on a single whole-video pose metric.

Measure at least:

- joint localization error where ground truth is available,
- missing-joint rate,
- left/right swap rate,
- segment-length inconsistency,
- temporal discontinuity,
- orientation error where definable,
- and failure-window-specific accuracy.

## Failure labels

Each critical interval should record relevant causes such as:

- `occlusion`,
- `self_occlusion`,
- `motion_blur`,
- `inversion`,
- `deep_flexion`,
- `twist`,
- `left_right_ambiguity`,
- `out_of_frame`,
- `entry_alignment`,
- `pose_lost`.

## Decision rule

Only proceed to a large synthetic-data generator after the experiment identifies specific residual failure modes that existing data and sequence constraints do not adequately solve.
