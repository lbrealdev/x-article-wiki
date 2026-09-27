# Source

- Author: @exolabs
- Article/Post: https://x.com/exolabs/status/2103617535765573959
- Date: 2026-09-25
- Title: DGX Spark Handbook

## Long summary

Handbook on running local LLM inference on NVIDIA DGX Spark (“golden brick”) boxes—quiet, low-power, 128 GB memory machines aimed at home/office use. Article body credits writing to @0xSero (reviewers @alexocheema and @alexzfunk; special thanks @MiaAI_lab). Related article URL: https://x.com/i/article/2103600009191079936.

Early skepticism focused on Spark’s ~273 GB/s memory bandwidth (far below an RTX 5090’s ~1,792 GB/s). The piece argues that smaller capable models, MoE, and speculative decoding now let Sparks feel cloud-parity for single-user tokens/s: cloud GPUs are faster but shared and tuned for cost/token; at home the box is all yours.

Sparks draw little power (~95 W with a model loaded; ~90–200 W when serving) and can be stacked (2–4 on a normal US circuit). Each has ConnectX-7 (dual QSFP, 200 Gb/s / 25 GB/s). With tensor parallelism, effective bandwidth scales near-linearly in NVIDIA’s tests (~546 / 819 / 1,092 GB/s for 2 / 3 / 4 Sparks); memory stacks to ~512 GB for four (~120 GB usable per Spark for AI). Write latency improved ~2× on two Sparks and ~3.7× on four; prompt read scales less well.

MoE models (e.g. Qwen3.6-35B activating ~3B params/token) suit Spark’s bandwidth better than dense peers. Speculative decoding (MTP, DSpark, DFlash) boosts throughput for ~1–2 GB extra memory. One Spark can serve eight+ concurrent sessions (example: Qwen3.6-35B ~40 tok/s each vs ~37 tok/s cited for ChatGPT Pro GPT-6-Astra). Spark’s high compute-per-byte vs Mac (EXO numbers) plus 4-bit hardware helps multi-user serving.

Costs: list rose from $3,999 → $4,699; street/used higher (~$5k–$6k; author saw $7,999 on NVIDIA’s site ~21 Sep). Buying tips: any GB10 OEM box works; prefer ~4 TB SSD; buy the QSFP cable with a second Spark. Power/noise vs multi-GPU rigs: one Spark ~$12/mo 24/7 at 18¢/kWh; four Sparks + switch ~$66–100/mo vs ~$300/mo for a 1,600 W GPU tower.

Practical path: start one Spark (LM Studio → Qwen3.6-35B, then MiaAI Lab / author recipes on vLLM/SGLang); two Sparks (recommended stopping point for most—GLM-5.3-Flash, DeepSeek-V4.1-Flash, etc.); three as a switchless triangle; four via 200 GbE switch (e.g. MikroTik) or switchless ring (SparkRing, alpha). Quantization (NVFP4 for native 4-bit on-chip; EXL3 for bit-width choice) is how large models fit; tip: 4 bits for smaller models, 3 bits for larger. Quality checks: top-1 agreement and KL divergence.

Advanced: Spark is weak at decode (memory-bound) but strong at prefill (compute-bound)—suited to quantization, MoE pruning (REAP), evals, long-context reads, and fine-tuning that scales across Sparks. Software niche: sm_121 (GB10); b12x / ExLlamaV3 / lil fill Spark-specific gaps. Author would buy again; if restarting, would buy two on day one.

## Key claims

- DGX Spark is a quiet, low-power local AI box with 128 GB memory but relatively low ~273 GB/s bandwidth vs discrete GPUs like RTX 5090.
- MoE architectures and speculative decoding (MTP / DSpark / DFlash) are what pushed local Spark inference over a usable speed threshold.
- Linking Sparks over ConnectX-7 adds memory and near-linear write bandwidth (NVIDIA scaling: ~2× write on two Sparks, ~3.7× on four).
- One Spark can serve ~8 concurrent Qwen3.6-35B sessions at ~40 tok/s each—comparable to cited ChatGPT Pro GPT-6-Astra ~37 tok/s for one user.
- Street price exceeds original MSRP; author cites ~$5k cheapest / ~$6k used / $7,999 seen on NVIDIA’s site around 21 Sep 2026.
- One Spark ~90–200 W (~$12/mo always-on at 18¢/kWh); four Sparks + switch ~500 W + ~240 W switch (~$66–100/mo)—far quieter and cheaper to run than multi-GPU towers.
- Recommended growth path: one Spark → recipes for speed/agents → two Sparks + QSFP cable (stop for most) → three/four only for largest models or multi-model load.
- NVFP4 and EXL3 quantization are how large models fit; 4-bit is the sweet spot for smaller models, 3-bit for larger ones.
- Spark’s strength for research is prefill/reading (quantize, prune, eval, long context), not decode; author moved pruning/EXL3/benchmarking onto Sparks and reserved faster GPUs for serving.
- GB10 (Spark) and data-center GB300 share the Grace Blackwell design family enough that software patterns transfer; Spark GPU compute capability is sm_121.

## Links

- https://x.com/exolabs/status/2103617535765573959
- https://x.com/i/article/2103600009191079936
- https://www.nvidia.com/en-us/products/workstations/dgx-spark/
- https://docs.nvidia.com/dgx/dgx-spark/spark-clustering.html
- https://developer.nvidia.com/blog/scaling-autonomous-ai-agents-and-workloads-with-nvidia-dgx-spark/
- https://build.nvidia.com/spark
- https://build.nvidia.com/spark/speculative-decoding
- https://build.nvidia.com/spark/multi-sparks-through-switch
- https://github.com/NVIDIA/dgx-spark-playbooks/tree/main/nvidia/dgx-dashboard
- https://github.com/MiaAI-Lab
- https://github.com/MiaAI-Lab/sparkDash
- https://github.com/MiaAI-Lab/sparkring
- https://github.com/0xSero/local-ai-registry
- https://github.com/local-inference-lab/b12x
- https://github.com/CerebrasResearch/reap
- https://huggingface.co/nvidia/Qwen3.6-35B-A3B-NVFP4
- https://huggingface.co/Qwen/Qwen3.6-35B-A3B
- https://huggingface.co/deepseek-ai/DeepSeek-V4.1-Flash
- https://huggingface.co/zai-org/GLM-5.3-Flash
- https://developer.nvidia.com/blog/introducing-nvfp4-for-efficient-and-accurate-low-precision-inference/
- https://blog.exolabs.net/nvidia-dgx-spark/
- https://www.servethehome.com/nvidia-dgx-spark-review-the-gb10-machine-is-so-freaking-cool/4/
- https://videocardz.com/newz/nvidia-officially-raises-dgx-spark-founders-edition-msrp-to-4699
- https://www.exxactcorp.com/blog/deep-learning/what-you-need-to-build-a-4x-nvidia-dgx-spark-cluster-switch-cabling-power
- https://inferencex.semianalysis.com/inference/qwen-3-8-flash-next
- https://x.com/MiaAI_lab
- https://x.com/0xSero

## Actionables

- If evaluating local Spark inference: buy one GB10 box (prefer ~4 TB SSD), run Qwen3.6-35B via NVIDIA’s LM Studio playbook first, then move to a MiaAI Lab / local-ai-registry recipe (vLLM/SGLang) for speed and multi-agent serving.
- For most users wanting stronger models: plan for two Sparks plus a short QSFP cable; follow NVIDIA’s connect-two-Sparks playbook and a tested dual-Spark recipe (e.g. GLM-5.3-Flash or DeepSeek-V4.1-Flash).
- Prefer MoE + speculative-decoding builds; use NVFP4 when available, else EXL3—and check top-1 agreement / KL divergence on model cards before trusting heavy quantization.
- Stand Sparks on their sides with airflow space; set up Tailscale for remote access; join Discord/Reddit/X communities for debugging.
- Only add a third/fourth Spark (triangle or MikroTik 200 GbE switch / switchless ring) if you need largest models or multi-model concurrency; pin SparkRing versions if using switchless four-Spark rings (alpha).
- For research workflows: use Sparks for prefill-heavy jobs (quantize, REAP prune, eval, long-context checks) and keep faster discrete GPUs for interactive decode if available.
