---
title: Building the V6 Workflow
layout: default
parent: ComfyUI
nav_order: 7
---

# Building the V6 Workflow

This page builds the V6 text-to-image graph node by node. The finished result is the file linked below, so building by hand is optional: an operator who just wants to run V6 can download it and skip to Prerequisites. For why V6 is shaped this way (the subject anchor and the drift fix), see [V6 Workflow]({% link comfyui/workflow-v6.md %}).

This describes the text-to-image form of V6 as exported. The optional reference-image (img2img) path is an expansion seam and is covered under [Baseline Capabilities and Expansion]({% link comfyui/baseline-capabilities.md %}).

## Download the workflow

[Download V6_Workflow.json]({{ '/comfyui/assets/V6_Workflow.json' | relative_url }})

To use it, drag the `.json` file onto the ComfyUI canvas, or load it with Workflow > Open. Loading the file still requires the models and the custom node listed under Prerequisites, plus a running Ollama service. Once loaded, the only field to edit per run is the concept node described in Stage 1.

## Prerequisites

Install these before building or loading V6:

- Checkpoint: Juggernaut XL Ragnarok, at `models/checkpoints/juggernautXL_ragnarokBy.safetensors`.
- Upscale model: 4x-UltraSharp, at `models/upscale_models/4x-UltraSharp.pth`.
- Custom node: the comfyui-ollama extension, which supplies the V2 Ollama nodes (`OllamaConnectivityV2`, `OllamaGenerateV2`, `OllamaOptionsV2`). Install it through ComfyUI Manager.
- Ollama: the service running locally with the `llama3.1:8b` model pulled. See [Ollama Installation]({% link ollama/installation.md %}).

## How the graph flows

V6 takes one operator input (a plain-English concept) and routes it to three places at once:

- the subject anchor, a `CLIPTextEncode` that encodes the concept words verbatim and is never processed by the model;
- the positive scene builder, where Ollama writes only the setting and atmosphere;
- the negative builder, where Ollama writes only what to exclude.

The subject anchor and the Ollama scene are merged with `ConditioningConcat`, and the merged conditioning drives a two-pass sampler (a base pass, then a hires refine), followed by a 4x model upscale and a save. Pinning the subject outside the model is what stops subject drift.

## Build steps

The graph is 18 nodes. Build it in stages. After placing the nodes for a stage, make the connections listed under that stage, and set widget values exactly as given.

### Stage 1: Checkpoint and concept input

| Node | Type | Settings |
|---|---|---|
| Load Checkpoint | CheckpointLoaderSimple | ckpt_name: `juggernautXL_ragnarokBy.safetensors` |
| Concept node | Primitive (PrimitiveNode) | title it "Your Concept (plain English - this is what gets generated)" |

The concept node is the only field the operator edits per run. It holds the plain-English concept and feeds three nodes downstream. A Primitive adopts the type of whatever it connects to, so its links are made in later stages, after which it presents as a single text box.

### Stage 2: Ollama connection and options

| Node | Type | Settings |
|---|---|---|
| Ollama connectivity | OllamaConnectivityV2 | url: `http://127.0.0.1:11434`, model: `llama3.1:8b`, keep_alive: 5 minutes |
| Ollama options | OllamaOptionsV2 | title "Ollama Seed / Options (fix or randomize)"; seed control set to randomize |

The options node randomizes the Ollama seed each run. That is what makes each run produce a different scene even though the image sampler seed is fixed (see Operating notes).

### Stage 3: Prompt builders

Add two `OllamaGenerateV2` nodes:

| Node | Type | Role |
|---|---|---|
| Scene builder | OllamaGenerateV2 | writes the positive scene and atmosphere |
| Negative builder | OllamaGenerateV2 | title "Negative Prompt Builder (Ollama)"; writes the negative prompt |

Paste this into the scene builder's system prompt:

```
You generate ONLY the background and atmosphere details for a photorealistic image. The main subject is handled separately, so you must NOT name, describe, or imply any animal, person, creature, or main subject of any kind. Based on the user's plain-English concept, output a comma-separated list of scene and style tags only: environment and setting, background elements, season and weather, time of day, lighting (key light, fill, direction), color palette, atmosphere and mood, camera and lens, depth of field, and composition. Vary these creatively from one generation to the next. Output ONLY comma-separated tags. No subject nouns, no preamble, no explanation, no markdown, no quotes, no line breaks.
```

Paste this into the negative builder's system prompt:

```
You write ONLY the negative prompt for SDXL photorealistic image generation. The user gives a short concept in plain casual language. NEVER list the user's main subject, or any synonym of it, as a negative. If the subject is a mouse, never output mouse, rodent, or similar. Output a single comma-separated list of things that should NOT appear: common quality and anatomy defects (blurry, out of focus, deformed, mutated, disfigured, bad anatomy, bad proportions, extra limbs, fused fingers, missing fingers, watermark, signature, text, logo, low quality, low resolution, jpeg artifacts, noise, oversaturated, overexposed, underexposed) and non-photographic styles (cartoon, anime, illustration, painting, drawing, 3d render, cgi, plastic looking, doll-like, toy). You may add a few defects relevant to the scene, but never the subject itself. Output ONLY comma-separated tags. No preamble, no explanation, no markdown, no quotes, no line breaks.
```

The prompt input on each builder is fed by a link, so convert the prompt widget to an input on both. Wire:

- Concept node STRING -> Scene builder, prompt
- Concept node STRING -> Negative builder, prompt
- Ollama connectivity connection -> Scene builder, connectivity
- Ollama connectivity connection -> Negative builder, connectivity
- Ollama options options -> Scene builder, options
- Ollama options options -> Negative builder, options

### Stage 4: Conditioning (subject anchor and merge)

Add three `CLIPTextEncode` nodes and one `ConditioningConcat`:

| Node | Type | Title (match the file) |
|---|---|---|
| Subject anchor | CLIPTextEncode | Subject Anchor (auto from your concept - do not edit) |
| Scene variation | CLIPTextEncode | Scene Variation (from Ollama) |
| Negative | CLIPTextEncode | Negative Prompt |
| Merge | ConditioningConcat | Merge Subject + Scene |

On all three `CLIPTextEncode` nodes, convert the text widget to an input so a link can feed it (drag a link onto the text field, or right-click the node and convert the text widget to an input). Wire:

- Load Checkpoint CLIP -> Subject anchor, clip
- Load Checkpoint CLIP -> Scene variation, clip
- Load Checkpoint CLIP -> Negative, clip
- Concept node STRING -> Subject anchor, text (the concept words, encoded verbatim)
- Scene builder result -> Scene variation, text
- Negative builder result -> Negative, text
- Subject anchor CONDITIONING -> Merge, conditioning_to
- Scene variation CONDITIONING -> Merge, conditioning_from

Merge order matters. The subject anchor is `conditioning_to` (it leads), and the Ollama scene is `conditioning_from` (appended after it).

### Stage 5: Two-pass sampling

| Node | Type | Settings |
|---|---|---|
| Empty latent | EmptyLatentImage | 1024 x 1024, batch 1 |
| KSampler - Pass 1 | KSampler | seed 156680208700286 (fixed), steps 28, cfg 6, sampler dpmpp_2m, scheduler karras, denoise 1.0 |
| Upscale Latent (Hires Fix) | LatentUpscale | bislerp, 1536 x 1536, crop disabled |
| KSampler - Pass 2 (Hires Refine) | KSampler | seed 156680208700286 (fixed), steps 18, cfg 6, sampler dpmpp_2m, scheduler karras, denoise 0.45 |

Wire:

- Load Checkpoint MODEL -> Pass 1, model
- Merge CONDITIONING -> Pass 1, positive
- Negative CONDITIONING -> Pass 1, negative
- Empty latent LATENT -> Pass 1, latent_image
- Pass 1 LATENT -> Upscale Latent, samples
- Upscale Latent LATENT -> Pass 2, latent_image
- Load Checkpoint MODEL -> Pass 2, model
- Merge CONDITIONING -> Pass 2, positive
- Negative CONDITIONING -> Pass 2, negative

Pass 1 generates at 1024 from an empty latent at full denoise. The latent is upscaled to 1536, and Pass 2 refines it at denoise 0.45, which adds detail without redrawing the image. Both passes read the same merged positive and the same negative.

### Stage 6: Decode, upscale, save

| Node | Type | Settings |
|---|---|---|
| VAE Decode | VAEDecode | - |
| Load Upscale Model | UpscaleModelLoader | model_name: `4x-UltraSharp.pth` |
| Upscale Image (model) | ImageUpscaleWithModel | - |
| Save Image | SaveImage | filename_prefix: `%date:yyyy-MM-dd%/image` |

Wire:

- Pass 2 LATENT -> VAE Decode, samples
- Load Checkpoint VAE -> VAE Decode, vae
- VAE Decode IMAGE -> Upscale Image (model), image
- Load Upscale Model UPSCALE_MODEL -> Upscale Image (model), upscale_model
- Upscale Image (model) IMAGE -> Save Image, images

The save prefix writes into a dated subfolder. SaveImage writes PNG with metadata. For metadata-free client deliverables, save as JPEG instead (see [V6 Workflow]({% link comfyui/workflow-v6.md %})).

## Operating notes

- Single touchpoint: per run, the operator edits only the concept in the Primitive node. Everything else stays fixed, in line with the single-operator, minimal-touchpoint design.
- Seed behavior: both KSamplers use a fixed seed. Run-to-run variety comes from the randomized Ollama seed in the options node, which rewrites the scene each run while the anchored subject holds. To vary the image structure as well, set a KSampler seed control to randomize.
- Blank prompt boxes: the three `CLIPTextEncode` text fields read empty during a run because their values arrive over links. This is expected. See [V6 Workflow]({% link comfyui/workflow-v6.md %}) for this and other gotchas.
