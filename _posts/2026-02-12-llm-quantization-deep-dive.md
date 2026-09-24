---
title: 'LLM Quantization Deep Dive: NF4, AWQ, GPTQ, and GGUF — Which Should You Use?'
date: 2026-02-12
permalink: /posts/2026/02/llm-quantization-deep-dive/
tags:
  - LLM Inference
  - Quantization
  - GPU Optimization
  - vLLM
  - Production AI
---

[Read on LinkedIn](https://lnkd.in/p/eR9Y2GdR)

`load_in_4bit=True` saves memory, but it doesn't save time. Here is why.

If you've been using bitsandbytes to load Llama-3 or Mistral in 4-bit, you might think you are getting free speed optimization. You aren't. **You are trading Latency for VRAM.**

---

**The "Storage" vs. "Math" Trap**

When you load a model with bitsandbytes (NF4), you are performing **Weight-Only Quantization**:
- **Storage:** Your GPU memory holds tiny 4-bit weights (Massive VRAM savings 📉)
- **Compute:** Your GPU cannot calculate directly in 4-bit NF4

The result? For every single layer during inference, the GPU has to:
1. Fetch the 4-bit weight
2. De-quantize it back to BF16/FP16 on the fly
3. Do the math in 16-bit

This extra de-quantization step creates computational overhead, often making it **slower than FP16**.

---

**Which Quantization Should You Actually Use?**

**Scenario A: Fine-Tuning (QLoRA)**
- **Format:** bitsandbytes (NF4)
- **Why:** Uses a "NormalFloat" data type that preserves the weight distribution better than standard integers. Perfect for training, bad for production latency.

**Scenario B: Production Serving (vLLM / TGI)**
- **Format:** AWQ or GPTQ
- **Why:** These formats utilize specialized **fused kernels**. They handle the de-quantization step efficiently in the L1 cache. This actually *increases* throughput (tokens/sec) because it solves the memory bandwidth bottleneck.

**Scenario C: Local Testing / Mac / CPU**
- **Format:** GGUF (llama.cpp)
- **Why:** Optimized for Apple Silicon and hybrid inference. Allows offloading specific layers to GPU while running the rest on CPU.

**Bonus — FP8 for NVIDIA Hopper (H100/H200):**
FP8 is native compute precision on H-series GPUs. Unlike NF4, FP8 doesn't require de-quantization — the GPU calculates directly in FP8. This gives you 50% VRAM reduction **and** increased throughput simultaneously. This is what I use in production with vLLM on H100 nodes.

---

**The Bottom Line:**
- Use **NF4** to fit the model on your GPU for Fine-Tuning
- Use **AWQ/GPTQ** to serve the model to users efficiently
- Use **FP8** on H100/H200 for maximum production throughput
- Use **GGUF** to run the model on your laptop

*"VRAM is saved in both NF4 and AWQ, but Speed is not created equal."*
