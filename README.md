# Local LLM 
A combination of hardware and software capable of running LLMs for coding locally. 

## Execution Environments

- [Arch Linux and Unsloth](./arch-linux-unsloth.md)
- [Windows WSL with vLLM](./windows-wsl-vllm.md)
- 
## Images
<img src="img/gpu-holder-screw.png" height=350px>
<img src="img/gpu-mount-design.png" width=350px>
<img src="img/final-machine.jpg" width=350px>
<img src="img/tpc-arch.png" width=350px>

## Tools

Setting CPU governance

```bash
sudo cpupower frequency-set -g performance
```

## References

- <https://zread.ai/abetlen/llama-cpp-python/20-kv-cache-strategies>
- <https://github.com/ggml-org/llama.cpp/discussions/14758>
