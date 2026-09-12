# Research Questions

## Primary problem

Can competitive-diving pose reconstruction remain reliable during frames in which conventional human-pose pipelines lose either the athlete, the athlete's global body state, the orientation interpretation, or the articulated pose?

The project focuses specifically on difficult configurations such as deep take-off flexion, inversion, tuck/pike positions, twisting, self-occlusion, motion blur, opening, and water entry.

A central methodological decision is to separate:

```text
target retention
from
orientation robustness
from
articulated-pose reconstruction
```

rather than treating every failure as generic keypoint error.

## RQ1 — Failure hierarchy

**At which level does the baseline first fail?**

Distinguish at least:

- person detection failure,
- target-identity loss,
- global-state retention with articulation collapse,
- orientation-prior failure,
- and structured but semantically incorrect pose.

This determines whether the main bottleneck is localization, tracking, orientation robustness, pose recognition, or later reconstruction.

## RQ2 — Controlled localization interventions

**If the same pose model is given progressively stronger localization information, which failure modes disappear?**

Compare:

- whole-frame inference,
- oracle athlete bounding box,
- oracle foreground mask,
- background-only negative control,
- and temporally propagated / tracked athlete crop.

This is intended to distinguish a true extreme-pose failure from a person-detection or target-retention failure.

## RQ3 — Background-induced hallucination

**Does the baseline produce confident human-pose predictions from background structures after the real athlete becomes visually atypical?**

The background-only control is specifically intended to test whether rails, advertising boards, architecture or other structures trigger false pose hypotheses.

## RQ4 — Rotational equivariance

**How strongly does a pose model depend on the conventional upright image orientation of the human body?**

Use a known-good athlete crop and rotate the *pixels* synthetically while preserving the same human configuration.

For image rotation \(R_\theta\) and pose estimator \(f\), test whether the model approximately satisfies:

```text
f(Rθ I) ≈ Rθ f(I)
```

after transforming predictions into a common coordinate system.

The purpose is to isolate orientation shift from:

- articulation change,
- motion blur,
- self-occlusion,
- tracking failure,
- and dive-phase change.

## RQ5 — Oracle orientation canonicalization

**If a difficult diving crop is rotated into a canonical image-plane orientation using an externally supplied orientation, does articulated-pose estimation improve?**

Compare:

- difficult crop in its original image orientation,
- the same pixels rotated using oracle orientation,
- and later, the same operation using estimated orientation.

This separates two questions:

1. can the pose model work after image-space canonicalization?
2. can an automatic system estimate a useful canonicalization angle?

The first question must be tested before the second.

## RQ6 — Temporal athlete retention

**Can a tracker retain the same athlete through the critical interval even when articulated pose estimation fails?**

This test intentionally asks less than pose estimation.

Success would show that:

```text
athlete tracking succeeds
while
articulated pose recognition fails
```

and would justify decoupling the two tasks.

## RQ7 — Temporal articulated reconstruction

**Once athlete localization and orientation effects are controlled, can temporal information recover a coherent pose during short intervals of severe ambiguity or missing visual evidence?**

Two modes should eventually be distinguished:

- causal / online reconstruction using only past and current frames,
- offline smoothing using past and future frames.

For post-performance coaching analysis, offline smoothing is a valid and potentially stronger setting.

## RQ8 — Projection-aware biomechanical constraints

**How much additional improvement comes from body geometry and biomechanical constraints once camera projection is accounted for?**

Candidate constraints include:

- fixed underlying 3D segment geometry,
- joint-angle ranges,
- angular-velocity limits,
- angular-acceleration limits,
- and temporal consistency.

The project must **not** assume constant observed 2D segment length, because 3D rotation and camera motion can change image-plane lengths substantially.

Constraints should initially be soft penalties rather than hard rules.

## RQ9 — Diving-phase priors

**Does knowledge of the current diving phase reduce pose ambiguity?**

Possible phase information includes:

- take-off preparation,
- deep flexion / compression,
- take-off,
- flight closing,
- tuck / pike / straight flight,
- twist,
- opening,
- entry alignment,
- and water contact.

Phase priors should be represented as ranges or distributions rather than a single idealized trajectory.

## RQ10 — Camera-aware flight physics

**Can lightweight physical constraints improve airborne trajectory and orientation reconstruction after camera motion has been separated from athlete motion?**

Initial physical priors may include:

- approximately ballistic centre-of-mass motion in world coordinates,
- temporal continuity of global orientation,
- approximate angular-momentum conservation during flight,
- and the relation between body configuration, moment of inertia, and angular velocity.

Broadcast-image pixel trajectories must not be treated as directly ballistic when the camera pans, tilts or zooms.

## RQ11 — Failure onset and recovery

**When does catastrophic failure begin, how long does it persist, and does the baseline recover the correct athlete and pose before water entry?**

Candidate temporal outcomes include:

- first catastrophic-failure frame,
- duration of failure,
- spontaneous recovery time,
- and whether recovery occurs before or only after a major phase transition.

## Success criterion for the first research phase

The first phase succeeds scientifically if it can:

1. localize the dominant failure layer,
2. quantify catastrophic target-loss and articulation-collapse events,
3. quantify orientation sensitivity independently of articulation change,
4. determine whether image-space canonicalization removes any part of the extreme-pose failure,
5. determine which controlled interventions remove which failures,
6. and then test the reconstruction ablation without changing the evaluation rules after the data have been inspected.

A negative result is also useful. If oracle image-space canonicalization does not help difficult diving poses, orientation shift can be deprioritized relative to articulation, occlusion and other mechanisms.
