---
title: 'LLM Model Sizing: How to Calculate VRAM Requirements for GPU Deployment'
date: 2026-08-15
permalink: /posts/2025/08/llm-model-sizing/
tags:
  - LLM Inference
  - GPU Infrastructure
  - VRAM
  - Model Deployment
  - Hardware-Aware AI
---

[Read on LinkedIn](https://lnkd.in/p/eaFWsUqH)

One of the most critical — and often underestimated — steps in deploying LLMs is understanding **how much GPU memory your model actually needs**.

This post breaks down the full VRAM calculation framework I use when sizing infrastructure for production LLM deployments:

**What goes into VRAM?**
- **Model weights** — depends on parameter count and precision (FP32, FP16, BF16, FP8, INT4)
- **KV-Cache** — scales with sequence length, batch size, and number of attention heads
- **Activation memory** — often overlooked but significant during inference
- **Framework overhead** — CUDA context, runtime buffers

**Quick formula:**
Model VRAM ≈ `(params × bytes_per_param) + KV_cache + activations + overhead`

**Example — Llama 3.1 70B at FP16:**
~140GB just for weights → needs 2× H100 (80GB each) at minimum

**Quantization impact:**
- FP16 → 2 bytes/param
- FP8 → 1 byte/param (50% reduction!)
- INT4 → 0.5 bytes/param (75% reduction!)

Getting this right before deployment saves massive infrastructure costs and prevents OOM failures in production.
