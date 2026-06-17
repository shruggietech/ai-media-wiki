---
title: Paid Template Limitations
layout: default
parent: ComfyUI
nav_order: 5
---

# Paid Template Limitations

ComfyUI ships a library of built-in template workflows, reached through Workflow > Browse Templates. Most are free and run entirely on local hardware. A subset does not. The Image API and Video API template categories are built around paid API nodes (also called Partner nodes) that run proprietary cloud models and charge per generation. This page records what those paid templates are, how to recognize one before running it, and why the production pipeline does not depend on them.

ComfyUI itself remains free and open source for local use. The limitation here is specific to certain template workflows, not to the application.

## What pay to use means here

A paid template is any built-in workflow that relies on ComfyUI API nodes. These nodes do not run a model on the local GPU. They send the request to a third-party commercial service (for example OpenAI, Black Forest Labs Flux Pro, Kling, Google Veo, Runway, or Stability), and the result returns over the network.

The cost mechanism:

- API nodes draw on prepaid ComfyUI credits, purchased through Settings > Credits. Payment goes to Comfy Org, which forwards to the model provider at roughly the provider's own rate.
- Billing is per generation. Each run of an API node spends credits based on the model, resolution, and quality selected.
- There is no free tier. An API node cannot execute at a zero credit balance.
- The approximate cost is shown as a badge on the API node before the run.

Some API nodes support a bring-your-own-key arrangement, where the operator supplies an existing provider API key instead of ComfyUI credits. That shifts where the bill lands, not the fact that the run is a paid call to an external service.

## How to recognize a paid template

Check before running any unfamiliar template:

- It sits under the Image API or Video API category in Browse Templates.
- Its nodes carry a per-generation credit badge.
- The graph references API or Partner nodes in place of a local CheckpointLoader plus KSampler chain. A free local template loads a checkpoint and samples on the GPU. A paid template hands the prompt to an external node and waits on a response.

If a template will not run without credits, or refuses to run at a zero balance, it is a paid template.

## Why the production pipeline avoids them

The standard ShruggieTech stack is local and free by design. It runs Juggernaut XL Ragnarok on the operator's GPU, with Ollama generating scene text locally. Nothing in the baseline leaves the machine, and a run costs nothing beyond electricity and time. Paid API templates break that model in several ways:

- Cost. Each run spends prepaid credits, and the spend is per generation rather than fixed. Iterating on a paid video template can consume credits quickly and unpredictably.
- Data egress. The prompt, and on image-to-image or video templates the input image, is uploaded to a third-party service. For client work, and especially for real-person likeness material, that is a confidentiality and consent concern the local pipeline does not have.
- Output licensing. A result returned by an external commercial model carries that provider's terms of use. Clearance for the local stack does not extend to it. An API-node output is not automatically cleared for client billing, and each provider's terms have to be checked on their own.
- Offline use and reproducibility. A paid template cannot run without network access and a funded account, and it depends on the external service staying available and unchanged. A local workflow is reproducible from its exported JSON alone.

For the single-operator, minimal-touchpoint pipeline, an external paid dependency adds a funded account, a network requirement, and a per-run cost to a process built to carry none of those.

## Relationship to the commercial licensing gate

This is a separate constraint from the face conditioning licensing gate. The pay-to-use limit is about credits and an external paid service. The licensing gate on IPAdapter FaceID Plus v2 and InstantID is about non-commercial model licenses on a local node. Both restrict what can enter a client-billable workflow, for different reasons. For the face and likeness licensing constraint, see [Baseline Capabilities and Expansion]({% link comfyui/baseline-capabilities.md %}).

## Standing rule

Default to free local templates. Treat any Image API or Video API template as out of scope for client deliverables unless a paid path has been explicitly approved on three points: cost, data handling, and output licensing. Adding an API node to the standard production graph is a deliberate, reviewed decision, not a default.
