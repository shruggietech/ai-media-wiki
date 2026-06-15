---
title: Choosing a Checkpoint
layout: default
parent: ComfyUI
nav_order: 3
---

# Choosing a Checkpoint

Our default pipeline runs Juggernaut XL Ragnarok, an SDXL checkpoint tuned for photorealism. It is the right tool for most of what we produce, but not for everything. This page is about knowing when to reach for it and when to load something else.

## What Juggernaut is built for

Juggernaut Ragnarok is a photorealism-first fine-tune. It is strongest on:

- Realistic people, animals, and landscapes, with convincing skin, fabric, and environmental texture.
- Cinematic and dramatic lighting, the look you want for hero shots and atmospheric scenes.
- Realistic-leaning digital painting. Ragnarok specifically improved at painterly and concept-art styles, so oil-paint, watercolor, and matte-painting looks come out well as long as they stay grounded in realistic shading.

In short, anything that should read as a photograph or a richly rendered painting is in its wheelhouse.

## Where it struggles

The model's realism bias works against you the moment you want a flat or stylized 2D look. It pulls stylized prompts back toward three-dimensional shading and realistic texture, which is exactly what you do not want for:

- Anime, manga, and cel-shaded characters.
- Flat vector-style or sticker-style art.
- Clean line art and other hard-edged 2D illustration.

You can sometimes coax a stylized result out of it, but you are fighting the model the whole way, and the output usually still reads as a realistic render of a cartoon rather than true 2D.

{: .note }
> The distinction that matters is not "photo versus art." Juggernaut handles realistic art fine. The split is realistic or painterly rendering (good) versus flat, stylized 2D (bad).

## What to use instead for 2D

When the brief calls for anime, manga, cel-shaded, or flat illustrated styles, load a checkpoint trained for that look rather than forcing Juggernaut. A dedicated anime or illustration SDXL checkpoint gets you there with far less prompt effort and a cleaner result. Swapping the checkpoint is a one-node change in the workflow (the CheckpointLoader), so it is cheap to keep a second model on hand for these jobs.

## Other limitations to keep in mind

These apply regardless of style, and they are SDXL-tier limits rather than Juggernaut-specific faults:

- Text rendering is weak. Do not rely on it for legible words or logos.
- Faces lose detail at a distance. Crop in or upscale for small or far-away faces.
- It is an SDXL model. For single-reference identity or the very top tier of realism, a Flux-based pipeline (for example Flux into Juggernaut Ragnarok) does better, at the cost of more VRAM and complexity.
