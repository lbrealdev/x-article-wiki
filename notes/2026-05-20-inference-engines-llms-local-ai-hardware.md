# Source

- Author: [@TheAhmadOsman](https://x.com/TheAhmadOsman)
- Article/Post: https://x.com/TheAhmadOsman/status/2057183854444843202
- Date: 2026-05-20
- Title: Inference Engines for LLMs & Local AI Hardware (2026 Edition)

## Long summary

Part 3 of a Self-hosted LLMs / Local AI series (after GPU Memory Math and Memory Bandwidth). Core framing: do not pick an inference engine first — pick a hardware strategy, workload shape, and serving model; the engine follows. The engine is not the model; it is the traffic cop, memory manager, kernel dispatcher, scheduler, cache accountant, parallelism planner, API surface, and sometimes the deployment framework.

One-page decision guide: Laptop / edge / odd hardware → llama.cpp; Mac-first → MLX / MLX-LM; single RTX local → ExLlamaV2; 2-4+ NVIDIA / CUDA GPUs → ExLlamaV3; general production → vLLM; long-context / MoE / routing → SGLang; NVIDIA max performance → TensorRT-LLM; cluster orchestration → NVIDIA Dynamo.

Workload phases: prefill (compute-intensive, builds KV cache) vs decode (memory-bandwidth-bound, one token at a time). That split explains short-prompt/long-answer (decode/bandwidth/batching), long-prompt/short-answer (prefill/attention/chunked prefill), many users (scheduler/continuous batching/cache paging), long context (KV cache/paged attention/KV quantization), MoE (expert routing/parallelism), and multi-node (interconnect/NVLink/RDMA/pipeline/disaggregation). Recurring theme: inference performance is memory movement plus scheduling (PagedAttention, FlashAttention, speculative decoding).

Real bottlenecks: (1) memory bandwidth, not just VRAM — VRAM determines fit, bandwidth determines decode speed; Apple's M3 Ultra up to 819 GB/s unified-memory bandwidth vs NVIDIA H100 SXM 3.35 TB/s GPU memory bandwidth; fit is not speed; (2) KV cache growth with batch and context; (3) interconnect once models cross GPUs (tensor / pipeline / expert parallelism; without NVLink, pipeline can beat tensor); (4) scheduler quality; (5) runtime overhead (CUDA graphs, kernel fusion, sampling, tokenizer, HTTP, LoRA switching, structured decoding).

Four engine families: portable local (llama.cpp, MLC LLM, ONNX Runtime GenAI, OpenVINO, Ollama-style tools); Apple/unified-memory (MLX, MLX-LM); consumer CUDA quant (ExLlamaV2, ExLlamaV3); production serving (vLLM, SGLang, TensorRT-LLM, TGI, LMDeploy) — plus orchestration layers like Dynamo.

- **llama.cpp:** portability king (Apple Silicon NEON/Accelerate/Metal; x86 AVX/AVX2/AVX512/AMX; RISC-V; CUDA; AMD HIP; MUSA; Vulkan; SYCL; hybrid offload; GGUF). llama-server is OpenAI-compatible with Anthropic Messages, reranking, continuous batching, multimodal, JSON schema, function calling, speculative decoding, web UI. Not for serious multi-node production; RPC backend is proof-of-concept, fragile, and insecure. Do not use on multi-GPU setups (prefer vLLM or ExLlamaV2).
- **MLX / MLX-LM:** Mac-first unified-memory stack; arrays live in shared memory. Large quantized models can fit where a 24 GB consumer GPU cannot, but are slower. Adds Hugging Face Hub, quantization, LoRA/full fine-tuning, distributed inference (MPI, Ring over TCP, JACCL over Thunderbolt, NCCL for CUDA); CUDA/CPU Linux packages exist. MLX-LM server warns it is not recommended for production (basic security only).
- **ExLlamaV2 / V3:** consumer CUDA quant engines. V2: paged attention, dynamic batching, prompt caching, KV dedup, speculative decoding — one RTX 3090/4090/5090, EXL2, local coding/chat. V3: EXL3 (QTIP-based), tensor/expert parallel for 2-4+ consumer GPUs or local MoE, TabbyAPI OpenAI-compatible server, continuous dynamic batching, multimodal; some models lack TP/EP support.
- **vLLM:** default open-source production server — PagedAttention, continuous batching, chunked prefill, prefix caching, CUDA/HIP graphs, broad quantization (FP8, MXFP8/MXFP4, NVFP4, INT8, INT4, GPTQ, AWQ, GGUF), speculative decoding, torch.compile, disaggregated prefill/decode/encode, many parallelism modes and APIs, multi-vendor backends. Multi-node typically uses Ray; still needs systems tuning.
- **SGLang:** for ugly workloads (structured outputs, long context, MoE, disaggregation, routing). RadixAttention prefix caching and prefill-decode disaggregation are differentiators.
- **TensorRT-LLM:** NVIDIA-max-performance (custom kernels, disaggregation, Wide Expert Parallelism, Dynamo/Triton integration). B200 can load FP4; H100+ FP8 can double performance and halve memory vs 16-bit with minimal accuracy loss. Shines on H100/H200/B200/GB200/GB300 fleets; awkward for AMD/Apple/Intel portability and small local setups.

Also covered: TGI (HF integration), MLC LLM / WebLLM (browser/mobile/apps), ONNX Runtime GenAI (Foundry Local, Windows ML, VS Code AI Toolkit; CPU/CUDA/DirectML/TensorRT-RTX/OpenVINO/QNN/WebGPU/AMD), OpenVINO GenAI (Intel Xeon/Arc/Core Ultra/NPUs), LMDeploy (TurboMind + PyTorch), NVIDIA Dynamo (fleet orchestration above engines). Explicit: DO NOT USE Ollama.

Hardware recipes map CPU-only, Mac, single/dual-quad RTX, 8×H100/H200, B200/GB200/GB300, AMD MI300–MI355, Intel, and browser/mobile stacks to those engines. Benchmarking must include model/weights/engine/hardware/workload detail and metrics (TTFT, TPOT, p50/p95/p99, tok/s, RPS, GPU memory, KV hit rate, prefill/decode throughput, cost per 1M tokens) — never single-user tok/s alone. Common mistakes: choosing by VRAM alone; tensor parallelism on weak interconnect; ignoring KV cache; treating local engines as production; assuming quantization formats are portable; ignoring architecture; trusting charts without workload shape (e.g. Llama 3.1 8B @ 1K/128 vs coding agent on Qwen 3.6 27B / Gemma 4 26B-A4B or 500-user RAG).

Opinionated map: local user → LM Studio or Harbor for convenience, llama.cpp for control, MLX on Mac, ExLlamaV2/V3 for CUDA; local agent → llama.cpp / MLX / vLLM by goal; internal team → vLLM (SGLang when structured/long-context/multi-LoRA/MoE/routing matter); customer scale → bake-off vLLM, SGLang, TensorRT-LLM (+ Dynamo when routing/disaggregation matter). Final principle: answer hardware, fit, prefill vs decode, context/concurrency, prefix sharing, architecture, convenience vs serving vs fleet, quant kernels, interconnect, and optimization target — then pick the engine.

## Key claims

- Pick hardware strategy, workload shape, and serving model first; the inference engine follows.
- Prefill is compute-intensive; decode is memory-bandwidth-bound — that distinction drives almost all engine/hardware choices.
- VRAM (capacity) determines fit; bandwidth determines decode speed — Apple's M3 Ultra up to 819 GB/s unified memory vs NVIDIA H100 SXM 3.35 TB/s; fit is not speed.
- Decision map: llama.cpp (laptop/edge/odd hardware); MLX / MLX-LM (Mac-first); ExLlamaV2 (single RTX); ExLlamaV3 (2-4+ NVIDIA/CUDA); vLLM (general production); SGLang (long-context/MoE/routing); TensorRT-LLM (NVIDIA max performance); NVIDIA Dynamo (cluster orchestration).
- llama.cpp owns portability/GGUF/hybrid offload but is not for serious multi-node production; do not use it (or Ollama) on multi-GPU setups — use vLLM or ExLlamaV2.
- MLX unified memory fits large quantized models that will not fit in 24 GB consumer VRAM, but is slower; MLX-LM server is not recommended for production.
- ExLlamaV2 is the enthusiast local CUDA/EXL2 engine; ExLlamaV3 extends to multi-GPU (2-4+) and local MoE with EXL3 (QTIP), with some model/parallelism caveats.
- vLLM is the default open-source production starting point; SGLang is for hostile traffic (disaggregation, RadixAttention, MoE/routing); TensorRT-LLM trades portability for NVIDIA absolute performance (FP8/FP4 on H100+/B200-class).
- Do not use Ollama; treat local engines as production only when security, observability, backpressure, routing, autoscaling, and SLA behavior are real.
- Quant formats (GGUF, EXL2, EXL3, AWQ, GPTQ, FP8, FP4, MLX, ONNX) are not interchangeable — use the format with optimized kernels on the target engine.
- Never benchmark engines on single-user tokens/sec alone; measure TTFT, TPOT, percentiles, concurrency, prefill vs decode, memory headroom, cache reuse, structured output, and LoRA separately.

## Actionables

- Choose an engine from the one-page map after fixing hardware, workload shape (prefill vs decode, context, concurrency), and serving model — not from brand familiarity.
- For multi-GPU setups, avoid llama.cpp and Ollama; prefer vLLM or ExLlamaV2/V3 depending on production vs local quantized goals.
- On Mac, start with MLX / MLX-LM for native workflows and llama.cpp for GGUF portability; do not treat MLX-LM’s server as production.
- For open-model production serving, start with vLLM; move to SGLang when structured outputs, long context, MoE, disaggregation, or routing dominate; bake-off TensorRT-LLM on NVIDIA-only fleets when absolute performance justifies tuning.
- Benchmark with exact model/weights/engine/hardware/workload metadata and production-shaped metrics (TTFT, TPOT, p95/p99, concurrency, KV hit rate, cost per 1M tokens); re-test after driver, CUDA, ROCm, model, or engine upgrades.
- Before picking, answer the ten questions in the source: hardware on hand, fit in fast vs system/unified memory, prefill vs decode bottleneck, context/concurrency, prefix sharing, model architecture, local vs production vs fleet, quant kernels, interconnect type, and optimization target.

## References

### X source

- https://x.com/TheAhmadOsman/status/2057183854444843202
- https://x.com/TheAhmadOsman/status/2040103488714068245
- https://x.com/TheAhmadOsman/status/2041331757329285589

### GitHub

- https://github.com/av/harbor

### Other

- https://www.ahmadosman.com/blog/do-not-use-llama-cpp-or-ollama-on-multi-gpus-setups-use-vllm-or-exllamav2/
