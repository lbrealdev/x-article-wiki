# Source

- Author: [@TheAhmadOsman](https://x.com/TheAhmadOsman)
- Article/Post: https://x.com/TheAhmadOsman/status/2040103488714068245
- Date: 2026-04-03
- Title: GPU Memory Math for LLMs (2026 Edition)

## Long summary

Running models locally, naive “model → VRAM” thinking breaks down once training and quantization are accounted for. The piece offers one core formula that covers common formats (FP16 / BF16, FP8 / INT8, GPTQ / AWQ / NF4, GGUF variants, and similar):

> **VRAM (in GB) ≈ Parameters (in billions) x (effective bits per weight ÷ 8)**

Core intuition: FP16 / BF16 → 16 bits → ~2 GB per 1B params; FP8 / INT8 → 8 bits → ~1 GB per 1B params; 4-bit quants → ~4 bits → ~0.5 GB per 1B params. GGUF schemes sit in between (Q6_K ~0.82, Q5_K ~0.69, Q4_K ~0.56, Q3_K ~0.43, Q2_K ~0.33 GB per 1B). Rule of thumb: FP16 = 2x model size, FP8 = 1x, 4-bit = 0.5x.

Weights are only part of the bill. KV cache grows with context (32K, 128K+), activations spike by runtime, batching/concurrency multiply usage (especially agent workloads), frameworks add overhead (Transformers, vLLM, TensorRT-LLM, llama.cpp), and CUDA Graphs reserve extra memory for latency/throughput stability. If you only budget for weights, you are already out of memory.

Worked sizes: 7B → ~14 / ~7 / ~3.5–4 GB (FP16 / FP8 / 4-bit); 13B → ~26 / ~13 / ~6–7 GB; 70B → ~140 / ~70 / ~35–40 GB; 405B → ~810 / ~405 / ~200+ GB. That is why people quantize aggressively, shard across GPUs (e.g. Tensor Parallelism), or move to the cloud.

Practical “what fits” by VRAM for weights: 8 GB (~3B FP16 / ~6–7B FP8 / ~12–13B 4-bit); 12 GB (~5B / ~10B / ~18–20B); 16 GB (~7B / ~13B / ~25B); 24 GB (~10–12B / ~20B / ~35–40B); 48 GB (~20–24B / ~40B / ~70–80B); 80 GB (~35–40B / ~70B / ~140B-class 4-bit). Even when math says it fits, add 10–30% extra VRAM for a safe run—and more for long context, high concurrency, or agent workflows.

MoE trap: “8x7B” sounds like 56B, but only a subset of experts run per token—compute cost ≠ memory cost. Total parameters drive memory footprint; active parameters drive speed. Depending on loading, you may still need memory for all experts or can shard them. Treating MoE like dense over- or underestimates badly.

GGUF is a container + quantization strategy for llama.cpp-style inference, CPU + GPU hybrid setups, and efficient memory use—not a universal cheat code. Those memory numbers are runtime-specific; in other frameworks weights may be dequantized and usage can jump. “It fits in 6 GB” is not universal truth.

Mental model: VRAM ≈ B x (bits ÷ 8), then adjust for runtime overhead, KV cache, and concurrency—so the question shifts from “Can I run this?” to “How do I want to run this?”

## Key claims

- VRAM (in GB) ≈ Parameters (in billions) x (effective bits per weight ÷ 8) explains FP16 / BF16, FP8 / INT8, GPTQ / AWQ / NF4, GGUF variants, and similar formats.
- FP16 / BF16 ≈ 2 GB per 1B params; FP8 / INT8 ≈ 1 GB per 1B; 4-bit ≈ 0.5 GB per 1B; GGUF Q6_K / Q5_K / Q4_K / Q3_K / Q2_K ≈ 0.82 / 0.69 / 0.56 / 0.43 / 0.33 GB per 1B.
- Model weights are only part of VRAM: KV cache, activations, batching/concurrency, framework overhead (Transformers, vLLM, TensorRT-LLM, llama.cpp), and CUDA Graphs also consume memory.
- Example weight footprints: 7B ~14 / ~7 / ~3.5–4 GB; 13B ~26 / ~13 / ~6–7 GB; 70B ~140 / ~70 / ~35–40 GB; 405B ~810 / ~405 / ~200+ GB (FP16 / FP8 / 4-bit).
- Approximate max model sizes by VRAM (weights): 8 GB → ~3B / ~6–7B / ~12–13B; 24 GB → ~10–12B / ~20B / ~35–40B; 80 GB → ~35–40B / ~70B / ~140B-class (FP16 / FP8 / 4-bit).
- Rule of thumb: add 10–30% extra VRAM for a safe run; long context (32K, 128K, etc.), high concurrency, and agent workflows need more.
- For MoE (e.g. “8x7B”), total parameters affect memory and active parameters affect speed; you may still need memory for all experts or can shard them.
- GGUF memory numbers apply to that runtime (llama.cpp-style / hybrid); other frameworks may dequantize and use far more memory.

## Actionables

- Size local LLM VRAM with VRAM ≈ Parameters (billions) x (effective bits per weight ÷ 8), using FP16 = 2x / FP8 = 1x / 4-bit = 0.5x as the baseline.
- Budget beyond weights: include KV cache, activations, batching/concurrency, and framework overhead; add at least 10–30% headroom, and more for 32K/128K+ context, high concurrency, or agent workloads.
- For MoE models, plan memory from total parameters (and loading/sharding of experts), not from active parameters alone.
- Treat GGUF “fits in N GB” figures as llama.cpp-runtime-specific; re-check memory when moving weights into other frameworks that may dequantize.

## References

### X source

- https://x.com/TheAhmadOsman/status/2040103488714068245
- https://t.co/sF6qq5uIXK

### Other

- https://pbs.twimg.com/media/HE_nHXmWwAAEshI.jpg
