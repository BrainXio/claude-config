# SETUP-OLLAMA.md

Detailed Ollama provider setup for claude-config. This file covers both
Ollama Cloud and local Ollama, including the hardware check and
quantization selection procedure.

## Ollama Cloud

Ollama Cloud uses `:cloud`-tagged models served through a local Ollama
proxy. No hardware check is needed — models run on remote infrastructure.

Copy `settings/env-ollama-cloud.json` after `settings/env.json` in the
settings composition order. It configures:

- `ANTHROPIC_BASE_URL`: `http://localhost:11434`
- `ANTHROPIC_AUTH_TOKEN`: `ollama`
- Default models: `deepseek-v4-pro:cloud`, `kimi-k2.6:cloud`,
  `qwen3.5:cloud`, `qwen3-coder-next:cloud`

No further steps required. Return to the main [AGENTS.md](AGENTS.md)
setup flow.

## Local Ollama

Local Ollama runs models directly on the host hardware. A hardware check
is required before setup — insufficient VRAM will cause out-of-memory
crashes at runtime.

### Step 1: Read configured models

Read `settings/env-ollama-local.json` and extract every model name from
the env values (e.g. `qwen3:32b`, `qwen3-coder:latest`).

### Step 2: Check available VRAM

Run:

```bash
nvidia-smi \
  --query-gpu=memory.total,memory.free \
  --format=csv,noheader,nounits 2>&1
```

If `nvidia-smi` fails (no NVIDIA GPU), there is no dedicated VRAM. Warn
the human:

> "No NVIDIA GPU detected. Local Ollama will fall back to CPU inference
> and will be extremely slow. Consider switching to Ollama Cloud instead.
> Proceed with local anyway?"

If the human insists, proceed with VRAM = 0 and skip to Step 5 (all
models will exceed the budget — the human must choose quantizations
knowingly).

### Step 3: Estimate VRAM requirements

For each model, estimate the VRAM needed at default Q4_K_M quantization:

- Parse the parameter size from the tag: `32b` → 32B, `8b` → 8B.
- For models without an explicit size (e.g. `qwen3-coder:latest`), fetch
  `https://ollama.com/library/{base_model}/tags` and find the size from
  the tag listing.
- **Rule of thumb at Q4_K_M: ~0.7 GB per billion parameters**, plus 2 GB
  overhead for context.
  - A 32B model needs ~24 GB.
  - A 14B model needs ~12 GB.
  - An 8B model needs ~8 GB.
- Sum the primary model and sub-agent model (they may be loaded
  concurrently).

### Step 4: If VRAM is sufficient

If available VRAM meets or exceeds the total, no changes are needed.
Copy `settings/env-ollama-local.json` after `settings/env.json` and
return to the main setup flow.

### Step 5: If VRAM falls short — quantization selection

For each model that is too large for the available VRAM:

1. **Extract the base model name** by stripping the tag:
   `qwen3:32b` → base `qwen3`.

1. **Fetch available tags:**
   `https://ollama.com/library/{base_model}/tags`

1. **Find smaller quantizations** for the same parameter size on the
   tags page. Quantization suffixes follow this convention:
   `-q8_0`, `-q6_K`, `-q5_K_M`, `-q4_K_M`, `-q3_K_M`, `-q2_K`,
   `-IQ3_XXS`, `-IQ2_XXS`.

1. **Estimate VRAM per quantization level** (always add 2 GB overhead):

   | Quantization | GB per B params |
   | ------------ | --------------- |
   | `-fp16`      | 2.0             |
   | `-q8_0`      | 1.0             |
   | `-q6_K`      | 0.8             |
   | `-q5_K_M`    | 0.75            |
   | `-q4_K_M`    | 0.7             |
   | `-q3_K_M`    | 0.55            |
   | `-q2_K`      | 0.45            |
   | `-IQ3_XXS`   | 0.4             |
   | `-IQ2_XXS`   | 0.35            |

1. **Pick the largest quantization that fits** the available VRAM budget
   for each model.

### Step 6: Present options to the human

> "Your GPU has X GB VRAM. The configured models need ~Y GB at default
> Q4_K_M quantization. This won't fit. Available smaller quantizations
> that would fit:
>
> - `model:tag-q3_K_M` (~Z GB)
> - `model:tag-q2_K` (~W GB)
>
> Which would you like to use? I'll update the settings file
> accordingly."

Wait for the human to choose. Once confirmed, copy
`settings/env-ollama-local.json` after `settings/env.json`, patching the
model tags with the chosen quantization suffixes.
