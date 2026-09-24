---
title: 'The LLM Inference Stack Deconstructed: Safetensors, vLLM, Triton, and NVIDIA NIM'
date: 2026-04-10
permalink: /posts/2026/04/llm-inference-stack-deconstructed/
tags:
  - LLM Inference
  - Triton Inference Server
  - vLLM
  - TensorRT
  - NVIDIA NIM
  - MLOps
---

[Read on LinkedIn](https://lnkd.in/p/eM3wVkNW)

Spinning up an LLM in a notebook can take just a few hours. Serving thousands of requests per second in production — reliably, efficiently, and at scale — is an entirely different challenge.

Here's a clear breakdown of how the modern inference stack actually works, and why each piece exists.

---

**1. The Formats: From Storage to Silicon**

Not all model weight formats are created equal. They serve different stages of the ML lifecycle:

- **Safetensors** — The standard for safe, fast model storage and sharing. Holds the weights but contains no information about the computational graph. The execution engine reconstructs the architecture dynamically at runtime, which adds overhead.

- **ONNX** — The universal translator. Captures both weights and the computational graph, making models portable across frameworks, CPUs, GPUs, and edge devices. Trade-off: lacks deep, hardware-specific optimization.

- **TensorRT Engine** — The performance ceiling. A highly optimized, Ahead-of-Time (AOT) compiled binary built for a specific GPU. The compiler fuses multiple operations into single CUDA kernels and locks in precision formats like FP8, squeezing out maximum throughput.

---

**2. vLLM vs. Triton Inference Server**

The core question: Why does Triton running a TensorRT engine generally achieve higher throughput and lower latency than vLLM serving raw Safetensors?

It comes down to execution philosophy:

- **vLLM** is flexible and developer-friendly. It interprets the model graph dynamically at runtime, which means there's always some framework overhead on every forward pass.

- **Triton + TensorRT** eliminates that overhead almost entirely. By the time Triton runs the model, the TensorRT compiler has already fused operations into monolithic CUDA kernels tailored specifically for the target GPU. There's nothing left to interpret — just execute.

---

**3. The Orchestration Layer: Where NVIDIA NIM Comes In**

Triton + TensorRT gives you the best raw performance. But the setup — compiling engines, managing GPU-specific builds, configuring routing, and wiring up endpoints — creates significant MLOps friction.

**NIM solves exactly this problem.**

NIM containerizes the entire stack. It automatically detects your GPU architecture, pulls the appropriate pre-compiled TensorRT engine, launches Triton, and exposes a standardized API endpoint — all from a single deployment command.

The core engineering trade-off has always been raw performance vs. developer velocity. What the modern inference stack — done right — gives you is **both**.
