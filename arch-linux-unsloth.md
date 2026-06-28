# Running through Unsloth + Arch Linux

## Commands

### PCIe testing

Getting PCIe lane bandwidth (possible):

```shell
sudo watch -c -n2 'lspci -s 00:01.0 -vvv | grep LnkSta'
```

Understanding actual bandwidth with `nvidia-smi`:

```shell
nvidia-smi dmon -s pceutv -d 1
```

Just RX/TX

```shell
nvidia-smi dmon -s t -d 1
```

### PCIe benchmarks

Stress-testing PCIe port
Should be possible with `gpu-burn-git`: <https://aur.archlinux.org/packages/gpu-burn-git>

### Using `nvbandwith`

See: https//github.com/NVIDIA/nvbandwidth

Make sure you have NVidia toolkit and `nvcc` installed.
Put the following lines into your `.bashrc`

```bash
export PATH=/opt/cuda/bin:$PATH
export LD_LIBRARY_PATH=/opt/cuda/lib64:$LD_LIBRARY_PATH
```

Add `boost-libs` with

```shell
sudo pacman -S boost boost-libs
```

Compile from source with

```bash
cmake .
make
```

Run benchmarks:

```bash
nvbandwidth -t host_to_device_bidirectional_memcpy_ce
nvbandwidth -t device_to_host_bidirectional_memcpy_ce
nvbandwidth -t host_to_device_bidirectional_memcpy_sm
nvbandwidth -t device_to_host_bidirectional_memcpy_sm
nvbandwidth -p device_to_host_bidirectional -i 16
nvbandwidth -p host_to_device_bidirectional -i 16
```

Reference numbers:

| PCIe generation | Signaling rate (GT/s per lane) | Encoding | Effective per-lane throughput (GB/s) | Effective x4 link, one-way (GB/s) | Effective x8 link, one-way (GB/s) |
|---|---:|---|---:|---:|---:|
| PCIe 3.0 | 8.0 | 128b/130b | 0.985 | 3.94 | 7.88 |
| PCIe 4.0 | 16.0 | 128b/130b | 1.969 | 7.88 | 15.75 |
| PCIe 5.0 | 32.0 | 128b/130b | 3.938 | 15.75 | 31.51 |

Notes:

- Values are unidirectional (one-way) effective line rates accounting for 128b/130b encoding overhead only. Additional PCIe protocol overhead further reduces payload throughput.
- PCIe links are full-duplex; the reverse direction has the same capacity.

## Target Hardware

- OS: Arch Linux
- GPU: RTX 5060 Ti 16GB
- Runtime: Unsloth + llama.cpp

## Possible Performance Bottlenecks

<!-- TODO local model bottlenecks VRAM throughput?? -->

## Optimizations per Task

### Qwen3.6-35B-A3B-GGUF

<!-- TODO local model optimizations and correct settings -->

<https://unsloth.ai/docs/models/qwen3.6#turn-on-off-thinking--preserve-thinking>

## Configuration and Optimization Tips

### 🧠 Memory & Quantization

- **Quantize model weights** — Reduce 16-bit weights to 8-bit or 4-bit using Unsloth's GGUF export
  to cut VRAM usage. The performance drop from 16→8 bit is far smaller than from 8→4 bit, so prefer
  Q8 where VRAM allows and fall back to Q4 when needed.
- **Prefer larger models with aggressive quantization over smaller precise ones** — A Q4 30B model
  generally outperforms a full-precision 7B model for coding tasks.
- **Be cautious with KV cache quantization** — `llama.cpp` supports KV cache quantization via
  `-ctk` / `-ctv` flags (e.g., `q8_0`). This is considered *more destructive* than weight
  quantization because it causes the model to lose detail in long reasoning traces. Benchmark your
  specific task before enabling it.
- **Prioritize a precise KV cache for coding/reasoning** — For agentic coding sessions where long
  context matters, keep KV cache at `f16` or `q8_0` and instead shrink the model size to fit VRAM.
  See related paper: <https://arxiv.org/pdf/2510.10964>
- **Monitor actual VRAM usage** — Use `nvidia-smi` (NVIDIA) or `rocm-smi` (AMD) to confirm the
  model fits in VRAM. Spilling into system RAM via llama.cpp's `-ngl` GPU offloading will
  drastically reduce tokens/second.

### 🏗️ Model Architecture & Selection

- **Prefer Hybrid Attention architectures** — Models like Qwen3 with Hybrid Attention have a
  significantly smaller KV cache footprint, letting you run a larger model within the same VRAM
  budget.
- **Use Instruct models for chat-based coding tools** — Use instruct-tuned variants (e.g.,
  `Qwen3-Coder-*-Instruct`) with tools like OpenCode or Continue. Use base models only for
  fill-in-the-middle (FIM) autocomplete.

### ⚡ llama.cpp Runtime Tuning

- **Maximize GPU layer offloading** — Pass `-ngl 99` (or the maximum layer count of your model) to
  offload all layers to the GPU. Partial offloading (`-ngl <n>`) is a valid fallback when VRAM is
  tight, at the cost of speed.
- **Set an appropriate context size** — Use `-c 65536` or larger for coding sessions. Prefer models
  with Hybrid Attention to keep the KV cache cost of a large context manageable.
- **Tune batch and thread counts** — Use `-b 512` or `-b 1024` to balance prompt processing
  throughput. Set `-t` to the number of physical CPU cores for CPU-assisted layers.
- **Watch time-to-first-token and tokens/second** — Both matter for interactive coding. If
  tokens/second is too low, reduce context size, increase quantization, or reduce `-ngl` to leave
  more VRAM headroom for the KV cache.

### 📐 Model Size Reference (VRAM Requirements — GGUF Q4)

| Model | Approx. VRAM |
|---|---|
| Qwen3-4B-Instruct Q4 | ~3 GB |
| Qwen2.5-14B-Instruct Q4 | ~9 GB |
| Qwen3-Coder-30B-A3B-Instruct Q4 | ~18 GB |
| Qwen3-Coder-30B-A3B-Instruct Q8 | ~33 GB |
| Qwen3-Next-80B-A3B-Instruct Q4 | ~48 GB |

Use the [VRAM calculator](https://apxml.com/tools/vram-calculator) for precise estimates including
KV cache overhead for your target context length.

### 📝 Usage & Workflow

- **Manage context deliberately** — Only include files and context directly relevant to the current
  task. Bloated context degrades performance and increases KV cache memory pressure.
- **Use OpenAI-compatible tools** — Tools that speak the OpenAI API standard (OpenCode, Continue,
  Aider, Roo Code) make it trivial to swap models without reconfiguring the tool.

## Bookmarks

### Hardware

- [PCIe Accelerator - Skymizer HTX301](https://wccftech.com/this-pcie-ai-accelerator-card-packs-384-gb-memory-run-700b-llms-240w/)
- [Blackhole PCIe](https://tenstorrent.com/hardware/cards)

### Embedded

- [RPi AI HAT +2](https://www.raspberrypi.com/documentation/accessories/ai-hat-plus.html#AIHAT-models)
- [RPi + M.2 via OCuLink](https://youtu.be/SPTYjF8qH0A)
- [RPi + M.2 via OCuLink #2](https://youtu.be/b58eok9TP9Y)
- [RPi + M.2 via OCuLink #3](https://youtu.be/jeuOMDfY-MI)
- [RPi + M.2 via OCuLink #4](https://youtu.be/AyR7iCS7gNI)

### Hardware Builds

- [Personal-AI-Computer](https://github.com/autonomous-ai/Personal-AI-Computer/tree/main)
- [Personal AI Computerhttps://github.com/autonomous-ai/Personal-AI-Computer/tree/main#bios-optimization-for-gpu-performance Blog](https://www.autonomous.ai/ourblog/how-to-build-your-own-personal-ai-computer)

### Benchmarks

- [Choosing right GPU for LLMs](https://www.llamabuilds.ai/blog/choosing-right-gpu-for-llms)
- [LLM Benchmarks](https://www.llamabuilds.ai/benchmarks)
- [swe-bench-verified](https://llm-stats.com/benchmarks/swe-bench-verified)
- [livecodebench](https://llm-stats.com/benchmarks/livecodebench)

### Tools

- [pi.dev](https://pi.dev/docs/latest)

### Useful Links

- [VRAM calculator](https://apxml.com/tools/vram-calculator)
- [Unsloth with OpenCode](https://unsloth.ai/docs/integrations/opencode)
- [Guide on local LLMs](https://www.reddit.com/r/LocalLLaMA/comments/1sknx6n/best_local_llms_apr_2026/)

### Blogposts

- [Local LLM introduction](https://www.aiforswes.com/p/you-dont-need-to-spend-100mo-on-claude)

### Benchmarking

- <https://openbenchmarking.org/>

### Papers

- [Context window VS model quantization](https://arxiv.org/pdf/2510.10964)
- [Qwen3 training and inference optimizations](https://qwen.ai/blog?id=4074cca80393150c248e508aa62983f9cb7d27cd&from=research.latest-advancements-list)
