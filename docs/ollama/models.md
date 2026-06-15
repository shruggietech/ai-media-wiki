---
title: Models
layout: default
parent: Ollama
nav_order: 2
---

# Models

The pipeline's workhorse is `llama3.1:8b`. It writes the positive and negative SDXL prompts, and it is a good balance of quality and footprint for an 11 GB card. This page covers managing models and deciding when it is worth changing the one you run.

## Pulling and listing

Models are pulled by name and tag:

```bash
ollama pull llama3.1:8b
```

See what is installed and how much disk each one uses:

```bash
ollama list
```

Remove one you no longer need. Models are large, so this matters on a smaller drive:

```bash
ollama rm <model:tag>
```

## Picking a model for prompt writing

Prompt writing is a light task. It does not need a large model, and a smaller one leaves more VRAM for image generation, which is the real bottleneck on our hardware. The tradeoff:

- Smaller (3B to 8B): faster, lighter on VRAM, and perfectly capable of rewriting a concept into an SDXL prompt. `llama3.1:8b` sits at the top of this range and is our default.
- Larger (13B and up): better reasoning and instruction-following, but they take more VRAM and time, and the quality gain on a task this simple is small. Rarely worth it here.

If you want the model to follow more involved prompt instructions, stay within the 8B class before reaching for something that competes harder with SDXL for memory.

## Where models are stored

By default Ollama stores models under your user profile (`~/.ollama/models`, or `C:\Users\<you>\.ollama\models` on Windows). If that drive is filling up, point Ollama at another location with the `OLLAMA_MODELS` environment variable and restart the service.
