# Research Plan

## 1. Problem

Human-pose estimation in competitive diving is difficult precisely where the motion becomes most informative.

Conventional pose estimators often perform adequately while the head and upper body remain visually above the pelvis and the legs remain in familiar lower-body configurations. Performance can deteriorate when the diver enters deep flexion, inversion, tuck/pike positions, twists, severe self-occlusion, motion blur, rapid opening, or near-vertical entry.

The project therefore treats difficult frames as observations of a continuous hidden biomechanical state rather than as independent images.

## 2. Working model

A conceptual latent state may contain quantities such as

- joint angles and angular velocities,
- segment geometry,
- centre-of-mass position and velocity,
- global body orientation,
- angular velocity,
- approximate angular momentum,
- and dive phase.

Each video frame is an incomplete and sometimes highly ambiguous observation of that state.

## 3. Research strategy

The project will proceed incrementally.

### Phase A — Dataset acquisition and audit

Obtain authorized research access to useful diving datasets and inspect:

- video availability,
- pose annotations,
- segmentation masks,
- phase annotations,
- frame rates,
- resolutions,
- dive labels,
- and licensing / redistribution constraints.

### Phase B — Baseline failure map

Run an existing open pose estimator on a deliberately difficult subset of dives.

Measure when and how the baseline fails.

### Phase C — Training-free temporal reconstruction

Test whether sequence-level smoothing and interpolation can recover missing or unstable joints.

### Phase D — Biomechanical constraints

Add soft constraints for:

- fixed segment lengths,
- plausible joint angles,
- angular velocity,
- angular acceleration,
- and trajectory continuity.

### Phase E — Dive-phase priors

Use approximate phase information and sport-specific pose/timing ranges to reduce the set of plausible reconstructions.

### Phase F — Lightweight flight physics

Add selected physical constraints during airborne motion, initially without full muscle or contact-force simulation.

### Phase G — Synthetic data, only if justified

A synthetic sequence generator will be built only if earlier experiments show that additional targeted training data are likely to help.

Synthetic generation should focus on actual failure modes rather than uniformly sampling the entire anatomically possible pose space.

## 4. Why synthetic data is not Phase A

Generating millions of anatomically possible poses can create a large but poorly targeted dataset.

Before building such a generator, the project should determine:

- which real poses cause failure,
- which sequence constraints already solve part of the problem,
- what ground truth is missing,
- and which synthetic cases would add information.

## 5. Evaluation philosophy

Whole-video averages can hide the most important errors.

Evaluation should separately report performance in critical windows such as:

- final take-off compression,
- first inversion,
- tight tuck / pike,
- heavy self-occlusion,
- motion blur,
- opening,
- entry alignment,
- and final visible frames before water contact.

## 6. Open-science boundary

The project aims to make methods, code, configurations, experiment definitions, and permitted results public.

Restricted source data will not be redistributed.
