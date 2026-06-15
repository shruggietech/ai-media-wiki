---
title: Using Ollama with ComfyUI
layout: default
parent: Ollama
nav_order: 3
---

# Using Ollama with ComfyUI

Ollama is the prompt engine for the image pipeline. ComfyUI sends a concept to Ollama, and Ollama writes the positive and negative SDXL prompts that drive generation. This page covers the connection itself. For how those prompts feed the graph, see the [V6 Workflow]({{ site.baseurl }}/comfyui/workflow-v6/) page.

## The connection

ComfyUI talks to Ollama over HTTP at `http://127.0.0.1:11434`. That address is stored in the workflow, so Ollama has to be running at that port whenever you generate. If you have moved Ollama to another host or port with `OLLAMA_HOST`, update the address in the `OllamaConnectivityV2` node to match.

## The nodes

The integration uses the V2 generation of the `comfyui-ollama` nodes:

- `OllamaConnectivityV2` holds the server address and the model name (`llama3.1:8b`).
- `OllamaGenerateV2` sends a prompt to the model and returns its text.
- `OllamaOptionsV2` carries generation options, including the seed and the fix-or-randomize toggle.

These replaced the older single `OllamaGenerate` node. A workflow built on the deprecated node will not map onto the current setup, so install or update `comfyui-ollama` through ComfyUI Manager and restart ComfyUI if the V2 nodes are missing.

## Seeds and the fix-or-randomize toggle

The shared `OllamaOptionsV2` seed drives the prompt builders. Fixing the seed makes Ollama produce the same prompt for the same input, which is useful when you are dialing in a subject and want to change one thing at a time. Randomizing it lets the scene and atmosphere vary from run to run.

## Caching: why Ollama sometimes does not re-run

ComfyUI caches node output by input signature. If nothing feeding an Ollama node has changed (same concept, same seed, same options), ComfyUI returns the cached text and Ollama does not execute again. This is expected and saves time. To force a fresh prompt, change the input or randomize the seed. Clearing the cache is not what you want here, because a randomized seed already forces a fresh run on its own.
