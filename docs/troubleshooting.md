# Troubleshooting Guide

# Frequently Asked Questions (FAQ)

This document covers the most common issues encountered while preparing datasets, training models, evaluating results, and deploying fire and smoke detection systems.

---

# Dataset Preparation

## Q: The training script says "Dataset YAML not found". What should I do?

### Symptoms

```text
Dataset YAML not found
```

### Solution

Verify that:

```text
dataset/
└── data.yaml
```

exists.

Check your command:

```bash
python scripts/training/train.py \
    --data datasets/my_dataset/data.yaml
```

Make sure the path is correct and accessible.

---

## Q: Training starts but reports zero images.

### Symptoms

```text
Train images: 0
Val images: 0
```

### Solution

Verify your dataset structure:

```text
dataset/

├── train/
│   └── images/
│
├── valid/
│   └── images/
│
└── test/
    └── images/
```

Also verify:

```yaml
train: train/images
val: valid/images
test: test/images
```

inside `data.yaml`.

---

## Q: Why are some images missing after running cleaner.py?

### Explanation

The cleaner intentionally removes:

* Corrupt images
* Zero-byte files
* Extreme aspect ratios
* Tiny images
* Exact duplicates

### Recommendation

Review the generated log file before retraining.

Always create a backup before running:

```bash
python scripts/dataset/cleaner.py
```

---

## Q: Why did cleaner.py remove my background images?

### Explanation

The cleaning pipeline maintains a target background ratio.

Excess background images may be removed to keep the dataset balanced.

### Recommendation

Review:

```python
TARGET_BACKGROUND_RATIO
```

inside the cleaning configuration if customization is required.

---

## Q: Why do I have empty label files?

### Explanation

Empty label files represent background images.

Example:

```text
image.jpg
image.txt
```

where:

```text
```

is intentionally empty.

This is expected behavior.

Do not delete these files.

---

## Q: My labels exist but objects are missing after standardization.

### Cause

Class mappings may have removed those annotations.

Example:

```bash
--map 0:0 1:1
```

will discard any class not listed.

### Solution

Verify class mappings before running:

```bash
python scripts/dataset/standardize.py
```

---

## Q: Why did standardize.py create new label files?

### Explanation

The script automatically creates empty labels for background images.

This ensures YOLO compatibility.

---

## Q: Why are duplicate images still present after cleaning?

### Explanation

The cleaner removes exact duplicates only.

Example:

```text
Same file hash
```

Near-identical CCTV frames require semantic deduplication.

Run:

```bash
python scripts/dataset/deduplication.py
```

---

## Q: Deduplication removed too many images.

### Cause

Similarity threshold is too low.

### Solution

Increase threshold.

Example:

```bash
python scripts/dataset/deduplication.py \
    --threshold 0.99
```

Higher threshold = fewer removals.

---

## Q: Deduplication is very slow.

### Explanation

Semantic deduplication uses:

* DINOv2
* FAISS
* HNSW Search

This is significantly more expensive than file-hash comparison.

### Recommendation

Use:

* SSD storage
* CUDA GPU
* Smaller batch sizes if memory is limited

---

## Q: How do I verify annotation quality?

### Solution

Visualize samples:

```bash
python scripts/dataset/visualize.py \
    --image-dir train/images \
    --label-dir train/labels \
    --classes fire smoke
```

Inspect:

* Missing boxes
* Wrong labels
* Incorrect class IDs
* Extremely small boxes

---

# Training

## Q: Training crashes with CUDA Out Of Memory.

### Symptoms

```text
CUDA out of memory
```

### Solutions

Reduce batch size:

```bash
--batch-size 32
```

Close other GPU-intensive applications.

---

## Q: GPU is not being used.

### Symptoms

```text
Training on CPU
```

### Check

Verify CUDA:

```bash
nvidia-smi
```

Verify PyTorch:

```python
import torch

print(torch.cuda.is_available())
```

Expected:

```text
True
```

---

## Q: Training is extremely slow.

### Possible Causes

* CPU training
* HDD storage
* Large image sizes
* Excessive worker count

### Recommendations

Use:

```bash
--device auto
```

Install CUDA-enabled PyTorch.

Store datasets on SSD.

---

## Q: Why is training stuck at 0%?

### Possible Causes

* Dataset path issues
* Corrupted labels
* Worker deadlocks
* Network-mounted storage

### Try

```bash
--workers 0
```

If training starts, the issue is likely worker-related.

---

## Q: Can I stop training and continue later?

### Yes

Resume using:

```bash
python scripts/training/train.py \
    --weights results/FS-experiment/weights/last.pt \
    --resume
```

---

## Q: Which metric matters most for fire detection?

### Recommendation

Prioritize:

```text
Recall
```

Reason:

```text
Missing a fire is generally worse than a false alarm.
```

---

# Evaluation

## Q: My mAP is very low. What should I check?

### Checklist

* Verify labels
* Verify class mappings
* Remove duplicates
* Inspect visualizations
* Review class balance

Most low-mAP issues originate from dataset quality.

---

## Q: Training mAP is high but test mAP is low.

### Cause

Overfitting.

### Solutions

* Add more data
* Increase diversity
* Add backgrounds
* Run semantic deduplication
* Reduce training epochs

---

## Q: Precision is high but recall is low.

### Meaning

The model is conservative.

It avoids false alarms but misses fires.

### Possible Fixes

* More fire examples
* Better augmentation
* Larger training dataset

---

## Q: Recall is high but precision is low.

### Meaning

The model detects most fires but produces many false alarms.

### Possible Fixes

* Add background images
* Improve negative samples
* Review annotation quality

---

## Q: Why are benchmark results different on another machine?

### Explanation

Performance depends on:

* GPU model
* CUDA version
* Drivers
* CPU
* RAM
* Storage

Always compare results using the same hardware configuration.

---

# Export & Deployment

## Q: ONNX export failed.

### Check

Verify:

```bash
pip install onnx onnxruntime
```

Also ensure:

```text
best.pt
```

exists.

---

## Q: TensorRT export failed.

### Common Causes

* TensorRT not installed
* CUDA mismatch
* Unsupported GPU

### Recommendation

Verify:

```bash
nvidia-smi
```

and TensorRT installation.

---

## Q: Can I build TensorRT on one machine and deploy on another?

### Recommendation

Build TensorRT engines on the target deployment machine whenever possible.

TensorRT engines are often hardware-specific.

---

## Q: Inference is slower than benchmark results.

### Explanation

Benchmarks measure:

```text
Raw model inference
```

Real deployment includes:

* Video decoding
* Frame resizing
* Post-processing
* Disk I/O
* Rendering

Actual FPS is usually lower.

---

## Q: Small smoke regions are not detected.

### Solutions

Use:

```bash
--imgsz 1280
```

or

Use quadrant inference mode.

Small objects benefit from larger effective resolution.

---

## Q: Output video is not being saved.

### Verify

* Output directory exists
* Write permissions are available
* Disk space is sufficient

---

# General Questions

## Q: Which script should I run first?

### Recommended Order

```text
standardize.py
      ↓
cleaner.py
      ↓
deduplication.py
      ↓
eda_stats.py
      ↓
visualize.py
      ↓
train.py
      ↓
benchmark.py
      ↓
export.py
      ↓
inference.py
```

---

## Q: Which logs should I keep?

Recommended:

```text
logs/

├── training logs
├── benchmark logs
├── export logs
└── inference logs
```

Logs are invaluable when reproducing experiments.

---

## Q: I found a bug. What information should I provide?

Include:

* Operating System
* Python Version
* CUDA Version
* GPU Model
* Full Command
* Complete Error Traceback
* Relevant Log Files

This information dramatically reduces debugging time.

---

# Still Need Help?

Before opening an issue:

* Run dataset validation
* Visualize annotations
* Review logs
* Reproduce the issue with a minimal example

Most issues can be resolved by validating dataset quality and configuration before training.
