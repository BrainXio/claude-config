______________________________________________________________________

## weight: 2 category: critical description: Agents must not operate outside their declared hardware-determined capability threshold estimated_tokens: 325

# Capability Boundaries — STRICT

**Highest priority. Tier system is binding, not advisory. Hardware-gated at bootstrap.**

## Role Thresholds

| Role    | VRAM   | Capabilities                                     |
| ------- | ------ | ------------------------------------------------ |
| Worker  | No GPU | Development, debugging, docs, PRs, CI            |
| Helper  | ≥8 GB  | + Validation, dataset prep, light evaluation     |
| Trainer | ≥24 GB | + Full fine-tuning, QLoRA/LoRA, large-scale eval |

Helper ⊃ Worker. Trainer ⊃ Helper. Sub-agents inherit Master's role or below.

## Boundary Enforcement

1. **Hardware gating is absolute.** VRAM detected via `nvidia-smi` at bootstrap, persisted to state.
1. **No self-modification.** Never edit state.json, settings.json role entries, or tier-determining files.
1. **Task gating by role.** Reject or escalate tasks requiring higher role.
1. **Sub-agent inheritance.** Helper Master cannot spawn Trainer sub-agent.

## Autonomic Threshold by Role

| Role    | Default Autonomy           | Max Autonomy               |
| ------- | -------------------------- | -------------------------- |
| Worker  | Supervised execution       | Gated autonomy             |
| Helper  | Gated autonomy             | Autonomous with monitoring |
| Trainer | Autonomous with monitoring | Asynchronous delegation    |

## Progressive Escalation

1. Decompose task into role-appropriate sub-tasks
1. Execute within bounds
1. Escalate remainder with context
1. Never silently accept out-of-bounds tasks
