# Dataset and Provenance Policy

## Principle

This repository separates **shareable project material** from **restricted third-party material**.

The fact that research code is public does not imply that datasets, source videos, annotations, masks, model weights, or other external assets may be redistributed.

## Allowed repository content

Subject to the license of each component, this repository may contain:

- original source code written for this project,
- experiment configurations,
- data schemas,
- dataset-loader interfaces,
- annotation tools,
- methodology,
- evaluation scripts,
- aggregate metrics,
- derived plots and tables where permitted,
- research notes,
- and reproducibility documentation.

## Material that must not be committed unless redistribution is explicitly permitted

- competition videos,
- extracted frames from restricted videos,
- restricted third-party annotations,
- segmentation masks,
- dataset archives,
- private download links,
- access tokens,
- credentials,
- signed release agreements,
- personal contact information contained in access agreements,
- and third-party model weights whose license forbids redistribution.

## FineDiving family

FineDiving and related resources must be obtained from their original maintainers under their applicable access terms.

This project will not redistribute restricted FineDiving-family material.

The local working copy should live outside Git version control or under ignored local data directories.

## Public repository boundary

A public experiment should be reproducible by documenting:

1. the required dataset,
2. where an authorized researcher can request or obtain it,
3. expected local directory structure,
4. preprocessing steps,
5. configuration,
6. model provenance,
7. evaluation procedure,
8. and result-generation commands.

Reproducibility does not require illegal or unauthorized redistribution of source data.

## Provenance record

Every external component used in an experiment should eventually have a provenance record containing at least:

| Field | Meaning |
|---|---|
| Name | Dataset, model, codebase, checkpoint, or asset |
| Source | Canonical project / paper / maintainer |
| Version | Commit, release, model version, or access date |
| License / terms | Applicable terms |
| Research use | Whether current planned use is permitted |
| Redistribution | Whether redistribution is permitted |
| Commercial use | If known; otherwise mark as unresolved |
| Local path | Local-only reference, never credentials |
| Notes | Important restrictions or ambiguities |

## Commercial use

The present project phase is non-commercial scientific research.

No assumption should be made that a research dataset or a model trained from restricted research-only data is suitable for commercial use.

If a later commercial implementation becomes relevant, its dataset, weights, provenance, and permissions should be audited separately.
