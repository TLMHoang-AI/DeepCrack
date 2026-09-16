# DeepCrack Research — Loss-Centric Road Crack Segmentation

> **Status: active research / in development**  
> This public repository is intentionally **documentation- and results-focused**. Training code, model implementations, experiment notebooks, checkpoints, and raw logs are withheld while the research is being consolidated for possible journal publication.

## Overview

This project studies **road crack segmentation through the lens of loss-function design and optimization**. Crack pixels are sparse, thin, and structurally sensitive: a small number of false positives, false negatives, or broken connections can strongly affect both region-overlap metrics and the visual continuity of a predicted crack.

The main research question is therefore not only *which architecture performs well*, but **how the training objective changes what a segmentation model learns**.

The work starts from a DeepCrack-derived segmentation pipeline and uses it as an experimental platform for systematic loss studies. The project investigates:

- individual segmentation objectives such as BCE, Dice, IoU, and Focal loss;
- multi-loss combinations, including boundary- and connectivity-oriented objectives;
- the effect of fixed manual loss weights;
- adaptive loss balancing with **GradNorm**;
- a multiplicative GradNorm variant;
- **uncertainty-based loss weighting**;
- whether loss-function behavior remains consistent across different segmentation architectures.

Architectural variants such as U-DeepCrack, attention-enhanced extensions, and lightweight models are used primarily as **validation platforms for the loss-function research**, rather than being the sole focus of the project.

## Research Questions

The project is organized around several questions:

- Which objective functions best handle severe crack/background imbalance?
- How do BCE, Dice, IoU, Focal, Boundary, and Connectivity-oriented losses change the precision-recall trade-off?
- Do combinations of complementary objectives generalize better than single losses?
- Can adaptive weighting prevent one objective from dominating multi-loss optimization?
- How do **GradNorm** and **uncertainty weighting** behave compared with fixed loss weights?
- Are improvements specific to U-DeepCrack, or do they transfer across segmentation architectures?

## Research Workflow

```text
Road-crack segmentation baseline
            |
            v
     Single-loss ablation
   BCE / Dice / IoU / Focal
            |
            v
     Multi-loss combinations
 region + imbalance + boundary
            |
            v
   Fixed-weight optimization
            |
            v
 Adaptive objective weighting
  |                       |
GradNorm          Uncertainty weighting
  |                       |
  +-----------+-----------+
              |
              v
   Cross-architecture validation
              |
              v
    Cross-dataset evaluation
```

## Model Platform

### U-DeepCrack

U-DeepCrack is the main experimental platform in the current consolidated results. It combines a VGG16-based hierarchical encoder with U-Net-style spatial reconstruction, skip connections, multi-scale side outputs, and deep supervision.

The architecture is useful for the loss study because multiple supervised outputs create a demanding optimization setting in which different objective components can produce gradients with substantially different scales and behaviors.

### Additional architecture studies

The broader internal study also includes DeepCrack-derived variants, attention-based extensions, and lightweight segmentation architectures such as LM-Net. These experiments are used to test whether observations from the loss study are architecture-specific or more general.

## Loss-Function Study

### Static objective ablations

The static study evaluates both individual and combined segmentation losses. The consolidated U-DeepCrack results currently contain **23 static-loss configurations**, including single objectives and increasingly complex combinations.

Representative configurations include:

| U-DeepCrack loss configuration | UDTIRI Val Dice | UDTIRI Val IoU | OmniCrack30K Test Dice | OmniCrack30K Test IoU | Patch Test Dice | Patch Test IoU |
|---|---:|---:|---:|---:|---:|---:|
| CE + Dice + IoU + Focal | 0.7100 | 0.5504 | **0.4039** | **0.2991** | 0.4088 | 0.3058 |
| IoU | 0.7005 | 0.5390 | 0.3989 | 0.2965 | **0.4111** | **0.3078** |
| Dice + CE | 0.7086 | 0.5487 | 0.3991 | 0.2975 | 0.4049 | 0.3046 |

These results illustrate an important theme of the project: a configuration that is strong on UDTIRI validation is not automatically the strongest under cross-dataset testing, so both optimization behavior and transfer performance need to be considered.

### Adaptive loss weighting

A second part of the project replaces manually fixed weights with adaptive weighting methods. Four U-DeepCrack adaptive experiments are currently consolidated:

| Adaptive method | Objective set | UDTIRI Val Precision | Val Recall | Val Dice | Val IoU | Best epoch |
|---|---|---:|---:|---:|---:|---:|
| GradNorm | BCE + Dice | 0.4643 | 0.8324 | 0.5835 | 0.4299 | 87 |
| GradNorm + Multiplicative | BCE + Dice | 0.4791 | **0.8530** | 0.6009 | 0.4461 | 47 |
| Uncertainty weighting | BCE + Dice | 0.4852 | **0.8764** | 0.6067 | 0.4522 | 88 |
| Uncertainty weighting | BCE + Dice + Focal | **0.5453** | 0.8156 | **0.6376** | **0.4847** | 79 |

The adaptive experiments expose a different optimization regime from the strongest static-loss runs. In particular, the adaptive methods tend to produce **high recall**, while the uncertainty-weighted BCE + Dice + Focal configuration gives the strongest validation Dice and IoU among the adaptive runs currently available.

The provided adaptive-weighting result file does **not** contain the corresponding OmniCrack30K test metrics. Those cells are therefore left blank in the consolidated CSV rather than estimated or reconstructed from unrelated runs.

## Evaluation Protocol

The project uses two crack-segmentation datasets with different roles:

- **UDTIRI-Crack** is used for model training and validation. The dataset setup available to this project does not provide a public test split used in the experiments, so UDTIRI results are reported only for train/validation.
- **OmniCrack30K** is used as an external test dataset to evaluate how a model trained on UDTIRI transfers beyond the training/validation distribution.

The OmniCrack30K numbers should therefore be interpreted as **cross-dataset test performance**, not as a same-dataset held-out UDTIRI test score.

## Consolidated Results

The public consolidated table is available at:

[`results/udeepcrack_results.csv`](results/udeepcrack_results.csv)

It currently contains **27 U-DeepCrack experiments**:

- **23** static loss-function ablations;
- **4** adaptive loss-weighting experiments.

Metrics are normalized to a **0–1 scale** and the table keeps source-traceability and data-quality notes. Historical source irregularities are flagged instead of silently corrected.

## Current Research Status

| Workstream | Status |
|---|---|
| DeepCrack baseline reproduction and analysis | Completed internally |
| U-DeepCrack experimental platform | Completed internally |
| Single-loss ablations | Completed across multiple objectives |
| Static multi-loss combinations | Completed across multiple configurations |
| GradNorm adaptive weighting | Evaluated internally |
| Multiplicative GradNorm variant | Evaluated internally |
| Uncertainty-based adaptive weighting | Evaluated internally |
| Attention-based architecture extension | Prototype / under evaluation |
| Cross-architecture validation | In progress |
| Final consolidated benchmark | In progress |
| Journal-ready method and manuscript | Not finalized |

## Evaluation Focus

Experiments track more than a single segmentation score. The analysis includes:

- Dice / F1;
- crack IoU and mean IoU;
- precision and recall;
- false-positive / false-negative behavior;
- crack continuity and boundary quality;
- optimization stability;
- computational cost;
- transfer from UDTIRI-Crack to OmniCrack30K.

The aim is to understand **why a loss formulation changes model behavior**, rather than treating loss selection as a minor implementation choice.

## Public Repository Scope

The following materials are intentionally **not distributed** in this repository:

- model and loss implementation code;
- training and evaluation scripts;
- Jupyter notebooks;
- experiment automation utilities;
- checkpoints and trained weights;
- raw training logs;
- unpublished implementation details.

Selected consolidated result tables are published when they communicate the research progress without exposing implementation details that may become part of a journal submission.

## Technologies Used Internally

The internal research workflow uses **Python, PyTorch, GPU-based training, semantic segmentation, deep supervision, multi-objective optimization, and systematic experiment tracking**.

## Research Context

The project builds on established ideas from DeepCrack, U-Net-style segmentation, multi-objective optimization, GradNorm, and uncertainty-based task weighting. These methods serve as research foundations and comparison points; project-specific implementation details remain private while the work is under development.

## Repository Status

This repository will be updated as the research matures and as additional results become appropriate for public release.
