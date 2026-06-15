---
title: Installation
layout: default
parent: ComfyUI
nav_order: 1
---

# Installing ComfyUI

This page covers everything needed before the V6 workflow JSON will load without errors.

## Hardware baseline

The setup is tuned for an RTX 2080 Ti (11 GB VRAM, Turing architecture) and runs comfortably on SDXL at that tier. On different hardware the workflow still loads, but generation times and the maximum stable resolution will shift. SDXL is the deliberate model choice over newer flagship-only models (Flux, Qwen) because it performs reliably on 8 to 12 GB cards.

## Prerequisites

Four components must be installed and present before the workflow runs:

1. ComfyUI itself
2. The `comfyui-ollama` extension (provides the V2 Ollama nodes)
3. Ollama running as a service, with the `llama3.1:8b` model pulled
4. The checkpoint and upscaler model files in the correct folders

### 1. ComfyUI

Install ComfyUI by the standard method (portable build or manual clone). Confirm it launches to a blank canvas in the browser before continuing. Nothing below functions until the base application runs.

```bash
git clone https://github.com/comfyanonymous/ComfyUI.git
cd ComfyUI
pip install -r requirements.txt
```

### 2. The Ollama extension

The workflow uses the V2 generation of the Ollama nodes, `OllamaConnectivityV2` and `OllamaGenerateV2`. These replaced the older single `OllamaGenerate` node, so any workflow built on the deprecated node will not map onto V6.

Install or update `comfyui-ollama` through ComfyUI Manager so the V2 nodes are available, then restart ComfyUI so they register.

### 3. Ollama service and model

Ollama is a separate program from ComfyUI and must be actively running in the background. If it is not, every generation fails at the prompt-building nodes. This is the single most common reason a fresh setup does not run, so verify it first.

Full steps are on the [Ollama Installation]({{ site.baseurl }}/ollama/installation/) page. The short version:

```bash
ollama pull llama3.1:8b
```

Leave Ollama running. The workflow talks to it at `http://127.0.0.1:11434`, and that address is baked into the workflow, so the service has to be live at that port whenever you generate.

### 4. Model files

Two model files must sit in the correct ComfyUI folders. The filenames in the workflow must match the filenames on disk exactly, or the loader nodes will show a red error.

| File | Location | Purpose |
|------|----------|---------|
| `juggernautXL_ragnarokBy.safetensors` | `ComfyUI/models/checkpoints/` | The SDXL base model |
| `4x-UltraSharp.pth` | `ComfyUI/models/upscale_models/` | The upscaler for the second pass |

The checkpoint is roughly 6.6 GB and is sourced from Civitai.

{: .warning }
> A mismatch between the filename in the workflow and the filename on disk is the most common cause of a red loader node. Copy the names exactly, including capitalization.