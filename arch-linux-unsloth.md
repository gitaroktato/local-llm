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
Should be possible with `gpu-burn-git`: https://aur.archlinux.org/packages/gpu-burn-git

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
- [Personal AI Computer Blog](https://www.autonomous.ai/ourblog/how-to-build-your-own-personal-ai-computer)

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
