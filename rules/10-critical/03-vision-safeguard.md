---
weight: 3
category: critical
description: No image data may be sent to a model that has not been verified as vision-capable
estimated_tokens: 240
---

# Vision Capability Safeguard — STRICT

**Highest priority. Enforced at tool-call boundary, not in prompt.**

## Absolute Rule

**No agent may send image data to a model not verified as vision-capable.**
Doing so produces a deterministic 400 API error, wastes tokens, stalls the session.

## Capability Source of Truth

At SessionStart, `check-model-vision` writes to `~/.your-org/data/model-capabilities.json`.
Contains entries for all declared models + `__active__`. Each entry: `model`, `vision` (true/false/null), `ollama_model`.

## Agent Behavior

1. **Before any screenshot, image read, or vision MCP tool call**, read `model-capabilities.json`.
1. If `vision: true` → proceed.
1. If `vision: false` or `null` → **abort**. Report to user that model lacks vision.
1. If file missing or stale, run `check-model-vision` synchronously and re-read.

## Fallback Heuristics (Ollama unreachable, no capability file)

- Base name contains `vl`, `vision`, `llava`, `bakllava`, `moondream`, `minicpm-v`, `kimi-k2` → presume capable.
- All others → presume **not** capable.

## No Silent Degradation

Never silently skip image input. Escalate to vision-capable model or request human guidance.

## Enforcement

- This rule is loaded after `00-human-sovereignty.md` and `00-capability-boundaries.md`.
- Violations are logged as medium-severity economy errors via `record_error`.
