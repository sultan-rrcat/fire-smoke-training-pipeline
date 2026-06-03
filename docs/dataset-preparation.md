# Dataset Preparation Guide

# Overview

This guide explains how to prepare a custom dataset for training using the Fire & Smoke Detection Training Pipeline.

By the end of this guide, you will be able to:

* Organize a YOLO dataset correctly
* Standardize class labels
* Clean low-quality samples
* Remove duplicate images
* Validate dataset integrity
* Visualize annotations
* Generate a training-ready dataset

---

# Supported Dataset Format

The pipeline expects datasets in standard YOLO Detection format.

## Directory Structure

```text
dataset/

├── train/
│   ├── images/
│   └── labels/
│
├── valid/
│   ├── images/
│   └── labels/
│
├── test/
│   ├── images/
│   └── labels/
│
└── data.yaml
```

---

# YOLO Label Format

Each image must have a corresponding text file.

Example:

```text
image001.jpg
image001.txt
```

Label format:

```text
<class_id> <x_center> <y_center> <width> <height>
```

Example:

```text
0 0.542 0.421 0.210 0.315
1 0.317 0.654 0.180 0.240
```

All coordinates must be normalized between:

```text
0.0 → 1.0
```

---

# Class Definitions

The training pipeline uses:

| Class ID | Class Name |
| -------- | ---------- |
| 0        | Fire       |
| 1        | Smoke      |

Example:

```text
0 -> Fire
1 -> Smoke
```

---

# Background Images

Background images are intentionally supported.

A background image:

```text
image.jpg
image.txt
```

Where:

```text
image.txt
```

is completely empty.

Example:

```text
```

(empty file)

These samples help reduce false positives during deployment.

---

# Step 1 — Download or Collect Data

Sources may include:

* CCTV footage
* Public datasets
* Roboflow exports
* Kaggle datasets
* Internal datasets

Before proceeding:

✅ Images should be clear

✅ Labels should be reviewed

✅ Corrupt files should be removed

---

# Step 2 — Standardize Dataset

Different datasets often use different class IDs.

Examples:

Dataset A

```text
0 = Fire
1 = Smoke
```

Dataset B

```text
0 = Smoke
1 = Fire
```

Dataset C

```text
0 = Fire
1 = Other
2 = Smoke
```

To unify labels:

```bash
python scripts/dataset/standardize.py \
    --dataset-dir datasets/my_dataset \
    --map 0:0 1:2 2:1 \
    --classes fire smoke other
```

Example mapping:

```text
Old Class 0 → New Class 0
Old Class 1 → New Class 2
Old Class 2 → New Class 1
```

The standardization stage can:

* Remap classes
* Remove unwanted classes
* Generate missing labels
* Remove orphan labels
* Create a clean data.yaml

---

# Step 3 — Clean Dataset

Run:

```bash
python scripts/dataset/cleaner.py \
    --dataset-dir datasets/my_dataset
```

Cleaning removes:

* Corrupt images
* Unreadable files
* Zero-byte files
* Extreme aspect ratios
* Very small images
* Exact duplicates

The script performs in-place cleaning.

## Important

Create a backup before running cleaning operations.

---

# Step 4 — Remove Semantic Duplicates

Exact duplicate removal is not enough.

Video datasets frequently contain:

```text
Frame 001
Frame 002
Frame 003
Frame 004
```

which are visually almost identical.

Run:

```bash
python scripts/dataset/deduplication.py \
    --dataset-dir datasets/my_dataset
```

Default similarity threshold:

```text
0.985
```

Technology used:

* DINOv2 embeddings
* FAISS similarity search
* HNSW indexing
* Union-Find clustering

Benefits:

* Better generalization
* Less overfitting
* Faster training
* Smaller datasets

---

# Step 5 — Analyze Dataset

Run:

```bash
python scripts/dataset/eda_stats.py \
    --dataset-dir datasets/my_dataset
```

Example output:

```text
Images
Labels
Background Images
Class Distribution
Missing Labels
Missing Images
```

Recommended checks:

### Missing Labels

Target:

```text
0
```

### Missing Images

Target:

```text
0
```

### Malformed Annotations

Target:

```text
0
```

---

# Step 6 — Visualize Labels

Always inspect samples before training.

Run:

```bash
python scripts/dataset/visualize.py \
    --image-dir datasets/my_dataset/train/images \
    --label-dir datasets/my_dataset/train/labels \
    --classes fire smoke
```

This tool:

* Draws bounding boxes
* Displays class names
* Randomly samples images
* Helps identify annotation issues

Examples of issues:

* Incorrect boxes
* Wrong class IDs
* Missing labels
* Oversized boxes
* Tiny boxes

---

# Step 7 — Verify data.yaml

Example:

```yaml
path: datasets/my_dataset

train: train/images
val: valid/images
test: test/images

nc: 2

names:
  - fire
  - smoke
```

Verify:

* All paths exist
* Class count matches labels
* Class names are correct

---

# Recommended Dataset Quality Checklist

Before training:

## Dataset Structure

* [ ] train/images exists
* [ ] train/labels exists
* [ ] valid/images exists
* [ ] valid/labels exists
* [ ] test/images exists
* [ ] test/labels exists

## Labels

* [ ] YOLO format
* [ ] Correct class IDs
* [ ] No malformed lines
* [ ] No orphan labels

## Images

* [ ] No corrupt files
* [ ] No tiny images
* [ ] No extreme aspect ratios

## Data Quality

* [ ] Semantic duplicates removed
* [ ] Class distribution reviewed
* [ ] Background images included
* [ ] Random samples visualized

---

# Recommended Dataset Sizes

| Use Case          | Images  |
| ----------------- | ------- |
| Prototype         | 1,000+  |
| Small Project     | 5,000+  |
| Production Model  | 20,000+ |
| Large Scale Model | 50,000+ |

More important than size:

* Label quality
* Dataset diversity
* Realistic scenarios

---

# Common Mistakes

## Missing Empty Labels

Wrong:

```text
image.jpg
```

Correct:

```text
image.jpg
image.txt
```

(empty label file)

---

## Wrong Class Mapping

Wrong:

```text
0 = Smoke
```

when model expects:

```text
0 = Fire
```

Always standardize first.

---

## Skipping Visualization

Many training failures originate from:

* Incorrect annotations
* Class mapping mistakes
* Dataset export errors

Always visualize samples before training.

---

# Next Step

Once your dataset passes validation, continue to:

```text
docs/training-guide.md
```

to begin training a YOLO fire and smoke detection model.
