# Deployment Guide

# Overview

Training a model is only half the journey.

This guide explains how to:

* Export trained models
* Convert models for deployment
* Run inference on videos
* Benchmark deployment performance
* Choose the correct runtime
* Prepare for production deployment

By the end of this guide, you will be able to deploy your Fire & Smoke Detection model using PyTorch, ONNX, or TensorRT.

---

# Deployment Workflow

```text
Train Model
     │
     ▼
best.pt
     │
     ▼
Export
     │
     ├── ONNX
     │
     └── TensorRT
     │
     ▼
Inference Validation
     │
     ▼
Production Deployment
```

---

# Supported Deployment Formats

| Format   | Extension | Purpose                   |
| -------- | --------- | ------------------------- |
| PyTorch  | .pt       | Research & Development    |
| ONNX     | .onnx     | Cross-platform Deployment |
| TensorRT | .engine   | NVIDIA GPU Deployment     |

---

# Deployment Strategy

## PyTorch

Best for:

* Research
* Validation
* Experimentation

Advantages:

* Easiest to use
* Native training format
* No conversion required

Disadvantages:

* Larger runtime overhead
* Slower than TensorRT

---

## ONNX

Best for:

* Edge devices
* Production APIs
* Cross-platform deployment

Advantages:

* Framework independent
* Widely supported
* Portable

Disadvantages:

* Requires ONNX Runtime
* Usually slower than TensorRT

---

## TensorRT

Best for:

* NVIDIA GPUs
* Real-time inference
* Production environments

Advantages:

* Lowest latency
* Highest FPS
* Optimized execution

Disadvantages:

* Hardware dependent
* Requires NVIDIA ecosystem

---

# Exporting Models

The export pipeline converts a trained PyTorch model into deployment-ready formats.

Script:

```bash
python scripts/deployment/export.py
```

---

# Export ONNX

Example:

```bash
python scripts/deployment/export.py \
    --trained-weights results/FS-baseline/weights/best.pt \
    --formats onnx
```

Output:

```text
best.onnx
```

---

# Export TensorRT

Example:

```bash
python scripts/deployment/export.py \
    --trained-weights results/FS-baseline/weights/best.pt \
    --formats tensorrt
```

Output:

```text
best.engine
```

---

# Export Multiple Formats

Recommended:

```bash
python scripts/deployment/export.py \
    --trained-weights results/FS-baseline/weights/best.pt \
    --formats onnx tensorrt
```

Outputs:

```text
best.onnx
best.engine
```

---

# FP16 Optimization

FP16 significantly improves inference speed on supported GPUs.

Enable:

```bash
python scripts/deployment/export.py \
    --trained-weights results/FS-baseline/weights/best.pt \
    --formats tensorrt \
    --half
```

Benefits:

* Lower memory usage
* Higher FPS
* Faster inference

---

# INT8 Quantization

For maximum performance:

```bash
python scripts/deployment/export.py \
    --trained-weights results/FS-baseline/weights/best.pt \
    --formats tensorrt \
    --int8
```

Benefits:

* Smaller model size
* Faster inference

Potential tradeoff:

```text
Minor accuracy reduction
```

Always benchmark after quantization.

---

# Dynamic Input Shapes

Enable dynamic resolutions:

```bash
python scripts/deployment/export.py \
    --trained-weights results/FS-baseline/weights/best.pt \
    --formats onnx \
    --dynamic
```

Benefits:

```text
Multiple input resolutions supported
```

Tradeoff:

```text
Slightly lower performance
```

---

# Choosing Input Resolution

Common options:

| Resolution | Use Case                 |
| ---------- | ------------------------ |
| 640        | Standard deployment      |
| 960        | Improved smoke detection |
| 1280       | Small smoke regions      |
| 1920       | Maximum accuracy         |

General rule:

```text
Higher Resolution
      ↓
Better Detection
      ↓
Higher Latency
```

---

# Running Inference

The repository includes a deployment validation pipeline.

Script:

```bash
python scripts/deployment/inference.py
```

---

# Standard Video Inference

Example:

```bash
python scripts/deployment/inference.py \
    --weights results/FS-baseline/weights/best.pt \
    --source demo.mp4 \
    --results-dir deployment_results
```

Pipeline:

```text
Video
   │
   ▼
YOLO Inference
   │
   ▼
Annotated Video
```

---

# ONNX Inference

Example:

```bash
python scripts/deployment/inference.py \
    --weights models/best.onnx \
    --source demo.mp4 \
    --results-dir deployment_results
```

---

# TensorRT Inference

Example:

```bash
python scripts/deployment/inference.py \
    --weights models/best.engine \
    --source demo.mp4 \
    --results-dir deployment_results
```

---

# Confidence Threshold

Default:

```text
0.50
```

Adjust:

```bash
--conf 0.30
```

Higher value:

```text
Fewer detections
Higher precision
```

Lower value:

```text
More detections
Higher recall
```

For fire safety systems:

```text
Prioritize recall
```

---

# Device Selection

Automatic:

```bash
--device auto
```

Specific GPU:

```bash
--device 0
```

CPU:

```bash
--device cpu
```

---

# Quadrant Inference Mode

Small smoke regions are difficult to detect.

To improve detection:

```bash
python scripts/deployment/inference.py \
    --weights best.pt \
    --source demo.mp4 \
    --results-dir deployment_results \
    --split-frame
```

---

# How Quadrant Inference Works

```text
Original Frame
       │
       ▼
Split Into 4 Regions
       │
       ▼
Run Inference
       │
       ▼
Merge Results
       │
       ▼
Output Video
```

Benefits:

* Better small-object detection
* Improved distant smoke detection
* Higher effective resolution

Tradeoffs:

* More processing
* Lower FPS
* Higher memory usage

---

# Deployment Benchmarking

Before production deployment, benchmark the exported model.

Example:

```bash
python scripts/evaluation/benchmark.py \
    --trained-weights best.pt
```

Record:

* mAP@50
* mAP@50-95
* Precision
* Recall
* CPU Latency
* GPU Latency
* FPS

---

# Recommended Deployment Profiles

## Development

```text
Format: PyTorch
Resolution: 640
Device: GPU
```

---

## Edge Device

```text
Format: ONNX
Resolution: 640
Device: CPU / Edge Accelerator
```

---

## Production NVIDIA Server

```text
Format: TensorRT
Resolution: 640
Precision: FP16
Device: GPU
```

---

## Maximum Accuracy

```text
Format: TensorRT
Resolution: 1280
Precision: FP16
```

Best for:

* Long-range cameras
* Industrial sites
* Large open environments

---

# Deployment Checklist

Before deployment:

* [ ] Model evaluated on test set
* [ ] Recall meets project requirements
* [ ] Benchmark results recorded
* [ ] ONNX export validated
* [ ] TensorRT export validated
* [ ] Inference tested on sample videos
* [ ] Output videos reviewed manually
* [ ] Deployment hardware verified
* [ ] Logs archived

---

# Production Recommendations

For real-world fire detection systems:

Prioritize:

```text
Recall
      ↓
Reliability
      ↓
Latency
      ↓
Precision
```

A slightly higher false alarm rate is generally preferable to missing a genuine fire event.

---

# Common Deployment Mistakes

## Exporting Without Benchmarking

Always benchmark after export.

Different runtimes may produce slightly different results.

---

## Building TensorRT On Different Hardware

TensorRT engines are often hardware-specific.

Build on the target deployment machine whenever possible.

---

## Using Extremely High Confidence Thresholds

Example:

```bash
--conf 0.90
```

This may suppress genuine fire detections.

---

## Ignoring Small Smoke Testing

Always validate:

* Distant smoke
* Small smoke plumes
* Low-light scenes
* Indoor environments
* Outdoor environments

---

# Next Steps

Once deployment validation is complete, consider:

* Live camera integration
* RTSP stream processing
* Alerting systems
* Edge deployment
* Cloud inference APIs
* Automated incident reporting

The exported model is now ready to be integrated into a complete fire and smoke monitoring system.
