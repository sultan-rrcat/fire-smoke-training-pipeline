# Training Guide

# Overview

This guide explains how to train a fire and smoke detection model using the Fire & Smoke Detection Training Pipeline.

By the end of this guide, you will be able to:

* Configure your dataset
* Select training weights
* Launch training
* Resume interrupted experiments
* Evaluate results
* Understand generated outputs

---

# Prerequisites

Before training, ensure that:

* Python is installed
* CUDA is installed (optional but recommended)
* Dataset preparation is complete
* `data.yaml` is valid
* Dependencies are installed

Refer to:

```text
docs/dataset-preparation.md
```

before proceeding.

---

# Hardware Recommendations

## Minimum Requirements

| Component | Requirement |
| --------- | ----------- |
| CPU       | 4+ Cores    |
| RAM       | 8 GB        |
| GPU       | Optional    |
| Storage   | 10 GB Free  |

---

## Recommended

| Component | Recommendation    |
| --------- | ----------------- |
| CPU       | 8+ Cores          |
| RAM       | 32 GB             |
| GPU       | RTX 3060 / Better |
| VRAM      | 8 GB+             |
| Storage   | SSD               |

---

## Large Dataset Training

| Component | Recommendation |
| --------- | -------------- |
| CPU       | 12+ Cores      |
| RAM       | 64 GB          |
| GPU       | Multiple GPUs  |
| VRAM      | 16 GB+         |

---

# Installation

Clone repository:

```bash
git clone <repository-url>

cd FIRE-SMOKE-TRAINING
```

Install dependencies:

```bash
pip install -r requirements.txt
```

Verify installation:

```bash
python scripts/training/train.py --help
```

Expected output:

```text
usage: train.py [options]
```

---

# Dataset Configuration

Ensure your dataset follows:

```text
dataset/

├── train/
├── valid/
├── test/
└── data.yaml
```

Example:

```yaml
path: datasets/sample-dataset

train: train/images
val: valid/images
test: test/images

nc: 2

names:
  - fire
  - smoke
```

---

# Choosing Initial Weights

The pipeline supports transfer learning.

Repository weights directory:

```text
weights/
└── yolov26n/
    └── yolo26n.pt
```

Available options:

| Weight     | Purpose         |
| ---------- | --------------- |
| yolo26n.pt | Fast training   |
| Custom .pt | Fine tuning     |
| last.pt    | Resume training |

---

# Quick Start Training

Basic command:

```bash
python scripts/training/train.py \
    --data datasets/sample-dataset/data.yaml \
    --results-dir results \
    --weights weights/yolov26n/yolo26n.pt \
    --name baseline
```

---

# Understanding Training Parameters

## Required Parameters

### Dataset

```bash
--data
```

Example:

```bash
--data datasets/sample-dataset/data.yaml
```

---

### Results Directory

```bash
--results-dir
```

Example:

```bash
--results-dir results
```

All experiment outputs will be stored here.

---

### Starting Weights

```bash
--weights
```

Example:

```bash
--weights weights/yolov26n/yolo26n.pt
```

---

### Experiment Name

```bash
--name
```

Example:

```bash
--name fire-smoke-v1
```

Output folder:

```text
results/
└── FS-fire-smoke-v1/
```

---

# Common Hyperparameters

## Epochs

```bash
--epochs
```

Default:

```text
150
```

Example:

```bash
--epochs 300
```

---

## Batch Size

```bash
--batch-size
```

Default:

```text
192
```

Adjust according to GPU memory.

Example:

```bash
--batch-size 32
```

---

## Image Size

```bash
--imgsz
```

Default:

```text
640
```

Examples:

```bash
--imgsz 640
```

```bash
--imgsz 1280
```

Larger images improve small-object detection but require more memory.

---

## Early Stopping

```bash
--patience
```

Default:

```text
20
```

Training stops if no improvement is observed.

Example:

```bash
--patience 50
```

---

## Workers

```bash
--workers
```

Default:

```text
8
```

Increase if CPU resources allow.

---

# Device Selection

## Automatic Detection

Recommended:

```bash
--device auto
```

The pipeline automatically detects:

* CPU
* Single GPU
* Multi-GPU environments

---

## Specific GPU

```bash
--device 0
```

---

## Multiple GPUs

```bash
--device 0,1
```

Example:

```bash
python scripts/training/train.py \
    --data datasets/sample-dataset/data.yaml \
    --results-dir results \
    --weights weights/yolov26n/yolo26n.pt \
    --name multi-gpu \
    --device 0,1
```

---

## CPU Training

```bash
--device cpu
```

Useful for testing only.

---

# Training Augmentations

The pipeline automatically applies domain-specific augmentations.

## HSV Augmentation

Simulates:

* Lighting changes
* Camera differences
* Weather variations

---

## Rotation

Range:

```text
±10°
```

Improves robustness to camera angle variation.

---

## Mosaic

Combines multiple images into a single training sample.

Benefits:

* Better context learning
* Improved small object detection

---

## MixUp

Blends images together.

Benefits:

* Regularization
* Reduced overfitting

---

## Close Mosaic

Disables mosaic near the end of training.

Purpose:

```text
Final epochs should resemble real deployment images.
```

---

# Debug Mode

Useful when validating a new dataset.

Example:

```bash
python scripts/training/train.py \
    --data datasets/sample-dataset/data.yaml \
    --results-dir results \
    --weights weights/yolov26n/yolo26n.pt \
    --name debug \
    --debug
```

Debug mode:

* Uses a small dataset fraction
* Trains for minimal epochs
* Finishes quickly

Recommended before launching long experiments.

---

# Resuming Training

If training was interrupted:

```bash
python scripts/training/train.py \
    --data datasets/sample-dataset/data.yaml \
    --results-dir results \
    --weights results/FS-baseline/weights/last.pt \
    --name baseline \
    --resume
```

Training continues from the previous checkpoint.

---

# Training Outputs

After completion:

```text
results/

└── FS-baseline/
    ├── weights/
    │   ├── best.pt
    │   └── last.pt
    │
    ├── results.csv
    ├── results.png
    ├── confusion_matrix.png
    ├── PR_curve.png
    ├── F1_curve.png
    └── labels.jpg
```

---

# Understanding Key Metrics

## mAP@50

Measures detection quality at IoU 0.50.

Higher is better.

---

## mAP@50-95

More strict metric.

Recommended for comparing models.

---

## Precision

Measures:

```text
False Alarm Resistance
```

High precision:

```text
Fewer false fire alerts
```

---

## Recall

Measures:

```text
Detection Coverage
```

High recall:

```text
Fewer missed fires
```

For fire safety systems, recall is often the most important metric.

---

# Best Practices

## Always Start With a Baseline

Train without changing defaults.

Record results.

Only then modify hyperparameters.

---

## Change One Variable At A Time

Avoid:

```text
Changing:
- Batch size
- Epochs
- Image size
- Optimizer

all together.
```

You won't know what caused improvements.

---

## Keep Experiment Names Meaningful

Good:

```text
Nano-640-v1
Nano-1280-v1
Nano-NoMixup-v1
```

Bad:

```text
test
test2
final
final_final
```

---

# Common Problems

## CUDA Out Of Memory

Reduce:

```bash
--batch-size
```

or

```bash
--imgsz
```

---

## Dataset Not Found

Verify:

```bash
--data
```

path is correct.

---

## Very Low mAP

Check:

* Labels
* Class mappings
* Visualization samples
* Dataset quality

---

## Overfitting

Symptoms:

```text
Training mAP ↑
Validation mAP ↓
```

Solutions:

* More data
* More backgrounds
* Better augmentation
* Semantic deduplication

---

# Next Step

After training completes, continue with:

```text
docs/evaluation-guide.md
```

to benchmark accuracy, latency, and deployment readiness.
