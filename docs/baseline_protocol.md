# Baseline Protocol v0.2

**Status:** Pre-data freeze candidate  
**Experiment:** Experiment 001 — Critical-Frame Failure Localization and Pose Reconstruction

## 1. Purpose

This protocol defines the baseline experiment **before detailed inspection of the requested diving datasets**.

Its primary purpose is not to maximize pose accuracy. It is to answer:

> **Where does a conventional human-pose pipeline first lose the diving motion, and how much of that failure is caused by image-plane orientation rather than by target loss, articulation or occlusion?**

Preliminary tests indicate that failure may be catastrophic. Once the diver enters highly inverted, compact or blurred configurations, the system may:

- fail to retain the athlete as the same target,
- hallucinate joints on background structures,
- retain only the approximate athlete trajectory,
- produce a structured but semantically incorrect skeleton,
- or fail because a normally recognizable human appearance is presented at an image-plane orientation outside the model's conventional upright prior.

The protocol therefore separates:

```text
camera / shot state
        ↓
athlete retention
        ↓
global body state
        ↓
orientation robustness
        ↓
articulated pose
```

before testing later sequence reconstruction.

---

## 2. Protocol-freeze principle

Before detailed dataset inspection, freeze as far as practical:

- the failure hierarchy,
- localization diagnostic variants,
- orientation diagnostic variants,
- primary failure metrics,
- critical-window concept,
- split principles,
- camera-state recording,
- and the order of the reconstruction ablation.

Dataset-specific thresholds may later require calibration, but any such change should be documented rather than silently replacing the pre-data rule.

---

## 3. Baseline model selection

Select at least one strong open pose estimator that:

- can run reproducibly on stored RGB video frames,
- exposes keypoint coordinates and confidence values,
- has a clearly documented license,
- and does not require diving-specific training.

Where computationally feasible, use two baselines with different operating points:

1. **accuracy-oriented baseline** — e.g. a strong ViTPose-class model;
2. **practical / fast baseline** — e.g. an RTMPose-class model.

The purpose is not to rank all pose models. It is to determine whether the observed failure is:

- specific to one implementation,
- or robust across substantially different pose baselines.

Record:

- model family,
- exact checkpoint,
- source URL / repository,
- commit or package version,
- input resolution,
- detector configuration,
- confidence thresholds,
- and preprocessing.

---

## 4. Required frame-level outputs

For every baseline frame, store at least:

```text
clip_id
frame_index
timestamp
detected_person_bbox
person_detection_score
keypoint_xy[J, 2]
keypoint_confidence[J]
pose_instance_score        # if available
camera_state_label         # when annotated
critical_phase_label       # when annotated
```

Where masks are available:

```text
athlete_mask
athlete_bbox_from_mask
athlete_centroid
```

Where ground-truth pose is available:

```text
gt_keypoint_xy
gt_visibility
gt_annotation_confidence
gt_left_right_ambiguity
```

For orientation diagnostics additionally store:

```text
input_rotation_degrees
canonicalization_source    # none / oracle / estimated
inverse_transform
reference_pose_id
```

Do not place restricted source data in the public repository.

---

## 5. Diagnostic localization matrix

### B0a — Whole-frame detector + pose

**Input:** original full frame  
**Task:** conventional detection and pose estimation

This is the ecological baseline and should reproduce the real catastrophic failure mode.

Record:

- missed person detections,
- wrong-person / wrong-region detections,
- background pose hallucinations,
- articulation quality,
- confidence behavior.

---

### B0b — Oracle athlete bounding box + same pose model

**Input:** true athlete crop / bounding box  
**Task:** same pose model, localization externally supplied

Purpose:

> Does the pose model still fail when it is told exactly where the athlete is?

Interpretation:

```text
B0a fails + B0b succeeds
→ localization / target retention is a major failure source

B0a fails + B0b fails
→ articulated extreme-pose recognition or orientation sensitivity remains a major failure source
```

The oracle box is a diagnostic intervention, not a proposed production solution.

---

### B0c — Oracle foreground mask + same pose model

**Input:** athlete foreground retained, background neutralized  
**Task:** same pose model

Purpose:

- remove rails, advertisements and venue structure,
- test the effect of clutter,
- separate athlete appearance from contextual false positives.

Where technically necessary, test more than one neutralization method so that an artificial mask edge does not itself become the dominant cue.

---

### B0d — Background-only negative control

**Input:** athlete removed / neutralized, original background retained  
**Task:** same pose model

Purpose:

> Does the model produce a confident human skeleton where no athlete remains?

This directly tests **background-induced pose hallucination**.

Record:

- whether a person / pose is detected,
- confidence,
- location,
- skeleton structure,
- relation to the actual athlete's removed region.

A high-confidence detection on rails, advertising boards or architecture is a catastrophic false positive even if its internal skeleton geometry looks plausible.

---

### B0e — Temporally propagated / tracked athlete crop + same pose model

**Initialization:** last reliable athlete observation before the critical failure interval  
**Task:** retain the same target region temporally, then run the same pose model

Purpose:

> If the system already knows which moving object is the diver, does pose estimation still collapse?

The tracking mechanism must not use future ground-truth pose labels.

---

### T0 — Tracker-only baseline

Track the athlete without estimating any articulated joints.

Possible outputs:

```text
bbox_t
centroid_t
track_confidence_t
```

Primary hypothesis:

```text
athlete tracking may survive
while
articulated pose estimation fails
```

This result would justify decoupling target retention from pose recognition.

---

### G0 — Global-state / silhouette baseline

Where an athlete mask is available, estimate only coarse projected body state.

Candidate quantities:

- foreground centroid,
- bounding box,
- projected width / height,
- area,
- principal axis by PCA,
- coarse projected orientation where meaningful.

This deliberately asks a simpler question than pose estimation:

> Can the system retain useful global information about the diver after joint-level articulation becomes unreliable?

---

## 6. Orientation robustness diagnostic

### 6.1 Motivation

A previous exploratory approach transformed pose coordinates mathematically, but that does not determine whether the pose network can interpret the athlete pixels at an unusual orientation.

The relevant comparison is:

```text
post-hoc coordinate rotation
```

versus:

```text
image pixels
→ image-space rotation / canonicalization
→ pose model
→ inverse coordinate transform
```

The latter is the diagnostic used here.

---

### O0a — Known-good crop, original orientation

Select athlete crops where the baseline pose is clearly usable in the original image orientation.

These are reference cases for orientation-only manipulation.

---

### O0b — Same known-good crop under synthetic rotations

Rotate the **same athlete pixels** synthetically while keeping the underlying human articulation unchanged.

A candidate initial rotation set is:

```text
0°, 45°, 90°, 135°, 180°, 225°, 270°, 315°
```

A denser grid may be added if inexpensive, but the final grid must be recorded before held-out evaluation.

The operation should include the entire athlete crop and enough padding to avoid clipping body parts after rotation.

After inference, transform predicted keypoints back into the original reference coordinate system.

The core equivariance test is:

```text
f(Rθ I) ≈ Rθ f(I)
```

where:

- `I` is the known-good crop,
- `Rθ` is the image-space rotation,
- `f` is the pose estimator.

This test changes orientation only; it does **not** introduce a real change in:

- articulation,
- motion blur,
- self-occlusion,
- athlete identity,
- dive phase.

---

### O1a — Difficult diving crop, original orientation

Use a difficult crop from a critical failure interval after athlete localization is controlled.

---

### O1b — Difficult crop with oracle canonicalization

Supply a manually or externally determined coarse body orientation and rotate the **pixels before pose inference** toward a canonical image-plane orientation.

After inference, transform predicted keypoints back into the original frame coordinates.

Purpose:

> Does essentially correct orientation information make the same difficult athlete pixels materially easier for the pose estimator?

If O1b fails similarly to O1a, orientation shift alone is unlikely to explain the difficult-pose failure.

---

### O1c — Estimated-orientation canonicalization

Attempt automatic orientation estimation only after O1b demonstrates that oracle canonicalization is beneficial.

This preserves a clean dependency:

```text
first:  does canonicalization help?
then:   can orientation be estimated well enough?
```

Do not build a complex orientation estimator merely to discover later that perfect orientation would not have helped.

---

### 6.2 Canonical orientation definition

Do not assume in advance that one exact anatomical vector is always observable.

Candidate canonical axes may include:

- coarse torso axis,
- head-to-pelvis axis when available,
- principal foreground axis,
- or another robust global-body orientation descriptor.

The protocol should record which definition is used.

For tightly tucked or twisted poses, global image-plane orientation can itself be ambiguous. Oracle labels should therefore allow an uncertainty / ambiguity field.

---

### 6.3 Rotation-equivariance metrics

For known-good crops, define a rotation-equivariance error:

```text
E_rot(θ) = d( f(Rθ I), Rθ f(I) )
```

after mapping both predictions into the same coordinate frame.

Candidate reporting:

- error versus rotation angle,
- successful-pose rate versus angle,
- confidence versus angle,
- F0–F3 failure rates versus angle.

The exact distance metric `d` and normalization must be frozen before final held-out evaluation.

---

### 6.4 Canonicalization gain

For difficult crops, compare O1a and O1b.

Candidate quantity:

```text
G_canon = error_original - error_oracle_canonicalized
```

where positive `G_canon` indicates improvement.

If full pose ground truth is unavailable, also report transitions in failure class, for example:

```text
F2 articulation collapse
→
F3 structured pose
```

or

```text
catastrophic failure
→
usable pose
```

according to predeclared qualitative / structural criteria.

---

### 6.5 Orientation estimation is not yet part of the core model

O1c is conditional on O1b.

The baseline protocol should therefore not assume that orientation canonicalization will become part of the final system.

Possible outcomes include:

```text
O0b fails strongly, O1b helps
→ orientation prior is a real bottleneck

O0b fails strongly, O1b does not help difficult poses
→ orientation sensitivity exists, but difficult-pose failure is dominated by other mechanisms

O0b is stable, O1b does not help
→ orientation shift is probably not a major issue

O1b helps strongly
→ automatic orientation estimation becomes a justified next problem
```

---

## 7. Failure taxonomy

A failure interval may transition between categories.

### F0 — Person detection failure

No usable athlete detection is produced.

### F1 — Target identity loss

The predicted target / skeleton leaves the athlete region and attaches to a wrong image region or background structure.

Typical manifestation:

- rail / advertising-board hallucination,
- skeleton centroid far from athlete foreground,
- confident pose on the wrong object.

### F2 — Global-state survival with articulation collapse

The predicted athlete location or flight path remains approximately correct, but individual joints become unstable or noise-like.

Typical manifestation:

- skeleton centroid follows the diver,
- limb assignments jump unpredictably,
- joint geometry varies catastrophically.

### F3 — Structured but semantically wrong pose

The skeleton remains coherent-looking but is wrong.

Examples:

- left/right swaps,
- arm/leg assignment errors,
- wrong global orientation,
- physically implausible articulation,
- systematic upright-human reinterpretation of an inverted diver.

### F4 — Orientation-sensitive failure mechanism

Use F4 as an additional mechanism label when controlled orientation experiments show that image-plane rotation itself substantially changes pose quality.

F4 may coexist with F0–F3; it does not replace them.

---

## 8. Failure timing

For each clip / critical interval, record where possible:

```text
t_last_reliable
t_first_failure
failure_type_at_onset
t_first_recovery
t_water_entry
failure_duration
recovered_before_entry  # yes / no
```

Also record the dive phase at:

- last reliable frame,
- first failure,
- recovery.

This may reveal whether the baseline fails systematically at the same semantic transition.

---

## 9. Critical event landmarks

Initial event labels:

### P0 — last clearly reliable pre-failure pose frame

The last frame before the critical region in which the baseline pose is still judged usable.

### P1 — onset of deep flexion / inversion

First frame of the take-off configuration that begins to violate conventional upright-human appearance strongly.

### P2 — take-off / loss of support

Transition from support to airborne motion.

### P3 — first clearly identifiable airborne pose

The first airborne state whose intended configuration can be identified with reasonable confidence.

### P4 — strongest occlusion / blur / inversion region

The interval expected to be most difficult for frame-wise pose.

### P5 — opening begins

First systematic transition from the compact flight configuration toward entry alignment.

### P6 — entry alignment begins

Body begins the final alignment for water entry.

### P7 — last useful pre-contact frame

Last frame before water contact in which body-state evaluation is still meaningful.

### P8 — first water contact

First visible contact with the water.

These labels must later receive dataset-specific annotation rules without changing their conceptual meaning.

---

## 10. Athlete-retention metrics

Where athlete bbox / mask ground truth exists, candidate metrics include:

### 10.1 Bounding-box overlap

Intersection over Union between predicted tracked athlete region and true athlete region.

### 10.2 Normalized centroid error

Distance between predicted pose / track centroid and athlete centroid, normalized by an athlete-size or frame-size scale.

### 10.3 Keypoints-inside-foreground fraction

Fraction of predicted keypoints that fall inside or within a small tolerance of the true athlete foreground.

This is particularly useful for detecting joint explosions into background structures.

### 10.4 Target-loss event rate

Fraction of clips / critical windows containing at least one F1 event.

### 10.5 Retention duration

Number of frames the correct athlete identity remains retained after P0.

---

## 11. Background-hallucination metrics

For B0d:

- false person-detection rate,
- false pose-detection rate,
- confidence distribution,
- number of keypoints predicted,
- skeleton spatial extent,
- distance from the removed athlete region.

A model should ideally produce **no confident athlete pose** in the background-only control.

---

## 12. Articulation metrics

Where trustworthy ground-truth keypoints exist:

- normalized joint localization error,
- visible-joint PCK / equivalent metric,
- missing-joint rate,
- left/right swap rate,
- joint assignment error,
- global-orientation error where definable.

Always stratify by:

- visibility,
- occlusion,
- critical phase,
- annotation confidence,
- and orientation condition where relevant.

---

## 13. Structural and temporal diagnostics

### 13.1 Pose explosion / temporal jump

Measure large frame-to-frame changes in predicted joint configuration.

Any threshold must be normalized appropriately and calibrated without using the held-out evaluation set.

### 13.2 Segment geometry

**Do not use constant observed 2D limb length as a physical invariant.**

For a true fixed-length 3D segment:

```text
3D length = constant
2D projected length = orientation- and camera-dependent
```

Initial 2D diagnostics may still detect extreme discontinuities, but they must be interpreted as plausibility cues rather than direct physical constraints.

### 13.3 Confidence versus plausibility

Model confidence and pose plausibility are different quantities.

Explicitly record cases where:

```text
confidence is high
but
target identity / geometry is clearly wrong
```

These are particularly important catastrophic failures.

---

## 14. Camera / shot state

At minimum annotate or infer a coarse camera-state category:

```text
fixed
pan / tilt
zoom
pan / tilt + zoom
cut / shot transition
uncertain
```

Reason:

- world-space centre-of-mass motion may be approximately ballistic,
- raw image-space motion may not be ballistic under camera motion.

Any flight-physics term must therefore be camera-aware or applied only where camera behavior is sufficiently controlled.

Image rotation in the O0/O1 diagnostic is an intentional preprocessing intervention and must be recorded separately from natural camera rotation / shot behavior.

---

## 15. Reconstruction ablation after diagnostic control

### B1 — Temporal articulated reconstruction

Use sequence information after athlete retention and orientation effects have been characterized.

### B2 — Projection-aware biomechanical constraints

Candidate soft priors:

- underlying 3D segment geometry,
- joint-angle ranges,
- angular-velocity limits,
- angular-acceleration limits,
- temporal consistency.

### B3 — Dive-phase priors

Condition plausible pose / transition distributions on the current dive phase.

### B4 — Camera-aware flight physics

Candidate soft priors:

- world-space centre-of-mass ballistics,
- orientation continuity,
- approximate angular-momentum conservation,
- configuration-dependent moment of inertia.

---

## 16. Causal versus offline sequence inference

Report separately where feasible.

### Causal

```text
estimate x_t using I_1 ... I_t
```

Relevant to live / near-live feedback.

### Offline smoother

```text
estimate x_t using I_1 ... I_T
```

Relevant to post-performance coaching and research.

Future frames can legitimately help resolve an occluded joint that becomes visible again later.

---

## 17. Dataset sampling and split rules

The Experiment 001 corpus should be enriched for the known failure conditions rather than uniformly sampled.

Candidate stratification:

- springboard / platform,
- dive family,
- tuck / pike / straight,
- twist / no twist,
- take-off failure severity,
- airborne self-occlusion,
- blur,
- entry difficulty.

### Held-out principle

Avoid a purely random split if it leaks the same:

- source video,
- competition session,
- background,
- broadcaster overlay,
- camera setup,
- or athlete

into both development and held-out evaluation.

Prefer grouped separation where metadata allows.

### Orientation-diagnostic sampling

O0b should be drawn from clearly successful baseline crops, not cherry-picked difficult examples.

O1a/O1b should be drawn from predeclared critical-window cases after localization control.

This prevents the orientation hypothesis from being tested only on examples selected because canonicalization appears promising.

---

## 18. Ground-truth uncertainty

For each manually or externally annotated joint, distinguish as far as possible:

```text
visible
occluded_but_inferable
out_of_frame
left_right_ambiguous
unknown
```

Also retain an annotation-confidence field.

For oracle canonicalization additionally record:

```text
orientation_angle
orientation_confidence
orientation_ambiguous  # yes / no
orientation_definition
```

A fully occluded joint inferred from biomechanics is not equivalent to a directly visible joint.

---

## 19. Primary reporting table

Freeze the general structure before result inspection.

Suggested diagnostic rows:

```text
B0a whole-frame
B0b oracle bbox
B0c oracle foreground
B0d background-only
B0e tracked crop
T0  tracker only
G0  global state
O0a known-good original
O0b known-good rotated
O1a difficult original
O1b difficult oracle-canonicalized
O1c estimated canonicalization
```

Suggested reconstruction rows:

```text
B1 + temporal reconstruction
B2 + biomechanics
B3 + phase prior
B4 + flight physics
```

Suggested result groups:

```text
all evaluable frames
take-off critical
flight critical
opening / entry critical
rotation-angle groups
```

Suggested failure columns:

```text
F0 detection failure
F1 target loss
F2 articulation collapse
F3 structured wrong pose
F4 orientation-sensitive mechanism
background hallucination
recovery before entry
```

Pose-error columns are added where suitable ground truth exists.

---

## 20. Interpretation rules

Do not claim that a later stage "solves pose estimation" merely because average error improves.

A useful result must identify **which failure mechanism changed**.

Examples:

```text
B0b eliminates F1 but not F2
→ localization solved, articulation still fails

B0c improves over B0b
→ background/context contributes additional error

B0d hallucinates high-confidence poses
→ background-induced false structure is directly demonstrated

T0 succeeds while B0e pose fails
→ tracking and articulation should be decoupled

O0b degrades strongly with angle
→ baseline lacks useful rotational equivariance

O1b substantially improves O1a
→ orientation canonicalization is a justified component to investigate further

O0b degrades but O1b does not improve difficult poses
→ orientation sensitivity exists but is not the dominant diving-pose failure

B1 improves F2 after B0e / O1 controls
→ temporal pose reconstruction adds value beyond target retention and orientation normalization
```

---

## 21. Versioning and deviations

This document is **v0.2**.

Changes from v0.1:

- added image-space rotation-equivariance diagnostic,
- added oracle orientation canonicalization,
- explicitly separated coordinate rotation from pixel-space canonicalization,
- added conditional estimated-orientation stage,
- added orientation-sensitive failure mechanism label F4,
- extended reporting and ground-truth fields for orientation analysis.

After real dataset access:

- record any necessary change,
- state why it was needed,
- distinguish calibration from hypothesis-changing redesign,
- do not silently overwrite inconvenient pre-data definitions.

The aim is not formal preregistration, but a transparent pre-data experimental record.
