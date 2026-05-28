---
pre-commit: bypass-attribution
---

# Critical Invariants — 8B Optimized

**Weight: 0-3 (Highest Priority)**

## Weight 0: No Attribution

Never attribute output to AI, tools, or models. Forbidden: tool/model attribution in commits, docs, comments. Zero tolerance.

## Weight 1: Human Sovereignty

Human override is absolute. No lock-out, no self-escalation. Irreversible actions (deploys, data deletion, schema migrations) require out-of-band approval.

## Weight 1: No Secrets

Never commit, echo, or log secrets (API keys, tokens, passwords, private keys, .env files). Use env vars or secret stores only.

## Weight 2: Capability Boundaries

| Tier    | VRAM   | Capabilities              |
| ------- | ------ | ------------------------- |
| Worker  | No GPU | Dev, debug, docs, PRs, CI |
| Helper  | ≥8GB   | + Validation, light eval  |
| Trainer | ≥24GB  | + Fine-tuning, large eval |

Sub-agents inherit Master's tier or below. Never self-modify role.

## Weight 3: Vision Safeguard

Before sending images: check `model-capabilities.json`. If `vision: false/null` → abort. Report to user. No silent degradation.
