---
title: Baseline Capabilities and Expansion
layout: default
parent: ComfyUI
nav_order: 4
---

# Baseline Capabilities and Expansion

The baseline graph is the V6 text-to-image workflow running on the standard rig. It is designed to be extended at fixed seams rather than rebuilt per task. This page describes what the baseline provides and where capability attaches when a job needs more than text-to-image. For the full V6 node architecture, see [V6 Workflow]({% link comfyui/workflow-v6.md %}).

## Standard rig

The baseline assumes the standard ShruggieTech stack. Components are commercial-cleared unless noted under [Licensing constraints](#licensing-constraints).

| Component | Value |
|---|---|
| GPU | RTX 2080 Ti (11 GB VRAM) |
| Base model | Juggernaut XL Ragnarok (SDXL) |
| Base generation resolution | 1344 x 768 |
| Upscaler | 4x-UltraSharp (model upscale) |
| Final delivery resolution | 1920 x 1080 |
| Prompt generation | Ollama (`llama3.1:8b`), scene and atmosphere only |

## Baseline capabilities

The text-to-image baseline provides the following, with no additional nodes wired:

- **Subject-anchored prompting.** The subject is pinned to the operator's literal words through a fixed `CLIPTextEncode` node that bypasses the model. Ollama writes only the scene and atmosphere, and the two are merged with `ConditioningConcat` before the sampler. The subject cannot drift between runs while the scene still varies.
- **Two-pass sampling with hires fix.** A base pass at 1344 x 768 followed by a second pass for detail recovery.
- **Model-based upscale.** 4x-UltraSharp upscales the decoded image, which is then resolved to the 1920 x 1080 delivery target.
- **Photoreal SDXL output** from Juggernaut XL Ragnarok.
- **Single-input operation.** The operator supplies one variable (the subject text). Everything else is fixed.
- **Metadata-free client delivery.** Saving as JPEG strips generation metadata reliably and is the default for client-billable output.

## Expansion surface

The baseline extends at three defined seams. Each adds capability without rebuilding the graph.

1. **Latent input seam.** The sampler's `latent_image` input accepts either an empty latent (text-to-image) or an encoded reference (image-to-image). This is the img2img attachment point.
2. **Conditioning seam.** Structure or style conditioning attaches to the conditioning path ahead of the sampler. This is where ControlNet and IPAdapter attach.
3. **Upscale and output seam.** The model upscale already sits here. Additional passes or output stages attach at this point.

## img2img expansion (established)

img2img is the built reference-image capability. It grounds a generation on an existing image (for example, adding an item to a portrait or a background to a line drawing) by feeding the sampler an encoded reference instead of an empty latent.

At the latent input seam:

- `LoadImage` feeds `VAEEncode`, which encodes the reference into latent space using the checkpoint VAE.
- The encoded latent is wired into KSampler Pass 1's `latent_image` input in place of `EmptyLatentImage`.
- `EmptyLatentImage` is retained in the graph but left disconnected, so the operator can revert to text-to-image without rebuilding the chain.
- Pass 1 denoise starts at 0.55. Lower it to preserve more of the input image, raise it to reinterpret the input more freely.

Size the reference to the working resolution before encoding. An oversized reference inflates the latent and can exhaust the 11 GB ceiling at the encode or upscale step.

The img2img variant is delivered as `AI_image_generator_img2img.json`. For the complete node listing, see [V6 Workflow]({% link comfyui/workflow-v6.md %}).

## Further expansion paths

These attach at the conditioning seam. Each is documented on its own page.

- **ControlNet Canny (xinsir V2).** Conditions generation on the edges of a reference to control composition and structure. Commercial-cleared.
- **IPAdapter Plus (ViT-H, non-FaceID).** Matches the style or look of a reference image. Commercial-cleared.

## Constraints that bound expansion

### VRAM ceiling

The 2080 Ti caps working memory at 11 GB. SDXL plus a model upscale plus one conditioning branch fits when passes are staged. Running every branch at full size at once does not. When memory is tight, reduce in this order: drop concurrent conditioning branches, lower the pre-upscale working resolution, then stage or tile the upscale.

### Single operator, minimal touchpoints

Every expansion keeps one obvious input per run. Avoid any addition that forces the operator to rewire nodes or change several values to get a normal result. If an expansion adds operator burden, document the lower-touch alternative alongside it.

### Licensing constraints

Client-billable output uses only commercial-cleared components. The baseline stack, img2img, ControlNet Canny (xinsir V2), and IPAdapter Plus (ViT-H) are cleared. Face conditioning (IPAdapter FaceID Plus v2, InstantID) depends on non-commercial insightface models and is not cleared for client work. The face and likeness track is maintained separately. Do not wire face conditioning into any client-billable graph.
