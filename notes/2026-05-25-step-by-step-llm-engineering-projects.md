# Source

- Author: [@TheAhmadOsman](https://x.com/TheAhmadOsman)
- Article/Post: https://x.com/TheAhmadOsman/status/2058745340895870985
- Date: 2026-05-25
- Title: Step-By-Step LLM Engineering Projects (2026 Edition)

## Long summary

A projects-first roadmap for turning LLM fundamentals into working systems you can build, measure, break, and explain. It sits after the author’s Local AI foundation series (GPU memory math, memory bandwidth, inference engines, and LLMs 101) and argues that by May 2026 you learn the field by reproducing the stack in order: tokenizer → embeddings → position → attention → Transformer blocks → objectives → decoding → cache → long context → routing → data → post-training → serving → evaluation → tools → alignment / safety.

The shared project loop for every item: build the primitive yourself before the library version; plot behavior (loss, latency, memory, attention heatmaps, routing histograms, entropy, failure galleries); break it on purpose (ablations, bad masks, aggressive quantization, starved data, collapsed routers, overloaded KV cache); explain what changed in a short technical note; ship an artifact (repo, notebook, blog, demo, benchmark, or reproducible experiment).

Parts I–IX cover core mechanics: BPE and SentencePiece-style tokenizers; embedding tables; sinusoidal, learned, RoPE, and ALiBi positional methods; scaled dot-product and multi-head attention; a pre-norm decoder block (RMSNorm / LayerNorm, often SwiGLU) stacked into a toy “mini-former”; causal / masked / prefix / denoising objectives (BERT, GPT-style, T5, UL2); sampling dashboards plus speculative decoding (Medusa, lookahead); KV cache with MQA, GQA, and DeepSeek-V2/V3 MLA; long-context work (sliding-window attention, StreamingLLM attention sinks, YaRN, Infini-attention); and hardware-aware attention (naive vs PyTorch SDPA vs FlashAttention / FlashAttention-3) with 2026 hardware budgets (NVIDIA Blackwell FP4/FP8, DGX B200-class HBM, Google Ironwood TPU, AMD MI300X 192GB HBM3).

Parts X–XXI move into systems and research: toy MoE routers and sparse trade-offs (Switch Transformer, Mixtral, DeepSeek-V3 671B total / 37B activated, Llama 4, Qwen3 MoE/dense lineup, Kimi K2.6 1T / 32B activated / 256K / MoonViT); state-space and diffusion alternatives (Mamba, Mamba-2, RetNet, LLaDA, Dream, Mercury); data pipelines and synthetic data (FineWeb, Dolma, DataComp-LM, phi-1); scaling laws (Kaplan-style, Chinchilla); post-training (InstructGPT, DPO, RLHF/PPO/GRPO/RLVR, o1, DeepSeek-R1, DeepSeekMath); quantization (GPTQ, AWQ, GGUF); serving (vLLM/PagedAttention, TensorRT-LLM, SGLang/RadixAttention); evaluation (HELM, MMLU, EleutherAI harness); RAG, tools, and agents (ReAct, Toolformer, DSPy); multimodal adapters (CLIP, Flamingo, LLaVA; Llama 4 and Kimi K2.6 as native multimodal); interpretability and safety (transformer circuits, Anthropic sparse autoencoders, Constitutional AI, red-teaming). A final capstone trains, tunes, quantizes, serves, evaluates, and documents one small model system.

A 12-week lock-in plan maps the same progression (weeks 1–2 representations/attention through weeks 11–12 capstone with SFT, LoRA/QLoRA, DPO, toy RL, RAG/tools, red-team, publish). After every project, ship five artifacts: implementation with tests, a reproducible notebook, at least three plots, a failure gallery, and a short write-up. Closing mindset: frameworks and demos hide model, cache, leakage, and failure modes—fundamentals remove the hiding places; build agents and products only on that bedrock.

## Key claims

- Tokenization, attention, KV cache, MoE, quantization, serving, RAG, agents, evaluation, and safety are parts of one system, not separate buzzwords.
- Learn by the loop: implement the primitive, plot behavior, break it on purpose, explain what changed, then ship an artifact.
- Build order matters: understand tokenization and attention/memory before training choices, serving stacks, or trusting empty product demos.
- Core mechanics span BPE/SentencePiece, embeddings, sinusoidal/learned/RoPE/ALiBi position, attention and multi-head attention, pre-norm Transformer blocks (RMSNorm/LayerNorm, often SwiGLU), mini-former training, and causal/masked/prefix/denoising objectives (BERT, GPT-style, T5, UL2).
- Decoding bridges logits to text; speculative decoding (plus Medusa and lookahead variants) accelerates autoregressive generation while preserving the target distribution when done correctly.
- KV cache is a central training-vs-inference difference; MQA, GQA, and DeepSeek MLA (DeepSeek-V2/V3) are major 2026 KV-memory design axes.
- Long context is a systems problem: Mistral 7B-style sliding-window + GQA, StreamingLLM attention sinks, YaRN, and Infini-attention address cost, stability, and extension—not a config-number bump.
- FlashAttention / FlashAttention-3 show that attention runtime depends on memory access and hardware-aware scheduling; 2026 engineering is hardware-constrained (Blackwell FP4/FP8, DGX B200-class, Ironwood TPU, MI300X 192GB HBM3).
- Sparse MoE examples cited: DeepSeek-V3 671B total / 37B activated; Llama 4 open-weight multimodal MoE; Qwen3-235B-A22B and Qwen3-30B-A3B plus six dense models; Kimi K2.6 1T total / 32B activated / 256K context / MoonViT.
- Beyond Transformers: Mamba / Mamba-2 / RetNet and diffusion-style LMs (LLaDA, Dream, Mercury) are serious alternatives to study with toys.
- Data quality and synthetic data (FineWeb, Dolma, DataComp-LM, phi-1) plus scaling laws (Kaplan-style, Chinchilla) are first-class engineering substrates.
- Post-training path: InstructGPT-style feedback, DPO, and RL methods (PPO, GRPO, RLVR) matter for assistants and reasoning (o1, DeepSeek-R1, DeepSeekMath).
- Serving and eval stacks named: GPTQ/AWQ/GGUF; vLLM PagedAttention, TensorRT-LLM, SGLang RadixAttention; HELM, MMLU, EleutherAI harness.
- Capstone stack includes RAG, tool/agent loops (ReAct, Toolformer, DSPy), multimodal adapters (CLIP, Flamingo, LLaVA), interpretability (circuits, sparse autoencoders), and safety/red-team evaluation (including Constitutional AI).
- A realistic ~12-week plan plus five publishable artifacts per project (code+tests, notebook, ≥3 plots, failure gallery, write-up) compounds fundamentals better than theory-only or demo-only learning.

## Actionables

- Follow the Local AI foundation series first, then work this roadmap in order from tokenizer through serving, evaluation, tools, and safety—do not jump to agents/products before the bedrock pieces.
- For each project: implement yourself, plot behavior, ablate/break on purpose, write a short technical note, and ship a repo/notebook/demo/benchmark.
- Use the 12-week lock-in plan as a default schedule (representations → training/objectives → inference → long context/MoE/data → post-training/eval → full capstone), adjusting pace but keeping the build–plot–break–explain loop.
- After every project, publish five artifacts: tested implementation, reproducible notebook, at least three charts, a failure gallery, and a short write-up of what changed your mind.
- Treat frameworks, benchmarks, and demos as hiding places until you can explain the underlying model, memory, data, inference, and evaluation rules.

## References

### X source

- https://x.com/TheAhmadOsman/status/2040103488714068245
- https://x.com/TheAhmadOsman/status/2041331757329285589
- https://x.com/TheAhmadOsman/status/2057183854444843202
- https://x.com/TheAhmadOsman/status/2057590224729911346
- https://x.com/TheAhmadOsman/status/2058617968628551718
