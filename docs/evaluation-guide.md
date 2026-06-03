# Evaluation Guide

# Overview

Training loss alone does not determine whether a model is suitable for deployment.

The evaluation stage measures:

* Detection accuracy
* False alarm behavior
* Missed detections
* Inference latency
* Throughput (FPS)
* Deployment readiness

This repository provides a unified benchmarking pipeline through:

```bash
python scripts/evaluation/benchmark.py
```

---

# Evaluation Workflow

```text
Trained Model
      │
      ▼
Dataset Validation
      │
      ▼
Accuracy Metrics
      │
      ▼
Speed Benchmarking
      │
      ▼
Deployment Decision
```

---

# Evaluation Types

The benchmark pipeline performs two independent evaluations.

## 1. Accuracy Evaluation

Measures:

* mAP@50
* mAP@50-95
* Precision
* Recall

Purpose:

```text
How accurately does the model detect fire and smoke?
```

---

## 2. Speed Evaluation

Measures:

* CPU inference latency
* GPU inference latency
* Frames Per Second (FPS)

Purpose:

```text
Can the model run fast enough for deployment?
```

---

# Basic Evaluation

Evaluate a trained model:

```bash
python scripts/evaluation/benchmark.py \
    --trained-weights results/FS-baseline/weights/best.pt \
    --data datasets/sample-dataset/data.yaml
```

---

# Evaluating Multiple Datasets

A model may perform well on one dataset and poorly on another.

Testing on multiple datasets helps measure generalization.

Example:

```bash
python scripts/evaluation/benchmark.py \
    --trained-weights results/FS-baseline/weights/best.pt \
    --data \
    datasets/site_a/data.yaml \
    datasets/site_b/data.yaml \
    datasets/site_c/data.yaml
```

---

# Speed Benchmark Only

If you only want latency and FPS:

```bash
python scripts/evaluation/benchmark.py \
    --trained-weights results/FS-baseline/weights/best.pt
```

The script will skip dataset validation and only benchmark inference speed.

---

# Image Size Configuration

Speed tests use a synthetic input image.

Default:

```text
640 × 640
```

Custom example:

```bash
python scripts/evaluation/benchmark.py \
    --trained-weights results/FS-baseline/weights/best.pt \
    --imgsz 1280
```

Larger image sizes generally:

| Effect     | Impact   |
| ---------- | -------- |
| Accuracy   | Increase |
| Latency    | Increase |
| FPS        | Decrease |
| GPU Memory | Increase |

---

# Benchmark Warmup

Neural networks often run slower during initial iterations.

Warmup iterations prepare the model before timing begins.

Default:

```text
10
```

Example:

```bash
python scripts/evaluation/benchmark.py \
    --trained-weights results/FS-baseline/weights/best.pt \
    --warmup 20
```

---

# Benchmark Runs

Default:

```text
100
```

Example:

```bash
python scripts/evaluation/benchmark.py \
    --trained-weights results/FS-baseline/weights/best.pt \
    --runs 500
```

More runs:

* Better statistical confidence
* Longer benchmark duration

---

# Understanding Accuracy Metrics

## mAP@50

Definition:

```text
Mean Average Precision at IoU 0.50
```

Measures:

```text
How well predicted boxes overlap ground truth boxes.
```

Typical interpretation:

| mAP@50      | Quality    |
| ----------- | ---------- |
| < 0.50      | Poor       |
| 0.50 - 0.70 | Acceptable |
| 0.70 - 0.85 | Good       |
| > 0.85      | Excellent  |

---

## mAP@50-95

More strict than mAP@50.

Evaluates:

```text
IoU 0.50 → 0.95
```

This metric is preferred for comparing models.

---

## Precision

Formula:

```text
TP / (TP + FP)
```

Measures:

```text
How many detections are correct?
```

High precision means:

* Fewer false alarms
* Fewer incorrect detections

Example:

```text
Precision = 0.95

95% of detections are correct.
```

---

## Recall

Formula:

```text
TP / (TP + FN)
```

Measures:

```text
How many real fires were detected?
```

High recall means:

* Fewer missed fires
* Better safety performance

Example:

```text
Recall = 0.97

97% of fires were detected.
```

---

# Precision vs Recall

In fire detection systems:

```text
Missing a fire is often worse than a false alarm.
```

Because of this:

```text
Recall should generally be prioritized.
```

Example:

| Model | Precision | Recall |
| ----- | --------- | ------ |
| A     | 0.98      | 0.70   |
| B     | 0.92      | 0.95   |

Model B is often preferred because it misses fewer fires.

---

# Understanding Speed Metrics

## Latency

Definition:

```text
Time required to process one image.
```

Example:

```text
5 ms/image
```

Lower is better.

---

## FPS

Definition:

```text
Frames Per Second
```

Formula:

```text
FPS = 1 / Inference Time
```

Example:

```text
10 ms/image
```

equals:

```text
100 FPS
```

Higher is better.

---

# CPU vs GPU Results

Typical pattern:

| Device | Speed  |
| ------ | ------ |
| CPU    | Slower |
| GPU    | Faster |

Example:

```text
CPU:
  75 ms/image
  13 FPS

GPU:
  4 ms/image
  250 FPS
```

Actual values depend on:

* Model size
* GPU model
* Image size
* CUDA version

---

# Evaluating Multiple Experiments

Suppose you trained:

```text
FS-Nano-v1
FS-Nano-v2
FS-Nano-v3
```

Create a comparison table:

| Experiment | mAP50 | mAP50-95 | Precision | Recall |
| ---------- | ----- | -------- | --------- | ------ |
| Nano-v1    | 0.82  | 0.58     | 0.87      | 0.84   |
| Nano-v2    | 0.85  | 0.62     | 0.90      | 0.88   |
| Nano-v3    | 0.83  | 0.60     | 0.91      | 0.92   |

Choose based on deployment requirements.

---

# Fire Detection Evaluation Strategy

When evaluating fire detection systems:

Prioritize:

1. Recall
2. mAP@50-95
3. Precision
4. FPS

Reason:

```text
Missing a fire event is usually more costly than a false alarm.
```

---

# Common Evaluation Mistakes

## Using Validation Data As Test Data

Wrong:

```text
Train → Validate → Report Results
```

Correct:

```text
Train → Validate → Test
```

Always maintain a dedicated test set.

---

## Comparing Different Datasets

Wrong:

```text
Model A → Dataset A

Model B → Dataset B
```

Comparisons become meaningless.

Always benchmark on the same dataset.

---

## Ignoring FPS

A model with:

```text
99% accuracy
```

may still be unusable if:

```text
1 FPS
```

---

## Ignoring Recall

A model with:

```text
99% precision
```

can still miss fires.

Always inspect recall.

---

# Recommended Benchmark Checklist

Before deployment:

* [ ] Evaluate on test split
* [ ] Record mAP50
* [ ] Record mAP50-95
* [ ] Record Precision
* [ ] Record Recall
* [ ] Record CPU latency
* [ ] Record GPU latency
* [ ] Record FPS
* [ ] Compare against previous versions
* [ ] Save benchmark logs

---

# Example Deployment Decision

Scenario:

| Metric    | Value |
| --------- | ----- |
| mAP50     | 0.88  |
| mAP50-95  | 0.65  |
| Precision | 0.91  |
| Recall    | 0.95  |
| GPU FPS   | 210   |

Assessment:

```text
High recall
Good precision
Strong localization
Real-time capable
```

Result:

```text
Ready for deployment testing
```

---

# Recommended Experiment Tracking

Store results:

```text
results/

├── FS-Nano-v1/
│
├── FS-Nano-v2/
│
└── benchmarks/
    ├── Nano-v1.md
    ├── Nano-v2.md
    └── comparison.xlsx
```

Tracking historical performance prevents regressions.

---

# Next Step

Once a model meets accuracy and performance requirements, continue to:

```text
docs/deployment-guide.md
```

to export ONNX and TensorRT models and validate inference pipelines.
