---
title: Installation
layout: default
parent: Ollama
nav_order: 1
---

# Installing Ollama

Ollama is a separate program from ComfyUI. It must be installed and actively running in the background, or every ComfyUI generation fails the moment it reaches the prompt-building nodes. This is the single most common reason a fresh setup does not run.

## Steps

1. Install Ollama for the target operating system from [ollama.com](https://ollama.com).

2. Pull the model the workflow expects:

```bash
   ollama pull llama3.1:8b
```

3. Leave Ollama running. The ComfyUI workflow communicates with it at `http://127.0.0.1:11434`. That address is baked into the workflow, so the service has to be live at that port whenever you generate.

## Confirm it is running

Open `http://127.0.0.1:11434` in a browser. A short "Ollama is running" response means the service is live and ComfyUI can reach it.

{: .warning }
> If generation in ComfyUI fails at the prompt nodes, check this first. A stopped Ollama service is the most likely cause before you start debugging the workflow itself.