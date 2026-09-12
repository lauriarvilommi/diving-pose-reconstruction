# Diving Pose Reconstruction

**Independent, non-commercial computer-vision research on robust human pose reconstruction in competitive diving.**

**Status:** Early-stage research — dataset acquisition, baseline design, and critical-failure mapping.

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

The central research question is whether these failure cases can be improved by treating pose estimation as a **sequence reconstruction problem** rather than an independent frame-by-frame detection problem.

## Core hypothesis

The project will test the following progression:

```text
frame-wise pose estimation
        ↓
+ temporal continuity
        ↓
+ fixed body geometry / biomechanical constraints
        ↓
+ diving-phase priors
        ↓
+ lightweight flight physics
```

The first goal is not to build a complete diving-analysis product. It is to determine which additional constraints materially improve pose reconstruction specifically in the frames where ordinary pose estimators fail.

## Initial research questions

1. Where and how do current pose estimators fail during competitive diving?
2. Can temporal continuity reconstruct poses during short periods of severe occlusion, inversion, or blur?
3. How much additional improvement comes from constant bone lengths and joint-range constraints?
4. Does diving-phase information reduce ambiguity further?
5. Can lightweight flight physics improve orientation and trajectory reconstruction during airborne phases?

See [`docs/research_questions.md`](docs/research_questions.md) for the current research-question set.

## Experiment 001

The first experiment is designed as a low-cost falsification test before any large synthetic-data system is built.

Planned ablation:

1. frame-wise baseline,
2. + temporal reconstruction,
3. + body-geometry constraints,
4. + joint and diving-phase priors,
5. + flight-physics constraints.

Evaluation will focus on **critical failure windows**, not only whole-video average pose accuracy.

See [`docs/experiment_001.md`](docs/experiment_001.md).

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

## Planned repository structure

```text
.
├── README.md
├── LICENSE
├── CITATION.cff
├── .gitignore
├── configs/
├── data/
├── docs/
├── notebooks/
├── results/
└── src/
```

The structure is intentionally small at this stage. Directories will grow only when experiments require them.

## Research independence

This is an independent research project and is not currently affiliated with a university, company, sports federation, or commercial product.

The current research phase is non-commercial.

## Reproducibility

The project will aim to document:

- dataset provenance and access conditions,
- model and checkpoint provenance,
- preprocessing,
- evaluation splits,
- critical-frame definitions,
- experiment configurations,
- ablations,
- failure cases,
- and negative results.

Restricted data will remain outside the public repository.

## License

Original source code and repository material are released under the MIT License unless a file states otherwise. This license does **not** apply to third-party datasets, videos, annotations, pretrained weights, or other external assets.

## Author

Lauri Arvilommi  
Independent researcher / software developer  
GitHub: [@lauriarvilommi](https://github.com/lauriarvilommi)
