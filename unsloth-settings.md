# Unsloth settings through UI

- KV cache: f16
- Speculative decoding: auto
- Parallel slots: 1
- Batch size: auto
- Micro-batch size: 1024
- Tensor parallelism: true
- Vision: true
- GPU memory: default
- VRAM budget: 98%
- GPUs: all on
- Chat Template: default
- Mmap / Mlock: auto
- Checkpoints: auto
- Cache RAM: auto

Additional args

```bash
--threads 10 --chat-template-file /home/gitaroktato/Projects/local-llm/chat-templates/qwen3.8-froggeric-v22.3.jinja --reasoning-format deepseek
```
