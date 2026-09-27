# Source

- Author: [@TheAhmadOsman](https://x.com/TheAhmadOsman)
- Article/Post: https://x.com/TheAhmadOsman/status/2041331757329285589
- Date: 2026-04-07
- Title: Memory Bandwidth for Local AI Hardware (2026 Edition)

## Long summary

Argues that for local AI, “bigger memory pool = better” collapses once you care about speed: capacity decides whether a model fits; bandwidth decides whether the box feels alive or crawls (e.g. ~3 tokens/sec). A 32GB RTX 5090 and RTX PRO 6000 can outrun a much larger unified-memory machine when the model fits, while a Mac Studio M3 Ultra, DGX Spark, or Strix Halo box can still be right when the model will not fit on a normal GPU (but will be slower, and not help for multi-agentic workflows).

Mental model given: `Local AI hardware = capacity × bandwidth × software stack` — capacity = what fits, bandwidth = how hard the box can breathe, software stack = how much of the spec sheet you can cash out. Rejects framing around “AI PC,” “NPU TOPS,” or marketing decks. Memory bandwidth is not tokens per second, but the cleanest first-pass way to tier hardware before arguing over single-prompt demos.

Current landscape (as listed):

- **1.8 TB/s class:** RTX PRO 6000 Blackwell, RTX 5090 → **1792 GB/s**
- **800 GB/s class:** Mac Studio M3 Ultra → **819 GB/s**
- **450–650 GB/s class:** Mac Studio M4 Max → **546 GB/s**; MacBook Pro M5 Max → **460–614 GB/s**; AMD Radeon AI PRO R9700 → **640 GB/s**; Tenstorrent Blackhole p150 → **512 GB/s**
- **250–300 GB/s unified-memory class:** DGX Spark → **273 GB/s**; Mac mini M4 Pro → **273 GB/s**; Ryzen AI Max / Strix Halo → **256 GB/s**
- **Thin-and-light AI PC class:** MacBook Air M5 → **153 GB/s**; Snapdragon X Elite → **135 GB/s**; Intel Lunar Lake → **136 GB/s**; Snapdragon X2 Elite → **152–228 GB/s**

Side note: capacity and bandwidth are often collapsed into one blob. A 32GB RTX 5090 and a 96GB RTX PRO 6000 Blackwell share the same bandwidth but live in different worlds once model size matters. Contrast: DGX Spark = 128GB unified memory at 273 GB/s; a Ryzen AI Max system can expose ~96GB as GPU memory; Mac Studio M3 Ultra goes up to 512GB at 819 GB/s. Bandwidth is not the whole story, but the fastest way to stop being confused.

Practical tiers: below ~150 GB/s = thin-and-light (not competing with workstation GPUs); around **250–300 GB/s** → unified memory starts getting interesting; around **450–650 GB/s** → serious workstation tier; at **800+ GB/s** → expensive, powerful, and fun. Local AI in 2026 is five markets pretending to be one.

Discrete GPUs remain bandwidth kings when the model fits, or when pooling via NVLink (now mostly server-side) or Gen 5 PCIe with Tensor Parallelism — especially NVIDIA given software support:

- RTX PRO 6000 Blackwell → 96GB @ 1792 GB/s
- RTX 5090 → 32GB @ 1792 GB/s
- RTX 4090 → 24GB @ 1008 GB/s
- RX 7900 XTX → 24GB @ 960 GB/s
- Radeon PRO W7900 → 48GB @ 864 GB/s
- AI PRO R9700 → 32GB @ 640 GB/s
- Arc Pro B65 → 32GB @ ~608 GB/s
- Arc Pro B60 → 24GB @ ~456 GB/s

Apple story: not the fastest, but usable — Mac mini M4 → 120 GB/s; MacBook Air M5 → 153 GB/s; Mac mini M4 Pro → 273 GB/s; MacBook Pro M5 Pro → 307 GB/s; M5 Max → up to 614 GB/s; Mac Studio M3 Ultra → **819 GB/s + up to 512GB memory**. Apple wins for one quiet box with huge memory without sharding across GPUs; loses when raw tokens/sec and concurrency matter most.

DGX Spark: 128GB unified memory, 273 GB/s, NVIDIA stack — bandwidth not impressive; coherent memory + software stack is; a developer appliance with NVFP4 support that is yet to mature. Strix Halo / Ryzen AI Max: 256-bit LPDDR5X, up to 128GB memory, ~256 GB/s bandwidth, up to ~96GB usable as GPU memory; Framework Desktop called out as interesting here.

“AI PC” trap: most are bandwidth-starved (Snapdragon X Elite 135 GB/s; Intel Lunar Lake 136 GB/s; MacBook Air M5 153 GB/s; Snapdragon X2 Elite up to ~228 GB/s) — fine for small models, assistants, edge workloads; not for 9B Dense playgrounds, serious multi-agent work, or long-context stress testing.

Tenstorrent wildcards: Wormhole n300 → 24GB @ 576 GB/s; Blackhole p150 → 32GB @ 512 GB/s + 800G interconnect; fully OSS stack, still maturing.

Why bigger boxes still feel slow: fitting ≠ serving. You still pay for bandwidth during decode, KV cache growth, dequantization, batching + concurrency, scheduler quality, and framework overhead — “it runs” = demo; “it serves” = system design. Multi-GPU is not linear: you buy interconnect (PCIe vs NVLink vs RDMA), topology, sync overhead, and software maturity.

Closing blunt map: NVIDIA → fastest raw speed; Apple Ultra → biggest one-box memory; Strix Halo → first real x86 unified-memory play; DGX Spark → coherent NVIDIA appliance; AMD / Intel Arc → rising alternatives; Tenstorrent → fully opensource stack. Stop asking “which hardware is best?” and ask “which bottleneck am I buying?”

## Key claims

- Local AI hardware should be framed as `capacity × bandwidth × software stack`, not as “AI PC” or “NPU TOPS.”
- Capacity decides what fits; bandwidth decides how hard the box can breathe; software decides how much of that you actually see.
- Memory bandwidth is not tokens per second, but the cleanest first-pass tiering metric for local AI hardware.
- 1.8 TB/s class: RTX PRO 6000 Blackwell and RTX 5090 at **1792 GB/s**; Mac Studio M3 Ultra at **819 GB/s**; mid tiers span ~450–650 GB/s and ~250–300 GB/s unified-memory; thin-and-light sits roughly ~135–228 GB/s.
- Same-bandwidth GPUs can differ wildly by capacity (32GB RTX 5090 vs 96GB RTX PRO 6000 Blackwell); unified boxes trade capacity for lower bandwidth (e.g. DGX Spark 128GB @ 273 GB/s vs M3 Ultra up to 512GB @ 819 GB/s).
- Below ~150 GB/s is thin-and-light; ~250–300 GB/s unified memory starts getting interesting; ~450–650 GB/s is serious workstation; 800+ GB/s is expensive/powerful — five markets pretending to be one.
- Discrete GPUs still dominate bandwidth when the model fits (or when pooled with NVLink / Gen 5 PCIe + Tensor Parallelism), especially NVIDIA for software support.
- Apple wins for one quiet high-capacity box (M3 Ultra: 819 GB/s + up to 512GB); loses when raw tokens/sec and concurrency matter most.
- DGX Spark’s value is coherent memory + NVIDIA stack (including immature NVFP4), not its 273 GB/s bandwidth.
- Strix Halo / Ryzen AI Max (~256 GB/s, up to ~96GB GPU-usable memory) is the first real x86 unified-memory contender; Framework Desktop is called out as interesting.
- Most “AI PCs” are bandwidth-starved and unsuitable for 9B Dense playgrounds, serious multi-agent workloads, or long-context stress testing.
- Fitting ≠ serving: decode bandwidth, KV cache, dequantization, batching/concurrency, scheduler, and framework overhead still bind; more GPUs ≠ linear scaling.

## Actionables

- When choosing local AI hardware, score candidates on capacity, memory bandwidth tier, and whether the software stack can deliver the spec — not marketing labels like “AI PC” or NPU TOPS.
- Ask “which bottleneck am I buying?” (fit vs breath vs software) instead of “which hardware is best?”
- Prefer discrete high-bandwidth GPUs (e.g. RTX 5090 / RTX PRO 6000 Blackwell at 1792 GB/s) when the model fits or can be sharded; prefer high-capacity unified boxes (e.g. Mac Studio M3 Ultra, DGX Spark, Strix Halo) when it will not fit on a normal GPU and you accept lower speed.
- Treat sub-~150 GB/s thin-and-light / AI PC machines as suitable for small models, assistants, and edge workloads — not as competitors to workstation GPUs or serious multi-agent / long-context setups.
- Do not equate “it runs” with “it serves”: plan for decode bandwidth, KV cache growth, dequantization, batching/concurrency, scheduler quality, and framework overhead; treat multi-GPU as buying interconnect, topology, sync cost, and software maturity, not linear speedups.

## References

N/A
