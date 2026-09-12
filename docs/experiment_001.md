# Experiment 001 — Critical-Frame Failure Localization and Pose Reconstruction

## Purpose

Test the core hypothesis as cheaply and diagnostically as possible before building a large synthetic-data generator or training a new diving-specific pose model.

Experiment 001 has two distinct stages:

1. **localize the failure layer**, and
2. **test reconstruction only after localization has been controlled**.

## Main hypotheses

### H1 — Catastrophic failure is not merely keypoint noise

During extreme diving configurations, a conventional pose pipeline may lose the athlete as a coherent target or hallucinate poses on background structures.

### H2 — Target retention and articulation can fail independently

A tracker or global-state estimator may retain the athlete and approximate flight path while articulated pose estimation collapses.

### H3 — Sequence-level constraints can improve reconstruction

Once athlete localization is controlled, temporal, biomechanical, phase and physical information may recover a more coherent pose during critical failure windows.

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

Synchronized diving should initially be excluded because two-person overlap introduces a separate tracking and identity-assignment problem.

## Development and held-out split

The exact split will be chosen only after the available dataset has been audited.

A first practical size may be approximately:

- 40 dives for development / parameter selection,
- 20 dives held out for evaluation.

However, the held-out set should **not** be a purely random sample if that would place the same competition, camera environment, source video or athlete in both sets.

Where metadata permits, use grouped separation by:

1. source video / competition session,
2. camera environment,
3. athlete identity where practical.

## Baseline model principle

Use a strong open human-pose estimator without diving-specific retraining.

The exact model(s) will be selected after a reproducibility and licensing audit.

At least one stronger accuracy-oriented model and one practical/fast model may be useful to determine whether a failure is architecture-specific or general.

## Stage 1 — Diagnostic baseline controls

Use the **same pose model** wherever possible so that input control, not model replacement, is the intervention.

### B0a — Whole-frame detector + pose

The conventional pipeline receives the full frame and must localize the athlete itself.

This reproduces the real failure mode, including possible background-induced hallucinations.

### B0b — Oracle athlete bounding box + same pose model

Provide the correct athlete region while changing nothing else.

Interpretation:

- if B0a fails but B0b succeeds, target localization / retention is a major bottleneck;
- if both fail, the articulated pose model itself is likely out of distribution.

### B0c — Oracle foreground mask + same pose model

Retain only the true athlete foreground, or neutralize the background, while preserving the athlete pixels.

This tests whether clutter and background structures drive the failure.

### B0d — Background-only negative control

Remove / neutralize the athlete while keeping the background.

A confident human-pose prediction in this condition is direct evidence of background-induced pose hallucination.

### B0e — Temporally propagated / tracked athlete crop + same pose model

Initialize from the last reliable pre-failure athlete observation and propagate the target region temporally without requiring the pose model to rediscover the person independently in every frame.

This tests whether target identity can be retained through the failure interval.

### T0 — Tracker-only baseline

Track the athlete as an object without estimating joints.

Primary question:

```text
Can tracking remain correct when articulated pose estimation fails?
```

### G0 — Global-state / silhouette baseline

Where a reliable mask is available, estimate simple global quantities such as:

- foreground centroid,
- bounding box,
- projected extent,
- principal axis / coarse orientation.

This tests whether global body state remains recoverable after articulated pose has collapsed.

## Stage 2 — Reconstruction ablation

Proceed only after Stage 1 has established how localization is controlled.

### B1 — Temporal pose reconstruction

Add sequence-level temporal information to the controlled athlete track.

### B2 — Projection-aware body geometry / biomechanics

Add soft constraints based on:

- fixed underlying 3D segment geometry,
- plausible joint angles,
- angular velocity,
- angular acceleration,
- and temporal consistency.

**Do not impose constant observed 2D bone length.** Perspective, out-of-plane rotation and camera motion can change image-plane segment lengths strongly.

### B3 — Joint and diving-phase priors

Add sport-specific phase information and phase-conditioned pose / timing ranges.

### B4 — Camera-aware flight physics

Add selected airborne physical constraints after accounting for camera motion.

Candidate priors include:

- approximately ballistic centre-of-mass motion in world coordinates,
- global-orientation continuity,
- approximate angular-momentum conservation,
- and configuration-dependent moment of inertia.

## Causal and offline reconstruction

Where practical, report two sequence settings:

### Causal / online

Estimate frame \(t\) using only frames up to \(t\).

### Offline smoothing

Reconstruct frame \(t\) using the complete observed sequence, including later frames.

Offline smoothing is a valid setting for post-performance coaching and research analysis and may recover information that is fundamentally ambiguous in a single frame.

## Critical landmarks

A first observation-oriented event scheme may include:

- `P0` — last clearly reliable pose frame before the difficult take-off region,
- `P1` — onset of deep flexion / inversion,
- `P2` — take-off / loss of support,
- `P3` — first clearly identifiable airborne pose,
- `P4` — strongest occlusion / blur / inversion region,
- `P5` — opening begins,
- `P6` — entry alignment begins,
- `P7` — last useful pre-contact frame,
- `P8` — first water contact.

These are experimental landmarks, not a claim that they form the final biomechanical ontology of diving.

Each landmark should later receive an explicit annotation rule before large-scale annotation begins.

## Failure taxonomy

### F0 — Person detection failure

No usable athlete detection is produced.

### F1 — Target identity loss

The predicted pose / target moves away from the athlete and attaches to a wrong image region or background structure.

### F2 — Global-state survival with articulation collapse

The system approximately retains the athlete location or flight path, but joint positions become unstable or effectively noise-like.

### F3 — Structured but semantically wrong pose

The skeleton remains human-like but is wrong in ways such as:

- left/right swaps,
- limb assignment errors,
- wrong global orientation,
- implausible articulation.

A clip may contain more than one failure type over time.

## Failure timing

Record, where possible:

- first catastrophic-failure frame,
- failure duration,
- spontaneous recovery frame,
- whether recovery occurs before water entry,
- and the dive phase at failure onset.

## Metrics

Do not rely only on a single whole-video pose metric.

Candidate metrics include:

### Athlete-retention metrics

- athlete bbox / mask overlap where ground truth is available,
- normalized distance between predicted pose centroid and athlete region,
- fraction of predicted keypoints inside the athlete foreground,
- target-loss event rate.

### Background-hallucination metrics

- pose detections in background-only controls,
- confidence of those detections,
- distance from the removed athlete's true region.

### Articulation metrics

Where trustworthy pose ground truth is available:

- joint localization error,
- missing-joint rate,
- left/right swap rate,
- orientation error where definable.

### Structural / temporal metrics

- projection-aware geometric plausibility,
- temporal discontinuity,
- velocity / acceleration outliers,
- catastrophic pose-explosion rate.

### Critical-window reporting

Report results separately for:

- all evaluable frames,
- take-off critical windows,
- flight critical windows,
- opening / entry critical windows.

## Camera / shot annotation

At minimum classify the camera behavior relevant to each clip, for example:

- fixed,
- pan / tilt,
- zoom,
- pan / tilt + zoom,
- uncertain.

Do not apply world-space ballistic assumptions directly to raw pixel trajectories without considering camera motion.

## Annotation confidence

Ground truth should distinguish:

- visible joints,
- occluded but inferable joints,
- out-of-frame joints,
- uncertain left/right assignment,
- and annotation confidence.

A manually guessed location of a fully occluded joint should not be treated as equally certain as a clearly visible joint.

## Decision rule

Only proceed to a large synthetic-data generator after Experiment 001 identifies specific residual failure modes that:

1. survive the controlled localization diagnostics,
2. are not adequately solved by temporal / biomechanical reconstruction,
3. and plausibly require targeted additional training data.

See [`baseline_protocol.md`](baseline_protocol.md) for the baseline freeze and implementation-facing details.
