# 🚀 Awesome LLM VRAM Sizing & Inference Pricing Cheatsheet (Late-2026 SOTA)

[![Interactive Calculator](https://img.shields.io/badge/Live_Interactive_Tool-modelanalysis.si%2Fhardware-38bdf8?style=for-the-badge&logo=cloudflare)](https://modelanalysis.si/hardware/)
[![API Pricing Matrix](https://img.shields.io/badge/Pricing_Matrix-modelanalysis.si%2Fpricing-10b981?style=for-the-badge)](https://modelanalysis.si/pricing.md)
[![License](https://img.shields.io/badge/License-CC_BY_4.0-blue?style=for-the-badge)](LICENSE)

The open-source reference guide for sizing GPU VRAM (weights + KV cache) across **NVIDIA RTX 50-Series (RTX 5090 32GB, RTX 5080 16GB)**, **Blackwell B200 / H200**, and benchmarking late-2026 frontier AI models (**GPT-6.1 Sol Pro**, **Claude Sonnet 5.5**, **Gemini 4 Argon**, **DeepSeek V4.1 Flash**, **Qwen 3.8 Max Prime**).

> ⚡ **Need interactive sliders, custom context horizons, or multi-GPU offloading math?**  
> Use the free, zero-ad interactive web engine at **[https://modelanalysis.si/hardware/](https://modelanalysis.si/hardware/)**.

---

## 🧮 1. The Real LLM VRAM Sizing Formula

Most online guides only measure model weights and completely ignore the **Key-Value (KV) Cache** and **CUDA runtime overhead**. This is why your GPU runs Out-of-Memory (OOM) the moment your prompt exceeds a few thousand tokens.

$$\text{Total VRAM Required} = \text{Model Weights} + \text{KV Cache} + \text{CUDA Overhead (\sim 1.2 - 2.0 GB)}$$

### A. Model Weights Math
$$\text{Weight VRAM (GB)} \approx \text{Parameter Count (Billions)} \times \text{Bytes per Parameter} \times 1.15$$

| Quantization Format | Bits per Weight | Bytes per Param | Example: 70B Model | Example: 671B MoE (28B Active) |
|---|---|---|---|---|
| **FP16 / BF16** (Unquantized) | 16-bit | 2.0 bytes | ~140 GB VRAM | ~1,200 GB VRAM |
| **FP8 / INT8** | 8-bit | 1.0 byte | ~72 GB VRAM | ~650 GB VRAM |
| **Q4_K_M / INT4** | 4-bit | ~0.55 bytes | ~40 GB VRAM | ~360 GB VRAM (Native 32GB on RTX 5090 via KTransformers) |

---

### B. KV Cache Math (The Silent VRAM Killer)

For modern models using Grouped-Query Attention (GQA) like Llama 3.3 and Qwen 3.8:

$$\text{KV Cache (GB)} = \frac{2 \times \text{Layers} \times \text{KV Heads} \times \text{Head Dim} \times \text{Context Length} \times \text{Bytes per Token}}{10^9}$$

* At **8k context** on a 70B model, KV cache takes ~1.3 GB.
* At **32k context**, KV cache expands to **~5.2 GB**.
* At **128k context**, KV cache alone consumes **~21 GB**!

---

## 🖥️ 2. Modern GPU Hardware Compatibility Matrix (RTX 50-Series & Blackwell)

| Hardware Configuration | Total VRAM & Memory | Supported Open Models (FP16) | Supported Open Models (FP8) | Supported Open Models (Q4_K_M) |
|---|---|---|---|---|
| **1× RTX 5070 / 4070** | **12 GB GDDR7** | 8B (8k ctx) | 8B (16k ctx) | 14B (8k ctx) |
| **1× RTX 5080 / 4080** | **16 GB GDDR7** | 8B (32k ctx) | 14B (8k ctx) | 27B / 32B (8k ctx) |
| **1× RTX 5090 (New Flagship)** | **32 GB GDDR7** | **14B (128k ctx)** | **32B (32k ctx)** | **70B (16k ctx natively!)** / 671B MoE via KTransformers |
| **2× RTX 5090** | **64 GB GDDR7** | 32B (64k ctx) | **70B (32k ctx comfortably)** | **70B (128k full context)** |
| **Apple Mac Studio M3 Ultra** | **128 GB – 192 GB Unified** | 70B (32k ctx) | 70B (128k ctx) | **DeepSeek 671B MoE (Full Local Execution)** |
| **1× NVIDIA H200 (141GB HBM3e)** | **141 GB HBM3e** | 70B (32k ctx) | 70B (128k ctx) | Multi-model Concurrent Serving |
| **NVIDIA Blackwell B200** | **192 GB HBM3e** | 120B+ (32k ctx) | DeepSeek 671B MoE | Multi-tenant High-Throughput Cluster |

> 🔗 Calculate your exact hardware configuration on [ModelAnalysis.si Hardware Engine](https://modelanalysis.si/hardware/).

---

## 💰 3. Late-2026 Frontier Model Economics (per 1M Tokens)

| Model Name | Lab / Provider | Input Rate | Cached Input Rate | Output Rate | SWE-bench Verified | Arena Elo | Context Horizon |
|---|---|---|---|---|---|---|---|
| **OpenAI GPT-6.1 Sol Pro** | OpenAI | **$2.00** | **$0.50** (-75%) | **$8.00** | **98.8%** | **1420 pts** | 1M |
| **Claude Sonnet 5.5** | Anthropic | **$2.00** | **$0.20** (-90%) | **$10.00** | **98.5%** | **1415 pts** | 500k |
| **Gemini 4 Argon** | Google DeepMind | **$1.50** | **$0.37** (-75%) | **$6.00** | **97.2%** | **1395 pts** | **2M** |
| **Qwen 3.8 Max Prime** | Alibaba Qwen | **$0.40** | **$0.10** (-75%) | **$1.20** | **96.0%** | **1380 pts** | 512k |
| **Kimi K3.1 Pro** | Moonshot AI | **$0.30** | **$0.07** (-75%) | **$1.20** | **95.5%** | **1370 pts** | 256k |
| **DeepSeek V4.1 Flash** | DeepSeek | **$0.15** | **$0.015** (-90%) | **$0.60** | **94.5%** | **1365 pts** | 256k |
| **DeepSeek R1 (Open)** | DeepSeek | **$0.55** | **$0.14** (-75%) | **$2.19** | **95.0%** | **1358 pts** | 128k |
| **Mistral Large 4.0** | Mistral AI | **$1.20** | **$0.30** (-75%) | **$3.60** | **95.0%** | **1375 pts** | 256k |

*Telemetry verified and tracked across 280+ models via [ModelAnalysis.si Pricing Protocol](https://modelanalysis.si/pricing.md).*

---

## 🛠️ Quick-Start Deployment Snippets

### Run Qwen 3.8 / Llama 3.3 70B on 1× RTX 5090 32GB (vLLM)
```bash
vllm serve Qwen/Qwen3.8-72B-Instruct-AWQ \
  --tensor-parallel-size 1 \
  --max-model-len 16384 \
  --gpu-memory-utilization 0.95
```

### Run DeepSeek R1 671B MoE with KTransformers (Single RTX 5090 32GB + 256GB Host RAM)
```bash
ktransformers --model_path deepseek-ai/DeepSeek-R1 \
  --optimize_rule_path ktransformers/optimize/optimize_rules/DeepSeek-V3-Chat.yaml \
  --max_new_tokens 2048
```

---

## 🔗 Useful Resources
* 🌐 **Interactive Hardware Sizing**: [https://modelanalysis.si/hardware/](https://modelanalysis.si/hardware/)
* ⚡ **Token Spend & Caching Simulator**: [https://modelanalysis.si/simulator/](https://modelanalysis.si/simulator/)
* 📊 **Multi-Metric Pareto Frontier**: [https://modelanalysis.si/overview](https://modelanalysis.si/overview)
* 🤖 **Autonomous Agent Protocol**: [https://modelanalysis.si/llms.txt](https://modelanalysis.si/llms.txt)
* 🧠 **MA-IQ Composite Index**: [https://modelanalysis.si/intelligence/](https://modelanalysis.si/intelligence/)

---

## 📄 License
CC BY 4.0 International. Created and maintained by [ModelAnalysis.si](https://modelanalysis.si).
