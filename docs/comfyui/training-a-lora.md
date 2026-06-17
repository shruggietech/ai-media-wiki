---
title: Training a LoRA
layout: default
parent: ComfyUI
nav_order: 6
---

# Training a LoRA

A LoRA is a small adapter that teaches the base checkpoint a specific subject, character, or style without retraining the whole model. It does not change the model you use itself, in our cases, Juggernaut XL Ragnarok. It loads alongside the checkpoint at generation time, adds tens of megabytes rather than gigabytes, and can be swapped or stacked per job. For ShruggieTech the main use is a persona identity: a reusable face and look that conditions output from owned, cleared training data, which is the commercial-clearable alternative to the non-commercial face-conditioning tools described under [Baseline Capabilities and Expansion]({% link comfyui/baseline-capabilities.md %}).

Training and generation are two separate stages. Training happens in a dedicated trainer (Kohya SS sd-scripts is the standard). The finished LoRA is then used inside ComfyUI. This page covers the SDXL path on the standard rig, with a character or persona LoRA as the primary example. The principles carry over to style and concept LoRAs.

## Hardware reality

This is the first thing to settle, because it changes where training happens.

SDXL LoRA training has a practical floor of about 12 GB VRAM. The standard rig is an RTX 2080 Ti at 11 GB, which sits below that floor. Training an SDXL LoRA on 11 GB is possible only with aggressive memory settings, and even then it is slow and prone to out-of-memory failures. Generating with a finished LoRA, by contrast, is cheap and runs fine on the standard rig.

The recommended approach separates the two stages:

- Train on a rented cloud GPU (for example a 16 GB or 24 GB instance). Training is a one-time cost per LoRA.
- Download the resulting `.safetensors` file and run it locally on the standard rig forever. Generation is the recurring use, and it has no cloud dependency.

If training locally on the 11 GB card anyway, the settings that make SDXL fit are: train the UNet only (`--network_train_unet_only`), use the Adafactor optimizer (which enables the fused backward pass, the main SDXL memory saving), enable gradient checkpointing, cache latents to disk, use mixed precision (bf16 or fp16), set batch size to 1, and close other GPU applications. Expect long run times.

Base model scope: this page describes the current SDXL pipeline. A Flux-based identity path (PuLID-Flux) is under consideration and would change both the trainer and the VRAM math. It is not the current process.

## Dataset

The dataset is the single largest factor in LoRA quality. Most of the result is decided here, before any training setting is touched.

- Quantity: a character or persona LoRA trains well on roughly 15 to 40 images. More images do not help if they are repetitive. Quality and variety beat raw count.
- Variety: include a mix of framing (close face, half body, full body), angles, expressions, lighting, and backgrounds. A dataset that is all one pose or one outfit teaches the LoRA that pose or outfit, and the result will refuse to vary.
- Consistency: the identity must stay constant across the set. Everything around it should change. The contrast between what stays fixed and what varies is what the LoRA learns to attach to the subject.
- Image quality: sharp, in focus, high resolution. Remove text and watermarks. Keep one clear subject per image. Avoid hands or objects across the face, which corrupt facial learning.
- Resolution: source at 1024 x 1024 or larger for SDXL. Enable aspect-ratio bucketing in the trainer so images of different shapes are used without cropping to square.

## Captioning

Each training image gets a text caption, and the captioning strategy decides how flexible the finished LoRA is.

The governing rule: caption what should remain variable, and leave the permanent identity uncaptioned so it binds to the trigger token.

- Trigger token: choose one short, rare token (a unique persona name works) and place it at the front of every caption. This token becomes the prompt handle that summons the LoRA at generation time.
- Caption the variable elements: background, setting, pose, expression, clothing, camera angle, and lighting. Naming these tells the LoRA they are interchangeable, which keeps them controllable later.
- Do not caption the fixed identity: face structure, and for a consistent persona also eye color, hair color, and skin tone. Anything left uncaptioned is absorbed into the trigger token. Tagging a permanent feature makes it detachable, so the identity weakens.
- Tag recurring accessories that should be optional (glasses, a specific hat) so they do not appear in every render.
- Keep caption style consistent across the whole set.

## Training parameters

Treat these as starting points to validate by test renders, not fixed values. The right numbers depend on the dataset and the optimizer, so adjust based on results rather than trusting any single setting blind.

- Network rank (dim): 32 is a solid default for a character or persona. Higher ranks add capacity and file size but also overfitting risk, and are rarely needed. Drop to a lower rank if NaN errors appear.
- Network alpha: set equal to the rank, or to half the rank for a gentler effect.
- Optimizer: Adafactor (pairs with the fused backward pass for lower VRAM) or AdamW8bit.
- Learning rate: a tuning dial tied to the optimizer. Start from the trainer's documented default for the chosen optimizer, then raise it (or train longer) if the LoRA underfits, or lower it if it overfits.
- Total steps: aim for roughly 4000 to 5000 steps for a character. Set image repeats and epoch count from the dataset size to land in that range; larger datasets need fewer repeats.
- Resolution: 1024 x 1024 with bucketing enabled.
- UNet only: train the UNet only for SDXL, which avoids erratic results from the model's dual text encoders.

## Validation and overfitting

- Save the LoRA at multiple points across training, not just the final step, and test each one. The best epoch is often not the last.
- Underfitting looks like a trigger token that barely changes the image or an identity that is not captured. Fix with more steps, a higher learning rate, or better data.
- Overfitting looks like every render reproducing a training image, poses or outfits that will not change, baked-in artifacts, or the model ignoring the rest of the prompt. Fix with fewer steps, a lower learning rate or rank, or a more varied dataset.
- Test in varied prompts and scenes, not just recreations of the training images, and test at several strengths.

## Using the LoRA in ComfyUI

- Place the trained `.safetensors` file in `models/loras`.
- Add a LoraLoader node between the CheckpointLoader and the sampler, feeding the same Juggernaut XL Ragnarok base it was trained against.
- Include the trigger token in the positive prompt to activate the learned subject.
- Strength: start near 0.7 to 1.0 and tune. Lower strength loosens adherence and allows more prompt freedom; higher strength locks the identity harder at the cost of flexibility.
- LoRAs stack, but each added LoRA competes for influence and for VRAM headroom against the 11 GB ceiling. Keep stacks minimal, in line with the single-operator, minimal-touchpoint design.

## Licensing and consent

Training data must be owned or licensed. For a real-person likeness, a signed consent and release is required before any image enters the dataset, and the real-person likeness intake process applies.

A LoRA trained on cleared data is commercial-usable. That is the point of the LoRA path for persona work: it provides reusable identity conditioning without the non-commercial insightface license that gates IPAdapter FaceID Plus v2 and InstantID. For the full commercial clearance picture, see [Baseline Capabilities and Expansion]({% link comfyui/baseline-capabilities.md %}).
