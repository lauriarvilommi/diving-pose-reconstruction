# Diving Pose Reconstruction

**Independent, non-commercial computer-vision research on robust human pose reconstruction in competitive diving.**

**Status:** Early-stage research — dataset acquisition and pre-data baseline-protocol freeze.

## Motivation

General-purpose human pose estimators often work well while a diver remains in relatively conventional upright configurations, but they can become unreliable during the most difficult parts of a dive:

- deep take-off flexion,
- inversion,
- tuck and pike positions,
- twisting,
- strong self-occlusion,
- motion blur,
- rapid opening,
- and near-vertical water entry.

Preliminary tests suggest that the failure can be **catastrophic rather than merely noisy**. Once the diver enters highly inverted or compact configurations, a conventional pose pipeline may lose the athlete as a coherent target, place joints on unrelated background structures, or retain only the approximate flight path while the articulated pose degenerates into unstable joint-position noise.

The project therefore separates four questions that are often conflated:

1. **Athlete retention:** does the system still know where the same diver is?
2. **Global body state:** can it retain the diver's approximate location, extent and orientation?
3. **Orientation robustness:** does the pose model fail because the same human configuration appears at an unusual image-plane orientation?
4. **Articulated pose:** can it estimate a coherent body configuration once localization and orientation effects are controlled?

## Core hypothesis

The project will test a hierarchical reconstruction strategy:

```text
camera / shot state
        ↓
athlete retention
        ↓
global body state
        ↓
orientation robustness / canonicalization
        ↓
articulated pose
        ↓
temporal reconstruction
        ↓
projection-aware biomechanical constraints
        ↓
diving-phase priors
        ↓
lightweight flight physics
```

The first goal is not to build a complete diving-analysis product. It is to determine **where the baseline pipeline actually fails** and which additional constraints materially improve reconstruction specifically in the critical frames where ordinary pose estimators become unreliable.

## Initial research questions

The current study asks, among other things:

1. Does failure begin with person/target retention, or only after the athlete has already been correctly localized?
2. Can oracle crops, foreground masks and background-only controls separate target-loss failures from articulated-pose failures?
3. Is the pose model approximately rotation-equivariant, or does a normally recognizable human become difficult simply when the image is rotated away from an upright orientation?
4. If a difficult diving crop is given an oracle orientation and rotated into a canonical image-plane orientation, does pose estimation improve?
5. Can temporal tracking retain the athlete through the failure interval without solving pose?
6. Once localization and orientation effects are controlled, can temporal reconstruction recover a coherent articulated pose?
7. How much do projection-aware body geometry and joint constraints help?
8. Does diving-phase information reduce the remaining ambiguity?
9. Can lightweight flight physics improve airborne trajectory and orientation reconstruction once camera motion is accounted for?

See [`docs/research_questions.md`](docs/research_questions.md) for the current research-question set.

## Experiment 001

The first experiment is designed as a low-cost falsification and failure-localization test before any large synthetic-data system is built.

The diagnostic baseline begins with the **same pose model under controlled changes to its input**:

```text
B0a  whole-frame detector + pose
B0b  oracle athlete bounding box + same pose model
B0c  oracle foreground mask + same pose model
B0d  background-only negative control
B0e  temporally propagated / tracked athlete crop + same pose model
T0   tracker-only baseline
G0   global-state / silhouette baseline
```

A separate orientation diagnostic tests whether image-plane orientation itself causes failure:

```text
O0a  known-good athlete crop, original orientation
O0b  same known-good crop under synthetic rotations
O1a  difficult diving crop, original orientation
O1b  same difficult crop with oracle canonicalization
O1c  estimated-orientation canonicalization
```

Only after the failure source has been localized does the reconstruction ablation proceed:

```text
B1  + pose temporal reconstruction
B2  + projection-aware body geometry / biomechanical constraints
B3  + joint and diving-phase priors
B4  + camera-aware lightweight flight physics
```

Evaluation will focus on **critical failure windows**, not only whole-video average pose accuracy.

See:

- [`docs/experiment_001.md`](docs/experiment_001.md)
- [`docs/baseline_protocol.md`](docs/baseline_protocol.md)

## Data policy

This repository does **not** redistribute restricted third-party datasets, videos, frames, annotations, masks, private download links, credentials, or signed access agreements.

Third-party datasets must be obtained directly from their original maintainers under their own terms. The repository is intended to contain only material that can lawfully be shared, such as:

- original source code,
- experiment configurations,
- data-loader interfaces,
- methodology,
- documentation,
- and aggregate or derived research results where permitted.

See [`docs/dataset_and_provenance_policy.md`](docs/dataset_and_provenance_policy.md).

## Repository structure

```text
.
├── README.md
├── LICENSE
├── CITATION.cff
├── .gitignore
└── docs/
    ├── baseline_protocol.md
    ├── data_sources.md
    ├── dataset_and_provenance_policy.md
    ├── experiment_001.md
    ├── project_status.md
    ├── research_plan.md
    └── research_questions.md
```

The structure is intentionally small at this stage. Additional directories will be added only when real code, configurations, notebooks or results require them.

## Research independence

This is an independent research project and is not currently affiliated with a university, company, sports federation, or commercial product.

The current research phase is non-commercial.

## Reproducibility

The project will aim to document:

- dataset provenance and access conditions,
- model and checkpoint provenance,
- preprocessing,
- source-grouped evaluation splits,
- camera / shot state,
- critical-frame definitions,
- diagnostic controls,
- orientation-normalization controls,
- experiment configurations,
- ablations,
- failure cases,
- recovery behavior,
- and negative results.

Restricted data will remain outside the public repository.

## Important geometric note

The project does **not** assume that observed 2D limb lengths remain constant. A fixed 3D body segment can project to very different 2D lengths as the diver rotates relative to the camera, and broadcast footage may contain pan, tilt and zoom.

Strong body-geometry constraints must therefore be **projection-aware** rather than treating image-plane segment length as a physical invariant.

Likewise, rotating predicted keypoints after inference is not equivalent to rotating the athlete pixels before inference. The orientation diagnostic explicitly tests the latter.

## License

Original source code and repository material are released under the MIT License unless a file states otherwise. This license does **not** apply to third-party datasets, videos, annotations, pretrained weights, or other external assets.

## Author

Lauri Arvilommi  
Independent researcher / software developer  
GitHub: [@lauriarvilommi](https://github.com/lauriarvilommi)
