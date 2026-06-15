---
title: V6 Workflow
layout: default
parent: ComfyUI
nav_order: 2
---

# V6 Workflow

V6 is the current image-generation graph. It locks the subject to your literal words and lets the model vary only the scene and atmosphere around it. The subject can no longer drift, but every run still produces a distinct image.

## Architecture

A single human input feeds three destinations automatically:

1. The subject anchor, which takes the operator text verbatim and is never processed by the model.
2. The positive scene builder, where Ollama writes the setting, lighting, and mood.
3. The negative builder, where Ollama writes what to exclude.

The subject anchor and the Ollama-written scene are merged with a `ConditioningConcat` node before they reach the sampler. This is what fixed the subject-drift problem from earlier versions: an 8B model rewriting the entire positive prompt each run could not reliably hold a subject (a requested mouse would come back as a fox or a cat), so the subject is pinned through a fixed `CLIPTextEncode` node that bypasses the model entirely.

## Reference image (img2img)

To ground a generation on an existing image (for example, adding a beanie to a portrait, or a background to a doodle), the workflow supports an img2img path:

- A `LoadImage` node feeds a `VAEEncode` node, which uses the checkpoint's VAE to encode the reference into latent space.
- That latent is wired into KSampler Pass 1's `latent_image` input, replacing `EmptyLatentImage`.
- Denoise on Pass 1 is set to `0.55` to balance structure preservation against creative change. Lower keeps more of the reference, higher lets the prompt take over.

## Troubleshooting and gotchas

### The positive prompt box looks blank during a run

This is expected ComfyUI behavior, not a fault. When a widget value is fed by a link, ComfyUI does not echo that value back into the hidden widget, so the box reads empty even though the prompt is flowing correctly.

### Nothing changes between runs

ComfyUI memoizes by input signature. If the inputs are unchanged, the node returns a cache hit and Ollama never re-executes. To force fresh output, change the input or randomize the seed.

### Subject drift, and why clearing the cache does not fix it

When the seed is randomized, each run already executes statelessly with fresh values, so clearing the cache changes nothing. Drift was never a caching problem. It was structural, caused by the model rewriting the whole prompt. The durable fix is the V6 subject anchor described above.

### Generation fails immediately at the prompt nodes

Ollama is almost certainly not running. Confirm the service is live at `http://127.0.0.1:11434` (see [Ollama Installation]({{ site.baseurl }}/ollama/installation/)).

### Removing metadata from client deliverables

Stripping PNG metadata proved unreliable through custom nodes and the `--disable-metadata` launch flag. The reliable method for metadata-free client output is to save as JPEG.
