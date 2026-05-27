# Decision: Model-Tool Template Privacy Architecture

**Date:** 2026-05-27
**Meeting Type:** Design Review
**Participants:** Architect, Critic, Stakeholder, Decider

## Context

The team needed to decide on an architecture for protecting model-tool templates from casual inspection while balancing debuggability, honest privacy claims, and delivery timeline.

## Decision

**Ship Hybrid Architecture: .pyc MVP + Remote Validation Phase 2**

| Layer                    | Implementation                                        | Timeline  |
| ------------------------ | ----------------------------------------------------- | --------- |
| **MVP (Phase 1)**        | Compiled `.pyc` templates with source deletion        | 1-2 days  |
| **Production (Phase 2)** | Remote validation service with signed template hashes | 1-2 weeks |
| **Enterprise**           | `--export-audit` flag generating signed attestations  | Phase 3   |

### Honest Privacy Claim

> Template Privacy: Model-tool templates are compiled to Python bytecode (.pyc) to prevent casual inspection. This provides obfuscation, not cryptographic protection. Determined attackers with local access can recover source via decompilation tools.

**Forbidden claims:** "encrypted", "secure", "tamper-proof", "cannot be inspected"

## Rationale

1. `.pyc` is sufficient friction — Raises bar from "curious developer reads template" to "determined attacker brings decompiler."
1. Remote validation is the real moat — Phase 2 ensures templates haven't been modified before execution.
1. Timeline is realistic — MVP in 1-2 days unblocks deployment.
1. Enterprise audit path is explicit — `--export-audit` command generates verifiable artifacts.

## Dissent Noted

**Critic's concerns:**

1. `.pyc` creates debugging blind spots — Mitigation: Dev mode ships plaintext templates
1. Timeline risk on remote validation — Mitigation: Phase 2 scoped to HMAC signing only
1. Templates may not be actual moat — Will be evaluated during Phase 2 design review

## Action Items

| Item                                            | Owner       | Deadline   |
| ----------------------------------------------- | ----------- | ---------- |
| Implement .pyc compilation with source deletion | Architect   | 2026-05-28 |
| Add dev/prod mode flag                          | Architect   | 2026-05-28 |
| Draft honest privacy wording for docs           | Stakeholder | 2026-05-28 |
| Design remote validation service API            | Architect   | 2026-05-30 |
| Critic review of .pyc implementation            | Critic      | 2026-05-29 |
