# Research Questions

## Primary problem

Can competitive-diving pose reconstruction remain reliable during frames in which conventional human-pose pipelines lose either the athlete, the athlete's global body state, or the articulated pose?

The project focuses specifically on difficult configurations such as deep take-off flexion, inversion, tuck/pike positions, twisting, self-occlusion, motion blur, opening, and water entry.

A central methodological decision is to **separate target retention from pose estimation** rather than treating every failure as a generic keypoint error.

## RQ1 — Failure hierarchy

**At which level does the baseline first fail?**

Distinguish at least:

- person detection failure,
- target-identity loss,
- global-state retention with articulation collapse,
- and structured but semantically incorrect pose.

This determines whether the main bottleneck is localization, tracking, pose recognition, or later reconstruction.

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

## RQ4 — Temporal athlete retention

**Can a tracker retain the same athlete through the critical interval even when articulated pose estimation fails?**

This test intentionally asks less than pose estimation. Success would show that:

```text
athlete tracking succeeds
while
articulated pose recognition fails
```

and would justify decoupling the two tasks.

## RQ5 — Temporal articulated reconstruction

**Once athlete localization is controlled, can temporal information recover a coherent pose during short intervals of severe ambiguity or missing visual evidence?**

Two modes should eventually be distinguished:

- causal / online reconstruction using only past and current frames,
- offline smoothing using past and future frames.

For post-performance coaching analysis, offline smoothing is a valid and potentially stronger setting.

## RQ6 — Projection-aware biomechanical constraints

**How much additional improvement comes from body geometry and biomechanical constraints once camera projection is accounted for?**

Candidate constraints include:

- fixed underlying 3D segment geometry,
- joint-angle ranges,
- angular-velocity limits,
- angular-acceleration limits,
- and temporal consistency.

The project must **not** assume constant observed 2D segment length, because 3D rotation and camera motion can change image-plane lengths substantially.

Constraints should initially be soft penalties rather than hard rules.

## RQ7 — Diving-phase priors

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

## RQ8 — Camera-aware flight physics

**Can lightweight physical constraints improve airborne trajectory and orientation reconstruction after camera motion has been separated from athlete motion?**

Initial physical priors may include:

- approximately ballistic centre-of-mass motion in world coordinates,
- temporal continuity of global orientation,
- approximate angular-momentum conservation during flight,
- and the relation between body configuration, moment of inertia, and angular velocity.

Broadcast-image pixel trajectories must not be treated as directly ballistic when the camera pans, tilts or zooms.

## RQ9 — Failure onset and recovery

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
3. determine which controlled interventions remove which failures,
4. and then test the reconstruction ablation without changing the evaluation rules after the data have been inspected.

A negative result is also useful. If a layer adds no measurable improvement, later work should not assume that layer is necessary.
