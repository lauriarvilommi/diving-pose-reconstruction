# Baseline Protocol v0.1

**Status:** Pre-data freeze candidate  
**Experiment:** Experiment 001 — Critical-Frame Failure Localization and Pose Reconstruction

## 1. Purpose

This protocol defines the baseline experiment **before detailed inspection of the requested diving datasets**.

Its primary purpose is not to maximize pose accuracy. It is to answer a more basic question:

> **Where does a conventional human-pose pipeline first lose the diving motion?**

Preliminary tests indicate that failure may be catastrophic. Once the diver enters highly inverted, compact or blurred configurations, the system may:

- fail to retain the athlete as the same target,
- hallucinate joints on background structures,
- retain only the approximate athlete trajectory,
- or produce a structured but semantically incorrect skeleton.

The protocol therefore separates:

```text
camera / shot state
        ↓
athlete retention
        ↓
global body state
        ↓
articulated pose
```

before testing later sequence reconstruction.

---

## 2. Protocol-freeze principle

Before detailed dataset inspection, freeze as far as practical:

- the failure hierarchy,
- diagnostic baseline variants,
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

Do not place restricted source data in the public repository.

---

## 5. Diagnostic baseline matrix

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
→ articulated extreme-pose recognition remains a major failure source
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

## 6. Failure taxonomy

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

---

## 7. Failure timing

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

## 8. Critical event landmarks

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

## 9. Athlete-retention metrics

Where athlete bbox / mask ground truth exists, candidate metrics include:

### 9.1 Bounding-box overlap

Intersection over Union between predicted tracked athlete region and true athlete region.

### 9.2 Normalized centroid error

Distance between predicted pose / track centroid and athlete centroid, normalized by an athlete-size or frame-size scale.

### 9.3 Keypoints-inside-foreground fraction

Fraction of predicted keypoints that fall inside or within a small tolerance of the true athlete foreground.

This is particularly useful for detecting joint explosions into background structures.

### 9.4 Target-loss event rate

Fraction of clips / critical windows containing at least one F1 event.

### 9.5 Retention duration

Number of frames the correct athlete identity remains retained after P0.

---

## 10. Background-hallucination metrics

For B0d:

- false person-detection rate,
- false pose-detection rate,
- confidence distribution,
- number of keypoints predicted,
- skeleton spatial extent,
- distance from the removed athlete region.

A model should ideally produce **no confident athlete pose** in the background-only control.

---

## 11. Articulation metrics

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
- and annotation confidence.

---

## 12. Structural and temporal diagnostics

### 12.1 Pose explosion / temporal jump

Measure large frame-to-frame changes in predicted joint configuration.

Any threshold must be normalized appropriately and calibrated without using the held-out evaluation set.

### 12.2 Segment geometry

**Do not use constant observed 2D limb length as a physical invariant.**

For a true fixed-length 3D segment:

```text
3D length = constant
2D projected length = orientation- and camera-dependent
```

Initial 2D diagnostics may still detect extreme discontinuities, but they must be interpreted as plausibility cues rather than direct physical constraints.

### 12.3 Confidence versus plausibility

Model confidence and pose plausibility are different quantities.

Explicitly record cases where:

```text
confidence is high
but
target identity / geometry is clearly wrong
```

These are particularly important catastrophic failures.

---

## 13. Camera / shot state

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

---

## 14. Reconstruction ablation after localization control

### B1 — Temporal articulated reconstruction

Use sequence information after athlete retention has been controlled.

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

## 15. Causal versus offline sequence inference

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

## 16. Dataset sampling and split rules

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

---

## 17. Ground-truth uncertainty

For each manually or externally annotated joint, distinguish as far as possible:

```text
visible
occluded_but_inferable
out_of_frame
left_right_ambiguous
unknown
```

Also retain an annotation-confidence field.

A fully occluded joint inferred from biomechanics is not equivalent to a directly visible joint.

---

## 18. Primary reporting table

Freeze the general structure before result inspection.

Suggested rows:

```text
B0a whole-frame
B0b oracle bbox
B0c oracle foreground
B0d background-only
B0e tracked crop
T0  tracker only
G0  global state
B1  + temporal reconstruction
B2  + biomechanics
B3  + phase prior
B4  + flight physics
```

Suggested result groups:

```text
all evaluable frames
take-off critical
flight critical
opening / entry critical
```

Suggested failure columns:

```text
F0 detection failure
F1 target loss
F2 articulation collapse
F3 structured wrong pose
background hallucination
recovery before entry
```

Pose-error columns are added where suitable ground truth exists.

---

## 19. Interpretation rules

Do not claim that a later stage "solves pose estimation" merely because average error improves.

A useful result must identify **which failure class changed**.

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

B1 improves F2 after B0e
→ temporal pose reconstruction adds value beyond target retention
```

---

## 20. Versioning and deviations

This document is **v0.1**.

After real dataset access:

- record any necessary change,
- state why it was needed,
- distinguish calibration from hypothesis-changing redesign,
- do not silently overwrite inconvenient pre-data definitions.

The aim is not formal preregistration, but a transparent pre-data experimental record.
