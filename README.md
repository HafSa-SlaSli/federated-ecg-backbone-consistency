# Federated ECG Backbone Consistency

## Overview

This repository contains the final experimental code and reproducibility materials for our project on **federated multi-label ECG classification under strong statistical heterogeneity**.

The study investigates how heterogeneous client data affect the internal representations learned by federated convolutional neural networks and compares **FedAvg** with **MOON-style representation-consistency training**.

The experiments are based on **PTB-XL v1.0.3** and simulate five hospitals using a patient-level strongly non-IID partition.

## Research Question

The main research question is:

**How does strong statistical heterogeneity affect CNN backbone consistency in federated ECG learning, and can MOON-style training improve representation alignment relative to FedAvg?**

The study evaluates both:

- Predictive performance
- Internal representation consistency

These two aspects are analyzed separately because improved representation alignment does not necessarily imply proportional improvements in predictive performance.

## Dataset

The experiments use:

**PTB-XL v1.0.3**

Main diagnostic superclasses:

- NORM : Normal ECG
- MI : Myocardial Infarction
- STTC : ST/T Change
- CD : Conduction Disturbance
- HYP : Hypertrophy

Final usable diagnostic subset:

- Training: 17,084 ECGs - folds 1–8
- Validation: 2,146 ECGs - fold 9
- Test: 2,158 ECGs - fold 10

The federated partition contains five simulated hospitals with zero patient overlap between clients.

## Methods

Three training settings are studied:

### Centralized Learning

A centralized reference model is trained on the complete training set using the same CNN architecture and preprocessing pipeline.

### FedAvg

Standard Federated Averaging is used with sample-count-weighted aggregation.

### MOON-Style Training

The MOON-style condition keeps the same federated communication and aggregation protocol as FedAvg while adding a model-contrastive representation-consistency objective.

Our implementation uses the pooled 256-dimensional backbone representation directly and does not use a projection head. Therefore, it is referred to consistently as **MOON-style training** rather than canonical MOON.

## CNN Architecture

The same 1D CNN is used for all experiments.

- Input: 12 ECG leads × 1000 samples
- Conv1: 12 → 32 channels
- Conv2: 32 → 64 channels
- Conv3: 64 → 128 channels
- Conv4: 128 → 256 channels
- GroupNorm + ReLU + MaxPool
- Adaptive Average Pooling
- Final 256-dimensional backbone representation
- Linear output layer with 5 labels

Total trainable parameters:

**224,549**

## Experimental Protocol

Main configuration:

- 5 simulated clients
- 20 communication rounds
- 2 local epochs per round
- Adam optimizer
- Learning rate: 1e-3
- Effective batch size: 32
- Micro-batch size: 4
- Gradient accumulation: 8
- Seeds: 42, 123, 2026
- Classification threshold: 0.5

Final predictive evaluation is performed only on **PTB-XL fold 10**.

Representation analysis is performed separately using a fixed subset of **500 ECGs from validation fold 9**.

## Evaluation Metrics

Predictive performance is evaluated using:

- Binary Cross-Entropy Loss
- Macro-F1
- Macro-AUROC
- Exact Match

Representation consistency is evaluated using **linear Centered Kernel Alignment (CKA)**.

Two CKA views are analyzed:

- Client–Client CKA
- Local–Global CKA

Representation similarity is measured at:

- Conv1
- Conv2
- Conv3
- Conv4
- Final pooled representation

## Main Predictive Results

| Method | BCE | Macro-F1 | Macro-AUROC | Exact Match |
|---|---:|---:|---:|---:|
| Centralized | 0.3818 ± 0.0109 | 0.7024 ± 0.0060 | 0.9134 ± 0.0006 | 0.5965 ± 0.0050 |
| FedAvg | 0.5030 ± 0.0445 | 0.4416 ± 0.0379 | 0.8489 ± 0.0063 | 0.4819 ± 0.0075 |
| MOON-style | 0.4373 ± 0.0109 | 0.4679 ± 0.0460 | 0.8585 ± 0.0067 | 0.4805 ± 0.0071 |

MOON-style training improves average BCE, Macro-F1, and Macro-AUROC relative to FedAvg.

However, **Exact Match remains essentially unchanged and is slightly lower**, so the results do not support a claim that MOON-style training improves every predictive metric.

## Representation Consistency Results

At round 20, the client–client CKA gain of MOON-style training over FedAvg increases markedly with backbone depth:

- Conv1: +0.0428
- Conv2: +0.1317
- Conv3: +0.1784
- Conv4: +0.2277
- Final: +0.1875

The largest observed gain occurs at **Conv4**.

The main empirical finding is that:

**Representation-level changes are substantially larger than predictive-performance changes.**

This indicates that representation analysis provides information about federated model behavior that is not captured by predictive metrics alone.

CKA is used only as an analytical similarity measure and should not be interpreted as proof of clinical robustness, fairness, privacy, safety, or causality.

## Repository Structure

```text
federated-ecg-backbone-consistency/
│
├── code/
│   ├── step6_5_1_create_N2_partition.py
│   ├── step6_13_N2_FedAvg_seed42_matched.py
│   ├── step6_13_N2_FedAvg_seed123.py
│   ├── step6_13_N2_FedAvg_seed2026.py
│   ├── step6_13_N2_MOON_seed42_matched.py
│   ├── step6_13_N2_MOON_seed123.py
│   ├── step6_13_N2_MOON_seed2026.py
│   ├── step6_14_multiseed_evaluation_CKA.py
│   ├── step6_15_final_figures.py
│   └── step6_16_centralized_3seeds_matched.py
│
├── data_protocol/
├── results/
├── figures/
└── README.md
```

## Data Availability

PTB-XL v1.0.3 is publicly available through PhysioNet.

Raw ECG waveform files are not redistributed in this repository.

The repository contains only project-specific experimental code, derived protocol files, results, and visualization materials needed to understand and reproduce the experimental workflow.

## Authors

**Hafssa Slasli**  
Master's Student — Artificial Intelligence and Emerging Technologies (MIATE)

GitHub: HafSa-SlaSli

**Imane Bahouh**  
Co-author

Both authors contributed to the academic project.

This repository is maintained by **Hafssa Slasli**.

## Important Note

This repository presents the experimental implementation and reproducibility materials associated with the academic project.

The scientific manuscript is not redistributed here unless publication or sharing permissions explicitly allow it.
