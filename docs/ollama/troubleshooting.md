---
title: Troubleshooting and VRAM
layout: default
parent: Ollama
nav_order: 4
---

# Troubleshooting and VRAM

## Generation fails at the prompt nodes

The usual cause is that Ollama is not running. ComfyUI cannot reach the service, so the `OllamaGenerateV2` node errors out. Confirm the service is live by opening `http://127.0.0.1:11434` in a browser; an "Ollama is running" response means it is up. If it is not, start Ollama and try again. Check this before debugging anything in the workflow itself.

## "model not found"

The service is running but the requested model is not pulled. Run `ollama list` to see what is installed, then pull the one the workflow expects:

```bash
ollama pull llama3.1:8b
```

The model name in the `OllamaConnectivityV2` node has to match a name from `ollama list` exactly.

## VRAM contention with image generation

This is the big one on our hardware. The 2080 Ti has 11 GB, and both the language model and SDXL want a share of it. `llama3.1:8b` holds roughly 5 GB while it is loaded, and SDXL generation plus the upscale pass wants most of the rest. If the language model is still resident when image generation runs, you can run out of VRAM, at which point Ollama quietly spills layers over to system RAM and everything slows down.

### How to see it

Run this while a generation is going:

```bash
ollama ps
```

It lists each loaded model, its size, and the processor split. A reading below 100 percent GPU means part of the model has been pushed onto the CPU, which is the slow-down symptom.

### The fix: unload the model after it writes the prompt

By default Ollama keeps a model in memory for five minutes after its last use (`OLLAMA_KEEP_ALIVE=5m`). That keeps the language model sitting in VRAM right through image generation. Set keep-alive to unload promptly instead, so the VRAM frees up for SDXL:

```bash
# unload immediately after each response
OLLAMA_KEEP_ALIVE=0
```

Set this as an environment variable and restart the Ollama service. The value semantics are easy to get backwards, so to be clear:

- `0` unloads the model immediately after it answers. Best when VRAM is tight, which is our case.
- `5m` is the default. A duration keeps the model loaded for that long after its last use.
- `-1` keeps it loaded forever. Do not use this here. It is the setting most likely to cause an out-of-memory error when image generation then tries to load.

The tradeoff with `0` is that the next generation reloads the language model, which adds a few seconds. On an 11 GB card that is almost always worth it to give image generation the headroom it needs.

### Alternative: keep the language model off the GPU

If you would rather reserve all 11 GB for image generation, you can run the prompt model on the CPU instead. Prompt writing is light, so the model being a little slower there is rarely noticeable, and the GPU stays fully available for SDXL. This is a reasonable choice if the per-run reload from `OLLAMA_KEEP_ALIVE=0` starts to annoy you.
