# Unsloth settings through UI

- Context window: 200000
- KV cache: q8_0
- Speculative decoding: MTP (draft-mtp, n-max 3; draft KV f16 — do NOT quantize, see issue #48)
- Parallel slots: 1
- Batch size: auto
- Micro-batch size: 512
- Tensor parallelism: true
- Vision: true
- GPU memory: default
- VRAM budget: 98%
- GPUs: all on
- Chat Template: default (custom file via additional args)
- Mmap / Mlock: auto
- Checkpoints: 10
- Cache RAM: auto

Additional args

```bash
--threads 10 --chat-template-file /home/gitaroktato/Projects/local-llm/chat-templates/qwen3.8-froggeric-v22.3.jinja --reasoning-format deepseek
```
