# Architecture

# Fire & Smoke Detection Training Pipeline

## Overview

The Fire & Smoke Detection Training Pipeline is an end-to-end framework for preparing datasets, training YOLO models, evaluating performance, and exporting optimized deployment artifacts.

The repository is designed to provide a reproducible workflow that allows researchers, engineers, and practitioners to train fire and smoke detection models on their own datasets with minimal modifications.

The pipeline consists of four major stages:

1. Dataset Standardization
2. Dataset Cleaning & Quality Assurance
3. Model Training & Evaluation
4. Deployment Preparation

---

# High-Level Architecture

```text
┌─────────────────────────┐
│  Raw YOLO Datasets      │
│ (Multiple Sources)      │
└────────────┬────────────┘
             │
             ▼
┌─────────────────────────┐
│     standardize.py      │
│ Class Remapping         │
│ Label Normalization     │
│ YAML Generation         │
└────────────┬────────────┘
             │
             ▼
┌─────────────────────────┐
│       cleaner.py        │
│ Corruption Removal      │
│ Duplicate Detection     │
│ Background Balancing    │
│ Dataset Validation      │
└────────────┬────────────┘
             │
             ▼
┌─────────────────────────┐
│    deduplication.py     │
│ DINOv2 Embeddings       │
│ FAISS Similarity Search │
│ Semantic Deduplication  │
└────────────┬────────────┘
             │
             ▼
┌─────────────────────────┐
│      eda_stats.py       │
│ Dataset Analysis        │
│ Integrity Validation    │
│ Class Distribution      │
└────────────┬────────────┘
             │
             ▼
┌─────────────────────────┐
│        train.py         │
│ YOLOv26 Training        │
│ Multi-GPU Support       │
│ Augmentation Pipeline   │
└────────────┬────────────┘
             │
             ▼
┌─────────────────────────┐
│      benchmark.py       │
│ Accuracy Evaluation     │
│ Speed Benchmarking      │
│ Cross-Dataset Testing   │
└────────────┬────────────┘
             │
             ▼
┌─────────────────────────┐
│       export.py         │
│ ONNX Export             │
│ TensorRT Export         │
└────────────┬────────────┘
             │
             ▼
┌─────────────────────────┐
│      inference.py       │
│ Video Inference         │
│ Deployment Validation   │
└─────────────────────────┘
```

---

# Dataset Engineering Pipeline

Dataset quality directly affects model performance.

Before training begins, all datasets pass through a multi-stage preparation workflow.

## Stage 1 — Dataset Standardization

Purpose:

* Normalize annotation formats
* Remap classes
* Remove unsupported classes
* Generate missing background labels
* Create a valid `data.yaml`

Input:

```text
Raw Dataset
```

Output:

```text
Standardized YOLO Dataset
```

Script:

```bash
python scripts/dataset/standardize.py
```

---

## Stage 2 — Dataset Cleaning

Purpose:

* Remove corrupt images
* Remove unreadable files
* Remove extreme aspect ratios
* Remove undersized images
* Remove exact duplicate files
* Balance background images

Input:

```text
Standardized Dataset
```

Output:

```text
Clean Dataset
```

Script:

```bash
python scripts/dataset/cleaner.py
```

---

## Stage 3 — Semantic Deduplication

Traditional duplicate detection only removes byte-identical files.

This stage removes visually similar images using deep feature embeddings.

Technology Stack:

* DINOv2
* FAISS
* HNSW Indexing
* Union-Find Clustering

Purpose:

* Remove near-duplicate CCTV frames
* Improve dataset diversity
* Reduce overfitting
* Reduce training time

Input:

```text
Clean Dataset
```

Output:

```text
Deduplicated Dataset
```

Script:

```bash
python scripts/dataset/deduplication.py
```

---

## Stage 4 — Dataset Analysis

Dataset analysis is performed before training to validate:

* Image counts
* Label counts
* Background samples
* Class distribution
* Missing image-label pairs
* Annotation integrity

Script:

```bash
python scripts/dataset/eda_stats.py
```

---

# Training Architecture

The training stage uses Ultralytics YOLO.

Current supported architecture:

```text
YOLOv26n
```

Future architectures can be added under:

```text
weights/
```

---

## Training Workflow

```text
Dataset
    │
    ▼
Data Loader
    │
    ▼
Augmentation Pipeline
    │
    ▼
YOLOv26 Backbone
    │
    ▼
Detection Head
    │
    ▼
Loss Computation
    │
    ▼
Optimizer
    │
    ▼
Checkpoint Saving
```

---

## Training Features

### Hardware Detection

Automatic detection of:

* CPU
* Single GPU
* Multi-GPU environments

### Reproducibility

The pipeline enforces:

```text
Seed = 42
Deterministic Training = Enabled
```

### Augmentations

The default augmentation strategy includes:

* HSV Augmentation
* Random Rotation
* Mosaic
* MixUp
* Close Mosaic

These augmentations were selected specifically for fire and smoke detection scenarios.

---

# Evaluation Architecture

After training, the model is evaluated using two complementary approaches.

## Accuracy Evaluation

Metrics:

* mAP@50
* mAP@50-95
* Precision
* Recall

Purpose:

```text
How accurate is the model?
```

---

## Speed Benchmarking

Metrics:

* CPU Latency
* GPU Latency
* FPS

Purpose:

```text
How fast is the model?
```

Script:

```bash
python scripts/evaluation/benchmark.py
```

---

# Deployment Architecture

Deployment artifacts are generated from trained PyTorch weights.

```text
PyTorch (.pt)
      │
      ├────────► ONNX
      │
      └────────► TensorRT
```

Supported Formats:

| Format   | Purpose                           |
| -------- | --------------------------------- |
| PyTorch  | Research                          |
| ONNX     | Cross-platform deployment         |
| TensorRT | High-performance NVIDIA inference |

---

# Inference Pipeline

The inference module validates deployment readiness using video streams.

Supported Modes:

### Standard Inference

```text
Video
  │
  ▼
YOLO
  │
  ▼
Detections
```

---

### Quadrant Inference

For small-object scenarios:

```text
Video
   │
   ▼
Frame Split (2×2)
   │
   ▼
YOLO Inference
   │
   ▼
Frame Stitching
   │
   ▼
Output Video
```

This mode improves detection performance on distant smoke and fire regions by increasing effective object scale.

---

# Repository Component Map

| Component        | Responsibility             |
| ---------------- | -------------------------- |
| standardize.py   | Dataset normalization      |
| cleaner.py       | Dataset cleaning           |
| deduplication.py | Semantic duplicate removal |
| eda_stats.py     | Dataset statistics         |
| visualize.py     | Annotation visualization   |
| train.py         | Model training             |
| benchmark.py     | Model evaluation           |
| export.py        | Model export               |
| inference.py     | Deployment inference       |

---

# Design Principles

The repository was designed around the following principles:

### Reproducibility

Experiments should be repeatable across systems.

### Data Quality First

Improving dataset quality often produces larger gains than increasing model size.

### Deployment-Oriented

Every trained model should be deployable with minimal additional engineering effort.

### Beginner Friendly

New users should be able to train on their own datasets without modifying source code.

### Extensible

New datasets, model architectures, exporters, and inference backends can be added without restructuring the repository.

---

# Future Roadmap

Planned improvements include:

* Additional YOLO model variants
* SAHI-based inference
* Real-time webcam inference
* TensorRT benchmarking
* Experiment tracking integration
* Automated hyperparameter sweeps
* Docker-based training environment
* CI/CD model validation workflows

---

# Architecture Summary

The Fire & Smoke Detection Training Pipeline provides a complete workflow for:

1. Standardizing datasets
2. Cleaning and validating annotations
3. Removing semantic duplicates
4. Training YOLO models
5. Benchmarking performance
6. Exporting deployment artifacts
7. Running production-style inference

This design enables reproducible training, consistent evaluation, and deployment-ready fire and smoke detection systems.
