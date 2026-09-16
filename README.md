# DeepCrack Research — Road Crack Segmentation

> **Status: active research / in development**  
> This public repository is intentionally **documentation-only**. Training code, model implementations, experiment notebooks, checkpoints, and raw logs are withheld while the research is being consolidated for possible journal publication.

## Overview

This project studies **road crack segmentation** with a focus on thin, highly imbalanced structures where small localization errors can strongly affect connectivity and region-overlap metrics.

The work started from reproducing and studying the DeepCrack family of methods, then expanded into architecture and objective-function experiments designed to understand which components improve crack localization, continuity, and training stability.

The repository is presented as a research showcase rather than a clone-and-run package. The current public version intentionally avoids implementation details that may be part of ongoing research.

## Research Questions

The project is organized around several questions:

- How should encoder-decoder architectures preserve thin crack structures while suppressing background noise?
- How do segmentation objectives such as BCE, Dice, IoU, Focal, Boundary, and Connectivity-oriented losses change optimization behavior?
- Can adaptive loss-balancing strategies improve training when multiple objectives produce gradients with very different scales?
- How stable are observed improvements across architectures rather than on a single model?
- Which changes improve crack continuity without trading away precision or producing excessive false positives?

## Research Directions

### 1. DeepCrack baseline study

A DeepCrack-style baseline is used to understand hierarchical feature supervision and the behavior of multi-scale side outputs for crack segmentation.

### 2. U-Net-style reconstruction

The project explores U-Net-inspired encoder-decoder variants that combine hierarchical supervision with progressive spatial reconstruction and skip connections. An attention-based extension has also been prototyped and is still being evaluated.

### 3. Loss-function study

A major part of the project is loss-centric experimentation. The study compares individual and combined segmentation objectives, including region-overlap and boundary-sensitive formulations, rather than treating the loss function as a fixed implementation detail.

### 4. Adaptive objective weighting

Adaptive weighting strategies such as **GradNorm** and **uncertainty-based weighting** are investigated to understand whether dynamic balancing can reduce domination by one loss component during multi-objective optimization.

### 5. Cross-architecture evaluation

The same ideas are being examined across multiple segmentation architectures, including DeepCrack-derived models and lightweight alternatives such as LM-Net, to distinguish architecture-specific gains from more general training effects.

## Current Research Status

| Workstream | Status |
|---|---|
| DeepCrack baseline reproduction and analysis | Completed internally |
| U-Net-style DeepCrack hybrid experiments | Completed internally |
| Attention-based extension | Prototype / under evaluation |
| Static multi-loss comparison | Completed across multiple configurations |
| GradNorm-based adaptive weighting | Evaluated internally |
| Uncertainty-based adaptive weighting | Evaluated internally |
| Cross-architecture validation | In progress |
| Final consolidated benchmark | In progress |
| Journal-ready method and manuscript | Not finalized |

## Evaluation Protocol

The project uses two crack-segmentation datasets with different roles:

- **UDTIRI-Crack** is used for model training and validation. Its public release does not provide the test split used by this project, so UDTIRI results in this repository are reported only for train/validation.
- **OmniCrack30K** is used as an external test dataset to evaluate how the trained model transfers beyond the UDTIRI training/validation distribution.

This means the OmniCrack30K numbers should be interpreted as **cross-dataset test performance**, not as a same-dataset held-out UDTIRI test score.

The consolidated UDeepCrack table is available at [`results/udeepcrack_results.csv`](results/udeepcrack_results.csv). It contains **27 UDeepCrack experiment rows** covering static loss ablations and adaptive loss-weighting methods. Metrics are stored on a **0–1 scale**.

## Representative UDeepCrack Results

The values below are representative internal results, **not final journal claims**. They are included to make the current experimental progress visible while the full benchmark is still being consolidated.

| UDeepCrack configuration | UDTIRI Val Dice | UDTIRI Val IoU | OmniCrack30K Test Dice | OmniCrack30K Test IoU | Patch Test Dice | Patch Test IoU |
|---|---:|---:|---:|---:|---:|---:|
| CE + Dice + IoU + Focal | 0.7100 | 0.5504 | **0.4039** | **0.2991** | 0.4088 | 0.3058 |
| IoU loss | 0.7005 | 0.5390 | 0.3989 | 0.2965 | **0.4111** | **0.3078** |
| Dice + CE | 0.7086 | 0.5487 | 0.3991 | 0.2975 | 0.4049 | 0.3046 |

Among the adaptive-weighting UDeepCrack runs in the provided results, **uncertainty weighting with BCE + Dice + Focal** reached validation Dice **0.6376** and validation IoU **0.4847** on UDTIRI-Crack. The corresponding OmniCrack30K test metrics were not present in the provided adaptive-weighting results file, so they are intentionally left blank in the consolidated CSV rather than inferred.

The consolidated file also preserves source-traceability fields and data-quality notes. One source row contains an apparent missing decimal point (`7725`, interpreted as `77.25%`), and one historical `Dice+Focal` test row has IoU values greater than Dice; those values are retained but explicitly flagged instead of silently corrected.

## Evaluation Focus

Experiments track segmentation behavior using metrics such as:

- Dice / F1
- Intersection over Union (IoU)
- Precision and recall
- false-positive / false-negative behavior
- qualitative crack continuity and boundary quality
- training stability and computational cost

The emphasis is not only on obtaining a single high score, but on understanding **why a particular architecture or objective behaves differently on thin crack structures**.

## Public Repository Scope

The following materials are intentionally **not distributed** in this repository:

- model and loss implementation code
- training and evaluation scripts
- Jupyter notebooks
- experiment patching / automation utilities
- checkpoints and trained weights
- raw training logs and internal result dumps
- unpublished implementation details

Selected consolidated result tables may be published when they do not expose implementation details that the team intends to keep private.

This is deliberate. The project is ongoing and the team is considering a journal submission, so the public repository is limited to a high-level description of the research scope and status.

## Technologies Used Internally

The research workflow uses Python and PyTorch, with GPU-based training and systematic experiment tracking for semantic segmentation.

## Research Context

The work draws on established ideas from DeepCrack, U-Net-style segmentation, attention mechanisms, and adaptive multi-objective optimization. Those methods are used as research foundations and comparison points; ongoing project-specific implementation and experimental details remain private.

## Repository Status

This repository will be updated as the research matures and as additional material becomes appropriate for public release.
