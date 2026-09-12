# Candidate Data Sources

This file records candidate datasets and their intended role in the research plan. It is not a claim that access has already been granted.

## FineDiving

**Role:** Primary candidate diving dataset for real competition video, dive labels, temporal/action structure, critical-phase sampling and benchmark development.

Project:
https://github.com/xujinglin/FineDiving

**Current status:** Access request planned / pending.

**Experiment 001 relevance:**

- source video for whole-frame baseline inference,
- critical-window selection,
- dive-family / phase metadata where available,
- source-grouped held-out evaluation.

**Repository policy:** Restricted source material will not be redistributed here.

## FineDiving-HM

**Role:** Candidate human foreground masks for segmentation, visibility, occlusion and controlled baseline diagnostics.

Project:
https://github.com/PKU-ICST-MIPL/FineParser_CVPR2024

**Current status:** Access request planned / pending.

**Experiment 001 relevance:**

FineDiving-HM may enable direct separation of target-localization failure from articulated-pose failure through:

- oracle athlete mask / crop experiments,
- foreground-only pose inference,
- background-only negative controls,
- athlete-retention measurements,
- simple silhouette / global-state baselines.

This makes the masks useful not only for segmentation but also as a diagnostic intervention in the causal structure of the pose pipeline.

## FineDiving-Pose / HP-MCoRe

**Role:** Candidate diving-specific pose annotations for keypoint-quality audit and articulated-pose evaluation.

Project:
https://github.com/lumos0507/hp-mcore

**Current status:** Publicly linked annotations identified; applicable usage / licensing terms should be confirmed before relying on them.

**Experiment 001 relevance:**

Pose annotations are particularly useful **after localization has been controlled**, for evaluating:

- articulation collapse,
- left/right swaps,
- joint localization,
- structured but semantically wrong poses,
- and recovery through critical phases.

## Additional candidate datasets

Other sports or motion datasets may later be considered for:

- dynamic pose priors,
- 3D pose lifting,
- extreme airborne poses,
- motion forecasting,
- and cross-dataset evaluation.

Each source must receive a separate provenance and license review before being used.

## Alternative / future self-collected data

If external research datasets are insufficient, a later collaboration with diving clubs, federations or research institutions may provide controlled multi-camera or high-frame-rate data.

Such a dataset would be especially valuable for:

- camera calibration,
- synchronized multi-view ground truth,
- testing monocular reconstruction against stronger 3D reference measurements,
- and collecting targeted examples of the exact failure modes found in Experiment 001.

This is a later research stage, not a prerequisite for the initial dataset-access and baseline phase.
