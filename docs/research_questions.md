# Research Questions

## Primary problem

Can competitive-diving pose reconstruction remain reliable during frames in which conventional frame-wise human-pose estimators become ambiguous or fail?

The project focuses specifically on difficult configurations such as deep take-off flexion, inversion, tuck/pike positions, twisting, self-occlusion, motion blur, opening, and water entry.

## RQ1 — Failure mapping

**Where and how do current pose estimators fail during competitive diving?**

Measure failure by dive phase and failure type rather than only by whole-video average accuracy.

Candidate failure categories include:

- missing joints,
- low-confidence detections,
- left/right swaps,
- anatomically implausible bone-length changes,
- identity or body-part swaps,
- orientation errors,
- complete pose loss,
- and temporally discontinuous jumps.

## RQ2 — Temporal reconstruction

**Can temporal continuity recover plausible joint trajectories during short periods of severe ambiguity or missing visual evidence?**

Initial tests should avoid retraining if possible and use existing pose estimates plus sequence-level reconstruction.

## RQ3 — Biomechanical constraints

**How much improvement comes from basic body constraints?**

Candidate constraints include:

- approximately constant segment lengths,
- joint-angle ranges,
- joint angular-velocity limits,
- joint angular-acceleration limits,
- and temporal smoothness.

Constraints should initially be soft penalties rather than hard rules.

## RQ4 — Diving-phase priors

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

Phase priors should be represented as ranges or distributions, not a single idealized trajectory.

## RQ5 — Flight physics

**Can lightweight physical constraints improve airborne orientation and trajectory reconstruction?**

Initial physical priors may include:

- approximately ballistic centre-of-mass motion,
- temporal continuity of global orientation,
- approximate angular-momentum conservation during flight,
- and the relation between body configuration, moment of inertia, and angular velocity.

The project should avoid unnecessary full-body muscle simulation until simpler constraints have been tested.

## Success criterion for the first research phase

The central hypothesis receives support if critical-frame reconstruction improves consistently across the planned ablations:

```text
baseline
→ + temporal
→ + body geometry
→ + dive phase
→ + flight physics
```

A negative result is also useful: if a layer adds no measurable improvement, later work should not assume that layer is necessary.
