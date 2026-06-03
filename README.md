# 🔥 Fire & Smoke Detection Training & Deployment Pipeline

[![Python](https://img.shields.io/badge/Python-3.9+-blue.svg)](https://www.python.org/)
[![Framework](https://img.shields.io/badge/Framework-YOLOv26-orange.svg)](https://github.com/ultralytics/ultralytics)
[![License](https://img.shields.io/badge/License-MIT-green.svg)](#license)

An end-to-end dataset engineering, quality assurance, training, evaluation, and production deployment framework optimized for Fire & Smoke Detection using **YOLOv26**.

This repository bridges the gap between raw web-scraped data and high-performance, edge-deployable inference solutions. It features robust pipelines for data cleaning, semantic deduplication via DINOv2 + FAISS, full/split-frame matrix inference, and production hardware export tools (ONNX & TensorRT).

## 🚀 Core Features

### 📊 Dataset Engineering & QA
* **Standardization:** Automated class remapping, bounding-box normalization, and dynamic training-ready YAML generator tracking.
* **Automated Cleaning:** Outlier and anomaly detection including structural corruption purging, tiny bounding box exclusion, extreme aspect ratio pruning, and automatic background null-label balancing.
* **Semantic Deduplication:** Feature extraction utilizing **DINOv2** combined with an **HNSW index via FAISS** to eliminate near-duplicate frames from video-extracted frames.

### 🏋️ Training & Advanced Evaluation
* **Unified Training:** Flexible script parsing featuring autonomous single/multi-GPU routing or clean fallback configurations to CPU hardware execution.
* **Comprehensive Benchmarking:** Direct accuracy assessment metrics (`mAP50`, `mAP50-95`, Precision, Recall) paired side-by-side with localized device inference latency benchmarks (FPS).

### 📦 Optimized Deployment
* **Hardware Compiler Export:** Standard CLI target compilation allowing seamless conversions into optimized **ONNX** runtimes or high-throughput **NVIDIA TensorRT (`.engine`)** formats.
* **Quadrant Split-and-Merge Pipeline:** Includes an advanced tiling mode that breaks video frames down into a `2×2` grid matrix for localized small-scale plume analysis before stitching bboxes back to native video scales.

## 🔧 Installation & Environment Setup

```bash
git clone http://10.10.30.65:3000/trainee-ai-ml/fire-smoke-training-pipeline.git
cd fire-smoke-training-pipeline

pip install -r requirements.txt
```

## 🛠️ Step-by-Step Production Workflow

### 1. Dataset Preparation & Validation

📖 **Documentation:** [`docs/dataset-preparation.md`](docs/dataset-preparation.md)

Review dataset structure requirements, class mappings, annotation formatting rules, and quality assurance procedures before processing data.

```bash
python scripts/dataset/standardize.py \
  --source /path/to/raw \
  --output /path/to/dataset

python scripts/dataset/cleaner.py \
  --data /path/to/dataset/data.yaml

python scripts/dataset/deduplication.py \
  --data /path/to/dataset/data.yaml \
  --threshold 0.85
```

---

### 2. Model Training

📖 **Documentation:** [`docs/training-guide.md`](docs/training-guide.md)

Review training configurations, hardware recommendations, hyperparameter tuning strategies, checkpoint management, and resume workflows.

```bash
python scripts/training/train.py \
  --weights yolov26n.pt \
  --data data.yaml \
  --imgsz 640 \
  --epochs 100 \
  --device 0
```

---

### 3. Evaluation & Benchmarking

📖 **Documentation:** [`docs/evaluation-guide.md`](docs/evaluation-guide.md)

Understand performance metrics, benchmark methodology, latency measurements, precision/recall interpretation, and deployment readiness criteria.

```bash
python scripts/evaluation/benchmark.py \
  --weights models/yolov26n/best.pt \
  --data data.yaml
```

---

### 4. Model Export & Deployment

📖 **Documentation:** [`docs/deployment-guide.md`](docs/deployment-guide.md)

Review ONNX export requirements, TensorRT compilation considerations, device compatibility, and production deployment recommendations.

```bash
python scripts/deployment/export.py \
  --trained-weights models/yolov26n/best.pt \
  --formats onnx tensorrt \
  --imgsz 640 \
  --half \
  --device 0
```

---

### 5. Inference

📖 **Documentation:** [`docs/deployment-guide.md`](docs/deployment-guide.md)

Run inference using exported models in either standard or split-frame mode.

```bash
# Standard Inference
python scripts/deployment/inference.py \
  --weights models/yolov26n/best.engine \
  --source test.mp4 \
  --results-dir result/

# Split-Frame Inference
python scripts/deployment/inference.py \
  --weights models/yolov26n/best.engine \
  --source test.mp4 \
  --results-dir result/ \
  --split-frame
```

---

### Need Help?

📖 **Architecture Overview:** [`docs/architecture.md`](docs/architecture.md)

Understand repository design, component relationships, data flow, and system architecture.

📖 **Troubleshooting Guide:** [`docs/troubleshooting.md`](docs/troubleshooting.md)

Common errors, debugging procedures, dependency conflicts, export failures, and deployment fixes.

## 📋 Best Practices

- Prioritize Recall for fire-safety applications.
- Compile TensorRT engines on the same GPU architecture used in production.

## 📄 License

MIT License

Copyright (c) 2026 Computer Vision Architecture Pipelines

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software.