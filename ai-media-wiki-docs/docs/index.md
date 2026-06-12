---
title: Home
layout: default
nav_order: 1
---

# AI Media Wiki

Internal reference for the AI media tooling ShruggieTech runs day to day. Setup
notes, working configs, and the gotchas that cost us an afternoon the first time.

## Sections

- [ComfyUI]({{ site.baseurl }}/comfyui/) — local image and video generation pipelines.
- [Ollama]({{ site.baseurl }}/ollama/) — local LLM serving for prompt generation and tooling.

## How this wiki is organized

Each tool gets a top-level section (a parent page) with child pages underneath
it for specific tasks (installation, models, workflows, troubleshooting). The
sidebar nav is built automatically from the `parent` and `nav_order` fields in
each page's front matter, so adding a page is just a matter of dropping in a new
Markdown file with the right header.
