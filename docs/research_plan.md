# Research Plan

## 1. Problem

Human-pose estimation in competitive diving is difficult precisely where the motion becomes most informative.

Conventional pose pipelines may perform adequately while the diver remains in familiar upright configurations, but performance can deteriorate sharply during deep flexion, inversion, tuck/pike positions, twists, severe self-occlusion, motion blur, rapid opening, or near-vertical entry.

Preliminary tests suggest that the failure may be **hierarchical and catastrophic**:

- the system may first lose the athlete as a coherent target,
- it may then place joints on unrelated background structures,
- or it may retain the approximate athlete trajectory while the articulated pose degenerates into unstable joint-position noise.

The project therefore does not treat all of these outcomes as the same "pose error".

## 2. Working architecture

The project separates the inference problem into layers:

```text
camera / shot state
        ↓
athlete retention
        ↓
global body state
        ↓
articulated pose
        ↓
sequence reconstruction
```

A conceptual latent state may eventually contain quantities such as:

- joint configuration and angular velocities,
- underlying 3D segment geometry,
- centre-of-mass position and velocity,
- global body orientation,
- angular velocity,
- approximate angular momentum,
- dive phase,
- and camera / shot state.

Each video frame is an incomplete and sometimes highly ambiguous observation of that state.

## 3. Why camera state matters

Broadcast footage may contain:

- pan,
- tilt,
- zoom,
- crop changes,
- and tracking motion.

Therefore a ballistic world-space trajectory does not necessarily appear ballistic in image coordinates.

Likewise, fixed 3D segment lengths do not imply constant observed 2D segment lengths when the diver rotates relative to the camera.

The project will therefore keep physical and geometric priors **projection-aware**.

## 4. Research strategy

The project will proceed incrementally.

### Phase A — Dataset acquisition and audit

Obtain authorized research access to useful diving datasets and inspect:

- video availability,
- pose annotations,
- human masks,
- phase annotations,
- frame rates,
- resolutions,
- camera / shot behavior,
- dive labels,
- and licensing / redistribution constraints.

### Phase B — Diagnostic baseline and failure localization

Run one or more strong open pose baselines on a deliberately difficult subset of dives.

Use controlled input interventions to determine whether the failure arises from:

- person detection,
- target retention,
- global-state estimation,
- or articulated-pose estimation.

The core diagnostic controls are:

- whole-frame inference,
- oracle athlete crop,
- oracle foreground mask,
- background-only negative control,
- temporally tracked athlete crop.

### Phase C — Tracker-only and global-state baselines

Test whether the athlete can be retained as an object through the same intervals in which pose estimation collapses.

Where suitable masks are available, evaluate simple global descriptors such as:

- foreground centroid,
- bounding box,
- projected spatial extent,
- and principal body axis.

This phase deliberately avoids solving articulated pose.

### Phase D — Training-free temporal pose reconstruction

Once localization is controlled, test whether sequence-level reconstruction can recover missing or unstable joints.

Separate:

- causal / online reconstruction,
- offline smoothing that may use future frames.

### Phase E — Projection-aware biomechanical constraints

Add soft constraints for:

- underlying 3D segment geometry,
- plausible joint angles,
- angular velocity,
- angular acceleration,
- and temporal consistency.

Do not use constant image-plane segment length as a physical invariant.

### Phase F — Dive-phase priors

Use approximate phase information and sport-specific pose/timing ranges to reduce the set of plausible reconstructions.

### Phase G — Camera-aware flight physics

Add selected physical constraints during airborne motion, initially without full muscle or contact-force simulation.

World-space physics must be separated from camera-induced image motion.

### Phase H — Synthetic data, only if justified

Build a synthetic sequence generator only if earlier experiments show that additional targeted training data are likely to help.

Synthetic generation should focus on measured residual failure modes rather than uniformly sampling the entire anatomically possible pose space.

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

The evaluation must also distinguish:

- loss of the athlete,
- retention of the athlete but loss of articulation,
- and a structured but semantically wrong skeleton.

## 6. Split philosophy

A random dive-level split may leak background, venue, broadcaster, camera and athlete-specific cues into both development and evaluation sets.

Where metadata allows, the held-out evaluation set should therefore be grouped by source/session/camera environment and, where practical, athlete identity.

## 7. Pre-data protocol freeze

The baseline protocol, failure taxonomy, primary metrics and split principles should be documented **before the requested datasets are inspected in detail**.

This reduces the risk of changing the experiment after seeing which measurements make the proposed method look favorable.

## 8. Open-science boundary

The project aims to make methods, code, configurations, experiment definitions, and permitted results public.

Restricted source data will not be redistributed.
