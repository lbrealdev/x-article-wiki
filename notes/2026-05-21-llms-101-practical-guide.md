# Source

- Author: [@TheAhmadOsman](https://x.com/TheAhmadOsman)
- Article/Post: https://x.com/TheAhmadOsman/status/2057590224729911346
- Date: 2026-05-21
- Title: LLMs 101: A Practical Guide (2026 Edition)

## Long summary

Model-first practical guide (as of May 21, 2026) to how LLMs work and how to run them locally. Core loop: text → tokens → Transformer → attention → KV cache → next-token selection, repeated one token at a time. Weights encode learned patterns; context is what the model sees now; the KV cache is working memory that keeps generation usable. Hardware, runtimes, quantization, context length, chat templates, decoding, RAG, serving, and model selection all fall out of those mechanics. Points to a companion three-part self-hosted / local AI series: GPU Memory Math, Memory Bandwidth, and Inference Engines.

Inference is a decoder-only loop: tokenize, forward pass, score every next-token candidate (logits → softmax probabilities), decode one token, append, repeat until stop/limit. Local speed is tokens per second; long prefill delays time-to-first-token; slow decode makes streaming feel sluggish. Tokens are integer IDs (words, fragments, punctuation, whitespace, byte fallbacks, special markers). Tokenizer family and vocabulary size change how much text fits, KV-cache size, latency, and multilingual/code efficiency. Context windows for local-capable models commonly span 8K–32K up through 128K, 256K, and even 1M on server-class systems—but supported length ≠ cheap, fast, or equally accurate context.

Most modern local chat LLMs are decoder-only Transformers: embeddings, RoPE (or similar) position, self-attention, MLP/feed-forward, layer norm/residuals, then output projection to vocabulary logits. Attention type matters for memory: classic MHA stores per-head KV and gets expensive at long context; MQA shares one K/V head across queries; GQA (common middle ground) shares K/V across query groups. FlashAttention / SDPA-style kernels cut attention memory traffic. Rule of thumb for older Llama-like 7B MHA: ~0.5 MiB/token FP16 KV cache (~2 GiB at 4K; ~16 GiB at 32K). Newer GQA/MQA and FP8/INT8 KV cache shrink this; treat sub-8-bit KV (KIVI, KVQuant, etc.) as research/workload-sensitive, not a casual desktop default. Speculative decoding (DFlash, DDTree/DTree, and related methods) attacks decode latency without erasing the KV memory bill.

Prefill processes the prompt (parallelizable, drives first-token wait); decode generates sequentially (drives streaming feel). Long prompts punish prefill; long answers punish decode; long chats grow the KV cache every turn. Decoding knobs (temperature, top-p/top-k, repetition penalties, stop sequences, constrained decoding, seeds) change voice and determinism without changing weights—start narrow for precise/code work; open up for creative work; use deterministic settings for evals. A runnable package is more than weights: architecture/config, weights (safetensors, GGUF, GPTQ, AWQ, EXL2, etc.), tokenizer, chat template, generation defaults, and license/model card. Wrong chat templates cause gibberish, role confusion, ignored system prompts, broken tools, and bad evals—treat templates like an API contract (`apply_chat_template`, model-specific templates in Harbor / llama.cpp / LM Studio / vLLM / SGLang).

Model types: base (completion, not chat), instruct, chat, reasoning, tool-tuned—default to a recent instruct/chat model that fits. “Local” means you control weights and runtime; it does not automatically mean offline, private, safe, cheap, or opensource. Success equation: model fit + correct prompt format + good runtime + realistic evals. Quantization ladder for 2026 local users: FP16/BF16 baseline; Q8/INT8 near-lossless but large; Q6/Q5 strong middle; Q4 consumer sweet spot; Q3/Q2 only when forced (math, code, structured output, tools degrade first). Weight quant ≠ KV-cache quant. Prefer safetensors; avoid untrusted pickle `.bin`; use GGUF for llama.cpp / Apple Silicon / LM Studio; ONNX for non-PyTorch accelerators; TensorRT-LLM for NVIDIA production engines; EXL2/GPTQ/AWQ in GPU-focused communities.

Runtimes: solo users start with Harbor, LM Studio, or llama.cpp; teams/private APIs with vLLM or SGLang; max NVIDIA production with TensorRT-LLM; browser/mobile with MLC or WebLLM. Serving modes progress from single-user local → team OpenAI-compatible API → production (continuous batching, prefix caching, speculative decoding, paged attention, parallelism, structured outputs, SLAs). Memory bill = quantized weights + KV cache + runtime overhead + concurrency + optional vision/speculative/adapter memory; MoE total params drive capacity, active params drive compute. Leave ~10–20% VRAM headroom. Hardware rule of thumb: 16 GB minimum comfortable GPU tier, 24 GB best-value enthusiast, 48 GB+ for stronger local work; decode is often memory-bandwidth-bound; CPU offload is for experiments, not performance.

Model choice: smallest model that wins your real workload. Memory gate: weights + KV + overhead ≤ ~80–90% of available memory. Practical starters: 7B–14B instruct at Q4/Q5 with 8K–32K for assistants; code-capable 14B–32B with tools/tests for coding; instruct + embeddings + reranker + RAG for documents; reasoning-tuned models with extra token budget; 1B–4B constrained setups for low resource. Long context is expensive attention (use with RAG, not instead of it). Multimodal inputs become tokens too—count image tokens in the same budget. 2026 scene framed by families/ecosystems (Qwen 3.5 / 3.6 including 27B dense up to 262,144 tokens with YaRN extension; Gemma 4; Kimi / Moonshot AI; GLM / Z.ai; DeepSeek; MiniMax; Mistral; Nemotron 3), not one best model. Inference research (PagedAttention, FP8 KV in vLLM, DFlash/DDTree, NVFP4) matters only when your runtime supports it cleanly.

Also covers failure-mode triage (OOM, templates, prefill/decode, RAG, JSON/tools, loops), stack growth path (Harbor/LM Studio → local server + RAG → vLLM/SGLang → TensorRT-LLM/custom), privacy/security baseline (safetensors/GGUF, sandboxes, secrets out of prompts/logs, versioning), evals on your Q4 stack not BF16 leaderboards, coding/agent guardrails, RAG pipeline stages, document workflows, edge (0.5B–4B), a local runbook, LoRA/QLoRA after simpler fixes fail, and the open-weight vs opensource distinction (read the license). Closing: local LLMs are mostly memory math + formatting + evaluation.

## Key claims

- An LLM generates one token at a time via tokenize → forward → logits/probabilities → decode → append → repeat; local speed is tokens per second.
- Tokens, not words, set context fit, KV-cache size, prefill cost, and multilingual/code efficiency; supported context length is not the same as fast or accurate context.
- Decoder-only Transformers stack embeddings, RoPE, attention, MLP, norm/residuals, and output projection; attention type (MHA / GQA / MQA) and kernels (FlashAttention / SDPA) strongly affect long-context memory and speed.
- KV cache memory scales with tokens × layers × kv_heads × head_dim × precision × 2; older Llama-like 7B MHA ≈ 0.5 MiB/token FP16 (~2 GiB at 4K, ~16 GiB at 32K); treat FP8/INT8 KV as the practical local floor and sub-8-bit KV as research-grade.
- Prefill drives time-to-first-token; decode drives streaming feel; long chats punish both as the KV cache grows every turn.
- Decoding controls change behavior without changing weights; greedy is often brittle; use narrow/deterministic settings for precise work and evals.
- A model package is weights + config + tokenizer + chat template + generation defaults + license; wrong templates invalidate evals—treat them like an API contract.
- Default to recent instruct/chat models; base models complete prompts rather than answer them.
- Local means you control weights/runtime, not automatic privacy, safety, offline use, or opensource status; success = model fit + correct format + good runtime + realistic evals.
- Quantization ladder: FP16/BF16 → Q8/INT8 → Q6/Q5 → Q4 (consumer sweet spot) → Q3/Q2 (last resort); a smaller higher-precision model can beat a larger over-quantized one.
- Prefer safetensors or reputable GGUF; avoid untrusted pickle `.bin` / casual `trust_remote_code`.
- Solo start: Harbor, LM Studio, or llama.cpp; teams: vLLM or SGLang; max NVIDIA production: TensorRT-LLM; pick runtime before format ecosystem.
- Total memory = quantized weights + KV for context + runtime overhead + concurrency + safety margin; leave ~10–20% headroom; MoE total params drive memory, active params drive compute.
- 2026 hardware tiers (quantized, sane context): 16 GB comfortable minimum, 24 GB best-value enthusiast, 48 GB+ stronger local; decode is often memory-bandwidth-bound; almost-fit CPU spill kills speed.
- Choose the smallest model that wins your real 20–50-prompt eval under the memory gate (weights + KV + overhead ≤ ~80–90% of memory); leaderboards are discovery, not substitution.
- Long context complements RAG; multimodal inputs consume the same token budget; as of May 21, 2026, think in families (Qwen 3.5/3.6, Gemma 4, Kimi, GLM, DeepSeek, MiniMax, Mistral, Nemotron 3), with Qwen 3.5/3.6 27B dense called out as a strong practical public-weight option (context up to 262,144 tokens; YaRN for longer in supported frameworks).
- Fine-tune with LoRA/QLoRA only after template, prompting, model choice, decoding, RAG/reranking, and few-shot fail; open-weight ≠ opensource—read the license.

## Actionables

- Internalize the inference loop (tokens, attention, KV cache, prefill vs decode) before buying GPUs or judging model quality.
- Start with a recent instruct/chat model that fits comfortably (often 7B–14B at Q4/Q5, 8K–32K), using Harbor, LM Studio, or llama.cpp and the correct chat template.
- Budget full memory: quantized weights + KV cache + runtime overhead + concurrency + ~10–20% headroom; prefer GQA/MQA and FP8/INT8 KV when needed; avoid casual sub-8-bit KV without hard benchmarks.
- Compare candidates on 20–50 real prompts (coding, docs, JSON/tools, long context) measuring quality, latency, memory, template reliability, and failures—not leaderboard rank alone.
- For documents/agents: build RAG with parse/chunk/embed/retrieve/rerank/cite before stuffing giant prompts; sandbox tools, least-privilege access, and keep policy checks outside the model.
- Prefer safetensors/GGUF from reputable sources; disable untrusted `trust_remote_code`; version model, quant, runtime, template, adapters, and evals; read licenses before commercial use.
- Grow the stack deliberately: desktop chat → local OpenAI-compatible server + simple RAG/evals → vLLM/SGLang private serving → TensorRT-LLM/custom only when efficiency at scale justifies the engineering.

## References

### X source

- https://x.com/TheAhmadOsman/status/2057590224729911346
- https://x.com/TheAhmadOsman/status/2040103488714068245
- https://x.com/TheAhmadOsman/status/2041331757329285589
- https://x.com/TheAhmadOsman/status/2057183854444843202

### Docs

- https://docs.vllm.ai/en/latest/features/quantization/quantized_kvcache/

### GitHub

- https://github.com/av/harbor

### HF

- https://huggingface.co/Qwen/Qwen3.5-27B
- https://huggingface.co/Qwen/Qwen3.6-27B

### Other

- https://ahmadosman.com/tokenizer
- https://arxiv.org/abs/2309.06180
- https://arxiv.org/abs/2401.18079
- https://arxiv.org/abs/2402.02750
- https://arxiv.org/abs/2602.06036
- https://arxiv.org/abs/2604.12989
