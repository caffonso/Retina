# Retina
Pipeline for diabetic retinopatie model prediction

# RetinaMNIST + APTOS: Class-Imbalance Experiments with Entropy-Guided Resampling

## Overview

This repository contains experiments for retinal disease classification using a combined **RetinaMNIST + APTOS** dataset.

The main objective is to investigate how different class-imbalance strategies affect:

* classification performance;
* minority-class learning;
* training stability;
* balanced accuracy;
* macro-F1;
* learning dynamics across epochs.

Special attention is given to **FERS (Fuzzy Entropy-Guided Resampling)** and **FERS combined with Focal Loss (FERS-FL)**.

The experiments compare entropy-based, fuzzy entropy-based, random, and focal-loss-based resampling strategies under the same training and evaluation protocol.

---

## Datasets

Two retinal image datasets are combined:

* **RetinaMNIST**
* **APTOS**

The datasets are not trained sequentially.

Instead, corresponding splits are merged:

```text
Training:
RetinaMNIST train + APTOS train

Validation:
RetinaMNIST validation + APTOS validation

Test:
RetinaMNIST test + APTOS test
```

The original split roles are preserved to avoid information leakage between training, validation, and test sets.

During training, samples from RetinaMNIST and APTOS belong to the same combined training pool and are shuffled before each epoch.

Therefore, a single model is trained on the joint distribution of both datasets rather than first learning one dataset and then the other.

---

## Image Configuration

The experiments use retinal images resized to:

```text
224 × 224 pixels
```

The preprocessing pipeline automatically handles image normalization according to the range detected in the stored image arrays.

---

## Experimental Strategies

The following strategies are evaluated.

### 1. Baseline

Standard training without oversampling.

```text
Resampling: None
Loss: Cross-Entropy
```

---

### 2. Random Select

Minority classes are randomly oversampled until reaching the size of the largest class.

```text
Resampling: Random oversampling
Loss: Cross-Entropy
```

---

### 3. Entropy Weighted

Samples with higher predictive entropy receive higher probability of being selected during oversampling.

A warm-up model is first trained to estimate predictive uncertainty.

```text
Resampling: Entropy-weighted probabilistic oversampling
Loss: Cross-Entropy
```

---

### 4. Fuzzy Entropy Weighted

Predictive entropy is combined with a fuzzy controller that adjusts the contribution of uncertainty-based and random sampling.

The controller considers factors such as:

* entropy distribution;
* class imbalance;
* duplicate pressure;
* relative class size.

```text
Resampling: Fuzzy entropy-guided probabilistic oversampling
Loss: Cross-Entropy
```

---

### 5. FERS

**FERS — Fuzzy Entropy-Guided Resampling**

FERS prioritizes high-uncertainty samples within minority classes.

Samples are ranked according to predictive entropy and the resampling policy combines:

* deterministic high-entropy sample selection;
* a small random safety component.

The objective is to expose the classifier more frequently to difficult and uncertain minority-class examples.

```text
Resampling: FERS
Loss: Cross-Entropy
```

---

### 6. Random Select + Focal Loss

Random oversampling is combined with Focal Loss.

```text
Resampling: Random oversampling
Loss: Focal Loss
```

---

### 7. FERS + Focal Loss

FERS is combined with Focal Loss.

This configuration investigates whether uncertainty-guided resampling and a loss function that emphasizes difficult examples can complement each other.

```text
Resampling: FERS
Loss: Focal Loss
```

This strategy is referred to as:

```text
FERS-FL
```

---

# Model

The results reported below correspond to:

```text
ResNet10
```

The lightweight ResNet10 architecture was selected because it provides a good balance between computational cost and classification performance for the combined retinal dataset.

---

# Training Protocol

Experiments are performed using multiple random seeds to evaluate robustness.

Current aggregated results contain:

```text
6 completed runs per strategy
```

Training is performed for up to:

```text
40 epochs
```

with:

```text
Optimizer: Adam
Learning rate: 1e-4
Batch size: 4
```

Entropy-based methods use a short warm-up stage before the resampling scores are calculated.

---

# Model Checkpointing

Because this is a class-imbalanced classification problem, model selection based only on ordinary accuracy may favor majority classes.

The pipeline therefore saves multiple checkpoints.

```text
final_model.keras
```

Model from the final training epoch.

```text
best_accuracy_model.keras
```

Checkpoint with the highest validation accuracy.

```text
best_macro_f1_model.keras
```

Checkpoint with the highest validation macro-F1.

```text
best_balanced_accuracy_model.keras
```

Checkpoint with the highest validation balanced accuracy.

The main:

```text
best_model.keras
```

is selected according to **validation balanced accuracy**.

This criterion is used consistently across all strategies.

---

# Evaluation Metrics

The experiments report several complementary metrics.

## Accuracy

Overall classification accuracy.

---

## Macro-F1

F1-score computed independently for each class and averaged without weighting by class frequency.

Macro-F1 is particularly useful for evaluating minority-class performance.

---

## Balanced Accuracy

Average recall across classes.

Balanced accuracy is one of the main evaluation metrics because the datasets are class imbalanced.

---

## AUL-F1

**Area Under the Learning F1 trajectory**

AUL-F1 summarizes macro-F1 performance across the full training process rather than considering only a single epoch.

A higher value indicates that the model maintains good macro-F1 performance during a larger portion of training.

---

## AUL Balanced Accuracy

Analogous to AUL-F1, but computed from the balanced-accuracy trajectory.

This metric evaluates how consistently the model maintains balanced classification performance throughout training.

---

## Stability Drop

Difference between the best macro-F1 reached during training and the macro-F1 obtained at the final epoch.

Lower values indicate greater training stability.

---

# Main Results

The following results are aggregated across six completed runs.

## Validation and Learning-Trajectory Results

| Strategy         |       Best Macro-F1 | Best Balanced Accuracy |              AUL-F1 | AUL Balanced Accuracy |
| ---------------- | ------------------: | ---------------------: | ------------------: | --------------------: |
| Baseline         |     0.5711 ± 0.0185 |        0.5763 ± 0.0199 |     0.4594 ± 0.0178 |       0.4712 ± 0.0158 |
| Entropy Weighted |     0.5509 ± 0.0153 |        0.5700 ± 0.0150 |     0.4728 ± 0.0114 |       0.4918 ± 0.0114 |
| **FERS**         |     0.5713 ± 0.0187 |    **0.5886 ± 0.0205** |     0.4789 ± 0.0073 |       0.4985 ± 0.0077 |
| **FERS-FL**      |     0.5506 ± 0.0133 |        0.5709 ± 0.0199 |     0.4807 ± 0.0077 |   **0.5000 ± 0.0091** |
| Fuzzy Entropy    |     0.5420 ± 0.0129 |        0.5671 ± 0.0070 |     0.4726 ± 0.0057 |       0.4931 ± 0.0083 |
| Random Select    |     0.5515 ± 0.0208 |        0.5685 ± 0.0169 | **0.4816 ± 0.0045** |       0.4987 ± 0.0037 |
| Random + FL      | **0.5766 ± 0.0133** |        0.5819 ± 0.0122 |     0.4796 ± 0.0113 |       0.4995 ± 0.0115 |

---

# Best-Checkpoint Test Results

The following table reports test performance using the checkpoint selected by validation balanced accuracy.

| Strategy         |       Test Accuracy |       Test Macro-F1 | Test Balanced Accuracy |
| ---------------- | ------------------: | ------------------: | ---------------------: |
| Baseline         |     0.6511 ± 0.0421 |     0.5145 ± 0.0427 |        0.5385 ± 0.0190 |
| Entropy Weighted |     0.6205 ± 0.0250 |     0.4850 ± 0.0277 |        0.5223 ± 0.0269 |
| **FERS**         |     0.6454 ± 0.0291 |     0.5056 ± 0.0278 |    **0.5415 ± 0.0167** |
| FERS-FL          |     0.6402 ± 0.0282 |     0.4957 ± 0.0161 |        0.5252 ± 0.0177 |
| Fuzzy Entropy    |     0.6195 ± 0.0218 |     0.4746 ± 0.0178 |        0.5160 ± 0.0168 |
| Random Select    |     0.6230 ± 0.0247 |     0.4775 ± 0.0253 |        0.5160 ± 0.0269 |
| Random + FL      | **0.6630 ± 0.0236** | **0.5185 ± 0.0141** |        0.5408 ± 0.0200 |

---

# Main Observations

## FERS achieves the highest checkpoint-based balanced accuracy

FERS obtains the highest mean test balanced accuracy when models are selected using validation balanced accuracy:

```text
FERS:
0.5415 ± 0.0167
```

The result is close to both Random + Focal Loss and the baseline, indicating that statistical testing is necessary before claiming significant superiority.

---

## FERS also obtains the highest validation balanced-accuracy peak

```text
FERS:
0.5886 ± 0.0205
```

This suggests that entropy-guided resampling can help the classifier reach highly balanced solutions during training.

---

## FERS-FL provides the best integrated balanced-learning trajectory

FERS-FL achieves the highest AUL Balanced Accuracy:

```text
FERS-FL:
0.5000 ± 0.0091
```

compared with:

```text
Baseline:
0.4712 ± 0.0158
```

This corresponds to an improvement of approximately:

```text
+6.1% relative
```

in integrated balanced accuracy over the full learning trajectory.

This result suggests that FERS-FL improves how consistently balanced classification performance is maintained during training.

---

## Focal Loss changes the behavior of FERS

FERS and FERS-FL show different strengths.

```text
FERS
→ stronger best checkpoint

FERS-FL
→ stronger integrated learning trajectory
```

Therefore, Focal Loss does not simply increase the peak performance of FERS.

Instead, it appears to modify the temporal behavior of the learning process.

---

## Final-epoch performance can be misleading

Several resampling strategies reach their best balanced performance well before the last epoch.

For this reason, evaluating only the model from the final epoch may underestimate the quality of a strategy.

Checkpoint selection using validation balanced accuracy provides a more appropriate evaluation for the class-imbalance setting.

---

# Running the Experiments

Activate the TensorFlow environment:

```bash
source /media/public/tf220/bin/activate
```

Run the full experimental pipeline:

```bash
python run_pipeline_retina_aptos_combined_balanced.py \
    --config config_retina_aptos_combined_balanced.yaml
```

---

# Scan Dataset Before Training

The dataset configuration can be checked without starting training:

```bash
python run_pipeline_retina_aptos_combined_balanced.py \
    --config config_retina_aptos_combined_balanced.yaml \
    --scan-only
```

This prints:

* dataset sizes;
* image shapes;
* class distributions;
* training sources;
* validation sources;
* test sources.

---

# Resume Interrupted Experiments

Interrupted experiments can be resumed using:

```bash
python run_pipeline_retina_aptos_combined_balanced.py \
    --config config_retina_aptos_combined_balanced.yaml \
    --continue-training
```

---

# Output Structure

Results are organized approximately as:

```text
results/
│
├── summary.json
│
├── dataset_metadata/
│
└── runs/
    └── retinamnist_aptos_combined_224/
        └── resnet10/
            ├── baseline/
            ├── random_select/
            ├── entropy_weighted/
            ├── fuzzy_entropy_weighted/
            ├── fers/
            ├── random_select_focal_loss/
            └── fers_focal_loss/
```

Each individual run contains files such as:

```text
run_result.json
training_history.json
final_model.keras
best_model.keras
best_accuracy_model.keras
best_macro_f1_model.keras
best_balanced_accuracy_model.keras
```

---

# Reproducibility

The experiments use fixed random seeds and record:

* model;
* strategy;
* seed;
* batch size;
* learning rate;
* number of epochs;
* validation trajectories;
* test metrics;
* checkpoint-selection metric;
* training time;
* warm-up time.

The aggregated `summary.json` contains mean and standard deviation across completed runs.

---

# Experimental Interpretation

The results indicate that evaluating class-imbalance methods solely through final accuracy provides an incomplete view of their behavior.

Entropy-guided resampling affects not only the maximum classification performance but also the **trajectory through which the classifier learns minority classes**.

The current results suggest two complementary findings:

1. **FERS achieves the strongest checkpoint-based balanced performance.**
2. **FERS-FL provides the strongest integrated balanced-learning trajectory.**

These findings motivate the use of trajectory-based metrics such as **AUL-F1** and **AUL Balanced Accuracy** alongside conventional final or best-checkpoint metrics.

---

# Status

Current ResNet10 experiments:

```text
Strategies: 7
Completed runs per strategy: 6
Datasets: RetinaMNIST + APTOS
Image size: 224 × 224
Primary checkpoint metric: validation balanced accuracy
```

Further statistical analysis should include paired comparisons between strategies before making claims of statistically significant superiority.

