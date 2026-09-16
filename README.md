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

## Representative Internal Results

The values below are included to show the current experimental progress, **not as final journal claims**. They come from historical internal runs and may change as the evaluation protocol is consolidated.

| Experiment | Representative validation result | Notes |
|---|---|---|
| U-DeepCrack hybrid | Crack IoU ≈ **0.5429** | Best checkpoint in one archived run; VGG16-based encoder-decoder with deep supervision |
| Archived static multi-loss run (`Scenario_16`) | Dice **0.7101**, Crack IoU **0.5505**, mIoU **0.7673** | Best recorded validation point at epoch 58 |
| Same `Scenario_16` checkpoint | Precision **0.6986**, Recall **0.7220** | Shows the precision/recall trade-off rather than reporting overlap alone |

The U-DeepCrack run used approximately **25.86M parameters** and the archived report recorded its best checkpoint at epoch 23. The static multi-loss run continued to epoch 78 before early stopping, with its best validation crack IoU observed earlier in training.

These numbers are intentionally labeled as **representative validation results**. They should not be interpreted as a finalized benchmark across architectures because some historical experiments used different training or evaluation details.

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

This is deliberate. The project is ongoing and the team is considering a journal submission, so the public repository is limited to a high-level description of the research scope and status.

## Technologies Used Internally

The research workflow uses Python and PyTorch, with GPU-based training and systematic experiment tracking for semantic segmentation.

## Research Context

The work draws on established ideas from DeepCrack, U-Net-style segmentation, attention mechanisms, and adaptive multi-objective optimization. Those methods are used as research foundations and comparison points; ongoing project-specific implementation and experimental details remain private.

## Repository Status

This repository will be updated as the research matures and as additional material becomes appropriate for public release.
