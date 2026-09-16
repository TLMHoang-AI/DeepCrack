# DeepCrack Research — U-DeepCrack Architecture & Loss Optimization

> **Status: active research / in development**  
> This public repository is intentionally **documentation- and results-focused**. Training code, model implementations, experiment notebooks, checkpoints, and raw logs are withheld while the research is being consolidated for possible journal publication.

## Overview

This project studies **road crack segmentation from two complementary directions: architecture design and loss-function optimization**.

The first direction is **U-DeepCrack**, a project-specific architectural extension developed from DeepCrack 2019. The goal is to improve spatial reconstruction of thin crack structures while preserving the hierarchical supervision that makes DeepCrack effective for multi-scale crack detection.

The second direction is a systematic study of **segmentation objectives and loss balancing**. Crack pixels are sparse, thin, and structurally sensitive, so the optimization objective strongly affects recall, false positives, continuity, and overlap metrics. The project therefore evaluates single losses, multi-loss combinations, fixed weighting, and adaptive weighting methods such as **GradNorm** and **uncertainty-based weighting**.

These two directions are intentionally studied together: U-DeepCrack provides a stronger architectural platform, while the loss-function study investigates how that platform should be optimized and whether the same conclusions transfer to other segmentation architectures.

## Main Research Contributions

### 1. U-DeepCrack architecture

U-DeepCrack extends the DeepCrack 2019 design with a U-Net-style reconstruction path. The implemented version combines:

- a VGG16-based hierarchical encoder;
- a four-stage decoder with progressive upsampling;
- encoder-decoder skip connections;
- double-convolution reconstruction blocks;
- five side outputs;
- deep supervision and fused prediction.

Compared with the original DeepCrack design, which mainly upsamples hierarchical side predictions independently, U-DeepCrack reconstructs spatial information progressively through a decoder while retaining multi-scale supervision.

An attention-gated extension of U-DeepCrack has also been prototyped and remains under evaluation.

### 2. Loss-function and optimization study

The project systematically investigates how the training objective changes segmentation behavior. The study includes:

- BCE / cross-entropy;
- Dice loss;
- IoU loss;
- Focal loss;
- Boundary-oriented loss;
- Connectivity-oriented loss;
- multi-loss combinations;
- fixed manual weighting;
- **GradNorm** adaptive weighting;
- a multiplicative GradNorm variant;
- **uncertainty-based loss weighting**.

The goal is not only to identify a high-scoring configuration, but to understand how different objectives trade precision, recall, overlap quality, crack continuity, and optimization stability.

### 3. Cross-architecture and cross-dataset validation

The loss strategies are also being evaluated beyond U-DeepCrack, including DeepCrack-derived and lightweight segmentation models such as LM-Net, to determine whether observed behavior is architecture-specific or more general.

Generalization is evaluated by training/validating on UDTIRI-Crack and testing on a different crack dataset, OmniCrack30K.

## Research Questions

The project is organized around several questions:

- Can U-Net-style reconstruction improve the spatial-detail limitations of the original DeepCrack architecture?
- How should hierarchical side supervision be combined with decoder-based reconstruction for thin crack structures?
- Which objective functions best handle severe crack/background imbalance?
- How do BCE, Dice, IoU, Focal, Boundary, and Connectivity-oriented losses change the precision-recall trade-off?
- Do combinations of complementary objectives generalize better than single losses?
- Can adaptive weighting prevent one objective from dominating multi-loss optimization?
- How do **GradNorm** and **uncertainty weighting** behave compared with fixed loss weights?
- Do architectural and loss-function improvements remain useful under cross-dataset evaluation?

## Research Workflow

```text
DeepCrack 2019 baseline study
            |
            +----------------------------+
            |                            |
            v                            v
  U-DeepCrack architecture       Loss-function study
  encoder + U-Net decoder        single / combined losses
  skip connections               fixed / adaptive weights
  deep supervision               GradNorm / uncertainty
            |                            |
            +-------------+--------------+
                          |
                          v
                Joint architecture-loss
                    experimentation
                          |
                          v
              Cross-architecture validation
                          |
                          v
              Cross-dataset evaluation
              UDTIRI -> OmniCrack30K
```

## U-DeepCrack Experimental Platform

U-DeepCrack is the main architecture represented in the public consolidated results. Its combination of hierarchical side outputs, decoder reconstruction, and deep supervision makes it useful for studying both **architectural reconstruction quality** and **multi-objective optimization behavior**.

The current internal implementation reported approximately **25.86M parameters** in one archived run. The architecture itself is part of the project's research contribution rather than merely a neutral benchmark for the loss study.

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

The adaptive experiments expose a different optimization regime from the strongest static-loss runs. In particular, the adaptive methods tend to produce high recall, while the uncertainty-weighted BCE + Dice + Focal configuration gives the strongest validation Dice and IoU among the adaptive runs currently available.

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
| DeepCrack 2019 reproduction and analysis | Completed internally |
| U-DeepCrack architectural extension | Implemented and evaluated internally |
| Single-loss ablations | Completed across multiple objectives |
| Static multi-loss combinations | Completed across multiple configurations |
| GradNorm adaptive weighting | Evaluated internally |
| Multiplicative GradNorm variant | Evaluated internally |
| Uncertainty-based adaptive weighting | Evaluated internally |
| Attention U-DeepCrack extension | Prototype / under evaluation |
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

The broader aim is to understand **how architecture design and objective design interact when segmenting thin, highly imbalanced crack structures**.

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

The internal research workflow uses **Python, PyTorch, GPU-based training, semantic segmentation, deep supervision, encoder-decoder architectures, multi-objective optimization, and systematic experiment tracking**.

## Research Context

The project builds on DeepCrack 2019 and established ideas from U-Net-style reconstruction, deep supervision, multi-objective optimization, GradNorm, and uncertainty-based weighting. **U-DeepCrack is the project's architectural extension of the DeepCrack baseline**, while the loss study forms the second major research axis. Project-specific implementation details remain private while the work is under development.

## Repository Status

This repository will be updated as the research matures and as additional results become appropriate for public release.
