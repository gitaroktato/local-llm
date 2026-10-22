The GitHub repo "club-dual-5060ti" looks directly relevant. Let me dig into it and related sources.



The GitHub repo has direct A/B test data. Let me gather more corroborating evidence — the vLLM sm_120 issue, Reddit threads, and other benchmarks.



One more attempt at the Reddit thread with the ~60 tok/s vLLM result, then I'll compile.



# Evidence: vLLM vs llama.cpp on dual RTX 5060 Ti (16GB × 2)

## Hardware context (verified)

| Spec | Value | Source |
|---|---|---|
| Architecture | Blackwell, compute capability **sm_120** | TechPowerUp, NVIDIA forums |
| VRAM | 16 GB GDDR7 per card → 32 GB total (two *independent* pools, not unified) | TechPowerUp; club-dual-5060ti |
| Memory bandwidth | **448 GB/s** per card | TechPowerUp (28 Gbps × 128-bit); hardware-corner.net |
| TDP / interface | 180 W, PCIe 5.0 x8 | hardware-corner.net |
| P2P | **None** — consumer cards have no NVLink; inter-GPU traffic goes over PCIe | club-dual-5060ti (measured: PCIe utilization <4%, bottleneck is VRAM bandwidth + cross-card latency) |

Key structural fact from a measured dual-rig teardown (Kurokosama/club-dual-5060ti): capacity is capped by the **single** 16 GB card, and GPU0 carries extra load (vocab embeddings, vision encoder), so it's usually the first to OOM.

## Direct A/B evidence on exactly this setup

### 1. Kurokosama/club-dual-5060ti — llama.cpp wins ~2× (Qwen3.8-27B)
Personal dual 2× 5060 Ti 16GB rig, same cards, A/B tested (Aug 2026):

| Stack | tok/s (decode) |
|---|---:|
| llama.cpp + MTP-3 (Q4_K_XL GGUF) | **61.3–67.8** ✅ |
| vLLM NVFP4, no MTP (Docker, TP2, FP8 KV) | 31.7 |
| vLLM NVFP4 + MTP-2 | **15.7** (slower than without MTP) |
| SGLang PP=2 | 6.4 |

Their stated conclusion: *"On dual cards llama.cpp is clearly about 2× faster than vLLM; enabling MTP on vLLM makes it slower — with no P2P, the draft model must sync across PCIe every round."* Same repo: llama.cpp Ornith-1.5-35B-A3B (MoE) Q4_K_M hits **~109 tok/s** at 256K ctx; vLLM is kept only as the OpenAI-API/multimodal fallback ("works, but not the fastest").

### 2. cute_agapan9087 (note.com, May 2026) — vLLM wins 1.7× (Qwen3.6-27B)
Same dual 5060 Ti rig, only the runtime changed:

| Stack | tok/s |
|---|---:|
| LM Studio / llama.cpp, Q4_K_M GGUF | 18 |
| **vLLM + NVFP4 (modelopt) + MTP n=3**, TP2, FP8 KV, 128k ctx | **30.8** (±0.4 over 5 runs) |

The win is attributed to three things: **NVFP4 uses Blackwell's native 4-bit tensor-core path that GGUF Q4_K_M cannot touch**, MTP speculative decoding, and PagedAttention/continuous batching. Critical gotchas documented: `--quantization modelopt` is mandatory (compressed-tensors silently routes NVFP4 through a slow path — a reported 2× difference on Reddit), `--kv-cache-dtype fp8` needed for 128k to fit, and TP=2 "is not a speed-up setting but a load-distribution setting; PCIe sync overhead remains."

### 3. 5p00kyy/club-5060ti — community repo with reproducible receipts (dual 5060 Ti, Linux, driver 595.58)
- **vLLM lane** (Qwen3.8-27B NVFP4 / FP8 KV / MTP3, TP2, ctx 122,880, max-num-seqs 1): decode **67.29 tok/s median** (66.99–67.41 across 3 runs of 118,660-prompt/512-output), prefill ~952.5 tok/s.
- **llama.cpp lane** (same hardware): Qwen3.6-27B UD-Q4_K_XL draft-MTP n=3 → ~29–38 tok/s; Qwen3.6-35B-A3B IQ4_XS → ~89–90 tok/s.
- **Matched-stack quality test** (Qwen3.8-27B, 68 graded cases each): llama.cpp Q6_K/f16-KV/draft-MTP2 scored **66/68** answer-correct and 66/68 combined-exact-contract vs vLLM NVFP4/FP8-KV/MTP3 at **65/68** and 61/68 — llama.cpp slightly ahead on strict output contracts (vLLM's misses were mostly JSON-fence formatting), both 10/10 on long-context marker retrieval. They explicitly frame this as a complete-stack comparison, not quantization-only proof.

### 4. Corroborating context
- r/LocalLLaMA thread (via foorilla mirror, Apr 2026): "Qwen3.6 27B on dual RTX 5060 Ti 16GB with vLLM: **~60 tok/s, 204k context working**" — consistent with the ~60–67 tok/s band above for a tuned vLLM NVFP4+MTP setup.
- hardware-corner.net (dual 5060 Ti vs RTX 3090, llama.cpp): Qwen3-30B-A3B MoE at **101 tok/s** @ 4K ctx, 13.4 @ 131K ctx; Qwen3-32B dense 18.2 → 9.6 tok/s. Confirms the rig's bandwidth ceiling (~448 GB/s/card) and that MoE models are where dual 5060 Ti shines.

## vLLM-specific evidence: sm_120 support friction

| Evidence | Detail |
|---|---|
| [vllm#35432](https://github.com/vllm-project/vllm/issues/35432) (Feb 26, 2026) | Prebuilt PyPI wheels/Docker images fail on RTX 50-series: `no kernel image is available for execution on the device` / `sm_120 not compatible`. Closed as completed May 15, 2026 via PR #35568. |
| [vllm#41614](https://github.com/vllm-project/vllm/issues/41614) + norens/vllm-blackwell-build (Jun 2026) | Even after the fix, **public wheels still failed at runtime on sm_120** for that user; required source build with `TORCH_CUDA_ARCH_LIST=12.0`, gcc-14 pin, FlashInfer cu128 wheel, torch nightly — 8 documented failure modes. |
| norens/vllm-blackwell-build | CUDA-graph capture (`enforce_eager=False`) **deadlocks** under vLLM's forked-subprocess EngineCore on Blackwell; `enforce_eager=True` works but is "~3× slower per call." Also: `gpu_memory_utilization < 0.90` OOMs the KV cache (not weights) on 16 GB cards. |
| llama.cpp #26750 (cited by Kurokosama) | Counter-evidence against speculation in general: CUDA-backend MTP draft acceptance collapsed to 40.7% vs Vulkan's 92%; A/B on dual 5060 Ti showed MTP *slowed* llama.cpp from ~109 to ~43–73 tok/s until fixed upstream. |

## Synthesis — what the evidence actually says

1. **It's not an engine question, it's a quantization-path question.** GGUF Q4_K_M (llama.cpp) cannot use Blackwell's NVFP4 tensor cores; vLLM with `--quantization modelopt` can. That single factor flips the winner: 30.8 vs 18 tok/s in one rig's A/B (vLLM ahead), while like-for-like Qwen3.8-27B tests put llama.cpp ~2× ahead (61–68 vs 31.7).
2. **Tuned, both engines converge at ~60–67 tok/s** for a 27B dense model on dual 5060 Ti (vLLM NVFP4+MTP3: 67.3 median; llama.cpp MTP-3: 61–68). The hardware ceiling is the 448 GB/s per-card bandwidth + PCIe cross-card sync, not the engine.
3. **Speculative decoding (MTP) over TP2 without P2P is unstable on this platform** — negative in Kurokosama's vLLM test (15.7 tok/s), positive in 5p00kyy's Aug-2026 vLLM test (67.3 with MTP3) and llama.cpp tests. Treat any MTP claim as date/build-specific; upstream fixes (llama.cpp #26750, vLLM kernel updates) changed results between May and August 2026.
4. **vLLM carries real sm_120 setup tax** (wheel failures, source-build requirements, enforce_eager deadlocks), while llama.cpp needs a self-compiled CUDA build for `sm_120` but then "just works" — and its Vulkan path doesn't use the second card at all.
5. **llama.cpp edges vLLM on output-contract quality** in the one controlled 68-case head-to-head (66/68 vs 61/68 combined), with identical long-context retrieval.

## Caveats
- All dual-5060-Ti data is single-request, consumer-rig evidence (2–4 independent rigs); no public throughput/batch-serving benchmark exists for this exact hardware — vLLM's core strength (continuous batching, high concurrency) can't be fairly evaluated on a 1-seq local rig.
- Quantizations differ across most comparisons (Q4_K_XL vs NVFP4 vs Q6_K), so cross-repo numbers are directional, not apples-to-apples; only Kurokosama's and 5p00kyy's matched-stack tests isolate the engine properly.

**Sources:** [Kurokosama/club-dual-5060ti](https://github.com/Kurokosama/club-dual-5060ti) · [5p00kyy/club-5060ti + benchmarks.md + stack-quality comparison](https://github.com/5p00kyy/club-5060ti/blob/main/docs/benchmarks.md) · [note.com: 18→30 tok/s vLLM NVFP4+MTP on 2×5060 Ti](https://note.com/cute_agapan9087/n/nb4b3456ca8b4?hl=en) · [vllm#35432](https://github.com/vllm-project/vllm/issues/35432) · [norens/vllm-blackwell-build](https://github.com/norens/vllm-blackwell-build) · [hardware-corner.net dual 5060 Ti vs 3090](https://www.hardware-corner.net/guides/dual-rtx-5060-ti-16gb-vs-rtx-3090-llm/) · [TechPowerUp 5060 Ti specs](https://www.techpowerup.com/gpu-specs/geforce-rtx-5060-ti-8-gb.c4246)